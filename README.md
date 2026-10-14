--[[
    💎 SkyWars Season 3 - Painel Admin (Cliente)
    Requer o script do servidor rodando em ServerScriptService.
--]]

local Players            = game:GetService("Players")
local RunService         = game:GetService("RunService")
local UserInputService   = game:GetService("UserInputService")
local TweenService       = game:GetService("TweenService")
local ReplicatedStorage  = game:GetService("ReplicatedStorage")
local CoreGui            = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer

-- ============================================================
-- REMOTES (aguarda o servidor criar)
-- ============================================================
local RemotesFolder = ReplicatedStorage:WaitForChild("SkyWarsS3_Remotes", 30)
if not RemotesFolder then
    warn("[SkyWars S3] Servidor não inicializou os remotes. Verifique o ServerScriptService.")
    return
end

local Remote_GetStats    = RemotesFolder:WaitForChild("GetStats")
local Remote_AddGems     = RemotesFolder:WaitForChild("AddGems")
local Remote_AddXP       = RemotesFolder:WaitForChild("AddXP")
local Remote_SetAutoFarm = RemotesFolder:WaitForChild("SetAutoFarm")

-- ============================================================
-- ESTADO
-- ============================================================
local State = {
    AutoGems = false,
    AutoXP   = false,
    Gems     = 0,
    XP       = 0,
    Level    = 1,
}

-- ============================================================
-- HELPERS
-- ============================================================
local function getHiddenUI()
    if gethui then return gethui() end
    return CoreGui
end

local function readRealStats()
    local ls = LocalPlayer:FindFirstChild("leaderstats")
    if ls then
        local g = ls:FindFirstChild("Gems")
        local x = ls:FindFirstChild("XP")
        local l = ls:FindFirstChild("Level")
        State.Gems  = g and g.Value or State.Gems
        State.XP    = x and x.Value or State.XP
        State.Level = l and l.Value or State.Level
    end
end

-- ============================================================
-- UI
-- ============================================================
local function createUI()
    local existing = getHiddenUI():FindFirstChild("SkyWarsS3_AdminUI")
    if existing then existing:Destroy() end

    local gui = Instance.new("ScreenGui")
    gui.Name = "SkyWarsS3_AdminUI"
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Parent = getHiddenUI()

    -- ===== JANELA PRINCIPAL =====
    local main = Instance.new("Frame")
    main.Name = "Main"
    main.Size = UDim2.new(0, 340, 0, 340)
    main.Position = UDim2.new(0.5, -170, 0.5, -170)
    main.BackgroundColor3 = Color3.fromRGB(18, 16, 30)
    main.BackgroundTransparency = 0.06
    main.BorderSizePixel = 0
    main.Active = true
    main.Parent = gui

    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 20)
    mainCorner.Parent = main

    local mainStroke = Instance.new("UIStroke")
    mainStroke.Color = Color3.fromRGB(140, 90, 255)
    mainStroke.Thickness = 1.5
    mainStroke.Transparency = 0.35
    mainStroke.Parent = main

    local bgGrad = Instance.new("UIGradient")
    bgGrad.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(32, 24, 58)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(14, 12, 26)),
    }
    bgGrad.Rotation = 130
    bgGrad.Parent = main

    -- Sombra
    local shadow = Instance.new("ImageLabel")
    shadow.Image = "rbxassetid://6014261993"
    shadow.ImageColor3 = Color3.new(0, 0, 0)
    shadow.ImageTransparency = 0.55
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(49, 49, 450, 450)
    shadow.Size = UDim2.new(1, 40, 1, 40)
    shadow.Position = UDim2.new(0, -20, 0, -20)
    shadow.BackgroundTransparency = 1
    shadow.ZIndex = 0
    shadow.Parent = main

    -- ===== TOP BAR =====
    local topBar = Instance.new("Frame")
    topBar.Size = UDim2.new(1, 0, 0, 48)
    topBar.BackgroundTransparency = 1
    topBar.Parent = main

    local title = Instance.new("TextLabel")
    title.Text = "💎  SkyWars S3 • Painel Admin"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 16
    title.TextColor3 = Color3.fromRGB(235, 225, 255)
    title.BackgroundTransparency = 1
    title.Size = UDim2.new(1, -100, 1, 0)
    title.Position = UDim2.new(0, 18, 0, 0)
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = topBar

    local function makeTopBtn(text, normal, hover, xOffset)
        local b = Instance.new("TextButton")
        b.Text = text
        b.Font = Enum.Font.GothamBold
        b.TextSize = 16
        b.TextColor3 = Color3.fromRGB(220, 210, 240)
        b.BackgroundColor3 = normal
        b.BorderSizePixel = 0
        b.Size = UDim2.new(0, 30, 0, 30)
        b.Position = UDim2.new(1, xOffset, 0, 9)
        b.AutoButtonColor = false
        b.Parent = topBar

        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(1, 0)
        c.Parent = b

        b.MouseEnter:Connect(function()
            TweenService:Create(b, TweenInfo.new(0.15), {BackgroundColor3 = hover}):Play()
        end)
        b.MouseLeave:Connect(function()
            TweenService:Create(b, TweenInfo.new(0.15), {BackgroundColor3 = normal}):Play()
        end)
        return b
    end

    local minBtn   = makeTopBtn("–", Color3.fromRGB(45, 40, 70),  Color3.fromRGB(75, 65, 115), -70)
    local closeBtn = makeTopBtn("✕", Color3.fromRGB(70, 30, 40),  Color3.fromRGB(125, 45, 60), -36)

    -- ===== PAINEL DE STATS =====
    local statsPanel = Instance.new("Frame")
    statsPanel.Size = UDim2.new(1, -32, 0, 84)
    statsPanel.Position = UDim2.new(0, 16, 0, 56)
    statsPanel.BackgroundColor3 = Color3.fromRGB(28, 24, 46)
    statsPanel.BackgroundTransparency = 0.15
    statsPanel.BorderSizePixel = 0
    statsPanel.Parent = main

    local spc = Instance.new("UICorner")
    spc.CornerRadius = UDim.new(0, 14)
    spc.Parent = statsPanel

    local function makeStatRow(yPos, icon, color)
        local iconL = Instance.new("TextLabel")
        iconL.Text = icon
        iconL.Font = Enum.Font.GothamBold
        iconL.TextSize = 20
        iconL.TextColor3 = color
        iconL.BackgroundTransparency = 1
        iconL.Size = UDim2.new(0, 40, 0, 34)
        iconL.Position = UDim2.new(0, 14, 0, yPos)
        iconL.TextXAlignment = Enum.TextXAlignment.Left
        iconL.Parent = statsPanel

        local value = Instance.new("TextLabel")
        value.Text = "0"
        value.Font = Enum.Font.GothamBold
        value.TextSize = 18
        value.TextColor3 = Color3.fromRGB(235, 230, 255)
        value.BackgroundTransparency = 1
        value.Size = UDim2.new(1, -70, 0, 34)
        value.Position = UDim2.new(0, 54, 0, yPos)
        value.TextXAlignment = Enum.TextXAlignment.Left
        value.Parent = statsPanel

        return value
    end

    local gemsValue = makeStatRow(8,  "💎", Color3.fromRGB(130, 210, 255))
    local xpValue   = makeStatRow(44, "⭐", Color3.fromRGB(255, 220, 130))

    -- ===== BOTÃO AUTO GEMS =====
    local function makeActionButton(text, yPos, colorNormal, colorHover, colorActive)
        local btn = Instance.new("TextButton")
        btn.Text = text
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 15
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.BackgroundColor3 = colorNormal
        btn.BorderSizePixel = 0
        btn.Size = UDim2.new(1, -32, 0, 58)
        btn.Position = UDim2.new(0, 16, 0, yPos)
        btn.AutoButtonColor = false
        btn.Parent = main

        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, 14)
        c.Parent = btn

        local g = Instance.new("UIGradient")
        g.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, colorNormal),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(
                math.max(colorNormal.R * 255 - 40, 0) / 255,
                math.max(colorNormal.G * 255 - 40, 0) / 255,
                math.max(colorNormal.B * 255 - 40, 0) / 255
            )),
        }
        g.Rotation = 90
        g.Parent = btn

        local stroke = Instance.new("UIStroke")
        stroke.Color = Color3.fromRGB(255, 255, 255)
        stroke.Thickness = 1
        stroke.Transparency = 0.7
        stroke.Parent = btn

        btn.MouseEnter:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.18), {BackgroundColor3 = colorHover}):Play()
        end)
        btn.MouseLeave:Connect(function()
            local target = State[text:find("Gemas") and "AutoGems" or "AutoXP"] and colorActive or colorNormal
            TweenService:Create(btn, TweenInfo.new(0.18), {BackgroundColor3 = target}):Play()
        end)

        return btn
    end

    local gemsBtn = makeActionButton("💎  Auto Gems  •  OFF", 152,
        Color3.fromRGB(110, 70, 230), Color3.fromRGB(140, 95, 255), Color3.fromRGB(40, 200, 120))

    local xpBtn = makeActionButton("⭐  Auto XP  •  OFF", 220,
        Color3.fromRGB(220, 150, 40), Color3.fromRGB(255, 180, 70), Color3.fromRGB(40, 200, 120))

    -- Footer
    local footer = Instance.new("TextLabel")
    footer.Text = "Alterações reais • salvas no servidor"
    footer.Font = Enum.Font.Gotham
    footer.TextSize = 10
    footer.TextColor3 = Color3.fromRGB(150, 140, 180)
    footer.BackgroundTransparency = 1
    footer.Size = UDim2.new(1, -32, 0, 20)
    footer.Position = UDim2.new(0, 16, 1, -26)
    footer.TextXAlignment = Enum.TextXAlignment.Center
    footer.Parent = main

    -- ========================================================
    -- LÓGICA DOS BOTÕES
    -- ========================================================
    local function setBtnVisual(btn, active, activeText, inactiveText, colorNormal, colorActive)
        if active then
            btn.Text = activeText
            TweenService:Create(btn, TweenInfo.new(0.25), {BackgroundColor3 = colorActive}):Play()
        else
            btn.Text = inactiveText
            TweenService:Create(btn, TweenInfo.new(0.25), {BackgroundColor3 = colorNormal}):Play()
        end
    end

    gemsBtn.MouseButton1Click:Connect(function()
        State.AutoGems = not State.AutoGems
        if State.AutoGems then State.AutoXP = false; Remote_SetAutoFarm:FireServer("gems") else
            if not State.AutoXP then Remote_SetAutoFarm:FireServer("off") end
        end
        setBtnVisual(gemsBtn, State.AutoGems,
            "💎  Auto Gems  •  ON",  "💎  Auto Gems  •  OFF",
            Color3.fromRGB(110, 70, 230), Color3.fromRGB(40, 200, 120))
        setBtnVisual(xpBtn, State.AutoXP,
            "⭐  Auto XP  •  ON",    "⭐  Auto XP  •  OFF",
            Color3.fromRGB(220, 150, 40), Color3.fromRGB(40, 200, 120))
    end)

    xpBtn.MouseButton1Click:Connect(function()
        State.AutoXP = not State.AutoXP
        if State.AutoXP then State.AutoGems = false; Remote_SetAutoFarm:FireServer("xp") else
            if not State.AutoGems then Remote_SetAutoFarm:FireServer("off") end
        end
        setBtnVisual(xpBtn, State.AutoXP,
            "⭐  Auto XP  •  ON",    "⭐  Auto XP  •  OFF",
            Color3.fromRGB(220, 150, 40), Color3.fromRGB(40, 200, 120))
        setBtnVisual(gemsBtn, State.AutoGems,
            "💎  Auto Gems  •  ON",  "💎  Auto Gems  •  OFF",
            Color3.fromRGB(110, 70, 230), Color3.fromRGB(40, 200, 120))
    end)

    -- ========================================================
    -- MINIMIZAR / FECHAR / ARRASTAR
    -- ========================================================
    local minimized = false
    local originalSize = main.Size

    minBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        if minimized then
            originalSize = main.Size
            TweenService:Create(main, TweenInfo.new(0.25), {Size = UDim2.new(0, 340, 0, 48)}):Play()
            statsPanel.Visible = false; gemsBtn.Visible = false; xpBtn.Visible = false; footer.Visible = false
            minBtn.Text = "+"
        else
            TweenService:Create(main, TweenInfo.new(0.25), {Size = originalSize}):Play()
            task.delay(0.15, function()
                statsPanel.Visible = true; gemsBtn.Visible = true; xpBtn.Visible = true; footer.Visible = true
            end)
            minBtn.Text = "–"
        end
    end)

    closeBtn.MouseButton1Click:Connect(function()
        State.AutoGems = false; State.AutoXP = false
        Remote_SetAutoFarm:FireServer("off")
        TweenService:Create(main, TweenInfo.new(0.2), {
            Size = UDim2.new(0, 0, 0, 0), BackgroundTransparency = 1
        }):Play()
        task.wait(0.22)
        gui:Destroy()
    end)

    -- Arrasto (mouse + toque)
    local dragging, dragStart, startPos
    topBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = main.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            main.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    -- ========================================================
    -- ATUALIZAÇÃO EM TEMPO REAL (leaderstats)
    -- ========================================================
    task.spawn(function()
        while gui.Parent do
            readRealStats()

            local newGemText = "💎  " .. tostring(State.Gems)
            local newXpText  = "⭐  " .. tostring(State.XP) .. "   (Lv " .. tostring(State.Level) .. ")"

            if gemsValue.Text ~= newGemText then
                gemsValue.Text = newGemText
                TweenService:Create(gemsValue, TweenInfo.new(0.1), {TextColor3 = Color3.fromRGB(255, 240, 160)}):Play()
                task.delay(0.15, function()
                    TweenService:Create(gemsValue, TweenInfo.new(0.2), {TextColor3 = Color3.fromRGB(235, 230, 255)}):Play()
                end)
            end

            if xpValue.Text ~= newXpText then
                xpValue.Text = newXpText
                TweenService:Create(xpValue, TweenInfo.new(0.1), {TextColor3 = Color3.fromRGB(255, 240, 160)}):Play()
                task.delay(0.15, function()
                    TweenService:Create(xpValue, TweenInfo.new(0.2), {TextColor3 = Color3.fromRGB(235, 230, 255)}):Play()
                end)
            end

            task.wait(0.15)
        end
    end)

    return gui
end

createUI()
print("[💎 SkyWars S3] Painel admin (cliente) carregado.")
