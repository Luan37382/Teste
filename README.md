local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local button = script.Parent
local noclipEnabled = false
local flyHeight = 5

-- Alternar estado do noclip
local function toggleNoclip()
	noclipEnabled = not noclipEnabled
	button.Text = noclipEnabled and "Desativar Noclip" or "Ativar Noclip"
end

-- Clique no botão
button.MouseButton1Click:Connect(toggleNoclip)

-- Loop de controle
RunService.Stepped:Connect(function()
	if noclipEnabled then
		character = player.Character or player.CharacterAdded:Wait()
		humanoidRootPart = character:WaitForChild("HumanoidRootPart")

		for _, part in pairs(character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = false
			end
		end

		-- Evita que o jogador caia
		local ray = Ray.new(humanoidRootPart.Position, Vector3.new(0, -100, 0))
		local hit, pos = workspace:FindPartOnRay(ray, character)
		if hit then
			local targetY = pos.Y + flyHeight
			humanoidRootPart.Velocity = Vector3.zero
			humanoidRootPart.CFrame = CFrame.new(
				humanoidRootPart.Position.X,
				math.max(humanoidRootPart.Position.Y, targetY),
				humanoidRootPart.Position.Z
			)
		end
	end
end)
