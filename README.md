if _G.TwistedHubLoaded then return end
_G.TwistedHubLoaded = true

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

local FullBright = false
local ESP = false

--------------------------------------------------
-- FULLBRIGHT / ANTI FOG (NÃO MODIFICADO)
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
-- GUI
--------------------------------------------------

local gui = Instance.new("ScreenGui")
gui.Name = "MgzHub"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = game:GetService("CoreGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,280,0,190)
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
-- TITLE
--------------------------------------------------

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,35)
title.BackgroundTransparency = 1
title.Text = "⚡ MGZ HUB"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(0,200,255)
title.Parent = frame

--------------------------------------------------
-- DRAG SYSTEM
--------------------------------------------------

local dragging = false
local dragStart
local startPos

title.InputBegan:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
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

        frame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )

    end

end)

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
-- BOTÃO PADRÃO
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
-- ESP (NÃO MODIFICADO)
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
