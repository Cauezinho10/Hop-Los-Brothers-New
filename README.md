--[[ 
📜 Hop Los Brothers - Script Unificado
Autor: Cauê
Descrição: Interface personalizável + Server Hop para servidores com +1M/s
]]

-- // Serviços
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- // GUI Principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HopLosBrothers"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent = game:GetService("CoreGui")

-- // Janela Principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 260)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -130)
MainFrame.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
MainFrame.BackgroundTransparency = 0.25
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- // Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "Hop Los Brothers"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextScaled = true
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- // TikTok
local TikTok = Instance.new("TextLabel")
TikTok.Size = UDim2.new(1, 0, 0, 20)
TikTok.Position = UDim2.new(0, 0, 1, -20)
TikTok.BackgroundTransparency = 1
TikTok.Text = "TikTok: LosBrothers.ofc"
TikTok.TextColor3 = Color3.fromRGB(255, 255, 255)
TikTok.TextScaled = true
TikTok.Font = Enum.Font.Gotham
TikTok.Parent = MainFrame

-- // Botões de Controle
local function criarBotaoControle(txt, xOffset)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 30, 0, 30)
    btn.Position = UDim2.new(1, -xOffset, 0, 5)
    btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    btn.Text = txt
    btn.TextColor3 = Color3.fromRGB(0, 0, 0)
    btn.TextScaled = true
    btn.Font = Enum.Font.GothamBold
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = btn
    btn.Parent = MainFrame
    return btn
end

local CloseBtn = criarBotaoControle("X", 35)
local MinimizeBtn = criarBotaoControle("-", 70)
local FullscreenBtn = criarBotaoControle("⛶", 105)

-- // Minimizado
local MinimizedButton = Instance.new("ImageButton")
MinimizedButton.Visible = false
MinimizedButton.Size = UDim2.new(0, 60, 0, 60)
MinimizedButton.BackgroundTransparency = 1
MinimizedButton.Image = "rbxassetid://" -- Coloque aqui o ID da imagem que você mandou
MinimizedButton.Parent = ScreenGui
MinimizedButton.Position = UDim2.new(0.5, -30, 0.8, 0)

-- // Confirmação de Fechar
local ConfirmFrame = Instance.new("Frame")
ConfirmFrame.Size = UDim2.new(0, 250, 0, 120)
ConfirmFrame.Position = UDim2.new(0.5, -125, 0.5, -60)
ConfirmFrame.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
ConfirmFrame.BackgroundTransparency = 0.25
ConfirmFrame.Visible = false
ConfirmFrame.Parent = ScreenGui
local cornerConfirm = Instance.new("UICorner")
cornerConfirm.CornerRadius = UDim.new(0, 12)
cornerConfirm.Parent = ConfirmFrame

local ConfirmText = Instance.new("TextLabel")
ConfirmText.Size = UDim2.new(1, 0, 0, 50)
ConfirmText.BackgroundTransparency = 1
ConfirmText.Text = "Você quer fechar esse script?"
ConfirmText.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmText.TextScaled = true
ConfirmText.Font = Enum.Font.GothamBold
ConfirmText.Parent = ConfirmFrame

local YesBtn = Instance.new("TextButton")
YesBtn.Size = UDim2.new(0.4, 0, 0, 40)
YesBtn.Position = UDim2.new(0.05, 0, 0.65, 0)
YesBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
YesBtn.Text = "Sim"
YesBtn.TextScaled = true
YesBtn.Font = Enum.Font.GothamBold
YesBtn.Parent = ConfirmFrame
local cornerYes = Instance.new("UICorner")
cornerYes.CornerRadius = UDim.new(0, 10)
cornerYes.Parent = YesBtn

local NoBtn = YesBtn:Clone()
NoBtn.Text = "Não"
NoBtn.Position = UDim2.new(0.55, 0, 0.65, 0)
NoBtn.Parent = ConfirmFrame

-- // Evento Botões
CloseBtn.MouseButton1Click:Connect(function()
    ConfirmFrame.Visible = true
end)

YesBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

NoBtn.MouseButton1Click:Connect(function()
    ConfirmFrame.Visible = false
end)

MinimizeBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    MinimizedButton.Visible = true
end)

MinimizedButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    MinimizedButton.Visible = false
end)

FullscreenBtn.MouseButton1Click:Connect(function()
    if MainFrame.Size ~= UDim2.new(1, 0, 1, 0) then
        MainFrame.Size = UDim2.new(1, 0, 1, 0)
        MainFrame.Position = UDim2.new(0, 0, 0, 0)
    else
        MainFrame.Size = UDim2.new(0, 400, 0, 260)
        MainFrame.Position = UDim2.new(0.5, -200, 0.5, -130)
    end
end)

-- // Server Hop
local ProcurandoLabel = Instance.new("TextLabel")
ProcurandoLabel.Size = UDim2.new(1, 0, 0, 20)
ProcurandoLabel.Position = UDim2.new(0, 0, 1, -40)
ProcurandoLabel.BackgroundTransparency = 1
ProcurandoLabel.Text = ""
ProcurandoLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
ProcurandoLabel.TextScaled = true
ProcurandoLabel.Font = Enum.Font.GothamBold
ProcurandoLabel.Parent = MainFrame

local function hopServer()
    ProcurandoLabel.Text = "Procurando servidor..."
    task.spawn(function()
        local servers = HttpService:JSONDecode(game:HttpGet(
            "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        )).data

        for _, server in ipairs(servers) do
            -- Aqui você precisaria de um jeito de filtrar servidores > 1M/s (se o jogo expõe essa info)
            if server.playing < server.maxPlayers and server.id ~= game.JobId then
                ProcurandoLabel.Text = "🔗 Servidor encontrado! Teleportando..."
                task.wait(2)
                TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, LocalPlayer)
                return
            end
        end
        ProcurandoLabel.Text = "Nenhum servidor encontrado!"
    end)
end

hopServer()
