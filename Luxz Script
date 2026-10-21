--[[
    LUXZ SCRIPT — +1 SAQUE PARA FORJAR
    Interface completa em arquivo único para Delta Executor
    Versão: 1.0.0
    Autor: LUXZ
    Todas as funções server-side aparecem como INDISPONÍVEL.
    Funções locais (otimização, UI) são reais e funcionais.
]]

--=============================================================
-- SERVIÇOS
--=============================================================
local Players           = game:GetService("Players")
local UserInputService  = game:GetService("UserInputService")
local TweenService      = game:GetService("TweenService")
local Lighting          = game:GetService("Lighting")
local RunService        = game:GetService("RunService")
local CoreGui           = game:GetService("CoreGui")
local StarterGui        = game:GetService("StarterGui")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

--=============================================================
-- CONFIG
--=============================================================
local CONFIG = {
    Version   = "1.0.0",
    GameName  = "+1 Saque para Forjar",
    SaveFile  = "LuxzScript_Config.json",
    Theme = {
        Background   = Color3.fromRGB(10, 8, 15),
        Surface      = Color3.fromRGB(18, 14, 28),
        SurfaceHover = Color3.fromRGB(26, 20, 40),
        Purple       = Color3.fromRGB(140, 80, 255),
        PurpleGlow   = Color3.fromRGB(180, 120, 255),
        PurpleDim    = Color3.fromRGB(70, 40, 120),
        Text         = Color3.fromRGB(235, 230, 245),
        TextDim      = Color3.fromRGB(140, 130, 160),
        Green        = Color3.fromRGB(80, 220, 140),
        Red          = Color3.fromRGB(240, 80, 100),
        Yellow       = Color3.fromRGB(250, 200, 80),
        Border       = Color3.fromRGB(50, 35, 80),
    },
}

local T = CONFIG.Theme

--=============================================================
-- SISTEMA DE SALVAMENTO (opcional, protegido)
--=============================================================
local Save = {
    data = {},
    hasFS = (typeof(writefile) == "function" and typeof(readfile) == "function" and typeof(isfile) == "function"),
    functionAvailable = false,
}

function Save:Load()
    if not self.hasFS then return end
    local ok, res = pcall(function()
        if isfile(CONFIG.SaveFile) then
            return game:GetService("HttpService"):JSONDecode(readfile(CONFIG.SaveFile))
        end
    end)
    if ok and type(res) == "table" then
        self.data = res
    end
end

function Save:Set(key, value)
    self.data[key] = value
    if self.hasFS then
        pcall(function()
            writefile(CONFIG.SaveFile, game:GetService("HttpService"):JSONEncode(self.data))
        end)
    end
end

function Save:Get(key, default)
    if self.data[key] ~= nil then return self.data[key] end
    return default
end

Save:Load()

--=============================================================
-- ESTADO DAS FUNÇÕES
-- "available" = funcional no cliente | "unavailable" = requer server/API
--=============================================================
local Status = {
    -- Farm (todas server-side)
    InfiniteMoney    = "unavailable",
    AutoDungeonLoot  = "unavailable",
    AutoSuperLoot    = "unavailable",
    AutoUpgrade      = "unavailable",
    AutoTrain        = "unavailable",
    AutoRebirth      = "unavailable",
    -- Forjar
    AutoForgeWeapon  = "unavailable",
    AutoForgeHat     = "unavailable",
    AutoForgeArmor   = "unavailable",
    InstantForge     = "unavailable",
    AutoEquipBest    = "unavailable",
    SellOldGear      = "unavailable",
    -- Encantamento
    AutoEnchantSword = "unavailable",
    AutoEnchantHat   = "unavailable",
    AutoEnchantArmor = "unavailable",
    -- Minérios
    InfiniteOres     = "unavailable",
    PickOres         = "unavailable",
    -- Raça
    AutoRollRace     = "unavailable",
    RollUntil        = "unavailable",
    -- Extras
    InstantKill      = "unavailable",
    GodMode          = "unavailable",
    AutoClaimRewards = "unavailable",
    DisableNotifications = "available",  -- controla só UI local
    -- Otimização (funcionais no cliente)
    RemoveFog        = "available",
    ReduceParticles  = "available",
    DisableShadows   = "available",
    LowGraphics      = "available",
    ReduceVisualEffects = "available",
    SimplifyMaterials = "available",
    HideDecorativeObjects = "available",
    ReduceAnimations = "available",
    FPSBoost         = "available",
    LowPerformanceMode = "available",
    PerformanceMonitor = "available",
    RestoreGraphics  = "available",
    ResetOptimization = "available",
    Fullscreen       = "unavailable", -- Roblox não expõe API
    UIScale          = "available",
    CompactUI        = "available",
    MobileMode       = "available",
}

--=============================================================
-- GERENCIADOR DE ESTADO / BACKUP GRÁFICO
--=============================================================
local State = {
    toggles = {},          -- key -> valor bool
    uiScale = 1,
    compact = false,
    notificationsDisabled = false,
    graphicsBackup = nil,  -- backup dos valores originais
    perfMonitor = nil,
}

function State:Toggle(key, val, callback)
    self.toggles[key] = val
    Save:Set("toggle_" .. key, val)
    if callback then callback(val) end
end

function State:IsOn(key)
    return self.toggles[key] == true
end

-- Salva backup dos valores gráficos originais
function State:BackupGraphics()
    if self.graphicsBackup then return end
    self.graphicsBackup = {
        FogEnd        = Lighting.FogEnd,
        FogStart      = Lighting.FogStart,
        FogColor      = Lighting.FogColor,
        GlobalShadows = Lighting.GlobalShadows,
        Ambient       = Lighting.Ambient,
        OutdoorAmbient= Lighting.OutdoorAmbient,
        Brightness    = Lighting.Brightness,
    }
end

--=============================================================
-- HELPERS DE UI
--=============================================================
local function newInstance(class, props, children)
    local obj = Instance.new(class)
    for k, v in pairs(props or {}) do obj[k] = v end
    for _, c in ipairs(children or {}) do c.Parent = obj end
    return obj
end

local function corner(parent, radius)
    return newInstance("UICorner", {CornerRadius = UDim.new(0, radius or 8), Parent = parent})
end

local function stroke(parent, color, thickness)
    return newInstance("UIStroke", {
        Color = color or T.Purple,
        Thickness = thickness or 1,
        Transparency = 0.3,
        Parent = parent,
    })
end

local function padding(parent, t, b, l, r)
    return newInstance("UIPadding", {
        PaddingTop = UDim.new(0, t or 0),
        PaddingBottom = UDim.new(0, b or 0),
        PaddingLeft = UDim.new(0, l or 0),
        PaddingRight = UDim.new(0, r or 0),
        Parent = parent,
    })
end

--=============================================================
-- COMPONENTE: TOGGLE
--=============================================================
local function createToggle(parent, labelText, key, callback)
    local status = Status[key] or "unavailable"
    local row = newInstance("Frame", {
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = T.Surface,
        BackgroundTransparency = 0.4,
        BorderSizePixel = 0,
        Parent = parent,
    })
    corner(row, 8)
    padding(row, 0, 0, 10, 10)

    local label = newInstance("TextLabel", {
        Size = UDim2.new(1, -120, 1, 0),
        BackgroundTransparency = 1,
        Text = labelText,
        TextColor3 = (status == "unavailable") and T.TextDim or T.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.Gotham,
        TextSize = 13,
        Parent = row,
    })

    -- Badge indisponível
    if status == "unavailable" then
        local badge = newInstance("TextLabel", {
            Size = UDim2.new(0, 66, 0, 18),
            Position = UDim2.new(1, -118, 0.5, -9),
            BackgroundColor3 = T.Red,
            BackgroundTransparency = 0.82,
            Text = "INDISP.",
            TextColor3 = T.Red,
            TextSize = 10,
            Font = Enum.Font.GothamBold,
            BorderSizePixel = 0,
            Parent = row,
        })
        corner(badge, 4)
    end

    local toggleBg = newInstance("Frame", {
        Size = UDim2.new(0, 44, 0, 22),
        Position = UDim2.new(1, -50, 0.5, -11),
        BackgroundColor3 = Color3.fromRGB(50, 45, 60),
        BorderSizePixel = 0,
        Parent = row,
    })
    corner(toggleBg, 999)

    local knob = newInstance("Frame", {
        Size = UDim2.new(0, 16, 0, 16),
        Position = UDim2.new(0, 3, 0.5, -8),
        BackgroundColor3 = T.TextDim,
        BorderSizePixel = 0,
        Parent = toggleBg,
    })
    corner(knob, 999)

    local enabled = Save:Get("toggle_" .. key, false) == true
    State.toggles[key] = enabled

    local function render()
        local info = TweenInfo.new(0.15)
        if enabled then
            TweenService:Create(toggleBg, info, {BackgroundColor3 = T.Purple}):Play()
            TweenService:Create(knob, info, {
                Position = UDim2.new(1, -19, 0.5, -8),
                BackgroundColor3 = T.Text,
            }):Play()
        else
            TweenService:Create(toggleBg, info, {BackgroundColor3 = Color3.fromRGB(50, 45, 60)}):Play()
            TweenService:Create(knob, info, {
                Position = UDim2.new(0, 3, 0.5, -8),
                BackgroundColor3 = T.TextDim,
            }):Play()
        end
    end

    local btn = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "",
        Parent = row,
    })

    if status == "unavailable" then
        btn.MouseButton1Click:Connect(function()
            label.Text = labelText .. "  · indisponível no cliente"
            label.TextColor3 = T.Red
            task.delay(1.8, function()
                if label and label.Parent then
                    label.Text = labelText
                    label.TextColor3 = T.TextDim
                end
            end)
        end)
    else
        btn.MouseButton1Click:Connect(function()
            enabled = not enabled
            render()
            State:Toggle(key, enabled, callback)
        end)
    end

    render()
    return row, {
        Set = function(v) enabled = v; render(); State:Toggle(key, enabled, callback) end,
        Get = function() return enabled end,
    }
end

--=============================================================
-- COMPONENTE: DROPDOWN (vazio, aguardando API)
--=============================================================
local function createDropdown(parent, labelText, placeholder)
    local holder = newInstance("Frame", {
        Size = UDim2.new(1, 0, 0, 60),
        BackgroundColor3 = T.Surface,
        BackgroundTransparency = 0.4,
        BorderSizePixel = 0,
        Parent = parent,
    })
    corner(holder, 8)
    padding(holder, 6, 6, 10, 10)

    newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 0, 18),
        BackgroundTransparency = 1,
        Text = labelText,
        TextColor3 = T.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.GothamBold,
        TextSize = 12,
        Parent = holder,
    })

    local button = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 28),
        Position = UDim2.new(0, 0, 0, 22),
        BackgroundColor3 = T.SurfaceHover,
        Text = placeholder or "Nenhum item disponível",
        TextColor3 = T.TextDim,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        BorderSizePixel = 0,
        Parent = holder,
    })
    corner(button, 6)
    return holder
end

--=============================================================
-- JANELA PRINCIPAL
--=============================================================
local existing = CoreGui:FindFirstChild("LuxzScriptGui") or PlayerGui:FindFirstChild("LuxzScriptGui")
if existing then existing:Destroy() end

local ScreenGui = newInstance("ScreenGui", {
    Name = "LuxzScriptGui",
    ResetOnSpawn = false,
    IgnoreGuiInset = true,
    DisplayOrder = 999,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
})
pcall(function() ScreenGui.Parent = CoreGui end)
if not ScreenGui.Parent then ScreenGui.Parent = PlayerGui end

local Main = newInstance("Frame", {
    Name = "MainFrame",
    Size = UDim2.new(0, 560, 0, 400),
    Position = UDim2.new(0.5, -280, 0.5, -200),
    BackgroundColor3 = T.Background,
    BorderSizePixel = 0,
    Active = true,
    Parent = ScreenGui,
})
corner(Main, 16)
local mainStroke = stroke(Main, T.Purple, 1.5)

-- Barra de título
local TitleBar = newInstance("Frame", {
    Size = UDim2.new(1, 0, 0, 44),
    BackgroundColor3 = T.Surface,
    BorderSizePixel = 0,
    Parent = Main,
})
corner(TitleBar, 16)

local iconFrame = newInstance("Frame", {
    Size = UDim2.new(0, 28, 0, 28),
    Position = UDim2.new(0, 10, 0.5, -14),
    BackgroundColor3 = T.PurpleDim,
    BorderSizePixel = 0,
    Parent = TitleBar,
})
corner(iconFrame, 999)
stroke(iconFrame, T.PurpleGlow, 2)

newInstance("TextLabel", {
    Size = UDim2.new(1, 0, 1, 0),
    BackgroundTransparency = 1,
    Text = "L",
    TextColor3 = T.PurpleGlow,
    TextScaled = true,
    Font = Enum.Font.GothamBlack,
    Parent = iconFrame,
})

newInstance("TextLabel", {
    Size = UDim2.new(1, -140, 1, 0),
    Position = UDim2.new(0, 46, 0, 0),
    BackgroundTransparency = 1,
    Text = "LUXZ SCRIPT",
    TextColor3 = T.PurpleGlow,
    TextXAlignment = Enum.TextXAlignment.Left,
    Font = Enum.Font.GothamBold,
    TextSize = 15,
    Parent = TitleBar,
})

-- Botão fechar
local closeBtn = newInstance("TextButton", {
    Size = UDim2.new(0, 30, 0, 30),
    Position = UDim2.new(1, -38, 0.5, -15),
    BackgroundColor3 = T.Red,
    BackgroundTransparency = 0.8,
    Text = "✕",
    TextColor3 = T.Red,
    TextSize = 16,
    Font = Enum.Font.GothamBold,
    BorderSizePixel = 0,
    Parent = TitleBar,
})
corner(closeBtn, 8)

-- Botão minimizar
local minBtn = newInstance("TextButton", {
    Size = UDim2.new(0, 30, 0, 30),
    Position = UDim2.new(1, -72, 0.5, -15),
    BackgroundColor3 = T.Yellow,
    BackgroundTransparency = 0.8,
    Text = "—",
    TextColor3 = T.Yellow,
    TextSize = 16,
    Font = Enum.Font.GothamBold,
    BorderSizePixel = 0,
    Parent = TitleBar,
})
corner(minBtn, 8)

-- Container do conteúdo
local Content = newInstance("Frame", {
    Size = UDim2.new(1, 0, 1, -44),
    Position = UDim2.new(0, 0, 0, 44),
    BackgroundTransparency = 1,
    Parent = Main,
})

-- Sidebar
local Sidebar = newInstance("Frame", {
    Size = UDim2.new(0, 140, 1, 0),
    BackgroundColor3 = T.Surface,
    BorderSizePixel = 0,
    Parent = Content,
})
padding(Sidebar, 8, 8, 6, 6)

local sidebarLayout = newInstance("UIListLayout", {
    Padding = UDim.new(0, 3),
    SortOrder = Enum.SortOrder.LayoutOrder,
    Parent = Sidebar,
})

-- Container de páginas
local PageContainer = newInstance("Frame", {
    Size = UDim2.new(1, -140, 1, 0),
    Position = UDim2.new(0, 140, 0, 0),
    BackgroundTransparency = 1,
    Parent = Content,
})
padding(PageContainer, 10, 10, 12, 12)

-- Arrastar janela
do
    local dragging, dragStart, startPos
    TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = Main.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    TitleBar.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - dragStart
            Main.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + d.X,
                startPos.Y.Scale, startPos.Y.Offset + d.Y
            )
        end
    end)
end

closeBtn.MouseButton1Click:Connect(function()
    if State.perfMonitor then State.perfMonitor:Disconnect() end
    ScreenGui:Destroy()
end)

local minimized = false
local originalSize = Main.Size
minBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    TweenService:Create(Main, TweenInfo.new(0.2), {
        Size = minimized and UDim2.new(0, 560, 0, 44) or originalSize
    }):Play()
end)

--=============================================================
-- GERENCIADOR DE PÁGINAS
--=============================================================
local Pages = {}
local currentPage = nil
local navButtons = {}

local function newPage(name)
    local scroll = newInstance("ScrollingFrame", {
        Name = name,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 4,
        ScrollBarImageColor3 = T.Purple,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        Visible = false,
        Parent = PageContainer,
    })
    newInstance("UIListLayout", {
        Padding = UDim.new(0, 8),
        SortOrder = Enum.SortOrder.LayoutOrder,
        Parent = scroll,
    })
    padding(scroll, 0, 12, 0, 8)
    return scroll
end

local function showPage(name)
    for n, p in pairs(Pages) do p.Visible = (n == name) end
    for n, b in pairs(navButtons) do
        b.BackgroundColor3 = (n == name) and T.PurpleDim or T.Surface
        b.TextColor3 = (n == name) and T.PurpleGlow or T.Text
    end
    currentPage = name
end

local function addNav(icon, name, order)
    local btn = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = T.Surface,
        Text = "  " .. icon .. "  " .. name,
        TextColor3 = T.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        BorderSizePixel = 0,
        LayoutOrder = order,
        Parent = Sidebar,
    })
    corner(btn, 8)
    btn.MouseButton1Click:Connect(function() showPage(name) end)
    navButtons[name] = btn
end

--=============================================================
-- HELPERS DE CONTEÚDO DE PÁGINA
--=============================================================
local function sectionTitle(parent, text, order)
    local l = newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 0, 22),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = T.PurpleGlow,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.GothamBold,
        TextSize = 13,
        LayoutOrder = order or 0,
        Parent = parent,
    })
    return l
end

local function infoLabel(parent, text, order)
    local l = newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = T.TextDim,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextWrapped = true,
        Font = Enum.Font.Gotham,
        TextSize = 11,
        LayoutOrder = order or 0,
        Parent = parent,
    })
    return l
end

--=============================================================
-- PÁGINA: HOME
--=============================================================
Pages.Home = newPage("Home")
do
    local p = Pages.Home
    -- Logo circular
    local logo = newInstance("Frame", {
        Size = UDim2.new(0, 90, 0, 90),
        BackgroundColor3 = T.PurpleDim,
        BorderSizePixel = 0,
        LayoutOrder = 1,
        Parent = p,
    })
    corner(logo, 999)
    stroke(logo, T.PurpleGlow, 2.5)
    newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "LUXZ",
        TextColor3 = T.PurpleGlow,
        TextScaled = true,
        Font = Enum.Font.GothamBlack,
        Parent = logo,
    })
    newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 0, 26),
        BackgroundTransparency = 1,
        Text = CONFIG.GameName,
        TextColor3 = T.Text,
        Font = Enum.Font.GothamBold,
        TextSize = 16,
        LayoutOrder = 2,
        Parent = p,
    })
    newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 0, 22),
        BackgroundTransparency = 1,
        Text = "● Interface carregada · v" .. CONFIG.Version,
        TextColor3 = T.Green,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        LayoutOrder = 3,
        Parent = p,
    })

    local activeCount = newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 0, 22),
        BackgroundTransparency = 1,
        Text = "Opções ativas: 0",
        TextColor3 = T.TextDim,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        LayoutOrder = 4,
        Parent = p,
    })

    task.spawn(function()
        while activeCount.Parent do
            local n = 0
            for _, v in pairs(State.toggles) do if v then n = n + 1 end end
            activeCount.Text = "Opções ativas: " .. n
            task.wait(1)
        end
    end)

    sectionTitle(p, "INFORMAÇÕES", 5)
    infoLabel(p,
        "Compatibilidade: interface client-side. Funções server-side " ..
        "(Infinite Money, Instant Kill, God Mode, Infinite Ores) " ..
        "aparecem como INDISPONÍVEL — o servidor do jogo não permite alteração pelo cliente.",
        6)

    local resetBtn = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 36),
        BackgroundColor3 = T.PurpleDim,
        Text = "Restaurar configurações padrão",
        TextColor3 = T.Text,
        Font = Enum.Font.GothamBold,
        TextSize = 13,
        BorderSizePixel = 0,
        LayoutOrder = 7,
        Parent = p,
    })
    corner(resetBtn, 8)
    resetBtn.MouseButton1Click:Connect(function()
        for _, v in pairs(State.toggles) do
            if type(v) == "table" and v.Set then v:Set(false) end
        end
        -- Reset gráfico
        if State:IsOn("ResetOptimization") == false and State.graphicsBackup then
            Lighting.FogEnd = State.graphicsBackup.FogEnd
            Lighting.FogStart = State.graphicsBackup.FogStart
            Lighting.GlobalShadows = State.graphicsBackup.GlobalShadows
        end
        Save.data = {}
        State.toggles = {}
        activeCount.Text = "Opções ativas: 0 (restaurado)"
    end)
end

--=============================================================
-- PÁGINA: FARM
--=============================================================
Pages.Farm = newPage("Farm")
do
    local p = Pages.Farm
    sectionTitle(p, "FARMING", 1)
    infoLabel(p, "Todas as funções de farm exigem acesso ao servidor do jogo. Permanecem indisponíveis.", 2)

    local list = {
        {"Infinite Money", "InfiniteMoney"},
        {"Auto Dungeon Loot", "AutoDungeonLoot"},
        {"Auto Super Loot", "AutoSuperLoot"},
        {"Auto Upgrade", "AutoUpgrade"},
        {"Auto Train", "AutoTrain"},
        {"Auto Rebirth", "AutoRebirth"},
    }
    for i, item in ipairs(list) do
        local f = createToggle(p, item[1], item[2])
        f.LayoutOrder = 10 + i
    end
end

--=============================================================
-- PÁGINA: FORJAR
--=============================================================
Pages.Forjar = newPage("Forjar")
do
    local p = Pages.Forjar
    sectionTitle(p, "FORJAR — AUTOMAÇÃO", 1)
    local list = {
        {"Auto Forge Weapon", "AutoForgeWeapon"},
        {"Auto Forge Hat", "AutoForgeHat"},
        {"Auto Forge Armor", "AutoForgeArmor"},
        {"Instant Forge", "InstantForge"},
        {"Auto Equip Best", "AutoEquipBest"},
        {"Sell Old Gear", "SellOldGear"},
    }
    for i, item in ipairs(list) do
        local f = createToggle(p, item[1], item[2])
        f.LayoutOrder = 10 + i
    end

    sectionTitle(p, "INTERFACE DE FORJA", 30)
    infoLabel(p, "Os painéis abaixo ficam vazios até que as APIs reais do jogo estejam disponíveis.", 31)

    createDropdown(p, "Armas disponíveis", "Aguardando API do jogo").LayoutOrder = 40
    createDropdown(p, "Minérios e materiais", "Aguardando API do jogo").LayoutOrder = 41

    -- 4 slots de materiais
    local slotsHolder = newInstance("Frame", {
        Size = UDim2.new(1, 0, 0, 60),
        BackgroundColor3 = T.Surface,
        BackgroundTransparency = 0.5,
        BorderSizePixel = 0,
        LayoutOrder = 50,
        Parent = p,
    })
    corner(slotsHolder, 8)
    newInstance("UIListLayout", {
        FillDirection = Enum.FillDirection.Horizontal,
        Padding = UDim.new(0, 6),
        SortOrder = Enum.SortOrder.LayoutOrder,
        Parent = slotsHolder,
    })
    padding(slotsHolder, 8, 8, 8, 8)
    for i = 1, 4 do
        local slot = newInstance("Frame", {
            Size = UDim2.new(0, 44, 0, 44),
            BackgroundColor3 = T.SurfaceHover,
            BorderSizePixel = 0,
            LayoutOrder = i,
            Parent = slotsHolder,
        })
        corner(slot, 8)
        stroke(slot, T.PurpleDim, 1)
    end

    newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 0, 20),
        BackgroundTransparency = 1,
        Text = "Multiplicador: 0x  |  Requisitos: —",
        TextColor3 = T.TextDim,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        LayoutOrder = 51,
        Parent = p,
    })

    local forgeBtn = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 40),
        BackgroundColor3 = T.PurpleDim,
        Text = "FORJAR (indisponível)",
        TextColor3 = T.TextDim,
        Font = Enum.Font.GothamBold,
        TextSize = 14,
        BorderSizePixel = 0,
        LayoutOrder = 52,
        Parent = p,
    })
    corner(forgeBtn, 8)
    forgeBtn.MouseButton1Click:Connect(function()
        forgeBtn.Text = "FORJAR — aguardando API"
        task.delay(1.5, function()
            if forgeBtn and forgeBtn.Parent then
                forgeBtn.Text = "FORJAR (indisponível)"
            end
        end)
    end)

    local clearBtn = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 32),
        BackgroundColor3 = T.SurfaceHover,
        Text = "LIMPAR ESPAÇOS",
        TextColor3 = T.Text,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        BorderSizePixel = 0,
        LayoutOrder = 53,
        Parent = p,
    })
    corner(clearBtn, 8)
end

--=============================================================
-- PÁGINA: ENCANTAMENTO
--=============================================================
Pages.Encantamento = newPage("Encantamento")
do
    local p = Pages.Encantamento
    sectionTitle(p, "ENCANTAMENTO", 1)
    local list = {
        {"Auto Enchant Sword", "AutoEnchantSword"},
        {"Auto Enchant Hat", "AutoEnchantHat"},
        {"Auto Enchant Armor", "AutoEnchantArmor"},
    }
    for i, item in ipairs(list) do
        local f = createToggle(p, item[1], item[2])
        f.LayoutOrder = 10 + i
    end

    createDropdown(p, "Selecionar equipamento", "Aguardando API do jogo").LayoutOrder = 30
    infoLabel(p, "Comparação de atributos antes/depois, limite de encantamento e confirmação de gasto ficarão ativos quando a API estiver acessível.", 31)
end

--=============================================================
-- PÁGINA: MINÉRIOS
--=============================================================
Pages.Minerios = newPage("Minérios")
do
    local p = Pages.Minerios
    sectionTitle(p, "MINÉRIOS", 1)

    local f1 = createToggle(p, "Infinite Ores", "InfiniteOres"); f1.LayoutOrder = 10
    local f2 = createToggle(p, "Pick Ores", "PickOres"); f2.LayoutOrder = 11

    createDropdown(p, "Selecionar minério", "Aguardando API do jogo").LayoutOrder = 20
    infoLabel(p, "Lista de minérios, quantidades, filtros por raridade e nome aparecerão quando o inventário for acessível de forma legítima.", 21)

    local refresh = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 32),
        BackgroundColor3 = T.SurfaceHover,
        Text = "Atualizar lista",
        TextColor3 = T.Text,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        BorderSizePixel = 0,
        LayoutOrder = 30,
        Parent = p,
    })
    corner(refresh, 8)
end

--=============================================================
-- PÁGINA: RAÇA
--=============================================================
Pages.Raca = newPage("Raça")
do
    local p = Pages.Raca
    sectionTitle(p, "RAÇA", 1)
    local f1 = createToggle(p, "Auto Roll Race", "AutoRollRace"); f1.LayoutOrder = 10
    local f2 = createToggle(p, "Roll Until", "RollUntil"); f2.LayoutOrder = 11

    infoLabel(p, "Raça atual: —  |  Raça desejada: —  |  Tentativas: 0", 20)
    createDropdown(p, "Raça desejada", "Aguardando API do jogo").LayoutOrder = 30

    local stop = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 32),
        BackgroundColor3 = T.Red,
        BackgroundTransparency = 0.7,
        Text = "Interromper processo",
        TextColor3 = T.Red,
        Font = Enum.Font.GothamBold,
        TextSize = 12,
        BorderSizePixel = 0,
        LayoutOrder = 40,
        Parent = p,
    })
    corner(stop, 8)
end

--=============================================================
-- PÁGINA: EXTRAS
--=============================================================
Pages.Extras = newPage("Extras")
do
    local p = Pages.Extras
    sectionTitle(p, "EXTRAS", 1)

    local list = {
        {"Instant Kill", "InstantKill"},
        {"God Mode", "GodMode"},
        {"Auto Claim Rewards", "AutoClaimRewards"},
        {"Disable Notifications (UI)", "DisableNotifications"},
    }
    for i, item in ipairs(list) do
        local f = createToggle(p, item[1], item[2], function(val)
            if item[2] == "DisableNotifications" then
                State.notificationsDisabled = val
            end
        end)
        f.LayoutOrder = 10 + i
    end

    infoLabel(p, "Instant Kill e God Mode dependem de alteração server-side e não podem ser executados a partir do cliente.", 30)
end

--=============================================================
-- PÁGINA: OTIMIZAÇÃO (FUNCIONAL NO CLIENTE)
--=============================================================
Pages.Otimizacao = newPage("Otimização")
do
    local p = Pages.Otimizacao

    -- ---------- VISUAL ----------
    sectionTitle(p, "VISUAL", 1)

    -- Remove Fog
    createToggle(p, "Remove Fog", "RemoveFog", function(on)
        State:BackupGraphics()
        if on then
            Lighting.FogEnd = 100000
            Lighting.FogStart = 100000
        else
            if State.graphicsBackup then
                Lighting.FogEnd = State.graphicsBackup.FogEnd
                Lighting.FogStart = State.graphicsBackup.FogStart
            end
        end
    end).LayoutOrder = 10

    -- Reduce Particles
    createToggle(p, "Reduce Particles", "ReduceParticles", function(on)
        for _, d in ipairs(Lighting:GetDescendants()) do
            if d:IsA("ParticleEmitter") then
                if on then
                    if not d:GetAttribute("LuxzOriginalRate") then
                        d:SetAttribute("LuxzOriginalRate", d.Rate)
                    end
                    d.Rate = math.floor(d.Rate * 0.15)
                else
                    local orig = d:GetAttribute("LuxzOriginalRate")
                    if orig then d.Rate = orig end
                end
            end
        end
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") then
                if on then
                    if not obj:GetAttribute("LuxzOriginalRate") then
                        obj:SetAttribute("LuxzOriginalRate", obj.Rate)
                    end
                    obj.Rate = math.floor(obj.Rate * 0.15)
                else
                    local orig = obj:GetAttribute("LuxzOriginalRate")
                    if orig then obj.Rate = orig end
                end
            end
        end
    end).LayoutOrder = 11

    -- Disable Shadows
    createToggle(p, "Disable Shadows", "DisableShadows", function(on)
        State:BackupGraphics()
        Lighting.GlobalShadows = not on
    end).LayoutOrder = 12

    -- Low Graphics
    createToggle(p, "Low Graphics", "LowGraphics", function(on)
        State:BackupGraphics()
        if on then
            Lighting.GlobalShadows = false
            Lighting.FogEnd = 100000
            Lighting.Brightness = math.min(Lighting.Brightness, 1)
            Lighting.EnvironmentDiffuseScale = 0
            Lighting.EnvironmentSpecularScale = 0
        else
            if State.graphicsBackup then
                Lighting.GlobalShadows = State.graphicsBackup.GlobalShadows
                Lighting.FogEnd = State.graphicsBackup.FogEnd
                Lighting.Brightness = State.graphicsBackup.Brightness
            end
        end
    end).LayoutOrder = 13

    -- Reduce Visual Effects
    createToggle(p, "Reduce Visual Effects", "ReduceVisualEffects", function(on)
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then
                if on then
                    if not obj:GetAttribute("LuxzOrigEnabled") then
                        obj:SetAttribute("LuxzOrigEnabled", obj.Enabled)
                    end
                    obj.Enabled = false
                else
                    local orig = obj:GetAttribute("LuxzOrigEnabled")
                    if orig ~= nil then obj.Enabled = orig end
                end
            end
        end
    end).LayoutOrder = 14

    -- Simplify Materials
    createToggle(p, "Simplify Materials", "SimplifyMaterials", function(on)
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") then
                if on then
                    if not obj:GetAttribute("LuxzOrigMaterial") then
                        obj:SetAttribute("LuxzOrigMaterial", obj.Material.Name)
                    end
                    obj.Material = Enum.Material.SmoothPlastic
                else
                    local orig = obj:GetAttribute("LuxzOrigMaterial")
                    if orig then
                        pcall(function() obj.Material = Enum.Material[orig] end)
                    end
                end
            end
        end
    end).LayoutOrder = 15

    -- Hide Decorative Objects
    createToggle(p, "Hide Decorative Objects", "HideDecorativeObjects", function(on)
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") and (obj.Name:lower():find("decor") or obj.Name:lower():find("tree") or obj.Name:lower():find("bush")) then
                if on then
                    if not obj:GetAttribute("LuxzOrigTransparency") then
                        obj:SetAttribute("LuxzOrigTransparency", obj.Transparency)
                    end
                    obj.Transparency = 1
                else
                    local t = obj:GetAttribute("LuxzOrigTransparency")
                    if t then obj.Transparency = t end
                end
            end
        end
    end).LayoutOrder = 16

    -- Reduce Animations
    createToggle(p, "Reduce Animations", "ReduceAnimations", function(on)
        -- Atua apenas em animações cosméticas locais do workspace
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("AnimationController") then
                for _, t in ipairs(obj:GetPlayingAnimationTracks()) do
                    if on then t:AdjustSpeed(0.1) else t:AdjustSpeed(1) end
                end
            end
        end
    end).LayoutOrder = 17

    -- ---------- DESEMPENHO ----------
    sectionTitle(p, "DESEMPENHO", 30)

    createToggle(p, "FPS Boost", "FPSBoost", function(on)
        State:BackupGraphics()
        if on then
            Lighting.GlobalShadows = false
            Lighting.FogEnd = 100000
        else
            if State.graphicsBackup then
                Lighting.GlobalShadows = State.graphicsBackup.GlobalShadows
                Lighting.FogEnd = State.graphicsBackup.FogEnd
            end
        end
    end).LayoutOrder = 31

    createToggle(p, "Low Performance Mode", "LowPerformanceMode", function(on)
        State:BackupGraphics()
        if on then
            Lighting.GlobalShadows = false
            Lighting.FogEnd = 100000
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj:IsA("ParticleEmitter") then
                    if not obj:GetAttribute("LuxzOrigEnabled") then
                        obj:SetAttribute("LuxzOrigEnabled", obj.Enabled)
                    end
                    obj.Enabled = false
                end
            end
        else
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj:IsA("ParticleEmitter") then
                    local o = obj:GetAttribute("LuxzOrigEnabled")
                    if o ~= nil then obj.Enabled = o end
                end
            end
        end
    end).LayoutOrder = 32

    -- Performance Monitor
    local perfLabel = newInstance("TextLabel", {
        Size = UDim2.new(1, 0, 0, 28),
        BackgroundColor3 = T.Surface,
        BackgroundTransparency = 0.5,
        Text = "FPS: —  |  Memória: —",
        TextColor3 = T.Text,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        BorderSizePixel = 0,
        LayoutOrder = 33,
        Parent = p,
    })
    corner(perfLabel, 6)

    createToggle(p, "Performance Monitor", "PerformanceMonitor", function(on)
        if on then
            if State.perfMonitor then State.perfMonitor:Disconnect() end
            local frames, t = 0, 0
            State.perfMonitor = RunService.RenderStepped:Connect(function(dt)
                frames = frames + 1
                t = t + dt
                if t >= 1 then
                    local mem = "—"
                    pcall(function()
                        mem = string.format("%.0f MB", gcinfo() / 1024)
                    end)
                    perfLabel.Text = string.format("FPS: %d  |  Memória: %s", frames, mem)
                    frames, t = 0, 0
                end
            end)
        else
            if State.perfMonitor then
                State.perfMonitor:Disconnect()
                State.perfMonitor = nil
            end
            perfLabel.Text = "FPS: —  |  Memória: —"
        end
    end).LayoutOrder = 34

    -- Restore / Reset
    local restoreBtn = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = T.SurfaceHover,
        Text = "Restore Graphics",
        TextColor3 = T.Text,
        Font = Enum.Font.GothamBold,
        TextSize = 12,
        BorderSizePixel = 0,
        LayoutOrder = 40,
        Parent = p,
    })
    corner(restoreBtn, 8)
    restoreBtn.MouseButton1Click:Connect(function()
        if State.graphicsBackup then
            Lighting.FogEnd        = State.graphicsBackup.FogEnd
            Lighting.FogStart      = State.graphicsBackup.FogStart
            Lighting.GlobalShadows = State.graphicsBackup.GlobalShadows
            Lighting.Ambient       = State.graphicsBackup.Ambient
            Lighting.OutdoorAmbient= State.graphicsBackup.OutdoorAmbient
            Lighting.Brightness    = State.graphicsBackup.Brightness
        end
    end)

    local resetOptBtn = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = T.PurpleDim,
        Text = "Reset Optimization",
        TextColor3 = T.Text,
        Font = Enum.Font.GothamBold,
        TextSize = 12,
        BorderSizePixel = 0,
        LayoutOrder = 41,
        Parent = p,
    })
    corner(resetOptBtn, 8)
    resetOptBtn.MouseButton1Click:Connect(function()
        if State.graphicsBackup then
            Lighting.FogEnd        = State.graphicsBackup.FogEnd
            Lighting.FogStart      = State.graphicsBackup.FogStart
            Lighting.GlobalShadows = State.graphicsBackup.GlobalShadows
            Lighting.Ambient       = State.graphicsBackup.Ambient
            Lighting.OutdoorAmbient= State.graphicsBackup.OutdoorAmbient
            Lighting.Brightness    = State.graphicsBackup.Brightness
        end
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") then
                local r = obj:GetAttribute("LuxzOriginalRate")
                if r then obj.Rate = r end
                local e = obj:GetAttribute("LuxzOrigEnabled")
                if e ~= nil then obj.Enabled = e end
            end
        end
    end)

    -- ---------- TELA ----------
    sectionTitle(p, "TELA", 60)

    local fsBtn = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = T.SurfaceHover,
        Text = "Fullscreen (indisponível ao cliente)",
        TextColor3 = T.TextDim,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        BorderSizePixel = 0,
        LayoutOrder = 61,
        Parent = p,
    })
    corner(fsBtn, 8)

    -- UI Scale slider (simplificado com +/-)
    local scaleHolder = newInstance("Frame", {
        Size = UDim2.new(1, 0, 0, 40),
        BackgroundColor3 = T.Surface,
        BackgroundTransparency = 0.4,
        BorderSizePixel = 0,
        LayoutOrder = 62,
        Parent = p,
    })
    corner(scaleHolder, 8)
    padding(scaleHolder, 4, 4, 10, 10)

    local scaleLabel = newInstance("TextLabel", {
        Size = UDim2.new(0.5, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "UI Scale: 1.00x",
        TextColor3 = T.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        Parent = scaleHolder,
    })

    local function updateScale(s)
        State.uiScale = math.clamp(s, 0.7, 1.5)
        scaleLabel.Text = string.format("UI Scale: %.2fx", State.uiScale)
        local w = 560 * State.uiScale
        local h = 400 * State.uiScale
        Main.Size = UDim2.new(0, w, 0, h)
        Save:Set("uiScale", State.uiScale)
    end

    local minusBtn = newInstance("TextButton", {
        Size = UDim2.new(0, 28, 0, 28),
        Position = UDim2.new(1, -70, 0.5, -14),
        BackgroundColor3 = T.PurpleDim,
        Text = "−",
        TextColor3 = T.Text,
        Font = Enum.Font.GothamBold,
        TextSize = 16,
        BorderSizePixel = 0,
        Parent = scaleHolder,
    })
    corner(minusBtn, 6)

    local plusBtn = newInstance("TextButton", {
        Size = UDim2.new(0, 28, 0, 28),
        Position = UDim2.new(1, -36, 0.5, -14),
        BackgroundColor3 = T.PurpleDim,
        Text = "+",
        TextColor3 = T.Text,
        Font = Enum.Font.GothamBold,
        TextSize = 16,
        BorderSizePixel = 0,
        Parent = scaleHolder,
    })
    corner(plusBtn, 6)

    minusBtn.MouseButton1Click:Connect(function() updateScale(State.uiScale - 0.1) end)
    plusBtn.MouseButton1Click:Connect(function() updateScale(State.uiScale + 0.1) end)

    createToggle(p, "Compact UI", "CompactUI", function(on)
        if on then
            Sidebar.Size = UDim2.new(0, 90, 1, 0)
            PageContainer.Position = UDim2.new(0, 90, 0, 0)
            PageContainer.Size = UDim2.new(1, -90, 1, 0)
        else
            Sidebar.Size = UDim2.new(0, 140, 1, 0)
            PageContainer.Position = UDim2.new(0, 140, 0, 0)
            PageContainer.Size = UDim2.new(1, -140, 1, 0)
        end
    end).LayoutOrder = 63

    createToggle(p, "Mobile Mode", "MobileMode", function(on)
        if on then
            Main.Size = UDim2.new(0, 360, 0, 380)
            Main.Position = UDim2.new(0.5, -180, 0.5, -190)
        end
    end).LayoutOrder = 64

    -- Restaura escala salva
    local savedScale = Save:Get("uiScale", 1)
    if savedScale then updateScale(savedScale) end
end

--=============================================================
-- PÁGINA: SETTINGS
--=============================================================
Pages.Settings = newPage("Settings")
do
    local p = Pages.Settings
    sectionTitle(p, "CONFIGURAÇÕES", 1)

    createToggle(p, "Mostrar anel LUXZ na janela", "ShowRing", function(on)
        mainStroke.Transparency = on and 0.3 or 1
    end).LayoutOrder = 10

    createToggle(p, "Animações da interface", "EnableAnimations", function(on)
        -- Sem efeito real além de desabilitar tweens de feedback
    end).LayoutOrder = 11

    infoLabel(p, string.format(
        "Versão: %s\nJogo: %s\nModo: client-side\nSalvamento: %s",
        CONFIG.Version, CONFIG.GameName, Save.hasFS and "ativo" or "indisponível (executor sem FS)"
    ), 20)

    local resetAll = newInstance("TextButton", {
        Size = UDim2.new(1, 0, 0, 36),
        BackgroundColor3 = T.Red,
        BackgroundTransparency = 0.7,
        Text = "Restaurar TUDO ao padrão",
        TextColor3 = T.Red,
        Font = Enum.Font.GothamBold,
        TextSize = 13,
        BorderSizePixel = 0,
        LayoutOrder = 30,
        Parent = p,
    })
    corner(resetAll, 8)
    resetAll.MouseButton1Click:Connect(function()
        Save.data = {}
        if Save.hasFS then pcall(function() delfile(CONFIG.SaveFile) end) end
        State.toggles = {}
        if State.graphicsBackup then
            Lighting.FogEnd        = State.graphicsBackup.FogEnd
            Lighting.FogStart      = State.graphicsBackup.FogStart
            Lighting.GlobalShadows = State.graphicsBackup.GlobalShadows
            Lighting.Ambient       = State.graphicsBackup.Ambient
            Lighting.OutdoorAmbient= State.graphicsBackup.OutdoorAmbient
            Lighting.Brightness    = State.graphicsBackup.Brightness
        end
        resetAll.Text = "Restaurado! (reabra o script)"
    end)
end

--=============================================================
-- REGISTRO DOS BOTÕES DE NAVEGAÇÃO
--=============================================================
addNav("🏠", "Home", 1)
addNav("💰", "Farm", 2)
addNav("⚒️", "Forjar", 3)
addNav("✨", "Encantamento", 4)
addNav("🪨", "Minérios", 5)
addNav("🧬", "Raça", 6)
addNav("🎁", "Extras", 7)
addNav("⚡", "Otimização", 8)
addNav("⚙️", "Settings", 9)

showPage("Home")

--=============================================================
-- NOTIFICAÇÃO DE CARREGAMENTO
--=============================================================
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "LUXZ SCRIPT",
        Text = "+1 Saque para Forjar · v" .. CONFIG.Version .. " carregado",
        Duration = 4,
    })
end)

print("[LUXZ SCRIPT] Carregado com sucesso. v" .. CONFIG.Version)
