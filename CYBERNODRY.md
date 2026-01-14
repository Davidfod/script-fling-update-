local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local flingActive = false
local minimized = false
local originalSize = UDim2.new(0, 220, 0, 160)

-- Interface GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CyberNodry_V9_5_PRO"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Main = Instance.new("Frame")
Main.Size = originalSize
Main.Position = UDim2.new(0.5, -110, 0.4, 0)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Main.Active = true
Main.ClipsDescendants = true
Main.Parent = ScreenGui
Instance.new("UICorner", Main)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, -60, 0, 35)
Title.Text = "NODRY V9.5 + ANTI"
Title.TextColor3 = Color3.fromRGB(0, 255, 150)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.Position = UDim2.new(0, 10, 0, 0)

-- Botões de Controle
local ExitBtn = Instance.new("TextButton", Main)
ExitBtn.Size = UDim2.new(0, 25, 0, 25)
ExitBtn.Position = UDim2.new(1, -30, 0, 5)
ExitBtn.Text = "X"
ExitBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
ExitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", ExitBtn)

local MinBtn = Instance.new("TextButton", Main)
MinBtn.Size = UDim2.new(0, 25, 0, 25)
MinBtn.Position = UDim2.new(1, -60, 0, 5)
MinBtn.Text = "-"
MinBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MinBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", MinBtn)

local Content = Instance.new("Frame", Main)
Content.Size = UDim2.new(1, 0, 1, -35)
Content.Position = UDim2.new(0, 0, 0, 35)
Content.BackgroundTransparency = 1

local AttackBtn = Instance.new("TextButton", Content)
AttackBtn.Size = UDim2.new(0, 180, 0, 45)
AttackBtn.Position = UDim2.new(0.5, -90, 0, 10)
AttackBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
AttackBtn.Text = "LIGAR AUTO-FLING"
AttackBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
AttackBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", AttackBtn)

local StopBtn = Instance.new("TextButton", Content)
StopBtn.Size = UDim2.new(0, 180, 0, 35)
StopBtn.Position = UDim2.new(0.5, -90, 0, 65)
StopBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
StopBtn.Text = "PARAR TUDO"
StopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", StopBtn)

--- SISTEMAS DE PROTEÇÃO (ANTI-FLING E NOCLIP) ---

local function resetPhysics()
    local char = LocalPlayer.Character
    if char then
        for _, part in pairs(char:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Velocity = Vector3.new(0, 0, 0)
                part.RotVelocity = Vector3.new(0, 0, 0)
            end
        end
    end
end

RunService.Stepped:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end

    for _, part in pairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            -- NOCLIP: Ativa quando o fling está ligado para evitar morrer em batidas
            if flingActive then
                part.CanCollide = false
            end

            -- ANTI-FLING: Se algo tentar te girar/lançar muito rápido, o script anula a força
            if part.Velocity.Magnitude > 70 or part.RotVelocity.Magnitude > 70 then
                if not flingActive then -- Não anula se for VOCÊ atacando
                    part.Velocity = Vector3.new(0, 0, 0)
                    part.RotVelocity = Vector3.new(0, 0, 0)
                end
            end
        end
    end
end)

--- LÓGICA DE ATAQUE ---

local function startFling()
    if flingActive then return end
    flingActive = true
    AttackBtn.Text = "ATIVADO!"
    AttackBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)

    while flingActive do
        local char = LocalPlayer.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")

        if not hrp or not hum or hum.Health <= 0 then
            LocalPlayer.CharacterAdded:Wait()
            task.wait(1)
            continue
        end

        for _, tPlayer in pairs(Players:GetPlayers()) do
            if not flingActive then break end
            
            if tPlayer ~= LocalPlayer and tPlayer.Character and tPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local tHRP = tPlayer.Character.HumanoidRootPart
                local tHum = tPlayer.Character:FindFirstChildOfClass("Humanoid")
                
                if tHum and tHum.Health > 0 then
                    local bv = Instance.new("BodyVelocity")
                    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                    bv.Velocity = Vector3.new(9e7, 9e7, 9e7)
                    bv.Parent = hrp
                    
                    local bav = Instance.new("BodyAngularVelocity")
                    bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
                    bav.AngularVelocity = Vector3.new(9e7, 9e7, 9e7)
                    bav.Parent = hrp

                    local startAtk = tick()
                    while flingActive and tPlayer.Character and tHRP.Parent and (tick() - startAtk < 1.5) do
                        RunService.Heartbeat:Wait()
                        if hum.Health <= 0 then break end
                        
                        -- Posicionamento estratégico (em cima do alvo)
                        hrp.CFrame = tHRP.CFrame * CFrame.new(0, 0.4, 0)
                        
                        if tHum.Health <= 0 or tHRP.Position.Y < -50 then break end
                    end
                    
                    bv:Destroy()
                    bav:Destroy()
                    resetPhysics()
                    task.wait(0.05)
                end
            end
        end
        task.wait(0.1)
    end
    resetPhysics()
    AttackBtn.Text = "LIGAR AUTO-FLING"
    AttackBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
end

--- EVENTOS ---

AttackBtn.MouseButton1Click:Connect(function() task.spawn(startFling) end)
StopBtn.MouseButton1Click:Connect(function() 
    flingActive = false 
    resetPhysics() 
end)

MinBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        Content.Visible = false
        Main:TweenSize(UDim2.new(0, 220, 0, 35), "Out", "Quart", 0.3, true)
        MinBtn.Text = "+"
    else
        Main:TweenSize(originalSize, "Out", "Quart", 0.3, true)
        task.wait(0.2)
        Content.Visible = true
        MinBtn.Text = "-"
    end
end)

ExitBtn.MouseButton1Click:Connect(function()
    flingActive = false
    resetPhysics()
    ScreenGui:Destroy()
end)

-- Draggable logic
local dragging, dragStart, startP
Main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true dragStart = input.Position startP = Main.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startP.X.Scale, startP.X.Offset + delta.X, startP.Y.Scale, startP.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
