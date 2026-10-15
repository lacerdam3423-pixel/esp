-- =====================================================================
-- ADVANCED CHEAT ENGINE UI & ESP SYSTEM (SHADOWPLAY SYSTEM v5.0)
-- STABLE: TASK.DEFER + TASK.WAIT, NO LAG, ESP ALWAYS VISIBLE
-- =====================================================================

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

local CONFIG_FILE = "shadowplayscript.json"

local Config = {
    HitboxSize = 15,
    HitboxActive = true,
    SpeedValue = 16,
    SpeedActive = false,
    JumpValue = 50,
    JumpActive = false,
    FovValue = 70,
    FovActive = false,
    
    EspOutlineActive = true,
    EspNameActive = true,
    EspTimeActive = true,
    EspRoleActive = true,
    EspHealthTextActive = true,
    EspTracersActive = true,
    EspRainbowActive = false,
    EspStudsActive = false,
    
    CurrentTab = "Main"
}

-- =====================================================================
-- LIMPEZA DE INSTÂNCIAS ANTERIORES
-- =====================================================================
local function cleanupPreviousInstances()
    local oldGui = game:GetService("CoreGui"):FindFirstChild("InstantLockBypassUI")
    if oldGui then oldGui:Destroy() end
end

cleanupPreviousInstances()

-- =====================================================================
-- DETECÇÃO DE INVENTÁRIO (COM CACHE)
-- =====================================================================
local roleCache = {}

local function GetPlayerRole(player)
    if roleCache[player] and tick() - roleCache[player].time < 0.5 then
        return table.unpack(roleCache[player].data)
    end

    local hasKnife = false
    local hasGun = false
    local activeWeapon = "Nenhuma"
    local distance = 0

    pcall(function()
        local backpack = player:FindFirstChildOfClass("Backpack")
        local character = player.Character
        
        local function scanContainer(container)
            if not container then return end
            for _, item in ipairs(container:GetChildren()) do
                if item:IsA("Tool") then
                    local itemName = item.Name:lower()
                    if itemName:find("knife") or itemName:find("faca") or itemName:find("blade") or 
                       itemName:find("scythe") or itemName:find("dagger") or itemName:find("murder") or 
                       itemName:find("sword") or itemName:find("cutter") or itemName:find("cleaver") then
                        hasKnife = true
                        activeWeapon = item.Name
                    elseif itemName:find("gun") or itemName:find("revolver") or itemName:find("pistol") or 
                           itemName:find("sheriff") or itemName:find("blaster") or itemName:find("rifle") or 
                           itemName:find("hero") or itemName:find("arma") or itemName:find("revolvers") then
                        hasGun = true
                        activeWeapon = item.Name
                    end
                end
            end
        end

        scanContainer(backpack)
        scanContainer(character)
        
        local localChar = LocalPlayer.Character
        local targetChar = player.Character
        if localChar and targetChar then
            local localHRP = localChar:FindFirstChild("HumanoidRootPart")
            local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
            if localHRP and targetHRP then
                distance = math.floor((localHRP.Position - targetHRP.Position).Magnitude)
            end
        end
    end)

    local roleData
    if hasKnife then
        roleData = {"Murderer (Knife)", Color3.fromRGB(255, 35, 35), activeWeapon, distance}
    elseif hasGun then
        roleData = {"Sheriff (Gun)", Color3.fromRGB(30, 144, 255), activeWeapon, distance}
    else
        roleData = {"Inocente", Color3.fromRGB(0, 230, 120), "Nenhuma", distance}
    end

    roleCache[player] = {data = roleData, time = tick()}
    return table.unpack(roleData)
end

-- =====================================================================
-- PERSISTÊNCIA DE DADOS
-- =====================================================================
local function LoadConfig()
    pcall(function()
        if readfile and isfile and isfile(CONFIG_FILE) then
            local data = readfile(CONFIG_FILE)
            local decoded = HttpService:JSONDecode(data)
            for k, v in pairs(decoded) do
                if Config[k] ~= nil then Config[k] = v end
            end
        end
    end)
end

local function SaveConfig()
    pcall(function()
        if writefile then
            local encoded = HttpService:JSONEncode(Config)
            writefile(CONFIG_FILE, encoded)
        end
    end)
end

LoadConfig()

-- =====================================================================
-- CORE GUI CONTAINER
-- =====================================================================
local ParentTarget
if gethui then
    ParentTarget = gethui()
elseif syn and syn.protect_gui then
    ParentTarget = game:GetService("CoreGui")
    syn.protect_gui(ScreenGui)
else
    ParentTarget = game:GetService("CoreGui")
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "InstantLockBypassUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = ParentTarget

local TracersFolder = Instance.new("Folder")
TracersFolder.Name = "TracersFolder"
TracersFolder.Parent = ScreenGui

-- =====================================================================
-- SISTEMA DE SMOOTH DRAG
-- =====================================================================
local function MakeSmoothDraggable(guiObj, handles)
    local dragging = false
    local dragInput, dragStart, startPos
    local currentTween

    handles = type(handles) == "table" and handles or {handles or guiObj}

    for _, handle in ipairs(handles) do
        handle.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                dragStart = input.Position
                startPos = guiObj.Position

                local connection
                connection = input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        dragging = false
                        connection:Disconnect()
                    end
                end)
            end
        end)

        handle.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                dragInput = input
            end
        end)
    end

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            local targetPosition = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
            
            if currentTween then currentTween:Cancel() end
            currentTween = TweenService:Create(guiObj, TweenInfo.new(0.15, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = targetPosition})
            currentTween:Play()
        end
    end)
end

-- =====================================================================
-- CRIAÇÃO DA INTERFACE GRÁFICA
-- =====================================================================
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 320, 0, 470)
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = Color3.fromRGB(50, 50, 50)
MainStroke.Thickness = 1.8

local TopBar = Instance.new("Frame", MainFrame)
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 35)
TopBar.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
TopBar.BorderSizePixel = 0

local Title = Instance.new("TextLabel", TopBar)
Title.Size = UDim2.new(1, -70, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "SHADOWPLAY SCRIPT V5.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 11
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left

MakeSmoothDraggable(MainFrame, {TopBar, MainFrame})

local MinimizeBtn = Instance.new("TextButton", TopBar)
MinimizeBtn.Size = UDim2.new(0, 25, 0, 25)
MinimizeBtn.Position = UDim2.new(1, -55, 0, 5)
MinimizeBtn.BackgroundTransparency = 1
MinimizeBtn.Text = "–"
MinimizeBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
MinimizeBtn.TextSize = 16
MinimizeBtn.Font = Enum.Font.GothamBold

local CloseBtn = Instance.new("TextButton", TopBar)
CloseBtn.Size = UDim2.new(0, 25, 0, 25)
CloseBtn.Position = UDim2.new(1, -28, 0, 5)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.TextSize = 14
CloseBtn.Font = Enum.Font.GothamBold

local OpenEmojiBtn = Instance.new("TextButton")
OpenEmojiBtn.Name = "OpenEmojiBtn"
OpenEmojiBtn.Size = UDim2.new(0, 45, 0, 45)
OpenEmojiBtn.Position = UDim2.new(0.05, 0, 0.2, 0)
OpenEmojiBtn.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
OpenEmojiBtn.Text = "⚙️"
OpenEmojiBtn.TextSize = 22
OpenEmojiBtn.Visible = false
OpenEmojiBtn.Parent = ScreenGui

Instance.new("UICorner", OpenEmojiBtn).CornerRadius = UDim.new(1, 0)
local EmojiStroke = Instance.new("UIStroke", OpenEmojiBtn)
EmojiStroke.Color = Color3.fromRGB(0, 255, 150)
EmojiStroke.Thickness = 2

MakeSmoothDraggable(OpenEmojiBtn, OpenEmojiBtn)

local SideBar = Instance.new("Frame", MainFrame)
SideBar.Name = "SideBar"
SideBar.Size = UDim2.new(0, 45, 1, -35)
SideBar.Position = UDim2.new(0, 0, 0, 35)
SideBar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
SideBar.BorderSizePixel = 0

local TabMainBtn = Instance.new("TextButton", SideBar)
TabMainBtn.Size = UDim2.new(0, 35, 0, 35)
TabMainBtn.Position = UDim2.new(0, 5, 0, 10)
TabMainBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
TabMainBtn.Text = "⚡"
TabMainBtn.TextSize = 16
Instance.new("UICorner", TabMainBtn).CornerRadius = UDim.new(0, 6)

local TabRenderBtn = Instance.new("TextButton", SideBar)
TabRenderBtn.Size = UDim2.new(0, 35, 0, 35)
TabRenderBtn.Position = UDim2.new(0, 5, 0, 50)
TabRenderBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
TabRenderBtn.Text = "👁️"
TabRenderBtn.TextSize = 16
Instance.new("UICorner", TabRenderBtn).CornerRadius = UDim.new(0, 6)

local TabEspBtn = Instance.new("TextButton", SideBar)
TabEspBtn.Size = UDim2.new(0, 35, 0, 35)
TabEspBtn.Position = UDim2.new(0, 5, 0, 90)
TabEspBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
TabEspBtn.Text = "👤"
TabEspBtn.TextSize = 16
Instance.new("UICorner", TabEspBtn).CornerRadius = UDim.new(0, 6)

local ContentArea = Instance.new("Frame", MainFrame)
ContentArea.Size = UDim2.new(1, -45, 1, -35)
ContentArea.Position = UDim2.new(0, 45, 0, 35)
ContentArea.BackgroundTransparency = 1

local PageMain = Instance.new("Frame", ContentArea)
PageMain.Size = UDim2.new(1, 0, 1, 0)
PageMain.BackgroundTransparency = 1
PageMain.Visible = true

local PageRender = Instance.new("Frame", ContentArea)
PageRender.Size = UDim2.new(1, 0, 1, 0)
PageRender.BackgroundTransparency = 1
PageRender.Visible = false

local PageEsp = Instance.new("Frame", ContentArea)
PageEsp.Size = UDim2.new(1, 0, 1, 0)
PageEsp.BackgroundTransparency = 1
PageEsp.Visible = false

local function updateStatusBorder(btn, stroke, isActive)
    if isActive then
        btn.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        stroke.Color = Color3.fromRGB(0, 255, 130)
    else
        btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
        btn.TextColor3 = Color3.fromRGB(180, 180, 180)
        stroke.Color = Color3.fromRGB(60, 60, 60)
    end
end

-- =====================================================================
-- ABA MAIN
-- =====================================================================
local HBLabel = Instance.new("TextLabel", PageMain)
HBLabel.Size = UDim2.new(1, -20, 0, 18)
HBLabel.Position = UDim2.new(0, 10, 0, 10)
HBLabel.BackgroundTransparency = 1
HBLabel.Text = "Hitbox Clássica (0 - 200):"
HBLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
HBLabel.TextSize = 10
HBLabel.Font = Enum.Font.Gotham
HBLabel.TextXAlignment = Enum.TextXAlignment.Left

local HBBox = Instance.new("TextBox", PageMain)
HBBox.Size = UDim2.new(1, -20, 0, 28)
HBBox.Position = UDim2.new(0, 10, 0, 30)
HBBox.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
HBBox.TextColor3 = Color3.fromRGB(255, 255, 255)
HBBox.Text = tostring(Config.HitboxSize)
HBBox.Font = Enum.Font.GothamMedium
HBBox.TextSize = 11
Instance.new("UICorner", HBBox).CornerRadius = UDim.new(0, 6)
local HBBoxStroke = Instance.new("UIStroke", HBBox)
HBBoxStroke.Color = Color3.fromRGB(50, 50, 50)

local HBBtn = Instance.new("TextButton", PageMain)
HBBtn.Size = UDim2.new(1, -20, 0, 28)
HBBtn.Position = UDim2.new(0, 10, 0, 63)
HBBtn.Text = Config.HitboxActive and "HITBOX: ATIVADA" or "HITBOX: DESATIVADA"
HBBtn.Font = Enum.Font.GothamBold
HBBtn.TextSize = 10
Instance.new("UICorner", HBBtn).CornerRadius = UDim.new(0, 6)
local HBBtnStroke = Instance.new("UIStroke", HBBtn)
updateStatusBorder(HBBtn, HBBtnStroke, Config.HitboxActive)

local SPLabel = Instance.new("TextLabel", PageMain)
SPLabel.Size = UDim2.new(1, -20, 0, 18)
SPLabel.Position = UDim2.new(0, 10, 0, 100)
SPLabel.BackgroundTransparency = 1
SPLabel.Text = "Velocidade Fixa (0 - 999):"
SPLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
SPLabel.TextSize = 10
SPLabel.Font = Enum.Font.Gotham
SPLabel.TextXAlignment = Enum.TextXAlignment.Left

local SPBox = Instance.new("TextBox", PageMain)
SPBox.Size = UDim2.new(1, -20, 0, 28)
SPBox.Position = UDim2.new(0, 10, 0, 120)
SPBox.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
SPBox.TextColor3 = Color3.fromRGB(255, 255, 255)
SPBox.Text = tostring(Config.SpeedValue)
SPBox.Font = Enum.Font.GothamMedium
SPBox.TextSize = 11
Instance.new("UICorner", SPBox).CornerRadius = UDim.new(0, 6)
local SPBoxStroke = Instance.new("UIStroke", SPBox)
SPBoxStroke.Color = Color3.fromRGB(50, 50, 50)

local SPBtn = Instance.new("TextButton", PageMain)
SPBtn.Size = UDim2.new(1, -20, 0, 28)
SPBtn.Position = UDim2.new(0, 10, 0, 153)
SPBtn.Text = Config.SpeedActive and "SPEED: ATIVADA" or "SPEED: DESATIVADA"
SPBtn.Font = Enum.Font.GothamBold
SPBtn.TextSize = 10
Instance.new("UICorner", SPBtn).CornerRadius = UDim.new(0, 6)
local SPBtnStroke = Instance.new("UIStroke", SPBtn)
updateStatusBorder(SPBtn, SPBtnStroke, Config.SpeedActive)

local JPLabel = Instance.new("TextLabel", PageMain)
JPLabel.Size = UDim2.new(1, -20, 0, 18)
JPLabel.Position = UDim2.new(0, 10, 0, 190)
JPLabel.BackgroundTransparency = 1
JPLabel.Text = "Pulo Fixo (0 - 999):"
JPLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
JPLabel.TextSize = 10
JPLabel.Font = Enum.Font.Gotham
JPLabel.TextXAlignment = Enum.TextXAlignment.Left

local JPBox = Instance.new("TextBox", PageMain)
JPBox.Size = UDim2.new(1, -20, 0, 28)
JPBox.Position = UDim2.new(0, 10, 0, 210)
JPBox.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
JPBox.TextColor3 = Color3.fromRGB(255, 255, 255)
JPBox.Text = tostring(Config.JumpValue)
JPBox.Font = Enum.Font.GothamMedium
JPBox.TextSize = 11
Instance.new("UICorner", JPBox).CornerRadius = UDim.new(0, 6)
local JPBoxStroke = Instance.new("UIStroke", JPBox)
JPBoxStroke.Color = Color3.fromRGB(50, 50, 50)

local JPBtn = Instance.new("TextButton", PageMain)
JPBtn.Size = UDim2.new(1, -20, 0, 28)
JPBtn.Position = UDim2.new(0, 10, 0, 243)
JPBtn.Text = Config.JumpActive and "JUMP: ATIVADO" or "JUMP: DESATIVADO"
JPBtn.Font = Enum.Font.GothamBold
JPBtn.TextSize = 10
Instance.new("UICorner", JPBtn).CornerRadius = UDim.new(0, 6)
local JPBtnStroke = Instance.new("UIStroke", JPBtn)
updateStatusBorder(JPBtn, JPBtnStroke, Config.JumpActive)

-- =====================================================================
-- ABA RENDER
-- =====================================================================
local FOVLabel = Instance.new("TextLabel", PageRender)
FOVLabel.Size = UDim2.new(1, -20, 0, 18)
FOVLabel.Position = UDim2.new(0, 10, 0, 10)
FOVLabel.BackgroundTransparency = 1
FOVLabel.Text = "FOV Infinito sem Limites (0 - 9999):"
FOVLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
FOVLabel.TextSize = 10
FOVLabel.Font = Enum.Font.Gotham
FOVLabel.TextXAlignment = Enum.TextXAlignment.Left

local FOVBox = Instance.new("TextBox", PageRender)
FOVBox.Size = UDim2.new(1, -20, 0, 28)
FOVBox.Position = UDim2.new(0, 10, 0, 30)
FOVBox.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
FOVBox.TextColor3 = Color3.fromRGB(255, 255, 255)
FOVBox.Text = tostring(Config.FovValue)
FOVBox.Font = Enum.Font.GothamMedium
FOVBox.TextSize = 11
Instance.new("UICorner", FOVBox).CornerRadius = UDim.new(0, 6)
local FOVBoxStroke = Instance.new("UIStroke", FOVBox)
FOVBoxStroke.Color = Color3.fromRGB(50, 50, 50)

local FOVBtn = Instance.new("TextButton", PageRender)
FOVBtn.Size = UDim2.new(1, -20, 0, 28)
FOVBtn.Position = UDim2.new(0, 10, 0, 63)
FOVBtn.Text = Config.FovActive and "FOV INFINITO: ATIVADO" or "FOV INFINITO: DESATIVADO"
FOVBtn.Font = Enum.Font.GothamBold
FOVBtn.TextSize = 10
Instance.new("UICorner", FOVBtn).CornerRadius = UDim.new(0, 6)
local FOVBtnStroke = Instance.new("UIStroke", FOVBtn)
updateStatusBorder(FOVBtn, FOVBtnStroke, Config.FovActive)

-- =====================================================================
-- ABA ESP
-- =====================================================================
local function createToggle(parent, yPos, text, configKey)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, -20, 0, 28)
    btn.Position = UDim2.new(0, 10, 0, yPos)
    btn.Text = text .. (Config[configKey] and ": ATIVADO" or ": DESATIVADO")
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 10
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    local stroke = Instance.new("UIStroke", btn)
    updateStatusBorder(btn, stroke, Config[configKey])

    btn.MouseButton1Click:Connect(function()
        Config[configKey] = not Config[configKey]
        btn.Text = text .. (Config[configKey] and ": ATIVADO" or ": DESATIVADO")
        updateStatusBorder(btn, stroke, Config[configKey])
        SaveConfig()
    end)
    return btn
end

createToggle(PageEsp, 10, "ESP HIGHLIGHT", "EspOutlineActive")
createToggle(PageEsp, 45, "ESP NOME / APLICAÇÃO", "EspNameActive")
createToggle(PageEsp, 80, "ESP GUN / KNIFE (MURDER & SHERIFF)", "EspRoleActive")
createToggle(PageEsp, 115, "ESP TIME DO JOGADOR", "EspTimeActive")
createToggle(PageEsp, 150, "ESP HP EM TEXTO", "EspHealthTextActive")
createToggle(PageEsp, 185, "LINHAS TRACERS (TRACK)", "EspTracersActive")
createToggle(PageEsp, 220, "ESP RAINBOW (CORES)", "EspRainbowActive")
createToggle(PageEsp, 255, "ESP DISTÂNCIA (STUDS)", "EspStudsActive")

-- =====================================================================
-- HOOK DE CÂMERA
-- =====================================================================
local FovConnection

local function hookCameraFov()
    if FovConnection then FovConnection:Disconnect() end

    if not Config.FovActive then
        Config.FovValue = math.floor(Camera.FieldOfView)
        FOVBox.Text = tostring(Config.FovValue)
    else
        Camera.FieldOfView = Config.FovValue
    end

    FovConnection = Camera:GetPropertyChangedSignal("FieldOfView"):Connect(function()
        if Config.FovActive then
            if Camera.FieldOfView ~= Config.FovValue then
                Camera.FieldOfView = Config.FovValue
            end
        else
            Config.FovValue = math.floor(Camera.FieldOfView)
            FOVBox.Text = tostring(Config.FovValue)
        end
    end)
end

hookCameraFov()
Workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
    Camera = Workspace.CurrentCamera
    hookCameraFov()
end)

-- =====================================================================
-- MONITORAMENTO DE PLAYER (COM TASK.DEFER)
-- =====================================================================
local PlayerConnections = {}

local function applyJumpValue(hum)
    if hum.UseJumpPower then
        hum.JumpPower = Config.JumpValue
    else
        hum.JumpHeight = Config.JumpValue
    end
end

local function hookAndMonitorHumanoid(hum)
    if not hum then return end
    
    if PlayerConnections[hum] then
        for _, conn in ipairs(PlayerConnections[hum]) do
            conn:Disconnect()
        end
    end
    PlayerConnections[hum] = {}

    if not Config.SpeedActive then
        Config.SpeedValue = math.floor(hum.WalkSpeed)
        SPBox.Text = tostring(Config.SpeedValue)
    else
        hum.WalkSpeed = Config.SpeedValue
    end

    if not Config.JumpActive then
        local currentJump = hum.UseJumpPower and hum.JumpPower or hum.JumpHeight
        Config.JumpValue = math.floor(currentJump)
        JPBox.Text = tostring(Config.JumpValue)
    else
        applyJumpValue(hum)
    end

    local c1 = hum:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
        if Config.SpeedActive and hum.WalkSpeed ~= Config.SpeedValue then
            hum.WalkSpeed = Config.SpeedValue
        end
    end)
    table.insert(PlayerConnections[hum], c1)

    local c2 = hum:GetPropertyChangedSignal("JumpPower"):Connect(function()
        if Config.JumpActive and hum.UseJumpPower and hum.JumpPower ~= Config.JumpValue then
            hum.JumpPower = Config.JumpValue
        end
    end)
    table.insert(PlayerConnections[hum], c2)
end

if LocalPlayer.Character then
    local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if hum then hookAndMonitorHumanoid(hum) end
end

LocalPlayer.CharacterAdded:Connect(function(char)
    local hum = char:WaitForChild("Humanoid", 5)
    if hum then hookAndMonitorHumanoid(hum) end
end)

-- =====================================================================
-- HITBOX (COM TASK.DEFER E STAGGER)
-- =====================================================================
local hitboxConnection

local function updateHitbox()
    if hitboxConnection then hitboxConnection:Disconnect() end
    
    if not Config.HitboxActive then return end
    
    hitboxConnection = RunService.Heartbeat:Connect(function()
        if not Config.HitboxActive then
            hitboxConnection:Disconnect()
            return
        end
        
        local players = Players:GetPlayers()
        for i, player in ipairs(players) do
            task.defer(function()
                if player ~= LocalPlayer and player.Character then
                    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                    if hrp and Config.HitboxSize > 0 then
                        hrp.Size = Vector3.new(Config.HitboxSize, Config.HitboxSize, Config.HitboxSize)
                        hrp.Transparency = 0.7
                        hrp.BrickColor = BrickColor.new("Really white")
                        hrp.Material = Enum.Material.Neon
                        hrp.CanCollide = false
                    end
                end
            end)
            
            if i % 3 == 0 then
                task.wait()
            end
        end
    end)
end

updateHitbox()

-- =====================================================================
-- ESP AVANÇADO (COM TASK.DEFER, SEM DESAPARECER)
-- =====================================================================
local playerEspData = {}
local espUpdateConnection

local function cleanupPlayerEsp(player)
    local data = playerEspData[player]
    if data then
        if data.highlight then data.highlight:Destroy() end
        if data.bgui then data.bgui:Destroy() end
        if data.tracerLine then data.tracerLine:Destroy() end
        playerEspData[player] = nil
    end
    roleCache[player] = nil
end

local function createPlayerEsp(player)
    if player == LocalPlayer then return end
    
    cleanupPlayerEsp(player)
    
    local character = player.Character
    if not character then return end
    
    local head = character:FindFirstChild("Head")
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not head or not humanoid then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Outline"
    highlight.FillTransparency = 0.8
    highlight.OutlineTransparency = 0.1
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Adornee = character
    highlight.Parent = ScreenGui

    local bgui = Instance.new("BillboardGui")
    bgui.Name = "ESP_NameTag"
    bgui.Adornee = head
    bgui.Size = UDim2.new(0, 200, 0, 75)
    bgui.StudsOffset = Vector3.new(0, 3.0, 0)
    bgui.AlwaysOnTop = true
    bgui.Parent = ScreenGui

    local label = Instance.new("TextLabel", bgui)
    label.Name = "NameLabel"
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextStrokeTransparency = 0.2
    label.TextSize = 13
    label.Font = Enum.Font.FredokaOne

    local tracerLine = Instance.new("Frame", TracersFolder)
    tracerLine.Name = "Tracer_" .. player.Name
    tracerLine.AnchorPoint = Vector2.new(0.5, 0.5)
    tracerLine.BorderSizePixel = 0
    tracerLine.BackgroundColor3 = Color3.fromRGB(0, 255, 150)
    tracerLine.Visible = false

    playerEspData[player] = {
        highlight = highlight,
        bgui = bgui,
        label = label,
        tracerLine = tracerLine,
        character = character,
        humanoid = humanoid
    }
end

local function updateAllEsp()
    local players = Players:GetPlayers()
    local playerCount = #players
    
    for i, player in ipairs(players) do
        task.defer(function()
            if player == LocalPlayer then return end
            
            local data = playerEspData[player]
            if not data then
                createPlayerEsp(player)
                data = playerEspData[player]
                if not data then return end
            end
            
            local character = player.Character
            local humanoid = character and character:FindFirstChildOfClass("Humanoid")
            
            if not character or not humanoid then
                data.highlight.Enabled = false
                data.bgui.Enabled = false
                data.tracerLine.Visible = false
                return
            end
            
            data.character = character
            data.humanoid = humanoid
            
            local anyEspActive = Config.EspOutlineActive or Config.EspNameActive or Config.EspRoleActive or 
                                Config.EspTimeActive or Config.EspHealthTextActive or Config.EspTracersActive or Config.EspStudsActive
            
            if not anyEspActive then
                data.highlight.Enabled = false
                data.bgui.Enabled = false
                data.tracerLine.Visible = false
                return
            end

            local roleTitle, roleColor, activeWeapon, distance = GetPlayerRole(player)
            local currentTargetColor = Color3.fromRGB(0, 255, 150)

            if Config.EspRainbowActive then
                currentTargetColor = Color3.fromHSV((os.clock() * 1.5) % 1, 1, 1)
            elseif Config.EspRoleActive then
                currentTargetColor = roleColor
            elseif Config.EspTimeActive and player.Team then
                currentTargetColor = player.TeamColor.Color
            end

            data.highlight.Enabled = Config.EspOutlineActive
            data.highlight.OutlineColor = currentTargetColor
            data.highlight.FillColor = currentTargetColor

            if Config.EspNameActive or Config.EspRoleActive or Config.EspTimeActive or Config.EspHealthTextActive or Config.EspStudsActive then
                data.bgui.Enabled = true
                local textStr = ""
                
                if Config.EspNameActive then
                    textStr = textStr .. "@" .. player.Name .. "\n"
                end
                if Config.EspRoleActive then
                    textStr = textStr .. "[" .. roleTitle .. "]\n"
                    if activeWeapon ~= "Nenhuma" then
                        textStr = textStr .. "Arma: " .. activeWeapon .. "\n"
                    end
                end
                if Config.EspTimeActive then
                    local teamName = player.Team and player.Team.Name or "Sem Time"
                    textStr = textStr .. "Time: " .. teamName .. "\n"
                end
                if Config.EspHealthTextActive then
                    textStr = textStr .. "HP: " .. math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth) .. "\n"
                end
                if Config.EspStudsActive then
                    textStr = textStr .. distance .. " studs"
                end
                
                data.label.Text = textStr
                data.label.TextColor3 = currentTargetColor
            else
                data.bgui.Enabled = false
            end

            local hrp = character:FindFirstChild("HumanoidRootPart")
            if Config.EspTracersActive and hrp then
                local vector, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    local targetPos = Vector2.new(vector.X, vector.Y)
                    local dist = (targetPos - screenCenter).Magnitude

                    data.tracerLine.Size = UDim2.new(0, dist, 0, 1.5)
                    data.tracerLine.Position = UDim2.new(0, (screenCenter.X + targetPos.X) / 2, 0, (screenCenter.Y + targetPos.Y) / 2)
                    data.tracerLine.Rotation = math.deg(math.atan2(targetPos.Y - screenCenter.Y, targetPos.X - screenCenter.X))
                    data.tracerLine.BackgroundColor3 = currentTargetColor
                    data.tracerLine.Visible = true
                else
                    data.tracerLine.Visible = false
                end
            else
                data.tracerLine.Visible = false
            end
        end)
        
        if i % 5 == 0 then
            task.wait()
        end
    end
end

if espUpdateConnection then espUpdateConnection:Disconnect() end
espUpdateConnection = RunService.Heartbeat:Connect(function()
    updateAllEsp()
end)

for _, p in ipairs(Players:GetPlayers()) do
    task.defer(createPlayerEsp, p)
end

Players.PlayerAdded:Connect(function(player)
    task.defer(createPlayerEsp, player)
end)

Players.PlayerRemoving:Connect(function(player)
    cleanupPlayerEsp(player)
end)

-- =====================================================================
-- CONTROLES DE NAVEGAÇÃO
-- =====================================================================
TabMainBtn.MouseButton1Click:Connect(function()
    PageMain.Visible = true; PageRender.Visible = false; PageEsp.Visible = false
    TabMainBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    TabRenderBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    TabEspBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
end)

TabRenderBtn.MouseButton1Click:Connect(function()
    PageMain.Visible = false; PageRender.Visible = true; PageEsp.Visible = false
    TabMainBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    TabRenderBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    TabEspBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
end)

TabEspBtn.MouseButton1Click:Connect(function()
    PageMain.Visible = false; PageRender.Visible = false; PageEsp.Visible = true
    TabMainBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    TabRenderBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    TabEspBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
end)

CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    OpenEmojiBtn.Visible = true
end)

OpenEmojiBtn.MouseButton1Click:Connect(function()
    OpenEmojiBtn.Visible = false
    MainFrame.Visible = true
end)

local isMinimized = false
MinimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    if isMinimized then
        MainFrame:TweenSize(UDim2.new(0, 320, 0, 35), Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.25, true)
        MinimizeBtn.Text = "+"
    else
        MainFrame:TweenSize(UDim2.new(0, 320, 0, 470), Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.25, true)
        MinimizeBtn.Text = "–"
    end
end)

HBBox.FocusLost:Connect(function()
    local num = tonumber(HBBox.Text)
    if num then Config.HitboxSize = math.clamp(num, 0, 200); HBBox.Text = tostring(Config.HitboxSize); SaveConfig(); updateHitbox() end
end)

HBBtn.MouseButton1Click:Connect(function()
    Config.HitboxActive = not Config.HitboxActive
    HBBtn.Text = Config.HitboxActive and "HITBOX: ATIVADA" or "HITBOX: DESATIVADA"
    updateStatusBorder(HBBtn, HBBtnStroke, Config.HitboxActive)
    SaveConfig()
    updateHitbox()
end)

SPBox.FocusLost:Connect(function()
    local num = tonumber(SPBox.Text)
    if num then Config.SpeedValue = math.clamp(num, 0, 999); SPBox.Text = tostring(Config.SpeedValue); SaveConfig() end
end)

SPBtn.MouseButton1Click:Connect(function()
    Config.SpeedActive = not Config.SpeedActive
    SPBtn.Text = Config.SpeedActive and "SPEED: ATIVADA" or "SPEED: DESATIVADA"
    updateStatusBorder(SPBtn, SPBtnStroke, Config.SpeedActive)
    SaveConfig()
end)

JPBox.FocusLost:Connect(function()
    local num = tonumber(JPBox.Text)
    if num then Config.JumpValue = math.clamp(num, 0, 999); JPBox.Text = tostring(Config.JumpValue); SaveConfig() end
end)

JPBtn.MouseButton1Click:Connect(function()
    Config.JumpActive = not Config.JumpActive
    JPBtn.Text = Config.JumpActive and "JUMP: ATIVADO" or "JUMP: DESATIVADO"
    updateStatusBorder(JPBtn, JPBtnStroke, Config.JumpActive)
    SaveConfig()
end)

FOVBox.FocusLost:Connect(function()
    local num = tonumber(FOVBox.Text)
    if num then Config.FovValue = math.clamp(num, 0, 9999); FOVBox.Text = tostring(Config.FovValue); SaveConfig() end
end)

FOVBtn.MouseButton1Click:Connect(function()
    Config.FovActive = not Config.FovActive
    FOVBtn.Text = Config.FovActive and "FOV INFINITO: ATIVADO" or "FOV INFINITO: DESATIVADO"
    updateStatusBorder(FOVBtn, FOVBtnStroke, Config.FovActive)
    if Config.FovActive then Camera.FieldOfView = Config.FovValue else Camera.FieldOfView = 70 end
    SaveConfig()
end)
