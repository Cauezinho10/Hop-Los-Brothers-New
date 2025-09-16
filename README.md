--[[ 
💻 Hop Los Brothers - Versão Final
--]]

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")
local player = Players.LocalPlayer
local placeId = game.PlaceId

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HopLosBrothers"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 280, 0, 160)
MainFrame.Position = UDim2.new(0.35, 0, 0.25, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
MainFrame.BackgroundTransparency = 0.25
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 28)
TitleBar.BackgroundTransparency = 1
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -90, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "Hop Los Brothers"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Botões de janela
local function CreateWindowButton(txt, pos, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 24, 0, 24)
    btn.Position = pos
    btn.Text = txt
    btn.BackgroundColor3 = color
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 18
    btn.Font = Enum.Font.GothamBold
    btn.Parent = TitleBar
    return btn
end

local MaxButton = CreateWindowButton("□", UDim2.new(1, -78, 0, 2), Color3.fromRGB(80, 80, 80))
local MinButton = CreateWindowButton("-", UDim2.new(1, -52, 0, 2), Color3.fromRGB(80, 80, 80))
local CloseButton = CreateWindowButton("X", UDim2.new(1, -26, 0, 2), Color3.fromRGB(200, 0, 0))

-- Texto TikTok
local TikTokText = Instance.new("TextLabel")
TikTokText.Size = UDim2.new(1, 0, 0, 20)
TikTokText.Position = UDim2.new(0, 0, 1, -22)
TikTokText.BackgroundTransparency = 1
TikTokText.Text = "TikTok: LosBrothers.ofc"
TikTokText.TextColor3 = Color3.fromRGB(255, 255, 255)
TikTokText.Font = Enum.Font.GothamSemibold
TikTokText.TextSize = 14
TikTokText.Parent = MainFrame

-- Botão de Server Hop
local ServerHopButton = Instance.new("TextButton")
ServerHopButton.Size = UDim2.new(0.8, 0, 0, 40)
ServerHopButton.Position = UDim2.new(0.1, 0, 0.35, 0)
ServerHopButton.Text = "Server Hop"
ServerHopButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ServerHopButton.BackgroundColor3 = Color3.fromRGB(0, 90, 220)
ServerHopButton.Font = Enum.Font.GothamBold
ServerHopButton.TextSize = 18
ServerHopButton.Parent = MainFrame

local UICornerBtn = Instance.new("UICorner")
UICornerBtn.CornerRadius = UDim.new(0, 8)
UICornerBtn.Parent = ServerHopButton

-- Texto de status
local StatusText = Instance.new("TextLabel")
StatusText.Size = UDim2.new(1, 0, 0, 20)
StatusText.Position = UDim2.new(0, 0, 0.8, 0)
StatusText.BackgroundTransparency = 1
StatusText.TextColor3 = Color3.fromRGB(255, 255, 255)
StatusText.Font = Enum.Font.Gotham
StatusText.TextSize = 16
StatusText.Text = ""
StatusText.Parent = MainFrame

-- Bolinha de carregamento
local LoadingImage = Instance.new("ImageLabel")
LoadingImage.Size = UDim2.new(0, 50, 0, 50)
LoadingImage.Position = UDim2.new(0.5, -25, 0.5, -25)
LoadingImage.BackgroundTransparency = 1
LoadingImage.Image = "rbxassetid://78899293433795"
LoadingImage.Visible = false
LoadingImage.Active = true
LoadingImage.Draggable = true
LoadingImage.Parent = ScreenGui

-- Frame de confirmação ao fechar
local ConfirmFrame = Instance.new("Frame")
ConfirmFrame.Size = UDim2.new(0, 200, 0, 100)
ConfirmFrame.Position = UDim2.new(0.5, -100, 0.5, -50)
ConfirmFrame.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
ConfirmFrame.Visible = false
ConfirmFrame.Parent = ScreenGui

local ConfirmText = Instance.new("TextLabel")
ConfirmText.Size = UDim2.new(1, 0, 0.6, 0)
ConfirmText.BackgroundTransparency = 1
ConfirmText.Text = "Quer fechar o script?"
ConfirmText.TextColor3 = Color3.fromRGB(0, 0, 0)
ConfirmText.Font = Enum.Font.GothamBold
ConfirmText.TextSize = 18
ConfirmText.Parent = ConfirmFrame

local YesButton = Instance.new("TextButton")
YesButton.Size = UDim2.new(0.4, 0, 0.3, 0)
YesButton.Position = UDim2.new(0.05, 0, 0.65, 0)
YesButton.Text = "Sim"
YesButton.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
YesButton.TextColor3 = Color3.fromRGB(255, 255, 255)
YesButton.Parent = ConfirmFrame

local NoButton = Instance.new("TextButton")
NoButton.Size = UDim2.new(0.4, 0, 0.3, 0)
NoButton.Position = UDim2.new(0.55, 0, 0.65, 0)
NoButton.Text = "Não"
NoButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
NoButton.TextColor3 = Color3.fromRGB(255, 255, 255)
NoButton.Parent = ConfirmFrame

-- Função para procurar servidor
local function FindServer()
    StatusText.Text = "🔎 Procurando servidor..."
    LoadingImage.Visible = true
    local servers = {}
    local cursor = ""
    repeat
        local response = game:HttpGet("https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100" .. (cursor ~= "" and "&cursor=" .. cursor or ""))
        local data = HttpService:JSONDecode(response)
        for _, v in pairs(data.data) do
            if v.playing < v.maxPlayers and (v.playerCount and v.playerCount > 1000000) then
                table.insert(servers, v.id)
            end
        end
        cursor = data.nextPageCursor or ""
    until cursor == "" or #servers > 0

    LoadingImage.Visible = false

    if #servers > 0 then
        StatusText.Text = "✅ Servidor encontrado! Entrando..."
        TeleportService:TeleportToPlaceInstance(placeId, servers[1], player)
    else
        StatusText.Text = "❌ Nenhum servidor encontrado."
    end
end

-- Eventos
ServerHopButton.MouseButton1Click:Connect(FindServer)
CloseButton.MouseButton1Click:Connect(function()
    ConfirmFrame.Visible = true
end)
YesButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)
NoButton.MouseButton1Click:Connect(function()
    ConfirmFrame.Visible = false
end)

-- Minimizar / Maximizar
MinButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
    LoadingImage.Visible = false
end)
MaxButton.MouseButton1Click:Connect(function()
    MainFrame.Size = UDim2.new(0, 350, 0, 200)
end)

GuiService.MenuOpened:Connect(function()
    LoadingImage.Visible = false
end)
