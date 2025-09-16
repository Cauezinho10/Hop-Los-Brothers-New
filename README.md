--// Hop Los Brothers (versão final)
--// Feito para ser bonito, rápido e funcional

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

-- Variáveis de UI
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.CoreGui

-- Criar janela principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 320, 0, 200)
mainFrame.Position = UDim2.new(0.5, -160, 0.5, -100)
mainFrame.BackgroundColor3 = Color3.fromRGB(135, 206, 250)
mainFrame.BackgroundTransparency = 0.25
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

mainFrame.ClipsDescendants = false
mainFrame.Name = "HopLosBrothers"

-- Arredondar bordas
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 15)
corner.Parent = mainFrame

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 0, 30)
title.Position = UDim2.new(0, 10, 0, 5)
title.Text = "Hop Los Brothers"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.Parent = mainFrame

-- Botões de fechar, minimizar e tela cheia
local closeBtn = Instance.new("TextButton")
closeBtn.Text = "X"
closeBtn.Size = UDim2.new(0, 25, 0, 25)
closeBtn.Position = UDim2.new(1, -30, 0, 5)
closeBtn.BackgroundTransparency = 1
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 20
closeBtn.Parent = mainFrame

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Text = "-"
minimizeBtn.Size = UDim2.new(0, 25, 0, 25)
minimizeBtn.Position = UDim2.new(1, -60, 0, 5)
minimizeBtn.BackgroundTransparency = 1
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 20
minimizeBtn.Parent = mainFrame

local fullscreenBtn = Instance.new("TextButton")
fullscreenBtn.Text = "▢"
fullscreenBtn.Size = UDim2.new(0, 25, 0, 25)
fullscreenBtn.Position = UDim2.new(1, -90, 0, 5)
fullscreenBtn.BackgroundTransparency = 1
fullscreenBtn.TextColor3 = Color3.new(1, 1, 1)
fullscreenBtn.Font = Enum.Font.GothamBold
fullscreenBtn.TextSize = 20
fullscreenBtn.Parent = mainFrame

-- Texto TikTok
local tiktok = Instance.new("TextLabel")
tiktok.Size = UDim2.new(1, 0, 0, 20)
tiktok.Position = UDim2.new(0, 0, 1, -25)
tiktok.Text = "TikTok: LosBrothers.ofc"
tiktok.TextColor3 = Color3.new(1, 1, 1)
tiktok.BackgroundTransparency = 1
tiktok.Font = Enum.Font.Gotham
tiktok.TextSize = 16
tiktok.Parent = mainFrame

-- Botão Server Hop
local hopButton = Instance.new("TextButton")
hopButton.Size = UDim2.new(0.7, 0, 0, 50)
hopButton.Position = UDim2.new(0.15, 0, 0.35, 0)
hopButton.Text = "Server Hop"
hopButton.Font = Enum.Font.GothamBold
hopButton.TextSize = 22
hopButton.TextColor3 = Color3.new(0, 0, 0)
hopButton.BackgroundColor3 = Color3.new(1, 1, 1)
hopButton.Parent = mainFrame

local hopCorner = Instance.new("UICorner")
hopCorner.CornerRadius = UDim.new(0, 10)
hopCorner.Parent = hopButton

-- Procurando servidor
local statusText = Instance.new("TextLabel")
statusText.Size = UDim2.new(1, 0, 0, 20)
statusText.Position = UDim2.new(0, 0, 1, -50)
statusText.Text = ""
statusText.TextColor3 = Color3.new(1, 1, 1)
statusText.BackgroundTransparency = 1
statusText.Font = Enum.Font.Gotham
statusText.TextSize = 16
statusText.Parent = mainFrame

-- Minimizar vira um quadrado com imagem
local miniIcon = Instance.new("ImageButton")
miniIcon.Visible = false
miniIcon.Size = UDim2.new(0, 50, 0, 50)
miniIcon.Image = "rbxassetid://INSERT_IMAGE_ID" -- Substitua pelo ID da imagem que você quer
miniIcon.BackgroundTransparency = 1
miniIcon.Parent = screenGui
miniIcon.Position = UDim2.new(0.1, 0, 0.8, 0)

-- Funções dos botões
closeBtn.MouseButton1Click:Connect(function()
	local confirm = Instance.new("TextButton")
	confirm.Size = UDim2.new(0, 200, 0, 100)
	confirm.Position = UDim2.new(0.5, -100, 0.5, -50)
	confirm.Text = "Você quer fechar esse script?\nClique para confirmar."
	confirm.Font = Enum.Font.GothamBold
	confirm.TextSize = 18
	confirm.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
	confirm.TextColor3 = Color3.new(1, 1, 1)
	confirm.Parent = screenGui
	confirm.MouseButton1Click:Connect(function()
		screenGui:Destroy()
	end)
end)

minimizeBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = false
	miniIcon.Visible = true
end)

miniIcon.MouseButton1Click:Connect(function()
	mainFrame.Visible = true
	miniIcon.Visible = false
end)

fullscreenBtn.MouseButton1Click:Connect(function()
	if mainFrame.Size ~= UDim2.new(1, 0, 1, 0) then
		mainFrame.Size = UDim2.new(1, 0, 1, 0)
		mainFrame.Position = UDim2.new(0, 0, 0, 0)
	else
		mainFrame.Size = UDim2.new(0, 320, 0, 200)
		mainFrame.Position = UDim2.new(0.5, -160, 0.5, -100)
	end
end)

-- Função de Server Hop
local function serverHop()
	statusText.Text = "🔄 Procurando servidor..."
	local servers = HttpService:JSONDecode(game:HttpGetAsync(
		"https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Desc&limit=100"
	))
	for _,server in ipairs(servers.data) do
		if server.playing < server.maxPlayers and server.ping and server.ping > 1000000 then
			TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, LocalPlayer)
			return
		end
	end
	statusText.Text = "❌ Nenhum servidor encontrado com +1M/s."
end

hopButton.MouseButton1Click:Connect(serverHop)
