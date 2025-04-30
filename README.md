-- LocalScript dentro de StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "NoclipTestGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 100, 0, 40)
button.Position = UDim2.new(0.05, 0, 0.9, 0)
button.Text = "Noclip: OFF"
button.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
button.TextColor3 = Color3.new(1,1,1)
button.Parent = screenGui

-- Noclip logic
local noclipActive = false

local function toggleNoclip()
	noclipActive = not noclipActive
	button.Text = "Noclip: " .. (noclipActive and "ON" or "OFF")
	button.BackgroundColor3 = noclipActive and Color3.fromRGB(85, 255, 85) or Color3.fromRGB(255, 85, 85)
end

button.MouseButton1Click:Connect(toggleNoclip)

-- Loop que roda noclip
RunService.RenderStepped:Connect(function()
	if noclipActive then
		local char = player.Character
		if char then
			for _, part in pairs(char:GetDescendants()) do
				if part:IsA("BasePart") and part.CanCollide then
					part.CanCollide = false
				end
			end
		end
	end
end)
