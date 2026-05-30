--[[
    ONYX HUB - Premium Roblox GUI Library v2
    Tema: Branco & Azul Neon
    - Tamanho maior, responsivo (PC / Tablet / Mobile)
    - Sidebar com toggles (liga/desliga)
    - Sistema de minimizar / maximizar (botão flutuante)
    - Apenas X fecha o menu (não destrói, só esconde)
    - Sistema de notificações (canto inferior direito)
    - Pills do topo corrigidas (label e valor não se sobrepõem)
    - Anti-kick (mesma lógica)
    - Auto Farm Gari (caminhando até o lixo + detecção de obstáculos)
    - Nitro Automático para veículos (tecla N ou botão mobile)
--------------------------------------------------------------------]]

local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local RunService       = game:GetService("RunService")
local CoreGui          = game:GetService("CoreGui")
local Debris           = game:GetService("Debris")

local LP = Players.LocalPlayer

---------------------------------------------------------------- THEME
local THEME = {
    Background = Color3.fromRGB(18, 10, 18),
    Surface    = Color3.fromRGB(28, 16, 28),
    SurfaceAlt = Color3.fromRGB(38, 22, 38),
    Sidebar    = Color3.fromRGB(22, 12, 22),
    Stroke     = Color3.fromRGB(80, 30, 70),
    Text       = Color3.fromRGB(255, 255, 255),
    SubText    = Color3.fromRGB(220, 190, 210),
    Accent     = Color3.fromRGB(255, 20, 147),
    AccentSoft = Color3.fromRGB(200, 15, 115),
    Success    = Color3.fromRGB(90, 220, 150),
    Danger     = Color3.fromRGB(255, 95, 95),
}

---------------------------------------------------------------- THEMES (presets)
local THEMES = {
    ["CLS Pink"] = {
        Background=Color3.fromRGB(18,10,18), Surface=Color3.fromRGB(28,16,28),
        SurfaceAlt=Color3.fromRGB(38,22,38), Sidebar=Color3.fromRGB(22,12,22),
        Stroke=Color3.fromRGB(80,30,70), Text=Color3.fromRGB(255,255,255),
        SubText=Color3.fromRGB(220,190,210), Accent=Color3.fromRGB(255,20,147),
        AccentSoft=Color3.fromRGB(200,15,115),
    },
    ["Midnight"] = {
        Background=Color3.fromRGB(10,12,20), Surface=Color3.fromRGB(16,20,32),
        SurfaceAlt=Color3.fromRGB(22,28,44), Sidebar=Color3.fromRGB(12,14,24),
        Stroke=Color3.fromRGB(40,50,80), Text=Color3.fromRGB(235,240,255),
        SubText=Color3.fromRGB(140,150,180), Accent=Color3.fromRGB(120,90,255),
        AccentSoft=Color3.fromRGB(80,60,200),
    },
    ["Lurs Style"] = {
        Background=Color3.fromRGB(8,8,12), Surface=Color3.fromRGB(14,14,20),
        SurfaceAlt=Color3.fromRGB(22,22,30), Sidebar=Color3.fromRGB(10,10,16),
        Stroke=Color3.fromRGB(60,30,60), Text=Color3.fromRGB(245,235,245),
        SubText=Color3.fromRGB(160,140,170), Accent=Color3.fromRGB(220,40,200),
        AccentSoft=Color3.fromRGB(170,30,160),
    },
    ["Onyx Blue"] = {
        Background=Color3.fromRGB(14,16,22), Surface=Color3.fromRGB(20,23,32),
        SurfaceAlt=Color3.fromRGB(26,30,42), Sidebar=Color3.fromRGB(16,19,27),
        Stroke=Color3.fromRGB(40,46,62), Text=Color3.fromRGB(240,244,252),
        SubText=Color3.fromRGB(145,155,175), Accent=Color3.fromRGB(70,150,255),
        AccentSoft=Color3.fromRGB(50,110,220),
    },
    ["Emerald"] = {
        Background=Color3.fromRGB(10,18,14), Surface=Color3.fromRGB(16,28,22),
        SurfaceAlt=Color3.fromRGB(22,38,30), Sidebar=Color3.fromRGB(12,22,16),
        Stroke=Color3.fromRGB(30,70,50), Text=Color3.fromRGB(235,255,245),
        SubText=Color3.fromRGB(150,190,170), Accent=Color3.fromRGB(40,220,140),
        AccentSoft=Color3.fromRGB(30,170,110),
    },
    ["Sunset"] = {
        Background=Color3.fromRGB(22,12,10), Surface=Color3.fromRGB(34,18,14),
        SurfaceAlt=Color3.fromRGB(46,24,18), Sidebar=Color3.fromRGB(26,14,12),
        Stroke=Color3.fromRGB(90,40,30), Text=Color3.fromRGB(255,245,235),
        SubText=Color3.fromRGB(220,180,160), Accent=Color3.fromRGB(255,120,40),
        AccentSoft=Color3.fromRGB(220,80,30),
    },
}

local CORNER    = UDim.new(0, 12)
local CORNER_SM = UDim.new(0, 8)
local TWEEN     = TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
local TWEEN_FAST= TweenInfo.new(0.14, Enum.EasingStyle.Quad,  Enum.EasingDirection.Out)

local IS_MOBILE = UserInputService.TouchEnabled and not UserInputService.MouseEnabled
local function ms(base) return IS_MOBILE and (base + 4) or base end
local function mh(base) return IS_MOBILE and math.floor(base * 1.35) or base end

---------------------------------------------------------------- HELPERS
local function new(class, props, children)
    local i = Instance.new(class)
    for k, v in pairs(props or {}) do i[k] = v end
    for _, c in ipairs(children or {}) do c.Parent = i end
    return i
end
local function corner(p, r) return new("UICorner", {CornerRadius = r or CORNER, Parent = p}) end
local function stroke(p, c, t, tr)
    return new("UIStroke", {
        Color = c or THEME.Stroke, Thickness = t or 1,
        Transparency = tr or 0,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Parent = p,
    })
end
local function padding(p, v)
    return new("UIPadding", {
        PaddingTop = UDim.new(0, v), PaddingBottom = UDim.new(0, v),
        PaddingLeft = UDim.new(0, v), PaddingRight = UDim.new(0, v), Parent = p,
    })
end
local function tween(o, p, i) TweenService:Create(o, i or TWEEN, p):Play() end

---------------------------------------------------------------- ONYX
local Onyx = {}
Onyx.__index = Onyx

function Onyx:Init()
    local old = (CoreGui:FindFirstChild("OnyxHub")) or (LP:FindFirstChild("PlayerGui") and LP.PlayerGui:FindFirstChild("OnyxHub"))
    if old then old:Destroy() end

    local gui = new("ScreenGui", {
        Name = "OnyxHub", IgnoreGuiInset = true,
        ResetOnSpawn = false, ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    })
    pcall(function() gui.Parent = CoreGui end)
    if not gui.Parent then gui.Parent = LP:WaitForChild("PlayerGui") end
    self.Gui = gui
    if self.NotificationsEnabled == nil then self.NotificationsEnabled = true end

    local size = IS_MOBILE and UDim2.new(0.95, 0, 0.82, 0) or UDim2.fromOffset(780, 500)

    local main = new("Frame", {
        Name = "Main",
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromScale(0.5, 0.5),
        Size = size,
        BackgroundColor3 = THEME.Background,
        BackgroundTransparency = 0.04,
        Parent = gui,
    })
    corner(main, CORNER); stroke(main, THEME.Accent, 1, 0.6)
    new("UISizeConstraint", {MinSize = Vector2.new(360, 320), MaxSize = Vector2.new(920, 600), Parent = main})
    new("UIAspectRatioConstraint", {
        AspectRatio = 1.56, DominantAxis = Enum.DominantAxis.Width,
        AspectType = Enum.AspectType.ScaleWithParentSize, Parent = main,
    })
    new("ImageLabel", {
        Name = "Shadow", AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromScale(0.5, 0.5),
        Size = UDim2.new(1, 50, 1, 50),
        BackgroundTransparency = 1,
        Image = "rbxassetid://5028857084",
        ImageColor3 = Color3.fromRGB(0, 0, 0),
        ImageTransparency = 0.5,
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(24, 24, 276, 276),
        ZIndex = 0, Parent = main,
    })

    self.Main = main
    self._BaseSize = size

    self:BuildSidebar()
    self:BuildContent()
    self:BuildTopBar()
    self:BuildOpenButton()
    self:BuildNotifications()

    main.Size = UDim2.fromScale(0, 0)
    tween(main, {Size = size}, TweenInfo.new(0.4, Enum.EasingStyle.Quint, Enum.EasingDirection.Out))

    self:_StartStats()
    return self
end

---------------------------------------------------------------- SIDEBAR
function Onyx:BuildSidebar()
    local side = new("Frame", {
        Name = "Sidebar",
        Size = UDim2.new(0, 170, 1, 0),
        BackgroundColor3 = THEME.Sidebar,
        Parent = self.Main,
    })
    corner(side, CORNER)
    new("Frame", {
        Size = UDim2.new(0, 14, 1, 0), Position = UDim2.new(1, -14, 0, 0),
        BackgroundColor3 = THEME.Sidebar, BorderSizePixel = 0, Parent = side,
    })

    local logo = new("Frame", {Size = UDim2.new(1, 0, 0, 50), BackgroundTransparency = 1, Parent = side})
    new("UIListLayout", {
        FillDirection = Enum.FillDirection.Horizontal,
        HorizontalAlignment = Enum.HorizontalAlignment.Center,
        VerticalAlignment = Enum.VerticalAlignment.Center,
        Padding = UDim.new(0, 4), Parent = logo,
    })
    new("TextLabel", {
        Size = UDim2.fromOffset(40, 50), BackgroundTransparency = 1,
        Font = Enum.Font.GothamBold, TextSize = 17, TextColor3 = THEME.Text,
        Text = "CLS", Parent = logo,
    })
    new("TextLabel", {
        Size = UDim2.fromOffset(35, 50), BackgroundTransparency = 1,
        Font = Enum.Font.GothamBold, TextSize = 17, TextColor3 = THEME.Accent,
        Text = "HUB", Parent = logo,
    })

    local tabs = new("ScrollingFrame", {
        Name = "Tabs",
        Position = UDim2.new(0, 8, 0, 58),
        Size = UDim2.new(1, -16, 1, -58 - 78),
        BackgroundTransparency = 1, BorderSizePixel = 0,
        ScrollBarThickness = 0,
        CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = side,
    })
    new("UIListLayout", {Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder, Parent = tabs})

    self.Sidebar = side
    self.TabsHolder = tabs
    self:BuildProfileCard(side)
end

function Onyx:BuildProfileCard(parent)
    local card = new("Frame", {
        Name = "Profile",
        AnchorPoint = Vector2.new(0.5, 1),
        Position = UDim2.new(0.5, 0, 1, -10),
        Size = UDim2.new(1, -16, 0, 60),
        BackgroundColor3 = THEME.SurfaceAlt, Parent = parent,
    })
    corner(card, UDim.new(0, 10)); stroke(card, THEME.Accent, 1, 0.7)

    local avatar = new("ImageLabel", {
        Position = UDim2.new(0, 8, 0.5, 0), AnchorPoint = Vector2.new(0, 0.5),
        Size = UDim2.fromOffset(40, 40), BackgroundColor3 = THEME.Surface,
        Image = "", Parent = card,
    })
    pcall(function()
        avatar.Image = Players:GetUserThumbnailAsync(LP.UserId,
            Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size100x100)
    end)
    new("UICorner", {CornerRadius = UDim.new(1, 0), Parent = avatar})
    stroke(avatar, THEME.Accent, 1, 0.5)

    new("TextLabel", {
        Position = UDim2.new(0, 56, 0, 10), Size = UDim2.new(1, -60, 0, 18),
        BackgroundTransparency = 1, Font = Enum.Font.GothamBold,
        TextSize = 13, TextColor3 = THEME.Text,
        Text = "NoBary", TextXAlignment = Enum.TextXAlignment.Left, Parent = card,
    })
    new("TextLabel", {
        Position = UDim2.new(0, 56, 0, 28), Size = UDim2.new(1, -60, 0, 18),
        BackgroundTransparency = 1, Font = Enum.Font.Gotham,
        TextSize = 11, TextColor3 = THEME.Accent,
        Text = "Premium Build", TextXAlignment = Enum.TextXAlignment.Left, Parent = card,
    })
end

---------------------------------------------------------------- TOP BAR
function Onyx:BuildTopBar()
    local top = new("Frame", {
        Name = "TopBar",
        Position = UDim2.new(0, 170, 0, 0),
        Size = UDim2.new(1, -170, 0, 46),
        BackgroundTransparency = 1, Parent = self.Main,
    })
    self.TopBar = top

    do
        local dragging, dragStart, startPos
        local main = self.Main
        top.Active = true
        top.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                dragging = true; dragStart = i.Position; startPos = main.Position
            end
        end)
        top.InputEnded:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                dragging = false
            end
        end)
        UserInputService.InputChanged:Connect(function(i)
            if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
                local d = i.Position - dragStart
                main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X,
                                          startPos.Y.Scale, startPos.Y.Offset + d.Y)
            end
        end)
    end

    local function pill(name, label, value, color, posX)
        local p = new("Frame", {
            Position = UDim2.new(0, posX, 0.5, 0),
            AnchorPoint = Vector2.new(0, 0.5),
            Size = UDim2.fromOffset(150, 30),
            BackgroundColor3 = THEME.SurfaceAlt,
            Parent = top,
        })
        corner(p, CORNER_SM); stroke(p, THEME.Stroke, 1, 0.4)

        local dot = new("Frame", {
            Position = UDim2.fromOffset(10, 11), Size = UDim2.fromOffset(8, 8),
            BackgroundColor3 = color, BorderSizePixel = 0, Parent = p,
        })
        new("UICorner", {CornerRadius = UDim.new(1, 0), Parent = dot})

        local holder = new("Frame", {
            Position = UDim2.fromOffset(24, 0),
            Size = UDim2.new(1, -32, 1, 0),
            BackgroundTransparency = 1, Parent = p,
        })
        new("TextLabel", {
            Size = UDim2.new(0.6, 0, 1, 0), BackgroundTransparency = 1,
            Font = Enum.Font.Gotham, TextSize = 10, TextColor3 = THEME.SubText,
            Text = label, TextXAlignment = Enum.TextXAlignment.Left, Parent = holder,
        })
        local val = new("TextLabel", {
            Position = UDim2.new(0.6, 0, 0, 0),
            Size = UDim2.new(0.4, 0, 1, 0),
            BackgroundTransparency = 1, Font = Enum.Font.GothamBold,
            TextSize = 11, TextColor3 = THEME.Text,
            Text = value, TextXAlignment = Enum.TextXAlignment.Right, Parent = holder,
        })
        self[name] = val
        return p
    end

    pill("PerfPill", "PERFORMANCE", "75 FPS", THEME.Success, 12)
    pill("LatPill",  "LATENCY",     "0 MS",   THEME.Accent,  172)

    local function ctrl(text, x, color)
        local b = new("TextButton", {
            AnchorPoint = Vector2.new(1, 0.5),
            Position = UDim2.new(1, x, 0.5, 0),
            Size = UDim2.fromOffset(30, 30),
            BackgroundColor3 = THEME.SurfaceAlt,
            Text = text, Font = Enum.Font.GothamBold,
            TextSize = 16, TextColor3 = color, AutoButtonColor = false,
            Parent = top,
        })
        corner(b, CORNER_SM); stroke(b, THEME.Stroke, 1, 0.4)
        b.MouseEnter:Connect(function() tween(b, {BackgroundColor3 = color}, TWEEN_FAST); tween(b, {TextColor3 = Color3.new(1,1,1)}, TWEEN_FAST) end)
        b.MouseLeave:Connect(function() tween(b, {BackgroundColor3 = THEME.SurfaceAlt}, TWEEN_FAST); tween(b, {TextColor3 = color}, TWEEN_FAST) end)
        return b
    end

    local mini = ctrl("-", -12, THEME.Accent)
    mini.MouseButton1Click:Connect(function() self:Minimize() end)

    local close = ctrl("X", -50, THEME.Danger)
    close.MouseButton1Click:Connect(function() self:Minimize() end)
end

---------------------------------------------------------------- OPEN BUTTON
function Onyx:BuildOpenButton()
    local btn = new("ImageButton", {
        Name = "OpenButton",
        AnchorPoint = Vector2.new(0, 0.5),
        Position = UDim2.new(0, 14, 0.5, 0),
        Size = UDim2.fromOffset(46, 46),
        BackgroundColor3 = THEME.Background,
        Image = "", AutoButtonColor = false, Visible = false,
        Parent = self.Gui,
    })
    corner(btn, UDim.new(1, 0)); stroke(btn, THEME.Accent, 2, 0.2)

    new("TextLabel", {
        Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
        Font = Enum.Font.GothamBold, TextSize = 18, TextColor3 = THEME.Accent,
        Text = "+", Parent = btn,
    })
    local glow = new("UIStroke", {
        Color = THEME.Accent, Thickness = 6, Transparency = 0.85, Parent = btn,
    })

    btn.MouseEnter:Connect(function() tween(glow, {Transparency = 0.6, Thickness = 8}, TWEEN_FAST) end)
    btn.MouseLeave:Connect(function() tween(glow, {Transparency = 0.85, Thickness = 6}, TWEEN_FAST) end)
    btn.MouseButton1Click:Connect(function() self:Maximize() end)

    local dragging, dragStart, startPos
    btn.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = true; dragStart = i.Position; startPos = btn.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - dragStart
            btn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X,
                                     startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    self.OpenBtn = btn
end

function Onyx:Minimize()
    tween(self.Main, {Size = UDim2.fromScale(0, 0)},
        TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.In))
    task.wait(0.26)
    self.Main.Visible = false
    self.OpenBtn.Visible = true
    self.OpenBtn.Size = UDim2.fromOffset(0, 0)
    tween(self.OpenBtn, {Size = UDim2.fromOffset(46, 46)},
        TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out))
    self:Notify("Menu minimizado", "Clique no ícone azul para abrir.")
end

function Onyx:Maximize()
    self.OpenBtn.Visible = false
    self.Main.Visible = true
    self.Main.Size = UDim2.fromScale(0, 0)
    tween(self.Main, {Size = self._BaseSize},
        TweenInfo.new(0.35, Enum.EasingStyle.Quint, Enum.EasingDirection.Out))
end

---------------------------------------------------------------- APPLY THEME
function Onyx:ApplyTheme(name)
    local preset = THEMES[name]; if not preset then return end
    local old = {Accent=THEME.Accent, AccentSoft=THEME.AccentSoft, Text=THEME.Text,
        SubText=THEME.SubText, Background=THEME.Background, Surface=THEME.Surface,
        SurfaceAlt=THEME.SurfaceAlt, Sidebar=THEME.Sidebar, Stroke=THEME.Stroke}
    for k,v in pairs(preset) do THEME[k] = v end
    if not self.Gui then return end
    local function near(a,b) return a and b and math.abs(a.R-b.R)<0.012 and math.abs(a.G-b.G)<0.012 and math.abs(a.B-b.B)<0.012 end
    local map = {
        {old.Accent, THEME.Accent}, {old.AccentSoft, THEME.AccentSoft},
        {old.Text, THEME.Text}, {old.SubText, THEME.SubText},
        {old.Background, THEME.Background}, {old.Surface, THEME.Surface},
        {old.SurfaceAlt, THEME.SurfaceAlt}, {old.Sidebar, THEME.Sidebar},
        {old.Stroke, THEME.Stroke},
    }
    local props = {"BackgroundColor3","TextColor3","ImageColor3","Color","ScrollBarImageColor3","PlaceholderColor3"}
    for _,d in ipairs(self.Gui:GetDescendants()) do
        for _,p in ipairs(props) do
            local ok, val = pcall(function() return d[p] end)
            if ok and typeof(val) == "Color3" then
                for _,pair in ipairs(map) do
                    if near(val, pair[1]) then pcall(function() d[p] = pair[2] end) break end
                end
            end
        end
    end
end

---------------------------------------------------------------- CONTENT
function Onyx:BuildContent()
    local content = new("Frame", {
        Name = "Content",
        Position = UDim2.new(0, 170, 0, 46),
        Size = UDim2.new(1, -170, 1, -46),
        BackgroundTransparency = 1, Parent = self.Main,
    })
    padding(content, 12)
    self.Content = content
    self.Tabs = {}
end

---------------------------------------------------------------- NOTIFICATIONS
function Onyx:BuildNotifications()
    local holder = new("Frame", {
        Name = "Notifications",
        AnchorPoint = Vector2.new(0.5, 0),
        Position = UDim2.new(0.5, 0, 0, 16),
        Size = UDim2.new(0, 320, 1, -32),
        BackgroundTransparency = 1, Parent = self.Gui,
    })
    new("UIListLayout", {
        Padding = UDim.new(0, 8),
        VerticalAlignment = Enum.VerticalAlignment.Top,
        HorizontalAlignment = Enum.HorizontalAlignment.Center,
        SortOrder = Enum.SortOrder.LayoutOrder, Parent = holder,
    })
    self.NotifHolder = holder
end

function Onyx:Notify(title, text, duration)
    if self.NotificationsEnabled == false then return end
    duration = duration or 3
    local n = new("Frame", {
        Size = UDim2.new(1, 0, 0, 56),
        BackgroundColor3 = THEME.Surface,
        Parent = self.NotifHolder,
    })
    corner(n, CORNER_SM); stroke(n, THEME.Accent, 1, 0.5)

    new("Frame", {
        Size = UDim2.new(0, 3, 1, -10),
        Position = UDim2.new(0, 6, 0, 5),
        BackgroundColor3 = THEME.Accent, BorderSizePixel = 0, Parent = n,
    })
    new("TextLabel", {
        Position = UDim2.new(0, 16, 0, 6), Size = UDim2.new(1, -22, 0, 18),
        BackgroundTransparency = 1, Font = Enum.Font.GothamBold,
        TextSize = 12, TextColor3 = THEME.Text,
        Text = title, TextXAlignment = Enum.TextXAlignment.Left, Parent = n,
    })
    new("TextLabel", {
        Position = UDim2.new(0, 16, 0, 26), Size = UDim2.new(1, -22, 0, 22),
        BackgroundTransparency = 1, Font = Enum.Font.Gotham,
        TextSize = 11, TextColor3 = THEME.SubText, TextWrapped = true,
        Text = text, TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Top, Parent = n,
    })

    n.Position = UDim2.new(0, 0, 0, -80)
    tween(n, {Position = UDim2.new(0, 0, 0, 0)})

    task.delay(duration, function()
        if n.Parent then
            tween(n, {BackgroundTransparency = 1}, TWEEN_FAST)
            for _, c in ipairs(n:GetDescendants()) do
                if c:IsA("TextLabel") then tween(c, {TextTransparency = 1}, TWEEN_FAST)
                elseif c:IsA("Frame") then tween(c, {BackgroundTransparency = 1}, TWEEN_FAST)
                elseif c:IsA("UIStroke") then tween(c, {Transparency = 1}, TWEEN_FAST) end
            end
            task.wait(0.2); n:Destroy()
        end
    end)
end

---------------------------------------------------------------- TABS
function Onyx:AddTab(name, icon)
    local tab = {Name = name}
    tab.Button = new("TextButton", {
        Size = UDim2.new(1, 0, 0, IS_MOBILE and 48 or 40),
        BackgroundColor3 = THEME.SurfaceAlt, BackgroundTransparency = 1,
        Text = "", AutoButtonColor = false, Parent = self.TabsHolder,
    })
    corner(tab.Button, CORNER_SM)

    tab.Indicator = new("Frame", {
        Size = UDim2.new(0, 3, 0.6, 0),
        Position = UDim2.new(0, 0, 0.2, 0),
        BackgroundColor3 = THEME.Accent, BackgroundTransparency = 1,
        BorderSizePixel = 0, Parent = tab.Button,
    })
    corner(tab.Indicator, UDim.new(1, 0))

    new("TextLabel", {
        Position = UDim2.new(0, 14, 0, 0), Size = UDim2.new(1, -18, 1, 0),
        BackgroundTransparency = 1, Font = Enum.Font.Gotham,
        TextSize = ms(14), TextColor3 = THEME.SubText,
        Text = (icon and (icon .. "  ") or "") .. name,
        TextXAlignment = Enum.TextXAlignment.Left, Name = "Label", Parent = tab.Button,
    })

    tab.Page = new("ScrollingFrame", {
        Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
        BorderSizePixel = 0, ScrollBarThickness = 2,
        ScrollBarImageColor3 = THEME.Accent,
        CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Visible = false, Parent = self.Content,
    })
    new("UIListLayout", {Padding = UDim.new(0, 10), Parent = tab.Page})

    tab.Button.MouseEnter:Connect(function()
        if self.Active ~= tab then tween(tab.Button, {BackgroundTransparency = 0.85}, TWEEN_FAST) end
    end)
    tab.Button.MouseLeave:Connect(function()
        if self.Active ~= tab then tween(tab.Button, {BackgroundTransparency = 1}, TWEEN_FAST) end
    end)
    tab.Button.MouseButton1Click:Connect(function() self:SelectTab(tab) end)

    table.insert(self.Tabs, tab)
    if not self.Active then self:SelectTab(tab) end

    function tab:AddCard(title) return Onyx._BuildCard(Onyx, self, title) end
    return tab
end

function Onyx:SelectTab(tab)
    for _, t in ipairs(self.Tabs) do
        t.Page.Visible = false
        tween(t.Button, {BackgroundTransparency = 1}, TWEEN_FAST)
        tween(t.Indicator, {BackgroundTransparency = 1}, TWEEN_FAST)
        tween(t.Button.Label, {TextColor3 = THEME.SubText}, TWEEN_FAST)
    end
    tab.Page.Visible = true
    tween(tab.Button, {BackgroundTransparency = 0.78}, TWEEN_FAST)
    tween(tab.Indicator, {BackgroundTransparency = 0}, TWEEN_FAST)
    tween(tab.Button.Label, {TextColor3 = THEME.Text}, TWEEN_FAST)
    self.Active = tab
end

---------------------------------------------------------------- CARDS
function Onyx._BuildCard(hub, tab, title)
    local card = new("Frame", {
        Size = UDim2.new(1, -4, 0, 40),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundColor3 = THEME.Surface,
        Parent = tab.Page,
    })
    corner(card, CORNER); stroke(card, THEME.Stroke, 1, 0.3); padding(card, 12)
    new("UIListLayout", {Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder, Parent = card})

    if title and title ~= "" then
        new("TextLabel", {
            Name = "CardTitle", LayoutOrder = 0,
            Size = UDim2.new(1, 0, 0, 20), BackgroundTransparency = 1,
            Font = Enum.Font.GothamBold, TextSize = 13, TextColor3 = THEME.Text,
            Text = title, TextXAlignment = Enum.TextXAlignment.Left, Parent = card,
        })
    end

    local api = {Frame = card}

    local function row(h)
        local r = new("Frame", {
            Size = UDim2.new(1, 0, 0, h or 30),
            BackgroundColor3 = THEME.SurfaceAlt, Parent = card,
        })
        corner(r, CORNER_SM); padding(r, 8)
        return r
    end

    function api:AddInfo(label, value)
        local r = row(mh(24))
        new("TextLabel", {
            Size = UDim2.new(0.5, 0, 1, 0), BackgroundTransparency = 1,
            Font = Enum.Font.Gotham, TextSize = ms(11), TextColor3 = THEME.SubText,
            Text = label, TextXAlignment = Enum.TextXAlignment.Left, Parent = r,
        })
        local v = new("TextLabel", {
            Position = UDim2.new(0.5, 0, 0, 0), Size = UDim2.new(0.5, 0, 1, 0),
            BackgroundTransparency = 1, Font = Enum.Font.GothamBold,
            TextSize = ms(11), TextColor3 = THEME.Text, Text = tostring(value),
            TextXAlignment = Enum.TextXAlignment.Right, Parent = r,
        })
        return v
    end

    function api:AddToggle(text, default, cb)
        local r = row(mh(30))
        new("TextLabel", {
            Size = UDim2.new(1, -56, 1, 0), BackgroundTransparency = 1,
            Font = Enum.Font.Gotham, TextSize = ms(12), TextColor3 = THEME.Text,
            Text = text, TextXAlignment = Enum.TextXAlignment.Left, Parent = r,
        })
        local sw = new("TextButton", {
            AnchorPoint = Vector2.new(1, 0.5),
            Position = UDim2.new(1, 0, 0.5, 0),
            Size = UDim2.fromOffset(IS_MOBILE and 52 or 38, IS_MOBILE and 28 or 20),
            BackgroundColor3 = THEME.Stroke,
            Text = "", AutoButtonColor = false, Parent = r,
        })
        corner(sw, UDim.new(1, 0))
        local KS = IS_MOBILE and 24 or 16
        local knob = new("Frame", {
            Size = UDim2.fromOffset(KS, KS), Position = UDim2.fromOffset(2, 2),
            BackgroundColor3 = THEME.Text, BorderSizePixel = 0, Parent = sw,
        })
        corner(knob, UDim.new(1, 0))

        local state = default and true or false
        local function refresh(silent)
            if state then
                tween(sw, {BackgroundColor3 = THEME.Accent})
                tween(knob, {Position = UDim2.fromOffset((IS_MOBILE and 26 or 20), 2)})
            else
                tween(sw, {BackgroundColor3 = THEME.Stroke})
                tween(knob, {Position = UDim2.fromOffset(2, 2)})
            end
            if cb then pcall(cb, state) end
            if not silent then
                hub:Notify(text, state and "Ativado" or "Desativado", 2.5)
            end
        end
        sw.MouseButton1Click:Connect(function() state = not state; refresh() end)
        if state then refresh(true) end
        return {Set = function(_, v) state = v; refresh() end}
    end

    function api:AddSlider(text, min, max, default, cb)
        min, max = min or 0, max or 100
        local value = default or min
        local OUTER_H = IS_MOBILE and 64 or 44
        local BAR_H   = IS_MOBILE and 10 or 5
        local KNOB    = IS_MOBILE and 22 or 14
        local outer = new("Frame", {
            Size = UDim2.new(1, 0, 0, OUTER_H),
            BackgroundColor3 = THEME.SurfaceAlt, Parent = card,
        })
        corner(outer, CORNER_SM); padding(outer, 8)
        new("TextLabel", {
            Size = UDim2.new(1, -60, 0, IS_MOBILE and 22 or 16), BackgroundTransparency = 1,
            Font = Enum.Font.Gotham, TextSize = ms(12), TextColor3 = THEME.Text,
            Text = text, TextXAlignment = Enum.TextXAlignment.Left, Parent = outer,
        })
        local vLbl = new("TextLabel", {
            AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, 0, 0, 0),
            Size = UDim2.fromOffset(60, IS_MOBILE and 22 or 16), BackgroundTransparency = 1,
            Font = Enum.Font.GothamBold, TextSize = ms(11), TextColor3 = THEME.Accent,
            Text = tostring(value), TextXAlignment = Enum.TextXAlignment.Right, Parent = outer,
        })
        local touchZone = new("Frame", {
            Position = UDim2.new(0, -4, 1, -(BAR_H+18)),
            Size = UDim2.new(1, 8, 0, BAR_H+28),
            BackgroundTransparency = 1, Parent = outer,
        })
        touchZone.Active = true
        local bar = new("Frame", {
            AnchorPoint = Vector2.new(0, 0.5),
            Position = UDim2.new(0, 4, 0.5, 0),
            Size = UDim2.new(1, -8, 0, BAR_H),
            BackgroundColor3 = THEME.Stroke, BorderSizePixel = 0, Parent = touchZone,
        })
        corner(bar, UDim.new(1, 0))
        local fill = new("Frame", {
            Size = UDim2.fromScale((value - min) / (max - min), 1),
            BackgroundColor3 = THEME.Accent, BorderSizePixel = 0, Parent = bar,
        })
        corner(fill, UDim.new(1, 0))
        local knob = new("Frame", {
            AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.new((value - min) / (max - min), 0, 0.5, 0),
            Size = UDim2.fromOffset(KNOB, KNOB),
            BackgroundColor3 = THEME.Text, BorderSizePixel = 0, Parent = bar,
        })
        corner(knob, UDim.new(1, 0)); stroke(knob, THEME.Accent, 2, 0)

        local dragging = false
        local function update(x)
            local rel = math.clamp((x - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
            value = math.floor(min + (max - min) * rel + 0.5)
            fill.Size = UDim2.fromScale(rel, 1)
            knob.Position = UDim2.new(rel, 0, 0.5, 0)
            vLbl.Text = tostring(value)
            if cb then pcall(cb, value) end
        end
        touchZone.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                dragging = true; update(i.Position.X)
            end
        end)
        UserInputService.InputChanged:Connect(function(i)
            if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
                update(i.Position.X)
            end
        end)
        UserInputService.InputEnded:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                dragging = false
            end
        end)
        return outer
    end

    function api:AddDropdown(text, options, cb)
        local current = options[1] or "—"
        local r = row(mh(30))
        new("TextLabel", {
            Size = UDim2.new(0.45, 0, 1, 0), BackgroundTransparency = 1,
            Font = Enum.Font.Gotham, TextSize = ms(12), TextColor3 = THEME.Text,
            Text = text, TextXAlignment = Enum.TextXAlignment.Left, Parent = r,
        })
        local btn = new("TextButton", {
            AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, 0, 0.5, 0),
            Size = UDim2.new(0.5, 0, 0, IS_MOBILE and 30 or 22),
            BackgroundColor3 = THEME.Surface, AutoButtonColor = false,
            Font = Enum.Font.Gotham, TextSize = ms(11), TextColor3 = THEME.Text,
            Text = "  " .. current .. "   ▾",
            TextXAlignment = Enum.TextXAlignment.Left, Parent = r,
        })
        corner(btn, CORNER_SM); stroke(btn, THEME.Stroke, 1, 0.4)

        local DH = IS_MOBILE and 34 or 24
        local list = new("Frame", {
            Visible = false, Size = UDim2.new(1, 0, 0, #options * DH),
            BackgroundColor3 = THEME.Surface, Parent = card,
        })
        corner(list, CORNER_SM); stroke(list, THEME.Stroke, 1, 0.3)
        new("UIListLayout", {Parent = list})
        for _, opt in ipairs(options) do
            local b = new("TextButton", {
                Size = UDim2.new(1, 0, 0, DH),
                BackgroundTransparency = 1, AutoButtonColor = false,
                Font = Enum.Font.Gotham, TextSize = ms(11), TextColor3 = THEME.SubText,
                Text = opt, Parent = list,
            })
            b.MouseEnter:Connect(function() b.TextColor3 = THEME.Accent end)
            b.MouseLeave:Connect(function() b.TextColor3 = THEME.SubText end)
            b.MouseButton1Click:Connect(function()
                current = opt; btn.Text = "  " .. current .. "   ▾"
                list.Visible = false
                if cb then pcall(cb, opt) end
                hub:Notify(text, "Selecionado: " .. opt, 2.5)
            end)
        end
        btn.MouseButton1Click:Connect(function() list.Visible = not list.Visible end)
        return r
    end

    function api:AddButton(text, cb)
        local b = new("TextButton", {
            Size = UDim2.new(1, 0, 0, mh(30)),
            BackgroundColor3 = THEME.Accent, BackgroundTransparency = 0.1,
            AutoButtonColor = false,
            Font = Enum.Font.GothamBold, TextSize = ms(12), TextColor3 = THEME.Text,
            Text = text, Parent = card,
        })
        corner(b, CORNER_SM)
        b.MouseEnter:Connect(function() tween(b, {BackgroundTransparency = 0}) end)
        b.MouseLeave:Connect(function() tween(b, {BackgroundTransparency = 0.1}) end)
        b.MouseButton1Click:Connect(function()
            if cb then pcall(cb) end
            hub:Notify(text, "Executado ✓", 2)
        end)
        return b
    end

    function api:AddKeybind(text, default, cb)
        local r = row(30)
        new("TextLabel", {
            Size = UDim2.new(1, -70, 1, 0), BackgroundTransparency = 1,
            Font = Enum.Font.Gotham, TextSize = 12, TextColor3 = THEME.Text,
            Text = text, TextXAlignment = Enum.TextXAlignment.Left, Parent = r,
        })
        local key = default or Enum.KeyCode.E
        local btn = new("TextButton", {
            AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, 0, 0.5, 0),
            Size = UDim2.fromOffset(64, 22),
            BackgroundColor3 = THEME.Surface, AutoButtonColor = false,
            Font = Enum.Font.GothamBold, TextSize = 10, TextColor3 = THEME.Accent,
            Text = key.Name, Parent = r,
        })
        corner(btn, CORNER_SM); stroke(btn, THEME.Stroke, 1, 0.4)
        btn.MouseButton1Click:Connect(function()
            btn.Text = "..."
            local c; c = UserInputService.InputBegan:Connect(function(i)
                if i.UserInputType == Enum.UserInputType.Keyboard then
                    key = i.KeyCode; btn.Text = key.Name
                    if cb then pcall(cb, key) end
                    hub:Notify(text, "Tecla: " .. key.Name, 2)
                    c:Disconnect()
                end
            end)
        end)
        return btn
    end

    function api:AddTextbox(text, placeholder, cb)
        local r = row(30)
        new("TextLabel", {
            Size = UDim2.new(0.4, 0, 1, 0), BackgroundTransparency = 1,
            Font = Enum.Font.Gotham, TextSize = 12, TextColor3 = THEME.Text,
            Text = text, TextXAlignment = Enum.TextXAlignment.Left, Parent = r,
        })
        local tb = new("TextBox", {
            AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, 0, 0.5, 0),
            Size = UDim2.new(0.55, 0, 0, 22),
            BackgroundColor3 = THEME.Surface, BorderSizePixel = 0,
            Font = Enum.Font.Gotham, TextSize = 11,
            TextColor3 = THEME.Text, PlaceholderColor3 = THEME.SubText,
            PlaceholderText = placeholder or "...",
            Text = "", ClearTextOnFocus = false, Parent = r,
        })
        corner(tb, CORNER_SM); stroke(tb, THEME.Stroke, 1, 0.4)
        tb.FocusLost:Connect(function() if cb then pcall(cb, tb.Text) end end)
        return tb
    end

    return api
end

---------------------------------------------------------------- STATS
function Onyx:_StartStats()
    task.spawn(function()
        while self.Gui and self.Gui.Parent do
            local fps = math.clamp(math.floor(1 / RunService.RenderStepped:Wait()), 1, 999)
            local ping = 0
            pcall(function() ping = math.floor(LP:GetNetworkPing() * 1000) end)
            if self.PerfPill then self.PerfPill.Text = fps .. " FPS" end
            if self.LatPill  then self.LatPill.Text  = ping .. " MS" end
            if self.FPSLabel then self.FPSLabel.Text = tostring(fps) end
            if self.PingLabel then self.PingLabel.Text = ping .. " ms" end
            if self.PlayersLabel then self.PlayersLabel.Text = tostring(#Players:GetPlayers()) end
        end
    end)
end

---------------------------------------------------------------- HELPERS EXTRAS
local function CardAddLabel(api, text, color)
    local lbl = new("TextLabel", {
        Size = UDim2.new(1, 0, 0, 18), BackgroundTransparency = 1,
        Font = Enum.Font.Gotham, TextSize = 11,
        TextColor3 = color or THEME.SubText,
        Text = text, TextXAlignment = Enum.TextXAlignment.Left,
        TextWrapped = true, Parent = api.Frame,
    })
    return lbl
end

local function CardAddColorPicker(api, hub, text, default, cb)
    local outer = new("Frame", {
        Size = UDim2.new(1, 0, 0, 80),
        BackgroundColor3 = THEME.SurfaceAlt, Parent = api.Frame,
    })
    corner(outer, CORNER_SM); padding(outer, 8)
    new("TextLabel", {
        Size = UDim2.new(1, -40, 0, 16), BackgroundTransparency = 1,
        Font = Enum.Font.Gotham, TextSize = 12, TextColor3 = THEME.Text,
        Text = text, TextXAlignment = Enum.TextXAlignment.Left, Parent = outer,
    })
    local preview = new("Frame", {
        AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, 0, 0, 0),
        Size = UDim2.fromOffset(30, 30), BackgroundColor3 = default,
        BorderSizePixel = 0, Parent = outer,
    })
    new("UICorner", {CornerRadius = UDim.new(1, 0), Parent = preview})
    local row = new("Frame", {
        Position = UDim2.fromOffset(0, 30), Size = UDim2.new(1, 0, 0, 34),
        BackgroundTransparency = 1, Parent = outer,
    })
    new("UIListLayout", {
        FillDirection = Enum.FillDirection.Horizontal,
        Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder,
        Parent = row,
    })
    local cols = {
        Color3.fromRGB(255,255,255), Color3.fromRGB(255,80,0), Color3.fromRGB(255,60,60),
        Color3.fromRGB(80,220,80), Color3.fromRGB(60,140,255), Color3.fromRGB(180,60,255),
        Color3.fromRGB(255,220,0), Color3.fromRGB(0,220,220), Color3.fromRGB(255,100,200),
    }
    for _, c in ipairs(cols) do
        local b = new("TextButton", {
            Size = UDim2.fromOffset(34, 34), Text = "",
            BackgroundColor3 = c, AutoButtonColor = true, Parent = row,
        })
        new("UICorner", {CornerRadius = UDim.new(1, 0), Parent = b})
        b.MouseButton1Click:Connect(function()
            preview.BackgroundColor3 = c
            if cb then pcall(cb, c) end
            if hub then hub:Notify(text, "Cor atualizada", 1.5) end
        end)
    end
    return preview
end

local function CardAddHotkeyField(api, hub, text, getHotkey, setHotkey)
    local r = new("Frame", {
        Size = UDim2.new(1, 0, 0, 30),
        BackgroundColor3 = THEME.SurfaceAlt, Parent = api.Frame,
    })
    corner(r, CORNER_SM); padding(r, 8)
    new("TextLabel", {
        Size = UDim2.new(0.5, 0, 1, 0), BackgroundTransparency = 1,
        Font = Enum.Font.Gotham, TextSize = 12, TextColor3 = THEME.Text,
        Text = text, TextXAlignment = Enum.TextXAlignment.Left, Parent = r,
    })
    local function fmt(h)
        if not h or not h.Key then return "?" end
        return tostring(h.Key.Name or h.Key)
    end
    local btn = new("TextButton", {
        AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, 0, 0.5, 0),
        Size = UDim2.fromOffset(110, 22),
        BackgroundColor3 = THEME.Surface, AutoButtonColor = false,
        Font = Enum.Font.GothamBold, TextSize = 11, TextColor3 = THEME.Accent,
        Text = fmt(getHotkey()), Parent = r,
    })
    corner(btn, CORNER_SM); stroke(btn, THEME.Stroke, 1, 0.4)
    btn.MouseButton1Click:Connect(function()
        btn.Text = "Pressione..."
        task.wait(0.12)
        local conn; local done=false
        conn = UserInputService.InputBegan:Connect(function(input, processed)
            if done or processed then return end
            local uit = input.UserInputType
            if uit == Enum.UserInputType.MouseButton1 or uit == Enum.UserInputType.MouseButton2 or uit == Enum.UserInputType.MouseButton3 then
                setHotkey({Kind="UserInputType", Key=uit})
                btn.Text = fmt(getHotkey()); done=true; conn:Disconnect()
                hub:Notify(text, "Hotkey: "..btn.Text, 2); return
            end
            if input.KeyCode and input.KeyCode ~= Enum.KeyCode.Unknown then
                setHotkey({Kind="KeyCode", Key=input.KeyCode})
                btn.Text = fmt(getHotkey()); done=true; conn:Disconnect()
                hub:Notify(text, "Hotkey: "..btn.Text, 2)
            end
        end)
        task.delay(5, function()
            if not done then done=true; pcall(function() conn:Disconnect() end)
                btn.Text = fmt(getHotkey()); hub:Notify(text, "Captura cancelada", 2)
            end
        end)
    end)
    return btn
end

local function CardAddPlayerList(api)
    local sf = new("ScrollingFrame", {
        Size = UDim2.new(1, 0, 0, 220), BackgroundTransparency = 1,
        BorderSizePixel = 0, ScrollBarThickness = 3,
        ScrollBarImageColor3 = THEME.Accent,
        CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = api.Frame,
    })
    new("UIListLayout", {Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder, Parent = sf})
    return sf
end

---------------------------------------------------------------- BUILD UI
local HttpService     = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Workspace       = game:GetService("Workspace")
local Camera          = Workspace.CurrentCamera

local hub = setmetatable({}, Onyx):Init()

local function notify(title, text, dur) hub:Notify(title, text, dur or 2) end

-- =========================================================================
--                           STATE / TOGGLES
-- =========================================================================
local Toggles = {
    Aimbot=false, AimbotMobile=false, AimbotMobileLook=false, AimbotHotkey=false,
    AimbotSmooth=false, CheckTeam=false, CheckWall=false, Spinbot=false,
    Noclip=false, AntiSit=true, AntiAfk=true,
    UseWhitelist=false, HitboxExpanded=false, LegitHitbox=false,
    SpeedCar=false, VFly=false,
    AutoFarmGari=false,
    NitroAuto=false,
}
local Whitelist = {}
local AimbotConfig = {
    FOVRadius=150, HitPart="Head", SpinSpeed=50,
    HeadSize=1.0, HeadTransparency=0, SmoothSpeed=0.15,
    Hotkey = {Kind="KeyCode", Key=Enum.KeyCode.E},
}
local FOVColor = THEME.Accent
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local hasDrawing = pcall(function() return Drawing end) and Drawing ~= nil

local lastKickTime = 0
local kickCooldown = 5
local carVelocity = 0
local carSensitivity = 2

local nitroForce = 50
local nitroConnection = nil
local nitroButtonMobile = nil

local TELEPORT_DELAY_SECONDS = 10

local noclipConn, antiSitConn, antiAfkThread, spinbotConn
local aimbotThread, mobileAutoAimConn, mobileLookConn, headSizeConn
local smoothAimbotConn, hotkeyAimbotActive = nil, false
local hitboxConn, hitboxOriginais, originaisLegit, visualPartsLegit = nil, {}, {}, {}
local hitboxSize, hitboxTransparency, legitTam = 15, 0.7, 6
local fovCircle
local aimbotActive = false

local function GetChar() return LP.Character end
local function GetHRP() local c=GetChar(); return c and c:FindFirstChild("HumanoidRootPart") end
local function SafeDrawing(t) if not hasDrawing then return nil end local ok,o=pcall(function() return Drawing.new(t) end); return ok and o or nil end
local function IsWhitelisted(plr) if not Toggles.UseWhitelist then return false end return Whitelist[plr.Name]==true end
local function FireProx(prompt)
    if not prompt or not prompt:IsA("ProximityPrompt") then return end
    if type(fireproximityprompt)=="function" then pcall(fireproximityprompt, prompt); return end
    pcall(function()
        prompt:InputHoldBegin(Enum.UserInputType.MouseButton1); task.wait(0.05)
        prompt:InputHoldEnd(Enum.UserInputType.MouseInput)
    end)
end

local function teleportTo(x, y, z)
    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then hrp.CFrame = CFrame.new(x, y, z) end
end

-- ============================ NOCLIP / ANTI-SIT / ANTI-AFK / SPINBOT
local function ToggleNoclip(s)
    Toggles.Noclip = s
    if s then
        if noclipConn then return end
        noclipConn = RunService.Stepped:Connect(function()
            if not Toggles.Noclip then return end
            local ch = GetChar(); if not ch then return end
            for _,p in ipairs(ch:GetDescendants()) do if p:IsA("BasePart") then pcall(function() p.CanCollide=false end) end end
        end)
        notify("Noclip", "Ativado ✓")
    else
        if noclipConn then noclipConn:Disconnect(); noclipConn=nil end
        local ch = GetChar()
        if ch then for _,p in ipairs(ch:GetDescendants()) do if p:IsA("BasePart") then pcall(function() p.CanCollide=true end) end end end
        notify("Noclip", "Desativado ✕")
    end
end
local function ToggleAntiSit(s, silent)
    Toggles.AntiSit = s
    if s then
        if antiSitConn then return end
        antiSitConn = RunService.Heartbeat:Connect(function()
            if not Toggles.AntiSit then return end
            local ch=GetChar(); if not ch then return end
            local h=ch:FindFirstChildOfClass("Humanoid")
            if h then if h.Sit then h.Sit=false end
                if h:GetState()==Enum.HumanoidStateType.Seated then h:ChangeState(Enum.HumanoidStateType.Running) end
            end
        end)
        if not silent then notify("Anti-Sit", "Ativado ✓") end
    else
        if antiSitConn then antiSitConn:Disconnect(); antiSitConn=nil end
        if not silent then notify("Anti-Sit", "Desativado ✕") end
    end
end
local function ToggleAntiAfk(s)
    Toggles.AntiAfk = s
    if s then
        if antiAfkThread then return end
        antiAfkThread = task.spawn(function()
            local t=false
            while Toggles.AntiAfk do
                task.wait(50)
                local ch=GetChar(); local h=ch and ch:FindFirstChildOfClass("Humanoid")
                if h then t=not t; pcall(function() h:Move(t and Vector3.new(0.1,0,0) or Vector3.new(-0.1,0,0),false); task.wait(0.1); h:Move(Vector3.new(),false) end) end
            end
        end)
        notify("Anti-AFK", "Ativado ✓")
    else
        if antiAfkThread then pcall(task.cancel, antiAfkThread); antiAfkThread=nil end
        notify("Anti-AFK", "Desativado ✕")
    end
end
local function ToggleSpinbot(s)
    Toggles.Spinbot = s
    if s then
        if spinbotConn then return end
        spinbotConn = RunService.Heartbeat:Connect(function()
            if not Toggles.Spinbot then return end
            local h=GetHRP()
            if h then pcall(function() h.CFrame=h.CFrame*CFrame.Angles(0,math.rad(AimbotConfig.SpinSpeed),0) end) end
        end)
        notify("Spinbot", "Ativado ✓")
    else
        if spinbotConn then spinbotConn:Disconnect(); spinbotConn=nil end
        notify("Spinbot", "Desativado ✕")
    end
end

-- ============================ HITBOX
local function expandirUpperTorso(ch)
    local p = ch and ch:FindFirstChild("UpperTorso")
    if p and p:IsA("BasePart") then
        if not hitboxOriginais[p] then
            hitboxOriginais[p] = {Size=p.Size,Transparency=p.Transparency,Material=p.Material,Color=p.Color,CanCollide=p.CanCollide,Massless=p.Massless}
        end
        pcall(function()
            p.Size = Vector3.new(hitboxSize,hitboxSize,hitboxSize)
            p.Transparency = hitboxTransparency
            p.Material = Enum.Material.Neon
            p.Color = Color3.fromRGB(255,0,0)
            p.CanCollide = false; p.Massless = true
        end)
    end
end
local function criarVisualPart(t)
    if visualPartsLegit[t] then return end
    local ok,v = pcall(function() return t:Clone() end); if not ok or not v then return end
    v.Size=t.Size; v.CFrame=t.CFrame; v.Anchored=false; v.CanCollide=false; v.Transparency=0
    v.Parent = t.Parent
    local w=Instance.new("Weld"); w.Part0=t; w.Part1=v; w.Parent=v
    visualPartsLegit[t]=v
end
local function expandirTronco(ch)
    local t = ch and ch:FindFirstChild("UpperTorso")
    if t and t:IsA("BasePart") then
        if not originaisLegit[t] then
            originaisLegit[t]={Size=t.Size,Transparency=t.Transparency}; criarVisualPart(t)
        end
        pcall(function() t.Size=Vector3.new(legitTam,legitTam,legitTam); t.Transparency=1 end)
    end
end
local function startHitboxLoop()
    if hitboxConn then return end
    hitboxConn = RunService.RenderStepped:Connect(function()
        if Toggles.HitboxExpanded then
            for _,plr in ipairs(Players:GetPlayers()) do
                if plr~=LP and plr.Character then pcall(expandirUpperTorso, plr.Character) end
            end
        end
        if Toggles.LegitHitbox then
            for _,plr in ipairs(Players:GetPlayers()) do
                if plr~=LP and plr.Character then pcall(expandirTronco, plr.Character) end
            end
        end
    end)
end
local function stopAndRestoreHitbox()
    if hitboxConn then hitboxConn:Disconnect(); hitboxConn=nil end
    for p,o in pairs(hitboxOriginais) do
        if p and p.Parent and o then pcall(function() p.Size=o.Size; p.Transparency=o.Transparency; p.Material=o.Material; p.Color=o.Color; p.CanCollide=o.CanCollide; p.Massless=o.Massless end) end
        hitboxOriginais[p]=nil
    end
    for t,o in pairs(originaisLegit) do
        if t and t.Parent and o then pcall(function() t.Size=o.Size; t.Transparency=o.Transparency end) end
        if visualPartsLegit[t] then pcall(function() visualPartsLegit[t]:Destroy() end); visualPartsLegit[t]=nil end
        originaisLegit[t]=nil
    end
end

-- ============================ AIMBOT
local function ApplyHeadSize()
    local ch=GetChar(); if not ch then return end
    local h=ch:FindFirstChild("Head")
    if h and h:IsA("BasePart") then
        pcall(function() h.Size=Vector3.new(AimbotConfig.HeadSize,AimbotConfig.HeadSize,AimbotConfig.HeadSize); h.Transparency=AimbotConfig.HeadTransparency end)
    end
end
local function StartHeadSizeLoop()
    if headSizeConn then headSizeConn:Disconnect() end
    headSizeConn = RunService.Heartbeat:Connect(function()
        if Toggles.Aimbot or Toggles.AimbotMobile or Toggles.AimbotHotkey or Toggles.AimbotMobileLook then ApplyHeadSize() end
    end)
end
local function GetValidPlayers()
    local l={}
    for _,p in ipairs(Players:GetPlayers()) do
        if p==LP then continue end
        if IsWhitelisted(p) then continue end
        if Toggles.CheckTeam then
            local m=LP.Team; local t=p.Team
            if m and t and m==t then continue end
        end
        local ch=p.Character; if not ch then continue end
        local h=ch:FindFirstChildOfClass("Humanoid")
        if h and h.Health>0 then table.insert(l,p) end
    end
    return l
end
local function IsVisible(part)
    if not Toggles.CheckWall then return true end
    local hrp=GetHRP(); if not hrp then return false end
    local rp=RaycastParams.new(); rp.FilterType=Enum.RaycastFilterType.Blacklist; rp.FilterDescendantsInstances={GetChar()}
    local r=workspace:Raycast(hrp.Position,(part.Position-hrp.Position).Unit*500,rp)
    if r and r.Instance then return r.Instance:IsDescendantOf(part.Parent) or r.Instance==part end
    return true
end
local function GetClosestInFOV()
    local cam=Camera; if not cam then return nil end
    local cx,cy=cam.ViewportSize.X/2,cam.ViewportSize.Y/2
    local ref=(isMobile or Toggles.AimbotMobileLook) and Vector2.new(cx,cy) or UserInputService:GetMouseLocation()
    local best,bestD=nil,AimbotConfig.FOVRadius
    for _,p in ipairs(GetValidPlayers()) do
        local ch=p.Character; if not ch then continue end
        local part=ch:FindFirstChild(AimbotConfig.HitPart) or ch:FindFirstChild("Head") or ch:FindFirstChild("HumanoidRootPart")
        if not part then continue end
        if not IsVisible(part) then continue end
        local ok,sp,onScr=pcall(function() return cam:WorldToViewportPoint(part.Position) end)
        if ok and onScr then
            local d=(Vector2.new(sp.X,sp.Y)-ref).Magnitude
            if d<bestD then bestD=d; best=p end
        end
    end
    return best
end
local function DoAim()
    local tg=GetClosestInFOV(); if not tg then return end
    local ch=tg.Character; if not ch then return end
    local part=ch:FindFirstChild(AimbotConfig.HitPart) or ch:FindFirstChild("Head") or ch:FindFirstChild("HumanoidRootPart")
    if part and Camera then pcall(function() Camera.CFrame=CFrame.new(Camera.CFrame.Position, part.Position) end) end
end
local function DoSmoothAim()
    local tg=GetClosestInFOV(); if not tg then return end
    local ch=tg.Character; if not ch then return end
    local part=ch:FindFirstChild(AimbotConfig.HitPart) or ch:FindFirstChild("Head") or ch:FindFirstChild("HumanoidRootPart")
    if part and Camera then pcall(function()
        local t=CFrame.new(Camera.CFrame.Position, part.Position)
        Camera.CFrame = Camera.CFrame:Lerp(t, AimbotConfig.SmoothSpeed)
    end) end
end
local function AimbotLoopPC()
    while Toggles.Aimbot do
        if Toggles.AimbotSmooth then DoSmoothAim()
        else
            local aiming=UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
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
    local h=AimbotConfig.Hotkey; if not h or not h.Key then return false end
    if h.Kind=="KeyCode" then return input.KeyCode==h.Key end
    if h.Kind=="UserInputType" then return input.UserInputType==h.Key end
    return false
end
local function SetupHotkeyAimbot()
    if smoothAimbotConn then smoothAimbotConn:Disconnect() end
    smoothAimbotConn = RunService.RenderStepped:Connect(function()
        if not Toggles.AimbotHotkey or not hotkeyAimbotActive then return end
        if Toggles.AimbotSmooth then DoSmoothAim() else DoAim() end
    end)
end
UserInputService.InputBegan:Connect(function(input) if Toggles.AimbotHotkey and IsHotkeyMatch(input) then hotkeyAimbotActive=true end end)
UserInputService.InputEnded:Connect(function(input) if IsHotkeyMatch(input) then hotkeyAimbotActive=false end end)

local function CreateFOVCircle()
    if fovCircle then pcall(function() fovCircle:Remove() end); fovCircle=nil end
    if not hasDrawing then return end
    local c=SafeDrawing("Circle"); if not c then return end
    pcall(function() c.Visible=false; c.Radius=AimbotConfig.FOVRadius; c.Color=FOVColor; c.Thickness=1; c.Filled=false; c.Transparency=1 end)
    fovCircle=c
    RunService.RenderStepped:Connect(function()
        if not fovCircle then return end
        pcall(function()
            if isMobile and Camera then fovCircle.Position=Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
            else fovCircle.Position=UserInputService:GetMouseLocation() end
            fovCircle.Visible = Toggles.Aimbot or Toggles.AimbotMobile or Toggles.AimbotHotkey or Toggles.AimbotMobileLook
            fovCircle.Radius = AimbotConfig.FOVRadius
            fovCircle.Color  = FOVColor
        end)
    end)
end

-- ============================ AUTO FARM GARI (caminhando + detecção de obstáculos)
local function findTrashEntries()
    local list = {}
    local lixeiro = workspace:FindFirstChild("Lixeiro")
    if not lixeiro then
        for _, v in ipairs(workspace:GetChildren()) do
            if string.find(string.lower(v.Name), "lixeiro") then lixeiro = v; break end
        end
    end
    if not lixeiro then return list end

    local function isPromptMatch(prompt)
        if not prompt then return false end
        local lname = string.lower(prompt.Name or "")
        local action = string.lower(prompt.ActionText or "")
        if lname:find("peg") or lname:find("lixo") or action:find("peg") or action:find("lixo") then
            return true
        end
        return false
    end

    local function inspectModel(model)
        for _, desc in ipairs(model:GetDescendants()) do
            if desc:IsA("ProximityPrompt") then
                if isPromptMatch(desc) then
                    table.insert(list, {target = model, prompt = desc})
                    return
                end
            end
        end
        for _, child in ipairs(model:GetDescendants()) do
            if child:IsA("RemoteEvent") or child:IsA("RemoteFunction") then
                local n = string.lower(child.Name or "")
                if n:find("peg") or n:find("lixo") or n:find("pega") then
                    table.insert(list, {target = model, remote = child})
                    return
                end
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

-- Função melhorada para andar até uma posição, desviando de obstáculos
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
        local result = workspace:Raycast(rayOrigin, rayDir * 3, raycastParams)
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
            while p and p ~= workspace do
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
        for attempt = 1, 3 do
            local ok, err = pcall(function()
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
            if not prompt or not prompt.Parent or not prompt.Parent:IsDescendantOf(workspace) then
                return true
            end
            if ok then
                if entry.target and not entry.target:IsDescendantOf(workspace) then return true end
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
        if entry.target and not entry.target:IsDescendantOf(workspace) then return true end
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
            if entry.target and not entry.target:IsDescendantOf(workspace) then return true end
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
                hub:Notify("Auto Farm", "Nenhum lixo encontrado...", 2)
                task.wait(5)
            else
                for i, entry in ipairs(entries) do
                    if not Toggles.AutoFarmGari then break end
                    if entry.target and not entry.target:IsDescendantOf(workspace) then
                        continue
                    end
                    local success = false
                    for attempt = 1, 3 do
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
                        hub:Notify("Auto Farm", "Aguardando " .. tostring(TELEPORT_DELAY_SECONDS) .. "s...", 1.4)
                        local waited = 0
                        while waited < TELEPORT_DELAY_SECONDS and Toggles.AutoFarmGari do
                            task.wait(0.5)
                            waited = waited + 0.5
                        end
                    end
                end
            end
            task.wait(0.6)
        end
    end)
end

local function toggleAutoFarm(state)
    Toggles.AutoFarmGari = state
    if state then
        notify("Auto Farm", "Auto Farm Gari ATIVADO! (caminhando + desvio de obstáculos)", 2)
        farmLoop()
    else
        notify("Auto Farm", "Auto Farm Gari DESATIVADO!", 2)
    end
end

-- ============================ NITRO AUTOMÁTICO
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
    while car and car ~= workspace do
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
                if size > maxSize then maxSize = size; mainPart = part end
            end
        end
        if not mainPart then mainPart = seat end
        for _, obj in pairs(mainPart:GetChildren()) do if obj:IsA("BodyVelocity") then obj:Destroy() end end
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
    if not seat then notify("Nitro", "Entre em um veículo primeiro!", 3); return end
    nitroMethod1(seat); task.wait(0.1); nitroMethod2(seat); task.wait(0.1); nitroMethod3(seat)
    notify("Nitro", "Nitro ativado!", 2)
end

local function toggleNitro(state)
    Toggles.NitroAuto = state
    if state then
        if nitroConnection then
            if isMobile and nitroConnection:IsA("TextButton") then nitroConnection:Destroy() else pcall(function() nitroConnection:Disconnect() end) end
        end
        if isMobile then
            if nitroButtonMobile then nitroButtonMobile:Destroy() end
            nitroButtonMobile = Instance.new("TextButton")
            nitroButtonMobile.Name = "NitroButtonMobile"
            nitroButtonMobile.Parent = hub.Gui
            nitroButtonMobile.Size = UDim2.new(0, 80, 0, 80)
            nitroButtonMobile.Position = UDim2.new(1, -100, 1, -100)
            nitroButtonMobile.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
            nitroButtonMobile.Text = "NITRO"
            nitroButtonMobile.TextColor3 = Color3.fromRGB(255, 255, 255)
            nitroButtonMobile.TextSize = 16
            nitroButtonMobile.ZIndex = 1000
            Instance.new("UICorner", nitroButtonMobile).CornerRadius = UDim.new(0, 20)
            nitroButtonMobile.MouseButton1Click:Connect(function() activateNitro() end)
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
            if isMobile and nitroConnection:IsA("TextButton") then nitroConnection:Destroy() else pcall(function() nitroConnection:Disconnect() end) end
            nitroConnection = nil
        end
        if nitroButtonMobile then nitroButtonMobile:Destroy(); nitroButtonMobile = nil end
        notify("Nitro", "Nitro automático desativado!", 3)
    end
end

-- ============================ LISTA DE PLAYERS (Whitelist)
local playerListSF
local function UpdatePlayerList()
    if not playerListSF then return end
    for _,c in ipairs(playerListSF:GetChildren()) do if c:IsA("Frame") then c:Destroy() end end
    local myHRP=GetHRP(); local list={}
    for _,p in ipairs(Players:GetPlayers()) do
        if p==LP then continue end
        local ch=p.Character; local hrp=ch and ch:FindFirstChild("HumanoidRootPart")
        local d=myHRP and hrp and math.floor((myHRP.Position-hrp.Position).Magnitude) or 999
        table.insert(list,{p=p,d=d})
    end
    table.sort(list,function(a,b) return a.d<b.d end)
    for _,it in ipairs(list) do
        local plr=it.p; local isFriend = Whitelist[plr.Name]==true
        local row = new("Frame", {Size=UDim2.new(1,-6,0,32), BackgroundColor3=THEME.SurfaceAlt, Parent=playerListSF})
        corner(row, CORNER_SM); padding(row, 6)
        new("TextLabel", {Size=UDim2.new(0.55,0,1,0), BackgroundTransparency=1, Font=Enum.Font.Gotham, TextSize=11, Text=plr.Name, TextXAlignment=Enum.TextXAlignment.Left, TextColor3 = isFriend and Color3.fromRGB(80,220,80) or THEME.Text, Parent=row})
        new("TextLabel", {Position=UDim2.new(0.55,0,0,0), Size=UDim2.new(0.2,0,1,0), BackgroundTransparency=1, Font=Enum.Font.Gotham, TextSize=11, Text=it.d.."m", TextColor3=THEME.SubText, Parent=row})
        local btn = new("TextButton", {AnchorPoint=Vector2.new(1,0.5), Position=UDim2.new(1,0,0.5,0), Size=UDim2.fromOffset(46,20), BackgroundColor3 = isFriend and THEME.Danger or THEME.Success, Font=Enum.Font.GothamBold, TextSize=10, TextColor3=Color3.new(1,1,1), Text=isFriend and "Remover" or "Amigo", Parent=row})
        corner(btn, UDim.new(1,0))
        btn.MouseButton1Click:Connect(function()
            if Whitelist[plr.Name] then Whitelist[plr.Name]=nil; notify("Whitelist", plr.Name.." removido", 1.5)
            else Whitelist[plr.Name]=true; notify("Whitelist", plr.Name.." adicionado", 1.5) end
            UpdatePlayerList()
        end)
    end
end
task.spawn(function() while hub.Gui and hub.Gui.Parent do if hub.Active and hub.Active.Name=="Amigos" then UpdatePlayerList() end; task.wait(1.5) end end)

-- =========================================================================
--                                ABAS DE UI
-- =========================================================================

-- HOME
local home = hub:AddTab("Home", nil)
do
    local c1 = home:AddCard("Active Features")
    c1:AddInfo("Features", "7 Active")
    c1:AddInfo("Performance", "Optimal")

    local c2 = home:AddCard("Player Info")
    c2:AddInfo("Account Age", LP.AccountAge .. " days")
    c2:AddInfo("User ID", tostring(LP.UserId))
    c2:AddInfo("Username", LP.Name)

    local c3 = home:AddCard("System Status")
    hub.FPSLabel    = c3:AddInfo("FPS", "—")
    hub.PingLabel   = c3:AddInfo("Ping", "—")
    hub.PlayersLabel= c3:AddInfo("Players Online", "—")
    local s = c3:AddInfo("Status", "Undetected ✓")
    s.TextColor3 = THEME.Success
end

-- AUTO FARM (Gari)
local farm = hub:AddTab("Auto Farm", nil)
do
    local c = farm:AddCard("Coleta de Lixo (Gari)")
    c:AddToggle("Auto Farm Gari (caminhando + desvio)", false, toggleAutoFarm)
    c:AddSlider("Delay entre coletas (s)", 2, 30, TELEPORT_DELAY_SECONDS, function(v)
        TELEPORT_DELAY_SECONDS = v
        notify("Auto Farm", "Delay ajustado para "..v.." segundos", 2)
    end)
    CardAddLabel(c, "O personagem andará até o lixo e desviará de obstáculos.")
end

-- COMBAT
local combat = hub:AddTab("Combat", nil)
do
    local c = combat:AddCard("Aimbot")
    if isMobile then
        c:AddToggle("Aimbot Mobile (Hold)", false, function(s)
            Toggles.AimbotMobile = s
            if s then SetupMobileAutoAim(); StartHeadSizeLoop() end
        end)
    end
    c:AddToggle("Aimbot Mobile (Look)", false, function(s)
        Toggles.AimbotMobileLook = s
        if s then SetupMobileLookAim(); StartHeadSizeLoop()
        else if mobileLookConn then mobileLookConn:Disconnect(); mobileLookConn=nil end end
    end)
    c:AddToggle("Aimbot (PC)", false, function(s)
        Toggles.Aimbot = s
        if s then aimbotThread = task.spawn(AimbotLoopPC); StartHeadSizeLoop()
        else if aimbotThread then pcall(task.cancel, aimbotThread); aimbotThread=nil end end
    end)
    c:AddToggle("Aimbot Hotkey (segurar)", false, function(s)
        Toggles.AimbotHotkey = s
        if s then SetupHotkeyAimbot(); StartHeadSizeLoop()
        else if smoothAimbotConn then smoothAimbotConn:Disconnect(); smoothAimbotConn=nil end; hotkeyAimbotActive=false end
    end)
    CardAddHotkeyField(c, hub, "Hotkey", function() return AimbotConfig.Hotkey end, function(h) AimbotConfig.Hotkey=h; SetupHotkeyAimbot() end)
    c:AddSlider("FOV Radius", 50, 300, AimbotConfig.FOVRadius, function(v) AimbotConfig.FOVRadius=v end)
    c:AddSlider("Smooth Speed (x100)", 5, 50, math.floor(AimbotConfig.SmoothSpeed*100), function(v) AimbotConfig.SmoothSpeed=v/100 end)
    c:AddSlider("Spin Speed", 10, 100, AimbotConfig.SpinSpeed, function(v) AimbotConfig.SpinSpeed=v end)
    c:AddToggle("Smooth Aimbot", false, function(s) Toggles.AimbotSmooth=s end)
    c:AddToggle("Ignorar Equipe", false, function(s) Toggles.CheckTeam=s end)
    c:AddToggle("Checar Parede", false, function(s) Toggles.CheckWall=s end)
    c:AddToggle("Spinbot", false, ToggleSpinbot)
    c:AddDropdown("Aim Part", {"Head","UpperTorso","HumanoidRootPart"}, function(v) AimbotConfig.HitPart=v end)
end

-- AMIGOS (Whitelist)
local friends = hub:AddTab("Amigos", nil)
do
    local c = friends:AddCard("Whitelist")
    c:AddToggle("Usar Whitelist", false, function(s) Toggles.UseWhitelist=s end)
    CardAddLabel(c, "Clique em 'Amigo' para adicionar à whitelist (não será mirado).")
    local list = friends:AddCard("Jogadores no servidor")
    playerListSF = CardAddPlayerList(list)
    task.delay(0.2, UpdatePlayerList)
end

-- OUTROS
local outros = hub:AddTab("Outros", nil)
do
    local m = outros:AddCard("Movimento")
    m:AddToggle("Noclip", false, ToggleNoclip)
    m:AddToggle("Anti-Sit", true, function(s) ToggleAntiSit(s) end)
    m:AddToggle("Anti-AFK", true, ToggleAntiAfk)

    local v = outros:AddCard("Veículos")
    v:AddToggle("Speed Car (W/S/Q)", false, function(s) Toggles.SpeedCar=s end)
    v:AddToggle("Fly Car (Tecla F)", false, function(s) Toggles.VFly=s end)
    v:AddSlider("Sensibilidade Carro", 0.5, 10, carSensitivity, function(val) carSensitivity=val end)
    v:AddToggle("Nitro Automático (Tecla N / Botão Mobile)", false, toggleNitro)
    v:AddSlider("Força do Nitro", 20, 150, nitroForce, function(val) nitroForce = val; notify("Nitro", "Força ajustada para: "..val, 1.6) end)

    local h = outros:AddCard("Hitbox")
    h:AddToggle("Hitbox Expandida", false, function(s) Toggles.HitboxExpanded=s; if s then startHitboxLoop() else stopAndRestoreHitbox() end end)
    h:AddSlider("Hitbox Size", 5, 50, hitboxSize, function(v) hitboxSize=v end)
    h:AddSlider("Hitbox Transparência (%)", 0, 100, math.floor(hitboxTransparency*100), function(v) hitboxTransparency=v/100 end)
    h:AddToggle("Hitbox LEGIT", false, function(s) Toggles.LegitHitbox=s; if s then startHitboxLoop() else stopAndRestoreHitbox() end end)
    h:AddSlider("LEGIT Size", 4, 15, legitTam, function(v) legitTam=v end)

    local s = outros:AddCard("Servidor")
    s:AddButton("Server Hop", function()
        pcall(function()
            local res = game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100")
            local data=HttpService:JSONDecode(res); local servers={}
            if data and data.data then for _,sv in ipairs(data.data) do if sv.playing<sv.maxPlayers and sv.id~=game.JobId then table.insert(servers,sv.id) end end end
            if #servers>0 then TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1,#servers)], LP) end
        end)
    end)
    s:AddButton("Rejoin", function() TeleportService:Teleport(game.PlaceId, LP) end)
end

-- CONFIG
local cfg = hub:AddTab("Config", "Cfg")
do
    local c = cfg:AddCard("Notificações")
    c:AddToggle("Ativar Notificações", true, function(s) hub.NotificationsEnabled = s end)
    CardAddLabel(c, "Desative para silenciar todas as mensagens do menu.")

    local lv = cfg:AddCard("Desempenho")
    local lightActive = false
    local function applyLightMode()
        for _,d in ipairs(workspace:GetDescendants()) do
            pcall(function()
                if d:IsA("BasePart") then
                    d.Material = Enum.Material.SmoothPlastic
                    d.Reflectance = 0
                elseif d:IsA("Decal") or d:IsA("Texture") then
                    d.Transparency = 1
                elseif d:IsA("ParticleEmitter") or d:IsA("Trail") or d:IsA("Smoke") or d:IsA("Fire") or d:IsA("Sparkles") or d:IsA("Beam") then
                    d.Enabled = false
                elseif d:IsA("Explosion") then
                    d.Visible = false
                end
            end)
        end
        pcall(function()
            local L = game:GetService("Lighting")
            for _,fx in ipairs(L:GetChildren()) do
                if fx:IsA("BlurEffect") or fx:IsA("BloomEffect") or fx:IsA("SunRaysEffect") or fx:IsA("ColorCorrectionEffect") or fx:IsA("DepthOfFieldEffect") then
                    fx.Enabled = false
                end
            end
            L.GlobalShadows = false
            L.FogEnd = 1e9
        end)
        pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
    end
    lv:AddToggle("Game Leve (sem texturas)", false, function(s)
        lightActive = s
        if s then
            applyLightMode()
            if not hub._lightConn then
                hub._lightConn = workspace.DescendantAdded:Connect(function(d)
                    if not lightActive then return end
                    task.wait(0.05)
                    pcall(function()
                        if d:IsA("BasePart") then d.Material = Enum.Material.SmoothPlastic; d.Reflectance = 0
                        elseif d:IsA("Decal") or d:IsA("Texture") then d.Transparency = 1
                        elseif d:IsA("ParticleEmitter") or d:IsA("Trail") or d:IsA("Smoke") or d:IsA("Fire") or d:IsA("Sparkles") or d:IsA("Beam") then d.Enabled = false end
                    end)
                end)
            end
        else
            if hub._lightConn then hub._lightConn:Disconnect(); hub._lightConn = nil end
            notify("Game Leve", "Recarregue para restaurar texturas")
        end
    end)

    local CONFIG_FILE = "OnyxHub_Config.json"
    local function hasFS() return type(writefile)=="function" and type(readfile)=="function" and type(isfile)=="function" end
    local function snapshotConfig()
        local data = {
            Toggles = {},
            AimbotConfig = {
                FOVRadius = AimbotConfig.FOVRadius,
                HitPart   = AimbotConfig.HitPart,
                SpinSpeed = AimbotConfig.SpinSpeed,
                HeadSize  = AimbotConfig.HeadSize,
                HeadTransparency = AimbotConfig.HeadTransparency,
                SmoothSpeed = AimbotConfig.SmoothSpeed,
            },
            HitboxSize = hitboxSize,
            HitboxTransparency = hitboxTransparency,
            LegitTam = legitTam,
            FOVColor = {FOVColor.R, FOVColor.G, FOVColor.B},
            Whitelist = {},
            NotificationsEnabled = hub.NotificationsEnabled,
            carSensitivity = carSensitivity,
            nitroForce = nitroForce,
            TELEPORT_DELAY_SECONDS = TELEPORT_DELAY_SECONDS,
        }
        for k,v in pairs(Toggles) do data.Toggles[k] = v end
        for name,_ in pairs(Whitelist) do table.insert(data.Whitelist, name) end
        return data
    end
    local function applyConfig(data)
        if type(data) ~= "table" then return end
        if type(data.Toggles) == "table" then
            for k,v in pairs(data.Toggles) do Toggles[k] = v end
        end
        if type(data.AimbotConfig) == "table" then
            for k,v in pairs(data.AimbotConfig) do AimbotConfig[k] = v end
        end
        if data.HitboxSize then hitboxSize = data.HitboxSize end
        if data.HitboxTransparency then hitboxTransparency = data.HitboxTransparency end
        if data.LegitTam then legitTam = data.LegitTam end
        if type(data.FOVColor)=="table" then FOVColor = Color3.new(data.FOVColor[1],data.FOVColor[2],data.FOVColor[3]) end
        if type(data.Whitelist)=="table" then
            Whitelist = {}
            for _,n in ipairs(data.Whitelist) do Whitelist[n] = true end
        end
        if data.NotificationsEnabled ~= nil then hub.NotificationsEnabled = data.NotificationsEnabled end
        if data.carSensitivity then carSensitivity = data.carSensitivity end
        if data.nitroForce then nitroForce = data.nitroForce end
        if data.TELEPORT_DELAY_SECONDS then TELEPORT_DELAY_SECONDS = data.TELEPORT_DELAY_SECONDS end
    end

    local sc = cfg:AddCard("Salvar / Carregar Config")
    CardAddLabel(sc, "Salva em um único arquivo (substitui o anterior).")
    sc:AddButton("Salvar Config", function()
        if not hasFS() then notify("Config", "Executor sem suporte a arquivos"); return end
        local ok, json = pcall(function() return HttpService:JSONEncode(snapshotConfig()) end)
        if not ok then notify("Config", "Erro ao codificar"); return end
        local okw = pcall(function() writefile(CONFIG_FILE, json) end)
        if okw then notify("Config", "Salvo em " .. CONFIG_FILE) else notify("Config", "Falha ao salvar") end
    end)
    sc:AddButton("Recarregar Config", function()
        if not hasFS() then notify("Config", "Executor sem suporte a arquivos"); return end
        if not isfile(CONFIG_FILE) then notify("Config", "Nenhuma config salva"); return end
        local ok, raw = pcall(function() return readfile(CONFIG_FILE) end)
        if not ok or not raw then notify("Config", "Falha ao ler"); return end
        local okd, data = pcall(function() return HttpService:JSONDecode(raw) end)
        if not okd then notify("Config", "Arquivo inválido"); return end
        applyConfig(data)
        notify("Config", "Config carregada")
    end)
    sc:AddButton("Apagar Config Salva", function()
        if not hasFS() then notify("Config", "Executor sem suporte"); return end
        if type(delfile)=="function" and isfile(CONFIG_FILE) then
            pcall(function() delfile(CONFIG_FILE) end)
            notify("Config", "Arquivo apagado")
        else
            notify("Config", "Nenhuma config para apagar")
        end
    end)

    local th = cfg:AddCard("Tema do Menu")
    local themeNames = {"CLS Pink","Midnight","Lurs Style","Onyx Blue","Emerald","Sunset"}
    th:AddDropdown("Escolher tema", themeNames, function(v)
        hub:ApplyTheme(v)
        hub:Notify("Tema", "Aplicado: "..v, 2)
    end)

    local mm = cfg:AddCard("Janela")
    mm:AddButton("Minimizar Menu", function() hub:Minimize() end)
    mm:AddButton("Maximizar Menu", function() hub:Maximize() end)
    CardAddLabel(mm, "Você também pode arrastar o menu pela barra do topo.")

    local a = cfg:AddCard("Ações")
    a:AddButton("Recarregar Personagem", function()
        local ch=GetChar(); local h=ch and ch:FindFirstChildOfClass("Humanoid"); if h then h.Health=0 end
    end)
    a:AddButton("Unload (remover GUI)", function()
        if hub.Gui then hub.Gui:Destroy() end
        if fovCircle then pcall(function() fovCircle:Remove() end) end
        stopAndRestoreHitbox()
        if nitroConnection then
            if isMobile and nitroConnection:IsA("TextButton") then nitroConnection:Destroy() else pcall(function() nitroConnection:Disconnect() end) end
        end
    end)
end

-- ============================ LOOP DE VEÍCULOS E OUTROS (sem ESP)
RunService.RenderStepped:Connect(function()
    -- Veículos
    local ch = LP.Character
    if ch and ch:FindFirstChild("Humanoid") and ch.Humanoid.SeatPart then
        local seatPart = ch.Humanoid.SeatPart
        if Toggles.VFly and UserInputService:IsKeyDown(Enum.KeyCode.F) then
            local direction = Vector3.new(0,0,0)
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

-- Inicialização
task.spawn(CreateFOVCircle)
ToggleAntiSit(true, true)
task.delay(0.4, function() hub:Notify("CLS HUB + EXTRA", "Carregado com sucesso (Auto Farm Gari com desvio de obstáculos)", 4) end)

return hub
