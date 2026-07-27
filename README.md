-- =============================================
-- R9ZEN KNIFE DUEL | MOBILE VERSION
-- =============================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local TextChatService = game:GetService("TextChatService")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local Camera = workspace.CurrentCamera
local LP = Players.LocalPlayer

-- SETTINGS
local AimbotEnabled = false
local AimFOV = 200
local AimPart = "Head"
local AimHolding = false
local HitboxEnabled = false
local HitboxSize = 10
local ESPEnabled = false
local FlyEnabled = false
local FlySpeed = 60
local NoclipEnabled = false
local SpeedOn = false
local SpeedVal = 30
local GameFOV = 70
local MenuOpen = true

local Highlights = {}
local HitboxOrig = {}
local FlyBV = nil
local FlyBG = nil
local NoclipConn = nil

-- =============== ANTI-CHEAT BYPASS ===============
task.spawn(function()
    task.wait(0.5)
    pcall(function()
        local mt = getrawmetatable(game)
        setreadonly(mt, false)
        local oldNC = mt.__namecall
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if method == "FireServer" or method == "InvokeServer" then
                local name = tostring(self.Name):lower()
                if name == "report" or name == "reportplayer" or 
                   name == "banplayer" or name == "kickplayer" or
                   name:find("anticheat") or name:find("antiexploit") then
                    return nil
                end
            end
            return oldNC(self, ...)
        end)
        setreadonly(mt, true)
    end)
    pcall(function() hookfunction(LP.Kick, function(...) return end) end)
    pcall(function() LP.Kick = function() end end)
end)

-- =============== AVATAR ===============
task.spawn(function()
    for i = 1, 3 do
        task.wait(2)
        pcall(function()
            local desc = Players:GetHumanoidDescriptionFromUserId(9648015490)
            if desc and LP.Character then
                local hum = LP.Character:FindFirstChildOfClass("Humanoid")
                if hum then hum:ApplyDescription(desc) end
            end
        end)
    end
end)

LP.CharacterAdded:Connect(function(char)
    task.wait(2)
    pcall(function()
        local desc = Players:GetHumanoidDescriptionFromUserId(9648015490)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if desc and hum then hum:ApplyDescription(desc) end
    end)
end)

-- =============== AUTO CHAT ===============
local function SendChat(msg)
    pcall(function()
        local channel = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
        if channel then channel:SendAsync(msg) return end
    end)
    pcall(function()
        game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(msg, "All")
    end)
end

task.spawn(function()
    task.wait(30)
    while true do
        pcall(function() SendChat("R9zenOnTop") end)
        task.wait(180)
    end
end)

-- =============== AIMBOT ===============
local function GetTarget()
    local closest, shortest = nil, AimFOV
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    local myChar = LP.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil end
    
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LP then continue end
        if LP.Team and plr.Team and LP.Team == plr.Team then continue end
        
        local char = plr.Character
        if not char then continue end
        local part = char:FindFirstChild(AimPart)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not part or not hum or hum.Health <= 0 then continue end
        
        local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
        if onScreen then
            local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
            if dist < shortest then
                shortest = dist
                closest = plr
            end
        end
    end
    return closest
end

-- =============== HITBOX ===============
local function ApplyHitbox()
    if not HitboxEnabled then return end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr == LP then continue end
        if LP.Team and plr.Team and LP.Team == plr.Team then continue end
        if plr.Character then
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health <= 0 then continue end
            local head = plr.Character:FindFirstChild("Head")
            if head then
                pcall(function()
                    if not HitboxOrig[head] then
                        HitboxOrig[head] = {size=head.Size, trans=head.Transparency, mat=head.Material, col=head.Color, cc=head.CanCollide}
                    end
                    head.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
                    head.Transparency = 0.5
                    head.CanCollide = false
                    head.Material = Enum.Material.ForceField
                    head.Color = Color3.fromRGB(255, 0, 0)
                end)
            end
        end
    end
end

local function RestoreHitbox()
    for part, d in pairs(HitboxOrig) do
        pcall(function()
            if part and part.Parent then
                part.Size = d.size
                part.Transparency = d.trans
                part.Material = d.mat
                part.Color = d.col
                part.CanCollide = d.cc
            end
        end)
    end
    HitboxOrig = {}
end

task.spawn(function()
    while true do
        task.wait(0.2)
        if HitboxEnabled then pcall(ApplyHitbox) end
    end
end)

-- =============== ESP ===============
local function AddESP(char)
    if not char then return end
    if Highlights[char] then return end
    pcall(function()
        local hl = Instance.new("Highlight")
        hl.Name = "R9ESP"
        hl.FillColor = Color3.fromRGB(240, 240, 240)
        hl.OutlineColor = Color3.fromRGB(255, 255, 255)
        hl.FillTransparency = 0.5
        hl.OutlineTransparency = 0
        hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        hl.Parent = char
        Highlights[char] = hl
    end)
end

local function RemoveAllESP()
    for c, h in pairs(Highlights) do
        pcall(function() if h and h.Parent then h:Destroy() end end)
    end
    Highlights = {}
end

task.spawn(function()
    while true do
        task.wait(1)
        if ESPEnabled then
            pcall(function()
                for _, plr in pairs(Players:GetPlayers()) do
                    if plr ~= LP and plr.Character then
                        AddESP(plr.Character)
                    end
                end
                -- Temizle
                for char, hl in pairs(Highlights) do
                    if not char or not char.Parent then
                        pcall(function() if hl then hl:Destroy() end end)
                        Highlights[char] = nil
                    end
                end
            end)
        end
    end
end)

-- =============== FLY ===============
local flyKeys = {W=false, A=false, S=false, D=false, Up=false, Down=false}
local flyMoveDir = Vector3.new(0, 0, 0)

local function StartFly()
    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    if FlyBV then FlyBV:Destroy() end
    if FlyBG then FlyBG:Destroy() end
    FlyBV = Instance.new("BodyVelocity")
    FlyBV.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    FlyBV.Velocity = Vector3.new(0, 0, 0)
    FlyBV.Parent = hrp
    FlyBG = Instance.new("BodyGyro")
    FlyBG.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    FlyBG.P = 10000
    FlyBG.CFrame = hrp.CFrame
    FlyBG.Parent = hrp
end

local function StopFly()
    if FlyBV then FlyBV:Destroy() FlyBV = nil end
    if FlyBG then FlyBG:Destroy() FlyBG = nil end
end

RunService.Heartbeat:Connect(function()
    if not FlyEnabled or not FlyBV or not FlyBG then return end
    local moveDir = Vector3.new(0, 0, 0)
    local camCF = Camera.CFrame
    if flyKeys.W then moveDir = moveDir + camCF.LookVector end
    if flyKeys.S then moveDir = moveDir - camCF.LookVector end
    if flyKeys.A then moveDir = moveDir - camCF.RightVector end
    if flyKeys.D then moveDir = moveDir + camCF.RightVector end
    if flyKeys.Up then moveDir = moveDir + Vector3.new(0, 1, 0) end
    if flyKeys.Down then moveDir = moveDir + Vector3.new(0, -1, 0) end
    if moveDir.Magnitude > 0 then moveDir = moveDir.Unit * FlySpeed end
    FlyBV.Velocity = moveDir
    FlyBG.CFrame = camCF
end)

-- =============== NOCLIP ===============
local function EnableNoclip()
    if NoclipConn then NoclipConn:Disconnect() end
    NoclipConn = RunService.Stepped:Connect(function()
        if not NoclipEnabled then return end
        local char = LP.Character
        if not char then return end
        for _, part in pairs(char:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide then
                pcall(function() part.CanCollide = false end)
            end
        end
    end)
end

local function DisableNoclip()
    if NoclipConn then NoclipConn:Disconnect() NoclipConn = nil end
end

LP.CharacterAdded:Connect(function()
    task.wait(1)
    if NoclipEnabled then EnableNoclip() end
    if FlyEnabled then StartFly() end
end)

-- =============== SPEED ===============
task.spawn(function()
    while true do
        task.wait(0.2)
        if SpeedOn then
            pcall(function()
                local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
                if hum then hum.WalkSpeed = SpeedVal end
            end)
        end
    end
end)

-- =============== GUI (MOBILE OPTIMIZED) ===============
local PG
pcall(function() if gethui then PG = gethui() else PG = game:GetService("CoreGui") end end)
if not PG then PG = LP:WaitForChild("PlayerGui") end

pcall(function() for _, v in pairs(PG:GetChildren()) do if v.Name == "R9M" then v:Destroy() end end end)

local G = Instance.new("ScreenGui") G.Name = "R9M" G.ResetOnSpawn = false G.IgnoreGuiInset = true G.DisplayOrder = 99999 G.Parent = PG

-- LOGO (MOBIL - küçük ve draggable)
local Logo = Instance.new("TextButton")
Logo.Size = UDim2.new(0, 50, 0, 50)
Logo.Position = UDim2.new(0, 15, 0, 150)
Logo.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
Logo.Text = "R9"
Logo.TextColor3 = Color3.new(1, 1, 1)
Logo.Font = Enum.Font.GothamBlack
Logo.TextSize = 18
Logo.BorderSizePixel = 0
Logo.Active = true
Logo.Draggable = true
Logo.Parent = G
Instance.new("UICorner", Logo).CornerRadius = UDim.new(0, 8)
local logoStr = Instance.new("UIStroke", Logo)
logoStr.Color = Color3.fromRGB(200, 200, 200)
logoStr.Thickness = 2

-- MAIN (MOBIL - küçük ekran için)
local M = Instance.new("Frame")
M.Size = UDim2.new(0, 260, 0, 380)
M.Position = UDim2.new(0.5, -130, 0.5, -190)
M.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
M.BorderSizePixel = 0
M.Active = true
M.Parent = G
Instance.new("UICorner", M).CornerRadius = UDim.new(0, 8)
local mStr = Instance.new("UIStroke", M)
mStr.Color = Color3.fromRGB(200, 200, 200)
mStr.Thickness = 2

-- BG R9Z
local BGText = Instance.new("TextLabel")
BGText.Size = UDim2.new(1, 0, 1, 0)
BGText.BackgroundTransparency = 1
BGText.Text = "R9Z"
BGText.TextColor3 = Color3.fromRGB(30, 30, 30)
BGText.Font = Enum.Font.GothamBlack
BGText.TextSize = 160
BGText.TextTransparency = 0.5
BGText.ZIndex = 1
BGText.Parent = M

-- Header
local H = Instance.new("Frame")
H.Size = UDim2.new(1, 0, 0, 40)
H.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
H.BorderSizePixel = 0
H.Active = true
H.ZIndex = 10
H.Parent = M
Instance.new("UICorner", H).CornerRadius = UDim.new(0, 8)

local hb = Instance.new("Frame")
hb.Size = UDim2.new(1, 0, 0, 2)
hb.Position = UDim2.new(0, 0, 1, -2)
hb.BackgroundColor3 = Color3.new(1, 1, 1)
hb.BorderSizePixel = 0
hb.ZIndex = 11
hb.Parent = H

local TL = Instance.new("TextLabel")
TL.Size = UDim2.new(1, -50, 1, 0)
TL.Position = UDim2.new(0, 10, 0, 0)
TL.BackgroundTransparency = 1
TL.Text = "R9ZEN MOBILE"
TL.TextColor3 = Color3.new(1, 1, 1)
TL.Font = Enum.Font.GothamBlack
TL.TextSize = 14
TL.TextXAlignment = Enum.TextXAlignment.Left
TL.ZIndex = 11
TL.Parent = H

local XB = Instance.new("TextButton")
XB.Size = UDim2.new(0, 30, 0, 30)
XB.Position = UDim2.new(1, -38, 0, 5)
XB.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
XB.Text = "X"
XB.TextColor3 = Color3.new(1, 1, 1)
XB.Font = Enum.Font.GothamBold
XB.TextSize = 14
XB.BorderSizePixel = 0
XB.ZIndex = 15
XB.Parent = H
Instance.new("UICorner", XB).CornerRadius = UDim.new(0, 4)
XB.MouseButton1Click:Connect(function() MenuOpen = false M.Visible = false end)

local Stat = Instance.new("TextLabel")
Stat.Size = UDim2.new(1, -16, 0, 14)
Stat.Position = UDim2.new(0, 8, 0, 43)
Stat.BackgroundTransparency = 1
Stat.Text = "Ready"
Stat.TextColor3 = Color3.fromRGB(180, 180, 180)
Stat.Font = Enum.Font.Gotham
Stat.TextSize = 10
Stat.TextXAlignment = Enum.TextXAlignment.Left
Stat.ZIndex = 10
Stat.Parent = M

-- Drag
local mD, mS, mPo = false, nil, nil
H.InputBegan:Connect(function(i) 
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then 
        mD = true mS = i.Position mPo = M.Position 
    end 
end)
H.InputEnded:Connect(function(i) 
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then 
        mD = false 
    end 
end)
UserInputService.InputChanged:Connect(function(i) 
    if mD and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then 
        local d = i.Position - mS 
        M.Position = UDim2.new(mPo.X.Scale, mPo.X.Offset + d.X, mPo.Y.Scale, mPo.Y.Offset + d.Y) 
    end 
end)

Logo.MouseButton1Click:Connect(function()
    MenuOpen = not MenuOpen
    if MenuOpen then 
        M.Visible = true
        TweenService:Create(M, TweenInfo.new(0.15), {Position = UDim2.new(0.5, -130, 0.5, -190)}):Play()
    else
        TweenService:Create(M, TweenInfo.new(0.1), {Position = UDim2.new(0.5, -130, 1.5, 0)}):Play()
        task.wait(0.1) M.Visible = false
    end
end)

-- Scroll (BÜYÜK dokunmatik için)
local SC = Instance.new("ScrollingFrame")
SC.Size = UDim2.new(1, -12, 1, -65)
SC.Position = UDim2.new(0, 6, 0, 60)
SC.BackgroundTransparency = 1
SC.BorderSizePixel = 0
SC.ScrollBarThickness = 5
SC.ScrollBarImageColor3 = Color3.fromRGB(200, 200, 200)
SC.AutomaticCanvasSize = Enum.AutomaticSize.Y
SC.ScrollingEnabled = true
SC.ZIndex = 10
SC.Parent = M
Instance.new("UIListLayout", SC).Padding = UDim.new(0, 5)

-- MOBILE TOGGLE (Büyük butonlar - dokunmatik dostu)
local function Sec(t)
    local f = Instance.new("Frame") 
    f.Size = UDim2.new(1, 0, 0, 22) 
    f.BackgroundColor3 = Color3.fromRGB(35, 35, 35) 
    f.BorderSizePixel = 0 
    f.ZIndex = 11
    f.Parent = SC
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 4)
    local bar = Instance.new("Frame") bar.Size = UDim2.new(0, 3, 1, 0) bar.BackgroundColor3 = Color3.new(1,1,1) bar.BorderSizePixel = 0 bar.ZIndex = 12 bar.Parent = f
    local l = Instance.new("TextLabel") 
    l.Size = UDim2.new(1, -12, 1, 0) 
    l.Position = UDim2.new(0, 10, 0, 0) 
    l.BackgroundTransparency = 1 
    l.Text = t 
    l.TextColor3 = Color3.new(1, 1, 1)
    l.Font = Enum.Font.GothamBold 
    l.TextSize = 11
    l.TextXAlignment = Enum.TextXAlignment.Left 
    l.ZIndex = 12
    l.Parent = f
end

local function Tog(text, def, cb)
    local f = Instance.new("Frame") 
    f.Size = UDim2.new(1, 0, 0, 36) 
    f.BackgroundColor3 = Color3.fromRGB(18, 18, 18) 
    f.BorderSizePixel = 0 
    f.ZIndex = 11
    f.Parent = SC
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 4)
    
    local l = Instance.new("TextLabel") 
    l.Size = UDim2.new(0.65, 0, 1, 0) 
    l.Position = UDim2.new(0, 8, 0, 0) 
    l.BackgroundTransparency = 1 
    l.Text = text 
    l.TextColor3 = Color3.fromRGB(220, 220, 220)
    l.Font = Enum.Font.GothamSemibold 
    l.TextSize = 11
    l.TextXAlignment = Enum.TextXAlignment.Left 
    l.ZIndex = 12
    l.Parent = f
    
    local b = Instance.new("TextButton") 
    b.Size = UDim2.new(0, 60, 0, 26) 
    b.Position = UDim2.new(1, -68, 0.5, -13) 
    b.BackgroundColor3 = def and Color3.new(1,1,1) or Color3.fromRGB(40, 40, 40)
    b.Text = def and "ON" or "OFF" 
    b.TextColor3 = def and Color3.new(0,0,0) or Color3.new(1,1,1)
    b.Font = Enum.Font.GothamBold 
    b.TextSize = 12
    b.BorderSizePixel = 0 
    b.ZIndex = 12
    b.Parent = f
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 4)
    
    local s = def
    b.MouseButton1Click:Connect(function()
        s = not s
        b.Text = s and "ON" or "OFF"
        b.BackgroundColor3 = s and Color3.new(1,1,1) or Color3.fromRGB(40, 40, 40)
        b.TextColor3 = s and Color3.new(0,0,0) or Color3.new(1,1,1)
        cb(s)
    end)
end

local function Sli(text, min, max, def, cb)
    local f = Instance.new("Frame") 
    f.Size = UDim2.new(1, 0, 0, 42) 
    f.BackgroundColor3 = Color3.fromRGB(18, 18, 18) 
    f.BorderSizePixel = 0 
    f.ZIndex = 11
    f.Parent = SC
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 4)
    
    local l = Instance.new("TextLabel") 
    l.Size = UDim2.new(1, -12, 0, 16) 
    l.Position = UDim2.new(0, 8, 0, 3) 
    l.BackgroundTransparency = 1 
    l.Text = text .. ": " .. def 
    l.TextColor3 = Color3.fromRGB(220, 220, 220)
    l.Font = Enum.Font.GothamSemibold 
    l.TextSize = 11
    l.TextXAlignment = Enum.TextXAlignment.Left 
    l.ZIndex = 12
    l.Parent = f
    
    local bg = Instance.new("Frame") 
    bg.Size = UDim2.new(1, -16, 0, 8) 
    bg.Position = UDim2.new(0, 8, 0, 25) 
    bg.BackgroundColor3 = Color3.fromRGB(40, 40, 40) 
    bg.BorderSizePixel = 0 
    bg.ZIndex = 12
    bg.Parent = f
    Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 3)
    
    local fl = Instance.new("Frame") 
    fl.Size = UDim2.new((def-min)/(max-min), 0, 1, 0) 
    fl.BackgroundColor3 = Color3.new(1, 1, 1)
    fl.BorderSizePixel = 0 
    fl.ZIndex = 13
    fl.Parent = bg
    Instance.new("UICorner", fl).CornerRadius = UDim.new(0, 3)
    
    local dr = false
    bg.InputBegan:Connect(function(i) 
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then 
            dr = true 
        end 
    end)
    UserInputService.InputEnded:Connect(function(i) 
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then 
            dr = false 
        end 
    end)
    UserInputService.InputChanged:Connect(function(i) 
        if dr and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local p = math.clamp((i.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
            local v = math.floor(min + (max-min) * p)
            fl.Size = UDim2.new(p, 0, 1, 0)
            l.Text = text .. ": " .. v
            cb(v)
        end 
    end)
end

-- MENU
Sec("AIMBOT")
Tog("Aimbot (Hold Button)", AimbotEnabled, function(v) AimbotEnabled = v end)
Sli("Aim FOV", 50, 500, AimFOV, function(v) AimFOV = v end)

Sec("HITBOX")
Tog("Hitbox Expander", HitboxEnabled, function(v) 
    HitboxEnabled = v 
    if v then ApplyHitbox() else RestoreHitbox() end
end)
Sli("Hitbox Size", 3, 30, HitboxSize, function(v) 
    HitboxSize = v 
    if HitboxEnabled then ApplyHitbox() end
end)

Sec("ESP")
Tog("Player ESP", ESPEnabled, function(v) 
    ESPEnabled = v 
    if not v then RemoveAllESP() end
end)

Sec("MOVEMENT")
Tog("Fly", FlyEnabled, function(v) 
    FlyEnabled = v 
    if v then StartFly() else StopFly() end
end)
Sli("Fly Speed", 20, 200, FlySpeed, function(v) FlySpeed = v end)
Tog("Noclip", NoclipEnabled, function(v) 
    NoclipEnabled = v 
    if v then EnableNoclip() else DisableNoclip() end
end)
Tog("Speed Hack", SpeedOn, function(v) 
    SpeedOn = v 
    if not v then pcall(function() LP.Character.Humanoid.WalkSpeed = 16 end) end
end)
Sli("Speed", 16, 100, SpeedVal, function(v) SpeedVal = v end)

Sec("CAMERA")
Sli("FOV", 30, 120, GameFOV, function(v) GameFOV = v end)

-- =============== MOBILE AIM BUTTON (Sağ alt) ===============
local AimBtn = Instance.new("TextButton")
AimBtn.Size = UDim2.new(0, 80, 0, 80)
AimBtn.Position = UDim2.new(1, -100, 1, -180)
AimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
AimBtn.Text = "AIM"
AimBtn.TextColor3 = Color3.new(1, 1, 1)
AimBtn.Font = Enum.Font.GothamBlack
AimBtn.TextSize = 18
AimBtn.BorderSizePixel = 0
AimBtn.Parent = G
Instance.new("UICorner", AimBtn).CornerRadius = UDim.new(1, 0)
local aimStr = Instance.new("UIStroke", AimBtn)
aimStr.Color = Color3.new(1, 1, 1)
aimStr.Thickness = 2

AimBtn.MouseButton1Down:Connect(function() 
    AimHolding = true 
    aimStr.Color = Color3.fromRGB(255, 50, 50)
end)
AimBtn.MouseButton1Up:Connect(function() 
    AimHolding = false 
    aimStr.Color = Color3.new(1, 1, 1)
end)

-- =============== FLY BUTONLARI (Mobil için) ===============
local FlyGui = Instance.new("Frame")
FlyGui.Size = UDim2.new(0, 200, 0, 200)
FlyGui.Position = UDim2.new(0, 20, 1, -220)
FlyGui.BackgroundTransparency = 1
FlyGui.Parent = G

local function CreateFlyBtn(text, size, pos, key)
    local b = Instance.new("TextButton")
    b.Size = size
    b.Position = pos
    b.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    b.BackgroundTransparency = 0.3
    b.Text = text
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.GothamBold
    b.TextSize = 16
    b.BorderSizePixel = 0
    b.Parent = FlyGui
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
    local str = Instance.new("UIStroke", b) str.Color = Color3.new(1,1,1) str.Thickness = 1
    
    b.MouseButton1Down:Connect(function() flyKeys[key] = true b.BackgroundTransparency = 0 end)
    b.MouseButton1Up:Connect(function() flyKeys[key] = false b.BackgroundTransparency = 0.3 end)
    return b
end

CreateFlyBtn("↑", UDim2.new(0, 50, 0, 50), UDim2.new(0, 55, 0, 0), "W")
CreateFlyBtn("←", UDim2.new(0, 50, 0, 50), UDim2.new(0, 0, 0, 55), "A")
CreateFlyBtn("↓", UDim2.new(0, 50, 0, 50), UDim2.new(0, 55, 0, 55), "S")
CreateFlyBtn("→", UDim2.new(0, 50, 0, 50), UDim2.new(0, 110, 0, 55), "D")
CreateFlyBtn("+", UDim2.new(0, 45, 0, 45), UDim2.new(0, 155, 0, 0), "Up")
CreateFlyBtn("-", UDim2.new(0, 45, 0, 45), UDim2.new(0, 155, 0, 55), "Down")

-- Fly butonlarını göster/gizle
task.spawn(function()
    while true do
        task.wait(0.5)
        FlyGui.Visible = FlyEnabled
    end
end)

-- =============== MAIN LOOP ===============
RunService.RenderStepped:Connect(function()
    Camera = workspace.CurrentCamera
    local myChar = LP.Character
    
    Camera.FieldOfView = GameFOV
    
    -- AIMBOT
    if AimbotEnabled and AimHolding then
        local target = GetTarget()
        if target and target.Character and target.Character:FindFirstChild(AimPart) then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character[AimPart].Position)
        end
    end
    
    -- STATUS
    local s = {}
    table.insert(s, "R9")
    if AimbotEnabled and AimHolding then table.insert(s, "AIM") end
    if HitboxEnabled then table.insert(s, "HB"..HitboxSize) end
    if ESPEnabled then table.insert(s, "ESP") end
    if FlyEnabled then table.insert(s, "FLY") end
    if NoclipEnabled then table.insert(s, "NC") end
    if SpeedOn then table.insert(s, "SP"..SpeedVal) end
    Stat.Text = table.concat(s, "|")
end)

LP.CharacterRemoving:Connect(function() 
    StopFly() 
    RestoreHitbox()
end)

StarterGui:SetCore("SendNotification", {
    Title = "R9ZEN MOBILE",
    Text = "Knife Duel Mobile Version!",
    Duration = 4
})

print("R9ZEN Mobile V1 Loaded!")
print("AIM Button = Sağ alt")
print("Fly Buttons = Sol alt (Fly açıksa)")
print("R9 Logo = Menu aç/kapa")
