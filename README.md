-- HOP LOS BROTHERS FINAL LIMPO
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HopLosBrothers"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 320, 0, 230)
MainFrame.Position = UDim2.new(0.35, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(135, 206, 250) -- Azul claro
MainFrame.BackgroundTransparency = 0.25 -- 75% transparente
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 15)
UICorner.Parent = MainFrame

-- Título centralizado
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 5)
Title.Text = "Hop Los Brothers"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.BackgroundTransparency = 1
Title.TextScaled = true
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- TikTok embaixo
local TikTok = Instance.new("TextLabel")
TikTok.Size = UDim2.new(1, 0, 0, 20)
TikTok.Position = UDim2.new(0, 0, 1, -25)
TikTok.Text = "TikTok: LosBrothers.ofc"
TikTok.TextColor3 = Color3.fromRGB(255, 255, 255)
TikTok.BackgroundTransparency = 1
TikTok.TextScaled = true
TikTok.Font = Enum.Font.GothamBold
TikTok.Parent = MainFrame

-- Botão Hop Server
local HopButton = Instance.new("TextButton")
HopButton.Size = UDim2.new(0.8, 0, 0, 40)
HopButton.Position = UDim2.new(0.1, 0, 0.5, 0)
HopButton.Text = "Hop Server"
HopButton.TextScaled = true
HopButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
HopButton.TextColor3 = Color3.fromRGB(0, 0, 0)
HopButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 8)
ButtonCorner.Parent = HopButton

-- Minimizar / Maximizar / Fechar
local MinButton = Instance.new("TextButton")
MinButton.Size = UDim2.new(0, 25, 0, 25)
MinButton.Position = UDim2.new(1, -80, 0, 5)
MinButton.Text = "-"
MinButton.TextScaled = true
MinButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
MinButton.TextColor3 = Color3.fromRGB(0, 0, 0)
MinButton.Parent = MainFrame

local MaxButton = Instance.new("TextButton")
MaxButton.Size = UDim2.new(0, 25, 0, 25)
MaxButton.Position = UDim2.new(1, -50, 0, 5)
MaxButton.Text = "⛶"
MaxButton.TextScaled = true
MaxButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
MaxButton.TextColor3 = Color3.fromRGB(0, 0, 0)
MaxButton.Parent = MainFrame

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Position = UDim2.new(1, -25, 0, 5)
CloseButton.Text = "X"
CloseButton.TextScaled = true
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Parent = MainFrame

-- Bolinha
local BallButton = Instance.new("ImageButton")
BallButton.Size = UDim2.new(0, 60, 0, 60)
BallButton.Position = UDim2.new(0, 10, 1, -70)
BallButton.Image = "rbxassetid://COLOQUE_AQUI_O_ID_DA_FOTO"
BallButton.Visible = false
BallButton.BackgroundTransparency = 1
BallButton.Active = true
BallButton.Draggable = true
BallButton.Parent = ScreenGui

-- Minimizar
MinButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    BallButton.Visible = true
end)

-- Maximizar
MaxButton.MouseButton1Click:Connect(function()
    if MainFrame.Size ~= UDim2.new(1, 0, 1, 0) then
        MainFrame.Size = UDim2.new(1, 0, 1, 0)
        MainFrame.Position = UDim2.new(0, 0, 0, 0)
    else
        MainFrame.Size = UDim2.new(0, 320, 0, 230)
        MainFrame.Position = UDim2.new(0.35, 0, 0.3, 0)
    end
end)

-- Confirmar antes de fechar
CloseButton.MouseButton1Click:Connect(function()
    local confirm = Instance.new("Message")
    confirm.Text = "Você quer fechar esse script? (Clique OK para fechar)"
    confirm.Parent = ScreenGui
    task.wait(2)
    confirm:Destroy()
    MainFrame.Visible = false
    BallButton.Visible = false
end)

-- Clicar na bolinha para abrir de novo
BallButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    BallButton.Visible = false
end)

-- Teleporte de servidor
local function HopServer()
    local success, result = pcall(function()
        return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100"))
    end)

    if success and result and result.data then
        for _, server in ipairs(result.data) do
            if server.playing < server.maxPlayers then
                TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, LocalPlayer)
                return
            end
        end
    end
end

HopButton.MouseButton1Click:Connect(HopServer)
