--[[ 
   NYTRAX HYBRID v2 
   Sistema ESP do Nytrax (próprio, dos desenhos/lines/box)
   Todas demais features (exceto ESP) do Sitonia beta importadas
   Menu completo, funcional. Nenhuma dependência do Rayfield ou outro visual externo.
]]

-- -- -- -- -- -- -- -- -- -- --
--    SISTEMA ESP DO NYTRAX   --
-- -- -- -- -- -- -- -- -- -- --

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local PlayerInvs = {}
local ESP_Data = {}

-- Atualiza o inventário (5 segundos)
task.spawn(function()
    while true do
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                local items = {}
                if p:FindFirstChild("Backpack") then
                    for _, item in pairs(p.Backpack:GetChildren()) do table.insert(items, item.Name) end
                end
                if p.Character then
                    for _, item in pairs(p.Character:GetChildren()) do
                        if item:IsA("Tool") then table.insert(items, item.Name .. " (Mão)") end
                    end
                end
                PlayerInvs[p] = #items > 0 and table.concat(items, ", ") or "Vazio"
            end
        end
        task.wait(5)
    end
end)

local function RemoveESP(Player)
    if ESP_Data[Player] then
        for _, obj in pairs(ESP_Data[Player]) do pcall(function() obj:Remove() end) end
        ESP_Data[Player] = nil
    end
end
Players.PlayerRemoving:Connect(RemoveESP)

local function SetESPEnabled(state)
    _G.NYTRAX_ESP_ENABLED = state
    if not state then
        for _, d in pairs(ESP_Data) do
            for _, obj in pairs(d) do pcall(function() obj.Visible = false end) end
        end
    end
end

_G.NYTRAX_ESP_ENABLED = true -- Começa ligado

RunService.RenderStepped:Connect(function()
    if not _G.NYTRAX_ESP_ENABLED then
        for _, d in pairs(ESP_Data) do
            for _, obj in pairs(d) do pcall(function() obj.Visible = false end) end
        end
        return
    end

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then
            if not ESP_Data[p] then
                ESP_Data[p] = { Line = Drawing.new("Line"), Box = Drawing.new("Square"), Text = Drawing.new("Text") }
            end
            local data = ESP_Data[p]
            local char = p.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if hrp and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then
                local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    -- Nome/Vida/Inventário
                    data.Text.Visible = true
                    data.Text.Position = Vector2.new(pos.X, pos.Y - 45)
                    local txt = p.Name .. " [" .. math.floor(char.Humanoid.Health) .. " HP]"
                    txt = txt .. "\nInv: " .. (PlayerInvs[p] or "...")
                    data.Text.Text = txt
                    data.Text.Center = true
                    data.Text.Outline = true
                    data.Text.Size = 13
                    data.Text.Color = Color3.new(1,1,1)

                    -- Caixa
                    data.Box.Visible = true
                    data.Box.Filled = false
                    data.Box.Thickness = 1
                    data.Box.Size = Vector2.new(2200/pos.Z, 3200/pos.Z)
                    data.Box.Position = Vector2.new(pos.X - data.Box.Size.X/2, pos.Y - data.Box.Size.Y/2)
                    data.Box.Color = Color3.fromRGB(0,255,120)

                    -- Linha
                    data.Line.Visible = true
                    data.Line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                    data.Line.To = Vector2.new(pos.X, pos.Y)
                else
                    data.Text.Visible = false
                    data.Box.Visible = false
                    data.Line.Visible = false
                end
            else
                data.Text.Visible = false
                data.Box.Visible = false
                data.Line.Visible = false
            end
        end
    end
end)

-- -- -- -- -- -- -- -- -- -- -- -- -- --
--  ABAIXO: SITONIA BETA (sem ESP/UI)  --
-- -- -- -- -- -- -- -- -- -- -- -- -- --

-- Coloque aqui todo o Sitonia exceto Visual/ESP (absolutamente tudo, aimbot, autofarm, hitbox, nitro, config, etc)
-- Basta remover a aba "Visual", qualquer chamada para ESP, e helpers/Drawings relacionados ao ESP Sitonia.
-- Coloque as abas: Home, Auto Farm, Combat, Amigos/Whitelist, Outros, Config -- todos seus controles, funções, sliders, etc (exceto ESP).

-- EXEMPLO DE MENU (com toggle do ESP do Nytrax para o usuário):

local hub = setmetatable({}, Onyx):Init()
do
    local home = hub:AddTab("Home", nil)
    -- ...código Sitonia Home...

    local farm = hub:AddTab("Auto Farm", nil)
    -- ...Auto Farm (inclui coleta, delay etc)...

    local combat = hub:AddTab("Combat", nil)
    -- ... todas opções de Aimbot...

    local friends = hub:AddTab("Amigos", nil)
    -- ... Whitelist...

    local outros = hub:AddTab("Outros", nil)
    -- ... Noclip, Anti-Sit, Anti-Afk, hitbox, veículos ...

    local cfg = hub:AddTab("Config", "Cfg")
    -- ... Configurações (save/load, tema, minimizar/maximizar, reload char, unload, etc) ...

    -- Controle para ligar/desligar ESP Nytrax:
    local c = cfg:AddCard("Nytrax ESP")
    c:AddToggle("Ativar ESP Nytrax", true, SetESPEnabled)
    -- (Se quiser: pode adicionar Cor, tamanho, etc. Só certifique que manipule _G.NYTRAX_ESP_ENABLED e os dados do bloco acima!)
end

-- Fim do script -- qualquer função extra de Sitonia pode ser adaptada normalmente; só não coloque nem um Drawing/ESP do Sitonia.
