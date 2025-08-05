CONFIG
local aimbotKey = "Q" -- Key to toggle aimbot (change if needed)
local highlightColor = Color3.fromRGB(255,0,0) -- Red

-- Variables
local aimbotEnabled = false
local uis = game:GetService("UserInputService")
local players = game:GetService("Players")
local lplr = players.LocalPlayer
local mouse = lplr:GetMouse()
local camera = workspace.CurrentCamera

-- GUI
local scrGui = Instance.new("ScreenGui", game.CoreGui)
scrGui.Name = "AimbotMenu"
local frame = Instance.new("Frame", scrGui)
frame.Size = UDim2.new(0,200,0,70)
frame.Position = UDim2.new(0,20,0,100)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Text = "Aimbot Menu"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

local toggleBtn = Instance.new("TextButton", frame)
toggleBtn.Size = UDim2.new(1, -20, 0, 30)
toggleBtn.Position = UDim2.new(0, 10, 0, 35)
toggleBtn.Text = "Aimbot [OFF]"
toggleBtn.TextColor3 = Color3.new(1,1,1)
toggleBtn.BackgroundColor3 = Color3.fromRGB(80,0,0)
toggleBtn.Font = Enum.Font.SourceSans
toggleBtn.TextSize = 18

toggleBtn.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    if aimbotEnabled then
        toggleBtn.Text = "Aimbot [ON]"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(0,80,0)
    else
        toggleBtn.Text = "Aimbot [OFF]"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(80,0,0)
    end
end)

-- Toggle with keyboard
uis.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode[aimbotKey] then
        toggleBtn:MouseButton1Click()
    end
end)

-- Highlights
local function highlightEnemy(char)
    if char:FindFirstChild("AimbotHL") then return end
    local hl = Instance.new("Highlight")
    hl.Name = "AimbotHL"
    hl.FillColor = highlightColor
    hl.OutlineColor = Color3.fromRGB(0,0,0)
    hl.FillTransparency = 0.4
    hl.OutlineTransparency = 0.2
    hl.Parent = char
    hl.Adornee = char
end

local function removeHighlight(char)
    local hl = char:FindFirstChild("AimbotHL")
    if hl then hl:Destroy() end
end

local function highlightAllEnemies()
    for _, plr in ipairs(players:GetPlayers()) do
        if plr ~= lplr and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local isEnemy = true
            if plr.Team and lplr.Team and plr.Team == lplr.Team then
                isEnemy = false
            end
            if isEnemy then
                highlightEnemy(plr.Character)
            else
                removeHighlight(plr.Character)
            end
        end
    end
end

players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char)
        wait(1)
        highlightAllEnemies()
    end)
end)

game:GetService("RunService").RenderStepped:Connect(highlightAllEnemies)

-- Closest enemy function
local function getClosestEnemy()
    local closest, dist = nil, math.huge
    for _, plr in ipairs(players:GetPlayers()) do
        if plr ~= lplr and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local isEnemy = true
            if plr.Team and lplr.Team and plr.Team == lplr.Team then isEnemy = false end
            if isEnemy then
                local pos = camera:WorldToViewportPoint(plr.Character.HumanoidRootPart.Position)
                local mag = (Vector2.new(pos.X,pos.Y) - Vector2.new(mouse.X,mouse.Y)).Magnitude
                if mag < dist and pos.Z > 0 then
                    closest, dist = plr, mag
                end
            end
        end
    end
    return closest
end

-- Aimbot logic
game:GetService("RunService").RenderStepped:Connect(function()
    if not aimbotEnabled then return end
    local target = getClosestEnemy()
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        local part = target.Character.HumanoidRootPart
        camera.CFrame = CFrame.new(camera.CFrame.Position, part.Position)
    end
end)
