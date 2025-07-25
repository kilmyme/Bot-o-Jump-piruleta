local player = game.Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")
local UserInputService = game:GetService("UserInputService")

pcall(function() PlayerGui:FindFirstChild("JumpGui"):Destroy() end)

local gui = Instance.new("ScreenGui")
gui.Name = "JumpGui"
gui.Parent = PlayerGui
gui.ResetOnSpawn = false

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 45, 0, 20)  -- botão menor agora
button.Position = UDim2.new(0.7, 0, 0.8, 0)
button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
button.TextColor3 = Color3.new(1,1,1)
button.TextScaled = true
button.Text = "Jump"
button.Parent = gui

local corner = Instance.new("UICorner", button)
corner.CornerRadius = UDim.new(1, 0)

local dragging, dragInput, dragStart, startPos = false
button.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = button.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

button.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

local spinAnim = Instance.new("Animation")
spinAnim.AnimationId = "rbxassetid://18537367238"

local function setupCharacter(char)
	local humanoid = char:WaitForChild("Humanoid")
	local hrp = char:WaitForChild("HumanoidRootPart")
	local currentAnim = nil

	local function stopSpin()
		if currentAnim and currentAnim.IsPlaying then
			currentAnim:Stop()
			currentAnim:Destroy()
			currentAnim = nil
		end
	end

	humanoid:GetPropertyChangedSignal("FloorMaterial"):Connect(function()
		if humanoid.FloorMaterial ~= Enum.Material.Air then
			stopSpin()
		end
	end)

	local function dash()
		local dir = humanoid.MoveDirection
		local horizontalPower = 180
		local verticalPower = 130
		local impulse = Vector3.new(0, verticalPower, 0) + dir.Unit * horizontalPower
		if dir.Magnitude == 0 then impulse = Vector3.new(0, verticalPower, 0) end
		hrp.Velocity = impulse

		task.delay(0.1, function()
			if humanoid.FloorMaterial == Enum.Material.Air then
				stopSpin()
				currentAnim = humanoid:LoadAnimation(spinAnim)
				currentAnim:Play()
			end
		end)
	end

	button.MouseButton1Click:Connect(dash)
	button.TouchTap:Connect(dash)
end

if player.Character then
	setupCharacter(player.Character)
end

player.CharacterAdded:Connect(function(char)
	task.wait(1.5)
	setupCharacter(char)
end)
