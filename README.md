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
-- PERFORMANCE MODE
--------------------------------------------------

local function EnablePerformance()

for _,v in pairs(workspace:GetDescendants()) do

if v:IsA("Texture") or v:IsA("Decal") then
v:Destroy()
end

if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then
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
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = game:GetService("CoreGui")

--------------------------------------------------
-- FRAME
--------------------------------------------------

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,280,0,220)
frame.Position = UDim2.new(0,120,0,120)
frame.BackgroundColor3 = Color3.fromRGB(18,18,25)
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0,12)
corner.Parent = frame

local stroke = Instance.new("UIStroke")
stroke.Thickness = 2
stroke.Color = Color3.fromRGB(0,170,255)
stroke.Parent = frame

--------------------------------------------------
-- MOBILE BUTTON
--------------------------------------------------

local openButton = Instance.new("TextButton")
openButton.Size = UDim2.new(0,60,0,60)
openButton.Position = UDim2.new(0,20,0.5,-30)
openButton.BackgroundColor3 = Color3.fromRGB(40,40,40)
openButton.Text = "MGZ"
openButton.TextColor3 = Color3.new(1,1,1)
openButton.Font = Enum.Font.GothamBold
openButton.TextSize = 18
openButton.Parent = gui

local openCorner = Instance.new("UICorner")
openCorner.CornerRadius = UDim.new(1,0)
openCorner.Parent = openButton

openButton.MouseButton1Click:Connect(function()
frame.Visible = not frame.Visible
end)

--------------------------------------------------
-- PC KEY
--------------------------------------------------

UIS.InputBegan:Connect(function(input,gpe)

if gpe then return end

if input.KeyCode == Enum.KeyCode.RightShift then
frame.Visible = not frame.Visible
end

end)

--------------------------------------------------
-- TITLE
--------------------------------------------------

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,35)
title.BackgroundTransparency = 1
title.Text = "🌪 MGZ HUB"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(0,200,255)
title.Parent = frame

--------------------------------------------------
-- MINIMIZE
--------------------------------------------------

local minimize = Instance.new("TextButton")
minimize.Size = UDim2.new(0,30,0,25)
minimize.Position = UDim2.new(1,-35,0,5)
minimize.Text = "-"
minimize.Font = Enum.Font.GothamBold
minimize.TextSize = 18
minimize.BackgroundColor3 = Color3.fromRGB(35,35,45)
minimize.TextColor3 = Color3.new(1,1,1)
minimize.Parent = frame

local minimized = false

--------------------------------------------------
-- FPS
--------------------------------------------------

local fpsLabel = Instance.new("TextLabel")
fpsLabel.Size = UDim2.new(1,0,0,25)
fpsLabel.Position = UDim2.new(0,0,0,40)
fpsLabel.BackgroundTransparency = 1
fpsLabel.Font = Enum.Font.Gotham
fpsLabel.TextSize = 16
fpsLabel.TextColor3 = Color3.new(1,1,1)
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
-- BUTTON CREATOR
--------------------------------------------------

local function CreateButton(text,pos)

local button = Instance.new("TextButton")
button.Size = UDim2.new(0.9,0,0,32)
button.Position = UDim2.new(0.05,0,0,pos)
button.BackgroundColor3 = Color3.fromRGB(35,35,45)
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

local espButton = CreateButton("ESP: OFF",115)

espButton.MouseButton1Click:Connect(function()

ESP = not ESP

if ESP then

espButton.Text = "ESP: ON"

for _,player in pairs(Players:GetPlayers()) do

if player ~= LocalPlayer and player.Character then

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
local freecamButton = CreateButton("FREECAM: OFF",155)

local speed = 2
local sensitivity = 0.25
local rotX = 0
local rotY = 0
local camPos = camera.CFrame.Position

UIS.InputChanged:Connect(function(input)

if FreeCam and input.UserInputType == Enum.UserInputType.MouseMovement then

rotX = rotX - input.Delta.Y * sensitivity
rotY = rotY - input.Delta.X * sensitivity

end

end)

RunService.RenderStepped:Connect(function()

if FreeCam then

local move = Vector3.new()

if UIS:IsKeyDown(Enum.KeyCode.W) then
move += camera.CFrame.LookVector
end

if UIS:IsKeyDown(Enum.KeyCode.S) then
move -= camera.CFrame.LookVector
end

if UIS:IsKeyDown(Enum.KeyCode.A) then
move -= camera.CFrame.RightVector
end

if UIS:IsKeyDown(Enum.KeyCode.D) then
move += camera.CFrame.RightVector
end

if UIS:IsKeyDown(Enum.KeyCode.Space) then
move += Vector3.new(0,1,0)
end

if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then
move -= Vector3.new(0,1,0)
end

camPos += move * speed

camera.CFrame =
CFrame.new(camPos) *
CFrame.Angles(math.rad(rotX), math.rad(rotY),0)

end

end)

freecamButton.MouseButton1Click:Connect(function()

FreeCam = not FreeCam

if FreeCam then

freecamButton.Text = "FREECAM: ON"
camera.CameraType = Enum.CameraType.Scriptable
UIS.MouseBehavior = Enum.MouseBehavior.LockCenter
camPos = camera.CFrame.Position

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
-- MINIMIZE
--------------------------------------------------

minimize.MouseButton1Click:Connect(function()

minimized = not minimized

for _,v in pairs(frame:GetChildren()) do
if v:IsA("TextButton") or v:IsA("TextLabel") then
if v ~= title and v ~= minimize then
v.Visible = not minimized
end
end
end

if minimized then
frame.Size = UDim2.new(0,280,0,40)
else
frame.Size = UDim2.new(0,280,0,220)
end

end)
