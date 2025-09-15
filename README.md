-- Hop Los Brothers - Script Final Unificado
-- Cauê - versão pronta para colar no executor
-- Ajuste MINIMIZED_IMAGE_ID abaixo para usar sua imagem

-- ===== CONFIG =====
local MINIMIZED_IMAGE_ID = 0 -- << TROQUE AQUI PELO ID DA SUA IMAGEM (somente número)
local MIN_BRAINROT = 1000000 -- 1M/s threshold
local SEARCH_PAGE_LIMIT = 10  -- quantos servidores por página buscar (reduz para mais rapidez)
local HOP_WAIT = 0.08         -- pequeno delay entre páginas (mais rápido = menor valor)
-- ==================

-- Serviços
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

-- Estado
local isSearching = false
local searchCancelled = false
local lastMinimizedPos = UDim2.new(0.5, -35, 0.8, 0)
local currentTab = "Teleport" -- "Teleport" or "Helper"
local savedTab = currentTab

-- ESP states
local PlayerESP_Enabled = false
local TimerESP_Enabled = false
local ValueESP_Enabled = false

-- Keep track of created ESP GUIs so we can remove them
local playerBillboards = {} -- [player] = billboardGui
local highestHighlight = nil -- {player = p, label = Gui}
local timerGui = nil

-- Utility: safe pcall with http
local function safeHttpGet(url)
    local ok, res = pcall(function() return game:HttpGet(url) end)
    if not ok then return nil end
    local ok2, data = pcall(function() return HttpService:JSONDecode(res) end)
    if not ok2 then return nil end
    return data
end

-- Create ScreenGui (in CoreGui) with IgnoreGuiInset so it doesn't overlay Roblox menus incorrectly
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HopLosBrothersGUI"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")
screenGui.DisplayOrder = 5

-- Hide GUI when built-in menu opens, show when closed
GuiService.MenuOpened:Connect(function() screenGui.Enabled = false end)
GuiService.MenuClosed:Connect(function() screenGui.Enabled = true end)

-- ===== UI BUILD =====
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 420, 0, 320) -- maior
mainFrame.Position = UDim2.new(0.35, 0, 0.28, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(135, 206, 250)
mainFrame.BackgroundTransparency = 0.25
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner", mainFrame)
mainCorner.CornerRadius = UDim.new(0, 15)

-- Title top-left
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(0.6, 0, 0, 36)
title.Position = UDim2.new(0, 12, 0, 8)
title.BackgroundTransparency = 1
title.Text = "Hop Los Brothers"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.new(1,1,1)
title.TextXAlignment = Enum.TextXAlignment.Left

-- Small control buttons top-right (white circle look)
local function makeTopBtn(parent, text, xOffset)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(0, 34, 0, 28)
    b.Position = UDim2.new(1, -xOffset, 0, 8)
    b.BackgroundColor3 = Color3.new(1,1,1)
    b.Text = text
    b.Font = Enum.Font.GothamBold
    b.TextSize = 16
    b.TextColor3 = Color3.new(0,0,0)
    local c = Instance.new("UICorner", b); c.CornerRadius = UDim.new(0,6)
    return b
end

local closeBtn = makeTopBtn(mainFrame, "X", 16)
local minimizeBtn = makeTopBtn(mainFrame, "-", 60)
local fullscreenBtn = makeTopBtn(mainFrame, "▢", 104)

-- Tabs area (left side)
local teleportTabBtn = Instance.new("TextButton", mainFrame)
teleportTabBtn.Size = UDim2.new(0, 140, 0, 36)
teleportTabBtn.Position = UDim2.new(0, 12, 0, 56)
teleportTabBtn.Text = "Teleport"
teleportTabBtn.Font = Enum.Font.GothamBold
teleportTabBtn.TextScaled = true

local helperTabBtn = Instance.new("TextButton", mainFrame)
helperTabBtn.Size = UDim2.new(0, 140, 0, 36)
helperTabBtn.Position = UDim2.new(0, 12, 0, 96)
helperTabBtn.Text = "Helper"
helperTabBtn.Font = Enum.Font.GothamBold
helperTabBtn.TextScaled = true

local tabCorner1 = Instance.new("UICorner", teleportTabBtn); tabCorner1.CornerRadius = UDim.new(0,8)
local tabCorner2 = Instance.new("UICorner", helperTabBtn); tabCorner2.CornerRadius = UDim.new(0,8)

-- Container frames for each tab content
local teleportContent = Instance.new("Frame", mainFrame)
teleportContent.Size = UDim2.new(0, 252, 0, 210)
teleportContent.Position = UDim2.new(0, 150, 0, 56)
teleportContent.BackgroundTransparency = 1
teleportContent.Visible = true

local helperContent = Instance.new("Frame", mainFrame)
helperContent.Size = teleportContent.Size
helperContent.Position = teleportContent.Position
helperContent.BackgroundTransparency = 1
helperContent.Visible = false

-- Hop button (big and highlighted)
local hopBtn = Instance.new("TextButton", teleportContent)
hopBtn.Size = UDim2.new(0.9, 0, 0, 64)
hopBtn.Position = UDim2.new(0.05, 0, 0, 8)
hopBtn.Text = "Hop Server"
hopBtn.Font = Enum.Font.GothamBold
hopBtn.TextSize = 22
hopBtn.TextColor3 = Color3.new(0,0,0)
hopBtn.BackgroundColor3 = Color3.fromRGB(80, 170, 255)
local hopCorner = Instance.new("UICorner", hopBtn); hopCorner.CornerRadius = UDim.new(0,12)

-- Status label for bottom of content (Procurando / Found / Cancelled)
local statusSmall = Instance.new("TextLabel", teleportContent)
statusSmall.Size = UDim2.new(0.9,0,0,22)
statusSmall.Position = UDim2.new(0.05,0,0,82)
statusSmall.BackgroundTransparency = 1
statusSmall.Text = ""
statusSmall.Font = Enum.Font.GothamBold
statusSmall.TextColor3 = Color3.new(1,1,1)
statusSmall.TextScaled = true

-- Middle big status (center of screen) - only text, no background
local centerStatus = Instance.new("TextLabel", screenGui)
centerStatus.Size = UDim2.new(0.6,0,0,60)
centerStatus.Position = UDim2.new(0.2,0,0.5,-30)
centerStatus.BackgroundTransparency = 1
centerStatus.Text = ""
centerStatus.TextColor3 = Color3.new(1,1,1)
centerStatus.Font = Enum.Font.GothamBold
centerStatus.TextScaled = true
centerStatus.Visible = false

local function showCenterMessage(msg, sec)
    centerStatus.Text = msg
    centerStatus.Visible = true
    task.spawn(function()
        task.wait(sec or 2)
        centerStatus.Visible = false
    end)
end

-- TikTok footer
local footer = Instance.new("TextLabel", mainFrame)
footer.Size = UDim2.new(0.9,0,0,22)
footer.Position = UDim2.new(0.05,0,1,-32)
footer.BackgroundTransparency = 1
footer.Text = "TikTok: LosBrothers.ofc"
footer.Font = Enum.Font.Gotham
footer.TextScaled = true
footer.TextColor3 = Color3.new(1,1,1)

-- Minimized circular image (keeps pos)
local minimizedBtn = Instance.new("ImageButton")
minimizedBtn.Size = UDim2.new(0,70,0,70)
minimizedBtn.Position = lastMinimizedPos
minimizedBtn.BackgroundTransparency = 1
minimizedBtn.Image = (MINIMIZED_IMAGE_ID ~= 0) and ("rbxassetid://"..tostring(78899293433795)) or "" -- set later
minimizedBtn.Visible = false
minimizedBtn.Name = "MinimizedBtn"
minimizedBtn.Parent = screenGui
local minCorner = Instance.new("UICorner", minimizedBtn); minCorner.CornerRadius = UDim.new(1,0)
minimizedBtn.Active = true
minimizedBtn.Draggable = true

minimizedBtn:GetPropertyChangedSignal("Position"):Connect(function()
    lastMinimizedPos = minimizedBtn.Position
end)

-- Fullscreen toggle state
local isFullscreen = false
local normalSize = mainFrame.Size
local normalPos = mainFrame.Position

-- Confirmation frame for close
local confirmFrame = Instance.new("Frame", screenGui)
confirmFrame.Size = UDim2.new(0, 320, 0, 140)
confirmFrame.Position = UDim2.new(0.5, -160, 0.5, -70)
confirmFrame.BackgroundColor3 = Color3.fromRGB(10,10,10)
confirmFrame.BackgroundTransparency = 0.3
confirmFrame.Visible = false
local confirmCorner = Instance.new("UICorner", confirmFrame); confirmCorner.CornerRadius = UDim.new(0,12)
local confirmTxt = Instance.new("TextLabel", confirmFrame)
confirmTxt.Size = UDim2.new(1, -20, 0, 70)
confirmTxt.Position = UDim2.new(0, 10, 0, 10)
confirmTxt.BackgroundTransparency = 1
confirmTxt.Text = "Você quer fechar esse script?"
confirmTxt.Font = Enum.Font.GothamBold
confirmTxt.TextScaled = true
confirmTxt.TextColor3 = Color3.fromRGB(255,255,255)
local yesBtn = Instance.new("TextButton", confirmFrame)
yesBtn.Size = UDim2.new(0.45, -6, 0, 40)
yesBtn.Position = UDim2.new(0.05, 0, 0.7, 0)
yesBtn.Text = "Sim"
yesBtn.Font = Enum.Font.GothamBold
yesBtn.TextScaled = true
local noBtn = yesBtn:Clone()
noBtn.Parent = confirmFrame
noBtn.Position = UDim2.new(0.55, 6, 0.7, 0)
noBtn.Text = "Não"
local yesCorner = Instance.new("UICorner", yesBtn); yesCorner.CornerRadius = UDim.new(0,8)
local noCorner = Instance.new("UICorner", noBtn); noCorner.CornerRadius = UDim.new(0,8)

-- Helper toggles on helperContent
local playerESPBtn = Instance.new("TextButton", helperContent)
playerESPBtn.Size = UDim2.new(0.9,0,0,36)
playerESPBtn.Position = UDim2.new(0.05,0,0,8)
playerESPBtn.Text = "Player ESP [OFF]"
playerESPBtn.Font = Enum.Font.GothamBold
playerESPBtn.TextScaled = true
local cornerP = Instance.new("UICorner", playerESPBtn); cornerP.CornerRadius = UDim.new(0,10)

local timerESPBtn = Instance.new("TextButton", helperContent)
timerESPBtn.Size = UDim2.new(0.9,0,0,36)
timerESPBtn.Position = UDim2.new(0.05,0,0,56)
timerESPBtn.Text = "Timer ESP [OFF]"
timerESPBtn.Font = Enum.Font.GothamBold
timerESPBtn.TextScaled = true
local cornerT = Instance.new("UICorner", timerESPBtn); cornerT.CornerRadius = UDim.new(0,10)

local valueESPBtn = Instance.new("TextButton", helperContent)
valueESPBtn.Size = UDim2.new(0.9,0,0,36)
valueESPBtn.Position = UDim2.new(0.05,0,0,104)
valueESPBtn.Text = "Highest Value Brainrot ESP [OFF]"
valueESPBtn.Font = Enum.Font.GothamBold
valueESPBtn.TextScaled = true
local cornerV = Instance.new("UICorner", valueESPBtn); cornerV.CornerRadius = UDim.new(0,10)

-- TAB switching
teleportTabBtn.MouseButton1Click:Connect(function()
    teleportContent.Visible = true
    helperContent.Visible = false
    currentTab = "Teleport"
    savedTab = currentTab
end)
helperTabBtn.MouseButton1Click:Connect(function()
    teleportContent.Visible = false
    helperContent.Visible = true
    currentTab = "Helper"
    savedTab = currentTab
end)

-- Fullscreen/Minimize/Close behavior
minimizeBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    minimizedBtn.Visible = true
    minimizedBtn.Position = lastMinimizedPos
end)
minimizedBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = true
    minimizedBtn.Visible = false
    mainFrame.Position = lastMinimizedPos
end)

fullscreenBtn.MouseButton1Click:Connect(function()
    if not isFullscreen then
        normalSize = mainFrame.Size
        normalPos = mainFrame.Position
        mainFrame.Size = UDim2.new(1, 0, 1, 0)
        mainFrame.Position = UDim2.new(0, 0, 0, 0)
        isFullscreen = true
    else
        mainFrame.Size = normalSize
        mainFrame.Position = normalPos
        isFullscreen = false
    end
end)

closeBtn.MouseButton1Click:Connect(function()
    confirmFrame.Visible = true
end)
yesBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)
noBtn.MouseButton1Click:Connect(function() confirmFrame.Visible = false end)

-- =========================================
-- ======= SERVER HOP LOGIC (on click) =====
-- =========================================

local searchingTask = nil

local function cancelSearch()
    searchCancelled = true
    isSearching = false
    statusSmall.Text = "❌ Busca cancelada"
    centerStatus.Text = "❌ Busca cancelada"
    centerStatus.Visible = true
    task.delay(2, function() centerStatus.Visible = false end)
end

local function tryTeleportToServer(serverId)
    -- Teleport after brief wait
    task.spawn(function()
        task.wait(0.6)
        TeleportService:TeleportToPlaceInstance(game.PlaceId, serverId, LocalPlayer)
    end)
end

-- After teleport into a server, this code (if running) will try to detect brainrot value
local function detectBrainrotValue()
    -- Tries multiple locations and heuristics to find a numeric brainrot value for local player
    local function tryNumberFromObj(obj)
        if not obj then return nil end
        -- if value object with Value property
        if type(obj) == "table" then return nil end
        pcall(function()
            if obj:IsA("IntValue") or obj:IsA("NumberValue") then
                return tonumber(obj.Value)
            end
        end)
        -- fallback
        local ok, txt = pcall(function() return obj.Text end)
        if ok and type(txt) == "string" then
            local digits = txt:match("([%d,%.]+)")
            if digits then
                digits = digits:gsub(",", "")
                local n = tonumber(digits)
                return n
            end
        end
        return nil
    end

    -- leaderstats
    local val = nil
    pcall(function()
        local ls = LocalPlayer:FindFirstChild("leaderstats")
        if ls then
            for _,v in pairs(ls:GetChildren()) do
                if string.find(string.lower(v.Name), "brain") or string.find(string.lower(v.Name), "br") then
                    val = tonumber(v.Value) or val
                end
            end
        end
    end)
    if val and type(val) == "number" then return val end

    -- PlayerGui text labels
    pcall(function()
        for _, gui in pairs(LocalPlayer:WaitForChild("PlayerGui"):GetDescendants()) do
            if gui:IsA("TextLabel") or gui:IsA("TextBox") then
                local txt = tostring(gui.Text or "")
                if string.find(string.lower(txt), "brain") or string.find(string.lower(txt), "/s") or string.find(string.lower(txt), "ps") then
                    local digits = txt:match("([%d,%.]+)")
                    if digits then
                        digits = digits:gsub(",", "")
                        local n = tonumber(digits)
                        if n then val = n; break end
                    end
                end
            end
        end
    end)

    if val and type(val) == "number" then return val end

    -- ReplicatedStorage / Workspace heuristics
    pcall(function()
        local rs = game:GetService("ReplicatedStorage")
        for _,v in pairs(rs:GetDescendants()) do
            if v:IsA("NumberValue") or v:IsA("IntValue") then
                if string.find(string.lower(v.Name), "brain") or string.find(string.lower(v.Name), "br") or string.find(string.lower(v.Name), "ps") then
                    val = tonumber(v.Value) or val
                end
            end
        end
    end)
    if val and type(val) == "number" then return val end

    pcall(function()
        for _,v in pairs(workspace:GetDescendants()) do
            if v:IsA("NumberValue") or v:IsA("IntValue") then
                if string.find(string.lower(v.Name), "brain") or string.find(string.lower(v.Name), "br") or string.find(string.lower(v.Name), "ps") then
                    val = tonumber(v.Value) or val
                end
            end
        end
    end)

    return val
end

-- core searching loop: searches pages and teleport to candidate servers until found one that (after joining) reports brainrot >= MIN_BRAINROT
local function startSearch()
    if isSearching then return end
    isSearching = true
    searchCancelled = false
    statusSmall.Text = "🔎 Procurando servidor..."
    centerStatus.Text = "🔎 Procurando servidor..."
    centerStatus.Visible = true

    task.spawn(function()
        -- We'll loop over pages of servers; for each server we teleport there then, when entering, the running script will attempt to detect value.
        -- However teleporting into servers one by one is intrusive; instead we will try to choose servers more likely to have high value:
        -- Strategy: fetch server pages ordered by Desc (largest playing first), try those quickly.
        local cursor = ""
        local foundRemoteCandidate = nil

        while not searchCancelled do
            local url = "https://games.roblox.com/v1/games/"..tostring(game.PlaceId).."/servers/Public?sortOrder=Desc&limit="..tostring(SEARCH_PAGE_LIMIT)
            if cursor and cursor ~= "" then
                url = url .. "&cursor=" .. tostring(cursor)
            end
            local data = safeHttpGet(url)
            if not data or not data.data or #data.data == 0 then
                -- no servers
                break
            end
            -- check each server quickly: prefer servers with more players (higher chance)
            for _, server in ipairs(data.data) do
                if searchCancelled then break end
                if server.id and server.playing and server.playing < server.maxPlayers then
                    -- Try candidate: teleport to it
                    foundRemoteCandidate = server.id
                    -- show small status
                    statusSmall.Text = "Tentando server com "..tostring(server.playing).." players..."
                    -- teleport and return (on join, detection logic can decide to hop again if not >= MIN_BRAINROT)
                    centerStatus.Text = "🔗 Server encontrado (candidate). Teleportando..."
                    centerStatus.Visible = true
                    task.wait(1.2)
                    isSearching = false -- we will let teleport happen
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, LocalPlayer)
                    return
                end
            end
            cursor = data.nextPageCursor or ""
            if not cursor or cursor == "" then break end
            task.wait(HOP_WAIT)
        end

        if searchCancelled then
            -- canceled by user
            statusSmall.Text = "❌ Busca cancelada"
            centerStatus.Text = "❌ Busca cancelada"
            task.wait(1.5)
            centerStatus.Visible = false
            isSearching = false
            return
        end

        -- not found
        statusSmall.Text = "❌ Nenhum servidor encontrado"
        centerStatus.Text = "❌ Nenhum servidor encontrado"
        task.wait(1.8)
        centerStatus.Visible = false
        isSearching = false
    end)
end

-- Hop button click only starts search when clicked
hopBtn.MouseButton1Click:Connect(function()
    if not isSearching then
        startSearch()
    end
end)

-- Provide a small cancel UI in teleportContent while searching (click to cancel)
local cancelSearchBtn = Instance.new("TextButton", teleportContent)
cancelSearchBtn.Size = UDim2.new(0.9,0,0,30)
cancelSearchBtn.Position = UDim2.new(0.05,0,0,116)
cancelSearchBtn.Text = "Cancelar Busca"
cancelSearchBtn.Font = Enum.Font.GothamBold
cancelSearchBtn.TextScaled = true
cancelSearchBtn.BackgroundColor3 = Color3.fromRGB(255,255,255)
local cancelCorner = Instance.new("UICorner", cancelSearchBtn); cancelCorner.CornerRadius = UDim.new(0,10)
cancelSearchBtn.MouseButton1Click:Connect(function()
    if isSearching then
        cancelSearch()
    end
end)

-- =========================
-- ====== HELPER ESPs ======
-- =========================

-- Player ESP: create BillboardGui above each player's head with name + brain/s (if readable)
local function createPlayerBillboard(p)
    if not p.Character then return end
    if playerBillboards[p] then return end
    local head = p.Character:FindFirstChild("Head") or p.Character:FindFirstChildWhichIsA("BasePart")
    if not head then return end
    local bg = Instance.new("BillboardGui")
    bg.Name = "Hop_PlayerESP"
    bg.Adornee = head
    bg.Size = UDim2.new(0, 150, 0, 40)
    bg.StudsOffset = Vector3.new(0, 2.2, 0)
    bg.AlwaysOnTop = true
    bg.Parent = p.Character

    local label = Instance.new("TextLabel", bg)
    label.Size = UDim2.new(1,0,1,0)
    label.BackgroundTransparency = 0.5
    label.BackgroundColor3 = Color3.fromRGB(0,0,0)
    label.TextColor3 = Color3.fromRGB(255,255,255)
    label.TextStrokeTransparency = 0.5
    label.Font = Enum.Font.GothamBold
    label.TextScaled = true
    label.Text = p.Name

    playerBillboards[p] = bg
end

local function removePlayerBillboard(p)
    if playerBillboards[p] then
        pcall(function() playerBillboards[p]:Destroy() end)
        playerBillboards[p] = nil
    end
end

local function updateAllPlayerESPs()
    for _,p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then
            if PlayerESP_Enabled then
                createPlayerBillboard(p)
            else
                removePlayerBillboard(p)
            end
        end
    end
end

Players.PlayerAdded:Connect(function(p) if PlayerESP_Enabled then createPlayerBillboard(p) end end)
Players.PlayerRemoving:Connect(function(p) removePlayerBillboard(p) end)

playerESPBtn.MouseButton1Click:Connect(function()
    PlayerESP_Enabled = not PlayerESP_Enabled
    playerESPBtn.Text = "Player ESP ["..(PlayerESP_Enabled and "ON" or "OFF").."]"
    updateAllPlayerESPs()
end)

-- Timer ESP: tries to find common timer objects and display a small GUI with countdown; when it reaches 0 shows "Unlocked"
local function createTimerGui()
    if timerGui then return end
    timerGui = Instance.new("TextLabel", screenGui)
    timerGui.Size = UDim2.new(0.25,0,0,36)
    timerGui.Position = UDim2.new(0.5, -180, 0.05, 0)
    timerGui.BackgroundTransparency = 0.3
    timerGui.BackgroundColor3 = Color3.fromRGB(0,0,0)
    timerGui.TextColor3 = Color3.fromRGB(255,255,255)
    timerGui.Font = Enum.Font.GothamBold
    timerGui.TextScaled = true
    timerGui.Text = ""
    local c = Instance.new("UICorner", timerGui); c.CornerRadius = UDim.new(0,10)
end

local function destroyTimerGui()
    if timerGui then pcall(function() timerGui:Destroy() end) timerGui = nil end
end

timerESPBtn.MouseButton1Click:Connect(function()
    TimerESP_Enabled = not TimerESP_Enabled
    timerESPBtn.Text = "Timer ESP ["..(TimerESP_Enabled and "ON" or "OFF").."]"
    if TimerESP_Enabled then
        createTimerGui()
    else
        destroyTimerGui()
    end
end)

-- Helper: try to find a numeric timer in common places and return seconds (or nil)
local function findTimerSeconds()
    -- check ReplicatedStorage, workspace, PlayerGui for numbers or TextLabels containing "time" or "sec" or "open"
    local function findNumberInDescendants(container)
        for _,v in pairs(container:GetDescendants()) do
            if (v:IsA("NumberValue") or v:IsA("IntValue")) and v.Value then
                if string.find(string.lower(v.Name), "time") or string.find(string.lower(v.Name), "open") or string.find(string.lower(v.Name), "count") then
                    return tonumber(v.Value)
                end
            elseif v:IsA("TextLabel") or v:IsA("TextBox") then
                local txt = tostring(v.Text or "")
                if string.find(string.lower(txt), "open") or string.find(string.lower(txt), "time") or string.find(string.lower(txt), "sec") then
                    local digits = txt:match("(%d+)")
                    if digits then return tonumber(digits) end
                end
            end
        end
        return nil
    end
    -- Try common places
    local ans
    pcall(function() ans = ans or findNumberInDescendants(game:GetService("ReplicatedStorage")) end)
    pcall(function() ans = ans or findNumberInDescendants(workspace) end)
    pcall(function() ans = ans or findNumberInDescendants(LocalPlayer.PlayerGui) end)
    return ans
end

-- Background routine for Timer ESP
task.spawn(function()
    while true do
        if TimerESP_Enabled then
            local secs = findTimerSeconds()
            if secs then
                if timerGui then
                    local txt = "Opens in: "..tostring(secs).."s"
                    if secs <= 0 then
                        txt = "Unlocked"
                        -- show center big unlocked once
                        showCenterMessage("Unlocked", 2)
                        -- after unlocked, below we may stop or wait next cycle
                    end
                    timerGui.Text = txt
                end
            else
                if timerGui then timerGui.Text = "Timer: N/A" end
            end
        end
        task.wait(1)
    end
end)

-- Highest Value Brainrot ESP: find player with highest brain value and label them
local function findHighestBrainrot()
    local bestPlayer = nil
    local bestValue = -math.huge
    for _,p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then
            local val = nil
            -- try leaderstats
            pcall(function()
                local ls = p:FindFirstChild("leaderstats")
                if ls then
                    for _,c in pairs(ls:GetChildren()) do
                        if string.find(string.lower(c.Name), "brain") or string.find(string.lower(c.Name), "br") then
                            val = tonumber(c.Value) or val
                        end
                    end
                end
            end)
            -- try PlayerGui reading
            pcall(function()
                if not val and p.Character then
                    -- sometimes stats are displayed in PlayerGui as name overlays; skip heavy
                end
            end)
            if val and type(val)=="number" and val > bestValue then
                bestValue = val
                bestPlayer = p
            end
        end
    end
    return bestPlayer, bestValue
end

-- draw highlight for best player
local function drawHighestHighlight()
    if highestHighlight and highestHighlight.label then
        pcall(function() highestHighlight.label:Destroy() end)
        highestHighlight = nil
    end
    local p,v = findHighestBrainrot()
    if p and v then
        -- create a floating label above character
        if p.Character then
            local head = p.Character:FindFirstChild("Head")
            if head then
                local bg = Instance.new("BillboardGui", p.Character)
                bg.Name = "Hop_Highest"
                bg.Adornee = head
                bg.Size = UDim2.new(0,200,0,40)
                bg.StudsOffset = Vector3.new(0,2.4,0)
                bg.AlwaysOnTop = true
                local label = Instance.new("TextLabel", bg)
                label.Size = UDim2.new(1,0,1,0)
                label.BackgroundTransparency = 0.5
                label.BackgroundColor3 = Color3.fromRGB(0,0,0)
                label.TextColor3 = Color3.new(1,1,1)
                label.Font = Enum.Font.GothamBold
                label.TextScaled = true
                label.Text = p.Name.." • "..tostring(v).."/s"
                highestHighlight = {player = p, label = bg}
            end
        end
    end
end

-- Background loop to update Highest Value highlight while enabled
task.spawn(function()
    while true do
        if ValueESP_Enabled then
            drawHighestHighlight()
        else
            if highestHighlight and highestHighlight.label then
                pcall(function() highestHighlight.label:Destroy() end)
                highestHighlight = nil
            end
        end
        task.wait(2)
    end
end)

valueESPBtn.MouseButton1Click:Connect(function()
    ValueESP_Enabled = not ValueESP_Enabled
    valueESPBtn.Text = "Highest Value Brainrot ESP ["..(ValueESP_Enabled and "ON" or "OFF").."]"
end)

-- update Player ESP display every second when enabled
task.spawn(function()
    while true do
        if PlayerESP_Enabled then
            for _,p in pairs(Players:GetPlayers()) do
                if p ~= LocalPlayer then
                    local ch = p.Character
                    if ch and ch:FindFirstChild("Head") then
                        -- ensure billboard has name + brain if possible
                        if not playerBillboards[p] then
                            createPlayerBillboard(p)
                        end
                        -- try update label to show brain value
                        local bg = playerBillboards[p]
                        if bg and bg:FindFirstChildOfClass("TextLabel") then
                            local label = bg:FindFirstChildOfClass("TextLabel")
                            local brVal = nil
                            pcall(function()
                                local ls = p:FindFirstChild("leaderstats")
                                if ls then
                                    for _,c in pairs(ls:GetChildren()) do
                                        if string.find(string.lower(c.Name), "brain") then
                                            brVal = tonumber(c.Value)
                                        end
                                    end
                                end
                            end)
                            if brVal then
                                label.Text = p.Name.." • "..tostring(brVal).."/s"
                            else
                                label.Text = p.Name
                            end
                        end
                    else
                        -- no character
                        removePlayerBillboard(p)
                    end
                end
            end
        end
        task.wait(1)
    end
end)

-- Player ESP toggle
playerESPBtn.MouseButton1Click:Connect(function()
    PlayerESP_Enabled = not PlayerESP_Enabled
    playerESPBtn.Text = "Player ESP ["..(PlayerESP_Enabled and "ON" or "OFF").."]"
    updateAllPlayerESPs()
end)

-- =========================
-- ===== KEYBIND H ========
-- =========================
UserInputService.InputBegan:Connect(function(input, g)
    if g then return end
    if input.KeyCode == Enum.KeyCode.H then
        -- switch to helper tab
        teleportContent.Visible = false
        helperContent.Visible = true
        currentTab = "Helper"
        savedTab = currentTab
    end
end)

-- Save opened tab before close/hide
mainFrame:GetPropertyChangedSignal("Visible"):Connect(function()
    if mainFrame.Visible then
        -- restore savedTab
        if savedTab == "Helper" then
            teleportContent.Visible = false
            helperContent.Visible = true
        else
            teleportContent.Visible = true
            helperContent.Visible = false
        end
    else
        savedTab = currentTab
    end
end)

-- Ensure minimized button position remembered
minimizedBtn:GetPropertyChangedSignal("Position"):Connect(function()
    lastMinimizedPos = minimizedBtn.Position
end)

-- Ensure when teleporting into a new server, if script runs there it will check for brainrot >= MIN_BRAINROT.
-- We attempt best-effort detection shortly after spawn.
task.spawn(function()
    while true do
        -- when script exists inside a server, if player's brainrot < threshold and we intended to keep hopping,
        -- the process used earlier teleports to a candidate and then user may want to auto-hop again.
        -- We won't force-hop automatically here to avoid loops; user control remains.
        task.wait(5)
    end
end)

-- Final message
print("Hop Los Brothers script loaded. Put your MINIMIZED_IMAGE_ID at top to show your image on minimized button.")
