-- LocalScript dentro de StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Espera personagem
local function getCharacter()
	local character = player.Character or player.CharacterAdded:Wait()
	return character
end

-- Criar GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "NoclipTestGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 100, 0, 40)
button.Position = UDim2.new(0.05, 0, 0.9, 0)
button.Text = "Noclip: OFF"
button.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.SourceSansBold
button.TextSize = 18
button.Parent = screenGui

-- Variáveis
local noclipActive = false
local humanoid = nil

-- Função de toggle
local function toggleNoclip()
	noclipActive = not noclipActive
	button.Text = "Noclip: " .. (noclipActive and "ON" or "OFF")
	button.BackgroundColor3 = noclipActive and Color3.fromRGB(85, 255, 85) or Color3.fromRGB(255, 85, 85)

	-- Troca o estado do Humanoid pra evitar travamentos
	local character = getCharacter()
	humanoid = character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		if noclipActive then
			humanoid:ChangeState(Enum.HumanoidStateType.Physics)
		else
			humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
		end
	end
end

-- Conectar botão
button.MouseButton1Click:Connect(toggleNoclip)

-- Loop contínuo para aplicar noclip
RunService.RenderStepped:Connect(function()
	if noclipActive then
		local character = getCharacter()
		if humanoid and character then
			-- Garante que as partes do personagem ficam sem colisão, mas com exceção do pé
			for _, part in ipairs(character:GetDescendants()) do
				if part:IsA("BasePart") then
					-- Deixa os "pés" com colisão pra não cair
					if part.Name ~= "LeftFoot" and part.Name ~= "RightFoot" and part.Name ~= "LowerTorso" then
						part.CanCollide = false
					end
				end
			end

			-- Garante que o personagem se move corretamente durante o noclip
			if humanoid.MoveDirection.Magnitude > 0 then
				-- Permite o movimento
				humanoid:Move(Vector3.new(0, 0, 0))  -- Aqui, mantém a posição do personagem
			end
		end
	end
end)
