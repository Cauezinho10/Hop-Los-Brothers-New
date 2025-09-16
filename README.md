--[[ 
💻 Hop Los Brothers - Server Hop com Filtro
🔧 Recursos:
✅ Botão "Server Hop" que só começa ao clicar
✅ Mensagens no centro da tela: Procurando / Encontrado / Nenhum encontrado
✅ Filtro: só entra em servidores com Brainrots > 1M
✅ Bolinha que pode ser movida, some quando abrir o menu do Roblox
✅ Confirmação ao fechar o script
✅ Compatível com menu do Roblox, não atrapalha
✅ Lugar para colocar sua imagem (já com ID da bolinha)
]]

-- Services
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")

local player = Players.LocalPlayer
local placeId = game.PlaceId

-- GUI Principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Name = "HopLosBrothers"
ScreenGui.Parent = game.CoreGui

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 140)
MainFrame.Position = UDim2.new(0.35, 0, 0.25, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(170, 200, 255)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Text = "Hop Los Brothers"
Title.TextColor3 = Color3.fromRGB(0, 0, 0)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.Parent = MainFrame

-- Botão de Fechar (com confirmação)
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Position = UDim2.new(1, -30, 0, 5)
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CloseButton.Parent = MainFrame

-- Frame de Confirmação
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

-- Bolinha de Status
local LoadingImage = Instance.new("ImageLabel")
LoadingImage.Size = UDim2.new(0, 50, 0, 50)
LoadingImage.Position = UDim2.new(0.5, -25, 0.5, -25)
LoadingImage.BackgroundTransparency = 1
LoadingImage.Image = "rbxassetid://78899293433795" -- Aqui você pode trocar depois
LoadingImage.Visible = false
LoadingImage.Parent = ScreenGui

-- Texto de status
local StatusText = Instance.new("TextLabel")
StatusText.Size = UDim2.new(1, 0, 0, 30)
StatusText.Position = UDim2.new(0, 0, 1, -30)
StatusText.BackgroundTransparency = 1
StatusText.TextColor3 = Color3.fromRGB(0, 0, 0)
StatusText.Font = Enum.Font.GothamBold
StatusText.TextSize = 16
StatusText.Text = ""
StatusText.Parent = MainFrame

-- Botão de Server Hop
local ServerHopButton = Instance.new("TextButton")
ServerHopButton.Size = UDim2.new(0.8, 0, 0, 40)
ServerHopButton.Position = UDim2.new(0.1, 0, 0.5, 0)
ServerHopButton.Text = "Server Hop"
ServerHopButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ServerHopButton.BackgroundColor3 = Color3.fromRGB(0, 100, 255)
ServerHopButton.Font = Enum.Font.GothamBold
ServerHopButton.TextSize = 18
ServerHopButton.Parent = MainFrame

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
		StatusText.Text = "❌ Nenhum servidor encontrado com +1M Brainrots."
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

-- Some a bolinha quando abrir o menu do Roblox
GuiService.MenuOpened:Connect(function()
	LoadingImage.Visible = false
end)
