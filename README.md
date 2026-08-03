local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

pcall(function()
    if game.CoreGui:FindFirstChild("EggHub") then
        game.CoreGui.EggHub:Destroy()
    end
end)

-- Fire Remote
local function fireZap(id)
    pcall(function()
        local Event = ReplicatedStorage:WaitForChild("ZAP"):WaitForChild("ZAP_RELIABLE")
        local bytes = { id }
        local b = buffer.create(#bytes)
        for i = 1, #bytes do
            buffer.writeu8(b, i - 1, bytes[i])
        end
        Event:FireServer(b, {})
    end)
end

-- Cari semua egg
local function getEggs()
    local eggs = {}
    for _, obj in pairs(workspace:GetDescendants()) do
        local name = string.lower(obj.Name)
        if (obj:IsA("Model") or obj:IsA("BasePart")) and 
           (name:find("egg") or name:find("lucky") or name:find("brainrot") or name:find("electric")) then
            local part = obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart")) or obj
            if part then
                table.insert(eggs, {Obj = obj, Part = part, Name = obj.Name})
            end
        end
    end
    return eggs
end

-- Teleport ke egg terdekat / bagus
local function teleportToBestEgg()
    local myChar = LocalPlayer.Character
    if not myChar then return end
    local root = myChar:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local closest = nil
    local shortest = math.huge

    for _, egg in pairs(getEggs()) do
        local dist = (egg.Part.Position - root.Position).Magnitude
        if dist < shortest then
            closest = egg
            shortest = dist
        end
    end

    if closest then
        root.CFrame = CFrame.new(closest.Part.Position + Vector3.new(0, 3, 0))
        print("Teleport ke:", closest.Name)
    end
end

-- Instant Interact (fire proximityprompt / clickdetector)
local function instantInteract()
    local myChar = LocalPlayer.Character
    if not myChar then return end
    local root = myChar:FindFirstChild("HumanoidRootPart")
    if not root then return end

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local parent = obj.Parent
            if parent and parent:IsA("BasePart") then
                local dist = (parent.Position - root.Position).Magnitude
                if dist <= 20 then
                    pcall(function()
                        fireproximityprompt(obj)
                    end)
                end
            end
        elseif obj:IsA("ClickDetector") then
            local parent = obj.Parent
            if parent and parent:IsA("BasePart") then
                local dist = (parent.Position - root.Position).Magnitude
                if dist <= 20 then
                    pcall(function()
                        fireclickdetector(obj)
                    end)
                end
            end
        end
    end
end

-- ==================== GUI ====================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "EggHub"
ScreenGui.Parent = game.CoreGui

-- Tombol Open/Close
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 48, 0, 48)
ToggleBtn.Position = UDim2.new(0, 12, 0.4, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
ToggleBtn.Text = "🥚"
ToggleBtn.TextSize = 22
ToggleBtn.Parent = ScreenGui

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(1, 0)
toggleCorner.Parent = ToggleBtn

local toggleStroke = Instance.new("UIStroke")
toggleStroke.Color = Color3.fromRGB(80, 80, 90)
toggleStroke.Thickness = 1.5
toggleStroke.Parent = ToggleBtn

-- Main GUI
local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 210, 0, 250)
Main.Position = UDim2.new(0.5, -105, 0.12, 0)
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
Main.Visible = true
Main.Parent = ScreenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = Main

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 28)
Title.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
Title.Text = "  Find the Egg"
Title.TextColor3 = Color3.fromRGB(230, 230, 240)
Title.TextSize = 13
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Main

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 10)
titleCorner.Parent = Title

-- Buttons
local function createBtn(text, y, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 180, 0, 34)
    btn.Position = UDim2.new(0.5, -90, 0, y)
    btn.BackgroundColor3 = color or Color3.fromRGB(35, 35, 42)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 12
    btn.Font = Enum.Font.GothamBold
    btn.Parent = Main
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 7)
    c.Parent = btn
    return btn
end

local FireBtn = createBtn("Fire Remote (30)", 38, Color3.fromRGB(40, 80, 120))
local AutoBtn = createBtn("Auto Fire: OFF", 78)
local TPBtn = createBtn("Teleport ke Egg", 118, Color3.fromRGB(80, 50, 120))
local InteractBtn = createBtn("Instant Interact: OFF", 158)
local Status = Instance.new("TextLabel")
Status.Size = UDim2.new(1, -20, 0, 20)
Status.Position = UDim2.new(0, 10, 0, 205)
Status.BackgroundTransparency = 1
Status.Text = "Status: Siap"
Status.TextColor3 = Color3.fromRGB(100, 200, 120)
Status.TextSize = 12
Status.Font = Enum.Font.Gotham
Status.Parent = Main

local AutoFire = false
local InstantOn = false

-- Open/Close
ToggleBtn.MouseButton1Click:Connect(function()
    Main.Visible = not Main.Visible
end)

FireBtn.MouseButton1Click:Connect(function()
    fireZap(30)
    Status.Text = "Status: Fired!"
    task.delay(0.5, function() Status.Text = "Status: Siap" end)
end)

AutoBtn.MouseButton1Click:Connect(function()
    AutoFire = not AutoFire
    AutoBtn.Text = AutoFire and "Auto Fire: ON" or "Auto Fire: OFF"
    AutoBtn.BackgroundColor3 = AutoFire and Color3.fromRGB(40, 100, 50) or Color3.fromRGB(35, 35, 42)
    Status.Text = AutoFire and "Status: Auto Fire KENCENG" or "Status: Siap"
end)

TPBtn.MouseButton1Click:Connect(function()
    teleportToBestEgg()
    Status.Text = "Status: Teleported!"
    task.delay(0.8, function() Status.Text = "Status: Siap" end)
end)

InteractBtn.MouseButton1Click:Connect(function()
    InstantOn = not InstantOn
    InteractBtn.Text = InstantOn and "Instant Interact: ON" or "Instant Interact: OFF"
    InteractBtn.BackgroundColor3 = InstantOn and Color3.fromRGB(40, 100, 50) or Color3.fromRGB(35, 35, 42)
    Status.Text = InstantOn and "Status: Instant Interact aktif" or "Status: Siap"
end)

-- Auto Fire Loop (kenceng)
spawn(function()
    while true do
        if AutoFire then
            fireZap(30)
            fireZap(30)
            fireZap(30)
        end
        task.wait(0.05)
    end
end)

-- Instant Interact Loop
spawn(function()
    while true do
        if InstantOn then
            instantInteract()
        end
        task.wait(0.2)
    end
end)

print("Egg Hub lengkap loaded")
