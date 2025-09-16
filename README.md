-- Hop Los Brothers (Minimize com Bolinha)
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Criar GUI principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HopLosBrothers"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 220)
MainFrame.Position = UDim2.new(0.35, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
MainFrame.BackgroundTransparency = 0.25 -- 75% transparente
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true -- Permite mover o script
MainFrame.Parent = ScreenGui

-- UICorner para bordas arredondadas
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 15)
corner.Parent = MainFrame

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -40, 0, 30)
Title.Position = UDim2.new(0, 10, 0, 5)
Title.Text = "Hop Los Brothers"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.BackgroundTransparency = 1
Title.TextScaled = true
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- TikTok
local TikTok = Instance.new("TextLabel")
TikTok.Size = UDim2.new(1, 0, 0, 20)
TikTok.Position = UDim2.new(0, 0, 1, -25)
TikTok.Text = "TikTok: LosBrothers.ofc"
TikTok.TextColor3 = Color3.fromRGB(255, 255, 255)
TikTok.BackgroundTransparency = 1
TikTok.TextScaled = true
TikTok.Font = Enum.Font.GothamBold
TikTok.Parent = MainFrame

-- Botão minimizar
local MinButton = Instance.new("TextButton")
MinButton.Size = UDim2.new(0, 25, 0, 25)
MinButton.Position = UDim2.new(1, -80, 0, 5)
MinButton.Text = "-"
MinButton.TextScaled = true
MinButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
MinButton.TextColor3 = Color3.fromRGB(0, 0, 0)
MinButton.Parent = MainFrame

-- Bolinha (vai aparecer ao minimizar)
local BallButton = Instance.new("ImageButton")
BallButton.Size = UDim2.new(0, 60, 0, 60)
BallButton.Position = UDim2.new(0, 10, 1, -70)
BallButton.Image = "rbxassetid://COLOQUE_AQUI_O_ID_DA_FOTO" -- <- VOCÊ TROCA PELO NÚMERO DA IMAGEM
BallButton.Visible = false
BallButton.BackgroundTransparency = 1
BallButton.Active = true
BallButton.Draggable = true
BallButton.Parent = ScreenGui

-- Função minimizar
MinButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    BallButton.Visible = true
end)

-- Clicar na bolinha para abrir de novo
BallButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    BallButton.Visible = false
end)
