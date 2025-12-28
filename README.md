-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- VARIÁVEIS DO SPEED
local speedConn
local baseSpeed = 27
local active = false

-- FUNÇÕES DE PERSONAGEM
local function GetCharacter()
	local Char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
	local HRP = Char:WaitForChild("HumanoidRootPart")
	local Hum = Char:FindFirstChildOfClass("Humanoid")
	return Char, HRP, Hum
end

local function getMovementInput()
	local Char, HRP, Hum = GetCharacter()
	if not Char or not HRP or not Hum then
		return Vector3.new(0,0,0)
	end

	local moveVector = Hum.MoveDirection
	if moveVector.Magnitude > 0.1 then
		return Vector3.new(moveVector.X, 0, moveVector.Z).Unit
	end

	return Vector3.new(0,0,0)
end

local function startSpeedControl()
	if speedConn then return end

	speedConn = RunService.Heartbeat:Connect(function()
		local Char, HRP, Hum = GetCharacter()
		if not Char or not HRP or not Hum then return end

		local inputDirection = getMovementInput()
		if inputDirection.Magnitude > 0 then
			HRP.AssemblyLinearVelocity = Vector3.new(
				inputDirection.X * baseSpeed,
				HRP.AssemblyLinearVelocity.Y,
				inputDirection.Z * baseSpeed
			)
		else
			HRP.AssemblyLinearVelocity = Vector3.new(
				0,
				HRP.AssemblyLinearVelocity.Y,
				0
			)
		end
	end)
end

local function stopSpeedControl()
	if speedConn then
		speedConn:Disconnect()
		speedConn = nil
	end

	local _, HRP = GetCharacter()
	if HRP then
		HRP.AssemblyLinearVelocity = Vector3.new(0, HRP.AssemblyLinearVelocity.Y, 0)
	end
end

-- ================= GUI =================

local gui = Instance.new("ScreenGui")
gui.Name = "GuiSimples"
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local speedButton = Instance.new("ImageButton")
speedButton.Name = "Icone"
speedButton.Parent = gui
speedButton.Size = UDim2.new(0, 80, 0, 80)
speedButton.Position = UDim2.new(0, 20, 0, 20)
speedButton.BackgroundTransparency = 1

-- ÍCONE
speedButton.Image = "rbxassetid://7734053495"

-- CLIQUE NO ÍCONE
speedButton.MouseButton1Click:Connect(function()
	active = not active

	if active then
		startSpeedControl()
	else
		stopSpeedControl()
	end
end)
