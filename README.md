-- LocalScript único: Painel de Teste (seguro, sem loadstring)
-- Colocar em StarterGui (ex: StarterGui > ScreenGui > LocalScript)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

-- === GUI ===
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BorralhoDevourTest"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 260, 0, 360)
frame.Position = UDim2.new(0.02, 0, 0.15, 0)
frame.BackgroundColor3 = Color3.fromRGB(28,28,28)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local uiCorner = Instance.new("UICorner", frame)
uiCorner.CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 44)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Borralho Devour"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.new(1,1,1)

local function makeButton(text, y)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, -20, 0, 36)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(18,18,18)
    btn.BorderSizePixel = 0
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 15
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Text = text
    local c = Instance.new("UICorner", btn)
    c.CornerRadius = UDim.new(0, 8)
    return btn
end

local offsetY = 54
local btnLag = makeButton("Lag", offsetY)
local btnAntiDevour = makeButton("Anti Devour", offsetY + 44)
local btnSwitchClone = makeButton("Switch Clone", offsetY + 88)
local btnSpamFlood = makeButton("Spam: FLOOD", offsetY + 132)
local btnSpamOff = makeButton("Spam: OFF", offsetY + 176)
local btnLaser = makeButton("Laser Cap (OFF)", offsetY + 220)
local btnSpeed = makeButton("Speed: OFF", offsetY + 264)
local btnESP = makeButton("ESP: OFF", offsetY + 308)

-- Small status label
local status = Instance.new("TextLabel", frame)
status.Size = UDim2.new(1, -20, 0, 20)
status.Position = UDim2.new(0, 10, 1, -28)
status.BackgroundTransparency = 1
status.Text = "Status: pronto"
status.TextColor3 = Color3.fromRGB(200,200,200)
status.TextSize = 13
status.Font = Enum.Font.Gotham

-- === Estados ===
local spamFlooding = false
local laserOn = false
local speedOn = false
local espOn = false

-- Helper: safe character references
local function getChar()
    return LocalPlayer.Character
end
local function getRoot()
    local ch = getChar()
    return ch and ch:FindFirstChild("HumanoidRootPart")
end

-- === LAG (simulação visual) ===
btnLag.MouseButton1Click:Connect(function()
    local root = getRoot()
    if not root then return end
    status.Text = "Status: Lag simulado"
    -- pequena vibração/teleport para frente e para trás
    for i = 1, 8 do
        root.CFrame = root.CFrame * CFrame.new(0, 0, 2)
        task.wait(0.06)
        root.CFrame = root.CFrame * CFrame.new(0, 0, -2)
        task.wait(0.06)
    end
    status.Text = "Status: pronto"
end)

-- === ANTI DEVOUR (simples) ===
btnAntiDevour.MouseButton1Click:Connect(function()
    local root = getRoot()
    if not root then return end
    status.Text = "Status: Anti Devour ativo (curto)"
    local orig = root.Anchored
    root.Anchored = true
    task.delay(0.45, function()
        if root and root.Parent then
            root.Anchored = orig
            status.Text = "Status: pronto"
        end
    end)
end)

-- === SWITCH CLONE (tempo curto) ===
btnSwitchClone.MouseButton1Click:Connect(function()
    local ch = getChar()
    if not ch then return end
    status.Text = "Status: Clone criado"
    local clone = ch:Clone()
    clone.Name = "TestClone_"..tostring(LocalPlayer.UserId)
    clone.Parent = workspace
    -- coloca semi-transparente e remove depois
    for _, part in ipairs(clone:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = math.clamp((part.Transparency or 0) + 0.3, 0, 1)
            part.CanCollide = false
        end
    end
    task.delay(3, function() if clone and clone.Parent then clone:Destroy() end; status.Text = "Status: pronto" end)
end)

-- === SPAM FLOOD / SPAM OFF ===
btnSpamFlood.MouseButton1Click:Connect(function()
    if spamFlooding then
        spamFlooding = false
        status.Text = "Status: spam parado"
        return
    end
    spamFlooding = true
    status.Text = "Status: spam FLOOD ativo"
    task.spawn(function()
        local i = 0
        while spamFlooding do
            i = i + 1
            -- simulação: print (você pode trocar por outra ação de teste)
            print("FLOOD TEST", i)
            task.wait(0.08)
            if i >= 1000 then break end
        end
        status.Text = "Status: pronto"
    end)
end)

btnSpamOff.MouseButton1Click:Connect(function()
    spamFlooding = false
    status.Text = "Status: spam OFF"
end)

-- === SPEED toggle (simulado alterando WalkSpeed local) ===
local DEFAULT_WALKSPEED = 16
local SPEED_VALUE = 28
btnSpeed.MouseButton1Click:Connect(function()
    local ch = getChar()
    if not ch then return end
    local hum = ch:FindFirstChildWhichIsA("Humanoid")
    if not hum then return end
    speedOn = not speedOn
    if speedOn then
        hum.WalkSpeed = SPEED_VALUE
        btnSpeed.Text = "Speed: ON"
        status.Text = "Status: Speed ON"
    else
        hum.WalkSpeed = DEFAULT_WALKSPEED
        btnSpeed.Text = "Speed: OFF"
        status.Text = "Status: Speed OFF"
    end
end)

-- === ESP (simulado com BillboardGui) ===
local espMarkers = {}
local function createESPForPlayer(plr)
    if espMarkers[plr] then return end
    if not plr.Character then return end
    local humRoot = plr.Character:FindFirstChild("HumanoidRootPart")
    if not humRoot then return end
    local bb = Instance.new("BillboardGui")
    bb.Name = "ESP_BB"
    bb.Adornee = humRoot
    bb.Size = UDim2.new(0, 80, 0, 30)
    bb.StudsOffset = Vector3.new(0, 2.6, 0)
    bb.AlwaysOnTop = true
    local label = Instance.new("TextLabel", bb)
    label.Size = UDim2.new(1,0,1,0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1,0.9,0.2)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14
    label.Text = plr.Name
    bb.Parent = plr:FindFirstChildWhichIsA("Backpack") or plr:WaitForChild("PlayerGui") -- safe place
    espMarkers[plr] = bb
end

local function removeAllESP()
    for plr, bb in pairs(espMarkers) do
        if bb and bb.Parent then bb:Destroy() end
        espMarkers[plr] = nil
    end
end

btnESP.MouseButton1Click:Connect(function()
    espOn = not espOn
    if espOn then
        btnESP.Text = "ESP: ON"
        status.Text = "Status: ESP ON"
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer then createESPForPlayer(plr) end
        end
    else
        btnESP.Text = "ESP: OFF"
        status.Text = "Status: ESP OFF"
        removeAllESP()
    end
end)

Players.PlayerAdded:Connect(function(plr)
    if espOn then
        -- espera character
        plr.CharacterAdded:Connect(function()
            createESPForPlayer(plr)
        end)
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    if espMarkers[plr] then
        if espMarkers[plr].Parent then espMarkers[plr]:Destroy() end
        espMarkers[plr] = nil
    end
end)

-- === AUTO GRAB SIMULADO ===
-- procura por objetos "Pickup" ou "Braid" no workspace — você pode ajustar o filtro
local function findNearestPickup(maxDist)
    local root = getRoot()
    if not root then return nil end
    local nearest, nd = nil, math.huge
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and (obj.Name:lower():find("pickup") or obj.Name:lower():find("brair") or obj.Name:lower():find("brairot")) then
            local d = (obj.Position - root.Position).Magnitude
            if d < nd and d <= maxDist then nd = d; nearest = obj end
        end
    end
    return nearest, nd
end

-- small UI to trigger simulated AutoGrab by clicking the title (optional)
title.MouseButton1Click:Connect(function()
    local pickup, d = findNearestPickup(80)
    if pickup then
        -- simula pegar: move o player para o item rapidamente e volta
        local root = getRoot()
        if not root then return end
        local old = root.CFrame
        root.CFrame = CFrame.new(pickup.Position + Vector3.new(0, 3, 0))
        task.wait(0.25)
        root.CFrame = old
        status.Text = "AutoGrab simulado (teleport curto)"
        task.delay(1, function() status.Text = "Status: pronto" end)
    else
        status.Text = "AutoGrab: nenhum pickup encontrado"
        task.delay(1.2, function() status.Text = "Status: pronto" end)
    end
end)

-- === LASER CAP (visual que segue o player mais próximo) ===
-- cria attachments/beam no jogador local; ao ativar, procura o alvo mais próximo e prende o beam nele
local laser = {}
do
    local att0, beam = nil, nil
    local function make()
        att0 = Instance.new("Attachment")
        att0.Name = "Laser_Att0"
        beam = Instance.new("Beam")
        beam.Name = "LaserBeam"
        beam.Attachment0 = att0
        beam.Attachment1 = nil
        beam.FaceCamera = true
        beam.Width0 = 0.4
        beam.Width1 = 0.18
        beam.LightEmission = 1
        beam.LightInfluence = 0
        beam.TextureSpeed = 2
        beam.Segments = 4
        local col = Instance.new("ColorSequence")
        col.Keypoints = { ColorSequenceKeypoint.new(0, Color3.fromRGB(255,60,60)), ColorSequenceKeypoint.new(1, Color3.fromRGB(255,200,200)) }
        beam.Color = col
        return att0, beam
    end

    att0, beam = make()
    laser.att0 = att0
    laser.beam = beam
end

local function attachOrigin()
    local root = getRoot()
    if root and laser.att0 and (not laser.att0.Parent) then
        laser.att0.Parent = root
        laser.att0.Position = Vector3.new(0, 1.2, 0)
        laser.beam.Parent = laser.att0.Parent or workspace
    end
end

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.1)
    attachOrigin()
end)
attachOrigin()

local function findNearestPlayerWithin(distMax)
    local root = getRoot()
    if not root then return nil end
    local nearest, nd = nil, math.huge
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local d = (plr.Character.HumanoidRootPart.Position - root.Position).Magnitude
            if d < nd and d <= distMax then nd = d; nearest = plr end
        end
    end
    return nearest, nd
end

local LASER_RANGE = 200
local laserTarget = nil
local laserTick = 0

btnLaser.MouseButton1Click:Connect(function()
    laserOn = not laserOn
    btnLaser.Text = laserOn and "Laser Cap (ON)" or "Laser Cap (OFF)"
    status.Text = laserOn and "Status: Laser ON" or "Status: Laser OFF"
    if not laserOn then
        if laser.beam and laser.beam.Attachment1 and laser.beam.Attachment1.Parent then
            laser.beam.Attachment1:Destroy()
            laser.beam.Attachment1 = nil
        end
        laserTarget = nil
    end
end)

-- atualiza laser visual a cada frame se ligado
RunService.Heartbeat:Connect(function(dt)
    -- ensure origin attachment exists
    attachOrigin()

    if laserOn then
        laserTick = laserTick + dt
        -- update target a pouco mais lento para reduzir checagens
        if laserTick > 0.06 then
            laserTick = 0
            local target, d = findNearestPlayerWithin(LASER_RANGE)
            laserTarget = target
        end

        if laserTarget and laserTarget.Character and laserTarget.Character:FindFirstChild("HumanoidRootPart") then
            local targetRoot = laserTarget.Character.HumanoidRootPart
            if not laser.beam.Attachment1 or not laser.beam.Attachment1.Parent then
                local att1 = Instance.new("Attachment")
                att1.Name = "Laser_Att1"
                att1.Parent = targetRoot
                att1.Position = Vector3.new(0, 0, 0)
                laser.beam.Attachment1 = att1
                laser.beam.Parent = laser.att0.Parent or workspace
            else
                -- se já tem attachment1, apenas garante que o parent ainda é o targetRoot
                if laser.beam.Attachment1.Parent ~= targetRoot then
                    laser.beam.Attachment1:Destroy()
                    local att1 = Instance.new("Attachment")
                    att1.Name = "Laser_Att1"
                    att1.Parent = targetRoot
                    att1.Position = Vector3.new(0, 0, 0)
                    laser.beam.Attachment1 = att1
                end
            end
            -- opcional: efeito de "pulso" na largura
            local pulse = 0.25 + (math.sin(time() * 15) * 0.06)
            laser.beam.Width0 = pulse
            laser.beam.Width1 = pulse * 0.45
        else
            -- sem alvo: limpa attachment1
            if laser.beam.Attachment1 and laser.beam.Attachment1.Parent then
                laser.beam.Attachment1:Destroy()
                laser.beam.Attachment1 = nil
            end
        end
    end

    -- atualiza ESPs se ativo (verifica se novos jogadores entraram)
    if espOn then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and not espMarkers[plr] and plr.Character then
                createESPForPlayer(plr)
            end
        end
    end
end)

-- === Segurança / limpeza ao sair ===
LocalPlayer.AncestryChanged:Connect(function()
    if not LocalPlayer:IsDescendantOf(game) then
        if laser.beam and laser.beam.Attachment1 and laser.beam.Attachment1.Parent then
            laser.beam.Attachment1:Destroy()
            laser.beam.Attachment1 = nil
        end
        if laser.att0 and laser.att0.Parent then laser.att0:Destroy() end
        removeAllESP()
    end
end)

-- FIM DO SCRIPT
