-- // RinX Aimlock Mobile + Rainbow FOV | Full Feature
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera

local LocalPlayer = Players.LocalPlayer

-- Cài đặt
local FOV = 140
local Smoothing = 0.5
local AimPart = "Head"           -- Head / UpperTorso / HumanoidRootPart
local TeamCheck = true
local Prediction = 0.1           -- Dự đoán chuyển động (mobile lag)
local RainbowSpeed = 2

local AimlockEnabled = false
local Target = nil

-- FOV Circle Rainbow
local Circle = Drawing.new("Circle")
Circle.Thickness = 2.5
Circle.NumSides = 100
Circle.Radius = FOV
Circle.Filled = false
Circle.Transparency = 1
Circle.Visible = true

-- Tạo GUI Toggle cho Mobile
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 150, 0, 60)
ToggleButton.Position = UDim2.new(0, 20, 0, 20)
ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 100)
ToggleButton.Text = "Aimlock: OFF"
ToggleButton.TextColor3 = Color3.new(1,1,1)
ToggleButton.TextScaled = true
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 12)
Corner.Parent = ToggleButton

-- Rainbow Effect
local Hue = 0
RunService.RenderStepped:Connect(function(dt)
    Hue = (Hue + dt * RainbowSpeed) % 1
    Circle.Color = Color3.fromHSV(Hue, 1, 1)
    
    Circle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    Circle.Radius = FOV
end)

-- Toggle Function
ToggleButton.MouseButton1Click:Connect(function()
    AimlockEnabled = not AimlockEnabled
    if AimlockEnabled then
        ToggleButton.Text = "Aimlock: ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
    else
        ToggleButton.Text = "Aimlock: OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 100)
        Target = nil
    end
end)

-- Tìm mục tiêu gần nhất trong FOV
local function GetClosestPlayer()
    local Closest, Distance = nil, math.huge
    local MousePos = UserInputService:GetMouseLocation()

    for _, Player in ipairs(Players:GetPlayers()) do
        if Player \~= LocalPlayer and Player.Character then
            local Char = Player.Character
            local Humanoid = Char:FindFirstChild("Humanoid")
            local Part = Char:FindFirstChild(AimPart)

            if Humanoid and Humanoid.Health > 0 and Part then
                if TeamCheck and Player.Team == LocalPlayer.Team then continue end

                local Pos = Part.Position + (Part.Velocity * Prediction)
                local ScreenPos, OnScreen = Camera:WorldToViewportPoint(Pos)

                if OnScreen then
                    local Dist = (Vector2.new(ScreenPos.X, ScreenPos.Y) - MousePos).Magnitude
                    if Dist < Distance and Dist < FOV then
                        Distance = Dist
                        Closest = Part
                    end
                end
            end
        end
    end
    return Closest
end

-- Main Aimlock Loop
RunService.RenderStepped:Connect(function()
    if AimlockEnabled then
        Target = GetClosestPlayer()
        
        if Target then
            local TargetPos = Camera:WorldToScreenPoint(Target.Position + (Target.Velocity * Prediction))
            local MousePos = UserInputService:GetMouseLocation()
            
            local AimX = (TargetPos.X - MousePos.X) * Smoothing
            local AimY = (TargetPos.Y - MousePos.Y) * Smoothing
            
            mousemoverel(AimX, AimY)
        end
    end
end)

print("✅ RinX Mobile Aimlock + Rainbow FOV Loaded!")
print("Nhấn nút trên màn hình để bật/tắt")
