-- ============================================
-- DUELIST HUB v1.3 - AUTO CLICK + STATS MELHORADOS
-- ============================================

local startTime = tick()
local validKey = "brxx"
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

local flying = false
local flySpeed = 50
local bodyGyro = nil
local bodyVelocity = nil

local antiAFKEnabled = false
local antiAFKConnection = nil

local autoClickEnabled = false
local clickSpeed = 10

-- Pega país (simulado)
function GetCountry()
    local s, r = pcall(function()
        return game:GetService("LocalizationService"):GetCountryRegionForPlayerAsync(player)
    end)
    if s and r then return r else return "Unknown" end
end

-- Pega players online no jogo
function GetPlayersInGame()
    return #game:GetService("Players"):GetPlayers()
end

-- Pega players no script (simulado - só você)
function GetScriptUsers()
    return 1
end

function GetUsageTime()
    local s = math.floor(tick() - startTime)
    local m = math.floor(s / 60)
    s = s % 60
    if m > 0 then return m .. "m " .. s .. "s" else return s .. "s" end
end

function GetExecutor()
    local s, r = pcall(function() return identifyexecutor() end)
    if s and r then return r else return "Unknown" end
end

function Notify(title, text, duration)
    local gui = Instance.new("ScreenGui", player.PlayerGui)
    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0, 300, 0, 60)
    frame.Position = UDim2.new(1, -310, 0, 20)
    frame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    frame.BorderSizePixel = 1
    frame.BorderColor3 = Color3.fromRGB(180, 20, 20)
    
    local redBar = Instance.new("Frame", frame)
    redBar.Size = UDim2.new(0, 4, 1, 0)
    redBar.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
    redBar.BorderSizePixel = 0
    
    local tl = Instance.new("TextLabel", frame)
    tl.Size = UDim2.new(1, -20, 0, 22)
    tl.Position = UDim2.new(0, 15, 0, 5)
    tl.BackgroundTransparency = 1
    tl.Text = title
    tl.Font = Enum.Font.GothamBlack
    tl.TextSize = 14
    tl.TextColor3 = Color3.fromRGB(255, 255, 255)
    tl.TextXAlignment = Enum.TextXAlignment.Left
    
    local txt = Instance.new("TextLabel", frame)
    txt.Size = UDim2.new(1, -20, 0, 20)
    txt.Position = UDim2.new(0, 15, 0, 30)
    txt.BackgroundTransparency = 1
    txt.Text = text
    txt.Font = Enum.Font.Gotham
    txt.TextSize = 11
    txt.TextColor3 = Color3.fromRGB(180, 180, 180)
    txt.TextXAlignment = Enum.TextXAlignment.Left
    
    frame:TweenPosition(UDim2.new(1, -310, 0, 20), "Out", "Quad", 0.3, true)
    
    spawn(function()
        wait(duration or 4)
        frame:TweenPosition(UDim2.new(1, 20, 0, 20), "In", "Quad", 0.3, true)
        wait(0.3)
        gui:Destroy()
    end)
end

-- ============================================
-- KEY SCREEN
-- ============================================
local KeyGui = Instance.new("ScreenGui", player.PlayerGui)

local KeyBg = Instance.new("Frame", KeyGui)
KeyBg.Size = UDim2.new(1, 0, 1, 0)
KeyBg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
KeyBg.BorderSizePixel = 0

local Box = Instance.new("Frame", KeyBg)
Box.Size = UDim2.new(0, 400, 0, 260)
Box.Position = UDim2.new(0.5, -200, 0.5, -130)
Box.BackgroundColor3 = Color3.fromRGB(8, 8, 8)
Box.BorderSizePixel = 2
Box.BorderColor3 = Color3.fromRGB(180, 20, 20)

local Logo = Instance.new("TextLabel", Box)
Logo.Size = UDim2.new(1, 0, 0, 35)
Logo.Position = UDim2.new(0, 0, 0, 18)
Logo.BackgroundTransparency = 1
Logo.Text = "DUELIST ALL SCRIPT"
Logo.Font = Enum.Font.GothamBlack
Logo.TextSize = 24
Logo.TextColor3 = Color3.fromRGB(255, 255, 255)

local SubLogo = Instance.new("TextLabel", Box)
SubLogo.Size = UDim2.new(1, 0, 0, 18)
SubLogo.Position = UDim2.new(0, 0, 0, 52)
SubLogo.BackgroundTransparency = 1
SubLogo.Text = "Premium Access v1.3"
SubLogo.Font = Enum.Font.Gotham
SubLogo.TextSize = 10
SubLogo.TextColor3 = Color3.fromRGB(140, 140, 140)

local Inp = Instance.new("TextBox", Box)
Inp.Size = UDim2.new(0, 320, 0, 42)
Inp.Position = UDim2.new(0.5, -160, 0, 100)
Inp.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Inp.TextColor3 = Color3.fromRGB(255, 255, 255)
Inp.PlaceholderText = "Key..."
Inp.PlaceholderColor3 = Color3.fromRGB(60, 60, 60)
Inp.Font = Enum.Font.Gotham
Inp.TextSize = 16
Inp.BorderSizePixel = 1
Inp.BorderColor3 = Color3.fromRGB(50, 50, 50)
Inp.TextXAlignment = Enum.TextXAlignment.Center

local Btn = Instance.new("TextButton", Box)
Btn.Size = UDim2.new(0, 320, 0, 44)
Btn.Position = UDim2.new(0.5, -160, 0, 155)
Btn.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
Btn.Text = "UNLOCK"
Btn.Font = Enum.Font.GothamBlack
Btn.TextSize = 14
Btn.BorderSizePixel = 0
Btn.AutoButtonColor = false

Btn.MouseButton1Click:Connect(function()
    if Inp.Text == validKey then
        KeyGui:Destroy()
        LoadScreen()
    else
        Inp.Text = ""
    end
end)

-- ============================================
-- LOADING SCREEN
-- ============================================
function LoadScreen()
    local LGui = Instance.new("ScreenGui", player.PlayerGui)
    
    local LBg = Instance.new("Frame", LGui)
    LBg.Size = UDim2.new(1, 0, 1, 0)
    LBg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    LBg.BorderSizePixel = 0
    
    local LBox = Instance.new("Frame", LBg)
    LBox.Size = UDim2.new(0, 450, 0, 180)
    LBox.Position = UDim2.new(0.5, -225, 0.5, -90)
    LBox.BackgroundColor3 = Color3.fromRGB(8, 8, 8)
    LBox.BorderSizePixel = 2
    LBox.BorderColor3 = Color3.fromRGB(180, 20, 20)
    
    local LT = Instance.new("TextLabel", LBox)
    LT.Size = UDim2.new(1, 0, 0, 40)
    LT.Position = UDim2.new(0, 0, 0, 25)
    LT.BackgroundTransparency = 1
    LT.Text = "DUELIST ALL SCRIPT v1.3"
    LT.Font = Enum.Font.GothamBlack
    LT.TextSize = 22
    LT.TextColor3 = Color3.fromRGB(255, 255, 255)
    
    local BarBg = Instance.new("Frame", LBox)
    BarBg.Size = UDim2.new(0, 360, 0, 8)
    BarBg.Position = UDim2.new(0.5, -180, 0, 85)
    BarBg.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
    BarBg.BorderSizePixel = 1
    BarBg.BorderColor3 = Color3.fromRGB(50, 50, 50)
    
    local Bar = Instance.new("Frame", BarBg)
    Bar.Size = UDim2.new(0, 0, 1, 0)
    Bar.BackgroundColor3 = Color3.fromRGB(200, 20, 20)
    Bar.BorderSizePixel = 0
    
    local Perc = Instance.new("TextLabel", LBox)
    Perc.Size = UDim2.new(1, 0, 0, 22)
    Perc.Position = UDim2.new(0, 0, 0, 100)
    Perc.BackgroundTransparency = 1
    Perc.Text = "0%"
    Perc.Font = Enum.Font.GothamBold
    Perc.TextSize = 14
    Perc.TextColor3 = Color3.fromRGB(255, 255, 255)
    
    local Stat = Instance.new("TextLabel", LBox)
    Stat.Size = UDim2.new(1, 0, 0, 20)
    Stat.Position = UDim2.new(0, 0, 0, 125)
    Stat.BackgroundTransparency = 1
    Stat.Text = "Loading..."
    Stat.Font = Enum.Font.Gotham
    Stat.TextSize = 10
    Stat.TextColor3 = Color3.fromRGB(150, 150, 150)
    
    spawn(function()
        local steps = {{20, "Starting..."},{50, "Loading..."},{80, "Preparing..."},{100, "Done!"}}
        wait(0.3)
        for _, s in pairs(steps) do
            pcall(function()
                Bar:TweenSize(UDim2.new(s[1]/100, 0, 1, 0), "Out", "Quad", 0.4, true)
            end)
            Perc.Text = s[1] .. "%"
            Stat.Text = s[2]
            wait(0.45)
        end
        wait(0.3)
        for i = 0, 1, 0.05 do
            LBg.BackgroundTransparency = i
            for _, v in pairs(LBox:GetDescendants()) do
                if v:IsA("TextLabel") then v.TextTransparency = i
                elseif v:IsA("Frame") then v.BackgroundTransparency = i end
            end
            wait(0.02)
        end
        LGui:Destroy()
        StartHub()
    end)
end

-- ============================================
-- HUB PRINCIPAL v1.3
-- ============================================
function StartHub()
    local Hub = Instance.new("ScreenGui", player.PlayerGui)
    
    Notify("DUELIST HUB v1.3", "Welcome! Auto Click added!", 5)
    
    local Main = Instance.new("Frame", Hub)
    Main.Size = UDim2.new(0, 750, 0, 550)
    Main.Position = UDim2.new(0.5, -375, 0.5, -275)
    Main.BackgroundColor3 = Color3.fromRGB(6, 6, 6)
    Main.BorderSizePixel = 2
    Main.BorderColor3 = Color3.fromRGB(180, 20, 20)
    Main.Active = true
    Main.Draggable = true
    
    local TitleBar = Instance.new("Frame", Main)
    TitleBar.Size = UDim2.new(1, 0, 0, 42)
    TitleBar.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    TitleBar.BorderSizePixel = 0
    
    local TRedLine = Instance.new("Frame", TitleBar)
    TRedLine.Size = UDim2.new(1, 0, 0, 2)
    TRedLine.Position = UDim2.new(0, 0, 1, -2)
    TRedLine.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
    TRedLine.BorderSizePixel = 0
    
    local HubTitle = Instance.new("TextLabel", TitleBar)
    HubTitle.Size = UDim2.new(1, -100, 1, 0)
    HubTitle.Position = UDim2.new(0, 15, 0, 0)
    HubTitle.BackgroundTransparency = 1
    HubTitle.Text = "DUELIST ALL SCRIPT v1.3"
    HubTitle.Font = Enum.Font.GothamBlack
    HubTitle.TextSize = 16
    HubTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    HubTitle.TextXAlignment = Enum.TextXAlignment.Left
    
    local CloseBtn = Instance.new("TextButton", TitleBar)
    CloseBtn.Size = UDim2.new(0, 42, 0, 42)
    CloseBtn.Position = UDim2.new(1, -42, 0, 0)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
    CloseBtn.Text = "X"
    CloseBtn.Font = Enum.Font.GothamBlack
    CloseBtn.TextSize = 20
    CloseBtn.BorderSizePixel = 0
    CloseBtn.AutoButtonColor = false
    CloseBtn.MouseButton1Click:Connect(function() Hub:Destroy() end)
    
    local TabContainer = Instance.new("Frame", Main)
    TabContainer.Size = UDim2.new(0, 150, 1, -42)
    TabContainer.Position = UDim2.new(0, 0, 0, 42)
    TabContainer.BackgroundColor3 = Color3.fromRGB(8, 8, 8)
    TabContainer.BorderSizePixel = 0
    
    local TabSepLine = Instance.new("Frame", TabContainer)
    TabSepLine.Size = UDim2.new(0, 2, 1, 0)
    TabSepLine.Position = UDim2.new(1, -2, 0, 0)
    TabSepLine.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
    TabSepLine.BorderSizePixel = 0
    
    local ContentArea = Instance.new("Frame", Main)
    ContentArea.Size = UDim2.new(1, -150, 1, -42)
    ContentArea.Position = UDim2.new(0, 150, 0, 42)
    ContentArea.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    ContentArea.BorderSizePixel = 0
    
    local tabs = {}
    local tabNames = {"Credits", "Stats", "Duelist", "Fly", "Anti AFK", "Auto Click"}
    local tabContents = {}
    
    for i, name in pairs(tabNames) do
        local tabBtn = Instance.new("TextButton", TabContainer)
        tabBtn.Size = UDim2.new(1, -8, 0, 42)
        tabBtn.Position = UDim2.new(0, 4, 0, (i-1)*46 + 8)
        tabBtn.BackgroundColor3 = i == 1 and Color3.fromRGB(180, 20, 20) or Color3.fromRGB(15, 15, 15)
        tabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        tabBtn.Text = name
        tabBtn.Font = Enum.Font.GothamBlack
        tabBtn.TextSize = 13
        tabBtn.BorderSizePixel = 0
        tabBtn.AutoButtonColor = false
        
        local content = Instance.new("ScrollingFrame", ContentArea)
        content.Size = UDim2.new(1, -24, 1, -24)
        content.Position = UDim2.new(0, 12, 0, 12)
        content.BackgroundColor3 = Color3.fromRGB(14, 14, 14)
        content.BorderSizePixel = 1
        content.BorderColor3 = Color3.fromRGB(35, 35, 35)
        content.Visible = (i == 1)
        content.ScrollBarThickness = 5
        content.ScrollBarImageColor3 = Color3.fromRGB(180, 20, 20)
        content.CanvasSize = UDim2.new(0, 0, 0, 450)
        
        tabBtn.MouseButton1Click:Connect(function()
            for _, c in pairs(tabContents) do c.Visible = false end
            content.Visible = true
            for _, b in pairs(tabs) do b.BackgroundColor3 = Color3.fromRGB(15, 15, 15) end
            tabBtn.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
        end)
        
        table.insert(tabs, tabBtn)
        table.insert(tabContents, content)
    end
    
    local function AddLabel(parent, text, y, size, color)
        local lbl = Instance.new("TextLabel", parent)
        lbl.Size = UDim2.new(1, -30, 0, 26)
        lbl.Position = UDim2.new(0, 15, 0, y)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.Font = Enum.Font.Gotham
        lbl.TextSize = size or 13
        lbl.TextColor3 = color or Color3.fromRGB(200, 200, 200)
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        return lbl
    end
    
    -- ABA 1: CREDITS
    local c1 = tabContents[1]
    AddLabel(c1, "CREATORS", 15, 15, Color3.fromRGB(255,255,255))
    AddLabel(c1, "Creator: Xx_Fl", 45)
    AddLabel(c1, "Helper: Satoro", 75)
    AddLabel(c1, "Program: Deepseek", 105)
    AddLabel(c1, "Version: v1.3", 140)
    AddLabel(c1, "New: Auto Click System", 170, 12, Color3.fromRGB(255,200,100))
    
    -- ============================================
    -- ABA 2: STATS MELHORADOS
    -- ============================================
    local c2 = tabContents[2]
    AddLabel(c2, "SYSTEM INFO", 15, 15, Color3.fromRGB(255,255,255))
    
    local executorLabel = AddLabel(c2, "Executor: " .. GetExecutor(), 45, 13, Color3.fromRGB(100,255,100))
    local countryLabel = AddLabel(c2, "Country: " .. GetCountry(), 75)
    local gamePlayersLabel = AddLabel(c2, "Players in Game: " .. GetPlayersInGame(), 105)
    local scriptUsersLabel = AddLabel(c2, "Script Users: " .. GetScriptUsers(), 135)
    
    AddLabel(c2, "", 165)
    AddLabel(c2, "PERFORMANCE", 185, 15, Color3.fromRGB(255,255,255))
    local timeLabel = AddLabel(c2, "Time: " .. GetUsageTime(), 215)
    AddLabel(c2, "Status: Running", 245, 13, Color3.fromRGB(100,255,100))
    
    AddLabel(c2, "", 275)
    AddLabel(c2, "LAST UPDATE", 295, 15, Color3.fromRGB(255,255,255))
    AddLabel(c2, "v1.3 - " .. os.date("%d/%m/%Y"), 325)
    AddLabel(c2, "Added: Auto Click", 355)
    AddLabel(c2, "Removed: Rollback", 385)
    
    -- Atualiza stats a cada 3 segundos
    spawn(function()
        while wait(3) do
            pcall(function()
                timeLabel.Text = "Time: " .. GetUsageTime()
                gamePlayersLabel.Text = "Players in Game: " .. GetPlayersInGame()
                scriptUsersLabel.Text = "Script Users: " .. GetScriptUsers()
            end)
        end
    end)
    
    -- ABA 3: DUELIST
    local c3 = tabContents[3]
    AddLabel(c3, "ANIME DUELISTS", 15, 17, Color3.fromRGB(255,255,255))
    AddLabel(c3, "Place ID: 135858844777165", 45, 12, Color3.fromRGB(150,150,150))
    
    local duelBtn = Instance.new("TextButton", c3)
    duelBtn.Size = UDim2.new(0, 280, 0, 50)
    duelBtn.Position = UDim2.new(0.5, -140, 0, 80)
    duelBtn.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
    duelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    duelBtn.Text = "Execute Duelist Script"
    duelBtn.Font = Enum.Font.GothamBlack
    duelBtn.TextSize = 14
    duelBtn.BorderSizePixel = 0
    duelBtn.AutoButtonColor = false
    
    duelBtn.MouseEnter:Connect(function() duelBtn.BackgroundColor3 = Color3.fromRGB(220,30,30) end)
    duelBtn.MouseLeave:Connect(function() duelBtn.BackgroundColor3 = Color3.fromRGB(180,20,20) end)
    
    duelBtn.MouseButton1Click:Connect(function()
        if game.PlaceId == 135858844777165 then
            Notify("DUELIST", "Loading script...", 3)
            pcall(function()
                loadstring(game:HttpGet("https://raw.githubusercontent.com/ProxyHubDev/Gold/refs/heads/main/src/scripts/animeduelists.lua"))()
            end)
        else
            Notify("WRONG GAME", "Join Anime Duelists!", 4)
        end
    end)
    
    AddLabel(c3, "1. Join Anime Duelists", 150, 12)
    AddLabel(c3, "2. Click button above", 175, 12)
    
    -- ABA 4: FLY
    local c4 = tabContents[4]
    AddLabel(c4, "FLY SYSTEM", 15, 15, Color3.fromRGB(255,255,255))
    
    local flyToggle = Instance.new("TextButton", c4)
    flyToggle.Size = UDim2.new(0, 280, 0, 45)
    flyToggle.Position = UDim2.new(0.5, -140, 0, 50)
    flyToggle.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
    flyToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    flyToggle.Text = "Fly: OFF"
    flyToggle.Font = Enum.Font.GothamBlack
    flyToggle.TextSize = 14
    flyToggle.BorderSizePixel = 1
    flyToggle.BorderColor3 = Color3.fromRGB(50, 50, 50)
    flyToggle.AutoButtonColor = false
    
    local flyOn = false
    flyToggle.MouseButton1Click:Connect(function()
        flyOn = not flyOn
        if flyOn then
            flyToggle.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
            flyToggle.Text = "Fly: ON"
            flying = true
            if character and character:FindFirstChild("HumanoidRootPart") then
                if bodyGyro then bodyGyro:Destroy() end
                if bodyVelocity then bodyVelocity:Destroy() end
                bodyGyro = Instance.new("BodyGyro", character.HumanoidRootPart)
                bodyGyro.P = 9e4 bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
                bodyGyro.CFrame = character.HumanoidRootPart.CFrame
                bodyVelocity = Instance.new("BodyVelocity", character.HumanoidRootPart)
                bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
                humanoid.PlatformStand = true
            end
        else
            flyToggle.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
            flyToggle.Text = "Fly: OFF"
            flying = false
            if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
            if bodyVelocity then bodyVelocity:Destroy() bodyVelocity = nil end
            humanoid.PlatformStand = false
        end
    end)
    
    AddLabel(c4, "WASD = Move | Space = Up | Shift = Down", 115)
    
    -- ABA 5: ANTI AFK
    local c5 = tabContents[5]
    AddLabel(c5, "ANTI AFK", 15, 15, Color3.fromRGB(255,255,255))
    
    local afkToggle = Instance.new("TextButton", c5)
    afkToggle.Size = UDim2.new(0, 280, 0, 45)
    afkToggle.Position = UDim2.new(0.5, -140, 0, 50)
    afkToggle.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
    afkToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    afkToggle.Text = "Anti AFK: OFF"
    afkToggle.Font = Enum.Font.GothamBlack
    afkToggle.TextSize = 14
    afkToggle.BorderSizePixel = 1
    afkToggle.BorderColor3 = Color3.fromRGB(50, 50, 50)
    afkToggle.AutoButtonColor = false
    
    afkToggle.MouseButton1Click:Connect(function()
        antiAFKEnabled = not antiAFKEnabled
        if antiAFKEnabled then
            afkToggle.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
            afkToggle.Text = "Anti AFK: ON"
            antiAFKConnection = game:GetService("Players").LocalPlayer.Idled:Connect(function()
                local VirtualUser = game:GetService("VirtualUser")
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
        else
            afkToggle.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
            afkToggle.Text = "Anti AFK: OFF"
            if antiAFKConnection then antiAFKConnection:Disconnect() antiAFKConnection = nil end
        end
    end)
    
    -- ============================================
    -- ABA 6: AUTO CLICK
    -- ============================================
    local c6 = tabContents[6]
    AddLabel(c6, "AUTO CLICK SYSTEM", 15, 17, Color3.fromRGB(255,255,255))
    AddLabel(c6, "Automatically clicks for you", 45, 12, Color3.fromRGB(255,200,100))
    
    local clickToggle = Instance.new("TextButton", c6)
    clickToggle.Size = UDim2.new(0, 280, 0, 50)
    clickToggle.Position = UDim2.new(0.5, -140, 0, 80)
    clickToggle.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
    clickToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    clickToggle.Text = "Auto Click: OFF"
    clickToggle.Font = Enum.Font.GothamBlack
    clickToggle.TextSize = 14
    clickToggle.BorderSizePixel = 1
    clickToggle.BorderColor3 = Color3.fromRGB(50, 50, 50)
    clickToggle.AutoButtonColor = false
    
    clickToggle.MouseButton1Click:Connect(function()
        autoClickEnabled = not autoClickEnabled
        if autoClickEnabled then
            clickToggle.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
            clickToggle.Text = "Auto Click: ON (" .. clickSpeed .. " CPS)"
            clickToggle.BorderColor3 = Color3.fromRGB(180, 20, 20)
            Notify("AUTO CLICK", "Clicking at " .. clickSpeed .. " CPS!", 3)
        else
            clickToggle.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
            clickToggle.Text = "Auto Click: OFF"
            clickToggle.BorderColor3 = Color3.fromRGB(50, 50, 50)
        end
    end)
    
    -- Slider de velocidade do click
    local clickSlider = Instance.new("TextBox", c6)
    clickSlider.Size = UDim2.new(0, 280, 0, 35)
    clickSlider.Position = UDim2.new(0.5, -140, 0, 150)
    clickSlider.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    clickSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
    clickSlider.PlaceholderText = "Click Speed (1-30 CPS): " .. clickSpeed
    clickSlider.PlaceholderColor3 = Color3.fromRGB(80, 80, 80)
    clickSlider.Font = Enum.Font.Gotham
    clickSlider.TextSize = 13
    clickSlider.BorderSizePixel = 1
    clickSlider.BorderColor3 = Color3.fromRGB(50, 50, 50)
    clickSlider.TextXAlignment = Enum.TextXAlignment.Center
    
    clickSlider.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local num = tonumber(clickSlider.Text)
            if num and num >= 1 and num <= 30 then
                clickSpeed = num
                if autoClickEnabled then
                    clickToggle.Text = "Auto Click: ON (" .. clickSpeed .. " CPS)"
                end
                clickSlider.PlaceholderText = "Click Speed (1-30 CPS): " .. clickSpeed
            end
            clickSlider.Text = ""
        end
    end)
    
    AddLabel(c6, "Default: 10 CPS | Max: 30 CPS", 200, 12)
    AddLabel(c6, "", 225)
    AddLabel(c6, "HOW TO USE:", 245, 14, Color3.fromRGB(255,255,255))
    AddLabel(c6, "1. Set click speed (1-30 CPS)", 270, 12)
    AddLabel(c6, "2. Enable Auto Click", 295, 12)
    AddLabel(c6, "3. Hold mouse over what you want to click", 320, 12)
    
    -- Loop do Auto Click
    spawn(function()
        while wait(1 / clickSpeed) do
            if autoClickEnabled then
                pcall(function()
                    local mouse = player:GetMouse()
                    game:GetService("VirtualInputManager"):SendMouseButtonEvent(
                        mouse.X, mouse.Y, 0, true, nil, 0
                    )
                    wait(0.05)
                    game:GetService("VirtualInputManager"):SendMouseButtonEvent(
                        mouse.X, mouse.Y, 0, false, nil, 0
                    )
                end)
            end
        end
    end)
    
    -- ============================================
    -- FLY LOOP
    -- ============================================
    game:GetService("RunService").RenderStepped:Connect(function()
        if flying and bodyGyro and bodyVelocity and character and character:FindFirstChild("HumanoidRootPart") then
            local cam = workspace.CurrentCamera
            local dir = Vector3.new(0, 0, 0)
            local inp = game:GetService("UserInputService")
            if inp:IsKeyDown(Enum.KeyCode.W) then dir = dir + cam.CFrame.LookVector end
            if inp:IsKeyDown(Enum.KeyCode.S) then dir = dir - cam.CFrame.LookVector end
            if inp:IsKeyDown(Enum.KeyCode.A) then dir = dir - cam.CFrame.RightVector end
            if inp:IsKeyDown(Enum.KeyCode.D) then dir = dir + cam.CFrame.RightVector end
            if inp:IsKeyDown(Enum.KeyCode.Space) then dir = dir + Vector3.new(0, 1, 0) end
            if inp:IsKeyDown(Enum.KeyCode.LeftShift) then dir = dir - Vector3.new(0, 1, 0) end
            if dir.Magnitude > 0 then dir = dir.Unit * flySpeed end
            bodyVelocity.Velocity = dir
            bodyGyro.CFrame = CFrame.lookAt(character.HumanoidRootPart.Position, character.HumanoidRootPart.Position + cam.CFrame.LookVector)
        end
    end)
    
    player.CharacterAdded:Connect(function(c)
        character = c
        humanoid = c:WaitForChild("Humanoid")
        if flying then
            wait(0.5)
            if bodyGyro then bodyGyro:Destroy() end
            if bodyVelocity then bodyVelocity:Destroy() end
            bodyGyro = Instance.new("BodyGyro", c.HumanoidRootPart)
            bodyGyro.P = 9e4 bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
            bodyGyro.CFrame = c.HumanoidRootPart.CFrame
            bodyVelocity = Instance.new("BodyVelocity", c.HumanoidRootPart)
            bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
            humanoid.PlatformStand = true
        end
    end)
end
