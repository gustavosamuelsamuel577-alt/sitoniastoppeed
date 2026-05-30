--[[
    ONYX HUB (Base UI) + FLUXOVIP Features + Head Size (from ilha bela) + Save/Load completo
    - Interface do ilha bela mantida (temas, notificações, minimizar, arrastar)
    - Funcionalidades completas do fluxovip implementadas
    - Adicionado: Head Size (própria cabeça) com slider e transparência
    - Sistema de salvar/carregar config (inclui head size e todas toggles)
    - CORREÇÃO: Aba Misc agora exibe todas as funções corretamente
--]]

local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local RunService       = game:GetService("RunService")
local CoreGui          = game:GetService("CoreGui")
local Camera           = workspace.CurrentCamera
local SoundService     = game:GetService("SoundService")
local HttpService      = game:GetService("HttpService")
local TeleportService  = game:GetService("TeleportService")
local Workspace        = game:GetService("Workspace")

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
        SurfaceAlt=Color3.fromRGB(26,30,42), Sidebar=Color3.fromRGB(12,14,24),
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
local TWEEN_FAST= TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

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

---------------------------------------------------------------- ONYX HUB (UI)
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
        Text = "FLUXO", Parent = logo,
    })
    new("TextLabel", {
        Size = UDim2.fromOffset(35, 50), BackgroundTransparency = 1,
        Font = Enum.Font.GothamBold, TextSize = 17, TextColor3 = THEME.Accent,
        Text = "VIP", Parent = logo,
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
        Text = LP.Name, TextXAlignment = Enum.TextXAlignment.Left, Parent = card,
    })
    new("TextLabel", {
        Position = UDim2.new(0, 56, 0, 28), Size = UDim2.new(1, -60, 0, 18),
        BackgroundTransparency = 1, Font = Enum.Font.Gotham,
        TextSize = 11, TextColor3 = THEME.Accent,
        Text = "Premium Build", TextXAlignment = Enum.TextXAlignment.Left, Parent = card,
    })
end

function Onyx:BuildTopBar()
    local top = new("Frame", {
        Name = "TopBar",
        Position = UDim2.new(0, 170, 0, 0),
        Size = UDim2.new(1, -170, 0, 46),
        BackgroundTransparency = 1, Parent = self.Main,
    })
    self.TopBar = top

    -- Drag
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

    pill("PerfPill", "FPS", "75", THEME.Success, 12)
    pill("LatPill",  "PING",  "0", THEME.Accent,  172)

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
    self:Notify("Menu minimizado", "Clique no ícone para abrir.")
end

function Onyx:Maximize()
    self.OpenBtn.Visible = false
    self.Main.Visible = true
    self.Main.Size = UDim2.fromScale(0, 0)
    tween(self.Main, {Size = self._BaseSize},
        TweenInfo.new(0.35, Enum.EasingStyle.Quint, Enum.EasingDirection.Out))
end

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

----------------------------------------------------------------
--                     FLUXOVIP FEATURES + HEAD SIZE
----------------------------------------------------------------

-- Variáveis de estado (global)
local aimbotEnabled = false
local FOVRadius = 200
local lockedTarget = nil
local killCheckEnabled = true
local wallCheckEnabled = true
local HeadOffset = 1

-- HEAD SIZE (from ilha bela)
local headSizeEnabled = false      -- toggle para aplicar na própria cabeça
local headSizeValue = 1.0          -- tamanho (1 = normal, até 5)
local headTransparency = 0         -- 0 = opaco, 1 = invisível
local headSizeConnection = nil

local drawingSupported, _ = pcall(function() return Drawing end)
local function newDrawing(kind)
    if not drawingSupported then return nil end
    local ok, obj = pcall(function() return Drawing.new(kind) end)
    if ok then return obj end
    return nil
end

local drawingFOV = nil
local fovVisible = true
if drawingSupported then
    drawingFOV = newDrawing("Circle")
    if drawingFOV then
        drawingFOV.Radius = FOVRadius
        drawingFOV.Color = THEME.Accent
        drawingFOV.Thickness = 2
        drawingFOV.Filled = false
        drawingFOV.Visible = false
    end
end

-- Aplicar head size no próprio personagem
local function applyHeadSize()
    local char = LP.Character
    if not char then return end
    local head = char:FindFirstChild("Head")
    if head and head:IsA("BasePart") then
        pcall(function()
            head.Size = Vector3.new(headSizeValue, headSizeValue, headSizeValue)
            head.Transparency = headTransparency
        end)
    end
end

local function startHeadSizeLoop()
    if headSizeConnection then return end
    headSizeConnection = RunService.Heartbeat:Connect(function()
        if headSizeEnabled then
            applyHeadSize()
        else
            -- restaurar tamanho normal caso desligado
            local char = LP.Character
            if char then
                local head = char:FindFirstChild("Head")
                if head and head:IsA("BasePart") then
                    pcall(function()
                        head.Size = Vector3.new(1, 1, 1)
                        head.Transparency = 0
                    end)
                end
            end
        end
    end)
end

-- Aimbot helpers
local function findHeadPart(char)
    if not char then return nil end
    for _, part in ipairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            local lname = part.Name:lower()
            if lname:find("head") or lname:find("cabeca") or lname:find("cabesa") then
                return part
            end
        end
    end
    local highest = nil
    for _, part in ipairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            if not highest or part.Position.Y > highest.Position.Y then
                highest = part
            end
        end
    end
    return highest or char:FindFirstChild("HumanoidRootPart") or char:FindFirstChildWhichIsA("BasePart")
end

local function findChestPart(char)
    if not char then return nil end
    for _, part in ipairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            local lname = part.Name:lower()
            if lname:find("torso") or lname:find("uppertorso") or lname:find("chest") or lname:find("peito") then
                return part
            end
        end
    end
    return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChildWhichIsA("BasePart")
end

local bodyShotModeEnabled = false
local bodyShotThreshold = 2
local shotCounts = {}
local prevHealth = {}

local function resetShotDataForPlayer(plr) shotCounts[plr] = 0; prevHealth[plr] = nil end

local friendSet = {}
local function isFriend(plr) return plr and friendSet[plr.UserId] == true end

local function getBestTarget()
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local closest, closestDist = nil, math.huge
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character and not isFriend(plr) then
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if killCheckEnabled then if not hum or hum.Health <= 0 then continue end end
            local headPart = findHeadPart(plr.Character)
            local chestPart = findChestPart(plr.Character) or headPart
            local targetPart = headPart
            if bodyShotModeEnabled then
                local cnt = shotCounts[plr] or 0
                if cnt < bodyShotThreshold then targetPart = chestPart or headPart else targetPart = headPart end
            end
            if targetPart then
                local aimPosition = targetPart.Position
                if targetPart ~= headPart and HeadOffset ~= 0 then
                    aimPosition = aimPosition + Vector3.new(0, HeadOffset, 0)
                end
                local screenPos, onScreen = Camera:WorldToViewportPoint(aimPosition)
                if onScreen then
                    if wallCheckEnabled then
                        local rayParams = RaycastParams.new()
                        rayParams.FilterDescendantsInstances = {LP.Character}
                        rayParams.FilterType = Enum.RaycastFilterType.Blacklist
                        local rayResult = workspace:Raycast(Camera.CFrame.Position, (aimPosition - Camera.CFrame.Position).Unit * 1000, rayParams)
                        if rayResult and rayResult.Instance and rayResult.Instance:IsDescendantOf(plr.Character) then
                            local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                            if dist < closestDist and dist <= FOVRadius then closest, closestDist = targetPart, dist end
                        end
                    else
                        local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                        if dist < closestDist and dist <= FOVRadius then closest, closestDist = targetPart, dist end
                    end
                end
            end
        end
    end
    return closest
end

local function updateFOVVisual()
    if drawingSupported and drawingFOV then
        pcall(function()
            drawingFOV.Radius = FOVRadius
            drawingFOV.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
            drawingFOV.Color = THEME.Accent
            drawingFOV.Visible = aimbotEnabled and fovVisible
        end)
    end
end

-- RapidFire
local rapidFireEnabled = false
local rapidFireDelay = 0.06
local actionDown = false
local rapidFireLoop = nil

local function startRapidFire()
    if rapidFireLoop then return end
    rapidFireLoop = task.spawn(function()
        while actionDown and rapidFireEnabled do
            pcall(function()
                local char = LP.Character
                if char then
                    local tool = nil
                    for _, c in ipairs(char:GetChildren()) do
                        if c:IsA("Tool") then tool = c; break end
                    end
                    if tool and tool.Parent == char then
                        pcall(function() tool:Activate() end)
                    end
                end
            end)
            task.wait(rapidFireDelay)
        end
        rapidFireLoop = nil
    end)
end

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        actionDown = true
        if rapidFireEnabled then startRapidFire() end
    end
end)
UserInputService.InputEnded:Connect(function(input, processed)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        actionDown = false
        mobileAimDown = false
    end
end)

-- Hold-to-aim
local holdToAimEnabled = false
local aimKey = Enum.KeyCode.E
local mobileAimDown = false

RunService.RenderStepped:Connect(function()
    updateFOVVisual()
    if not aimbotEnabled then return end
    if holdToAimEnabled then
        local active = false
        if IS_MOBILE then active = mobileAimDown else active = UserInputService:IsKeyDown(aimKey) end
        if not active then lockedTarget = nil; return end
    end
    local target = getBestTarget()
    if target then
        pcall(function()
            local aimPos = target.Position
            local headPart = findHeadPart(target.Parent)
            if headPart and target ~= headPart and HeadOffset ~= 0 then
                aimPos = aimPos + Vector3.new(0, HeadOffset, 0)
            end
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, aimPos)
            lockedTarget = target
        end)
    else lockedTarget = nil end
end)

-- ESP (Billboard, Highlight, Drawing)
local billboardEnabled = false
local highlightEnabled = true
local espEnabled = false
local tracersEnabled = false
local distanceEnabled = false
local rainbowESPEnabled = false
local box3dEnabled = false
local espColor = Color3.fromRGB(64, 156, 255)
local ESPS = {}

local function ensureStorage(name)
    local g = LP.PlayerGui:FindFirstChild(name)
    if not g then
        g = Instance.new("Folder")
        g.Name = name
        g.Parent = LP.PlayerGui
    end
    return g
end

local function createHighlightForCharacter(char, plr)
    if not char then return end
    pcall(function()
        local storage = ensureStorage("Stopped_Highlight_Storage")
        local nodeName = (plr and plr.Name) or char.Name or tostring(char:GetDebugId())
        if storage:FindFirstChild(nodeName) then storage[nodeName]:Destroy() end
        local h = Instance.new("Highlight")
        h.Name = nodeName
        h.FillColor = THEME.Accent
        h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        h.FillTransparency = 0.25
        h.OutlineColor = Color3.fromRGB(255,255,255)
        h.Parent = storage
        if char then h.Adornee = char end
    end)
end

local function destroyHighlightForCharacter(char)
    if not char then return end
    local storage = LP.PlayerGui:FindFirstChild("Stopped_Highlight_Storage")
    if not storage then return end
    for _, h in ipairs(storage:GetChildren()) do
        if h:IsA("Highlight") and h.Adornee == char then
            pcall(function() h:Destroy() end)
        end
    end
end

local function createBillboardForCharacter(char, plr)
    if not char or not plr then return end
    local storage = ensureStorage("Stopped_Billboard_Storage")
    local nodeName = plr.Name.."_bb"
    if storage:FindFirstChild(nodeName) then storage[nodeName]:Destroy() end

    local head = findHeadPart(char)
    if not head then return end

    local bb = Instance.new("BillboardGui")
    bb.Name = nodeName
    bb.Size = UDim2.new(0, 140, 0, 48)
    bb.StudsOffset = Vector3.new(0, 2.4, 0)
    bb.Adornee = head
    bb.AlwaysOnTop = true
    bb.Parent = storage

    local bg = Instance.new("Frame", bb)
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundTransparency = 0.55
    bg.BackgroundColor3 = Color3.fromRGB(10,10,10)
    bg.BorderSizePixel = 0
    Instance.new("UICorner", bg).CornerRadius = UDim.new(0,6)

    local nameLbl = Instance.new("TextLabel", bg)
    nameLbl.Size = UDim2.new(1, -8, 0, 18)
    nameLbl.Position = UDim2.new(0,4,0,2)
    nameLbl.BackgroundTransparency = 1
    nameLbl.Font = Enum.Font.GothamBold
    nameLbl.TextSize = 14
    nameLbl.TextColor3 = THEME.Text
    nameLbl.TextXAlignment = Enum.TextXAlignment.Left
    nameLbl.Text = plr.Name

    local infoLbl = Instance.new("TextLabel", bg)
    infoLbl.Size = UDim2.new(1, -8, 0, 16)
    infoLbl.Position = UDim2.new(0,4,0,22)
    infoLbl.BackgroundTransparency = 1
    infoLbl.Font = Enum.Font.Gotham
    infoLbl.TextSize = 12
    infoLbl.TextColor3 = THEME.SubText
    infoLbl.TextXAlignment = Enum.TextXAlignment.Left
    infoLbl.Text = "0m | HP: --"

    local dot = Instance.new("Frame", bg)
    dot.Name = "Dot"
    dot.Size = UDim2.new(0,8,0,8)
    dot.Position = UDim2.new(0,4,0,20)
    dot.BackgroundColor3 = THEME.Accent
    dot.BorderSizePixel = 0
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1,0)
end

local function destroyBillboardForCharacter(char, plr)
    local storage = LP.PlayerGui:FindFirstChild("Stopped_Billboard_Storage")
    if not storage then return end
    local nodeName = (plr and plr.Name) and (plr.Name.."_bb") or nil
    if nodeName and storage:FindFirstChild(nodeName) then pcall(function() storage[nodeName]:Destroy() end) end
end

local function destroyAllBillboards()
    local storage = LP.PlayerGui:FindFirstChild("Stopped_Billboard_Storage")
    if storage then pcall(function() storage:Destroy() end) end
end
local function destroyAllHighlights()
    local storage = LP.PlayerGui:FindFirstChild("Stopped_Highlight_Storage")
    if storage then pcall(function() storage:Destroy() end) end
end

local function createESPObjectsForPlayer(plr)
    if not drawingSupported then return nil end
    if ESPS[plr] then return ESPS[plr] end
    local data = {}
    data.box = newDrawing("Quad")
    if data.box then data.box.Thickness = 2; data.box.Transparency = 1; data.box.Visible = false end
    data.healthText = newDrawing("Text")
    if data.healthText then data.healthText.Size = 14; data.healthText.Color = Color3.fromRGB(255,255,255); data.healthText.Center = true; data.healthText.Outline = true; data.healthText.Visible = false end
    data.healthBar = newDrawing("Line")
    if data.healthBar then data.healthBar.Thickness = 4; data.healthBar.Transparency = 1; data.healthBar.Visible = false end
    data.tracer = newDrawing("Line")
    if data.tracer then data.tracer.Thickness = 2; data.tracer.Transparency = 0.8; data.tracer.Visible = false end
    data.distanceText = newDrawing("Text")
    if data.distanceText then data.distanceText.Size = 14; data.distanceText.Color = Color3.fromRGB(200,200,255); data.distanceText.Center = true; data.distanceText.Outline = true; data.distanceText.Visible = false end
    data.nameText = newDrawing("Text")
    if data.nameText then data.nameText.Size = 15; data.nameText.Color = Color3.fromRGB(200,230,255); data.nameText.Center = true; data.nameText.Outline = true; data.nameText.Visible = false end
    data.box3dLines = {}
    for i = 1, 12 do
        local l = newDrawing("Line")
        if l then l.Thickness = 2; l.Visible = false end
        table.insert(data.box3dLines, l)
    end
    ESPS[plr] = data
    return data
end

local function removeESPObjects(plr)
    local data = ESPS[plr]
    if not data then return end
    local function safeRemove(obj)
        if not obj then return end
        pcall(function()
            if obj.Visible ~= nil then obj.Visible = false end
            if obj.Remove then obj:Remove() end
        end)
    end
    safeRemove(data.box)
    safeRemove(data.healthText)
    safeRemove(data.healthBar)
    safeRemove(data.tracer)
    safeRemove(data.distanceText)
    safeRemove(data.nameText)
    for _, l in ipairs(data.box3dLines) do safeRemove(l) end
    ESPS[plr] = nil
end

RunService.RenderStepped:Connect(function()
    if billboardEnabled then
        local storage = LP.PlayerGui:FindFirstChild("Stopped_Billboard_Storage")
        if storage then
            for _, bb in ipairs(storage:GetChildren()) do
                if bb:IsA("BillboardGui") then
                    local ador = bb.Adornee
                    if not ador or not ador.Parent then pcall(function() bb:Destroy() end) else
                        local plr = Players:GetPlayerFromCharacter(ador.Parent)
                        if plr and not isFriend(plr) then
                            local humanoid = ador.Parent:FindFirstChildOfClass("Humanoid")
                            local hp = humanoid and math.floor(humanoid.Health) or 0
                            local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                            local dist = root and (root.Position - ador.Position).Magnitude or 0
                            local textLabels = {}
                            for _,c in ipairs(bb:GetDescendants()) do if c:IsA("TextLabel") then table.insert(textLabels, c) end end
                            local infoLabel = textLabels[2]
                            if infoLabel then
                                local distDisplay = (dist>=1000) and string.format("%.1fk", dist/1000) or string.format("%dm", math.floor(dist))
                                infoLabel.Text = distDisplay.." | HP: "..tostring(hp)
                            end
                        end
                    end
                end
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    if not drawingSupported then return end
    for plr, data in pairs(ESPS) do
        if not plr or not plr.Character or plr == LP or isFriend(plr) then
            if data.box then data.box.Visible = false end
            if data.healthText then data.healthText.Visible = false end
            if data.healthBar then data.healthBar.Visible = false end
            if data.tracer then data.tracer.Visible = false end
            if data.distanceText then data.distanceText.Visible = false end
            if data.nameText then data.nameText.Visible = false end
            for _, l in ipairs(data.box3dLines) do if l then l.Visible = false end end
        else
            local char = plr.Character
            local root = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso") or char:FindFirstChildWhichIsA("BasePart")
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not root or not hum or hum.Health <= 0 then
                if data.box then data.box.Visible = false end
                if data.healthText then data.healthText.Visible = false end
                if data.healthBar then data.healthBar.Visible = false end
                if data.tracer then data.tracer.Visible = false end
                if data.distanceText then data.distanceText.Visible = false end
                if data.nameText then data.nameText.Visible = false end
                for _, l in ipairs(data.box3dLines) do if l then l.Visible = false end end
            else
                local pos = root.Position
                local size = Vector3.new(2, 3, 1.5)
                local tl = Camera:WorldToViewportPoint(pos + Vector3.new(-size.X, size.Y, 0))
                local tr = Camera:WorldToViewportPoint(pos + Vector3.new(size.X, size.Y, 0))
                local bl = Camera:WorldToViewportPoint(pos + Vector3.new(-size.X, -size.Y, 0))
                local br = Camera:WorldToViewportPoint(pos + Vector3.new(size.X, -size.Y, 0))

                local color = rainbowESPEnabled and (Color3.fromHSV((tick() % 5) / 5, 1, 1)) or espColor

                if data.box and espEnabled and not box3dEnabled and tl.Z > 0 and tr.Z > 0 and bl.Z > 0 and br.Z > 0 then
                    data.box.PointA = Vector2.new(tl.X, tl.Y)
                    data.box.PointB = Vector2.new(tr.X, tr.Y)
                    data.box.PointC = Vector2.new(br.X, br.Y)
                    data.box.PointD = Vector2.new(bl.X, bl.Y)
                    data.box.Color = color
                    data.box.Visible = true
                elseif data.box then
                    data.box.Visible = false
                end

                if espEnabled and box3dEnabled then
                    local corners = (function(cf, s)
                        local c = {}
                        local sx, sy, sz = s.X/2, s.Y/2, s.Z/2
                        local points = {
                            Vector3.new(-sx, -sy, -sz), Vector3.new(sx, -sy, -sz),
                            Vector3.new(sx, sy, -sz), Vector3.new(-sx, sy, -sz),
                            Vector3.new(-sx, -sy, sz), Vector3.new(sx, -sy, sz),
                            Vector3.new(sx, sy, sz), Vector3.new(-sx, sy, sz)
                        }
                        for i, p in ipairs(points) do
                            local world = root.CFrame:PointToWorldSpace(p)
                            local screen, vis = Camera:WorldToViewportPoint(world)
                            c[i] = {Vector2.new(screen.X, screen.Y), vis and screen.Z > 0}
                        end
                        return c
                    end)(root.CFrame, size * 1.1)
                    local indices = {{1,2},{2,3},{3,4},{4,1},{5,6},{6,7},{7,8},{8,5},{1,5},{2,6},{3,7},{4,8}}
                    for i = 1, 12 do
                        local a, b = indices[i][1], indices[i][2]
                        local line = data.box3dLines[i]
                        if corners[a][2] and corners[b][2] and line then
                            line.From = corners[a][1]
                            line.To = corners[b][1]
                            line.Color = color
                            line.Visible = true
                        elseif line then
                            line.Visible = false
                        end
                    end
                else
                    for _, l in ipairs(data.box3dLines) do if l then l.Visible = false end end
                end

                if tl.Z > 0 and tr.Z > 0 and bl.Z > 0 and br.Z > 0 then
                    if espEnabled then
                        if data.nameText then
                            data.nameText.Text = plr.DisplayName or plr.Name
                            data.nameText.Position = Vector2.new((tl.X + tr.X)/2, tl.Y - 18)
                            data.nameText.Color = color
                            data.nameText.Visible = true
                        end
                        if data.healthText then
                            data.healthText.Text = tostring(math.floor(hum.Health)) .. " HP"
                            data.healthText.Position = Vector2.new((tl.X + tr.X)/2, tl.Y - 32)
                            data.healthText.Color = color
                            data.healthText.Visible = true
                        end
                        if data.healthBar then
                            local hpPercent = hum.Health / math.max(1, hum.MaxHealth)
                            local barHeight = (bl.Y - tl.Y) * hpPercent
                            data.healthBar.From = Vector2.new(bl.X - 6, bl.Y)
                            data.healthBar.To = Vector2.new(bl.X - 6, bl.Y - barHeight)
                            data.healthBar.Color = hpPercent > 0.5 and Color3.fromRGB(0,255,0) or hpPercent > 0.2 and Color3.fromRGB(255,255,0) or Color3.fromRGB(255,0,0)
                            data.healthBar.Visible = true
                        end
                        if distanceEnabled and data.distanceText then
                            local dist = (Camera.CFrame.Position - root.Position).Magnitude
                            data.distanceText.Text = string.format("%.1f m", dist)
                            data.distanceText.Position = Vector2.new((bl.X + br.X)/2, bl.Y + 18)
                            data.distanceText.Visible = true
                            data.distanceText.Color = color
                        elseif data.distanceText then
                            data.distanceText.Visible = false
                        end
                    else
                        if data.nameText then data.nameText.Visible = false end
                        if data.healthText then data.healthText.Visible = false end
                        if data.healthBar then data.healthBar.Visible = false end
                        if data.distanceText then data.distanceText.Visible = false end
                    end
                    if data.tracer then
                        data.tracer.Visible = tracersEnabled
                        if tracersEnabled then
                            data.tracer.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                            data.tracer.To = Vector2.new((bl.X + br.X)/2, (bl.Y + br.Y)/2)
                            data.tracer.Color = color
                        end
                    end
                else
                    if data.nameText then data.nameText.Visible = false end
                    if data.healthText then data.healthText.Visible = false end
                    if data.healthBar then data.healthBar.Visible = false end
                    if data.tracer then data.tracer.Visible = false end
                    if data.distanceText then data.distanceText.Visible = false end
                end
            end
        end
    end
end)

Players.PlayerAdded:Connect(function(plr)
    resetShotDataForPlayer(plr)
    plr.CharacterAdded:Connect(function(char)
        if highlightEnabled then createHighlightForCharacter(char, plr) end
        if billboardEnabled then createBillboardForCharacter(char, plr) end
        if drawingSupported then createESPObjectsForPlayer(plr) end
    end)
    if plr.Character then
        local char = plr.Character
        if highlightEnabled then createHighlightForCharacter(char, plr) end
        if billboardEnabled then createBillboardForCharacter(char, plr) end
        if drawingSupported then createESPObjectsForPlayer(plr) end
    end
end)
Players.PlayerRemoving:Connect(function(plr)
    local char = plr.Character
    if char then
        destroyHighlightForCharacter(char)
        destroyBillboardForCharacter(char, plr)
    end
    shotCounts[plr] = nil; prevHealth[plr] = nil; friendSet[plr.UserId] = nil
    if drawingSupported then removeESPObjects(plr) end
end)

for _, plr in ipairs(Players:GetPlayers()) do
    resetShotDataForPlayer(plr)
    plr.CharacterAdded:Connect(function(char)
        if highlightEnabled then createHighlightForCharacter(char, plr) end
        if billboardEnabled then createBillboardForCharacter(char, plr) end
        if drawingSupported then createESPObjectsForPlayer(plr) end
    end)
    if plr.Character then
        local char = plr.Character
        if highlightEnabled then createHighlightForCharacter(char, plr) end
        if billboardEnabled then createBillboardForCharacter(char, plr) end
    end
end

-- Movement (Spinbot, Fake Dash, Fake Lag)
local spinbotEnabled = false
local spinSpeed = 50
local spinbotConnection = nil
local function ToggleSpinbot(state)
    spinbotEnabled = state
    if state then
        if spinbotConnection then return end
        spinbotConnection = RunService.Heartbeat:Connect(function()
            if not spinbotEnabled then return end
            local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                pcall(function() hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(spinSpeed), 0) end)
            end
        end)
    else
        if spinbotConnection then spinbotConnection:Disconnect(); spinbotConnection = nil end
    end
end

local fakeDashEnabled = false
local DashDistance = 8
local DashCooldown = 0.25
local LastDashTime = 0
local dashLoop = nil

local fakeLagEnabled = false
local LagDistance = 15
local LagCooldown = 1.0
local LagDistanceVariacoes = {12, 15, 18}
local forceIndex = 1
local LastLagTime = 0
local lagLoop = nil

-- Misc: Freeze, Hitbox, Audio
local freezePlayer = false
local hitboxExpanded = false
local hitboxSize = 15
local hitboxTransparency = 0.7
local hitboxOriginais = {}
local legitAtivo = false
local legitTam = 6
local originaisLegit = {}
local visualPartsLegit = {}

local audioEnhancerEnabled = false
local hitSound = Instance.new("Sound", workspace)
hitSound.SoundId = "rbxassetid://9120386403"
hitSound.Volume = 1
hitSound.Name = "FluxoHitSound"
local lastHitHealths = {}

local function expandirUpperTorso(char)
    local part = char:FindFirstChild("UpperTorso")
    if part and part:IsA("BasePart") then
        if not hitboxOriginais[part] then
            hitboxOriginais[part] = {
                Size = part.Size,
                Transparency = part.Transparency,
                Material = part.Material,
                Color = part.Color,
                CanCollide = part.CanCollide,
                Massless = part.Massless,
            }
        end
        part.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
        part.Transparency = hitboxTransparency
        part.Material = Enum.Material.Neon
        part.Color = Color3.fromRGB(255, 0, 0)
        part.CanCollide = false
        part.Massless = true
    end
end

local function criarVisualPart(torso)
    if visualPartsLegit[torso] then return end
    local ok, visual = pcall(function() return torso:Clone() end)
    if not ok or not visual then return end
    visual.Size = torso.Size
    visual.CFrame = torso.CFrame
    visual.Anchored = false
    visual.CanCollide = false
    visual.Transparency = 0
    visual.Parent = torso.Parent
    local weld = Instance.new("Weld")
    weld.Part0 = torso
    weld.Part1 = visual
    weld.Parent = visual
    visualPartsLegit[torso] = visual
end

local function expandirTronco(char)
    local torso = char:FindFirstChild("UpperTorso")
    if torso and torso:IsA("BasePart") then
        if not originaisLegit[torso] then
            originaisLegit[torso] = {
                Size = torso.Size,
                Transparency = torso.Transparency
            }
            criarVisualPart(torso)
        end
        torso.Size = Vector3.new(legitTam, legitTam, legitTam)
        torso.Transparency = 1
    end
end

task.spawn(function()
    while task.wait(0.5) do
        if audioEnhancerEnabled then
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj:IsA("Sound") then
                    local soundId = tostring(obj.SoundId or ""):lower()
                    if soundId:find("gun") or soundId:find("shoot") or soundId:find("fire") 
                       or soundId:find("weapon") or soundId:find("rifle") or soundId:find("pistol") then
                        obj.Volume = math.clamp((obj.Volume or 1) * 0.3, 0, 0.3)
                    end
                end
            end
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LP and player.Character then
                    local humanoid = player.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        for _, sound in pairs(humanoid:GetChildren()) do
                            if sound:IsA("Sound") and string.lower(sound.Name):find("running") then
                                sound.Volume = math.min((sound.Volume or 1) * 1.8, 1.5)
                            end
                        end
                    end
                end
            end
            SoundService.AmbientReverb = Enum.ReverbType.NoReverb
        end
    end
end)

task.spawn(function()
    while task.wait(0.1) do
        if audioEnhancerEnabled then
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LP and player.Character then
                    local humanoid = player.Character:FindFirstChild("Humanoid")
                    if humanoid and humanoid.Health > 0 then
                        if not lastHitHealths[player] then
                            lastHitHealths[player] = humanoid.Health
                        end
                        if humanoid.Health < lastHitHealths[player] then
                            hitSound:Stop()
                            hitSound:Play()
                        end
                        lastHitHealths[player] = humanoid.Health
                    end
                end
            end
        end
    end
end)

-- Bypass (ocultar menu com cliques)
local bypassEnabled = false
local clickTimes = {}
local shiftTimes = {}
local previousParent = nil
local prevMenuVisible = true
local prevMaximizeVisible = true

local function enableBypass()
    if bypassEnabled then return end
    bypassEnabled = true
    prevMenuVisible = hub.Main.Visible
    prevMaximizeVisible = (hub.Main:FindFirstChild("TopBar") and hub.Main.TopBar:FindFirstChild("Maximize")) and hub.Main.TopBar.Maximize.Visible or true
    previousParent = hub.Main.Parent
    hub.Main.Parent = nil
    if hub.OpenBtn then hub.OpenBtn.Visible = false end
    hub:Notify("Bypass","Menu oculto. Clique 5x ou Shift 5x para restaurar.",3)
end

local function disableBypass()
    if not bypassEnabled then return end
    bypassEnabled = false
    if previousParent then hub.Main.Parent = previousParent; previousParent = nil end
    hub.Main.Visible = prevMenuVisible
    if hub.Main:FindFirstChild("TopBar") and hub.Main.TopBar:FindFirstChild("Maximize") then
        hub.Main.TopBar.Maximize.Visible = prevMaximizeVisible
    end
    hub:Notify("Bypass","Menu restaurado.",2)
end

UserInputService.InputBegan:Connect(function(input, processed)
    if not bypassEnabled then return end
    local focus = UserInputService:GetFocusedTextBox()
    if focus and focus ~= "" then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        table.insert(clickTimes, tick())
        while clickTimes[1] and tick() - clickTimes[1] > 1.2 do table.remove(clickTimes, 1) end
        if #clickTimes >= 5 then clickTimes = {}; disableBypass() end
    end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        local kc = input.KeyCode
        if kc == Enum.KeyCode.LeftShift or kc == Enum.KeyCode.RightShift then
            table.insert(shiftTimes, tick())
            while shiftTimes[1] and tick() - shiftTimes[1] > 1.2 do table.remove(shiftTimes, 1) end
            if #shiftTimes >= 5 then shiftTimes = {}; disableBypass() end
        end
    end
end)

-- BHOP
local bhopConn = nil
local function toggleBhop(state)
    if state then
        if bhopConn then return end
        bhopConn = RunService.Heartbeat:Connect(function()
            local char = LP.Character
            if not char then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not hum then return end
            local mv = hum.MoveDirection
            local onGround = hum.FloorMaterial ~= Enum.Material.Air
            if onGround and mv.Magnitude > 0.2 then hum.Jump = true end
        end)
    else
        if bhopConn then bhopConn:Disconnect(); bhopConn = nil end
    end
end

-- Copy clothes
local selectedCopyTarget = nil
local function copyClothes()
    if not selectedCopyTarget then hub:Notify("Copiar Roupas","Nenhum jogador selecionado.",2.5) return end
    local targetChar = selectedCopyTarget.Character
    if not targetChar then hub:Notify("Copiar Roupas","Jogador sem character.",2.5) return end
    local myChar = LP.Character
    if not myChar then hub:Notify("Copiar Roupas","Seu character não está pronto.",2.5) return end
    local applied = false
    pcall(function()
        for _, v in ipairs(myChar:GetChildren()) do if v:IsA("Shirt") or v:IsA("Pants") then v:Destroy() end end
        local shirt = targetChar:FindFirstChildOfClass("Shirt") or targetChar:FindFirstChild("Shirt")
        local pants = targetChar:FindFirstChildOfClass("Pants") or targetChar:FindFirstChild("Pants")
        if shirt and shirt.ShirtTemplate and shirt.ShirtTemplate ~= "" then
            local ns = Instance.new("Shirt"); ns.ShirtTemplate = shirt.ShirtTemplate; ns.Parent = myChar; applied = true
        end
        if pants and pants.PantsTemplate and pants.PantsTemplate ~= "" then
            local np = Instance.new("Pants"); np.PantsTemplate = pants.PantsTemplate; np.Parent = myChar; applied = true
        end
        if not applied and selectedCopyTarget.UserId then
            local ok, desc = pcall(function() return Players:GetHumanoidDescriptionFromUserId(selectedCopyTarget.UserId) end)
            if ok and desc then
                local hum = myChar:FindFirstChildOfClass("Humanoid")
                if hum then hum:ApplyDescription(desc); applied = true end
            end
        end
    end)
    if applied then hub:Notify("Copiar Roupas","Roupas copiadas!",2.5) else hub:Notify("Copiar Roupas","Falha ao copiar.",2.5) end
end

-- Main loops for hitbox, freeze, etc.
RunService.RenderStepped:Connect(function()
    if freezePlayer then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LP and plr.Character then
                local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
                if hrp then pcall(function() hrp.Anchored = true end) end
            end
        end
    end
    if hitboxExpanded then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LP and plr.Character then pcall(expandirUpperTorso, plr.Character) end
        end
    end
    if legitAtivo then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LP and plr.Character then pcall(expandirTronco, plr.Character) end
        end
    end
end)

-- Fake Dash/Lag loops
local function startDashLoop()
    if dashLoop then return end
    dashLoop = RunService.Heartbeat:Connect(function()
        if fakeDashEnabled and tick() - LastDashTime > DashCooldown then
            local char = LP.Character
            if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") then
                local hrp = char.HumanoidRootPart
                local hum = char.Humanoid
                if hum.MoveDirection.Magnitude > 0 then
                    local dir = hrp.CFrame.LookVector
                    local rayParams = RaycastParams.new()
                    rayParams.FilterDescendantsInstances = {char}
                    rayParams.FilterType = Enum.RaycastFilterType.Blacklist
                    local rayResult = workspace:Raycast(hrp.Position, dir * DashDistance, rayParams)
                    if not rayResult then
                        hrp.CFrame = hrp.CFrame + dir * DashDistance
                    end
                    LastDashTime = tick()
                end
            end
        end
    end)
end

local function startLagLoop()
    if lagLoop then return end
    lagLoop = RunService.Heartbeat:Connect(function()
        if fakeLagEnabled and tick() - LastLagTime > LagCooldown then
            local char = LP.Character
            if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") then
                local hrp = char.HumanoidRootPart
                local hum = char.Humanoid
                if hum.MoveDirection.Magnitude > 0 then
                    forceIndex = (forceIndex % #LagDistanceVariacoes) + 1
                    local dist = LagDistanceVariacoes[forceIndex]
                    local direction = hrp.CFrame.LookVector
                    local rayParams = RaycastParams.new()
                    rayParams.FilterDescendantsInstances = {char}
                    rayParams.FilterType = Enum.RaycastFilterType.Blacklist
                    local rayResult = workspace:Raycast(hrp.Position, direction * dist, rayParams)
                    if not rayResult or (rayResult and not rayResult.Instance.CanCollide) then
                        hrp.CFrame = hrp.CFrame + direction * dist
                    end
                    LastLagTime = tick()
                end
            end
        end
    end)
end

----------------------------------------------------------------
--                     CONSTRUÇÃO DAS ABAS (UI)
----------------------------------------------------------------
local hub = setmetatable({}, Onyx):Init()
hub.NotificationsEnabled = true

-- Aba Home
local home = hub:AddTab("Home", nil)
do
    local c1 = home:AddCard("Active Features")
    c1:AddInfo("Features", "Aimbot • ESP • Movement • Head Size")
    c1:AddInfo("Status", "Carregado ✓")
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

-- Aba Aimbots
local aimTab = hub:AddTab("Aimbots", nil)
do
    local c = aimTab:AddCard("Aimbot")
    c:AddToggle("Aimbot", false, function(s) aimbotEnabled = s; updateFOVVisual() end)
    c:AddSlider("FOV", 50, 400, FOVRadius, function(v) FOVRadius = v; updateFOVVisual() end)
    c:AddToggle("FOV Visível", true, function(s) fovVisible = s; updateFOVVisual() end)
    c:AddToggle("Kill Check", true, function(s) killCheckEnabled = s end)
    c:AddToggle("Wall Check", true, function(s) wallCheckEnabled = s end)
    c:AddSlider("Head Offset", 0, 5, HeadOffset, function(v) HeadOffset = v end)

    local bodyCard = aimTab:AddCard("Body Shot Progressivo")
    bodyCard:AddToggle("Ativar", false, function(s)
        bodyShotModeEnabled = s
        if not s then for _, p in ipairs(Players:GetPlayers()) do shotCounts[p] = 0 end end
    end)
    bodyCard:AddSlider("Tiros no peito antes de focar cabeça", 1, 10, bodyShotThreshold, function(v) bodyShotThreshold = v end)

    local rapidCard = aimTab:AddCard("RapidFire")
    rapidCard:AddToggle("RapidFire (segurar mouse/touch)", false, function(s) rapidFireEnabled = s end)
    rapidCard:AddSlider("Delay (ms)", 10, 200, rapidFireDelay*1000, function(v) rapidFireDelay = v/1000 end)

    local holdCard = aimTab:AddCard("Hold to Aim")
    holdCard:AddToggle("Ativar", false, function(s) holdToAimEnabled = s end)
    holdCard:AddKeybind("Tecla (PC)", Enum.KeyCode.E, function(key) aimKey = key end)
    if IS_MOBILE then
        holdCard:AddButton("Botão mobile (Toque aqui)", function()
            mobileAimDown = true
            task.wait(0.1)
            mobileAimDown = false
            hub:Notify("Mobile Aim","Toque para simular pressão",1.5)
        end)
    end
end

-- Aba Visual
local visTab = hub:AddTab("Visual", nil)
do
    local c = visTab:AddCard("Highlight")
    c:AddToggle("Player Highlight", true, function(s)
        highlightEnabled = s
        if s then
            for _, plr in ipairs(Players:GetPlayers()) do if plr ~= LP and plr.Character then createHighlightForCharacter(plr.Character, plr) end end
        else
            destroyAllHighlights()
        end
    end)
    local bill = visTab:AddCard("Billboard ESP")
    bill:AddToggle("Ativar", false, function(s)
        billboardEnabled = s
        if s then
            for _, plr in ipairs(Players:GetPlayers()) do if plr ~= LP and plr.Character then createBillboardForCharacter(plr.Character, plr) end end
        else
            destroyAllBillboards()
        end
    end)
    local draw = visTab:AddCard("Drawing ESP")
    draw:AddToggle("ESP (Box 2D)", false, function(s)
        espEnabled = s
        if drawingSupported and espEnabled then
            for _, plr in ipairs(Players:GetPlayers()) do if plr ~= LP then createESPObjectsForPlayer(plr) end end
        else
            for plr,_ in pairs(ESPS) do removeESPObjects(plr) end
        end
    end)
    draw:AddToggle("Tracers", false, function(s) tracersEnabled = s end)
    draw:AddToggle("Distance", false, function(s) distanceEnabled = s end)
    draw:AddToggle("Rainbow", false, function(s) rainbowESPEnabled = s end)
    draw:AddToggle("Box 3D", false, function(s) box3dEnabled = s end)
    draw:AddTextbox("Cor (R,G,B)", "64,156,255", function(txt)
        local r,g,b = txt:match("(%d+)%s*,%s*(%d+)%s*,%s*(%d+)")
        if r and g and b then espColor = Color3.fromRGB(tonumber(r), tonumber(g), tonumber(b)) end
    end)
end

-- Aba Movement
local movTab = hub:AddTab("Movement", nil)
do
    local spin = movTab:AddCard("Spinbot")
    spin:AddToggle("Spinbot", false, ToggleSpinbot)
    spin:AddSlider("Velocidade (deg/frame)", 1, 100, spinSpeed, function(v) spinSpeed = v end)

    local dash = movTab:AddCard("Fake Dash")
    dash:AddToggle("Ativar", false, function(s)
        fakeDashEnabled = s
        if s then startDashLoop() elseif dashLoop then dashLoop:Disconnect(); dashLoop = nil end
    end)
    dash:AddSlider("Distância (m)", 1, 30, DashDistance, function(v) DashDistance = v end)
    dash:AddSlider("Cooldown (ms)", 50, 1000, DashCooldown*1000, function(v) DashCooldown = v/1000 end)

    local lag = movTab:AddCard("Fake Lag")
    lag:AddToggle("Ativar", false, function(s)
        fakeLagEnabled = s
        if s then startLagLoop() elseif lagLoop then lagLoop:Disconnect(); lagLoop = nil end
    end)
    lag:AddSlider("Distância base", 5, 50, LagDistance, function(v) LagDistance = v end)
    lag:AddSlider("Cooldown (ms)", 100, 2000, LagCooldown*1000, function(v) LagCooldown = v/1000 end)
    lag:AddTextbox("Variações (separadas por vírgula)", "12,15,18", function(txt)
        local arr = {}
        for num in txt:gmatch("(%d+)") do table.insert(arr, tonumber(num)) end
        if #arr > 0 then LagDistanceVariacoes = arr end
    end)
end

-- Aba Misc (agora com todas as funções)
local miscTab = hub:AddTab("Misc", nil)
do
    -- Freeze
    miscTab:AddCard("Freeze"):AddToggle("Congelar Players", false, function(s) freezePlayer = s end)

    -- Hitbox 100hs
    local hb = miscTab:AddCard("Hitbox 100hs")
    hb:AddToggle("Ativar", false, function(s)
        hitboxExpanded = s
        if not s then
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LP and player.Character then
                    local part = player.Character:FindFirstChild("UpperTorso")
                    if part and hitboxOriginais[part] then
                        local o = hitboxOriginais[part]
                        pcall(function()
                            part.Size = o.Size; part.Transparency = o.Transparency; part.Material = o.Material
                            part.Color = o.Color; part.CanCollide = o.CanCollide; part.Massless = o.Massless
                        end)
                        hitboxOriginais[part] = nil
                    end
                end
            end
        end
    end)
    hb:AddSlider("Tamanho", 5, 50, hitboxSize, function(v) hitboxSize = v end)
    hb:AddSlider("Transparência (%)", 0, 100, hitboxTransparency*100, function(v) hitboxTransparency = v/100 end)

    -- Hitbox LEGIT
    local legit = miscTab:AddCard("Hitbox LEGIT")
    legit:AddToggle("Ativar", false, function(s)
        legitAtivo = s
        if not s then
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= LP and plr.Character then
                    local torso = plr.Character:FindFirstChild("UpperTorso")
                    if torso and originaisLegit[torso] then
                        pcall(function()
                            torso.Size = originaisLegit[torso].Size
                            torso.Transparency = originaisLegit[torso].Transparency
                        end)
                        if visualPartsLegit[torso] then pcall(function() visualPartsLegit[torso]:Destroy() end) end
                        originaisLegit[torso] = nil
                    end
                end
            end
        end
    end)
    legit:AddSlider("Tamanho LEGIT", 4, 15, legitTam, function(v) legitTam = v end)

    -- Áudio
    local audio = miscTab:AddCard("Áudio")
    audio:AddToggle("Melhorar Áudio (reduz tiros, aumenta passos)", false, function(s) audioEnhancerEnabled = s end)
    audio:AddToggle("Hit Sound", false, function(s) if s then audioEnhancerEnabled = true else audioEnhancerEnabled = false end end)

    -- HEAD SIZE (adicionado)
    local headCard = miscTab:AddCard("Head Size (próprio personagem)")
    headCard:AddToggle("Ativar Head Size", false, function(s)
        headSizeEnabled = s
        if s then
            startHeadSizeLoop()
            applyHeadSize()
        else
            local char = LP.Character
            if char and char:FindFirstChild("Head") then
                pcall(function()
                    char.Head.Size = Vector3.new(1, 1, 1)
                    char.Head.Transparency = 0
                end)
            end
        end
    end)
    headCard:AddSlider("Tamanho da cabeça", 1, 5, headSizeValue, function(v)
        headSizeValue = v
        if headSizeEnabled then applyHeadSize() end
    end)
    headCard:AddSlider("Transparência da cabeça (%)", 0, 100, headTransparency*100, function(v)
        headTransparency = v / 100
        if headSizeEnabled then applyHeadSize() end
    end)
end

-- Aba Config (Amigos, Tema, Save/Load, Unload)
local cfgTab = hub:AddTab("Config", nil)
do
    local friendCard = cfgTab:AddCard("Amigos (não mirar)")
    local listFrame = nil
    local function rebuildFriendEntries()
        if not listFrame then return end
        for _,c in ipairs(listFrame:GetChildren()) do if not c:IsA("UIListLayout") then c:Destroy() end end
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LP then
                local row = Instance.new("Frame")
                row.Size = UDim2.new(1, -12, 0, 28)
                row.BackgroundColor3 = THEME.SurfaceAlt
                row.Parent = listFrame
                corner(row, CORNER_SM)
                local lbl = Instance.new("TextLabel", row)
                lbl.Size = UDim2.new(0.65, 0, 1, 0)
                lbl.Position = UDim2.new(0, 8, 0, 0)
                lbl.BackgroundTransparency = 1
                lbl.Font = Enum.Font.Gotham
                lbl.Text = p.Name
                lbl.TextColor3 = THEME.Text
                lbl.TextSize = 14
                lbl.TextXAlignment = Enum.TextXAlignment.Left
                local btn = Instance.new("TextButton", row)
                btn.Size = UDim2.new(0, 80, 0, 20)
                btn.Position = UDim2.new(1, -92, 0.5, -10)
                btn.BackgroundColor3 = (friendSet[p.UserId] and THEME.Accent) or Color3.fromRGB(48,42,58)
                btn.Font = Enum.Font.GothamBold
                btn.Text = (friendSet[p.UserId] and "Amigo") or "Marcar"
                btn.TextColor3 = Color3.fromRGB(255,255,255)
                btn.AutoButtonColor = true
                corner(btn, CORNER_SM)
                btn.MouseButton1Click:Connect(function()
                    friendSet[p.UserId] = not friendSet[p.UserId] and true or nil
                    btn.BackgroundColor3 = (friendSet[p.UserId] and THEME.Accent) or Color3.fromRGB(48,42,58)
                    btn.Text = (friendSet[p.UserId] and "Amigo") or "Marcar"
                end)
            end
        end
    end
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -12, 0, 200)
    scroll.BackgroundTransparency = 1
    scroll.ScrollBarThickness = 6
    scroll.Active = true
    scroll.ScrollingEnabled = true
    scroll.Parent = friendCard.Frame
    listFrame = scroll
    local layout = Instance.new("UIListLayout", scroll)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 6)
    rebuildFriendEntries()
    Players.PlayerAdded:Connect(rebuildFriendEntries)
    Players.PlayerRemoving:Connect(rebuildFriendEntries)
    friendCard:AddButton("Limpar Amigos", function()
        friendSet = {}
        rebuildFriendEntries()
    end)

    -- Temas
    local themeCard = cfgTab:AddCard("Tema do Menu")
    local themeNames = {"CLS Pink","Midnight","Lurs Style","Onyx Blue","Emerald","Sunset"}
    themeCard:AddDropdown("Escolher tema", themeNames, function(v)
        hub:ApplyTheme(v)
        hub:Notify("Tema", "Aplicado: "..v, 2)
    end)

    -- Save/Load
    local saveCard = cfgTab:AddCard("Salvar / Carregar")
    local CONFIG_FILE = "FluxoVip_Config.json"
    local function hasFS() return type(writefile)=="function" and type(readfile)=="function" and type(isfile)=="function" end
    local function snapshotConfig()
        local data = {
            aimbotEnabled = aimbotEnabled,
            FOVRadius = FOVRadius,
            fovVisible = fovVisible,
            killCheckEnabled = killCheckEnabled,
            wallCheckEnabled = wallCheckEnabled,
            HeadOffset = HeadOffset,
            bodyShotModeEnabled = bodyShotModeEnabled,
            bodyShotThreshold = bodyShotThreshold,
            rapidFireEnabled = rapidFireEnabled,
            rapidFireDelay = rapidFireDelay,
            holdToAimEnabled = holdToAimEnabled,
            aimKey = aimKey.Name,
            highlightEnabled = highlightEnabled,
            billboardEnabled = billboardEnabled,
            espEnabled = espEnabled,
            tracersEnabled = tracersEnabled,
            distanceEnabled = distanceEnabled,
            rainbowESPEnabled = rainbowESPEnabled,
            box3dEnabled = box3dEnabled,
            espColor = {espColor.R, espColor.G, espColor.B},
            spinbotEnabled = spinbotEnabled,
            spinSpeed = spinSpeed,
            fakeDashEnabled = fakeDashEnabled,
            DashDistance = DashDistance,
            DashCooldown = DashCooldown,
            fakeLagEnabled = fakeLagEnabled,
            LagDistance = LagDistance,
            LagCooldown = LagCooldown,
            LagDistanceVariacoes = LagDistanceVariacoes,
            freezePlayer = freezePlayer,
            hitboxExpanded = hitboxExpanded,
            hitboxSize = hitboxSize,
            hitboxTransparency = hitboxTransparency,
            legitAtivo = legitAtivo,
            legitTam = legitTam,
            audioEnhancerEnabled = audioEnhancerEnabled,
            friendSet = {},
            NotificationsEnabled = hub.NotificationsEnabled,
            headSizeEnabled = headSizeEnabled,
            headSizeValue = headSizeValue,
            headTransparency = headTransparency,
        }
        for uid,_ in pairs(friendSet) do table.insert(data.friendSet, uid) end
        return data
    end
    local function applyConfig(data)
        if type(data) ~= "table" then return end
        aimbotEnabled = data.aimbotEnabled or false
        FOVRadius = data.FOVRadius or 200
        fovVisible = data.fovVisible == nil and true or data.fovVisible
        killCheckEnabled = data.killCheckEnabled == nil and true or data.killCheckEnabled
        wallCheckEnabled = data.wallCheckEnabled == nil and true or data.wallCheckEnabled
        HeadOffset = data.HeadOffset or 1
        bodyShotModeEnabled = data.bodyShotModeEnabled or false
        bodyShotThreshold = data.bodyShotThreshold or 2
        rapidFireEnabled = data.rapidFireEnabled or false
        rapidFireDelay = data.rapidFireDelay or 0.06
        holdToAimEnabled = data.holdToAimEnabled or false
        if data.aimKey then aimKey = Enum.KeyCode[data.aimKey] or Enum.KeyCode.E end
        highlightEnabled = data.highlightEnabled == nil and true or data.highlightEnabled
        billboardEnabled = data.billboardEnabled or false
        espEnabled = data.espEnabled or false
        tracersEnabled = data.tracersEnabled or false
        distanceEnabled = data.distanceEnabled or false
        rainbowESPEnabled = data.rainbowESPEnabled or false
        box3dEnabled = data.box3dEnabled or false
        if data.espColor then espColor = Color3.new(data.espColor[1], data.espColor[2], data.espColor[3]) end
        spinbotEnabled = data.spinbotEnabled or false; ToggleSpinbot(spinbotEnabled)
        spinSpeed = data.spinSpeed or 50
        fakeDashEnabled = data.fakeDashEnabled or false
        DashDistance = data.DashDistance or 8
        DashCooldown = data.DashCooldown or 0.25
        fakeLagEnabled = data.fakeLagEnabled or false
        LagDistance = data.LagDistance or 15
        LagCooldown = data.LagCooldown or 1.0
        if data.LagDistanceVariacoes then LagDistanceVariacoes = data.LagDistanceVariacoes end
        freezePlayer = data.freezePlayer or false
        hitboxExpanded = data.hitboxExpanded or false
        hitboxSize = data.hitboxSize or 15
        hitboxTransparency = data.hitboxTransparency or 0.7
        legitAtivo = data.legitAtivo or false
        legitTam = data.legitTam or 6
        audioEnhancerEnabled = data.audioEnhancerEnabled or false
        if data.friendSet then
            friendSet = {}
            for _,uid in ipairs(data.friendSet) do friendSet[tonumber(uid)] = true end
        end
        if data.NotificationsEnabled ~= nil then hub.NotificationsEnabled = data.NotificationsEnabled end
        headSizeEnabled = data.headSizeEnabled or false
        headSizeValue = data.headSizeValue or 1
        headTransparency = data.headTransparency or 0
        if headSizeEnabled then
            startHeadSizeLoop()
            applyHeadSize()
        else
            local char = LP.Character
            if char and char:FindFirstChild("Head") then
                pcall(function()
                    char.Head.Size = Vector3.new(1,1,1)
                    char.Head.Transparency = 0
                end)
            end
        end
        rebuildFriendEntries()
        updateFOVVisual()
        if highlightEnabled then
            for _, plr in ipairs(Players:GetPlayers()) do if plr ~= LP and plr.Character then createHighlightForCharacter(plr.Character, plr) end end
        end
        if billboardEnabled then
            for _, plr in ipairs(Players:GetPlayers()) do if plr ~= LP and plr.Character then createBillboardForCharacter(plr.Character, plr) end end
        end
        if espEnabled and drawingSupported then
            for _, plr in ipairs(Players:GetPlayers()) do if plr ~= LP then createESPObjectsForPlayer(plr) end end
        end
    end
    saveCard:AddButton("Salvar Configuração", function()
        if not hasFS() then hub:Notify("Config", "Executor sem suporte a arquivos") return end
        local ok, json = pcall(function() return HttpService:JSONEncode(snapshotConfig()) end)
        if ok then pcall(function() writefile(CONFIG_FILE, json) end); hub:Notify("Config", "Salvo") end
    end)
    saveCard:AddButton("Carregar Configuração", function()
        if not hasFS() then hub:Notify("Config", "Executor sem suporte") return end
        if not isfile(CONFIG_FILE) then hub:Notify("Config", "Nenhum arquivo") return end
        local ok, raw = pcall(readfile, CONFIG_FILE)
        if ok then
            local okd, data = pcall(HttpService.JSONDecode, HttpService, raw)
            if okd then applyConfig(data); hub:Notify("Config", "Carregado") end
        end
    end)
    saveCard:AddButton("Apagar Config", function()
        if hasFS() and isfile(CONFIG_FILE) then pcall(delfile, CONFIG_FILE); hub:Notify("Config", "Apagado") end
    end)

    -- Unload
    local unloadCard = cfgTab:AddCard("Menu")
    unloadCard:AddButton("Unload (remover GUI)", function()
        if hub.Gui then hub.Gui:Destroy() end
        if drawingFOV then pcall(function() drawingFOV:Remove() end) end
        for plr,_ in pairs(ESPS) do removeESPObjects(plr) end
        if dashLoop then dashLoop:Disconnect() end
        if lagLoop then lagLoop:Disconnect() end
        if spinbotConnection then spinbotConnection:Disconnect() end
        if bhopConn then bhopConn:Disconnect() end
        if headSizeConnection then headSizeConnection:Disconnect() end
        hitboxOriginais = {}; originaisLegit = {}; visualPartsLegit = {}
        hub = nil
    end)
end

-- Aba Outros (Bypass, BHOP, Copiar roupas, Server Hop, Rejoin)
local outTab = hub:AddTab("Outros", nil)
do
    outTab:AddCard("Bypass"):AddToggle("Bypass (ocultar menu)", false, function(s)
        if s then enableBypass() else disableBypass() end
    end)

    outTab:AddCard("BHOP"):AddToggle("BHOP", false, toggleBhop)

    local copyCard = outTab:AddCard("Copiar Roupas")
    local selectedLabel = copyCard:AddInfo("Jogador selecionado", "Nenhum")
    local function updateSelectList()
        for _,c in ipairs(copyCard.Frame:GetDescendants()) do if c:IsA("TextButton") and c.Name == "PlayerSelect" then c:Destroy() end end
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LP then
                local btn = Instance.new("TextButton")
                btn.Name = "PlayerSelect"
                btn.Size = UDim2.new(1, -24, 0, 28)
                btn.BackgroundColor3 = THEME.SurfaceAlt
                btn.Font = Enum.Font.Gotham
                btn.Text = p.Name
                btn.TextColor3 = THEME.Text
                btn.AutoButtonColor = true
                corner(btn, CORNER_SM)
                btn.Parent = copyCard.Frame
                btn.MouseButton1Click:Connect(function()
                    selectedCopyTarget = p
                    selectedLabel.Text = p.Name
                    hub:Notify("Selecionado", p.Name, 1.5)
                end)
            end
        end
    end
    copyCard:AddButton("Atualizar lista de players", updateSelectList)
    copyCard:AddButton("Copiar roupas", copyClothes)
    updateSelectList()

    local servCard = outTab:AddCard("Servidor")
    servCard:AddButton("Server Hop", function()
        pcall(function()
            local res = game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100")
            local data = HttpService:JSONDecode(res)
            local servers = {}
            if data and data.data then
                for _,sv in ipairs(data.data) do
                    if sv.playing < sv.maxPlayers and sv.id ~= game.JobId then table.insert(servers, sv.id) end
                end
            end
            if #servers > 0 then TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1,#servers)], LP) end
        end)
    end)
    servCard:AddButton("Rejoin", function() TeleportService:Teleport(game.PlaceId, LP) end)
end

hub:Notify("FLUXOVIP", "Menu carregado com sucesso! Head Size e todas as funções estão disponíveis.", 4)
