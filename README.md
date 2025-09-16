-- Script para colocar no ServerScriptService
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Criar RemoteEvent para comunicação cliente-servidor
local toggleInvisibilityEvent = Instance.new("RemoteEvent")
toggleInvisibilityEvent.Name = "ToggleInvisibility"
toggleInvisibilityEvent.Parent = ReplicatedStorage

-- Função para tornar jogador invisível/visível
local function togglePlayerVisibility(player)
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    -- Verifica se já está invisível
    local isInvisible = character:GetAttribute("Invisible") or false
    
    if isInvisible then
        -- Tornar visível
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.Transparency = 0
            elseif part:IsA("Accessory") then
                local handle = part:FindFirstChild("Handle")
                if handle then
                    handle.Transparency = 0
                end
            end
        end
        
        -- Restaurar acessórios
        local head = character:FindFirstChild("Head")
        if head then
            for _, child in pairs(head:GetChildren()) do
                if child:IsA("Decal") then
                    child.Transparency = 0
                end
            end
        end
        
        character:SetAttribute("Invisible", false)
    else
        -- Tornar invisível
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.Transparency = 1
            elseif part:IsA("Accessory") then
                local handle = part:FindFirstChild("Handle")
                if handle then
                    handle.Transparency = 1
                end
            end
        end
        
        -- Ocultar rosto/decals
        local head = character:FindFirstChild("Head")
        if head then
            for _, child in pairs(head:GetChildren()) do
                if child:IsA("Decal") then
                    child.Transparency = 1
                end
            end
        end
        
        character:SetAttribute("Invisible", true)
    end
end

-- Conectar evento
toggleInvisibilityEvent.OnServerEvent:Connect(function(player)
    togglePlayerVisibility(player)
end)

-- LocalScript para colocar no StarterPlayerScripts
--[[
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Aguardar o RemoteEvent
local toggleInvisibilityEvent = ReplicatedStorage:WaitForChild("ToggleInvisibility")

-- Criar GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "InvisibilityGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Frame principal (arrastável)
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 200, 0, 100)
mainFrame.Position = UDim2.new(0, 50, 0, 50)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

-- Cantos arredondados
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = mainFrame

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Size = UDim2.new(1, 0, 0, 30)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Sistema de Invisibilidade"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame

-- Botão de invisibilidade
local invisButton = Instance.new("TextButton")
invisButton.Name = "InvisButton"
invisButton.Size = UDim2.new(0.8, 0, 0, 40)
invisButton.Position = UDim2.new(0.1, 0, 0.4, 0)
invisButton.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
invisButton.BorderSizePixel = 0
invisButton.Text = "Invisível"
invisButton.TextColor3 = Color3.fromRGB(255, 255, 255)
invisButton.TextScaled = true
invisButton.Font = Enum.Font.Gotham
invisButton.Parent = mainFrame

-- Cantos arredondados do botão
local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 8)
buttonCorner.Parent = invisButton

-- Status do jogador
local isInvisible = false

-- Função de animação do botão
local function animateButton(button, scale)
    local tween = TweenService:Create(
        button,
        TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {Size = button.Size * scale}
    )
    tween:Play()
    
    tween.Completed:Connect(function()
        local reverseTween = TweenService:Create(
            button,
            TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Size = button.Size / scale}
        )
        reverseTween:Play()
    end)
end

-- Função do botão
invisButton.MouseButton1Click:Connect(function()
    -- Animação do botão
    animateButton(invisButton, 1.1)
    
    -- Alternar estado
    isInvisible = not isInvisible
    
    -- Atualizar texto e cor do botão
    if isInvisible then
        invisButton.Text = "Visível"
        invisButton.BackgroundColor3 = Color3.fromRGB(255, 87, 87)
    else
        invisButton.Text = "Invisível"
        invisButton.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
    end
    
    -- Enviar evento para o servidor
    toggleInvisibilityEvent:FireServer()
end)

-- Efeito hover
invisButton.MouseEnter:Connect(function()
    local tween = TweenService:Create(
        invisButton,
        TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {BackgroundColor3 = invisButton.BackgroundColor3:lerp(Color3.fromRGB(255, 255, 255), 0.1)}
    )
    tween:Play()
end)

invisButton.MouseLeave:Connect(function()
    local targetColor = isInvisible and Color3.fromRGB(255, 87, 87) or Color3.fromRGB(0, 162, 255)
    local tween = TweenService:Create(
        invisButton,
        TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {BackgroundColor3 = targetColor}
    )
    tween:Play()
end)
--]]
