-- CompleteGUI_FlyingPlatform3D.lua
-- LocalScript (StarterPlayer > StarterPlayerScripts)

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- referências do personagem
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")

-- variáveis
local jumpActive = false
local skyActive = false
local antiActive = false
local platform
local workspaceGravity = 196.2
local moonGravity = 120
local platformOffset = Vector3.new(0,5,0)
local platformFollowConn

-- GUI principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CompleteGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 280, 0, 240)
frame.Position = UDim2.new(0.5, -140, 0.7, 0)
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0,16)
corner.Parent = frame

-- função criar botão
local function createButton(name, posY)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -20, 0, 40)
	btn.Position = UDim2.new(0,10,0,posY)
	btn.Text = name .. ": OFF"
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.FredokaOne
	btn.TextScaled = true
	btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
	btn.Parent = frame

	local bcorner = Instance.new("UICorner")
	bcorner.CornerRadius = UDim.new(0,10)
	bcorner.Parent = btn
	return btn
end

-- Botões
local jumpBtn = createButton("Jump Boost", 10)
local skyBtn = createButton("Sky Platform", 60)
local antiBtn = createButton("Anti-Ragdoll", 110)
local wallBtn = createButton("Wall Transparency", 160)

-- Textbox para JumpPower
local jumpBox = Instance.new("TextBox")
jumpBox.Size = UDim2.new(1, -20, 0, 30)
jumpBox.Position = UDim2.new(0,10,0,210)
jumpBox.Text = "75"
jumpBox.PlaceholderText = "Digite JumpPower"
jumpBox.TextColor3 = Color3.new(1,1,1)
jumpBox.Font = Enum.Font.FredokaOne
jumpBox.TextScaled = true
jumpBox.BackgroundColor3 = Color3.fromRGB(40,40,40)
jumpBox.Parent = frame

local boxCorner = Instance.new("UICorner")
boxCorner.CornerRadius = UDim.new(0,8)
boxCorner.Parent = jumpBox

-- variáveis auxiliares
local heartbeatConn
local wallConn

-- Jump Boost Moon Jump
jumpBtn.MouseButton1Click:Connect(function()
	jumpActive = not jumpActive
	if jumpActive then
		pcall(function()
			humanoid.UseJumpPower = true
			humanoid.JumpPower = tonumber(jumpBox.Text) or 75
			Workspace.Gravity = moonGravity
		end)
		jumpBtn.Text = "Jump Boost: ON"

		-- Moon Jump
		if heartbeatConn then heartbeatConn:Disconnect() end
		heartbeatConn = RunService.Heartbeat:Connect(function()
			if jumpActive and humanoid.FloorMaterial == Enum.Material.Air then
				local vel = hrp.Velocity
				if vel.Y < 0 then
					pcall(function()
						hrp.Velocity = Vector3.new(vel.X, vel.Y * 0.6, vel.Z)
					end)
				end
			end
		end)
	else
		pcall(function()
			humanoid.JumpPower = 50
			Workspace.Gravity = 196.2
		end)
		jumpBtn.Text = "Jump Boost: OFF"
		if heartbeatConn then heartbeatConn:Disconnect() heartbeatConn = nil end
	end
end)

-- Atualiza JumpPower ao digitar valor
jumpBox.FocusLost:Connect(function(enterPressed)
	if enterPressed and jumpActive then
		local val = tonumber(jumpBox.Text)
		if val then
			pcall(function() humanoid.JumpPower = val end)
		end
	end
end)

-- Sky Platform 3D dinâmica
skyBtn.MouseButton1Click:Connect(function()
	skyActive = not skyActive
	if skyActive then
		if not platform then
			pcall(function()
				platform = Instance.new("Part")
				platform.Size = Vector3.new(40,2,40)
				platform.Anchored = true
				platform.Material = Enum.Material.Neon
				platform.Transparency = 0.3
				platform.Color = Color3.fromRGB(0,170,255)
				platform.Name = "SkyPlatform_"..tostring(math.random(1000,9999))
				platform.Parent = Workspace
			end)
		end

		if platformFollowConn then platformFollowConn:Disconnect() end
		platformFollowConn = RunService.Heartbeat:Connect(function()
			if hrp and platform then
				pcall(function()
					-- plataforma segue player X/Y/Z (base voadora completa)
					platform.Position = hrp.Position - Vector3.new(0, -platformOffset.Y, 0)
				end)
			end
		end)

		skyBtn.Text = "Sky Platform: ON"
	else
		if platformFollowConn then platformFollowConn:Disconnect() platformFollowConn = nil end
		if platform then pcall(function() platform:Destroy() end) platform = nil end
		skyBtn.Text = "Sky Platform: OFF"
	end
end)

-- Anti-Ragdoll
local function removeRagdollConstraints(model)
	for _,v in pairs(model:GetDescendants()) do
		if v:IsA("BallSocketConstraint") or v:IsA("HingeConstraint") then
			pcall(function() v:Destroy() end)
		end
	end
end

antiBtn.MouseButton1Click:Connect(function()
	antiActive = not antiActive
	if antiActive then
		removeRagdollConstraints(char)
		char.DescendantAdded:Connect(function(desc)
			if antiActive and (desc:IsA("BallSocketConstraint") or desc:IsA("HingeConstraint")) then
				pcall(function() desc:Destroy() end)
			end
		end)
		antiBtn.Text = "Anti-Ragdoll: ON"
	else
		antiBtn.Text = "Anti-Ragdoll: OFF"
	end
end)

-- Wall Transparency
wallBtn.MouseButton1Click:Connect(function()
	if wallConn then
		wallConn:Disconnect()
		wallConn = nil
		wallBtn.Text = "Wall Transparency: OFF"
	else
		wallConn = RunService.Heartbeat:Connect(function()
			for _,part in pairs(Workspace:GetDescendants()) do
				if part:IsA("Part") and part.CanCollide and part.Anchored then
					pcall(function() part.Transparency = 0.5 end)
				end
			end
		end)
		wallBtn.Text = "Wall Transparency: ON"
	end
end)

-- Atualiza referências ao respawnar
player.CharacterAdded:Connect(function(newChar)
	char = newChar
	humanoid = newChar:WaitForChild("Humanoid")
	hrp = newChar:WaitForChild("HumanoidRootPart")
end)
