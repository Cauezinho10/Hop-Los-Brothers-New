-- Hop Los Brothers v3 (Painel Melhorado)
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local MIN_BRAINROT = 1000000 -- valor mínimo para considerar servidor bom

-- GUI principal
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 280, 0, 180)
Frame.Position = UDim2.new(0.5, -140, 0.5, -90)
Frame.BackgroundColor3 = Color3.fromRGB(150, 200, 255)
Frame.BackgroundTransparency = 0.25
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 14)
Corner.Parent = Frame

-- Função para criar botões do topo
local function createTopButton(text, offsetX)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 30, 0, 25)
    btn.Position = UDim2.new(1, offsetX, 0, 5)
    btn.AnchorPoint = Vector2.new(1, 0)
    btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextColor3 = Color3.fromRGB(0, 0, 0)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    btn.Parent = Frame
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 6)
    c.Parent = btn
    return btn
end

local CloseBtn = createTopButton("X", -5)
local MinBtn = createTopButton("-", -40)
local FullBtn = createTopButton("□", -75)

-- Título
local Title = Instance.new("TextLabel")
Title.Text = "Hop Los Brothers"
Title.Size = UDim2.new(1, 0, 0, 25)
Title.Position = UDim2.new(0, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextScaled = true
Title.Parent = Frame

-- Botão principal
local HopButton = Instance.new("TextButton")
HopButton.Size = UDim2.new(0.8, 0, 0, 35)
HopButton.Position = UDim2.new(0.1, 0, 0.45, 0)
HopButton.Text = "Server Hop"
HopButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
HopButton.TextColor3 = Color3.fromRGB(0, 0, 0)
HopButton.Font = Enum.Font.GothamBold
HopButton.TextScaled = true
HopButton.Parent = Frame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 10)
ButtonCorner.Parent = HopButton

-- Texto TikTok
local TikTok = Instance.new("TextLabel")
TikTok.Text = "TikTok: LosBrothers.ofc"
TikTok.Size = UDim2.new(1, 0, 0, 20)
TikTok.Position = UDim2.new(0, 0, 1, -22)
TikTok.BackgroundTransparency = 1
TikTok.TextColor3 = Color3.fromRGB(255, 255, 255)
TikTok.Font = Enum.Font.Gotham
TikTok.TextScaled = true
TikTok.Parent = Frame

-- Função para buscar brainrot
local function GetBrainrot()
    local val = nil
    pcall(function()
        if LocalPlayer:FindFirstChild("leaderstats") then
            for _, v in pairs(LocalPlayer.leaderstats:GetChildren()) do
                if string.find(string.lower(v.Name), "brain") then
                    val = tonumber(v.Value)
                end
            end
        end
    end)
    return val
end

-- Hop rápido
local function HopServer()
    local servers = {}
    local cursor = ""
    repeat
        local success, result = pcall(function()
            return HttpService:JSONDecode(
                game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100"..(cursor ~= "" and "&cursor="..cursor or ""))
            )
        end)
        if success and result and result.data then
            for _, v in pairs(result.data) do
                if v.playing < v.maxPlayers then
                    table.insert(servers, v.id)
                end
            end
            cursor = result.nextPageCursor or ""
        else
            break
        end
    until cursor == ""
    if #servers > 0 then
        TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1,#servers)], LocalPlayer)
    end
end

-- Clique para hop
HopButton.MouseButton1Click:Connect(function()
    while task.wait(1.5) do -- hop mais rápido
        local brainrot = GetBrainrot()
        if brainrot and brainrot >= MIN_BRAINROT then
            warn("Servidor bom encontrado! Parando.")
            break
        else
            HopServer()
        end
    end
end)

-- Minimizar → Bolinha arrastável
MinBtn.MouseButton1Click:Connect(function()
    Frame.Visible = false
    local MiniButton = Instance.new("TextButton")
    MiniButton.Size = UDim2.new(0, 40, 0, 40)
    MiniButton.Position = UDim2.new(0, 10, 0.9, 0)
    MiniButton.Text = "+"
    MiniButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    MiniButton.TextColor3 = Color3.fromRGB(0, 0, 0)
    MiniButton.Font = Enum.Font.GothamBold
    MiniButton.TextScaled = true
    MiniButton.Active = true
    MiniButton.Draggable = true -- agora pode mover a bolinha
    MiniButton.Parent = ScreenGui
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = MiniButton
    MiniButton.MouseButton1Click:Connect(function()
        Frame.Visible = true
        MiniButton:Destroy()
    end)
end)

-- Tela cheia
local full = false
FullBtn.MouseButton1Click:Connect(function()
    if not full then
        Frame.Size = UDim2.new(1, 0, 1, 0)
        Frame.Position = UDim2.new(0, 0, 0, 0)
        full = true
    else
        Frame.Size = UDim2.new(0, 280, 0, 180)
        Frame.Position = UDim2.new(0.5, -140, 0.5, -90)
        full = false
    end
end)

-- Fechar → Confirmação
CloseBtn.MouseButton1Click:Connect(function()
    local Confirm = Instance.new("Frame")
    Confirm.Size = UDim2.new(0, 200, 0, 100)
    Confirm.Position = UDim2.new(0.5, -100, 0.5, -50)
    Confirm.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    Confirm.Parent = ScreenGui
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 10)
    c.Parent = Confirm

    local Text = Instance.new("TextLabel")
    Text.Text = "Quer fechar o script?"
    Text.Size = UDim2.new(1, 0, 0, 40)
    Text.Position = UDim2.new(0, 0, 0, 10)
    Text.BackgroundTransparency = 1
    Text.TextColor3 = Color3.fromRGB(0, 0, 0)
    Text.Font = Enum.Font.GothamBold
    Text.TextScaled = true
    Text.Parent = Confirm

    local Yes = Instance.new("TextButton")
    Yes.Text = "Sim"
    Yes.Size = UDim2.new(0.4, 0, 0, 30)
    Yes.Position = UDim2.new(0.1, 0, 1, -40)
    Yes.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    Yes.TextColor3 = Color3.fromRGB(255, 255, 255)
    Yes.Font = Enum.Font.GothamBold
    Yes.TextScaled = true
    Yes.Parent = Confirm
    local cy = Instance.new("UICorner")
    cy.CornerRadius = UDim.new(0, 6)
    cy.Parent = Yes

    local No = Yes:Clone()
    No.Text = "Não"
    No.Position = UDim2.new(0.55, 0, 1, -40)
    No.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
    No.Parent = Confirm

    Yes.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)
    No.MouseButton1Click:Connect(function()
        Confirm:Destroy()
-- Variável para guardar posição da bolinha
local lastMiniPos = UDim2.new(0, 10, 0.9, 0)

MinBtn.MouseButton1Click:Connect(function()
    Frame.Visible = false
    local MiniButton = Instance.new("TextButton")
    MiniButton.Size = UDim2.new(0, 40, 0, 40)
    MiniButton.Position = lastMiniPos -- reaparece no último lugar salvo
    MiniButton.Text = "+"
    MiniButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    MiniButton.TextColor3 = Color3.fromRGB(0, 0, 0)
    MiniButton.Font = Enum.Font.GothamBold
    MiniButton.TextScaled = true
    MiniButton.Active = true
    MiniButton.Draggable = true
    MiniButton.Parent = ScreenGui

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = MiniButton

    -- Salva posição sempre que o jogador solta a bolinha
    MiniButton.MouseLeave:Connect(function()
        lastMiniPos = MiniButton.Position
    end)

    MiniButton.MouseButton1Click:Connect(function()
        lastMiniPos = MiniButton.Position -- salva última posição antes de abrir
        Frame.Visible = true
        MiniButton:Destroy()
    end)
end)
    end)
end)
