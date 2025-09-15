--// Hop Los Brothers v4.0 - Script Unificado
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")
local player = Players.LocalPlayer

-- Criar ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HopLosBrothers"
ScreenGui.DisplayOrder = 5 -- Fica atrás do menu de pause
ScreenGui.Parent = game.CoreGui

-- Criar Janela Principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 280, 0, 180)
MainFrame.Position = UDim2.new(0.35, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(135, 206, 250) -- azul claro
MainFrame.BackgroundTransparency = 0.25
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

-- Bordas Arredondadas
local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 10)
Corner.Parent = MainFrame

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -10, 0, 30)
Title.Position = UDim2.new(0, 5, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "Hop Los Brothers"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextColor3 = Color3.new(1, 1, 1)
Title.TextStrokeTransparency = 0.5
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = MainFrame

-- Função de Minimizar
local Minimized = false
local Bubble

local function toggleMinimize()
    if not Minimized then
        MainFrame.Visible = false
        Bubble.Visible = true
    else
        MainFrame.Visible = true
        Bubble.Visible = false
    end
    Minimized = not Minimized
end

-- Criar Bolinha para Restaurar
Bubble = Instance.new("TextButton")
Bubble.Size = UDim2.new(0, 40, 0, 40)
Bubble.Position = UDim2.new(0.05, 0, 0.8, 0)
Bubble.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
Bubble.Text = "+"
Bubble.Visible = false
Bubble.Parent = ScreenGui

-- Permitir mover bolinha
Bubble.Active = true
Bubble.Draggable = true
local lastBubblePos = Bubble.Position
Bubble:GetPropertyChangedSignal("Position"):Connect(function()
    lastBubblePos = Bubble.Position
end)

Bubble.MouseButton1Click:Connect(function()
    Bubble.Visible = false
    MainFrame.Visible = true
    Minimized = false
    Bubble.Position = lastBubblePos
end)

-- Botões Minimizar, Maximizar e Fechar
local function createButton(text, position, callback)
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(0, 25, 0, 25)
    Button.Position = position
    Button.Text = text
    Button.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    Button.TextColor3 = Color3.fromRGB(0, 0, 0)
    Button.Font = Enum.Font.GothamBold
    Button.TextSize = 14
    Button.Parent = MainFrame
    Button.MouseButton1Click:Connect(callback)

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = Button
end

createButton("-", UDim2.new(1, -80, 0, 5), toggleMinimize)

createButton("▢", UDim2.new(1, -50, 0, 5), function()
    MainFrame.Size = MainFrame.Size == UDim2.new(0, 280, 0, 180)
        and UDim2.new(1, -20, 1, -20)
        or UDim2.new(0, 280, 0, 180)
end)

createButton("X", UDim2.new(1, -25, 0, 5), function()
    local confirm = Instance.new("TextButton")
    confirm.Size = UDim2.new(0, 150, 0, 80)
    confirm.Position = UDim2.new(0.5, -75, 0.5, -40)
    confirm.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    confirm.TextColor3 = Color3.new(1, 1, 1)
    confirm.Text = "Fechar Script?"
    confirm.Parent = ScreenGui

    confirm.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)
end)

-- Botão Server Hop
local HopButton = Instance.new("TextButton")
HopButton.Size = UDim2.new(0.7, 0, 0, 40)
HopButton.Position = UDim2.new(0.15, 0, 0.4, 0)
HopButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
HopButton.TextColor3 = Color3.new(0, 0, 0)
HopButton.Text = "Server Hop"
HopButton.Font = Enum.Font.GothamBold
HopButton.TextSize = 20
HopButton.Parent = MainFrame

local corner2 = Instance.new("UICorner")
corner2.CornerRadius = UDim.new(0, 8)
corner2.Parent = HopButton

-- TikTok embaixo
local TikTok = Instance.new("TextLabel")
TikTok.Size = UDim2.new(1, 0, 0, 30)
TikTok.Position = UDim2.new(0, 0, 1, -30)
TikTok.BackgroundTransparency = 1
TikTok.Text = "TikTok: LosBrothers.ofc"
TikTok.Font = Enum.Font.Gotham
TikTok.TextSize = 14
TikTok.TextColor3 = Color3.new(1, 1, 1)
TikTok.Parent = MainFrame

-- Função Server Hop
HopButton.MouseButton1Click:Connect(function()
    local servers = HttpService:JSONDecode(
        game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100")
    )
    for _, server in ipairs(servers.data) do
        if server.playing < server.maxPlayers then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, player)
            break
        end
    end
end)

-- Ocultar quando menu do Roblox abre
GuiService.MenuOpened:Connect(function()
    ScreenGui.Enabled = false
end)
GuiService.MenuClosed:Connect(function()
    ScreenGui.Enabled = true
end)
