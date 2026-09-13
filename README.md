--[[
    SVEY HUB PREMIUM v61 (CLEAN FIX)
    - 2-этапный Key System
    - Watermark (клик = открыть/закрыть меню)
    - Рабочий Dick (палка + головка + 2 яйца)
    - Silent Aim через metatable-хук
    - Anti-Aim через AlignOrientation
    - Config синхронизирует UI
]]

-- ========== СЕРВИСЫ ==========
local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local TweenService     = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Lighting         = game:GetService("Lighting")
local Camera           = workspace.CurrentCamera
local LocalPlayer      = Players.LocalPlayer
local PlayerGui        = LocalPlayer:WaitForChild("PlayerGui")
local IS_MOBILE        = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled

local GameMode = "MM2"
pcall(function()
    local placeName = game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name
    if placeName:lower():find("флик") or placeName:lower():find("flick") or placeName:lower():find("fps") then
        GameMode = "FPS"
    end
end)

-- ========== ТЕМА ==========
local THEME = {
    BG = Color3.fromRGB(12, 10, 16), BG2 = Color3.fromRGB(15, 13, 20),
    BG3 = Color3.fromRGB(20, 17, 28), BG4 = Color3.fromRGB(28, 24, 38),
    Text = Color3.fromRGB(240, 238, 245), TextDim = Color3.fromRGB(140, 135, 155),
    Accent = Color3.fromRGB(140, 90, 245), Border = Color3.fromRGB(48, 42, 65),
    Murderer = Color3.fromRGB(240, 50, 60), Sheriff = Color3.fromRGB(55, 130, 255),
    Hero = Color3.fromRGB(250, 205, 45), Innocent = Color3.fromRGB(55, 215, 115),
}

local COLOR_PRESETS = {
    {Name="Purple",Color=Color3.fromRGB(185,115,255)},
    {Name="Blue",  Color=Color3.fromRGB(60,120,255)},
    {Name="Red",   Color=Color3.fromRGB(255,60,60)},
    {Name="Green", Color=Color3.fromRGB(100,255,150)},
    {Name="White", Color=Color3.fromRGB(255,255,255)},
    {Name="Yellow",Color=Color3.fromRGB(255,235,60)},
    {Name="Skin",  Color=Color3.fromRGB(255,200,150)},
    {Name="Pink",  Color=Color3.fromRGB(255,105,180)},
    {Name="Cyan",  Color=Color3.fromRGB(0,255,255)},
    {Name="Orange",Color=Color3.fromRGB(255,140,0)},
    {Name="Lime",  Color=Color3.fromRGB(180,255,60)},
    {Name="Black", Color=Color3.fromRGB(20,20,20)},
}

local SKYBOX_PRESETS = {
    {Name="Default", Ambient=Color3.fromRGB(70,70,70), Outdoor=Color3.fromRGB(140,140,140), ClockTime=14, Brightness=2},
    {Name="Custom ID", Ambient=Color3.fromRGB(50,30,70), Outdoor=Color3.fromRGB(120,60,180), ClockTime=0, Brightness=1, CustomId="rbxassetid://15983996673"},
    {Name="Space",  Ambient=Color3.fromRGB(0,0,0), Outdoor=Color3.fromRGB(0,0,0), ClockTime=0, Brightness=0},
}

-- ========== STATE ==========
local State = {
    silentAim=false, silentAimTargetPart="Head", showFov=false, silentAimFOV=150,
    Aimbot=false, AimbotFOV=120, AimbotSmoothness=0.5,
    HitboxExpander=false, HitboxSize=5, HitboxTransparency=0.7,
    AntiAim=false, AntiAimMode="Spin", AntiAimSpeed=50,
    roleEsp=false, espMurder=false, espSheriff=false, espHero=false, espInnocents=false, espAll=false,
    highlightEsp=false, highlightColor=Color3.fromRGB(180, 90, 255),
    skeletonEsp=false, skeletonColor=Color3.fromRGB(255, 255, 255),
    box2D=false, box2DMode="Full", box2DMM2=false, box2DColor=Color3.fromRGB(255, 255, 255),
    box2DFill=false, box2DFillColor=Color3.fromRGB(160, 90, 255),
    box3D=false, box3DColor=Color3.fromRGB(160, 90, 255),
    healthBar=false, distanceEsp=false, distanceColor=Color3.fromRGB(255, 255, 255),
    arrowEsp=false, arrowEspColor=Color3.fromRGB(255, 255, 255),
    snaplines=false, snaplineOrigin="Bottom", snaplineColor=Color3.fromRGB(255, 255, 255),
    aura=false, auraColor=Color3.fromRGB(185, 115, 255),
    selfWings=false, selfWingsColor=Color3.fromRGB(255, 255, 255),
    glassChams=false, glassChamsAll=false, glassChamsColor=Color3.fromRGB(160, 100, 255), ChamsTransparency=0.35,
    killFlash=false, killFlashColor=Color3.fromRGB(185, 90, 255),
    customFog=false, fogDistance=250, fogColor=Color3.fromRGB(140, 100, 220), fogDensity=0.3,
    Hat=false, HatColor=Color3.fromRGB(100, 255, 150), HatSize=1,
    Dick=false, DickSize=1, DickColor=Color3.fromRGB(255, 200, 150),
    BodyMode="None",
    SpeedHack=false, SpeedValue=32, SpeedMethod="Humanoid",
    JumpHack=false, JumpValue=50,
    Fly=false, FlySpeed=100,
    Noclip=false, InfiniteJump=false, bhop=false, AntiVoid=false, AntiFling=false,
    ThirdPerson=false, TP_Distance=15, strafeType="Hybrid",
    FlingMode="Rage", AutoFlingMurderer=false, AutoFlingSheriff=false, AutoFlingHero=false,
    Skybox="Custom ID",
    fovColor=Color3.fromRGB(255, 60, 60), hitboxColor=Color3.fromRGB(185,115,255),
    BlurEnabled=false, StretchEnabled=false, Language="RU",
}

local Effects = {HatModel={}, DickParts={}, SilentAimTarget=nil}
local auraObjects = { parts = {}, emitters = {}, wings = nil, floorRing = nil }
local chamsOriginals = {}
local playerRoles = {}
local originalSheriff = nil
local flingDebounce = false
local cached2D = {}

local Character = LocalPlayer.Character
local Humanoid  = Character and Character:FindFirstChildOfClass("Humanoid")

LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char
    Humanoid  = char:WaitForChild("Humanoid", 10)
    task.wait(0.5)
    -- Восстанавливаем Dick/Hat после респавна
    if State.Dick then
        Effects.DickParts = {}
        applyDick()
    end
    if State.Hat then
        Effects.HatModel = {}
        applyHat()
    end
end)

-- ========== РОЛИ ==========
local function resolveRole(player)
    if not player then return "Innocent" end
    if playerRoles[player.Name] then return playerRoles[player.Name] end
    local char = player.Character
    local bp   = player:FindFirstChild("Backpack")
    if (char and char:FindFirstChild("Knife")) or (bp and bp:FindFirstChild("Knife")) then
        playerRoles[player.Name] = "Murderer"; return "Murderer"
    end
    if (char and char:FindFirstChild("Gun")) or (bp and bp:FindFirstChild("Gun")) then
        if originalSheriff == nil or originalSheriff == player then
            originalSheriff = player; playerRoles[player.Name] = "Sheriff"; return "Sheriff"
        end
        playerRoles[player.Name] = "Hero"; return "Hero"
    end
    return "Innocent"
end

local function getRoleColor(role)
    if role == "Murderer" then return THEME.Murderer end
    if role == "Sheriff"  then return THEME.Sheriff  end
    if role == "Hero"     then return THEME.Hero     end
    return THEME.Innocent
end

local function getMurdererPlayer()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and resolveRole(p) == "Murderer" then return p end
    end
end
local function getSheriffPlayer()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and resolveRole(p) == "Sheriff" then return p end
    end
end
local function getHeroPlayer()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and resolveRole(p) == "Hero" then return p end
    end
end

local function SendNotification(title, text, duration)
    pcall(function()
        game:GetService("StarterGui"):SetCore("SendNotification",
            {Title=title, Text=text, Duration=duration or 3})
    end)
end

-- ========== GUI ==========
pcall(function()
    local old = PlayerGui:FindFirstChild("SVEYHub")
    if old then old:Destroy() end
end)

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SVEYHub"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

local ESPContainer = Instance.new("Folder", ScreenGui)
ESPContainer.Name = "ESPContainer"

-- ========== KEY SYSTEM ==========
local KeyState = {Stage = 0, Valid = false, Gui = nil}

local function showKeyWindow()
    if KeyState.Gui then KeyState.Gui:Destroy() end
    local g = Instance.new("ScreenGui")
    g.Name = "SVEY_KeySystem"; g.ResetOnSpawn = false
    g.IgnoreGuiInset = true; g.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    g.Parent = PlayerGui
    KeyState.Gui = g

    local bg = Instance.new("Frame", g)
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    bg.BackgroundTransparency = 0.5; bg.BorderSizePixel = 0; bg.ZIndex = 500

    local frame = Instance.new("Frame", g)
    frame.Size = UDim2.new(0, 340, 0, 250)
    frame.Position = UDim2.new(0.5, -170, 0.5, -125)
    frame.BackgroundColor3 = THEME.BG; frame.BorderSizePixel = 0
    frame.Active = true; frame.ZIndex = 501
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)
    local fs = Instance.new("UIStroke", frame); fs.Color = THEME.Accent; fs.Thickness = 2

    local logo = Instance.new("Frame", frame)
    logo.Size = UDim2.new(0, 55, 0, 55); logo.Position = UDim2.new(0.5, -27, 0, 18)
    logo.BackgroundColor3 = THEME.Accent; logo.BorderSizePixel = 0; logo.ZIndex = 502
    Instance.new("UICorner", logo).CornerRadius = UDim.new(0, 14)
    local ll = Instance.new("TextLabel", logo)
    ll.Size = UDim2.new(1,0,1,0); ll.BackgroundTransparency = 1
    ll.Text = "S"; ll.TextColor3 = Color3.fromRGB(255,255,255)
    ll.Font = Enum.Font.GothamBlack; ll.TextSize = 30; ll.ZIndex = 503

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1, 0, 0, 24); title.Position = UDim2.new(0, 0, 0, 82)
    title.BackgroundTransparency = 1; title.Text = "SVEY HUB — KEY"
    title.TextColor3 = THEME.Text; title.Font = Enum.Font.GothamBold
    title.TextSize = 15; title.ZIndex = 502

    local info = Instance.new("TextLabel", frame)
    info.Size = UDim2.new(1, -20, 0, 30); info.Position = UDim2.new(0, 10, 0, 108)
    info.BackgroundTransparency = 1; info.Text = "Введи ключ #1"
    info.TextColor3 = THEME.TextDim; info.Font = Enum.Font.Gotham
    info.TextSize = 11; info.ZIndex = 502

    local input = Instance.new("TextBox", frame)
    input.Size = UDim2.new(1, -40, 0, 36); input.Position = UDim2.new(0, 20, 0, 145)
    input.BackgroundColor3 = THEME.BG3; input.BorderSizePixel = 0
    input.PlaceholderText = "Введи ключ..."; input.PlaceholderColor3 = THEME.TextDim
    input.Text = ""; input.TextColor3 = THEME.Text
    input.Font = Enum.Font.Gotham; input.TextSize = 13
    input.ClearTextOnFocus = false; input.ZIndex = 502
    Instance.new("UICorner", input).CornerRadius = UDim.new(0, 8)
    local is = Instance.new("UIStroke", input); is.Color = THEME.Border; is.Thickness = 1

    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, -40, 0, 40); btn.Position = UDim2.new(0, 20, 1, -56)
    btn.BackgroundColor3 = THEME.Accent; btn.Text = "ПРОВЕРИТЬ"
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.GothamBold; btn.TextSize = 14
    btn.AutoButtonColor = false; btn.ZIndex = 502
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

    local function shake()
        local orig = frame.Position
        for i = 1, 3 do
            TweenService:Create(frame, TweenInfo.new(0.05), {Position = orig + UDim2.new(0, 8, 0, 0)}):Play()
            task.wait(0.05)
            TweenService:Create(frame, TweenInfo.new(0.05), {Position = orig - UDim2.new(0, 8, 0, 0)}):Play()
            task.wait(0.05)
        end
        TweenService:Create(frame, TweenInfo.new(0.05), {Position = orig}):Play()
    end

    btn.MouseButton1Click:Connect(function()
        local entered = input.Text:lower():gsub("^%s+", ""):gsub("%s+$", "")
        if KeyState.Stage == 0 then
            if entered == "osloeb" then
                KeyState.Stage = 1; input.Text = ""
                info.Text = "✅ Ключ #1 принят!\nВведи ключ #2"
                info.TextColor3 = Color3.fromRGB(55, 215, 115)
                btn.Text = "ПРОВЕРИТЬ #2"
                SendNotification("SVEY KEY", "Этап 1/2 пройден", 2)
            else
                info.Text = "❌ Неверный ключ #1"
                info.TextColor3 = Color3.fromRGB(240, 50, 60)
                input.Text = ""; shake()
            end
        elseif KeyState.Stage == 1 then
            local clean1 = entered:gsub("%s+", "")
            local clean2 = entered:gsub("%s+", " ")
            if clean1 == "давидгоджо" or clean2 == "давид годжо"
               or clean1 == "davidgojo" or clean2 == "david gojo" then
                KeyState.Stage = 2; KeyState.Valid = true
                info.Text = "✅ ВСЕ КЛЮЧИ ПРИНЯТЫ!"
                info.TextColor3 = Color3.fromRGB(55, 215, 115)
                btn.Text = "ВОЙТИ"
                SendNotification("SVEY KEY", "Этап 2/2 пройден!", 3)
                task.wait(0.6)
                if KeyState.Gui then KeyState.Gui:Destroy() KeyState.Gui = nil end
            else
                info.Text = "❌ Неверный ключ #2"
                info.TextColor3 = Color3.fromRGB(240, 50, 60)
                input.Text = ""; shake()
            end
        end
    end)
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(180,130,255)}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = THEME.Accent}):Play()
    end)
    task.wait(0.2)
    pcall(function() input:CaptureFocus() end)
end

showKeyWindow()

-- ========== WATERMARK (кликабельный — открывает меню) ==========
local Watermark = Instance.new("Frame", ScreenGui)
Watermark.Size = UDim2.new(0, 220, 0, 24)
Watermark.Position = UDim2.new(1, -235, 0, 75)
Watermark.BackgroundColor3 = Color3.fromRGB(12, 10, 16)
Watermark.BackgroundTransparency = 0.15
Watermark.BorderSizePixel = 0
Watermark.ZIndex = 200
Instance.new("UICorner", Watermark).CornerRadius = UDim.new(0, 6)
local wmS = Instance.new("UIStroke", Watermark)
wmS.Color = THEME.Accent; wmS.Thickness = 1; wmS.Transparency = 0.3

local wmDot = Instance.new("Frame", Watermark)
wmDot.Size = UDim2.new(0, 6, 0, 6); wmDot.Position = UDim2.new(0, 8, 0.5, -3)
wmDot.BackgroundColor3 = Color3.fromRGB(55, 215, 115); wmDot.BorderSizePixel = 0
Instance.new("UICorner", wmDot).CornerRadius = UDim.new(1, 0)

local wmTitle = Instance.new("TextLabel", Watermark)
wmTitle.Size = UDim2.new(0, 60, 1, 0); wmTitle.Position = UDim2.new(0, 20, 0, 0)
wmTitle.BackgroundTransparency = 1; wmTitle.Text = "SVEY"
wmTitle.TextColor3 = THEME.Accent; wmTitle.Font = Enum.Font.GothamBlack
wmTitle.TextSize = 12; wmTitle.TextXAlignment = Enum.TextXAlignment.Left

local wmPremium = Instance.new("TextLabel", Watermark)
wmPremium.Size = UDim2.new(0, 55, 1, 0); wmPremium.Position = UDim2.new(0, 55, 0, 0)
wmPremium.BackgroundTransparency = 1; wmPremium.Text = "PREMIUM"
wmPremium.TextColor3 = Color3.fromRGB(255, 200, 60); wmPremium.Font = Enum.Font.GothamBold
wmPremium.TextSize = 9; wmPremium.TextXAlignment = Enum.TextXAlignment.Left

local wmFPS = Instance.new("TextLabel", Watermark)
wmFPS.Size = UDim2.new(0, 50, 1, 0); wmFPS.Position = UDim2.new(0, 120, 0, 0)
wmFPS.BackgroundTransparency = 1; wmFPS.Text = "60 FPS"
wmFPS.TextColor3 = Color3.fromRGB(55, 215, 115); wmFPS.Font = Enum.Font.RobotoMono
wmFPS.TextSize = 10; wmFPS.TextXAlignment = Enum.TextXAlignment.Left

local wmPing = Instance.new("TextLabel", Watermark)
wmPing.Size = UDim2.new(0, 45, 1, 0); wmPing.Position = UDim2.new(0, 177, 0, 0)
wmPing.BackgroundTransparency = 1; wmPing.Text = "0 MS"
wmPing.TextColor3 = Color3.fromRGB(140, 135, 155); wmPing.Font = Enum.Font.RobotoMono
wmPing.TextSize = 10; wmPing.TextXAlignment = Enum.TextXAlignment.Left

-- Клик по всему Watermark
local wmClick = Instance.new("TextButton", Watermark)
wmClick.Size = UDim2.new(1, 0, 1, 0)
wmClick.BackgroundTransparency = 1
wmClick.Text = ""; wmClick.ZIndex = 210
wmClick.MouseButton1Click:Connect(function()
    if _G.SVEY_TOGGLE_MENU then
        _G.SVEY_TOGGLE_MENU()
    end
end)
wmClick.MouseEnter:Connect(function()
    TweenService:Create(Watermark, TweenInfo.new(0.15), {BackgroundTransparency = 0}):Play()
end)
wmClick.MouseLeave:Connect(function()
    TweenService:Create(Watermark, TweenInfo.new(0.15), {BackgroundTransparency = 0.15}):Play()
end)

-- FPS / Ping обновление
local fpsFrames, fpsLast = 0, tick()
RunService.RenderStepped:Connect(function()
    fpsFrames = fpsFrames + 1
    if tick() - fpsLast >= 1 then
        local fps = fpsFrames
        wmFPS.Text = fps .. " FPS"
        if fps >= 50 then wmFPS.TextColor3 = Color3.fromRGB(55, 215, 115)
        elseif fps >= 30 then wmFPS.TextColor3 = Color3.fromRGB(250, 205, 45)
        else wmFPS.TextColor3 = Color3.fromRGB(240, 50, 60) end
        fpsFrames = 0; fpsLast = tick()
    end
end)
task.spawn(function()
    while Watermark.Parent do
        local ping = 0
        pcall(function()
            ping = math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue())
        end)
        wmPing.Text = ping .. " MS"
        if ping < 100 then wmPing.TextColor3 = Color3.fromRGB(55, 215, 115)
        elseif ping < 200 then wmPing.TextColor3 = Color3.fromRGB(250, 205, 45)
        else wmPing.TextColor3 = Color3.fromRGB(240, 50, 60) end
        task.wait(1)
    end
end)
-- [[ ЧАСТЬ 2/3 ]]

-- ========== MAIN FRAME ==========
local MainFrame = Instance.new("Frame")
local vpSize = Camera.ViewportSize
if IS_MOBILE then
    local w = math.floor(vpSize.X * 0.94)
    local h = math.floor(vpSize.Y * 0.85)
    MainFrame.Size = UDim2.new(0, w, 0, h)
    MainFrame.Position = UDim2.new(0.5, -w/2, 0.5, -h/2)
else
    MainFrame.Size = UDim2.new(0, 720, 0, 480)
    MainFrame.Position = UDim2.new(0.5, -360, 0.5, -240)
end
MainFrame.BackgroundColor3 = THEME.BG
MainFrame.BorderSizePixel = 0; MainFrame.Active = true
MainFrame.Visible = false; MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)
local mStroke = Instance.new("UIStroke", MainFrame)
mStroke.Color = THEME.Border; mStroke.Thickness = 1.2

-- Тогл меню
_G.SVEY_TOGGLE_MENU = function()
    MainFrame.Visible = not MainFrame.Visible
    if MainFrame.Visible then
        wmS.Color = Color3.fromRGB(240, 50, 60)
        wmDot.BackgroundColor3 = Color3.fromRGB(240, 50, 60)
        if State.BlurEnabled then
            local b = Lighting:FindFirstChild("SVEY_Blur")
            if not b then b = Instance.new("BlurEffect", Lighting); b.Name = "SVEY_Blur" end
            TweenService:Create(b, TweenInfo.new(0.3), {Size = 15}):Play()
        end
    else
        wmS.Color = THEME.Accent
        wmDot.BackgroundColor3 = Color3.fromRGB(55, 215, 115)
        local b = Lighting:FindFirstChild("SVEY_Blur")
        if b then TweenService:Create(b, TweenInfo.new(0.3), {Size = 0}):Play() end
    end
end

-- ========== HEADER ==========
local Header = Instance.new("Frame", MainFrame)
Header.Size = UDim2.new(1, 0, 0, 48)
Header.BackgroundColor3 = THEME.BG2; Header.BorderSizePixel = 0; Header.ZIndex = 2
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 10)
local hCover = Instance.new("Frame", Header)
hCover.Size = UDim2.new(1, 0, 0, 24); hCover.Position = UDim2.new(0, 0, 1, -24)
hCover.BackgroundColor3 = THEME.BG2; hCover.BorderSizePixel = 0

local hIcon = Instance.new("Frame", Header)
hIcon.Size = UDim2.new(0, 28, 0, 28); hIcon.Position = UDim2.new(0, 15, 0.5, -14)
hIcon.BackgroundColor3 = THEME.Accent; hIcon.BorderSizePixel = 0; hIcon.ZIndex = 3
Instance.new("UICorner", hIcon).CornerRadius = UDim.new(0, 6)
local hiLabel = Instance.new("TextLabel", hIcon)
hiLabel.Size = UDim2.new(1,0,1,0); hiLabel.BackgroundTransparency = 1
hiLabel.Text = "S"; hiLabel.TextColor3 = Color3.fromRGB(255,255,255)
hiLabel.Font = Enum.Font.GothamBlack; hiLabel.TextSize = 16; hiLabel.ZIndex = 4

local hTitle = Instance.new("TextLabel", Header)
hTitle.Size = UDim2.new(0, 300, 1, 0); hTitle.Position = UDim2.new(0, 55, 0, 0)
hTitle.BackgroundTransparency = 1; hTitle.Text = "Ragebot"
hTitle.TextColor3 = THEME.Text; hTitle.Font = Enum.Font.GothamBold
hTitle.TextSize = 18; hTitle.TextXAlignment = Enum.TextXAlignment.Left; hTitle.ZIndex = 3

local CloseBtn = Instance.new("TextButton", Header)
CloseBtn.Size = UDim2.new(0, 32, 0, 32); CloseBtn.Position = UDim2.new(1, -42, 0.5, -16)
CloseBtn.BackgroundColor3 = THEME.BG3; CloseBtn.Text = "✕"
CloseBtn.TextColor3 = THEME.Text; CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 16; CloseBtn.AutoButtonColor = false; CloseBtn.ZIndex = 3
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
CloseBtn.MouseButton1Click:Connect(function()
    if _G.SVEY_TOGGLE_MENU then _G.SVEY_TOGGLE_MENU() end
end)

-- ========== SIDEBAR ==========
local Sidebar = Instance.new("Frame", MainFrame)
Sidebar.Size = UDim2.new(0, 65, 1, -48); Sidebar.Position = UDim2.new(0, 0, 0, 48)
Sidebar.BackgroundColor3 = THEME.BG2; Sidebar.BorderSizePixel = 0; Sidebar.ZIndex = 2

local SidebarScroll = Instance.new("ScrollingFrame", Sidebar)
SidebarScroll.Size = UDim2.new(1, 0, 1, 0)
SidebarScroll.BackgroundTransparency = 1; SidebarScroll.BorderSizePixel = 0
SidebarScroll.ScrollBarThickness = 0
SidebarScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
SidebarScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
local SidebarLayout = Instance.new("UIListLayout", SidebarScroll)
SidebarLayout.Padding = UDim.new(0, 2)
SidebarLayout.SortOrder = Enum.SortOrder.LayoutOrder
local SidebarPad = Instance.new("UIPadding", SidebarScroll)
SidebarPad.PaddingTop = UDim.new(0, 4); SidebarPad.PaddingBottom = UDim.new(0, 4)

local ContentFrame = Instance.new("Frame", MainFrame)
ContentFrame.Size = UDim2.new(1, -65, 1, -48); ContentFrame.Position = UDim2.new(0, 65, 0, 48)
ContentFrame.BackgroundColor3 = THEME.BG; ContentFrame.BorderSizePixel = 0; ContentFrame.ZIndex = 2

-- ========== ИКОНКИ ==========
local iconMap = {
    Legitbot    = "rbxassetid://98370306170658",
    Ragebot     = "rbxassetid://107512202152283",
    Enemy       = "rbxassetid://127827492424396",
    World       = "rbxassetid://100545076798978",
    SkinChanger = "rbxassetid://107710676270056",
    Misc        = "rbxassetid://103915348216097",
    Config      = "rbxassetid://97047003599965",
}

local Tabs, TabPages, currentTab = {}, {}, nil

local function switchTab(name)
    for tabName, data in pairs(Tabs) do
        local active = tabName == name
        data.Button.BackgroundColor3 = active and THEME.BG3 or THEME.BG2
        if data.Indicator then data.Indicator.Visible = active end
        if data.Icon then
            if data.Icon:IsA("ImageLabel") then
                data.Icon.ImageColor3 = active and THEME.Accent or THEME.TextDim
            else
                data.Icon.TextColor3 = active and THEME.Accent or THEME.TextDim
            end
        end
        if data.NameLbl then
            data.NameLbl.TextColor3 = active and THEME.Accent or THEME.TextDim
        end
        if TabPages[tabName] then TabPages[tabName].Visible = active end
    end
    currentTab = name
    hTitle.Text = name
end

local function createTab(name, icon, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 46)
    btn.BackgroundColor3 = THEME.BG2
    btn.Text = ""; btn.LayoutOrder = order; btn.Parent = SidebarScroll

    local iconElement
    if typeof(icon) == "string" and icon:find("rbxassetid://") then
        iconElement = Instance.new("ImageLabel", btn)
        iconElement.Size = UDim2.new(0, 22, 0, 22); iconElement.Position = UDim2.new(0.5, -11, 0, 4)
        iconElement.BackgroundTransparency = 1; iconElement.Image = icon
        iconElement.ImageColor3 = THEME.TextDim; iconElement.ScaleType = Enum.ScaleType.Fit
    else
        iconElement = Instance.new("TextLabel", btn)
        iconElement.Size = UDim2.new(1, 0, 0, 20); iconElement.Position = UDim2.new(0, 0, 0, 4)
        iconElement.BackgroundTransparency = 1; iconElement.Text = icon
        iconElement.TextColor3 = THEME.TextDim
        iconElement.Font = Enum.Font.GothamBold; iconElement.TextSize = 18
    end

    local nameLbl = Instance.new("TextLabel", btn)
    nameLbl.Size = UDim2.new(1, 0, 0, 10); nameLbl.Position = UDim2.new(0, 0, 0, 30)
    nameLbl.BackgroundTransparency = 1; nameLbl.Text = name
    nameLbl.TextColor3 = THEME.TextDim; nameLbl.Font = Enum.Font.Gotham; nameLbl.TextSize = 7

    local indicator = Instance.new("Frame", btn)
    indicator.Size = UDim2.new(0, 3, 1, 0); indicator.Position = UDim2.new(0, 0, 0, 0)
    indicator.BackgroundColor3 = THEME.Accent; indicator.BorderSizePixel = 0
    indicator.Visible = false

    local tabContainer = Instance.new("Frame", ContentFrame)
    tabContainer.Size = UDim2.new(1, 0, 1, 0)
    tabContainer.BackgroundTransparency = 1; tabContainer.Visible = false

    local leftCol = Instance.new("ScrollingFrame", tabContainer)
    leftCol.Size = UDim2.new(0.5, -6, 1, 0); leftCol.BackgroundTransparency = 1
    leftCol.BorderSizePixel = 0; leftCol.ScrollBarThickness = 3
    leftCol.ScrollBarImageColor3 = THEME.Accent
    leftCol.AutomaticCanvasSize = Enum.AutomaticSize.Y
    local leftLayout = Instance.new("UIListLayout", leftCol)
    leftLayout.Padding = UDim.new(0, 4); leftLayout.SortOrder = Enum.SortOrder.LayoutOrder
    local leftPad = Instance.new("UIPadding", leftCol)
    leftPad.PaddingTop = UDim.new(0, 8); leftPad.PaddingLeft = UDim.new(0, 8)
    leftPad.PaddingRight = UDim.new(0, 4); leftPad.PaddingBottom = UDim.new(0, 8)

    local rightCol = Instance.new("ScrollingFrame", tabContainer)
    rightCol.Size = UDim2.new(0.5, -6, 1, 0); rightCol.Position = UDim2.new(0.5, 6, 0, 0)
    rightCol.BackgroundTransparency = 1; rightCol.BorderSizePixel = 0
    rightCol.ScrollBarThickness = 3; rightCol.ScrollBarImageColor3 = THEME.Accent
    rightCol.AutomaticCanvasSize = Enum.AutomaticSize.Y
    local rightLayout = Instance.new("UIListLayout", rightCol)
    rightLayout.Padding = UDim.new(0, 4); rightLayout.SortOrder = Enum.SortOrder.LayoutOrder
    local rightPad = Instance.new("UIPadding", rightCol)
    rightPad.PaddingTop = UDim.new(0, 8); rightPad.PaddingLeft = UDim.new(0, 4)
    rightPad.PaddingRight = UDim.new(0, 8); rightPad.PaddingBottom = UDim.new(0, 8)

    btn.MouseButton1Click:Connect(function() switchTab(name) end)
    Tabs[name] = {Button=btn, Indicator=indicator, Icon=iconElement, NameLbl=nameLbl}
    TabPages[name] = tabContainer
    return leftCol, rightCol
end

-- ========== UI ЭЛЕМЕНТЫ ==========
local order = 0
local function nextOrder() order = order + 1; return order end

-- Реестр элементов для Config-синхронизации
local UI = { toggles = {}, sliders = {}, dropdowns = {}, colors = {} }

local function sectionLabel(parent, text)
    local l = Instance.new("TextLabel", parent)
    l.Size = UDim2.new(1, 0, 0, 22); l.BackgroundTransparency = 1
    l.Text = string.upper(text); l.TextColor3 = THEME.Accent
    l.Font = Enum.Font.GothamBold; l.TextSize = 11
    l.TextXAlignment = Enum.TextXAlignment.Left; l.LayoutOrder = nextOrder()
    return l
end

-- ========== COLOR PICKER ==========
local ActivePicker = nil
local function openColorPicker(title, initialColor, onChanged)
    if ActivePicker then ActivePicker:Destroy(); ActivePicker = nil end
    local pickerGui = Instance.new("ScreenGui")
    pickerGui.Name = "SVEY_Picker"; pickerGui.ResetOnSpawn = false
    pickerGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    pickerGui.Parent = PlayerGui

    local frame = Instance.new("Frame", pickerGui)
    frame.Size = UDim2.new(0, 280, 0, 260)
    frame.Position = UDim2.new(0.5, -140, 0.5, -130)
    frame.BackgroundColor3 = THEME.BG; frame.BorderSizePixel = 0; frame.Active = true
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
    local fs = Instance.new("UIStroke", frame); fs.Color = THEME.Accent; fs.Thickness = 1.5

    local titleLbl = Instance.new("TextLabel", frame)
    titleLbl.Size = UDim2.new(1, -40, 0, 24); titleLbl.Position = UDim2.new(0, 12, 0, 6)
    titleLbl.BackgroundTransparency = 1; titleLbl.Text = string.upper(title)
    titleLbl.TextColor3 = THEME.Accent; titleLbl.Font = Enum.Font.GothamBold
    titleLbl.TextSize = 12; titleLbl.TextXAlignment = Enum.TextXAlignment.Left

    local closeBtn = Instance.new("TextButton", frame)
    closeBtn.Size = UDim2.new(0, 22, 0, 22); closeBtn.Position = UDim2.new(1, -28, 0, 6)
    closeBtn.BackgroundColor3 = THEME.BG3; closeBtn.Text = "✕"
    closeBtn.TextColor3 = THEME.Text; closeBtn.Font = Enum.Font.GothamBold; closeBtn.TextSize = 12
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 4)
    closeBtn.MouseButton1Click:Connect(function()
        pickerGui:Destroy(); ActivePicker = nil
    end)

    local grid = Instance.new("Frame", frame)
    grid.Size = UDim2.new(1, -20, 0, 200); grid.Position = UDim2.new(0, 10, 0, 34)
    grid.BackgroundTransparency = 1
    local gridL = Instance.new("UIGridLayout", grid)
    gridL.CellSize = UDim2.new(0, 38, 0, 30)
    gridL.CellPadding = UDim2.new(0, 6, 0, 6)
    gridL.SortOrder = Enum.SortOrder.LayoutOrder

    for i, preset in ipairs(COLOR_PRESETS) do
        local swatch = Instance.new("TextButton", grid)
        swatch.BackgroundColor3 = preset.Color; swatch.Text = ""
        swatch.AutoButtonColor = false; swatch.LayoutOrder = i
        Instance.new("UICorner", swatch).CornerRadius = UDim.new(0, 5)
        local ss = Instance.new("UIStroke", swatch)
        ss.Color = Color3.fromRGB(80,80,80); ss.Thickness = 1; ss.Transparency = 0.3
        swatch.MouseButton1Click:Connect(function()
            if onChanged then onChanged(preset.Color) end
            pickerGui:Destroy(); ActivePicker = nil
        end)
    end
    ActivePicker = pickerGui
end

local function createColorRow(parent, text, default, callback, stateKey)
    local row = Instance.new("Frame", parent)
    row.Size = UDim2.new(1, 0, 0, 28); row.BackgroundColor3 = THEME.BG3
    row.LayoutOrder = nextOrder()
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 4)
    local label = Instance.new("TextLabel", row)
    label.Size = UDim2.new(1, -50, 1, 0); label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1; label.Text = text
    label.TextColor3 = THEME.Text; label.Font = Enum.Font.Gotham
    label.TextSize = 12; label.TextXAlignment = Enum.TextXAlignment.Left

    local swatch = Instance.new("TextButton", row)
    swatch.Size = UDim2.new(0, 24, 0, 24); swatch.Position = UDim2.new(1, -34, 0.5, -12)
    swatch.BackgroundColor3 = default; swatch.Text = ""; swatch.AutoButtonColor = false
    Instance.new("UICorner", swatch).CornerRadius = UDim.new(0, 5)
    local swS = Instance.new("UIStroke", swatch)
    swS.Color = Color3.fromRGB(80,80,80); swS.Thickness = 1

    swatch.MouseButton1Click:Connect(function()
        openColorPicker(text, swatch.BackgroundColor3, function(newColor)
            swatch.BackgroundColor3 = newColor
            if callback then callback(newColor) end
        end)
    end)
    if stateKey then UI.colors[stateKey] = swatch end
    return row
end

-- ========== TOGGLE ==========
local function createToggle(parent, text, default, callback, stateKey)
    local row = Instance.new("Frame", parent)
    row.Size = UDim2.new(1, 0, 0, 28); row.BackgroundColor3 = THEME.BG3
    row.LayoutOrder = nextOrder()
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 4)
    local label = Instance.new("TextLabel", row)
    label.Size = UDim2.new(1, -55, 1, 0); label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1; label.Text = text
    label.TextColor3 = default and THEME.Accent or THEME.Text
    label.Font = Enum.Font.Gotham; label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left

    local bg = Instance.new("Frame", row)
    bg.Size = UDim2.new(0, 32, 0, 18); bg.Position = UDim2.new(1, -40, 0.5, -9)
    bg.BackgroundColor3 = default and THEME.Accent or THEME.BG4; bg.BorderSizePixel = 0
    Instance.new("UICorner", bg).CornerRadius = UDim.new(1, 0)
    local dot = Instance.new("Frame", bg)
    dot.Size = UDim2.new(0, 14, 0, 14)
    dot.Position = default and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)
    dot.BackgroundColor3 = Color3.fromRGB(255,255,255); dot.BorderSizePixel = 0
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local value = default
    local function setVisual(v)
        value = v
        bg.BackgroundColor3 = v and THEME.Accent or THEME.BG4
        dot.Position = v and UDim2.new(1,-16,0.5,-7) or UDim2.new(0,2,0.5,-7)
        label.TextColor3 = v and THEME.Accent or THEME.Text
    end

    local btn = Instance.new("TextButton", row)
    btn.Size = UDim2.new(1, 0, 1, 0); btn.BackgroundTransparency = 1; btn.Text = ""
    btn.MouseButton1Click:Connect(function()
        value = not value
        TweenService:Create(bg, TweenInfo.new(0.15), {BackgroundColor3 = value and THEME.Accent or THEME.BG4}):Play()
        TweenService:Create(dot, TweenInfo.new(0.15), {Position = value and UDim2.new(1,-16,0.5,-7) or UDim2.new(0,2,0.5,-7)}):Play()
        TweenService:Create(label, TweenInfo.new(0.15), {TextColor3 = value and THEME.Accent or THEME.Text}):Play()
        if callback then pcall(callback, value) end
    end)

    if stateKey then
        UI.toggles[stateKey] = { set = setVisual, get = function() return value end }
    end
    return row
end

-- ========== SLIDER ==========
local function createSlider(parent, text, min, max, default, callback, isFloat, stateKey)
    local row = Instance.new("Frame", parent)
    row.Size = UDim2.new(1, 0, 0, 36); row.BackgroundColor3 = THEME.BG3
    row.LayoutOrder = nextOrder()
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 4)
    local label = Instance.new("TextLabel", row)
    label.Size = UDim2.new(0.5, -10, 0, 14); label.Position = UDim2.new(0, 10, 0, 3)
    label.BackgroundTransparency = 1; label.Text = text
    label.TextColor3 = THEME.Text; label.Font = Enum.Font.Gotham
    label.TextSize = 11; label.TextXAlignment = Enum.TextXAlignment.Left
    local valueLbl = Instance.new("TextLabel", row)
    valueLbl.Size = UDim2.new(0.5, -10, 0, 14); valueLbl.Position = UDim2.new(0.5, 0, 0, 3)
    valueLbl.BackgroundTransparency = 1
    valueLbl.Text = isFloat and string.format("%.1f", default) or tostring(default)
    valueLbl.TextColor3 = THEME.Accent; valueLbl.Font = Enum.Font.GothamBold
    valueLbl.TextSize = 11; valueLbl.TextXAlignment = Enum.TextXAlignment.Right
    local bar = Instance.new("Frame", row)
    bar.Size = UDim2.new(1, -20, 0, 4); bar.Position = UDim2.new(0, 10, 0, 26)
    bar.BackgroundColor3 = THEME.BG4; bar.BorderSizePixel = 0
    Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)
    local fill = Instance.new("Frame", bar)
    fill.Size = UDim2.new((default-min)/(max-min), 0, 1, 0)
    fill.BackgroundColor3 = THEME.Accent; fill.BorderSizePixel = 0
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)
    local handle = Instance.new("Frame", bar)
    handle.Size = UDim2.new(0, 12, 0, 12)
    handle.Position = UDim2.new((default-min)/(max-min), -6, 0.5, -6)
    handle.BackgroundColor3 = Color3.fromRGB(255,255,255); handle.BorderSizePixel = 0
    Instance.new("UICorner", handle).CornerRadius = UDim.new(1, 0)

    local value = default
    local function setVisual(v, fireCallback)
        value = math.clamp(v, min, max)
        valueLbl.Text = isFloat and string.format("%.1f", value) or tostring(math.floor(value))
        local np = (value - min) / (max - min)
        fill.Size = UDim2.new(np, 0, 1, 0)
        handle.Position = UDim2.new(np, -6, 0.5, -6)
        if fireCallback and callback then pcall(callback, value) end
    end

    local dragging = false
    local function updateFromX(mx)
        local bx = bar.AbsolutePosition.X
        local bw = bar.AbsoluteSize.X
        local pct = math.clamp((mx - bx) / bw, 0, 1)
        local nv = min + pct * (max - min)
        if not isFloat then nv = math.floor(nv + 0.5) end
        setVisual(nv, true)
    end
    local clickArea = Instance.new("TextButton", bar)
    clickArea.Size = UDim2.new(1, 0, 4, 0); clickArea.Position = UDim2.new(0, 0, 0, -8)
    clickArea.BackgroundTransparency = 1; clickArea.Text = ""
    clickArea.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true; updateFromX(input.Position.X)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch) then
            updateFromX(input.Position.X)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    if stateKey then UI.sliders[stateKey] = { set = setVisual, get = function() return value end } end
    return row
end

-- ========== DROPDOWN ==========
local function createDropdown(parent, text, options, default, callback, stateKey)
    local row = Instance.new("Frame", parent)
    row.Size = UDim2.new(1, 0, 0, 28); row.BackgroundColor3 = THEME.BG3
    row.LayoutOrder = nextOrder(); row.ClipsDescendants = false; row.ZIndex = 2
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 4)
    local label = Instance.new("TextLabel", row)
    label.Size = UDim2.new(0.4, -10, 1, 0); label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1; label.Text = text
    label.TextColor3 = THEME.Text; label.Font = Enum.Font.Gotham
    label.TextSize = 11; label.TextXAlignment = Enum.TextXAlignment.Left

    local sel = Instance.new("TextButton", row)
    sel.Size = UDim2.new(0.55, -10, 0, 22); sel.Position = UDim2.new(0.4, 5, 0.5, -11)
    sel.BackgroundColor3 = THEME.BG4; sel.Text = default .. "  ▾"
    sel.TextColor3 = THEME.Text; sel.Font = Enum.Font.Gotham; sel.TextSize = 11
    sel.AutoButtonColor = false; sel.ZIndex = 3
    Instance.new("UICorner", sel).CornerRadius = UDim.new(0, 4)

    local list = Instance.new("Frame", row)
    list.Size = UDim2.new(0.55, -10, 0, #options * 22)
    list.Position = UDim2.new(0.4, 5, 1, 2)
    list.BackgroundColor3 = THEME.BG2; list.BorderSizePixel = 0
    list.Visible = false; list.ZIndex = 10
    Instance.new("UICorner", list).CornerRadius = UDim.new(0, 4)
    local lScroll = Instance.new("ScrollingFrame", list)
    lScroll.Size = UDim2.new(1, 0, 1, 0); lScroll.BackgroundTransparency = 1
    lScroll.BorderSizePixel = 0; lScroll.ScrollBarThickness = 3
    lScroll.CanvasSize = UDim2.new(0, 0, 0, #options * 22); lScroll.ZIndex = 10
    Instance.new("UIListLayout", lScroll)

    local value = default
    local function setVisual(v)
        value = v
        sel.Text = v .. "  ▾"
    end

    for i, opt in ipairs(options) do
        local ob = Instance.new("TextButton", lScroll)
        ob.Size = UDim2.new(1, 0, 0, 22); ob.BackgroundColor3 = THEME.BG2
        ob.Text = opt; ob.TextColor3 = THEME.Text; ob.Font = Enum.Font.Gotham
        ob.TextSize = 11; ob.LayoutOrder = i; ob.ZIndex = 10; ob.AutoButtonColor = false
        ob.MouseButton1Click:Connect(function()
            setVisual(opt); list.Visible = false
            if callback then pcall(callback, opt) end
        end)
        ob.MouseEnter:Connect(function() ob.BackgroundColor3 = THEME.BG3 end)
        ob.MouseLeave:Connect(function() ob.BackgroundColor3 = THEME.BG2 end)
    end
    sel.MouseButton1Click:Connect(function() list.Visible = not list.Visible end)

    if stateKey then UI.dropdowns[stateKey] = { set = setVisual, get = function() return value end } end
    return row
end

-- ========== BUTTON ==========
local function createButton(parent, text, color, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, 0, 0, 28); btn.BackgroundColor3 = color or THEME.Accent
    btn.Text = text; btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.GothamBold; btn.TextSize = 11
    btn.LayoutOrder = nextOrder(); btn.AutoButtonColor = false
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    btn.MouseButton1Click:Connect(function() pcall(callback) end)
    return btn
end

-- Экспорт ссылок для Части 3
_G.SVEY = {
    UI        = UI,
    createTab = createTab,
    sectionLabel = sectionLabel,
    createToggle = createToggle,
    createSlider = createSlider,
    createDropdown = createDropdown,
    createColorRow = createColorRow,
    createButton = createButton,
    switchTab = switchTab,
    SendNotification = SendNotification,
    State = State,
    Effects = Effects,
    THEME = THEME,
    iconMap = iconMap,
    applyFog = function() end, -- будет перезаписан в Части 3
}
-- [[ ЧАСТЬ 3/3 ]]

local UI             = _G.SVEY.UI
local createTab      = _G.SVEY.createTab
local sectionLabel   = _G.SVEY.sectionLabel
local createToggle   = _G.SVEY.createToggle
local createSlider   = _G.SVEY.createSlider
local createDropdown = _G.SVEY.createDropdown
local createColorRow = _G.SVEY.createColorRow
local createButton   = _G.SVEY.createButton
local switchTab      = _G.SVEY.switchTab
local SendNotification = _G.SVEY.SendNotification
local THEME          = _G.SVEY.THEME
local iconMap        = _G.SVEY.iconMap

-- ========== ВКЛАДКИ ==========
local RageL, RageR   = createTab("Ragebot", iconMap.Ragebot, 1)
local EnemyL, EnemyR = createTab("Enemy",   iconMap.Enemy,   2)
local SkinL, SkinR   = createTab("SkinChanger", iconMap.SkinChanger, 3)
local LegitL, LegitR = createTab("Legitbot", iconMap.Legitbot, 4)
local WorldL, WorldR = createTab("World",   iconMap.World,   5)
local MiscL, MiscR   = createTab("Misc",    iconMap.Misc,    6)
local CfgL, CfgR     = createTab("Config",  iconMap.Config,  7)

-- ========== RAGEBOT ==========
sectionLabel(RageL, "Silent Aim")
createToggle(RageL, "Silent Aim", false, function(v) State.silentAim = v end, "silentAim")
createSlider(RageL, "Silent FOV", 20, 600, 150, function(v) State.silentAimFOV = v end, false, "silentAimFOV")
createToggle(RageL, "Show FOV Circle", false, function(v) State.showFov = v end, "showFov")
createColorRow(RageL, "FOV Color", Color3.fromRGB(255, 60, 60), function(c) State.fovColor = c end, "fovColor")
createDropdown(RageL, "Hitbox Part", {"Head", "Torso", "HRP"}, "Head", function(v) State.silentAimTargetPart = v end, "silentAimTargetPart")

sectionLabel(RageL, "Hitbox Expander")
createToggle(RageL, "Hitbox Expander", false, function(v)
    State.HitboxExpander = v
    if v then updateAllHitboxes() end
    updateHitboxVisibility()
end, "HitboxExpander")
createSlider(RageL, "Hitbox Size", 1, 20, 5, function(v) State.HitboxSize = v end, true, "HitboxSize")
createColorRow(RageL, "Hitbox Color", Color3.fromRGB(185, 115, 255), function(c) State.hitboxColor = c end, "hitboxColor")

sectionLabel(RageR, "Aimbot")
createToggle(RageR, "Aimbot", false, function(v) State.Aimbot = v end, "Aimbot")
createSlider(RageR, "Aimbot FOV", 20, 600, 120, function(v) State.AimbotFOV = v end, false, "AimbotFOV")
createSlider(RageR, "Smoothness", 0, 100, 50, function(v) State.AimbotSmoothness = v/100 end, false, "AimbotSmoothness")

sectionLabel(RageR, "Anti-Aim")
createToggle(RageR, "Anti-Aim", false, function(v) State.AntiAim = v end, "AntiAim")
createDropdown(RageR, "AA Mode", {"Spin", "Jitter", "Random", "Desync"}, "Spin", function(v) State.AntiAimMode = v end, "AntiAimMode")
createSlider(RageR, "AA Speed", 1, 500, 50, function(v) State.AntiAimSpeed = v end, false, "AntiAimSpeed")

-- ========== ENEMY ==========
sectionLabel(EnemyL, "MM2 Roles")
createToggle(EnemyL, "Role ESP",     false, function(v) State.roleEsp = v end, "roleEsp")
createToggle(EnemyL, "ESP Murderer", false, function(v) State.espMurder = v end, "espMurder")
createToggle(EnemyL, "ESP Sheriff",  false, function(v) State.espSheriff = v end, "espSheriff")
createToggle(EnemyL, "ESP Hero",     false, function(v) State.espHero = v end, "espHero")
createToggle(EnemyL, "ESP Innocents",false, function(v) State.espInnocents = v end, "espInnocents")
createToggle(EnemyL, "ESP All",      false, function(v) State.espAll = v end, "espAll")

sectionLabel(EnemyL, "2D Box")
createToggle(EnemyL, "2D Box ESP", false, function(v) State.box2D = v end, "box2D")
createToggle(EnemyL, "2D Box by Role", false, function(v) State.box2DMM2 = v end, "box2DMM2")
createDropdown(EnemyL, "Box Style", {"Full", "Corner"}, "Full", function(v) State.box2DMode = v end, "box2DMode")
createColorRow(EnemyL, "Box Color", Color3.fromRGB(255, 255, 255), function(c) State.box2DColor = c end, "box2DColor")
createToggle(EnemyL, "Filled Box", false, function(v) State.box2DFill = v end, "box2DFill")
createColorRow(EnemyL, "Fill Color", Color3.fromRGB(160, 90, 255), function(c) State.box2DFillColor = c end, "box2DFillColor")

sectionLabel(EnemyR, "3D Visuals")
createToggle(EnemyR, "Highlight ESP", false, function(v) State.highlightEsp = v end, "highlightEsp")
createColorRow(EnemyR, "Highlight Color", Color3.fromRGB(180, 90, 255), function(c) State.highlightColor = c end, "highlightColor")
createToggle(EnemyR, "3D Box ESP", false, function(v) State.box3D = v end, "box3D")
createColorRow(EnemyR, "3D Box Color", Color3.fromRGB(160, 90, 255), function(c) State.box3DColor = c end, "box3DColor")

sectionLabel(EnemyR, "Info Overlay")
createToggle(EnemyR, "Skeleton ESP", false, function(v) State.skeletonEsp = v end, "skeletonEsp")
createColorRow(EnemyR, "Skeleton Color", Color3.fromRGB(255, 255, 255), function(c) State.skeletonColor = c end, "skeletonColor")
createToggle(EnemyR, "Arrow ESP", false, function(v) State.arrowEsp = v end, "arrowEsp")
createColorRow(EnemyR, "Arrow Color", Color3.fromRGB(255, 255, 255), function(c) State.arrowEspColor = c end, "arrowEspColor")
createToggle(EnemyR, "Snaplines", false, function(v) State.snaplines = v end, "snaplines")
createColorRow(EnemyR, "Snapline Color", Color3.fromRGB(255, 255, 255), function(c) State.snaplineColor = c end, "snaplineColor")
createToggle(EnemyR, "Health Bar", false, function(v) State.healthBar = v end, "healthBar")
createToggle(EnemyR, "Distance ESP", false, function(v) State.distanceEsp = v end, "distanceEsp")
createColorRow(EnemyR, "Distance Color", Color3.fromRGB(255, 255, 255), function(c) State.distanceColor = c end, "distanceColor")

-- ========== SKINCHANGER ==========
sectionLabel(SkinL, "Aura")
createToggle(SkinL, "Celestial Aura", false, function(v)
    State.aura = v
    if v then buildAnimeAura() else cleanAnimeAura() end
end, "aura")
createColorRow(SkinL, "Aura Color", Color3.fromRGB(185, 115, 255), function(c) State.auraColor = c end, "auraColor")
createToggle(SkinL, "Angel Wings", false, function(v)
    State.selfWings = v
    if v then buildWings() else cleanWings() end
end, "selfWings")
createColorRow(SkinL, "Wings Color", Color3.fromRGB(255, 255, 255), function(c) State.selfWingsColor = c end, "selfWingsColor")

sectionLabel(SkinL, "Glass Chams")
createToggle(SkinL, "Glass Chams", false, function(v)
    State.glassChams = v
    if not v then restoreChams() end
end, "glassChams")
createToggle(SkinL, "Chams All Players", false, function(v) State.glassChamsAll = v end, "glassChamsAll")
createColorRow(SkinL, "Chams Color", Color3.fromRGB(160, 100, 255), function(c) State.glassChamsColor = c end, "glassChamsColor")
createSlider(SkinL, "Transparency", 0, 100, 35, function(v) State.ChamsTransparency = v/100 end, false, "ChamsTransparency")

sectionLabel(SkinR, "★★★ DICK (рабочий) ★★★")
createToggle(SkinR, "Dick", false, function(v)
    State.Dick = v
    if v then
        Effects.DickParts = {}
        applyDick()
    else
        clearDick()
    end
end, "Dick")
createSlider(SkinR, "Dick Size", 5, 30, 10, function(v)
    State.DickSize = v/10
    if State.Dick then
        Effects.DickParts = {}
        applyDick()
    end
end, true, "DickSize")
createColorRow(SkinR, "Dick Color", Color3.fromRGB(255, 200, 150), function(c)
    State.DickColor = c
    if State.Dick then
        Effects.DickParts = {}
        applyDick()
    end
end, "DickColor")

sectionLabel(SkinR, "Hat")
createToggle(SkinR, "Hat", false, function(v)
    State.Hat = v
    if v then applyHat() else clearHat() end
end, "Hat")
createSlider(SkinR, "Hat Size", 5, 30, 10, function(v)
    State.HatSize = v/10
    if State.Hat then applyHat() end
end, true, "HatSize")
createColorRow(SkinR, "Hat Color", Color3.fromRGB(100, 255, 150), function(c)
    State.HatColor = c
    if State.Hat then applyHat() end
end, "HatColor")

-- ========== LEGITBOT ==========
sectionLabel(LegitL, "Speed")
createToggle(LegitL, "Speed Hack", false, function(v) State.SpeedHack = v end, "SpeedHack")
createSlider(LegitL, "Speed Value", 16, 200, 32, function(v) State.SpeedValue = v end, false, "SpeedValue")
createDropdown(LegitL, "Speed Method", {"Humanoid", "Velocity", "CFrame"}, "Humanoid", function(v) State.SpeedMethod = v end, "SpeedMethod")

sectionLabel(LegitL, "Jump")
createToggle(LegitL, "Jump Hack", false, function(v) State.JumpHack = v end, "JumpHack")
createSlider(LegitL, "Jump Value", 50, 500, 50, function(v) State.JumpValue = v end, false, "JumpValue")

sectionLabel(LegitL, "Movement")
createToggle(LegitL, "Bhop & Strafe", false, function(v) State.bhop = v end, "bhop")
createToggle(LegitL, "Fly", false, function(v)
    State.Fly = v
    if v then startFly() else stopFly() end
end, "Fly")
createSlider(LegitL, "Fly Speed", 10, 500, 100, function(v) State.FlySpeed = v end, false, "FlySpeed")

sectionLabel(LegitR, "Protection")
createToggle(LegitR, "Noclip", false, function(v) State.Noclip = v end, "Noclip")
createToggle(LegitR, "Infinite Jump", false, function(v) State.InfiniteJump = v end, "InfiniteJump")
createToggle(LegitR, "Anti Void", false, function(v) State.AntiVoid = v end, "AntiVoid")
createToggle(LegitR, "Anti Fling", false, function(v) State.AntiFling = v end, "AntiFling")

sectionLabel(LegitR, "Camera")
createToggle(LegitR, "Third Person", false, function(v) State.ThirdPerson = v end, "ThirdPerson")
createSlider(LegitR, "TP Distance", 4, 50, 15, function(v) State.TP_Distance = v end, false, "TP_Distance")

-- ========== WORLD ==========
sectionLabel(WorldL, "Skybox")
local skyNames = {}
for _, s in ipairs(SKYBOX_PRESETS) do table.insert(skyNames, s.Name) end
createDropdown(WorldL, "Skybox Preset", skyNames, "Custom ID", function(v)
    State.Skybox = v; applySkybox(v)
end, "Skybox")

sectionLabel(WorldL, "Fog")
createToggle(WorldL, "Custom Fog", false, function(v) State.customFog = v; applyFog() end, "customFog")
createSlider(WorldL, "Fog Distance", 50, 1000, 250, function(v) State.fogDistance = v; applyFog() end, false, "fogDistance")
createSlider(WorldL, "Fog Density", 0, 100, 30, function(v) State.fogDensity = v/100; applyFog() end, false, "fogDensity")
createColorRow(WorldL, "Fog Color", Color3.fromRGB(140, 100, 220), function(c) State.fogColor = c; applyFog() end, "fogColor")

sectionLabel(WorldL, "Time")
createSlider(WorldL, "Clock Time", 0, 24, 14, function(v) Lighting.ClockTime = v end, true, "ClockTime")

sectionLabel(WorldR, "Fling")
createDropdown(WorldR, "Fling Mode", {"Rage", "Normal"}, "Rage", function(v) State.FlingMode = v end, "FlingMode")
createToggle(WorldR, "Auto Fling Murderer", false, function(v) State.AutoFlingMurderer = v end, "AutoFlingMurderer")
createToggle(WorldR, "Auto Fling Sheriff", false, function(v) State.AutoFlingSheriff = v end, "AutoFlingSheriff")
createToggle(WorldR, "Auto Fling Hero", false, function(v) State.AutoFlingHero = v end, "AutoFlingHero")

-- ========== MISC ==========
sectionLabel(MiscL, "Screen Options")
createToggle(MiscL, "Blur on Menu", false, function(v)
    State.BlurEnabled = v
    local oldBlur = Lighting:FindFirstChild("SVEY_Blur")
    if v then
        if not oldBlur then
            local b = Instance.new("BlurEffect", Lighting)
            b.Name = "SVEY_Blur"; b.Size = 0
        end
    else
        if oldBlur then oldBlur:Destroy() end
    end
end, "BlurEnabled")
createToggle(MiscL, "Stretch Screen", false, function(v)
    State.StretchEnabled = v
    Camera.FieldOfView = v and 100 or 70
end, "StretchEnabled")
createDropdown(MiscL, "Language", {"RU", "EN", "UA"}, "RU", function(v)
    State.Language = v
    SendNotification("Language", "Выбран: " .. v, 2)
end, "Language")

-- ========== CONFIG ==========
local Configs = {}
local HttpService = game:GetService("HttpService")

local function serializeConfig()
    local data = {}
    for k, v in pairs(State) do
        if type(v) == "boolean" or type(v) == "number" or type(v) == "string" then
            data[k] = v
        end
    end
    return data
end

local function syncUIToState()
    for key, ctrl in pairs(UI.toggles) do
        if State[key] ~= nil then ctrl.set(State[key]) end
    end
    for key, ctrl in pairs(UI.sliders) do
        if State[key] ~= nil then ctrl.set(State[key]) end
    end
    for key, ctrl in pairs(UI.dropdowns) do
        if State[key] ~= nil then ctrl.set(State[key]) end
    end
    for key, swatch in pairs(UI.colors) do
        if typeof(State[key]) == "Color3" then
            swatch.BackgroundColor3 = State[key]
        end
    end
end

local function applyConfig(data)
    for k, v in pairs(data) do
        if State[k] ~= nil and type(State[k]) == type(v) then
            State[k] = v
        end
    end
    syncUIToState()
    SendNotification("Config", "Конфиг загружен!", 2)
end

sectionLabel(CfgL, "Save")
for i = 1, 3 do
    createButton(CfgL, "Save Config #" .. i, THEME.Accent, function()
        Configs[tostring(i)] = serializeConfig()
        pcall(function()
            if writefile then
                writefile("SVEY_config" .. i .. ".json", HttpService:JSONEncode(Configs[tostring(i)]))
            end
        end)
        SendNotification("Config", "Сохранено в слот " .. i, 2)
    end)
end

sectionLabel(CfgR, "Load")
for i = 1, 3 do
    createButton(CfgR, "Load Config #" .. i, THEME.Accent, function()
        local data = Configs[tostring(i)]
        if not data then
            pcall(function()
                if readfile and isfile and isfile("SVEY_config" .. i .. ".json") then
                    data = HttpService:JSONDecode(readfile("SVEY_config" .. i .. ".json"))
                end
            end)
        end
        if data then applyConfig(data)
        else SendNotification("Config", "Слот " .. i .. " пуст", 2) end
    end)
end

local infoText = Instance.new("TextLabel", CfgR)
infoText.Size = UDim2.new(1, 0, 0, 100)
infoText.BackgroundColor3 = Color3.fromRGB(20, 17, 28)
infoText.BorderSizePixel = 0
infoText.Text = "━━━━━━━━━━━━━━━━\n\n⚠️ Скрипт сделан ИИ\n\n✅ Полностью БЕСПЛАТНЫЙ\n\n━━━━━━━━━━━━━━━━"
infoText.TextColor3 = Color3.fromRGB(255, 200, 60)
infoText.Font = Enum.Font.GothamBold; infoText.TextSize = 12
infoText.TextXAlignment = Enum.TextXAlignment.Center
infoText.TextYAlignment = Enum.TextYAlignment.Center
infoText.TextWrapped = true; infoText.LayoutOrder = 999
Instance.new("UICorner", infoText).CornerRadius = UDim.new(0, 6)
local is2 = Instance.new("UIStroke", infoText); is2.Color = THEME.Accent; is2.Thickness = 1.5

-- ========== HITBOX ==========
local Hitboxes = {}
local HitboxFolder = Instance.new("Folder", workspace)
HitboxFolder.Name = "SVEY_Hitboxes"

local function createHitbox(name, parentPart)
    local hb = Instance.new("Part")
    hb.Name = "SVEY_HB_"..name
    hb.Size = parentPart.Size * State.HitboxSize
    hb.Transparency = State.HitboxTransparency
    hb.CanCollide = false; hb.CanQuery = true; hb.CanTouch = false
    hb.Massless = true; hb.Anchored = false
    hb.Material = Enum.Material.ForceField
    hb.Color = State.hitboxColor or Color3.fromRGB(185,115,255)
    hb.Parent = HitboxFolder
    local weld = Instance.new("Weld", hb)
    weld.Part0 = parentPart; weld.Part1 = hb; weld.C0 = CFrame.new()
    return hb
end

local function createHitboxesForPlayer(plr)
    if Hitboxes[plr] then return Hitboxes[plr] end
    local char = plr.Character; if not char then return nil end
    local head = char:FindFirstChild("Head")
    local torso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not (head and hrp) then return nil end
    local data = {}
    if head  then data.head  = createHitbox("Head",  head)  end
    if torso then data.torso = createHitbox("Torso", torso) end
    if hrp   then data.hrp   = createHitbox("HRP",   hrp)   end
    Hitboxes[plr] = data
    return data
end

local function removeHitboxes(plr)
    if Hitboxes[plr] then
        for _, hb in pairs(Hitboxes[plr]) do
            if hb and hb.Parent then hb:Destroy() end
        end
        Hitboxes[plr] = nil
    end
end

function updateAllHitboxes()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            createHitboxesForPlayer(plr)
        end
    end
end

function updateHitboxVisibility()
    for _, hbs in pairs(Hitboxes) do
        for _, hb in pairs(hbs) do
            if hb and hb.Parent then
                if State.HitboxExpander then
                    hb.Size = hb.Parent.Size * State.HitboxSize
                    hb.Transparency = State.HitboxTransparency
                    hb.CanQuery = true
                else
                    hb.Size = hb.Parent.Size
                    hb.Transparency = 1
                    hb.CanQuery = false
                end
            end
        end
    end
end

Players.PlayerAdded:Connect(function(p)
    task.wait(1)
    if State.HitboxExpander then createHitboxesForPlayer(p) end
end)
Players.PlayerRemoving:Connect(removeHitboxes)

RunService.Heartbeat:Connect(function()
    if not State.HitboxExpander then return end
    pcall(function()
        for _, hbs in pairs(Hitboxes) do
            for _, hb in pairs(hbs) do
                if hb and hb.Parent and hb.Parent.Parent then
                    hb.Size = hb.Parent.Size * State.HitboxSize
                    hb.Transparency = State.HitboxTransparency
                    hb.Color = State.hitboxColor or Color3.fromRGB(185,115,255)
                end
            end
        end
    end)
end)

-- ========== FOV CIRCLE ==========
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false; FOVCircle.Thickness = 2
FOVCircle.NumSides = 64; FOVCircle.Radius = 150
FOVCircle.Color = Color3.fromRGB(255, 60, 60)
FOVCircle.Transparency = 0.8; FOVCircle.Filled = false
FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

RunService.RenderStepped:Connect(function()
    if State.silentAim and State.showFov then
        FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
        FOVCircle.Radius = State.silentAimFOV or 150
        FOVCircle.Color = State.fovColor or Color3.fromRGB(255, 60, 60)
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end)

-- ========== SILENT AIM (реальный хук) ==========
local aimTarget = nil

RunService.RenderStepped:Connect(function()
    if not State.silentAim and not State.Aimbot then aimTarget = nil; return end
    local target = nil
    local shortest = State.silentAimFOV or State.AimbotFOV or 200
    local center = Camera.ViewportSize / 2
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local targetPart = plr.Character:FindFirstChild("Head")
                if State.silentAimTargetPart == "Torso" then
                    targetPart = plr.Character:FindFirstChild("Torso")
                              or plr.Character:FindFirstChild("UpperTorso") or targetPart
                elseif State.silentAimTargetPart == "HRP" then
                    targetPart = plr.Character:FindFirstChild("HumanoidRootPart") or targetPart
                end
                if targetPart then
                    local ePos, vis = Camera:WorldToViewportPoint(targetPart.Position)
                    if vis and ePos.Z > 0 then
                        local d = (Vector2.new(ePos.X, ePos.Y) - center).Magnitude
                        if d < shortest then shortest = d; target = targetPart end
                    end
                end
            end
        end
    end
    aimTarget = target
    Effects.SilentAimTarget = target
    if State.Aimbot and target then
        local newCF = CFrame.new(Camera.CFrame.Position, target.Position)
        Camera.CFrame = Camera.CFrame:Lerp(newCF, State.AimbotSmoothness or 0.5)
    end
end)

-- Хук стрельбы (Silent Aim). Работает, если executor даёт getrawmetatable/hookmetamethod
pcall(function()
    if not (getrawmetatable and setreadonly and hookmetamethod) then return end
    local mt = getrawmetatable(game)
    local oldNamecall = mt.__namecall
    setreadonly(mt, false)
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if State.silentAim and aimTarget and aimTarget.Parent then
            if (method == "FireServer" or method == "InvokeServer") then
                local args = {...}
                for i, v in ipairs(args) do
                    if typeof(v) == "CFrame" then
                        args[i] = CFrame.new(v.Position, aimTarget.Position)
                    end
                end
                return oldNamecall(self, unpack(args))
            end
        end
        return oldNamecall(self, ...)
    end)
    setreadonly(mt, true)
end)

-- ========== ANTI-AIM (через AlignOrientation — не ломает физику) ==========
local aaAngle = 0
local aaAttachment, aaAlign, aaAttach2

local function setupAA()
    local char = Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    if aaAlign and aaAlign.Parent then aaAlign:Destroy() end
    if aaAttachment and aaAttachment.Parent then aaAttachment:Destroy() end
    if aaAttach2 and aaAttach2.Parent then aaAttach2:Destroy() end

    aaAttachment = Instance.new("Attachment", hrp)
    aaAttach2    = Instance.new("Attachment", hrp)
    aaAttach2.Position = Vector3.new(0, 0, 0.1)
    aaAlign = Instance.new("AlignOrientation", hrp)
    aaAlign.Attachment0 = aaAttachment
    aaAlign.Attachment1 = aaAttach2
    aaAlign.Mode = Enum.OrientationAlignmentMode.OneAttachment
    aaAlign.RigidityEnabled = true
    aaAlign.ReactionTorqueEnabled = true
    aaAlign.MaxTorque = 999999
    aaAlign.Responsiveness = 200
end

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.3)
    if State.AntiAim then setupAA() end
end)

RunService.RenderStepped:Connect(function(dt)
    if not State.AntiAim then
        if aaAlign then aaAlign.Enabled = false end
        return
    end
    local hrp = Character and Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    if not aaAlign or not aaAlign.Parent then setupAA() end
    if not aaAlign then return end
    aaAlign.Enabled = true

    if State.AntiAimMode == "Spin" then
        aaAngle = (aaAngle + State.AntiAimSpeed * dt) % 360
        aaAlign.CFrame = CFrame.Angles(0, math.rad(aaAngle), 0)
    elseif State.AntiAimMode == "Jitter" then
        aaAlign.CFrame = CFrame.Angles(0, math.rad(math.random(-State.AntiAimSpeed, State.AntiAimSpeed)), 0)
    elseif State.AntiAimMode == "Random" then
        aaAlign.CFrame = CFrame.Angles(0, math.rad(math.random(0, 360)), 0)
    elseif State.AntiAimMode == "Desync" then
        aaAngle = (aaAngle + State.AntiAimSpeed * 2 * dt) % 360
        aaAlign.CFrame = CFrame.Angles(0, math.rad(-aaAngle), 0)
    end
end)

-- ========== SPEED / JUMP / BHOP ==========
RunService.RenderStepped:Connect(function(dt)
    if not State.SpeedHack then return end
    pcall(function()
        local char = Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not (hrp and hum) then return end
        local moveDir = hum.MoveDirection
        if State.SpeedMethod == "Humanoid" then
            hum.WalkSpeed = State.SpeedValue
        elseif State.SpeedMethod == "Velocity" then
            if moveDir.Magnitude > 0 then
                hrp.Velocity = Vector3.new(moveDir.X * State.SpeedValue, hrp.Velocity.Y, moveDir.Z * State.SpeedValue)
            end
        elseif State.SpeedMethod == "CFrame" then
            if moveDir.Magnitude > 0 then
                hrp.CFrame = hrp.CFrame + (moveDir * (State.SpeedValue * dt))
            end
        end
    end)
end)

RunService.Heartbeat:Connect(function()
    if State.JumpHack then
        local hum = Character and Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.UseJumpPower = true; hum.JumpPower = State.JumpValue end
    end
end)

RunService.Heartbeat:Connect(function(dt)
    if not State.bhop then return end
    local char = Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not (hrp and hum) then return end
    local moveDir = hum.MoveDirection
    if hum.FloorMaterial ~= Enum.Material.Air then
        if moveDir.Magnitude > 0 then hum.Jump = true end
    else
        if moveDir.Magnitude > 0 then
            if State.strafeType == "Velocity" then
                hrp.Velocity = Vector3.new(moveDir.X * State.SpeedValue, hrp.Velocity.Y, moveDir.Z * State.SpeedValue)
            else
                local forward = Camera.CFrame.LookVector
                local flat = Vector3.new(forward.X, 0, forward.Z).Unit
                hrp.Velocity = Vector3.new(flat.X * State.SpeedValue, hrp.Velocity.Y, flat.Z * State.SpeedValue)
                hrp.CFrame = hrp.CFrame + (moveDir * 0.15)
            end
        end
    end
end)

-- ========== FLY ==========
local flyVel, flyGyro, flyConn
function startFly()
    if flyConn then flyConn:Disconnect() end
    local char = Character; if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum then return end
    flyVel = Instance.new("BodyVelocity", hrp)
    flyVel.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    flyVel.Velocity = Vector3.zero
    flyGyro = Instance.new("BodyGyro", hrp)
    flyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    flyGyro.P = 9e4; flyGyro.CFrame = hrp.CFrame
    hum.PlatformStand = true
    flyConn = RunService.RenderStepped:Connect(function()
        pcall(function()
            if not State.Fly then return end
            local cHrp = Character and Character:FindFirstChild("HumanoidRootPart")
            if not cHrp then return end
            local moveDir = Vector3.zero
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0,1,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir = moveDir - Vector3.new(0,1,0) end
            if moveDir.Magnitude > 0 then moveDir = moveDir.Unit end
            flyVel.Velocity = moveDir * State.FlySpeed
            flyGyro.CFrame = Camera.CFrame
        end)
    end)
end
function stopFly()
    if flyConn then flyConn:Disconnect() flyConn = nil end
    if flyVel then flyVel:Destroy() flyVel = nil end
    if flyGyro then flyGyro:Destroy() flyGyro = nil end
    local hum = Character and Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.PlatformStand = false end
end

-- ========== THIRD PERSON ==========
local tpActive, tpConn = false, nil
local function enableTP()
    if tpActive then return end
    tpActive = true
    LocalPlayer.CameraMode = Enum.CameraMode.Classic
    LocalPlayer.CameraMaxZoomDistance = State.TP_Distance or 15
    LocalPlayer.CameraMinZoomDistance = 4
    if tpConn then tpConn:Disconnect() end
    tpConn = RunService.RenderStepped:Connect(function()
        local char = Character; if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            Camera.CameraSubject = hum
            Camera.CameraType = Enum.CameraType.Custom
        end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            local camPos = Camera.CFrame.Position
            local charPos = hrp.Position + Vector3.new(0, 1.5, 0)
            local dist = (camPos - charPos).Magnitude
            local targetDist = State.TP_Distance or 15
            if dist < targetDist - 0.5 then
                local dir = (camPos - charPos).Unit
                Camera.CFrame = CFrame.new(charPos + dir * targetDist, charPos)
            end
        end
    end)
end
local function disableTP()
    if not tpActive then return end
    tpActive = false
    if tpConn then tpConn:Disconnect() tpConn = nil end
    LocalPlayer.CameraMode = Enum.CameraMode.Classic
    LocalPlayer.CameraMaxZoomDistance = 128
    LocalPlayer.CameraMinZoomDistance = 0.5
end
RunService.RenderStepped:Connect(function()
    if State.ThirdPerson and not tpActive then enableTP()
    elseif not State.ThirdPerson and tpActive then disableTP() end
end)

-- ========== NOCLIP / VOID / FLING-PROTECT ==========
RunService.Stepped:Connect(function()
    if not State.Noclip then return end
    pcall(function()
        if Character then
            for _, p in ipairs(Character:GetDescendants()) do
                if p:IsA("BasePart") and p.CanCollide then p.CanCollide = false end
            end
        end
    end)
end)
UserInputService.JumpRequest:Connect(function()
    if not State.InfiniteJump then return end
    local hum = Character and Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.Jump = true end
end)
local lastSafePos = Vector3.new(0, 5, 0)
RunService.Heartbeat:Connect(function()
    if not State.AntiVoid then return end
    pcall(function()
        local hrp = Character and Character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        if hrp.Position.Y > -20 then
            lastSafePos = hrp.Position
        elseif hrp.Position.Y < -40 then
            hrp.CFrame = CFrame.new(lastSafePos + Vector3.new(0, 4, 0))
            hrp.Velocity = Vector3.zero
        end
    end)
end)
RunService.Stepped:Connect(function()
    if not State.AntiFling then return end
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            for _, part in ipairs(plr.Character:GetChildren()) do
                if part:IsA("BasePart") then part.CanCollide = false end
            end
        end
    end
end)

-- ========== AURA ==========
function buildAnimeAura()
    local char = Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local f = Instance.new("ParticleEmitter", hrp)
    f.Name = "AnimeBodyFlame"
    f.Texture = "rbxassetid://243098098"
    f.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255,255,255)),
        ColorSequenceKeypoint.new(0.35, State.auraColor),
        ColorSequenceKeypoint.new(1, State.auraColor)
    })
    f.LightEmission = 1; f.LightInfluence = 0
    f.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1.6),
        NumberSequenceKeypoint.new(0.5, 3.6),
        NumberSequenceKeypoint.new(1, 0)
    })
    f.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.4),
        NumberSequenceKeypoint.new(0.65, 0.7),
        NumberSequenceKeypoint.new(1, 1)
    })
    f.Lifetime = NumberRange.new(0.45, 0.65)
    f.Rate = 45; f.Speed = NumberRange.new(3, 5.5)
    f.SpreadAngle = Vector2.new(12, 12)
    table.insert(auraObjects.emitters, f)

    local gAtt = Instance.new("Attachment", hrp)
    gAtt.Name = "GroundAuraAtt"; gAtt.Position = Vector3.new(0, -2.8, 0)
    table.insert(auraObjects.emitters, gAtt)
    local wave = Instance.new("ParticleEmitter", gAtt)
    wave.Texture = "rbxassetid://1084991219"
    wave.Color = ColorSequence.new(State.auraColor)
    wave.LightEmission = 1; wave.LightInfluence = 0
    wave.Orientation = Enum.ParticleOrientation.VelocityPerpendicular
    wave.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1.5),
        NumberSequenceKeypoint.new(1, 9.5)
    })
    wave.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(0.7, 0.5),
        NumberSequenceKeypoint.new(1, 1)
    })
    wave.Lifetime = NumberRange.new(0.5, 0.8); wave.Rate = 8
    table.insert(auraObjects.emitters, wave)

    local ring = Instance.new("Part", workspace)
    ring.Name = "SVEY_FloorRing"
    ring.Size = Vector3.new(7.5, 0.05, 7.5)
    ring.CFrame = hrp.CFrame * CFrame.new(0, -2.75, 0)
    ring.Anchored = true; ring.CanCollide = false
    ring.Material = Enum.Material.Neon
    ring.Color = State.auraColor; ring.Transparency = 0.45
    local m = Instance.new("SpecialMesh", ring)
    m.MeshType = Enum.MeshType.Cylinder
    auraObjects.floorRing = ring
end
function cleanAnimeAura()
    for _, item in ipairs(auraObjects.emitters) do
        if item then item:Destroy() end
    end
    table.clear(auraObjects.emitters)
    if auraObjects.floorRing then
        auraObjects.floorRing:Destroy()
        auraObjects.floorRing = nil
    end
end
RunService.RenderStepped:Connect(function()
    if not State.aura then return end
    local hrp = Character and Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    if not auraObjects.floorRing then buildAnimeAura() end
    if auraObjects.floorRing then
        auraObjects.floorRing.CFrame = CFrame.new(hrp.Position - Vector3.new(0, 2.75, 0))
                                       * CFrame.Angles(0, tick()*3, 0)
        auraObjects.floorRing.Color = State.auraColor
    end
end)

function buildWings()
    local torso = Character and (Character:FindFirstChild("UpperTorso")
                  or Character:FindFirstChild("Torso"))
    if not torso or auraObjects.wings then return end
    local mdl = Instance.new("Model", Character); mdl.Name = "SVEY_Wings"
    for side = -1, 1, 2 do
        local w = Instance.new("Part", mdl)
        w.Size = Vector3.new(2.8, 1.8, 0.1)
        w.Material = Enum.Material.Neon
        w.Color = State.selfWingsColor; w.Transparency = 0.25
        w.CanCollide = false
        local mesh = Instance.new("SpecialMesh", w)
        mesh.MeshType = Enum.MeshType.Wedge
        mesh.Scale = Vector3.new(side, 1, 1)
        local weld = Instance.new("Weld", w)
        weld.Part0 = torso; weld.Part1 = w
        weld.C0 = CFrame.new(side*1.6, 0.6, 0.8)
                * CFrame.Angles(math.rad(15), math.rad(side*25), math.rad(side*-18))
    end
    auraObjects.wings = mdl
end
function cleanWings()
    if auraObjects.wings then
        auraObjects.wings:Destroy()
        auraObjects.wings = nil
    end
end

-- ========== CHAMS ==========
local function applyChams(char)
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            if not chamsOriginals[part] then
                chamsOriginals[part] = {
                    material = part.Material,
                    color = part.Color,
                    transparency = part.Transparency,
                }
            end
            part.Material = Enum.Material.ForceField
            part.Color = State.glassChamsColor
            part.Transparency = State.ChamsTransparency
        end
    end
end
function restoreChams()
    for part, orig in pairs(chamsOriginals) do
        if part and part.Parent then
            part.Material = orig.material
            part.Color = orig.color
            part.Transparency = orig.transparency
        end
    end
    table.clear(chamsOriginals)
end
RunService.RenderStepped:Connect(function()
    if not State.glassChams then
        if next(chamsOriginals) then restoreChams() end
        return
    end
    if State.glassChamsAll then
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Character then applyChams(p.Character) end
        end
    elseif LocalPlayer.Character then
        applyChams(LocalPlayer.Character)
    end
end)

-- ========== HAT ==========
function clearHat()
    for _, p in ipairs(Effects.HatModel or {}) do
        if p and p.Parent then p:Destroy() end
    end
    Effects.HatModel = {}
end
function applyHat()
    if not Character then return end
    clearHat()
    local head = Character:FindFirstChild("Head"); if not head then return end
    local size = State.HatSize or 1
    local color = State.HatColor or Color3.fromRGB(100, 255, 150)
    local c = Instance.new("Part", Character)
    c.Name = "SVEY_Hat"
    c.Size = Vector3.new(2.5*size, 1.5*size, 2.5*size)
    c.Anchored = false; c.CanCollide = false; c.Color = color
    local m = Instance.new("SpecialMesh", c)
    m.MeshType = Enum.MeshType.FileMesh
    m.MeshId = "rbxassetid://1033714"
    m.Scale = Vector3.new(1.2*size, 1*size, 1.2*size)
    local w = Instance.new("Weld", c)
    w.Part0 = head; w.Part1 = c; w.C0 = CFrame.new(0, 1.0, 0)
    Effects.HatModel = {c}
end

-- ========== DICK (НОВЫЙ, РАБОЧИЙ: палка + головка + 2 яйца) ==========
function clearDick()
    for _, p in ipairs(Effects.DickParts or {}) do
        if p and p.Parent then p:Destroy() end
    end
    Effects.DickParts = {}
end

function applyDick()
    if not Character then return end
    clearDick()
    if not State.Dick then return end

    local torso = Character:FindFirstChild("Torso")
               or Character:FindFirstChild("UpperTorso")
    if not torso then return end

    local size  = State.DickSize or 1
    local color = State.DickColor or Color3.fromRGB(255, 200, 150)
    -- Цвет яиц чуть темнее — «как надо»
    local ballsColor = Color3.new(
        math.clamp(color.R * 0.85, 0, 1),
        math.clamp(color.G * 0.75, 0, 1),
        math.clamp(color.B * 0.75, 0, 1)
    )

    -- 1) ПАЛКА (цилиндр) — торчит вперёд из паха
    local shaft = Instance.new("Part", Character)
    shaft.Name = "SVEY_Dick_Shaft"
    shaft.Size = Vector3.new(0.7*size, 2.2*size, 0.7*size)
    shaft.Anchored = false; shaft.CanCollide = false
    shaft.Material = Enum.Material.SmoothPlastic
    shaft.Color = color; shaft.Massless = true
    local shaftMesh = Instance.new("SpecialMesh", shaft)
    shaftMesh.MeshType = Enum.MeshType.Cylinder
    local shaftWeld = Instance.new("Weld", shaft)
    shaftWeld.Part0 = torso
    shaftWeld.Part1 = shaft
    -- от паха вперёд (Roblox смотрит в -Z)
    shaftWeld.C0 = CFrame.new(0, -0.3*size, -1.3*size)

    -- 2) ГОЛОВКА (шар) — на конце палки
    local headPart = Instance.new("Part", Character)
    headPart.Name = "SVEY_Dick_Head"
    headPart.Shape = Enum.PartType.Ball
    headPart.Size = Vector3.new(0.9*size, 0.9*size, 0.9*size)
    headPart.Anchored = false; headPart.CanCollide = false
    headPart.Material = Enum.Material.SmoothPlastic
    headPart.Color = color; headPart.Massless = true
    local headWeld = Instance.new("Weld", headPart)
    headWeld.Part0 = shaft
    headWeld.Part1 = headPart
    -- на конце палки (сверху цилиндра в его локальных координатах Y)
    headWeld.C0 = CFrame.new(0, 1.15*size, 0)

    -- 3) ДВА ЯЙЦА у паха (сферы по бокам основания палки)
    local ball1 = Instance.new("Part", Character)
    ball1.Name = "SVEY_Dick_Ball1"
    ball1.Shape = Enum.PartType.Ball
    ball1.Size = Vector3.new(0.65*size, 0.65*size, 0.65*size)
    ball1.Anchored = false; ball1.CanCollide = false
    ball1.Material = Enum.Material.SmoothPlastic
    ball1.Color = ballsColor; ball1.Massless = true
    local ballW1 = Instance.new("Weld", ball1)
    ballW1.Part0 = torso
    ballW1.Part1 = ball1
    -- левое яйцо (X = -1) чуть позади основания палки
    ballW1.C0 = CFrame.new(-0.4*size, -0.7*size, -0.7*size)

    local ball2 = Instance.new("Part", Character)
    ball2.Name = "SVEY_Dick_Ball2"
    ball2.Shape = Enum.PartType.Ball
    ball2.Size = Vector3.new(0.65*size, 0.65*size, 0.65*size)
    ball2.Anchored = false; ball2.CanCollide = false
    ball2.Material = Enum.Material.SmoothPlastic
    ball2.Color = ballsColor; ball2.Massless = true
    local ballW2 = Instance.new("Weld", ball2)
    ballW2.Part0 = torso
    ballW2.Part1 = ball2
    -- правое яйцо (X = +1)
    ballW2.C0 = CFrame.new(0.4*size, -0.7*size, -0.7*size)

    Effects.DickParts = { shaft, headPart, ball1, ball2 }
    SendNotification("Dick", "Комплект установлен 😏", 2)
end

-- ========== FOG ==========
local defaultLighting = {
    FogStart = Lighting.FogStart,
    FogEnd = Lighting.FogEnd,
    FogColor = Lighting.FogColor,
}
function applyFog()
    pcall(function()
        if State.customFog then
            local oldAtm = Lighting:FindFirstChild("SVEY_Atm")
            if oldAtm then oldAtm:Destroy() end
            Lighting.FogStart = 0
            Lighting.FogEnd = State.fogDistance or 250
            Lighting.FogColor = State.fogColor or Color3.fromRGB(140, 100, 220)
            local atm = Instance.new("Atmosphere", Lighting)
            atm.Name = "SVEY_Atm"
            atm.Density = State.fogDensity or 0.3
            atm.Color = State.fogColor or Color3.fromRGB(140, 100, 220)
        else
            Lighting.FogStart = defaultLighting.FogStart
            Lighting.FogEnd = defaultLighting.FogEnd
            Lighting.FogColor = defaultLighting.FogColor
            local atm = Lighting:FindFirstChild("SVEY_Atm")
            if atm then atm:Destroy() end
        end
    end)
end
RunService.Heartbeat:Connect(function()
    if not State.customFog then return end
    pcall(function()
        Lighting.FogStart = 0
        Lighting.FogEnd = State.fogDistance or 250
        Lighting.FogColor = State.fogColor
        local atm = Lighting:FindFirstChild("SVEY_Atm")
        if atm then atm.Density = State.fogDensity end
    end)
end)

function applySkybox(name)
    local preset
    for _, p in ipairs(SKYBOX_PRESETS) do
        if p.Name == name then preset = p; break end
    end
    if not preset then return end
    pcall(function()
        Lighting.Ambient = preset.Ambient
        Lighting.OutdoorAmbient = preset.Outdoor
        Lighting.ClockTime = preset.ClockTime
        Lighting.Brightness = preset.Brightness
        if preset.CustomId then
            local sky = Lighting:FindFirstChildOfClass("Sky") or Instance.new("Sky", Lighting)
            sky.SkyboxBk = preset.CustomId; sky.SkyboxDn = preset.CustomId
            sky.SkyboxFt = preset.CustomId; sky.SkyboxLf = preset.CustomId
            sky.SkyboxRt = preset.CustomId; sky.SkyboxUp = preset.CustomId
        end
    end)
end

-- ========== ESP ЦИКЛ ==========
local r15Limbs = {
    {"Head","UpperTorso"},{"UpperTorso","LowerTorso"},
    {"UpperTorso","LeftUpperArm"},{"LeftUpperArm","LeftLowerArm"},{"LeftLowerArm","LeftHand"},
    {"UpperTorso","RightUpperArm"},{"RightUpperArm","RightLowerArm"},{"RightLowerArm","RightHand"},
    {"LowerTorso","LeftUpperLeg"},{"LeftUpperLeg","LeftLowerLeg"},{"LeftLowerLeg","LeftFoot"},
    {"LowerTorso","RightUpperLeg"},{"RightUpperLeg","RightLowerLeg"},{"RightLowerLeg","RightFoot"},
}
local r6Limbs = {
    {"Head","Torso"},{"Torso","Left Arm"},{"Torso","Right Arm"},
    {"Torso","Left Leg"},{"Torso","Right Leg"},
}

local function getHealthColor(a)
    a = math.clamp(a, 0, 1)
    if a >= 0.5 then
        return Color3.fromRGB(245,215,55):Lerp(Color3.fromRGB(55,225,105), (a-0.5)*2)
    else
        return Color3.fromRGB(240,50,60):Lerp(Color3.fromRGB(245,215,55), a*2)
    end
end

local function getOrCreate2D(player)
    if cached2D[player] then return cached2D[player] end
    local c = Instance.new("Folder", ESPContainer)
    c.Name = player.Name .. "_ESP"
    local box = Instance.new("Frame", c)
    box.BackgroundTransparency = 1; box.BorderSizePixel = 0; box.Visible = false
    local stroke = Instance.new("UIStroke", box); stroke.Thickness = 1.2
    local corners = {}
    for i = 1, 8 do
        local fr = Instance.new("Frame", box)
        fr.BorderSizePixel = 0
        table.insert(corners, fr)
    end
    local hpT = Instance.new("Frame", c)
    hpT.BackgroundColor3 = Color3.fromRGB(15,15,20)
    hpT.BorderSizePixel = 0; hpT.Visible = false
    local hpB = Instance.new("Frame", hpT); hpB.BorderSizePixel = 0
    local distL = Instance.new("TextLabel", c)
    distL.BackgroundTransparency = 1; distL.Font = Enum.Font.RobotoMono
    distL.TextSize = 10; distL.TextStrokeTransparency = 0.3; distL.Visible = false
    local roleL = Instance.new("TextLabel", c)
    roleL.BackgroundTransparency = 1; roleL.Font = Enum.Font.GothamBold
    roleL.TextSize = 10; roleL.TextStrokeTransparency = 0.3; roleL.Visible = false
    local snap = Instance.new("Frame", c)
    snap.AnchorPoint = Vector2.new(0.5,0.5); snap.BorderSizePixel = 0; snap.Visible = false
    local arrow = Instance.new("Frame", c)
    arrow.Size = UDim2.new(0,16,0,16); arrow.AnchorPoint = Vector2.new(0.5,0.5)
    arrow.BackgroundTransparency = 1; arrow.Visible = false
    local arrowL = Instance.new("TextLabel", arrow)
    arrowL.Size = UDim2.new(1,0,1,0); arrowL.BackgroundTransparency = 1
    arrowL.Font = Enum.Font.GothamBold; arrowL.Text = "▶"; arrowL.TextSize = 16
    local skel = {}
    for i = 1, 15 do
        local l = Instance.new("Frame", c)
        l.AnchorPoint = Vector2.new(0.5,0.5); l.BorderSizePixel = 0; l.Visible = false
        table.insert(skel, l)
    end
    local data = {
        container = c, box = box, stroke = stroke, corners = corners,
        hpTrack = hpT, hpBar = hpB, dist = distL, role = roleL,
        snapline = snap, arrow = arrow, arrowLabel = arrowL, skeletonLines = skel,
    }
    cached2D[player] = data
    return data
end

Players.PlayerRemoving:Connect(function(plr)
    if cached2D[plr] then
        cached2D[plr].container:Destroy()
        cached2D[plr] = nil
    end
end)

local boxOffsets = {
    Vector3.new(-1.8,2.7,-1.2), Vector3.new(1.8,2.7,-1.2),
    Vector3.new(-1.8,-2.7,-1.2),Vector3.new(1.8,-2.7,-1.2),
    Vector3.new(-1.8,2.7,1.2),  Vector3.new(1.8,2.7,1.2),
    Vector3.new(-1.8,-2.7,1.2), Vector3.new(1.8,-2.7,1.2),
}

local function applyHighlight(inst, color, tag)
    if not inst or not inst.Parent then return end
    local hl = inst:FindFirstChild(tag)
    if not hl then
        hl = Instance.new("Highlight"); hl.Name = tag; hl.Adornee = inst
        pcall(function() hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop end)
        hl.FillTransparency = 0.5; hl.OutlineTransparency = 0.1
        hl.Parent = inst
    end
    hl.FillColor = color; hl.OutlineColor = color; hl.Enabled = true
end
local function stripHL(inst, tag)
    if not inst then return end
    local hl = inst:FindFirstChild(tag)
    if hl then hl.Enabled = false end
end

RunService.RenderStepped:Connect(function()
    pcall(function()
        local char = LocalPlayer.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer then
                local tChar = plr.Character
                local tHrp = tChar and tChar:FindFirstChild("HumanoidRootPart")
                local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
                local ui = getOrCreate2D(plr)
                if tChar and tHrp and tHum and tHum.Health > 0 then
                    local role = resolveRole(plr)
                    local vis = State.espAll
                        or (State.espMurder and role == "Murderer")
                        or (State.espSheriff and role == "Sheriff")
                        or (State.espHero and role == "Hero")
                        or (State.espInnocents and role == "Innocent")
                    local rColor = getRoleColor(role)

                    if vis then applyHighlight(tChar, rColor, "SVEY_ESP")
                    else stripHL(tChar, "SVEY_ESP") end
                    if State.highlightEsp then applyHighlight(tChar, State.highlightColor, "SVEY_Custom")
                    else stripHL(tChar, "SVEY_Custom") end

                    if State.box3D then
                        local b3 = tChar:FindFirstChild("SVEY_3DWire")
                        if not b3 then
                            b3 = Instance.new("SelectionBox", tChar)
                            b3.Name = "SVEY_3DWire"
                            b3.LineThickness = 0.04
                            b3.SurfaceTransparency = 1
                            b3.Adornee = tChar
                        end
                        b3.Color3 = State.box3DColor; b3.Visible = true
                    end

                    -- Skeleton
                    if State.skeletonEsp then
                        local isR15 = tHum.RigType == Enum.HumanoidRigType.R15
                        local limbs = isR15 and r15Limbs or r6Limbs
                        for idx, line in ipairs(ui.skeletonLines) do
                            local pair = limbs[idx]
                            if pair then
                                local p1 = tChar:FindFirstChild(pair[1])
                                local p2 = tChar:FindFirstChild(pair[2])
                                if p1 and p2 then
                                    local s1, v1 = Camera:WorldToViewportPoint(p1.Position)
                                    local s2, v2 = Camera:WorldToViewportPoint(p2.Position)
                                    if v1 and v2 and s1.Z > 0 and s2.Z > 0 then
                                        local d = (Vector2.new(s2.X,s2.Y)-Vector2.new(s1.X,s1.Y)).Magnitude
                                        local mid = (Vector2.new(s1.X,s1.Y)+Vector2.new(s2.X,s2.Y))/2
                                        local ang = math.deg(math.atan2(s2.Y-s1.Y, s2.X-s1.X))
                                        line.Size = UDim2.new(0,d,0,1.4)
                                        line.Position = UDim2.new(0,mid.X,0,mid.Y)
                                        line.Rotation = ang
                                        line.BackgroundColor3 = State.skeletonColor
                                        line.Visible = true
                                    else line.Visible = false end
                                else line.Visible = false end
                            else line.Visible = false end
                        end
                    else
                        for _, l in ipairs(ui.skeletonLines) do l.Visible = false end
                    end

                    -- Box bounds
                    local minX, minY, maxX, maxY = math.huge, math.huge, -math.huge, -math.huge
                    local allVis = true
                    for _, off in ipairs(boxOffsets) do
                        local scr, v = Camera:WorldToViewportPoint(tHrp.Position + off)
                        if v and scr.Z > 0 then
                            minX = math.min(minX, scr.X); maxX = math.max(maxX, scr.X)
                            minY = math.min(minY, scr.Y); maxY = math.max(maxY, scr.Y)
                        else allVis = false; break end
                    end
                    local cScr, cVis = Camera:WorldToViewportPoint(tHrp.Position)

                    if allVis and (State.box2D or State.box2DMM2) then
                        local w = math.clamp(maxX-minX, 6, 2000)
                        local h = math.clamp(maxY-minY, 8, 2500)
                        local col = State.box2DMM2 and rColor or State.box2DColor
                        ui.box.Size = UDim2.new(0,w,0,h)
                        ui.box.Position = UDim2.new(0,minX,0,minY)
                        if State.box2DMode == "Corner" then
                            ui.stroke.Enabled = false
                            local cLen = math.clamp(w*0.25, 4, 14)
                            local cT = 1.5
                            for _, c in ipairs(ui.corners) do c.Visible = false end
                            local positions = {
                                {UDim2.new(0,0,0,0), UDim2.new(0,cLen,0,cT)},
                                {UDim2.new(0,0,0,0), UDim2.new(0,cT,0,cLen)},
                                {UDim2.new(1,-cLen,0,0), UDim2.new(0,cLen,0,cT)},
                                {UDim2.new(1,-cT,0,0), UDim2.new(0,cT,0,cLen)},
                                {UDim2.new(0,0,1,-cT), UDim2.new(0,cLen,0,cT)},
                                {UDim2.new(0,0,1,-cLen), UDim2.new(0,cT,0,cLen)},
                                {UDim2.new(1,-cLen,1,-cT), UDim2.new(0,cLen,0,cT)},
                                {UDim2.new(1,-cT,1,-cLen), UDim2.new(0,cT,0,cLen)},
                            }
                            for i, pos in ipairs(positions) do
                                ui.corners[i].Position = pos[1]
                                ui.corners[i].Size = pos[2]
                                ui.corners[i].BackgroundColor3 = col
                                ui.corners[i].Visible = true
                            end
                        else
                            ui.stroke.Enabled = true; ui.stroke.Color = col
                            for _, c in ipairs(ui.corners) do c.Visible = false end
                        end
                        ui.box.BackgroundColor3 = State.box2DFillColor
                        ui.box.BackgroundTransparency = State.box2DFill and 0.5 or 1
                        ui.box.Visible = true
                    else
                        ui.box.Visible = false
                        for _, c in ipairs(ui.corners) do c.Visible = false end
                    end

                    if State.roleEsp and allVis then
                        ui.role.Text = string.upper(role)
                        ui.role.TextColor3 = rColor
                        ui.role.Size = UDim2.new(0, maxX-minX+40, 0, 14)
                        ui.role.Position = UDim2.new(0, minX-20, 0, minY-16)
                        ui.role.Visible = true
                    else ui.role.Visible = false end

                    if State.healthBar and allVis then
                        local p = math.clamp(tHum.Health/tHum.MaxHealth, 0, 1)
                        local h = maxY-minY
                        ui.hpTrack.Size = UDim2.new(0,3,0,h)
                        ui.hpTrack.Position = UDim2.new(0,minX-7,0,minY)
                        ui.hpTrack.Visible = true
                        local fh = math.clamp(h*p, 0, h)
                        ui.hpBar.Size = UDim2.new(1,0,0,fh)
                        ui.hpBar.Position = UDim2.new(0,0,1,-fh)
                        ui.hpBar.BackgroundColor3 = getHealthColor(p)
                    else ui.hpTrack.Visible = false end

                    if State.distanceEsp and allVis and hrp then
                        local d = (hrp.Position - tHrp.Position).Magnitude
                        ui.dist.Text = string.format("[%dm]", math.floor(d))
                        ui.dist.TextColor3 = State.distanceColor
                        ui.dist.Size = UDim2.new(0, maxX-minX, 0, 14)
                        ui.dist.Position = UDim2.new(0, minX, 0, maxY+2)
                        ui.dist.Visible = true
                    else ui.dist.Visible = false end

                    if State.snaplines and cVis and cScr.Z > 0 then
                        local sc = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                        local fromP = State.snaplineOrigin == "Center"
                            and sc or Vector2.new(sc.X, Camera.ViewportSize.Y)
                        local toP = Vector2.new(cScr.X, maxY)
                        local d = (toP-fromP).Magnitude
                        local mid = (fromP+toP)/2
                        local ang = math.deg(math.atan2(toP.Y-fromP.Y, toP.X-fromP.X))
                        ui.snapline.Size = UDim2.new(0,d,0,1.2)
                        ui.snapline.Position = UDim2.new(0,mid.X,0,mid.Y)
                        ui.snapline.Rotation = ang
                        ui.snapline.BackgroundColor3 = State.snaplineColor
                        ui.snapline.Visible = true
                    else ui.snapline.Visible = false end

                    if State.arrowEsp then
                        if not cVis or cScr.Z <= 0 then
                            local sc = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                            local dir = (Vector2.new(cScr.X, cScr.Y) - sc).Unit
                            local bp = sc + (dir * math.min(sc.X-35, sc.Y-35))
                            local ang = math.deg(math.atan2(dir.Y, dir.X))
                            ui.arrow.Position = UDim2.new(0, bp.X, 0, bp.Y)
                            ui.arrow.Rotation = ang
                            ui.arrowLabel.TextColor3 = State.arrowEspColor
                            ui.arrow.Visible = true
                        else ui.arrow.Visible = false end
                    else ui.arrow.Visible = false end
                else
                    ui.box.Visible = false
                    ui.hpTrack.Visible = false
                    ui.dist.Visible = false
                    ui.role.Visible = false
                    ui.snapline.Visible = false
                    ui.arrow.Visible = false
                    for _, c in ipairs(ui.corners) do c.Visible = false end
                    for _, l in ipairs(ui.skeletonLines) do l.Visible = false end
                end
            end
        end
    end)
end)

-- ========== FLING ==========
local function executeFling(targetPlr)
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local tChar = targetPlr and targetPlr.Character
    local tHrp = tChar and tChar:FindFirstChild("HumanoidRootPart")
    local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
    if not (hrp and tHrp and hum and tHum and tHum.Health > 0) then return end
    local orig = hrp.CFrame
    local bav = Instance.new("BodyAngularVelocity", hrp)
    bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bav.P = 2500000
    bav.AngularVelocity = Vector3.new(200000, 200000, 200000)
    pcall(function() hum:ChangeState(Enum.HumanoidStateType.Physics) end)
    local start = tick(); local flip = 1
    while tick() - start < 1.3 and tHrp and tHrp.Parent and tHum.Health > 0 do
        local pred = tHrp.Position + (tHrp.Velocity * 0.14)
        flip = -flip
        if State.FlingMode == "Rage" then
            local so = tHrp.CFrame.RightVector * (flip * 3.5)
            hrp.CFrame = CFrame.new(pred + so + Vector3.new(0, math.random(-1,1)*0.5, 0))
            hrp.Velocity = (so.Unit * 80000) + Vector3.new(0, 15000, 0)
        else
            hrp.CFrame = CFrame.new(pred + Vector3.new(0,-0.4,0))
            hrp.Velocity = Vector3.new(50000, 50000, 50000)
        end
        RunService.Heartbeat:Wait()
    end
    bav:Destroy(); hrp.Velocity = Vector3.zero
    pcall(function()
        hrp.CFrame = orig + Vector3.new(0,3,0)
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
    end)
end

task.spawn(function()
    while task.wait(0.6) do
        if not flingDebounce then
            if State.AutoFlingMurderer then
                local m = getMurdererPlayer()
                if m then flingDebounce = true; executeFling(m); flingDebounce = false end
            elseif State.AutoFlingSheriff then
                local s = getSheriffPlayer()
                if s then flingDebounce = true; executeFling(s); flingDebounce = false end
            elseif State.AutoFlingHero then
                local h = getHeroPlayer()
                if h then flingDebounce = true; executeFling(h); flingDebounce = false end
            end
        end
    end
end)

-- ========== СТАРТ ==========
pcall(applySkybox, State.Skybox)
switchTab("Ragebot")

SendNotification("SVEY", "v61 CLEAN загружен!", 5)
print("[SVEY v61] Загружен. Mode: "..GameMode)
