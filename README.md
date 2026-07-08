-- Fly GUI Script with Draggable Minimized Button - by samu164742
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Fly Variables
local flying = false
local flySpeed = 50
local flyConnection
local moveDirection = Vector3.new()

-- Joystick Variables
local joystickActive = false
local joystickPosition = Vector2.new()
local joystickCenter = Vector2.new()
local joystickRadius = 50
local inputObject = nil

-- Menu state
local menuMinimized = false

-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyGUI_Mobile"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = game:GetService("CoreGui")

wait(0.5)

-- ============================================
-- MINIMIZED BUTTON (Draggable - Starts in center)
-- ============================================

local minimizedButton = Instance.new("TextButton")
minimizedButton.Size = UDim2.new(0, 45, 0, 45)
minimizedButton.Position = UDim2.new(0.5, -22, 0.5, -22) -- CENTRO DA TELA
minimizedButton.BackgroundColor3 = Color3.fromRGB(140, 30, 255)
minimizedButton.Text = "✈"
minimizedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizedButton.TextSize = 20
minimizedButton.Font = Enum.Font.GothamBold
minimizedButton.BorderSizePixel = 2
minimizedButton.BorderColor3 = Color3.fromRGB(180, 60, 255)
minimizedButton.Visible = true -- COMEÇA VISÍVEL
minimizedButton.Active = true
minimizedButton.Draggable = true -- ARRASTÁVEL
minimizedButton.ZIndex = 100
minimizedButton.Parent = screenGui

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(1, 0)
minCorner.Parent = minimizedButton

-- Label abaixo do avião
local minLabel = Instance.new("TextLabel")
minLabel.Size = UDim2.new(0, 80, 0, 15)
minLabel.Position = UDim2.new(0.5, -40, 1, 5)
minLabel.BackgroundTransparency = 1
minLabel.Text = "Fly GUI"
minLabel.TextColor3 = Color3.fromRGB(180, 130, 255)
minLabel.TextSize = 10
minLabel.Font = Enum.Font.GothamSemibold
minLabel.TextStrokeTransparency = 0.5
minLabel.TextStrokeColor3 = Color3.fromRGB(120, 0, 255)
minLabel.ZIndex = 100
minLabel.Parent = minimizedButton

-- ============================================
-- MAIN FLY GUI (Starts HIDDEN)
-- ============================================

-- Main Frame
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 280, 0, 110)
mainFrame.Position = UDim2.new(0.5, -140, 0.82, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.BackgroundTransparency = 0.2
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(128, 0, 255)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Visible = false -- COMEÇA ESCONDIDO
mainFrame.ZIndex = 100
mainFrame.Parent = screenGui

-- YouTube Button INSIDE menu (Top Left)
local ytButton = Instance.new("TextButton")
ytButton.Size = UDim2.new(0, 22, 0, 22)
ytButton.Position = UDim2.new(0, 8, 0, 2)
ytButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ytButton.Text = "▶"
ytButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ytButton.TextSize = 11
ytButton.Font = Enum.Font.GothamBold
ytButton.BorderSizePixel = 0
ytButton.ZIndex = 101
ytButton.Parent = mainFrame

local ytCorner = Instance.new("UICorner")
ytCorner.CornerRadius = UDim.new(1, 0)
ytCorner.Parent = ytButton

-- Title
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(0, 90, 0, 20)
titleLabel.Position = UDim2.new(0, 32, 0, 3)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "FLY GUI"
titleLabel.TextColor3 = Color3.fromRGB(180, 130, 255)
titleLabel.TextSize = 16
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = mainFrame

-- Minimize Button (Top Right)
local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0, 22, 0, 22)
minimizeButton.Position = UDim2.new(1, -28, 0, 2)
minimizeButton.BackgroundColor3 = Color3.fromRGB(120, 120, 120)
minimizeButton.Text = "−"
minimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeButton.TextSize = 16
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.BorderSizePixel = 0
minimizeButton.ZIndex = 101
minimizeButton.Parent = mainFrame

local minCorner2 = Instance.new("UICorner")
minCorner2.CornerRadius = UDim.new(1, 0)
minCorner2.Parent = minimizeButton

-- Credit
local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(0, 100, 0, 16)
creditLabel.Position = UDim2.new(1, -130, 0, 5)
creditLabel.BackgroundTransparency = 1
creditLabel.Text = "by:samu164742"
creditLabel.TextColor3 = Color3.fromRGB(180, 60, 255)
creditLabel.TextSize = 10
creditLabel.Font = Enum.Font.GothamBold
creditLabel.TextStrokeTransparency = 0
creditLabel.TextStrokeColor3 = Color3.fromRGB(120, 0, 255)
creditLabel.TextXAlignment = Enum.TextXAlignment.Right
creditLabel.Parent = mainFrame

-- Speed label
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0, 100, 0, 25)
speedLabel.Position = UDim2.new(0, 10, 0, 28)
speedLabel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
speedLabel.Text = "Speed: "..flySpeed
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.TextSize = 14
speedLabel.Font = Enum.Font.GothamSemibold
speedLabel.Parent = mainFrame

-- Minus button
local minusButton = Instance.new("TextButton")
minusButton.Size = UDim2.new(0, 50, 0, 25)
minusButton.Position = UDim2.new(0, 120, 0, 28)
minusButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
minusButton.Text = "-10"
minusButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minusButton.TextSize = 14
minusButton.Font = Enum.Font.GothamBold
minusButton.Parent = mainFrame

-- Plus button
local plusButton = Instance.new("TextButton")
plusButton.Size = UDim2.new(0, 50, 0, 25)
plusButton.Position = UDim2.new(0, 175, 0, 28)
plusButton.BackgroundColor3 = Color3.fromRGB(30, 200, 30)
plusButton.Text = "+10"
plusButton.TextColor3 = Color3.fromRGB(255, 255, 255)
plusButton.TextSize = 14
plusButton.Font = Enum.Font.GothamBold
plusButton.Parent = mainFrame

-- Fly button
local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(1, -20, 0, 30)
flyButton.Position = UDim2.new(0, 10, 0, 56)
flyButton.BackgroundColor3 = Color3.fromRGB(140, 30, 255)
flyButton.Text = "FLY"
flyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyButton.TextSize = 16
flyButton.Font = Enum.Font.GothamBold
flyButton.Parent = mainFrame

-- ============================================
-- YOUTUBE POPUP MENU
-- ============================================

local ytPopup = Instance.new("Frame")
ytPopup.Size = UDim2.new(0, 250, 0, 120)
ytPopup.Position = UDim2.new(0.5, -125, 0.4, -60)
ytPopup.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
ytPopup.BorderSizePixel = 2
ytPopup.BorderColor3 = Color3.fromRGB(255, 0, 0)
ytPopup.Visible = false
ytPopup.ZIndex = 300
ytPopup.Active = true
ytPopup.Draggable = true
ytPopup.Parent = screenGui

local ytIcon = Instance.new("TextLabel")
ytIcon.Size = UDim2.new(0, 50, 0, 50)
ytIcon.Position = UDim2.new(0, 15, 0, 15)
ytIcon.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ytIcon.Text = "▶"
ytIcon.TextColor3 = Color3.fromRGB(255, 255, 255)
ytIcon.TextSize = 25
ytIcon.Font = Enum.Font.GothamBold
ytIcon.ZIndex = 301
ytIcon.Parent = ytPopup

local ytIconCorner = Instance.new("UICorner")
ytIconCorner.CornerRadius = UDim.new(1, 0)
ytIconCorner.Parent = ytIcon

local channelName = Instance.new("TextLabel")
channelName.Size = UDim2.new(0, 170, 0, 20)
channelName.Position = UDim2.new(0, 70, 0, 15)
channelName.BackgroundTransparency = 1
channelName.Text = "samuzeira"
channelName.TextColor3 = Color3.fromRGB(255, 255, 255)
channelName.TextSize = 16
channelName.Font = Enum.Font.GothamBold
channelName.ZIndex = 301
channelName.TextXAlignment = Enum.TextXAlignment.Left
channelName.Parent = ytPopup

local subText = Instance.new("TextLabel")
subText.Size = UDim2.new(0, 170, 0, 15)
subText.Position = UDim2.new(0, 70, 0, 35)
subText.BackgroundTransparency = 1
subText.Text = "Se inscreva no canal!"
subText.TextColor3 = Color3.fromRGB(200, 200, 200)
subText.TextSize = 11
subText.Font = Enum.Font.GothamSemibold
subText.ZIndex = 301
subText.TextXAlignment = Enum.TextXAlignment.Left
subText.Parent = ytPopup

local linkBox = Instance.new("Frame")
linkBox.Size = UDim2.new(0, 220, 0, 25)
linkBox.Position = UDim2.new(0, 15, 0, 60)
linkBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
linkBox.BorderSizePixel = 1
linkBox.BorderColor3 = Color3.fromRGB(255, 0, 0)
linkBox.ZIndex = 301
linkBox.Parent = ytPopup

local linkText = Instance.new("TextLabel")
linkText.Size = UDim2.new(1, -10, 1, 0)
linkText.Position = UDim2.new(0, 5, 0, 0)
linkText.BackgroundTransparency = 1
linkText.Text = "youtube.com/@samuzeiraofc"
linkText.TextColor3 = Color3.fromRGB(150, 150, 255)
linkText.TextSize = 10
linkText.Font = Enum.Font.GothamSemibold
linkText.ZIndex = 302
linkText.TextXAlignment = Enum.TextXAlignment.Left
linkText.Parent = linkBox

local copyButton = Instance.new("TextButton")
copyButton.Size = UDim2.new(0, 100, 0, 25)
copyButton.Position = UDim2.new(0, 15, 0, 88)
copyButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
copyButton.Text = "📋 Copiar Link"
copyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
copyButton.TextSize = 11
copyButton.Font = Enum.Font.GothamBold
copyButton.ZIndex = 301
copyButton.Parent = ytPopup

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -35, 0, 5)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 30, 30)
closeButton.Text = "✕"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextSize = 16
closeButton.Font = Enum.Font.GothamBold
closeButton.BorderSizePixel = 0
closeButton.ZIndex = 301
closeButton.Parent = ytPopup

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(1, 0)
closeCorner.Parent = closeButton

-- ============================================
-- MINIMIZE / RESTORE FUNCTIONS
-- ============================================

local function minimizeMenu()
    menuMinimized = true
    mainFrame.Visible = false
    minimizedButton.Visible = true
end

local function restoreMenu()
    menuMinimized = false
    mainFrame.Visible = true
    minimizedButton.Visible = false
end

-- Minimize button click
minimizeButton.MouseButton1Click:Connect(function()
    minimizeMenu()
end)

-- Minimized button click to restore
minimizedButton.MouseButton1Click:Connect(function()
    restoreMenu()
end)

-- YouTube button click
ytButton.MouseButton1Click:Connect(function()
    ytPopup.Visible = not ytPopup.Visible
end)

-- Close button click
closeButton.MouseButton1Click:Connect(function()
    ytPopup.Visible = false
end)

-- Copy button click
copyButton.MouseButton1Click:Connect(function()
    if setclipboard then
        setclipboard("https://youtube.com/@samuzeiraofc?si=C1W1xuJVOhNvY2qS")
    end
    copyButton.Text = "✅ Copiado!"
    wait(1.5)
    copyButton.Text = "📋 Copiar Link"
end)

-- ============================================
-- ANALOG JOYSTICK
-- ============================================

local joystickFrame = Instance.new("Frame")
joystickFrame.Size = UDim2.new(0, 120, 0, 120)
joystickFrame.Position = UDim2.new(0.15, -60, 0.65, -60)
joystickFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
joystickFrame.BackgroundTransparency = 0.85
joystickFrame.BorderSizePixel = 2
joystickFrame.BorderColor3 = Color3.fromRGB(180, 60, 255)
joystickFrame.Visible = false
joystickFrame.ZIndex = 100
joystickFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = joystickFrame

local joystickThumb = Instance.new("Frame")
joystickThumb.Size = UDim2.new(0, 50, 0, 50)
joystickThumb.Position = UDim2.new(0.5, -25, 0.5, -25)
joystickThumb.BackgroundColor3 = Color3.fromRGB(180, 60, 255)
joystickThumb.BackgroundTransparency = 0.3
joystickThumb.BorderSizePixel = 0
joystickThumb.Parent = joystickFrame

local thumbCorner = Instance.new("UICorner")
thumbCorner.CornerRadius = UDim.new(1, 0)
thumbCorner.Parent = joystickThumb

-- ============================================
-- JOYSTICK INPUT
-- ============================================

local function updateJoystickPosition(input)
    if not joystickActive then return end
    
    local position = input.Position
    local delta = position - joystickCenter
    local distance = delta.Magnitude
    
    if distance > joystickRadius then
        delta = delta.Unit * joystickRadius
    end
    
    joystickThumb.Position = UDim2.new(
        0, 
        delta.X + joystickFrame.AbsoluteSize.X/2 - 25, 
        0, 
        delta.Y + joystickFrame.AbsoluteSize.Y/2 - 25
    )
    
    local normalizedInput = delta / joystickRadius
    joystickPosition = Vector2.new(normalizedInput.X, -normalizedInput.Y)
end

joystickFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        joystickActive = true
        inputObject = input
        joystickCenter = input.Position
        updateJoystickPosition(input)
    end
end)

joystickThumb.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        joystickActive = true
        inputObject = input
        joystickCenter = input.Position
        updateJoystickPosition(input)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if joystickActive and input == inputObject then
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
            updateJoystickPosition(input)
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input == inputObject then
        joystickActive = false
        inputObject = nil
        joystickPosition = Vector2.new()
        joystickThumb.Position = UDim2.new(0.5, -25, 0.5, -25)
    end
end)

-- ============================================
-- FLY FUNCTIONS
-- ============================================

local function startFly()
    if flying then return end
    if not player.Character then return end
    
    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    flying = true
    joystickFrame.Visible = true
    
    local bodyGyro = Instance.new("BodyGyro")
    bodyGyro.P = 9e4
    bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    bodyGyro.CFrame = hrp.CFrame
    bodyGyro.Parent = hrp
    
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    bodyVelocity.Parent = hrp
    
    humanoid.PlatformStand = true
    
    flyConnection = RunService.RenderStepped:Connect(function()
        if flying and player.Character then
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            if root and bodyVelocity and bodyGyro then
                moveDirection = Vector3.new()
                local camera = workspace.CurrentCamera
                
                if joystickActive and joystickPosition.Magnitude > 0.1 then
                    local forward = camera.CFrame.LookVector * joystickPosition.Y
                    local right = camera.CFrame.RightVector * joystickPosition.X
                    moveDirection = (forward + right)
                    if moveDirection.Magnitude > 1 then
                        moveDirection = moveDirection.Unit
                    end
                end
                
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    moveDirection = moveDirection + camera.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    moveDirection = moveDirection - camera.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    moveDirection = moveDirection - camera.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    moveDirection = moveDirection + camera.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    moveDirection = moveDirection + Vector3.new(0, 1, 0)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
                    moveDirection = moveDirection - Vector3.new(0, 1, 0)
                end
                
                if moveDirection.Magnitude > 0 then
                    moveDirection = moveDirection.Unit
                end
                
                bodyVelocity.Velocity = moveDirection * flySpeed
                bodyGyro.CFrame = camera.CFrame
            end
        end
    end)
    
    flyButton.Text = "DISABLE"
    flyButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
end

local function stopFly()
    if not flying then return end
    flying = false
    
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    
    if player.Character then
        local hrp = player.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            local bg = hrp:FindFirstChild("BodyGyro")
            if bg then bg:Destroy() end
            local bv = hrp:FindFirstChild("BodyVelocity")
            if bv then bv:Destroy() end
        end
        
        local hum = player.Character:FindFirstChild("Humanoid")
        if hum then
            hum.PlatformStand = false
        end
    end
    
    joystickFrame.Visible = false
    joystickActive = false
    inputObject = nil
    joystickPosition = Vector2.new()
    joystickThumb.Position = UDim2.new(0.5, -25, 0.5, -25)
    
    flyButton.Text = "FLY"
    flyButton.BackgroundColor3 = Color3.fromRGB(140, 30, 255)
end

-- Button connections
minusButton.MouseButton1Click:Connect(function()
    flySpeed = math.max(10, flySpeed - 10)
    speedLabel.Text = "Speed: "..flySpeed
end)

plusButton.MouseButton1Click:Connect(function()
    flySpeed = math.min(500, flySpeed + 10)
    speedLabel.Text = "Speed: "..flySpeed
end)

flyButton.MouseButton1Click:Connect(function()
    if flying then
        stopFly()
    else
        startFly()
    end
end)

-- Handle respawn
player.CharacterAdded:Connect(function(newChar)
    if flying then
        stopFly()
    end
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
end)
