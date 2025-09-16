-- Hop Los Brothers - Painel Transparente com Personagens
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local MIN_BRAINROT = 1000000 -- valor mínimo para considerar servidor bom

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 280, 0, 160)
Frame.Position = UDim2.new(0.5, -140, 0.5, -80)
Frame.BackgroundColor3 = Color3.fromRGB(47, 120, 200) -- azul do personagem
Frame.BackgroundTransparency = 0.25 -- 75% transparente
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 14)
Corner.Parent = Frame

-- Ícones dos personagens
local LeftIcon = Instance.new("ImageLabel")
LeftIcon.Size = UDim2.new(0, 40, 0, 40)
LeftIcon.Position = UDim2.new(0.05, 0, -0.25, 0)
LeftIcon.Image = "rbxassetid://123456789" -- Substitua pelo ID do personagem azul
LeftIcon.BackgroundTransparency = 1
LeftIcon.Parent = Frame

local RightIcon = Instance.new("ImageLabel")
RightIcon.Size = UDim2.new(0, 40, 0, 40)
RightIcon.Position = UDim2.new(0.8, 0, -0.25, 0)
RightIcon.Image = "rbxassetid://987654321" -- Substitua pelo ID do biscoito
RightIcon.BackgroundTransparency = 1
RightIcon.Parent = Frame

-- Título
local Title = Instance.new("TextLabel")
Title.Text = "LOS BOTHERS"
Title.Size = UDim2.new(1, 0, 0, 25)
Title.Position = UDim2.new(0, 0, 0, 10)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextScaled = true
Title.Parent = Frame

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

-- Botão de Server Hop
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
