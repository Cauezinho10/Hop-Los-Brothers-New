-- Hop Los Brothers - Painel com TikTok e Botões
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local MIN_BRAINROT = 1000000 -- valor mínimo para considerar servidor bom

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 280, 0, 180)
Frame.Position = UDim2.new(0.5, -140, 0.5, -90)
Frame.BackgroundColor3 = Color3.fromRGB(47, 120, 200)
Frame.BackgroundTransparency = 0.25 -- 75% transparente
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 14)
Corner.Parent = Frame

-- Botões do topo
local function createTopButton(text, posX)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 60, 0, 22)
    btn.Position = UDim2.new(0, posX, 0, 5)
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

local FullBtn = createTopButton("Tela", 10)
local MinBtn = createTopButton("-", 80)
local CloseBtn = createTopButton("X", 150)

-- Título
local Title = Instance.new("TextLabel")
Title.Text = "Hop Los Brothers"
Title.Size = UDim2.new(1, 0, 0, 25)
Title.Position = UDim2.new(0, 0, 0, 32)
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

-- Função para buscar Brainrot
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
    pcall(function()
        for _, gui in pairs(LocalPlayer.PlayerGui:GetDescendants()) do
            if gui:IsA("TextLabel") or gui:IsA("TextBox") then
                if string.find(string.lower(gui.Text), "brain") then
                    local num = tonumber(gui.Text:gsub("%D", ""))
                    if num then val = num end
                end
            end
        end
    end)
    return val
end

-- Função de Hop
local function HopServer()
    local servers = {}
    local cursor = ""
    repeat
        local success, result = pcall(function()
            return HttpService:JSONDecode(
                game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100" .. (cursor ~= "" and "&cursor=" .. cursor or ""))
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
        TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], LocalPlayer)
    end
end

-- Clique no botão para fazer hop automático
HopButton.MouseButton1Click:Connect(function()
    while task.wait(3) do
        local brainrot = GetBrainrot()
        if brainrot and brainrot >= MIN_BRAINROT then
            warn("Encontrado servidor com " .. brainrot .. " Brainrot! Parando.")
            break
        else
            warn("Brainrot baixo (".. tostring(brainrot) .."), pulando...")
            HopServer()
        end
    end
end)

-- Botões extras
MinBtn.MouseButton1Click:Connect(function()
    Frame.Visible = false
    local MiniButton = Instance.new("TextButton")
    MiniButton.Size = UDim2.new(0, 40, 0, 40)
    MiniButton.Position = UDim2.new(0, 10, 0.9, 0)
    MiniButton.Text = "+"
    MiniButton.BackgroundColor3 = Color3.fromRGB(47, 120, 200)
    MiniButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MiniButton.Font = Enum.Font.GothamBold
    MiniButton.TextScaled = true
    MiniButton.Parent = ScreenGui
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = MiniButton
    MiniButton.MouseButton1Click:Connect(function()
        Frame.Visible = true
        MiniButton:Destroy()
    end)
end)

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

FullBtn.MouseButton1Click:Connect(function()
    Frame.Size = UDim2.new(0, 400, 0, 220)
end)
