-- // [TAG] Meow Madness script // --

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

if LocalPlayer.PlayerGui:FindFirstChild("SeraphTeleportUI") then
    LocalPlayer.PlayerGui.SeraphTeleportUI:Destroy()
end

local teleportEnabled = false
local espEnabled = false
local fovEnabled = false
local hitboxEnabled = false
local playerNoclipEnabled = false
local teleportSpeed = 0.16
local stickTime = 0.14
local upDistance = 30
local maxTeleportDistance = 1000
local hitboxSize = 15

local targets = {}
local currentTargetIndex = 1
local currentStickTarget = nil
local stickConnection = nil
local fovConnection = nil
local espConnections = {}
local originalSizes = {}
local originalCanCollide = {}

-- ==================== UI ====================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SeraphTeleportUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = LocalPlayer.PlayerGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 520, 0, 365)  -- VFlyボタン追加で少し高く
Frame.Position = UDim2.new(0.5, -260, 0.10, 0)
Frame.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui
Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 14)

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Text = "[TAG] Meow Madness script"
Title.TextColor3 = Color3.fromRGB(0, 255, 160)
Title.TextScaled = true
Title.Font = Enum.Font.GothamBold
Title.BackgroundTransparency = 1

local TeleportStatus = Instance.new("TextLabel", Frame)
TeleportStatus.Position = UDim2.new(0, 10, 0, 32)
TeleportStatus.Size = UDim2.new(0.48, 0, 0, 18)
TeleportStatus.Text = "瞬足: OFF"
TeleportStatus.TextColor3 = Color3.fromRGB(255, 70, 70)
TeleportStatus.TextScaled = true
TeleportStatus.Font = Enum.Font.GothamSemibold
TeleportStatus.BackgroundTransparency = 1
TeleportStatus.TextXAlignment = Enum.TextXAlignment.Left

local EspStatus = Instance.new("TextLabel", Frame)
EspStatus.Position = UDim2.new(0, 10, 0, 50)
EspStatus.Size = UDim2.new(0.48, 0, 0, 18)
EspStatus.Text = "ESP: OFF"
EspStatus.TextColor3 = Color3.fromRGB(255, 70, 70)
EspStatus.TextScaled = true
EspStatus.Font = Enum.Font.GothamSemibold
EspStatus.BackgroundTransparency = 1
EspStatus.TextXAlignment = Enum.TextXAlignment.Left

local FovStatus = Instance.new("TextLabel", Frame)
FovStatus.Position = UDim2.new(0.52, 0, 0, 32)
FovStatus.Size = UDim2.new(0.46, 0, 0, 18)
FovStatus.Text = "FOV120: OFF"
FovStatus.TextColor3 = Color3.fromRGB(255, 70, 70)
FovStatus.TextScaled = true
FovStatus.Font = Enum.Font.GothamSemibold
FovStatus.BackgroundTransparency = 1
FovStatus.TextXAlignment = Enum.TextXAlignment.Left

local HitboxStatus = Instance.new("TextLabel", Frame)
HitboxStatus.Position = UDim2.new(0.52, 0, 0, 50)
HitboxStatus.Size = UDim2.new(0.46, 0, 0, 18)
HitboxStatus.Text = "Hitbox: OFF"
HitboxStatus.TextColor3 = Color3.fromRGB(255, 70, 70)
HitboxStatus.TextScaled = true
HitboxStatus.Font = Enum.Font.GothamSemibold
HitboxStatus.BackgroundTransparency = 1
HitboxStatus.TextXAlignment = Enum.TextXAlignment.Left

local NoclipStatus = Instance.new("TextLabel", Frame)
NoclipStatus.Position = UDim2.new(0, 10, 0, 68)
NoclipStatus.Size = UDim2.new(0.9, 0, 0, 18)
NoclipStatus.Text = "human noclip: OFF"
NoclipStatus.TextColor3 = Color3.fromRGB(255, 70, 70)
NoclipStatus.TextScaled = true
NoclipStatus.Font = Enum.Font.GothamSemibold
NoclipStatus.BackgroundTransparency = 1
NoclipStatus.TextXAlignment = Enum.TextXAlignment.Left

local SpeedLabel = Instance.new("TextLabel", Frame)
SpeedLabel.Position = UDim2.new(0, 10, 0, 90)
SpeedLabel.Size = UDim2.new(0.45, 0, 0, 16)
SpeedLabel.Text = "TP Speed: 0.16s"
SpeedLabel.TextColor3 = Color3.fromRGB(180, 180, 255)
SpeedLabel.TextScaled = true
SpeedLabel.Font = Enum.Font.Gotham
SpeedLabel.BackgroundTransparency = 1
SpeedLabel.TextXAlignment = Enum.TextXAlignment.Left

local SliderBG = Instance.new("Frame", Frame)
SliderBG.Size = UDim2.new(0.45, 0, 0, 20)
SliderBG.Position = UDim2.new(0.02, 0, 0, 108)
SliderBG.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
SliderBG.BorderSizePixel = 0
Instance.new("UICorner", SliderBG).CornerRadius = UDim.new(0, 8)

local SliderFill = Instance.new("Frame", SliderBG)
SliderFill.Size = UDim2.new(0.16, 0, 1, 0)
SliderFill.BackgroundColor3 = Color3.fromRGB(0, 220, 140)
SliderFill.BorderSizePixel = 0
Instance.new("UICorner", SliderFill).CornerRadius = UDim.new(0, 8)

local SliderButton = Instance.new("TextButton", SliderBG)
SliderButton.Size = UDim2.new(0, 20, 0, 20)
SliderButton.Position = UDim2.new(0.16, -10, 0, 0)
SliderButton.Text = ""
SliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
SliderButton.BorderSizePixel = 0
Instance.new("UICorner", SliderButton).CornerRadius = UDim.new(1, 0)

local HitboxLabel = Instance.new("TextLabel", Frame)
HitboxLabel.Position = UDim2.new(0.52, 0, 0, 90)
HitboxLabel.Size = UDim2.new(0.45, 0, 0, 16)
HitboxLabel.Text = "Hitbox Size: 15"
HitboxLabel.TextColor3 = Color3.fromRGB(255, 180, 100)
HitboxLabel.TextScaled = true
HitboxLabel.Font = Enum.Font.Gotham
HitboxLabel.BackgroundTransparency = 1
HitboxLabel.TextXAlignment = Enum.TextXAlignment.Left

local HitboxBG = Instance.new("Frame", Frame)
HitboxBG.Size = UDim2.new(0.45, 0, 0, 20)
HitboxBG.Position = UDim2.new(0.52, 0, 0, 108)
HitboxBG.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
HitboxBG.BorderSizePixel = 0
Instance.new("UICorner", HitboxBG).CornerRadius = UDim.new(0, 8)

local HitboxFill = Instance.new("Frame", HitboxBG)
HitboxFill.Size = UDim2.new(0.015, 0, 1, 0)
HitboxFill.BackgroundColor3 = Color3.fromRGB(255, 120, 50)
HitboxFill.BorderSizePixel = 0
Instance.new("UICorner", HitboxFill).CornerRadius = UDim.new(0, 8)

local HitboxButton = Instance.new("TextButton", HitboxBG)
HitboxButton.Size = UDim2.new(0, 20, 0, 20)
HitboxButton.Position = UDim2.new(0.015, -10, 0, 0)
HitboxButton.Text = ""
HitboxButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
HitboxButton.BorderSizePixel = 0
Instance.new("UICorner", HitboxButton).CornerRadius = UDim.new(1, 0)

local slidingTP, slidingHB = false, false

local function updateTPSlider(pos)
    local absPos = SliderBG.AbsolutePosition
    local absSize = SliderBG.AbsoluteSize
    local relative = math.clamp((pos.X - absPos.X) / absSize.X, 0, 1)
    SliderFill.Size = UDim2.new(relative, 0, 1, 0)
    SliderButton.Position = UDim2.new(relative, -10, 0, 0)
    teleportSpeed = 0.06 + (1 - relative) * 0.5
    SpeedLabel.Text = string.format("TP Speed: %.2fs", teleportSpeed)
end

local function updateHitboxSlider(pos)
    local absPos = HitboxBG.AbsolutePosition
    local absSize = HitboxBG.AbsoluteSize
    local relative = math.clamp((pos.X - absPos.X) / absSize.X, 0, 1)
    HitboxFill.Size = UDim2.new(relative, 0, 1, 0)
    HitboxButton.Position = UDim2.new(relative, -10, 0, 0)
    hitboxSize = math.floor(2 + relative * 9998)
    HitboxLabel.Text = "Hitbox Size: " .. hitboxSize
end

SliderButton.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then slidingTP = true end end)
HitboxButton.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then slidingHB = true end end)

UserInputService.InputChanged:Connect(function(i)
    if slidingTP and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then updateTPSlider(i.Position) end
    if slidingHB and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then updateHitboxSlider(i.Position) end
end)

UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        slidingTP = false
        slidingHB = false
    end
end)

SliderBG.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then slidingTP = true updateTPSlider(i.Position) end end)
HitboxBG.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then slidingHB = true updateHitboxSlider(i.Position) end end)

local function makeBtn(text, color, pos)
    local b = Instance.new("TextButton", Frame)
    b.Size = UDim2.new(0.46, 0, 0, 34)
    b.Position = pos
    b.Text = text
    b.TextScaled = true
    b.Font = Enum.Font.GothamBold
    b.BackgroundColor3 = color
    b.TextColor3 = Color3.new(1, 1, 1)
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 10)
    return b
end

local ToggleTeleport = makeBtn("TP all ON/OFF", Color3.fromRGB(0, 190, 110), UDim2.new(0.02, 0, 0, 145))
local ToggleESP     = makeBtn("ESP ON/OFF", Color3.fromRGB(0, 160, 220), UDim2.new(0.52, 0, 0, 145))
local ToggleFOV     = makeBtn("FOV120 ON/OFF", Color3.fromRGB(180, 80, 255), UDim2.new(0.02, 0, 0, 188))
local ToggleHitbox  = makeBtn("Hitbox ON/OFF", Color3.fromRGB(255, 100, 50), UDim2.new(0.52, 0, 0, 188))
local ToggleNoclip  = makeBtn("human ON/OFF", Color3.fromRGB(255, 60, 120), UDim2.new(0.02, 0, 0, 231))
local LoadVFlyBtn   = makeBtn("VFly Load button", Color3.fromRGB(255, 180, 40), UDim2.new(0.52, 0, 0, 231))
local CloseBtn      = makeBtn("UI closed", Color3.fromRGB(60, 60, 70), UDim2.new(0.27, 0, 0, 275))

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- VFly読み込みボタン
LoadVFlyBtn.MouseButton1Click:Connect(function()
    -- 定番のVFly（複数のソースから試す）
    local success = pcall(function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/XNEOFF/FlyGuiV3/main/FlyGuiV3.txt"))()
    end)
    
    if not success then
        -- フォールバック1
        pcall(function()
            loadstring(game:HttpGet("https://raw.githubusercontent.com/lerkermer/lua-projects/master/TrueSight-Release.lua"))()
        end)
    end
    
    if not success then
        -- シンプルな内蔵Fly（最後の手段）
        local flying = false
        local speed = 50
        local keys = {w=false, a=false, s=false, d=false, space=false, ctrl=false}
        
        local function startFly()
            local char = LocalPlayer.Character
            if not char then return end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not hrp or not hum then return end
            
            flying = true
            hum.PlatformStand = true
            
            local bv = Instance.new("BodyVelocity")
            bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
            bv.Velocity = Vector3.zero
            bv.Parent = hrp
            
            local bg = Instance.new("BodyGyro")
            bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
            bg.P = 9e4
            bg.Parent = hrp
            
            local conn
            conn = RunService.RenderStepped:Connect(function()
                if not flying or not hrp or not hrp.Parent then
                    conn:Disconnect()
                    return
                end
                bg.CFrame = Camera.CFrame
                local dir = Vector3.zero
                if keys.w then dir = dir + Camera.CFrame.LookVector end
                if keys.s then dir = dir - Camera.CFrame.LookVector end
                if keys.a then dir = dir - Camera.CFrame.RightVector end
                if keys.d then dir = dir + Camera.CFrame.RightVector end
                if keys.space then dir = dir + Vector3.new(0, 1, 0) end
                if keys.ctrl then dir = dir - Vector3.new(0, 1, 0) end
                bv.Velocity = dir.Unit * speed
                if dir.Magnitude < 0.1 then bv.Velocity = Vector3.zero end
            end)
            
            UserInputService.InputBegan:Connect(function(input, gpe)
                if gpe then return end
                if input.KeyCode == Enum.KeyCode.W then keys.w = true end
                if input.KeyCode == Enum.KeyCode.A then keys.a = true end
                if input.KeyCode == Enum.KeyCode.S then keys.s = true end
                if input.KeyCode == Enum.KeyCode.D then keys.d = true end
                if input.KeyCode == Enum.KeyCode.Space then keys.space = true end
                if input.KeyCode == Enum.KeyCode.LeftControl then keys.ctrl = true end
            end)
            UserInputService.InputEnded:Connect(function(input)
                if input.KeyCode == Enum.KeyCode.W then keys.w = false end
                if input.KeyCode == Enum.KeyCode.A then keys.a = false end
                if input.KeyCode == Enum.KeyCode.S then keys.s = false end
                if input.KeyCode == Enum.KeyCode.D then keys.d = false end
                if input.KeyCode == Enum.KeyCode.Space then keys.space = false end
                if input.KeyCode == Enum.KeyCode.LeftControl then keys.ctrl = false end
            end)
        end
        
        startFly()
    end
    
    print("VFly 読み込み試行完了")
end)

local dragging, dragStart, startPos = false, nil, nil
Frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        local m = input.Position
        local a = Frame.AbsolutePosition
        if m.Y > a.Y + 90 and m.Y < a.Y + 140 then return end
        dragging = true
        dragStart = input.Position
        startPos = Frame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local d = input.Position - dragStart
        Frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- ==================== ESP ====================
local function createNameTag(player)
    if player == LocalPlayer or not espEnabled then return end
    local char = player.Character
    if not char then return end
    local head = char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
    if not head then return end
    if head:FindFirstChild("SeraphNameTag") then head.SeraphNameTag:Destroy() end

    local bb = Instance.new("BillboardGui")
    bb.Name = "SeraphNameTag"
    bb.Adornee = head
    bb.Size = UDim2.new(0, 180, 0, 42)
    bb.StudsOffset = Vector3.new(0, 3.2, 0)
    bb.AlwaysOnTop = true
    bb.MaxDistance = 2000
    bb.Parent = head

    local f = Instance.new("Frame", bb)
    f.Size = UDim2.new(1, 0, 1, 0)
    f.BackgroundTransparency = 1

    local name = Instance.new("TextLabel", f)
    name.Size = UDim2.new(1, 0, 0, 18)
    name.Text = player.Name
    name.TextColor3 = Color3.fromRGB(0, 255, 200)
    name.TextScaled = true
    name.Font = Enum.Font.GothamBold
    name.BackgroundTransparency = 1
    name.TextStrokeTransparency = 0.3

    local dist = Instance.new("TextLabel", f)
    dist.Size = UDim2.new(1, 0, 0, 15)
    dist.Position = UDim2.new(0, 0, 0, 20)
    dist.Text = ""
    dist.TextColor3 = Color3.fromRGB(180, 200, 255)
    dist.TextScaled = true
    dist.Font = Enum.Font.Gotham
    dist.BackgroundTransparency = 1

    espConnections[player] = {billboard = bb, distLabel = dist}
end

local function removeNameTag(player)
    if espConnections[player] then
        pcall(function() espConnections[player].billboard:Destroy() end)
        espConnections[player] = nil
    end
end

local function forceUpdateAllESP()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            removeNameTag(plr)
            if espEnabled then createNameTag(plr) end
        end
    end
end

local function updateAllESP()
    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    for player, data in pairs(espConnections) do
        if data.distLabel and myRoot and player.Character then
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            if root then
                data.distLabel.Text = math.floor((root.Position - myRoot.Position).Magnitude) .. "s"
            end
        end
    end
end

-- ==================== Hitbox ====================
local function expandHitbox(player)
    if player == LocalPlayer then return end
    local char = player.Character
    if not char then return end
    for _, part in ipairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            if not originalSizes[part] then originalSizes[part] = part.Size end
            part.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
            part.Transparency = 0.7
            part.CanCollide = false
            part.Massless = true
        end
    end
end

local function restoreHitbox(player)
    local char = player.Character
    if not char then return end
    for _, part in ipairs(char:GetChildren()) do
        if part:IsA("BasePart") and originalSizes[part] then
            part.Size = originalSizes[part]
            part.Transparency = 0
            part.CanCollide = true
            part.Massless = false
        end
    end
end

local function applyAllHitboxes()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if hitboxEnabled then expandHitbox(player) else restoreHitbox(player) end
        end
    end
end

RunService.Heartbeat:Connect(function()
    if hitboxEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                for _, part in ipairs(player.Character:GetChildren()) do
                    if part:IsA("BasePart") and part.Size.X < hitboxSize * 0.9 then
                        part.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
                        part.Transparency = 0.7
                        part.CanCollide = false
                        part.Massless = true
                    end
                end
            end
        end
    end
end)

-- ==================== 人すり抜け ====================
local function setPlayerNoclip(state)
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            for _, part in ipairs(player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    if state then
                        if originalCanCollide[part] == nil then
                            originalCanCollide[part] = part.CanCollide
                        end
                        part.CanCollide = false
                    else
                        if originalCanCollide[part] ~= nil then
                            part.CanCollide = originalCanCollide[part]
                        end
                    end
                end
            end
        end
    end
end

RunService.Stepped:Connect(function()
    if playerNoclipEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                for _, part in ipairs(player.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end
    end
end)

-- ==================== タグ特化瞬足 ====================
local function isValidTarget(player)
    if player == LocalPlayer then return false end
    local char = player.Character
    if not char then return false end
    local root = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not (root and hum and hum.Health > 0) then return false end
    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myRoot then return false end
    return (root.Position - myRoot.Position).Magnitude <= maxTeleportDistance
end

local function updateTargets()
    targets = {}
    for _, plr in ipairs(Players:GetPlayers()) do
        if isValidTarget(plr) then
            table.insert(targets, plr)
        end
    end
end

local function getRoot(player)
    return player and player.Character and player.Character:FindFirstChild("HumanoidRootPart")
end

local function startStick(target)
    if stickConnection then
        stickConnection:Disconnect()
        stickConnection = nil
    end
    currentStickTarget = target

    stickConnection = RunService.Heartbeat:Connect(function()
        if not teleportEnabled or not currentStickTarget or not currentStickTarget.Character then
            if stickConnection then stickConnection:Disconnect() stickConnection = nil end
            return
        end
        local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        local tRoot = getRoot(currentStickTarget)
        if myRoot and tRoot then
            myRoot.CFrame = tRoot.CFrame * CFrame.new(0, 1.3, 1.0)
        end
    end)
end

local function stopStick()
    if stickConnection then
        stickConnection:Disconnect()
        stickConnection = nil
    end
    currentStickTarget = nil
end

local function doTeleportSequence(target)
    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local tRoot = getRoot(target)
    if not myRoot or not tRoot then return end

    myRoot.CFrame = tRoot.CFrame * CFrame.new(0, 1.3, 1.0)
    task.wait(0.03)

    tRoot = getRoot(target)
    if tRoot and myRoot then
        myRoot.CFrame = CFrame.new(tRoot.Position + Vector3.new(0, upDistance, 0))
    end
    task.wait(0.04)

    tRoot = getRoot(target)
    if tRoot and myRoot then
        myRoot.CFrame = tRoot.CFrame * CFrame.new(0, 1.3, 1.0)
    end

    startStick(target)
end

local function startTeleportLoop()
    while teleportEnabled do
        updateTargets()
        if #targets > 0 then
            if currentTargetIndex > #targets then
                currentTargetIndex = 1
            end
            local target = targets[currentTargetIndex]
            if target and isValidTarget(target) then
                doTeleportSequence(target)
            end
            currentTargetIndex = currentTargetIndex + 1
        end
        task.wait(teleportSpeed + stickTime)
        stopStick()
    end
    stopStick()
end

ToggleTeleport.MouseButton1Click:Connect(function()
    teleportEnabled = not teleportEnabled
    TeleportStatus.Text = "TP all: " .. (teleportEnabled and "ON (追跡+↑30)" or "OFF")
    TeleportStatus.TextColor3 = teleportEnabled and Color3.fromRGB(0, 255, 140) or Color3.fromRGB(255, 70, 70)
    if teleportEnabled then
        currentTargetIndex = 1
        task.spawn(startTeleportLoop)
    else
        stopStick()
    end
end)

ToggleESP.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    EspStatus.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
    EspStatus.TextColor3 = espEnabled and Color3.fromRGB(0, 255, 140) or Color3.fromRGB(255, 70, 70)
    if espEnabled then forceUpdateAllESP() else
        for _, plr in ipairs(Players:GetPlayers()) do removeNameTag(plr) end
    end
end)

ToggleFOV.MouseButton1Click:Connect(function()
    fovEnabled = not fovEnabled
    FovStatus.Text = "FOV120: " .. (fovEnabled and "ON" or "OFF")
    FovStatus.TextColor3 = fovEnabled and Color3.fromRGB(0, 255, 140) or Color3.fromRGB(255, 70, 70)
    if fovEnabled then
        if not fovConnection then
            fovConnection = RunService.RenderStepped:Connect(function() Camera.FieldOfView = 120 end)
        end
    else
        if fovConnection then fovConnection:Disconnect() fovConnection = nil end
        Camera.FieldOfView = 70
    end
end)

ToggleHitbox.MouseButton1Click:Connect(function()
    hitboxEnabled = not hitboxEnabled
    HitboxStatus.Text = "Hitbox: " .. (hitboxEnabled and "ON (" .. hitboxSize .. ")" or "OFF")
    HitboxStatus.TextColor3 = hitboxEnabled and Color3.fromRGB(0, 255, 140) or Color3.fromRGB(255, 70, 70)
    applyAllHitboxes()
end)

ToggleNoclip.MouseButton1Click:Connect(function()
    playerNoclipEnabled = not playerNoclipEnabled
    NoclipStatus.Text = "Cat noclip: " .. (playerNoclipEnabled and "ON" or "OFF")
    NoclipStatus.TextColor3 = playerNoclipEnabled and Color3.fromRGB(0, 255, 140) or Color3.fromRGB(255, 70, 70)
    setPlayerNoclip(playerNoclipEnabled)
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(1)
        if espEnabled then createNameTag(player) end
        if hitboxEnabled then expandHitbox(player) end
        if playerNoclipEnabled then setPlayerNoclip(true) end
    end)
end)
Players.PlayerRemoving:Connect(removeNameTag)

for _, player in ipairs(Players:GetPlayers()) do
    player.CharacterAdded:Connect(function()
        task.wait(1)
        if espEnabled then createNameTag(player) end
        if hitboxEnabled then expandHitbox(player) end
        if playerNoclipEnabled then setPlayerNoclip(true) end
    end)
end

RunService.Heartbeat:Connect(updateAllESP)

print("[TAG] Meow Madness script")
