local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

local FullBright = false
local ESP = false
local FreeCam = false
local PerformanceMode = false

--------------------------------------------------
-- FULLBRIGHT
--------------------------------------------------

local function EnableFullBright()
    Lighting.Brightness = 3
    Lighting.ClockTime = 14
    Lighting.FogEnd = 1000000
    Lighting.FogStart = 0
    Lighting.GlobalShadows = false
    Lighting.Ambient = Color3.fromRGB(255,255,255)
    Lighting.OutdoorAmbient = Color3.fromRGB(255,255,255)
end

local function DisableFullBright()
    Lighting.Brightness = 1
    Lighting.GlobalShadows = true
end

RunService.RenderStepped:Connect(function()
    if FullBright then
        EnableFullBright()
    end
end)

--------------------------------------------------
-- PERFORMANCE MODE (SEM MEXER NA DISTÂNCIA)
--------------------------------------------------

local function EnablePerformance()
    for _,v in pairs(workspace:GetDescendants()) do

        if v:IsA("Texture") or v:IsA("Decal") then
            v:Destroy()
        end

        if v:IsA("ParticleEmitter") then
            v.Enabled = false
        end

        if v:IsA("BasePart") then
            v.Material = Enum.Material.Plastic
            v.Reflectance = 0
        end

    end
end

--------------------------------------------------
-- GUI
--------------------------------------------------

local gui = Instance.new("ScreenGui")
gui.Name = "MgzHub"
gui.ResetOnSpawn = false
gui.Parent = game.CoreGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,360,0,260)
frame.Position = UDim2.new(0,120,0,120)
frame.BackgroundColor3 = Color3.fromRGB(30,0,0)
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0,12)
corner.Parent = frame

local stroke = Instance.new("UIStroke")
stroke.Thickness = 2
stroke.Color = Color3.fromRGB(255,0,0)
stroke.Parent = frame

--------------------------------------------------
-- TITLE
--------------------------------------------------

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,35)
title.BackgroundTransparency = 1
title.Text = "🔥 MGZ HUB"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(255,80,80)
title.Parent = frame

--------------------------------------------------
-- FPS COUNTER
--------------------------------------------------

local fpsLabel = Instance.new("TextLabel")
fpsLabel.Size = UDim2.new(1,0,0,25)
fpsLabel.Position = UDim2.new(0,0,0,40)
fpsLabel.BackgroundTransparency = 1
fpsLabel.Font = Enum.Font.Gotham
fpsLabel.TextSize = 16
fpsLabel.TextColor3 = Color3.fromRGB(255,255,255)
fpsLabel.Text = "FPS: ..."
fpsLabel.Parent = frame

local frames = 0
local last = tick()

RunService.RenderStepped:Connect(function()

    frames += 1

    if tick() - last >= 1 then
        fpsLabel.Text = "FPS: "..frames
        frames = 0
        last = tick()
    end

end)

--------------------------------------------------
-- BUTTON
--------------------------------------------------

local function CreateButton(text,pos)

    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.9,0,0,34)
    button.Position = UDim2.new(0.05,0,0,pos)
    button.BackgroundColor3 = Color3.fromRGB(60,0,0)
    button.Font = Enum.Font.GothamBold
    button.TextSize = 15
    button.TextColor3 = Color3.new(1,1,1)
    button.Text = text
    button.Parent = frame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,8)
    corner.Parent = button

    return button
end

--------------------------------------------------
-- FULLBRIGHT BUTTON
--------------------------------------------------

local fbButton = CreateButton("FULLBRIGHT: OFF",75)

fbButton.MouseButton1Click:Connect(function()

    FullBright = not FullBright

    if FullBright then
        fbButton.Text = "FULLBRIGHT: ON"
        EnableFullBright()
    else
        fbButton.Text = "FULLBRIGHT: OFF"
        DisableFullBright()
    end

end)

--------------------------------------------------
-- ESP
--------------------------------------------------

local espObjects = {}

local function CreateESP(player)

    if player == LocalPlayer then return end

    player.CharacterAdded:Connect(function(char)

        if ESP then

            local highlight = Instance.new("Highlight")
            highlight.FillTransparency = 1
            highlight.OutlineColor = Color3.fromRGB(255,0,0)
            highlight.Parent = char

            espObjects[player] = highlight

        end

    end)

end

for _,player in pairs(Players:GetPlayers()) do
    CreateESP(player)
end

Players.PlayerAdded:Connect(CreateESP)

--------------------------------------------------
-- ESP BUTTON
--------------------------------------------------

local espButton = CreateButton("ESP: OFF",115)

espButton.MouseButton1Click:Connect(function()

    ESP = not ESP

    if ESP then

        espButton.Text = "ESP: ON"

        for _,player in pairs(Players:GetPlayers()) do

            if player.Character then

                local highlight = Instance.new("Highlight")
                highlight.FillTransparency = 1
                highlight.OutlineColor = Color3.fromRGB(255,0,0)
                highlight.Parent = player.Character

                espObjects[player] = highlight

            end

        end

    else

        espButton.Text = "ESP: OFF"

        for _,esp in pairs(espObjects) do
            esp:Destroy()
        end

        espObjects = {}

    end

end)

--------------------------------------------------
-- FREECAM
--------------------------------------------------

local camera = workspace.CurrentCamera
local speed = 2
local sensitivity = 0.2

local rotX = 0
local rotY = 0

UIS.InputChanged:Connect(function(input)

    if FreeCam and input.UserInputType == Enum.UserInputType.MouseMovement then
        
        rotX = rotX - input.Delta.Y * sensitivity
        rotY = rotY - input.Delta.X * sensitivity

        camera.CFrame = CFrame.new(camera.CFrame.Position) *
            CFrame.Angles(math.rad(rotX), math.rad(rotY), 0)

    end

end)

RunService.RenderStepped:Connect(function()

    if FreeCam then

        local move = Vector3.new()

        if UIS:IsKeyDown(Enum.KeyCode.W) then
            move = move + camera.CFrame.LookVector
        end

        if UIS:IsKeyDown(Enum.KeyCode.S) then
            move = move - camera.CFrame.LookVector
        end

        if UIS:IsKeyDown(Enum.KeyCode.A) then
            move = move - camera.CFrame.RightVector
        end

        if UIS:IsKeyDown(Enum.KeyCode.D) then
            move = move + camera.CFrame.RightVector
        end

        if UIS:IsKeyDown(Enum.KeyCode.Space) then
            move = move + Vector3.new(0,1,0)
        end

        if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then
            move = move - Vector3.new(0,1,0)
        end

        camera.CFrame = camera.CFrame + move * speed

    end

end)

local freecamButton = CreateButton("FREECAM: OFF",155)

freecamButton.MouseButton1Click:Connect(function()

    FreeCam = not FreeCam

    if FreeCam then
        freecamButton.Text = "FREECAM: ON"
        camera.CameraType = Enum.CameraType.Scriptable
        UIS.MouseBehavior = Enum.MouseBehavior.LockCenter
    else
        freecamButton.Text = "FREECAM: OFF"
        camera.CameraType = Enum.CameraType.Custom
        UIS.MouseBehavior = Enum.MouseBehavior.Default
    end

end)

--------------------------------------------------
-- PERFORMANCE BUTTON
--------------------------------------------------

local perfButton = CreateButton("PERFORMANCE: OFF",195)

perfButton.MouseButton1Click:Connect(function()

    PerformanceMode = not PerformanceMode

    if PerformanceMode then
        perfButton.Text = "PERFORMANCE: ON"
        EnablePerformance()
    else
        perfButton.Text = "PERFORMANCE: OFF"
    end

end)

--------------------------------------------------
-- MOBILE BUTTON
--------------------------------------------------

local openButton = Instance.new("TextButton")
openButton.Size = UDim2.new(0,38,0,38)
openButton.Position = UDim2.new(0,20,0.5,0)
openButton.BackgroundColor3 = Color3.fromRGB(255,0,0)
openButton.Text = "MGZ"
openButton.TextColor3 = Color3.new(1,1,1)
openButton.Font = Enum.Font.GothamBold
openButton.TextSize = 14
openButton.Parent = gui

local corner2 = Instance.new("UICorner")
corner2.CornerRadius = UDim.new(1,0)
corner2.Parent = openButton

openButton.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)

--------------------------------------------------
-- MOBILE DRAG
--------------------------------------------------

local dragging = false
local dragStart
local startPos

openButton.InputBegan:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = openButton.Position
    end

end)

UIS.InputEnded:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end

end)

UIS.InputChanged:Connect(function(input)

    if dragging then

        local delta = input.Position - dragStart

        openButton.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )

    end

end)
