-- الخدمات
local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting   = game:GetService("Lighting")
local Camera     = workspace.CurrentCamera
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- اللاعب والكاميرا
local lp = Players.LocalPlayer
local cam = workspace.CurrentCamera

-- إعدادات
local aimOn = false
local boostOn = false
local aimbotRange = 100
local predictionEnabled = true  -- تمكين التنبؤ
local bodyPart = "Head"  -- اختيار جزء الجسم: "Head" أو "Torso"
local keybind = Enum.KeyCode.F -- مفتاح تفعيل الأيمبوت (تغيير المفتاح حسب الحاجة)

-- GUI
local gui = Instance.new("ScreenGui")
gui.Parent = lp:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 120)
frame.Position = UDim2.new(0.5, -100, 0.5, -60)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Parent = gui

-- زر Aimbot
local btnAim = Instance.new("TextButton")
btnAim.Size = UDim2.new(0, 180, 0, 40)
btnAim.Position = UDim2.new(0, 10, 0, 10)
btnAim.Text = "Aimbot: OFF"
btnAim.Parent = frame

btnAim.MouseButton1Click:Connect(function()
    aimOn = not aimOn
    btnAim.Text = "Aimbot: " .. (aimOn and "ON" or "OFF")
end)

-- زر FPS Boost
local btnBoost = Instance.new("TextButton")
btnBoost.Size = UDim2.new(0, 180, 0, 40)
btnBoost.Position = UDim2.new(0, 10, 0, 60)
btnBoost.Text = "FPS Boost: OFF"
btnBoost.Parent = frame

btnBoost.MouseButton1Click:Connect(function()
    boostOn = not boostOn
    btnBoost.Text = "FPS Boost: " .. (boostOn and "ON" or "OFF")
    if boostOn then
        for _, o in pairs(workspace:GetDescendants()) do
            if o:IsA("BasePart") then
                o.Material = Enum.Material.SmoothPlastic
                o.Reflectance = 0
            elseif o:IsA("Decal") then
                o.Transparency = 1
            end
        end
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 1e10
    end
end)

-- Aimbot logic (تنبؤ بحركة الهدف)
local function getClosest()
    local best, minD = nil, aimbotRange
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild(bodyPart) then
            local targetPos = p.Character[bodyPart].Position
            if predictionEnabled then
                -- تنبؤ بحركة الهدف (مثال بسيط)
                local velocity = p.Character.HumanoidRootPart.Velocity
                targetPos = targetPos + velocity * 0.1  -- تعديل بسيط حسب سرعة الحركة
            end
            local d = (targetPos - cam.CFrame.Position).Magnitude
            if d < minD then
                minD, best = d, p
            end
        end
    end
    return best
end

-- Wall Check (التحقق من الرؤية)
local function isVisible(target)
    local ray = Ray.new(cam.CFrame.Position, (target.Position - cam.CFrame.Position).unit * aimbotRange)
    local hitPart = workspace:FindPartOnRay(ray, lp.Character, false, true)
    return hitPart == target
end

RunService.RenderStepped:Connect(function()
    if aimOn then
        local t = getClosest()
        if t and t.Character and t.Character:FindFirstChild(bodyPart) then
            local targetPart = t.Character[bodyPart]
            if isVisible(targetPart.Position) then
                cam.CFrame = CFrame.new(cam.CFrame.Position, targetPart.Position)
            end
        end
    end
end)

-- تفعيل الأيمبوت بواسطة مفتاح
game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Keyboard then
        if input.KeyCode == keybind then
            aimOn = not aimOn
            btnAim.Text = "Aimbot: " .. (aimOn and "ON" or "OFF")
        end
    end
end)
