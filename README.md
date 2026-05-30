--[[
    ONYX HUB FEATURES + RAYFIELD GUI
    - Aimbot (PC/Mobile/Hotkey/Smooth/FOV/Wall/Team)
    - Hitbox Expandida e LEGIT
    - Auto Farm Gari (caminhando + desvio de obstáculos)
    - Nitro Automático (tecla N ou botão mobile)
    - Noclip, Anti-Sit, Anti-AFK, Spinbot
    - Speed Car e Fly Car
    - Whitelist (amigos não são mirados)
    - Salvar/Carregar configurações (executor com FS)
    - Temas do Rayfield (escuro/padrão)
--]]

-- Carregar Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Serviços
local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Debris = game:GetService("Debris")
local Workspace = game:GetService("Workspace")

-- ============================ CONFIGURAÇÕES INICIAIS ============================
local Toggles = {
    Aimbot = false,
    AimbotMobile = false,
    AimbotMobileLook = false,
    AimbotHotkey = false,
    AimbotSmooth = false,
    CheckTeam = false,
    CheckWall = false,
    Spinbot = false,
    Noclip = false,
    AntiSit = true,
    AntiAfk = true,
    UseWhitelist = false,
    HitboxExpanded = false,
    LegitHitbox = false,
    SpeedCar = false,
    VFly = false,
    AutoFarmGari = false,
    NitroAuto = false,
}
local Whitelist = {}
local AimbotConfig = {
    FOVRadius = 150,
    HitPart = "Head",
    SpinSpeed = 50,
    HeadSize = 1.0,
    HeadTransparency = 0,
    SmoothSpeed = 0.15,
    Hotkey = {Kind = "KeyCode", Key = Enum.KeyCode.E},
}
local hitboxSize = 15
local hitboxTransparency = 0.7
local legitTam = 6
local FOVColor = Color3.fromRGB(255, 20, 147) -- rosa
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local hasDrawing = pcall(function() return Drawing end) and Drawing ~= nil

local nitroForce = 50
local nitroConnection = nil
local nitroButtonMobile = nil
local TELEPORT_DELAY_SECONDS = 10
local carVelocity = 0
local carSensitivity = 2

-- Variáveis de controle
local noclipConn, antiSitConn, antiAfkThread, spinbotConn
local aimbotThread, mobileAutoAimConn, mobileLookConn, headSizeConn
local smoothAimbotConn, hotkeyAimbotActive = nil, false
local hitboxConn, hitboxOriginais, originaisLegit, visualPartsLegit = nil, {}, {}, {}
local fovCircle
local aimbotActive = false

-- ============================ FUNÇÕES AUXILIARES ============================
local function GetChar() return LP.Character end
local function GetHRP() local c = GetChar(); return c and c:FindFirstChild("HumanoidRootPart") end
local function IsWhitelisted(plr)
    if not Toggles.UseWhitelist then return false end
    return Whitelist[plr.Name] == true
end
local function notify(title, text, dur)
    Rayfield:Notify({Title = title, Content = text, Duration = dur or 2})
end

-- ============================ NOCLIP / ANTI-SIT / ANTI-AFK / SPINBOT ============================
local function ToggleNoclip(s)
    Toggles.Noclip = s
    if s then
        if noclipConn then return end
        noclipConn = RunService.Stepped:Connect(function()
            if not Toggles.Noclip then return end
            local ch = GetChar()
            if not ch then return end
            for _, p in ipairs(ch:GetDescendants()) do
                if p:IsA("BasePart") then pcall(function() p.CanCollide = false end) end
            end
        end)
        notify("Noclip", "Ativado ✓")
    else
        if noclipConn then noclipConn:Disconnect(); noclipConn = nil end
        local ch = GetChar()
        if ch then
            for _, p in ipairs(ch:GetDescendants()) do
                if p:IsA("BasePart") then pcall(function() p.CanCollide = true end) end
            end
        end
        notify("Noclip", "Desativado ✕")
    end
end

local function ToggleAntiSit(s, silent)
    Toggles.AntiSit = s
    if s then
        if antiSitConn then return end
        antiSitConn = RunService.Heartbeat:Connect(function()
            if not Toggles.AntiSit then return end
            local ch = GetChar()
            if not ch then return end
            local h = ch:FindFirstChildOfClass("Humanoid")
            if h then
                if h.Sit then h.Sit = false end
                if h:GetState() == Enum.HumanoidStateType.Seated then
                    h:ChangeState(Enum.HumanoidStateType.Running)
                end
            end
        end)
        if not silent then notify("Anti-Sit", "Ativado ✓") end
    else
        if antiSitConn then antiSitConn:Disconnect(); antiSitConn = nil end
        if not silent then notify("Anti-Sit", "Desativado ✕") end
    end
end

local function ToggleAntiAfk(s)
    Toggles.AntiAfk = s
    if s then
        if antiAfkThread then return end
        antiAfkThread = task.spawn(function()
            local t = false
            while Toggles.AntiAfk do
                task.wait(50)
                local ch = GetChar()
                local h = ch and ch:FindFirstChildOfClass("Humanoid")
                if h then
                    t = not t
                    pcall(function()
                        h:Move(t and Vector3.new(0.1, 0, 0) or Vector3.new(-0.1, 0, 0), false)
                        task.wait(0.1)
                        h:Move(Vector3.new(), false)
                    end)
                end
            end
        end)
        notify("Anti-AFK", "Ativado ✓")
    else
        if antiAfkThread then pcall(task.cancel, antiAfkThread); antiAfkThread = nil end
        notify("Anti-AFK", "Desativado ✕")
    end
end

local function ToggleSpinbot(s)
    Toggles.Spinbot = s
    if s then
        if spinbotConn then return end
        spinbotConn = RunService.Heartbeat:Connect(function()
            if not Toggles.Spinbot then return end
            local h = GetHRP()
            if h then
                pcall(function()
                    h.CFrame = h.CFrame * CFrame.Angles(0, math.rad(AimbotConfig.SpinSpeed), 0)
                end)
            end
        end)
        notify("Spinbot", "Ativado ✓")
    else
        if spinbotConn then spinbotConn:Disconnect(); spinbotConn = nil end
        notify("Spinbot", "Desativado ✕")
    end
end

-- ============================ HITBOX ============================
local function expandirUpperTorso(ch)
    local p = ch and ch:FindFirstChild("UpperTorso")
    if p and p:IsA("BasePart") then
        if not hitboxOriginais[p] then
            hitboxOriginais[p] = {
                Size = p.Size, Transparency = p.Transparency, Material = p.Material,
                Color = p.Color, CanCollide = p.CanCollide, Massless = p.Massless
            }
        end
        pcall(function()
            p.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
            p.Transparency = hitboxTransparency
            p.Material = Enum.Material.Neon
            p.Color = Color3.fromRGB(255, 0, 0)
            p.CanCollide = false
            p.Massless = true
        end)
    end
end

local function criarVisualPart(t)
    if visualPartsLegit[t] then return end
    local ok, v = pcall(function() return t:Clone() end)
    if not ok or not v then return end
    v.Size = t.Size
    v.CFrame = t.CFrame
    v.Anchored = false
    v.CanCollide = false
    v.Transparency = 0
    v.Parent = t.Parent
    local w = Instance.new("Weld")
    w.Part0 = t
    w.Part1 = v
    w.Parent = v
    visualPartsLegit[t] = v
end

local function expandirTronco(ch)
    local t = ch and ch:FindFirstChild("UpperTorso")
    if t and t:IsA("BasePart") then
        if not originaisLegit[t] then
            originaisLegit[t] = {Size = t.Size, Transparency = t.Transparency}
            criarVisualPart(t)
        end
        pcall(function()
            t.Size = Vector3.new(legitTam, legitTam, legitTam)
            t.Transparency = 1
        end)
    end
end

local function startHitboxLoop()
    if hitboxConn then return end
    hitboxConn = RunService.RenderStepped:Connect(function()
        if Toggles.HitboxExpanded then
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= LP and plr.Character then pcall(expandirUpperTorso, plr.Character) end
            end
        end
        if Toggles.LegitHitbox then
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= LP and plr.Character then pcall(expandirTronco, plr.Character) end
            end
        end
    end)
end

local function stopAndRestoreHitbox()
    if hitboxConn then hitboxConn:Disconnect(); hitboxConn = nil end
    for p, o in pairs(hitboxOriginais) do
        if p and p.Parent and o then
            pcall(function()
                p.Size = o.Size
                p.Transparency = o.Transparency
                p.Material = o.Material
                p.Color = o.Color
                p.CanCollide = o.CanCollide
                p.Massless = o.Massless
            end)
        end
        hitboxOriginais[p] = nil
    end
    for t, o in pairs(originaisLegit) do
        if t and t.Parent and o then
            pcall(function()
                t.Size = o.Size
                t.Transparency = o.Transparency
            end)
        end
        if visualPartsLegit[t] then
            pcall(function() visualPartsLegit[t]:Destroy() end)
            visualPartsLegit[t] = nil
        end
        originaisLegit[t] = nil
    end
end

-- ============================ AIMBOT ============================
local function ApplyHeadSize()
    local ch = GetChar()
    if not ch then return end
    local h = ch:FindFirstChild("Head")
    if h and h:IsA("BasePart") then
        pcall(function()
            h.Size = Vector3.new(AimbotConfig.HeadSize, AimbotConfig.HeadSize, AimbotConfig.HeadSize)
            h.Transparency = AimbotConfig.HeadTransparency
        end)
    end
end

local function StartHeadSizeLoop()
    if headSizeConn then headSizeConn:Disconnect() end
    headSizeConn = RunService.Heartbeat:Connect(function()
        if Toggles.Aimbot or Toggles.AimbotMobile or Toggles.AimbotHotkey or Toggles.AimbotMobileLook then
            ApplyHeadSize()
        end
    end)
end

local function GetValidPlayers()
    local list = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p == LP then continue end
        if IsWhitelisted(p) then continue end
        if Toggles.CheckTeam then
            local myTeam = LP.Team
            local theirTeam = p.Team
            if myTeam and theirTeam and myTeam == theirTeam then continue end
        end
        local ch = p.Character
        if not ch then continue end
        local h = ch:FindFirstChildOfClass("Humanoid")
        if h and h.Health > 0 then table.insert(list, p) end
    end
    return list
end

local function IsVisible(part)
    if not Toggles.CheckWall then return true end
    local hrp = GetHRP()
    if not hrp then return false end
    local rp = RaycastParams.new()
    rp.FilterType = Enum.RaycastFilterType.Blacklist
    rp.FilterDescendantsInstances = {GetChar()}
    local ray = Workspace:Raycast(hrp.Position, (part.Position - hrp.Position).Unit * 500, rp)
    if ray and ray.Instance then
        return ray.Instance:IsDescendantOf(part.Parent) or ray.Instance == part
    end
    return true
end

local function GetClosestInFOV()
    local cam = Camera
    if not cam then return nil end
    local cx, cy = cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2
    local ref = (isMobile or Toggles.AimbotMobileLook) and Vector2.new(cx, cy) or UserInputService:GetMouseLocation()
    local best, bestD = nil, AimbotConfig.FOVRadius
    for _, p in ipairs(GetValidPlayers()) do
        local ch = p.Character
        if not ch then continue end
        local part = ch:FindFirstChild(AimbotConfig.HitPart) or ch:FindFirstChild("Head") or ch:FindFirstChild("HumanoidRootPart")
        if not part then continue end
        if not IsVisible(part) then continue end
        local ok, sp, onScr = pcall(function() return cam:WorldToViewportPoint(part.Position) end)
        if ok and onScr then
            local d = (Vector2.new(sp.X, sp.Y) - ref).Magnitude
            if d < bestD then
                bestD = d
                best = p
            end
        end
    end
    return best
end

local function DoAim()
    local tg = GetClosestInFOV()
    if not tg then return end
    local ch = tg.Character
    if not ch then return end
    local part = ch:FindFirstChild(AimbotConfig.HitPart) or ch:FindFirstChild("Head") or ch:FindFirstChild("HumanoidRootPart")
    if part and Camera then
        pcall(function()
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, part.Position)
        end)
    end
end

local function DoSmoothAim()
    local tg = GetClosestInFOV()
    if not tg then return end
    local ch = tg.Character
    if not ch then return end
    local part = ch:FindFirstChild(AimbotConfig.HitPart) or ch:FindFirstChild("Head") or ch:FindFirstChild("HumanoidRootPart")
    if part and Camera then
        pcall(function()
            local targetCF = CFrame.new(Camera.CFrame.Position, part.Position)
            Camera.CFrame = Camera.CFrame:Lerp(targetCF, AimbotConfig.SmoothSpeed)
        end)
    end
end

local function AimbotLoopPC()
    while Toggles.Aimbot do
        if Toggles.AimbotSmooth then
            DoSmoothAim()
        else
            local aiming = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
            if aiming then DoAim() end
        end
        task.wait()
    end
end

local function SetupMobileAutoAim()
    if mobileAutoAimConn then mobileAutoAimConn:Disconnect() end
    mobileAutoAimConn = RunService.RenderStepped:Connect(function()
        if Toggles.AimbotMobile and aimbotActive then
            if Toggles.AimbotSmooth then DoSmoothAim() else DoAim() end
        end
    end)
end

local function SetupMobileLookAim()
    if mobileLookConn then mobileLookConn:Disconnect() end
    mobileLookConn = RunService.RenderStepped:Connect(function()
        if Toggles.AimbotMobileLook then
            if Toggles.AimbotSmooth then DoSmoothAim() else DoAim() end
        end
    end)
end

local function IsHotkeyMatch(input)
    local h = AimbotConfig.Hotkey
    if not h or not h.Key then return false end
    if h.Kind == "KeyCode" then
        return input.KeyCode == h.Key
    elseif h.Kind == "UserInputType" then
        return input.UserInputType == h.Key
    end
    return false
end

local function SetupHotkeyAimbot()
    if smoothAimbotConn then smoothAimbotConn:Disconnect() end
    smoothAimbotConn = RunService.RenderStepped:Connect(function()
        if not Toggles.AimbotHotkey or not hotkeyAimbotActive then return end
        if Toggles.AimbotSmooth then DoSmoothAim() else DoAim() end
    end)
end

UserInputService.InputBegan:Connect(function(input)
    if Toggles.AimbotHotkey and IsHotkeyMatch(input) then hotkeyAimbotActive = true end
end)
UserInputService.InputEnded:Connect(function(input)
    if IsHotkeyMatch(input) then hotkeyAimbotActive = false end
end)

local function CreateFOVCircle()
    if fovCircle then pcall(function() fovCircle:Remove() end); fovCircle = nil end
    if not hasDrawing then return end
    local c = Drawing.new("Circle")
    if not c then return end
    pcall(function()
        c.Visible = false
        c.Radius = AimbotConfig.FOVRadius
        c.Color = FOVColor
        c.Thickness = 1
        c.Filled = false
        c.Transparency = 1
    end)
    fovCircle = c
    RunService.RenderStepped:Connect(function()
        if not fovCircle then return end
        pcall(function()
            if isMobile and Camera then
                fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
            else
                fovCircle.Position = UserInputService:GetMouseLocation()
            end
            fovCircle.Visible = Toggles.Aimbot or Toggles.AimbotMobile or Toggles.AimbotHotkey or Toggles.AimbotMobileLook
            fovCircle.Radius = AimbotConfig.FOVRadius
            fovCircle.Color = FOVColor
        end)
    end)
end

-- ============================ AUTO FARM GARI ============================
local function findTrashEntries()
    local list = {}
    local lixeiro = Workspace:FindFirstChild("Lixeiro")
    if not lixeiro then
        for _, v in ipairs(Workspace:GetChildren()) do
            if string.find(string.lower(v.Name), "lixeiro") then lixeiro = v; break end
        end
    end
    if not lixeiro then return list end

    local function isPromptMatch(prompt)
        if not prompt then return false end
        local lname = string.lower(prompt.Name or "")
        local action = string.lower(prompt.ActionText or "")
        return lname:find("peg") or lname:find("lixo") or action:find("peg") or action:find("lixo")
    end

    local function inspectModel(model)
        for _, desc in ipairs(model:GetDescendants()) do
            if desc:IsA("ProximityPrompt") and isPromptMatch(desc) then
                table.insert(list, {target = model, prompt = desc})
                return
            end
        end
        for _, child in ipairs(model:GetDescendants()) do
            if (child:IsA("RemoteEvent") or child:IsA("RemoteFunction")) and string.find(string.lower(child.Name or ""), "peg") then
                table.insert(list, {target = model, remote = child})
                return
            end
        end
        local remote = model:FindFirstChild("PegarLixo") or model:FindFirstChild("pegarLixo") or model:FindFirstChild("pegaLixo")
        if remote and (remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction")) then
            table.insert(list, {target = model, remote = remote})
            return
        end
        local folder = model:FindFirstChild("lixo automatico")
        if folder and folder:IsA("Folder") then
            for _, desc in ipairs(folder:GetDescendants()) do
                if desc:IsA("ProximityPrompt") and isPromptMatch(desc) then
                    table.insert(list, {target = model, prompt = desc})
                    return
                elseif (desc:IsA("RemoteEvent") or desc:IsA("RemoteFunction")) and string.find(string.lower(desc.Name or ""), "pegar") then
                    table.insert(list, {target = model, remote = desc})
                    return
                end
            end
        end
    end

    for _, child in ipairs(lixeiro:GetChildren()) do
        if child:IsA("Model") or child:IsA("Folder") or child:IsA("BasePart") then
            pcall(function() inspectModel(child) end)
        end
    end
    return list
end

local function getModelBasePart(model)
    if not model then return nil end
    if model:IsA("BasePart") then return model end
    if model.PrimaryPart and model.PrimaryPart:IsA("BasePart") then return model.PrimaryPart end
    for _, v in ipairs(model:GetDescendants()) do
        if v:IsA("BasePart") then return v end
    end
    return nil
end

local function walkToPosition(targetPos)
    local char = LP.Character
    if not char then return false end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not humanoid or not hrp then return false end

    local DISTANCE_THRESHOLD = 4
    local CHECK_INTERVAL = 0.1
    local TIMEOUT = 12
    local STUCK_TIMEOUT = 2
    local SIDE_STEP_DISTANCE = 3

    local startTime = tick()
    local lastDist = (hrp.Position - targetPos).Magnitude
    local lastStuckCheck = startTime
    local stuckCount = 0

    humanoid.AutoRotate = true

    local function hasObstacle(direction)
        local rayOrigin = hrp.Position + Vector3.new(0, 1, 0)
        local rayDir = direction.Unit
        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = {char}
        local result = Workspace:Raycast(rayOrigin, rayDir * 3, raycastParams)
        return result ~= nil
    end

    local function getFreeDirection()
        local forward = (targetPos - hrp.Position).Unit
        local right = forward:Cross(Vector3.new(0, 1, 0)).Unit
        local left = -right

        if not hasObstacle(forward) then return forward end
        if not hasObstacle(right) then return right end
        if not hasObstacle(left) then return left end
        local diagRight = (forward + right).Unit
        if not hasObstacle(diagRight) then return diagRight end
        local diagLeft = (forward + left).Unit
        if not hasObstacle(diagLeft) then return diagLeft end
        return nil
    end

    while tick() - startTime < TIMEOUT do
        if not Toggles.AutoFarmGari then return false end
        char = LP.Character
        if not char then return false end
        humanoid = char:FindFirstChildOfClass("Humanoid")
        hrp = char:FindFirstChild("HumanoidRootPart")
        if not humanoid or not hrp then return false end

        local currentPos = hrp.Position
        local dist = (currentPos - targetPos).Magnitude
        if dist < DISTANCE_THRESHOLD then
            humanoid:MoveTo(hrp.Position)
            return true
        end

        if tick() - lastStuckCheck > STUCK_TIMEOUT then
            local newDist = (hrp.Position - targetPos).Magnitude
            if math.abs(newDist - lastDist) < 0.5 then
                stuckCount = stuckCount + 1
                if stuckCount >= 2 then
                    humanoid.Jump = true
                    local freeDir = getFreeDirection()
                    if freeDir then
                        local sidePos = hrp.Position + freeDir * SIDE_STEP_DISTANCE
                        humanoid:MoveTo(sidePos)
                    end
                    task.wait(0.3)
                    stuckCount = 0
                end
            else
                stuckCount = 0
            end
            lastDist = newDist
            lastStuckCheck = tick()
        end

        humanoid:MoveTo(targetPos)
        task.wait(CHECK_INTERVAL)
    end
    return false
end

local function attemptCollect(entry)
    if not entry then return false end

    if entry.prompt and entry.prompt:IsA("ProximityPrompt") then
        local prompt = entry.prompt
        local parentPart = prompt.Parent
        local targetPart = nil
        if parentPart and parentPart:IsA("BasePart") then
            targetPart = parentPart
        else
            local base = nil
            local p = prompt.Parent
            while p and p ~= Workspace do
                if p:IsA("BasePart") then base = p; break end
                p = p.Parent
            end
            targetPart = base
        end

        if targetPart then
            local ok = walkToPosition(targetPart.Position)
            if not ok then return false end
        end

        task.wait(0.2)
        local hold = tonumber(prompt.HoldDuration) or 0
        for _ = 1, 3 do
            local ok = pcall(function()
                if hold > 0 then
                    prompt:InputHoldBegin()
                    local waited = 0
                    local timeout = hold + 1.0
                    while prompt.Enabled and waited < timeout do
                        task.wait(0.1)
                        waited = waited + 0.1
                    end
                    prompt:InputHoldEnd()
                else
                    prompt:InputHoldBegin()
                    task.wait(0.08)
                    prompt:InputHoldEnd()
                end
            end)
            task.wait(0.25)
            if not prompt or not prompt.Parent or not prompt.Parent:IsDescendantOf(Workspace) then
                return true
            end
            if ok then
                if entry.target and not entry.target:IsDescendantOf(Workspace) then return true end
            end
            task.wait(0.15)
        end
        return false
    end

    if entry.remote then
        local rem = entry.remote
        local targetPart = getModelBasePart(entry.target)
        if targetPart then
            walkToPosition(targetPart.Position)
            task.wait(0.2)
        end
        local ok = false
        pcall(function()
            if rem:IsA("RemoteEvent") then
                pcall(function() rem:FireServer() end)
                pcall(function() rem:FireServer(entry.target) end)
                pcall(function() rem:FireServer(LP) end)
                pcall(function() rem:FireServer(entry.target, LP) end)
                ok = true
            elseif rem:IsA("RemoteFunction") then
                pcall(function() rem:InvokeServer() end)
                pcall(function() rem:InvokeServer(entry.target) end)
                pcall(function() rem:InvokeServer(LP) end)
                ok = true
            end
        end)
        task.wait(0.35)
        if entry.target and not entry.target:IsDescendantOf(Workspace) then return true end
        return ok
    end

    if entry.target then
        local cd = nil
        for _, d in ipairs(entry.target:GetDescendants()) do
            if d:IsA("ClickDetector") and (string.find(string.lower(d.Name or ""), "pegar") or string.find(string.lower(d.Name or ""), "lixo")) then
                cd = d
                break
            end
        end
        if cd then
            local targetPart = getModelBasePart(entry.target)
            if targetPart then
                walkToPosition(targetPart.Position)
                task.wait(0.2)
            end
            pcall(function() cd:MouseClick(LP) end)
            task.wait(0.3)
            if entry.target and not entry.target:IsDescendantOf(Workspace) then return true end
            return true
        end
    end
    return false
end

local function farmLoop()
    if not Toggles.AutoFarmGari then return end
    task.spawn(function()
        while Toggles.AutoFarmGari do
            local entries = findTrashEntries()
            if #entries == 0 then
                notify("Auto Farm", "Nenhum lixo encontrado...", 2)
                task.wait(5)
            else
                for _, entry in ipairs(entries) do
                    if not Toggles.AutoFarmGari then break end
                    if entry.target and not entry.target:IsDescendantOf(Workspace) then
                        goto continue
                    end
                    local success = false
                    for _ = 1, 3 do
                        if not Toggles.AutoFarmGari then break end
                        local ok = false
                        pcall(function() ok = attemptCollect(entry) end)
                        if ok then
                            success = true
                            break
                        end
                        task.wait(0.5)
                    end
                    if Toggles.AutoFarmGari then
                        notify("Auto Farm", "Aguardando " .. tostring(TELEPORT_DELAY_SECONDS) .. "s...", 1.4)
                        local waited = 0
                        while waited < TELEPORT_DELAY_SECONDS and Toggles.AutoFarmGari do
                            task.wait(0.5)
                            waited = waited + 0.5
                        end
                    end
                    ::continue::
                end
            end
            task.wait(0.6)
        end
    end)
end

local function toggleAutoFarm(state)
    Toggles.AutoFarmGari = state
    if state then
        notify("Auto Farm", "Auto Farm Gari ATIVADO! (caminhando + desvio)", 2)
        farmLoop()
    else
        notify("Auto Farm", "Auto Farm Gari DESATIVADO!", 2)
    end
end

-- ============================ NITRO AUTOMÁTICO ============================
local function getCurrentVehicleSeat()
    local char = LP.Character
    if not char or not char:FindFirstChild("Humanoid") then return nil end
    local humanoid = char.Humanoid
    if humanoid.SeatPart and humanoid.SeatPart:IsA("VehicleSeat") then
        return humanoid.SeatPart
    end
    return nil
end

local function getCarFromSeat(seat)
    if not seat then return nil end
    local car = seat.Parent
    while car and car ~= Workspace do
        local seats, parts = 0, 0
        for _, child in pairs(car:GetChildren()) do
            if child:IsA("VehicleSeat") or child:IsA("Seat") then seats = seats + 1 end
            if child:IsA("BasePart") then parts = parts + 1 end
        end
        if seats >= 1 and parts >= 3 then return car end
        car = car.Parent
    end
    return seat.Parent
end

local function nitroMethod1(seat)
    local car = getCarFromSeat(seat)
    if not car then return false end
    local success = pcall(function()
        local mainPart, maxSize = nil, 0
        for _, part in pairs(car:GetChildren()) do
            if part:IsA("BasePart") and part ~= seat then
                local size = part.Size.Magnitude
                if size > maxSize then
                    maxSize = size
                    mainPart = part
                end
            end
        end
        if not mainPart then mainPart = seat end
        for _, obj in pairs(mainPart:GetChildren()) do
            if obj:IsA("BodyVelocity") then obj:Destroy() end
        end
        local bv = Instance.new("BodyVelocity")
        bv.MaxForce = Vector3.new(4000, 0, 4000)
        bv.Velocity = mainPart.CFrame.LookVector * nitroForce
        bv.Parent = mainPart
        Debris:AddItem(bv, 0.5)
    end)
    return success
end

local function nitroMethod2(seat)
    local car = getCarFromSeat(seat)
    if not car then return false end
    local success = pcall(function()
        for _, part in pairs(car:GetChildren()) do
            if part:IsA("BasePart") then
                local currentVel = part.AssemblyLinearVelocity
                part.AssemblyLinearVelocity = currentVel + (part.CFrame.LookVector * nitroForce)
            end
        end
    end)
    return success
end

local function nitroMethod3(seat)
    local success = pcall(function()
        local bv = Instance.new("BodyVelocity")
        bv.MaxForce = Vector3.new(math.huge, 0, math.huge)
        bv.Velocity = seat.CFrame.LookVector * nitroForce
        bv.Parent = seat
        Debris:AddItem(bv, 0.3)
    end)
    return success
end

local function activateNitro()
    local seat = getCurrentVehicleSeat()
    if not seat then
        notify("Nitro", "Entre em um veículo primeiro!", 3)
        return
    end
    nitroMethod1(seat)
    task.wait(0.1)
    nitroMethod2(seat)
    task.wait(0.1)
    nitroMethod3(seat)
    notify("Nitro", "Nitro ativado!", 2)
end

local function toggleNitro(state)
    Toggles.NitroAuto = state
    if state then
        if nitroConnection then
            if isMobile and nitroConnection:IsA("TextButton") then
                nitroConnection:Destroy()
            else
                pcall(function() nitroConnection:Disconnect() end)
            end
        end
        if isMobile then
            if nitroButtonMobile then nitroButtonMobile:Destroy() end
            nitroButtonMobile = Instance.new("TextButton")
            nitroButtonMobile.Name = "NitroButtonMobile"
            nitroButtonMobile.Parent = LP.PlayerGui
            nitroButtonMobile.Size = UDim2.new(0, 80, 0, 80)
            nitroButtonMobile.Position = UDim2.new(1, -100, 1, -100)
            nitroButtonMobile.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
            nitroButtonMobile.Text = "NITRO"
            nitroButtonMobile.TextColor3 = Color3.fromRGB(255, 255, 255)
            nitroButtonMobile.TextSize = 16
            nitroButtonMobile.ZIndex = 1000
            Instance.new("UICorner", nitroButtonMobile).CornerRadius = UDim.new(0, 20)
            nitroButtonMobile.MouseButton1Click:Connect(activateNitro)
            nitroConnection = nitroButtonMobile
        else
            nitroConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
                if gameProcessed then return end
                if input.KeyCode == Enum.KeyCode.N then activateNitro() end
            end)
        end
        notify("Nitro", "Nitro automático ativado! (Tecla N)", 3)
    else
        if nitroConnection then
            if isMobile and nitroConnection:IsA("TextButton") then
                nitroConnection:Destroy()
            else
                pcall(function() nitroConnection:Disconnect() end)
            end
            nitroConnection = nil
        end
        if nitroButtonMobile then
            nitroButtonMobile:Destroy()
            nitroButtonMobile = nil
        end
        notify("Nitro", "Nitro automático desativado!", 3)
    end
end

-- ============================ WHITELIST / AMIGOS ============================
local function UpdatePlayerListButtons(section)
    -- Limpa botões antigos (exceto os dois primeiros: atualizar e remover todos)
    local children = section:GetChildren()
    for i = 3, #children do
        if children[i]:IsA("GuiObject") then
            children[i]:Destroy()
        end
    end

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP then
            local name = p.Name
            local isFriend = Whitelist[name] == true
            section:CreateButton({
                Name = (isFriend and "✅ ALIADO: " or "❌ INIMIGO: ") .. name,
                Callback = function()
                    if Whitelist[name] then
                        Whitelist[name] = nil
                        notify("Whitelist", name .. " removido", 1.5)
                    else
                        Whitelist[name] = true
                        notify("Whitelist", name .. " adicionado", 1.5)
                    end
                    UpdatePlayerListButtons(section)
                end
            })
        end
    end
end

-- ============================ CONFIGURAÇÕES (SALVAR/CARREGAR) ============================
local CONFIG_FILE = "OnyxHub_Rayfield_Config.json"
local function hasFS() return type(writefile) == "function" and type(readfile) == "function" and type(isfile) == "function" end

local function snapshotConfig()
    local data = {
        Toggles = {},
        AimbotConfig = {
            FOVRadius = AimbotConfig.FOVRadius,
            HitPart = AimbotConfig.HitPart,
            SpinSpeed = AimbotConfig.SpinSpeed,
            HeadSize = AimbotConfig.HeadSize,
            HeadTransparency = AimbotConfig.HeadTransparency,
            SmoothSpeed = AimbotConfig.SmoothSpeed,
        },
        HitboxSize = hitboxSize,
        HitboxTransparency = hitboxTransparency,
        LegitTam = legitTam,
        FOVColor = {FOVColor.R, FOVColor.G, FOVColor.B},
        Whitelist = {},
        carSensitivity = carSensitivity,
        nitroForce = nitroForce,
        TELEPORT_DELAY_SECONDS = TELEPORT_DELAY_SECONDS,
    }
    for k, v in pairs(Toggles) do data.Toggles[k] = v end
    for name, _ in pairs(Whitelist) do table.insert(data.Whitelist, name) end
    return data
end

local function applyConfig(data)
    if type(data) ~= "table" then return end
    if type(data.Toggles) == "table" then
        for k, v in pairs(data.Toggles) do Toggles[k] = v end
    end
    if type(data.AimbotConfig) == "table" then
        for k, v in pairs(data.AimbotConfig) do AimbotConfig[k] = v end
    end
    if data.HitboxSize then hitboxSize = data.HitboxSize end
    if data.HitboxTransparency then hitboxTransparency = data.HitboxTransparency end
    if data.LegitTam then legitTam = data.LegitTam end
    if type(data.FOVColor) == "table" then
        FOVColor = Color3.new(data.FOVColor[1], data.FOVColor[2], data.FOVColor[3])
    end
    if type(data.Whitelist) == "table" then
        Whitelist = {}
        for _, n in ipairs(data.Whitelist) do Whitelist[n] = true end
    end
    if data.carSensitivity then carSensitivity = data.carSensitivity end
    if data.nitroForce then nitroForce = data.nitroForce end
    if data.TELEPORT_DELAY_SECONDS then TELEPORT_DELAY_SECONDS = data.TELEPORT_DELAY_SECONDS end
end

-- ============================ CRIAÇÃO DA INTERFACE RAYFIELD ============================
local Window = Rayfield:CreateWindow({
    Name = "ONYX HUB v2 - Rayfield",
    LoadingTitle = "Carregando...",
    LoadingSubtitle = "by CLS",
    ConfigurationSaving = {Enabled = false},
    KeySystem = false,
})

-- Aba: COMBATE
local CombatTab = Window:CreateTab("Combate", 4483362458)

CombatTab:CreateSection("Aimbot")
CombatTab:CreateToggle({
    Name = "Aimbot (PC - botão direito)",
    CurrentValue = false,
    Callback = function(v)
        Toggles.Aimbot = v
        if v then
            aimbotThread = task.spawn(AimbotLoopPC)
            StartHeadSizeLoop()
        else
            if aimbotThread then pcall(task.cancel, aimbotThread); aimbotThread = nil end
        end
    end
})
if isMobile then
    CombatTab:CreateToggle({
        Name = "Aimbot Mobile (Hold to Aim)",
        CurrentValue = false,
        Callback = function(v)
            Toggles.AimbotMobile = v
            if v then SetupMobileAutoAim(); StartHeadSizeLoop() end
        end
    })
    CombatTab:CreateToggle({
        Name = "Aimbot Mobile (Look Direction)",
        CurrentValue = false,
        Callback = function(v)
            Toggles.AimbotMobileLook = v
            if v then SetupMobileLookAim(); StartHeadSizeLoop() else if mobileLookConn then mobileLookConn:Disconnect(); mobileLookConn = nil end end
        end
    })
end
CombatTab:CreateToggle({
    Name = "Aimbot Hotkey (segurar tecla)",
    CurrentValue = false,
    Callback = function(v)
        Toggles.AimbotHotkey = v
        if v then SetupHotkeyAimbot(); StartHeadSizeLoop() else if smoothAimbotConn then smoothAimbotConn:Disconnect(); smoothAimbotConn = nil end; hotkeyAimbotActive = false end
    end
})
CombatTab:CreateToggle({
    Name = "Smooth Aimbot",
    CurrentValue = false,
    Callback = function(v) Toggles.AimbotSmooth = v end
})
CombatTab:CreateSlider({
    Name = "Suavização",
    Range = {0.01, 0.5},
    Increment = 0.01,
    CurrentValue = 0.15,
    Callback = function(v) AimbotConfig.SmoothSpeed = v end
})
CombatTab:CreateSlider({
    Name = "Raio do FOV",
    Range = {50, 400},
    Increment = 10,
    CurrentValue = 150,
    Callback = function(v) AimbotConfig.FOVRadius = v end
})
CombatTab:CreateToggle({
    Name = "Ignorar Time",
    CurrentValue = false,
    Callback = function(v) Toggles.CheckTeam = v end
})
CombatTab:CreateToggle({
    Name = "Checar Parede (Wall Check)",
    CurrentValue = false,
    Callback = function(v) Toggles.CheckWall = v end
})
CombatTab:CreateToggle({
    Name = "Spinbot",
    CurrentValue = false,
    Callback = ToggleSpinbot
})
CombatTab:CreateDropdown({
    Name = "Hitbox do Aimbot",
    Options = {"Head", "UpperTorso", "HumanoidRootPart"},
    CurrentOption = "Head",
    Callback = function(v) AimbotConfig.HitPart = v end
})

-- Aba: HITBOX
local HitboxTab = Window:CreateTab("Hitbox", 4483362459)
HitboxTab:CreateSection("Expansão de Hitbox")
HitboxTab:CreateToggle({
    Name = "Hitbox Expandida (vermelha/neon)",
    CurrentValue = false,
    Callback = function(v)
        Toggles.HitboxExpanded = v
        if v then startHitboxLoop() else stopAndRestoreHitbox() end
    end
})
HitboxTab:CreateSlider({
    Name = "Tamanho da Hitbox",
    Range = {5, 50},
    Increment = 1,
    CurrentValue = 15,
    Callback = function(v) hitboxSize = v end
})
HitboxTab:CreateSlider({
    Name = "Transparência (%)",
    Range = {0, 100},
    Increment = 5,
    CurrentValue = 70,
    Callback = function(v) hitboxTransparency = v / 100 end
})
HitboxTab:CreateSection("Hitbox LEGIT")
HitboxTab:CreateToggle({
    Name = "Hitbox LEGIT (invisível maior)",
    CurrentValue = false,
    Callback = function(v)
        Toggles.LegitHitbox = v
        if v then startHitboxLoop() else stopAndRestoreHitbox() end
    end
})
HitboxTab:CreateSlider({
    Name = "Tamanho LEGIT",
    Range = {4, 15},
    Increment = 1,
    CurrentValue = 6,
    Callback = function(v) legitTam = v end
})

-- Aba: AUTO FARM
local FarmTab = Window:CreateTab("Auto Farm", 4483362457)
FarmTab:CreateSection("Coleta de Lixo (Gari)")
FarmTab:CreateToggle({
    Name = "Auto Farm Gari (caminhando + desvio)",
    CurrentValue = false,
    Callback = toggleAutoFarm
})
FarmTab:CreateSlider({
    Name = "Delay entre coletas (s)",
    Range = {2, 30},
    Increment = 1,
    CurrentValue = 10,
    Callback = function(v) TELEPORT_DELAY_SECONDS = v end
})
FarmTab:CreateParagraph({
    Title = "Informação",
    Content = "O personagem andará até o lixo e desviará de obstáculos automaticamente."
})

-- Aba: VEÍCULOS
local VehicleTab = Window:CreateTab("Veículos", 4483362461)
VehicleTab:CreateSection("Controle de Veículos")
VehicleTab:CreateToggle({
    Name = "Speed Car (W/S/Q)",
    CurrentValue = false,
    Callback = function(v) Toggles.SpeedCar = v end
})
VehicleTab:CreateToggle({
    Name = "Fly Car (Tecla F)",
    CurrentValue = false,
    Callback = function(v) Toggles.VFly = v end
})
VehicleTab:CreateSlider({
    Name = "Sensibilidade do Carro",
    Range = {0.5, 10},
    Increment = 0.5,
    CurrentValue = 2,
    Callback = function(v) carSensitivity = v end
})
VehicleTab:CreateToggle({
    Name = "Nitro Automático (Tecla N / Botão Mobile)",
    CurrentValue = false,
    Callback = toggleNitro
})
VehicleTab:CreateSlider({
    Name = "Força do Nitro",
    Range = {20, 150},
    Increment = 5,
    CurrentValue = 50,
    Callback = function(v) nitroForce = v end
})

-- Aba: MOVIMENTO / OUTROS
local MovementTab = Window:CreateTab("Movimento", 4483362460)
MovementTab:CreateSection("Geral")
MovementTab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Callback = ToggleNoclip
})
MovementTab:CreateToggle({
    Name = "Anti-Sit",
    CurrentValue = true,
    Callback = function(v) ToggleAntiSit(v) end
})
MovementTab:CreateToggle({
    Name = "Anti-AFK",
    CurrentValue = true,
    Callback = ToggleAntiAfk
})
MovementTab:CreateButton({
    Name = "Recarregar Personagem",
    Callback = function()
        local ch = GetChar()
        local h = ch and ch:FindFirstChildOfClass("Humanoid")
        if h then h.Health = 0 end
        notify("Personagem", "Recarregado!", 1.5)
    end
})
MovementTab:CreateButton({
    Name = "Server Hop",
    Callback = function()
        pcall(function()
            local res = game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100")
            local data = HttpService:JSONDecode(res)
            local servers = {}
            if data and data.data then
                for _, sv in ipairs(data.data) do
                    if sv.playing < sv.maxPlayers and sv.id ~= game.JobId then
                        table.insert(servers, sv.id)
                    end
                end
            end
            if #servers > 0 then
                TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], LP)
            end
        end)
    end
})
MovementTab:CreateButton({
    Name = "Rejoin",
    Callback = function() TeleportService:Teleport(game.PlaceId, LP) end
})

-- Aba: AMIGOS (Whitelist)
local FriendsTab = Window:CreateTab("Amigos", 4483362458)
local friendsSection = FriendsTab:CreateSection("Gerenciar Whitelist")
FriendsTab:CreateToggle({
    Name = "Usar Whitelist (não mira amigos)",
    CurrentValue = false,
    Callback = function(v) Toggles.UseWhitelist = v end
})
FriendsTab:CreateButton({
    Name = "🔄 ATUALIZAR LISTA",
    Callback = function()
        UpdatePlayerListButtons(friendsSection)
        notify("Amigos", "Lista atualizada!", 1.5)
    end
})
FriendsTab:CreateButton({
    Name = "❌ REMOVER TODOS OS ALIADOS",
    Callback = function()
        for n in pairs(Whitelist) do
            Whitelist[n] = nil
        end
        UpdatePlayerListButtons(friendsSection)
        notify("Amigos", "Todos removidos!", 1.5)
    end
})
-- Inicializar a lista
task.delay(0.5, function() UpdatePlayerListButtons(friendsSection) end)

-- Aba: CONFIGURAÇÕES
local ConfigTab = Window:CreateTab("Config", 4483362459)
ConfigTab:CreateSection("Salvar / Carregar")
ConfigTab:CreateButton({
    Name = "💾 Salvar Configurações",
    Callback = function()
        if not hasFS() then
            notify("Config", "Executor sem suporte a arquivos", 3)
            return
        end
        local ok, json = pcall(function() return HttpService:JSONEncode(snapshotConfig()) end)
        if not ok then notify("Config", "Erro ao codificar", 2) return end
        local okw = pcall(function() writefile(CONFIG_FILE, json) end)
        if okw then
            notify("Config", "Salvo em " .. CONFIG_FILE, 2)
        else
            notify("Config", "Falha ao salvar", 2)
        end
    end
})
ConfigTab:CreateButton({
    Name = "📂 Carregar Configurações",
    Callback = function()
        if not hasFS() then
            notify("Config", "Executor sem suporte a arquivos", 3)
            return
        end
        if not isfile(CONFIG_FILE) then
            notify("Config", "Nenhuma configuração salva", 2)
            return
        end
        local ok, raw = pcall(function() return readfile(CONFIG_FILE) end)
        if not ok or not raw then notify("Config", "Falha ao ler", 2) return end
        local okd, data = pcall(function() return HttpService:JSONDecode(raw) end)
        if not okd then notify("Config", "Arquivo inválido", 2) return end
        applyConfig(data)
        notify("Config", "Configuração carregada!", 2)
    end
})
ConfigTab:CreateButton({
    Name = "🗑️ Apagar Configuração",
    Callback = function()
        if not hasFS() then
            notify("Config", "Executor sem suporte", 2)
            return
        end
        if type(delfile) == "function" and isfile(CONFIG_FILE) then
            pcall(function() delfile(CONFIG_FILE) end)
            notify("Config", "Arquivo apagado", 2)
        else
            notify("Config", "Nenhum arquivo para apagar", 2)
        end
    end
})
ConfigTab:CreateSection("Tema")
ConfigTab:CreateDropdown({
    Name = "Tema do Rayfield",
    Options = {"Default", "Dark", "Amber", "Blue", "Pink"},
    CurrentOption = "Dark",
    Callback = function(v)
        Rayfield:SetTheme(v)
    end
})
ConfigTab:CreateButton({
    Name = "🔻 Minimizar Menu",
    Callback = function() Rayfield:Toggle() end
})
ConfigTab:CreateButton({
    Name = "❌ Unload (remover tudo)",
    Callback = function()
        if fovCircle then pcall(function() fovCircle:Remove() end) end
        stopAndRestoreHitbox()
        if nitroConnection then
            if isMobile and nitroConnection:IsA("TextButton") then
                nitroConnection:Destroy()
            else
                pcall(function() nitroConnection:Disconnect() end)
            end
        end
        Rayfield:Destroy()
        notify("Onyx Hub", "GUI descarregada", 2)
    end
})

-- ============================ INICIALIZAÇÃO DE LOOP E EVENTOS ============================
task.spawn(CreateFOVCircle)
ToggleAntiSit(true, true)

-- Loop principal para veículos e FOV (sem ESP)
RunService.RenderStepped:Connect(function()
    pcall(function()
        -- Atualizar círculo do FOV
        if fovCircle then
            if isMobile and Camera then
                fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
            else
                fovCircle.Position = UserInputService:GetMouseLocation()
            end
            fovCircle.Visible = Toggles.Aimbot or Toggles.AimbotMobile or Toggles.AimbotHotkey or Toggles.AimbotMobileLook
            fovCircle.Radius = AimbotConfig.FOVRadius
            fovCircle.Color = FOVColor
        end

        -- Veículos
        local ch = LP.Character
        if ch and ch:FindFirstChild("Humanoid") and ch.Humanoid.SeatPart then
            local seatPart = ch.Humanoid.SeatPart
            if Toggles.VFly and UserInputService:IsKeyDown(Enum.KeyCode.F) then
                local direction = Vector3.new(0, 0, 0)
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then direction = direction + Camera.CFrame.LookVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then direction = direction - Camera.CFrame.LookVector end
                seatPart.AssemblyLinearVelocity = direction * 400
            elseif Toggles.SpeedCar then
                if UserInputService:IsKeyDown(Enum.KeyCode.Q) then
                    carVelocity = 5000
                elseif seatPart.Throttle ~= 0 then
                    carVelocity = math.clamp(carVelocity + (seatPart.Throttle * carSensitivity), -5000, 5000)
                else
                    carVelocity = carVelocity * 0.98
                end
                seatPart.AssemblyLinearVelocity = (seatPart.CFrame.LookVector * carVelocity) + Vector3.new(0, -20, 0)
            end
        end
    end)
end)

notify("Onyx Hub", "Carregado com sucesso! (Rayfield + Onyx Features)", 4)
