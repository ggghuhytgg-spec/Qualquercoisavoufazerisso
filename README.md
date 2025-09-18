-- FloatSupermanGUI.lua
-- LocalScript (StarterPlayer > StarterPlayerScripts)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char, hrp, humanoid

-- CONFIG DEFAULT
local FLOAT_HEIGHT = 12
local MOVE_SPEED_AIR = 24

-- FLAGS
local floating = false
local airwalk = false
local godmode = false

local vectorForce, att, alignOri

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "FloatSupermanGUI"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 220, 0, 200)
main.Position = UDim2.new(0.7, 0, 0.2, 0)
main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
main.Active = true
main.Draggable = true
main.Parent = gui

Instance.new("UICorner", main).CornerRadius = UDim.new(0,16)
local shadow = Instance.new("UIStroke", main)
shadow.Thickness = 2
shadow.Color = Color3.fromRGB(0,150,255)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "⚡ Float Superman"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.Parent = main

-- Função criar botão
local function createButton(text,posY,color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0,180,0,30)
    btn.Position = UDim2.new(0.5,-90,0,posY)
    btn.Text = text
    btn.BackgroundColor3 = color
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.Parent = main
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,12)
    return btn
end

-- Função criar sliders
local function createSlider(name,posY,default,callback)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0,100,0,20)
    label.Position = UDim2.new(0,10,0,posY)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.fromRGB(255,255,255)
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = main

    local box = Instance.new("TextBox")
    box.Size = UDim2.new(0,80,0,20)
    box.Position = UDim2.new(0,120,0,posY)
    box.BackgroundColor3 = Color3.fromRGB(40,40,40)
    box.TextColor3 = Color3.fromRGB(255,255,255)
    box.Font = Enum.Font.Gotham
    box.TextSize = 14
    box.Text = tostring(default)
    box.Parent = main
    Instance.new("UICorner", box).CornerRadius = UDim.new(0,8)

    box.FocusLost:Connect(function()
        local num = tonumber(box.Text)
        if num then callback(num) end
    end)
end

-- Botões
local toggle = createButton("Ativar Float",40,Color3.fromRGB(0,120,255))
local airwalkBtn = createButton("Ativar AirWalk",75,Color3.fromRGB(150,0,255))
local godBtn = createButton("Ativar GodMode",110,Color3.fromRGB(0,200,100))

-- Sliders
createSlider("Altura:",140,FLOAT_HEIGHT,function(v) FLOAT_HEIGHT = v end)
createSlider("Velocidade:",170,MOVE_SPEED_AIR,function(v) MOVE_SPEED_AIR = v end)

-- Setup Character
local function setupCharacter(c)
    char = c
    humanoid = char:WaitForChild("Humanoid")
    hrp = char:WaitForChild("HumanoidRootPart")
end
if player.Character then setupCharacter(player.Character) end
player.CharacterAdded:Connect(setupCharacter)

-- Float
local function enableFloat()
    if floating or not hrp then return end
    floating = true
    att = Instance.new("Attachment",hrp)
    vectorForce = Instance.new("VectorForce")
    vectorForce.Attachment0 = att
    vectorForce.RelativeTo = Enum.ActuatorRelativeTo.World
    vectorForce.Force = Vector3.new()
    vectorForce.Parent = hrp

    alignOri = Instance.new("AlignOrientation")
    alignOri.Attachment0 = att
    alignOri.Responsiveness = 50
    alignOri.MaxTorque = 1e6
    alignOri.Parent = hrp

    toggle.Text = "Desativar Float"
    toggle.BackgroundColor3 = Color3.fromRGB(200,50,50)
end

local function disableFloat()
    floating = false
    if vectorForce then vectorForce:Destroy() vectorForce = nil end
    if alignOri then alignOri:Destroy() alignOri = nil end
    if att then att:Destroy() att = nil end
    toggle.Text = "Ativar Float"
    toggle.BackgroundColor3 = Color3.fromRGB(0,120,255)
end

toggle.MouseButton1Click:Connect(function()
    if floating then disableFloat() else enableFloat() end
end)

-- AirWalk
airwalkBtn.MouseButton1Click:Connect(function()
    airwalk = not airwalk
    if airwalk and hrp then
        airwalkBtn.Text = "Desativar AirWalk"
        airwalkBtn.BackgroundColor3 = Color3.fromRGB(200,50,50)
        humanoid.PlatformStand = true
    else
        airwalkBtn.Text = "Ativar AirWalk"
        airwalkBtn.BackgroundColor3 = Color3.fromRGB(150,0,255)
        humanoid.PlatformStand = false
    end
end)

-- GodMode travado
godBtn.MouseButton1Click:Connect(function()
    godmode = not godmode
    if godmode then
        godBtn.Text = "Desativar GodMode"
        godBtn.BackgroundColor3 = Color3.fromRGB(200,50,50)
    else
        godBtn.Text = "Ativar GodMode"
        godBtn.BackgroundColor3 = Color3.fromRGB(0,200,100)
    end
end)

-- Loop principal
RunService.Heartbeat:Connect(function(dt)
    -- Float
    if floating and hrp then
        local moveDir = humanoid.MoveDirection
        local desiredVel = Vector3.new(moveDir.X,0,moveDir.Z) * MOVE_SPEED_AIR
        vectorForce.Force = (desiredVel - hrp.Velocity) * hrp.AssemblyMass
        alignOri.CFrame = CFrame.new(hrp.Position, hrp.Position + (moveDir.Magnitude>0 and moveDir or hrp.CFrame.LookVector))
    end

    -- AirWalk
    if airwalk and hrp then
        local moveDir = humanoid.MoveDirection
        hrp.Velocity = Vector3.new(moveDir.X * MOVE_SPEED_AIR, 0, moveDir.Z * MOVE_SPEED_AIR)
    end

    -- GodMode
    if godmode and humanoid then
        humanoid.MaxHealth = math.huge
        humanoid.Health = math.huge
    end
end)

print("[Float Superman GUI] carregado com Float, AirWalk e GodMode travado!")
