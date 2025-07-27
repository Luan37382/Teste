local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variáveis globais
local aimActive = false
local espEnabled = false
local lockedTarget = nil
local fovSize = 100
local aimSmoothness = 0.2
local crosshairOffsetY = -0.3

-- Criar janela e abas
local Window = Rayfield:CreateWindow({
    Name = "TrueHub",
    LoadingTitle = "TrueHub Loading",
    LoadingSubtitle = "by TrueHubDEV",
    Theme = "Amethyst",
    ConfigurationSaving = {Enabled = true, FolderName = nil, FileName = "TrueHub"},
    Discord = {Enabled = true, Invite = "ufNv6TAtbd", RememberJoins = false},
    KeySystem = false
})

local AimTab = Window:CreateTab("Aim")
local ESPTab = Window:CreateTab("ESP")

-- Criar círculo FOV usando GUI
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "FOVGui"
local FOVCircle = Instance.new("Frame", ScreenGui)
FOVCircle.Size = UDim2.new(0, fovSize*2, 0, fovSize*2)
FOVCircle.Position = UDim2.new(0.5, -fovSize, 0.5, -fovSize)
FOVCircle.BackgroundColor3 = Color3.new(1, 0, 0)
FOVCircle.BackgroundTransparency = 0.7
FOVCircle.BorderSizePixel = 0
FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FOVCircle.Visible = false
FOVCircle.ClipsDescendants = true

-- Fazer o Frame parecer um círculo (usando UICorner)
local uicorner = Instance.new("UICorner", FOVCircle)
uicorner.CornerRadius = UDim.new(1,0)

-- Funções para AimLock Highlight
local currentHighlight = nil
local function createHighlight(character)
    if character then
        if currentHighlight then currentHighlight:Destroy() end
        local highlight = Instance.new("Highlight")
        highlight.Name = "AimLockHighlight"
        highlight.FillColor = Color3.new(1,0,0)
        highlight.FillTransparency = 0.7
        highlight.OutlineColor = Color3.new(1,1,1)
        highlight.OutlineTransparency = 0
        highlight.Adornee = character
        highlight.Parent = character
        currentHighlight = highlight
    end
end

local function removeHighlight()
    if currentHighlight then
        currentHighlight:Destroy()
        currentHighlight = nil
    end
end

local function releaseLock()
    lockedTarget = nil
    removeHighlight()
end

-- Função para buscar o jogador mais próximo do centro da tela dentro do FOV
local function getClosestTarget()
    local targetPart = nil
    local shortestDistance = math.huge
    local viewportSize = Camera.ViewportSize
    local center = Vector2.new(viewportSize.X/2, viewportSize.Y/2)
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local character = player.Character
            local aimPart = character:FindFirstChild("UpperTorso") or character:FindFirstChild("Torso") or character:FindFirstChild("HumanoidRootPart")
            if aimPart then
                local pos, onScreen = Camera:WorldToViewportPoint(aimPart.Position)
                if onScreen then
                    local screenPos = Vector2.new(pos.X, pos.Y)
                    local dist = (screenPos - center).Magnitude
                    if dist <= fovSize and dist < shortestDistance then
                        targetPart = aimPart
                        shortestDistance = dist
                    end
                end
            end
        end
    end
    return targetPart
end

-- Função principal para rodar o AimLock
RunService.RenderStepped:Connect(function()
    if aimActive then
        local targetPart = getClosestTarget()
        if targetPart then
            lockedTarget = targetPart
            createHighlight(lockedTarget.Parent)
            local targetPos = lockedTarget.Position
            local aimOffset = Vector3.new(0, 0, 0)
            if lockedTarget.Name == "Torso" then
                aimOffset = Vector3.new(0, -0.5, 0)
            elseif lockedTarget.Name == "UpperTorso" then
                aimOffset = Vector3.new(0, -1, 0)
            elseif lockedTarget.Name == "HumanoidRootPart" then
                aimOffset = Vector3.new(0, -1.5, 0)
            end
            targetPos = targetPos + aimOffset + Vector3.new(0, crosshairOffsetY, 0)
            local currentCF = Camera.CFrame
            local targetCF = CFrame.new(Camera.CFrame.Position, targetPos)
            Camera.CFrame = currentCF:Lerp(targetCF, 1 - aimSmoothness)
        else
            releaseLock()
        end
    else
        releaseLock()
    end
end)

-- Aim Toggle
AimTab:CreateToggle({
    Name = "Aim",
    CurrentValue = false,
    Flag = "AimToggle",
    Callback = function(value)
        aimActive = value
        FOVCircle.Visible = value
        if not value then
            releaseLock()
        end
    end
})

-- FOV Slider para alterar o tamanho do círculo
AimTab:CreateSlider({
    Name = "FOV Size",
    Range = {50, 300},
    Increment = 10,
    CurrentValue = fovSize,
    Flag = "FOVSlider",
    Callback = function(value)
        fovSize = value
        FOVCircle.Size = UDim2.new(0, fovSize * 2, 0, fovSize * 2)
        FOVCircle.Position = UDim2.new(0.5, -fovSize, 0.5, -fovSize)
    end
})

-- ESP
local ESPObjects = {}

local function updatePlayerESP(player)
    if player == LocalPlayer then return end
    local character = player.Character
    if character then
        local highlight = ESPObjects[player]
        if not highlight then
            highlight = Instance.new("Highlight")
            highlight.Name = "ESPHighlight"
            highlight.FillColor = Color3.fromRGB(0, 255, 0)
            highlight.OutlineColor = Color3.fromRGB(0, 0, 0)
            highlight.FillTransparency = 0.5
            highlight.OutlineTransparency = 0
            highlight.Adornee = character
            highlight.Parent = character
            ESPObjects[player] = highlight
        end
    end
end

-- Limpar ESP quando desligar
local function clearESP()
    for player, highlight in pairs(ESPObjects) do
        if highlight and highlight.Parent then
            highlight:Destroy()
        end
    end
    ESPObjects = {}
end

-- Loop para atualizar ESP se ativado
RunService.RenderStepped:Connect(function()
    if espEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            updatePlayerESP(player)
        end
    else
        clearESP()
    end
end)

-- ESP Toggle
ESPTab:CreateToggle({
    Name = "ESP On/Off",
    CurrentValue = false,
    Flag = "ESPToggle",
    Callback = function(value)
        espEnabled = value
        if not value then
            clearESP()
        end
    end
})
