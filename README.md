-- // Variáveis e Serviços
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Player = Players.LocalPlayer

-- // Criando a GUI principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 130)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -65)
MainFrame.BackgroundColor3 = Color3.fromRGB(135, 206, 250) -- Azul claro
MainFrame.BackgroundTransparency = 0.25
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

-- // Título
local Title = Instance.new("TextLabel")
Title.Text = "Hop Los Brothers"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(1, -60, 0, 30)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = MainFrame

-- // Botões
local CloseBtn = Instance.new("TextButton")
CloseBtn.Text = "X"
CloseBtn.Size = UDim2.new(0, 20, 0, 20)
CloseBtn.Position = UDim2.new(1, -25, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
CloseBtn.Parent = MainFrame

local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Text = "-"
MinimizeBtn.Size = UDim2.new(0, 20, 0, 20)
MinimizeBtn.Position = UDim2.new(1, -50, 0, 5)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
MinimizeBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
MinimizeBtn.Parent = MainFrame

local FullscreenBtn = Instance.new("TextButton")
FullscreenBtn.Text = "□"
FullscreenBtn.Size = UDim2.new(0, 20, 0, 20)
FullscreenBtn.Position = UDim2.new(1, -75, 0, 5)
FullscreenBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
FullscreenBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
FullscreenBtn.Parent = MainFrame

-- // Server Hop Button
local HopButton = Instance.new("TextButton")
HopButton.Text = "Server Hop"
HopButton.Font = Enum.Font.GothamBold
HopButton.TextSize = 20
HopButton.Size = UDim2.new(0.8, 0, 0, 40)
HopButton.Position = UDim2.new(0.1, 0, 0.4, 0)
HopButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
HopButton.TextColor3 = Color3.fromRGB(0, 0, 0)
HopButton.Parent = MainFrame

local TikTok = Instance.new("TextLabel")
TikTok.Text = "TikTok: LosBrothers.ofc"
TikTok.Font = Enum.Font.Gotham
TikTok.TextSize = 14
TikTok.TextColor3 = Color3.fromRGB(255, 255, 255)
TikTok.BackgroundTransparency = 1
TikTok.Size = UDim2.new(1, 0, 0, 20)
TikTok.Position = UDim2.new(0, 0, 1, -25)
TikTok.Parent = MainFrame

-- // Loading Label (oculto até usar)
local LoadingLabel = Instance.new("TextLabel")
LoadingLabel.Text = ""
LoadingLabel.Font = Enum.Font.GothamBold
LoadingLabel.TextSize = 16
LoadingLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
LoadingLabel.BackgroundTransparency = 1
LoadingLabel.Size = UDim2.new(1, 0, 0, 20)
LoadingLabel.Position = UDim2.new(0, 0, 0.85, 0)
LoadingLabel.Parent = MainFrame

-- // Funções
local savedPos = MainFrame.Position
local isFullscreen = false

MinimizeBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    local Ball = Instance.new("TextButton")
    Ball.Text = "◎"
    Ball.Size = UDim2.new(0, 30, 0, 30)
    Ball.Position = savedPos
    Ball.BackgroundTransparency = 1
    Ball.TextColor3 = Color3.fromRGB(255,255,255)
    Ball.TextScaled = true
    Ball.Parent = ScreenGui
    Ball.Active = true
    Ball.Draggable = true

    Ball.MouseButton1Click:Connect(function()
        savedPos = Ball.Position
        Ball:Destroy()
        MainFrame.Visible = true
        MainFrame.Position = savedPos
    end)
end)

FullscreenBtn.MouseButton1Click:Connect(function()
    if not isFullscreen then
        savedPos = MainFrame.Position
        MainFrame.Size = UDim2.new(1, 0, 1, 0)
        MainFrame.Position = UDim2.new(0, 0, 0, 0)
        isFullscreen = true
    else
        MainFrame.Size = UDim2.new(0, 250, 0, 130)
        MainFrame.Position = savedPos
        isFullscreen = false
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    local ConfirmFrame = Instance.new("Frame")
    ConfirmFrame.Size = UDim2.new(0, 200, 0, 100)
    ConfirmFrame.Position = UDim2.new(0.5, -100, 0.5, -50)
    ConfirmFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    ConfirmFrame.Parent = ScreenGui
    ConfirmFrame.Active = true

    local ConfirmText = Instance.new("TextLabel")
    ConfirmText.Text = "Você quer fechar esse script?"
    ConfirmText.Size = UDim2.new(1, 0, 0.6, 0)
    ConfirmText.BackgroundTransparency = 1
    ConfirmText.TextColor3 = Color3.fromRGB(255, 255, 255)
    ConfirmText.Parent = ConfirmFrame

    local YesBtn = Instance.new("TextButton")
    YesBtn.Text = "Sim"
    YesBtn.Size = UDim2.new(0.5, -5, 0.4, 0)
    YesBtn.Position = UDim2.new(0, 0, 0.6, 0)
    YesBtn.Parent = ConfirmFrame

    local NoBtn = Instance.new("TextButton")
    NoBtn.Text = "Não"
    NoBtn.Size = UDim2.new(0.5, -5, 0.4, 0)
    NoBtn.Position = UDim2.new(0.5, 5, 0.6, 0)
    NoBtn.Parent = ConfirmFrame

    YesBtn.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)
    NoBtn.MouseButton1Click:Connect(function()
        ConfirmFrame:Destroy()
    end)
end)

-- // Função do Server Hop com carregamento
HopButton.MouseButton1Click:Connect(function()
    task.spawn(function()
        LoadingLabel.Text = "🔄 Procurando servidor..."
        local cursor = ""
        local found = false
        while not found do
            local url = "https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Desc&limit=10"..(cursor ~= "" and "&cursor="..cursor or "")
            local success, result = pcall(function()
                return HttpService:JSONDecode(game:HttpGet(url))
            end)
            if success and result and result.data then
                for _, server in ipairs(result.data) do
                    if server.playing >= 1000000 and server.playing < server.maxPlayers then
                        found = true
                        LoadingLabel.Text = "✅ Servidor encontrado!"
                        task.wait(0.5)
                        TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, Player)
                        break
                    end
                end
                cursor = result.nextPageCursor or ""
                if cursor == "" then break end
            else
                break
            end
            task.wait(0.1)
        end
        if not found then
            LoadingLabel.Text = "❌ Nenhum servidor encontrado"
            task.wait(1.5)
            LoadingLabel.Text = ""
        end
    end)
end)
