local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

task.spawn(function()
    while task.wait(0.1) do
        if not LocalPlayer or not game:IsLoaded() then
            task.wait(1)
        end
        local _anti = getfenv(0)
        _anti.script = nil
        _anti.owner = nil
    end
end)

local _RUNTIME_ENV = {
    cfg = {
        aim = false,
        wall = false,
        smooth = 0.1,
        fov = 150,
        speed_car = false,
        vfly = false,
        sens = 2
    },
    aliados = {},
    botoes = {},
    objetos_esp = {box = {}, tracer = {}, name = {}},
    speed_vel = 0
}

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "POMBA MENU 1.02 - - GITMENU",
    LoadingTitle = "Carregando Menu...",
    LoadingSubtitle = "by Gyro",
    ConfigurationSaving = {Enabled = false},
    KeySystem = false,
})

local _CORE_ENGINE = {}

function _CORE_ENGINE:FormatMoney(v)
    local s = tostring(v)
    local k
    while true do
        s, k = string.gsub(s, "^(-?%d+)(%d%d%d)", '%1.%2')
        if k == 0 then break end
    end
    return s
end

function _CORE_ENGINE:CheckLineOfSight(target)
    if not target or not target.Parent then return false end
    local castParams = RaycastParams.new()
    castParams.FilterType = Enum.RaycastFilterType.Exclude
    castParams.FilterDescendantsInstances = {LocalPlayer.Character, target.Parent}
    local hitResult = workspace:Raycast(Camera.CFrame.Position, target.Position - Camera.CFrame.Position, castParams)
    return hitResult == nil
end

local CombatTab = Window:CreateTab("Combate", 4483362458)
local AliadoTab = Window:CreateTab("Lista de Aliados", 4483362457)
local VisualTab = Window:CreateTab("Visual", 4483362459)
local VeiculoTab = Window:CreateTab("Veículos", 4483362461)
local GuiTab = Window:CreateTab("Abrir GUI", 4483362460)

local function BuildInterface()
    AliadoTab:CreateButton({
        Name = "🔄 ATUALIZAR LISTA (Novos Players)",
        Callback = function()
            Rayfield:Notify({Title="Gitmenu System", Content="Sincronizado!", Duration=2})
        end
    })

    AliadoTab:CreateButton({
        Name = "❌ REMOVER TODOS OS ALIADOS",
        Callback = function()
            for n, _ in pairs(_RUNTIME_ENV.aliados) do
                _RUNTIME_ENV.aliados[n] = nil
                if _RUNTIME_ENV.botoes[n] then _RUNTIME_ENV.botoes[n]:Set("Inimigo: "..n) end
            end
        end
    })

    AliadoTab:CreateSection("Network Players")
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then
            local n = p.Name
            local b = AliadoTab:CreateButton({
                Name = "Inimigo: "..n,
                Callback = function()
                    _RUNTIME_ENV.aliados[n] = not _RUNTIME_ENV.aliados[n]
                    _RUNTIME_ENV.botoes[n]:Set(_RUNTIME_ENV.aliados[n] and "✅ ALIADO: "..n or "Inimigo: "..n)
                end
            })
            _RUNTIME_ENV.botoes[n] = b
        end
    end
end

BuildInterface()

CombatTab:CreateSection("Aimbot Master")
CombatTab:CreateToggle({Name = "Aimbot Ativo", CurrentValue = false, Callback = function(v) _RUNTIME_ENV.cfg.aim = v end})
CombatTab:CreateToggle({Name = "Wall Check", CurrentValue = false, Callback = function(v) _RUNTIME_ENV.cfg.wall = v end})
CombatTab:CreateSlider({Name = "Suavização", Range = {0.01, 1}, Increment = 0.01, CurrentValue = 0.1, Callback = function(v) _RUNTIME_ENV.cfg.smooth = v end})
CombatTab:CreateSlider({Name = "Raio FOV", Range = {10, 800}, Increment = 10, CurrentValue = 150, Callback = function(v) _RUNTIME_ENV.cfg.fov = v end})

VisualTab:CreateSection("Visual FX")
VisualTab:CreateToggle({Name = "ESP Players", CurrentValue = false, Callback = function(v) _RUNTIME_ENV.cfg.esp = v end})
VisualTab:CreateToggle({Name = "ESP Box", CurrentValue = false, Callback = function(v) _RUNTIME_ENV.cfg.esp_box = v end})
VisualTab:CreateToggle({Name = "ESP Tracer", CurrentValue = false, Callback = function(v) _RUNTIME_ENV.cfg.esp_tracer = v end})
VisualTab:CreateToggle({Name = "ESP Name", CurrentValue = false, Callback = function(v) _RUNTIME_ENV.cfg.esp_name = v end})

local _FOV_RENDERER = Drawing.new("Circle")
_FOV_RENDERER.Thickness = 2
_FOV_RENDERER.Filled = false

VeiculoTab:CreateSection("Vehicle Tuning")
VeiculoTab:CreateToggle({Name = "Speed Car (W/S/Q)", CurrentValue = false, Callback = function(v) _RUNTIME_ENV.cfg.speed_car = v end})
VeiculoTab:CreateToggle({Name = "Fly Car (Tecla F)", CurrentValue = false, Callback = function(v) _RUNTIME_ENV.cfg.vfly = v end})

local function OpenRemoteGui(name, frame)
    local pGui = LocalPlayer.PlayerGui
    local ui = pGui:FindFirstChild(name, true)
    if ui then
        local f = ui:FindFirstChild(frame or "Loja") or ui:FindFirstChild("Prefeitura") or ui:FindFirstChild("Rotas")
        if f then f.Visible = true end
    end
end

GuiTab:CreateButton({Name = "Farmacia", Callback = function() OpenRemoteGui("Farmacia") end})
GuiTab:CreateButton({Name = "LojaFac", Callback = function() OpenRemoteGui("LojaFac") end})
GuiTab:CreateButton({Name = "Mercado Negro", Callback = function() OpenRemoteGui("MercadoNegro") end})
GuiTab:CreateButton({Name = "Peixaria", Callback = function() OpenRemoteGui("Peixaria") end})
GuiTab:CreateButton({Name = "Prefeitura", Callback = function() OpenRemoteGui("PrefeituraGui") end})
GuiTab:CreateButton({Name = "Rotas", Callback = function() OpenRemoteGui("RotasFac") end})
GuiTab:CreateButton({Name = "Vender Joias", Callback = function() OpenRemoteGui("SellMarketUI") end})

local function CreateDrawings(p)
    if p == LocalPlayer then return end
    _RUNTIME_ENV.objetos_esp.box[p] = Drawing.new("Square")
    _RUNTIME_ENV.objetos_esp.tracer[p] = Drawing.new("Line")
    _RUNTIME_ENV.objetos_esp.name[p] = Drawing.new("Text")
    local text = _RUNTIME_ENV.objetos_esp.name[p]
    text.Size, text.Center, text.Outline = 14, true, true
end

local function ClearDrawings(p)
    local o = _RUNTIME_ENV.objetos_esp
    if o.box[p] then
        o.box[p]:Remove() o.tracer[p]:Remove() o.name[p]:Remove()
        o.box[p], o.tracer[p], o.name[p] = nil, nil, nil
    end
end

for _, p in pairs(Players:GetPlayers()) do CreateDrawings(p) end
Players.PlayerAdded:Connect(CreateDrawings)
Players.PlayerRemoving:Connect(ClearDrawings)

local function MasterLoop()
    _FOV_RENDERER.Visible = _RUNTIME_ENV.cfg.aim
    _FOV_RENDERER.Radius = _RUNTIME_ENV.cfg.fov
    _FOV_RENDERER.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    _FOV_RENDERER.Color = Color3.fromHSV(tick() % 5 / 5, 1, 1)

    if _RUNTIME_ENV.cfg.aim then
        local t, d = nil, _RUNTIME_ENV.cfg.fov
        for _, v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and not _RUNTIME_ENV.aliados[v.Name] and v.Character and v.Character:FindFirstChild("Head") and v.Character.Humanoid.Health > 0 then
                local sPos, vis = Camera:WorldToViewportPoint(v.Character.Head.Position)
                if vis then
                    local m = (Vector2.new(sPos.X, sPos.Y) - _FOV_RENDERER.Position).Magnitude
                    if m < d then
                        if _RUNTIME_ENV.cfg.wall then
                            if _CORE_ENGINE:CheckLineOfSight(v.Character.Head) then t = v.Character.Head d = m end
                        else t = v.Character.Head d = m end
                    end
                end
            end
        end
        if t then Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, t.Position), _RUNTIME_ENV.cfg.smooth) end
    end

    for _, p in pairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        local box, tracer, label = _RUNTIME_ENV.objetos_esp.box[p], _RUNTIME_ENV.objetos_esp.tracer[p], _RUNTIME_ENV.objetos_esp.name[p]
        if box and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local root = p.Character.HumanoidRootPart
            local hum = p.Character.Humanoid
            local pos, vis = Camera:WorldToViewportPoint(root.Position)

            if vis and _RUNTIME_ENV.cfg.esp then
                local ali = _RUNTIME_ENV.aliados[p.Name]
                local cor = ali and Color3.fromRGB(0, 162, 255) or (hum.Health > 0 and Color3.new(1,0,0) or Color3.new(0,0,0))
                
                if _RUNTIME_ENV.cfg.esp_box then
                    box.Visible, box.Color, box.Size = true, cor, Vector2.new(2000/pos.Z, 3000/pos.Z)
                    box.Position = Vector2.new(pos.X - box.Size.X/2, pos.Y - box.Size.Y/2)
                else
                    box.Visible = false
                end
                
                if _RUNTIME_ENV.cfg.esp_tracer then
                    tracer.Visible, tracer.Color = true, cor
                    tracer.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                    tracer.To = Vector2.new(pos.X, pos.Y + box.Size.Y/2)
                else
                    tracer.Visible = false
                end
                
                if _RUNTIME_ENV.cfg.esp_name then
                    local mStr = "0" pcall(function() mStr = _CORE_ENGINE:FormatMoney(p.leaderstats.Dinheiro.Value) end)
                    label.Text = (ali and "[ALIADO] " or "")..p.DisplayName.."\n$"..mStr
                    label.Position = Vector2.new(pos.X, pos.Y - box.Size.Y/2 - 35)
                    label.Color, label.Visible = cor, true
                else
                    label.Visible = false
                end
            else
                box.Visible, tracer.Visible, label.Visible = false, false, false
            end
        end
    end

    local ch = LocalPlayer.Character
    if ch and ch:FindFirstChild("Humanoid") and ch.Humanoid.SeatPart then
        local st = ch.Humanoid.SeatPart
        if _RUNTIME_ENV.cfg.vfly and UIS:IsKeyDown(Enum.KeyCode.F) then
            local v = Vector3.new(0,0,0)
            if UIS:IsKeyDown(Enum.KeyCode.W) then v = v + Camera.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.S) then v = v - Camera.CFrame.LookVector end
            st.AssemblyLinearVelocity = v * 400
        elseif _RUNTIME_ENV.cfg.speed_car then
            if UIS:IsKeyDown(Enum.KeyCode.Q) then _RUNTIME_ENV.speed_vel = 5
            elseif st.Throttle ~= 0 then _RUNTIME_ENV.speed_vel = math.clamp(_RUNTIME_ENV.speed_vel + (st.Throttle * _RUNTIME_ENV.cfg.sens), -5000, 5000)
            else _RUNTIME_ENV.speed_vel = _RUNTIME_ENV.speed_vel * 0.98 end
            st.AssemblyLinearVelocity = (st.CFrame.LookVector * _RUNTIME_ENV.speed_vel) + Vector3.new(0, -20, 0)
        end
    end
end

RunService.RenderStepped:Connect(function()
    pcall(MasterLoop)
end)
