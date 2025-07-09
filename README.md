local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

-- 🛡️ Anti-Cheat / Anti-Kick / Anti-Ban
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local oldNamecall = mt.__namecall

    mt.__namecall = newcclosure(function(self, ...)
        local args = {...}
        local method = getnamecallmethod()
        if tostring(self) == "Kick" or method == "Kick" then
            warn("🚫 Tentativa de Kick bloqueada!")
            return nil
        end
        if tostring(self) == "Ban" or method == "Ban" then
            warn("🚫 Tentativa de Ban detectada!")
            return nil
        end
        return oldNamecall(self, unpack(args))
    end)

    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")

    humanoid:GetPropertyChangedSignal("Health"):Connect(function()
        if humanoid.Health <= 0 then
            humanoid.Health = 100
            warn("❤️ Tentaram te matar — Vida restaurada")
        end
    end)

    humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
        if humanoid.WalkSpeed == 0 then
            humanoid.WalkSpeed = 16
            warn("🏃‍♂️ Tentaram travar sua velocidade — Corrigido")
        end
    end)
end)

-- ⚙️ Config
local autoFarmRunning1 = false
local autoFarmRunning2 = false
local startCFrame1 = CFrame.new(-84.2, 42.6, -4175.5) * CFrame.Angles(0, math.rad(90), 0)
local endZ1 = -4001.5

local startCFrame2 = CFrame.new(-192.2, 38.3, -4481.7) * CFrame.Angles(0, math.rad(90), 0)
local endZ2 = -4350.0

local shownMessages = {}

local function notify(txt)
    if shownMessages[txt] then return end
    shownMessages[txt] = true
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "🚗 AutoFarm",
            Text = txt,
            Duration = 3
        })
    end)
end

local function getCar()
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.SeatPart then
        local seat = humanoid.SeatPart
        local car = seat:FindFirstAncestorOfClass("Model")
        if car and car.PrimaryPart then
            return car
        end
    end
    return nil
end

local function isInCar()
    local char = player.Character
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    return humanoid and humanoid.SeatPart ~= nil
end

local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- 🖼️ UI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaFarmUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 250, 0, 180)
mainFrame.Position = UDim2.new(0, 20, 0.4, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local header = Instance.new("TextLabel", mainFrame)
header.Size = UDim2.new(1, 0, 0, 30)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
header.Text = "MOD MENU\nRSJGAMES"
header.TextColor3 = Color3.new(1, 1, 1)
header.Font = Enum.Font.GothamBold
header.TextSize = 14

-- 🖱️ Drag
local dragging = false
local dragStart, startPos
header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
header.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)
header.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- 🔘 Botão AutoFarm 1
local button1 = Instance.new("TextButton", mainFrame)
button1.Size = UDim2.new(1, -20, 0, 40)
button1.Position = UDim2.new(0, 10, 0, 40)
button1.Text = "AutoFarm 1 OFF"
button1.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button1.TextColor3 = Color3.new(1, 1, 1)
button1.Font = Enum.Font.GothamBold
button1.TextSize = 16

-- 🔘 Botão AutoFarm 2
local button2 = Instance.new("TextButton", mainFrame)
button2.Size = UDim2.new(1, -20, 0, 40)
button2.Position = UDim2.new(0, 10, 0, 90)
button2.Text = "AutoFarm 2 OFF"
button2.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button2.TextColor3 = Color3.new(1, 1, 1)
button2.Font = Enum.Font.GothamBold
button2.TextSize = 16

-- 🔁 Threads
local autoDriveThread1, teleportThread1
local autoDriveThread2, teleportThread2

local function stopAutoFarm1(msg)
    autoFarmRunning1 = false
    button1.Text = "AutoFarm 1 OFF"
    pressW(false)
    if autoDriveThread1 then task.cancel(autoDriveThread1) end
    if teleportThread1 then task.cancel(teleportThread1) end
    if msg then notify(msg) end
end

local function stopAutoFarm2(msg)
    autoFarmRunning2 = false
    button2.Text = "AutoFarm 2 OFF"
    pressW(false)
    if autoDriveThread2 then task.cancel(autoDriveThread2) end
    if teleportThread2 then task.cancel(teleportThread2) end
    if msg then notify(msg) end
end

-- 🚗 Iniciar AutoFarm 1
button1.MouseButton1Click:Connect(function()
    autoFarmRunning1 = not autoFarmRunning1
    button1.Text = autoFarmRunning1 and "AutoFarm 1 ON" or "AutoFarm 1 OFF"
    shownMessages = {}

    if autoFarmRunning1 then
        notify("✅ AutoFarm 1 iniciado")
        local car = getCar()
        if not car then notify("❗ Entra no carro para iniciar") end
        repeat car = getCar() wait(1) until car

        wait(0.5)
        car:PivotTo(startCFrame1)
        car.AssemblyLinearVelocity = Vector3.zero
        car.AssemblyAngularVelocity = Vector3.zero
        wait(1)

        autoDriveThread1 = task.spawn(function()
            while autoFarmRunning1 do
                pressW(true)
                wait(10)
            end
        end)

        teleportThread1 = task.spawn(function()
            while autoFarmRunning1 do
                if not isInCar() then return stopAutoFarm1("⛔ Saiu do carro") end
                local car = getCar()
                if not car or not car.Parent then return stopAutoFarm1("🚫 Carro removido") end
                if car.PrimaryPart.Position.Z >= endZ1 then
                    car:PivotTo(startCFrame1)
                    car.AssemblyLinearVelocity = Vector3.zero
                    car.AssemblyAngularVelocity = Vector3.zero
                    wait(1)
                end
                wait(0.1)
            end
        end)
    else
        stopAutoFarm1("🛑 AutoFarm 1 parado")
    end
end)

-- 🚙 Iniciar AutoFarm 2
button2.MouseButton1Click:Connect(function()
    autoFarmRunning2 = not autoFarmRunning2
    button2.Text = autoFarmRunning2 and "AutoFarm 2 ON" or "AutoFarm 2 OFF"
    shownMessages = {}

    if autoFarmRunning2 then
        notify("✅ AutoFarm 2 iniciado")
        local car = getCar()
        if not car then notify("❗ Entra no carro para iniciar") end
        repeat car = getCar() wait(1) until car

        wait(0.5)
        car:PivotTo(startCFrame2)
        car.AssemblyLinearVelocity = Vector3.zero
        car.AssemblyAngularVelocity = Vector3.zero
        wait(1)

        autoDriveThread2 = task.spawn(function()
            while autoFarmRunning2 do
                pressW(true)
                wait(10)
            end
        end)

        teleportThread2 = task.spawn(function()
            while autoFarmRunning2 do
                if not isInCar() then return stopAutoFarm2("⛔ Saiu do carro") end
                local car = getCar()
                if not car or not car.Parent then return stopAutoFarm2("🚫 Carro removido") end
                if car.PrimaryPart.Position.Z >= endZ2 then
                    car:PivotTo(startCFrame2)
                    car.AssemblyLinearVelocity = Vector3.zero
                    car.AssemblyAngularVelocity = Vector3.zero
                    wait(1)
                end
                wait(0.1)
            end
        end)
    else
        stopAutoFarm2("🛑 AutoFarm 2 parado")
    end
end)

-- 🔽 Minimizar
local minimize = Instance.new("TextButton", mainFrame)
minimize.Size = UDim2.new(0, 25, 0, 25)
minimize.Position = UDim2.new(1, -55, 0, 2)
minimize.Text = "-"
minimize.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.Font = Enum.Font.GothamBold
minimize.TextSize = 16
minimize.MouseButton1Click:Connect(function()
    local min = (mainFrame.Size.Y.Offset <= 40)
    button1.Visible = min
    button2.Visible = min
    mainFrame.Size = min and UDim2.new(0, 250, 0, 180) or UDim2.new(0, 250, 0, 35)
end)

-- ❌ Fechar
local close = Instance.new("TextButton", mainFrame)
close.Size = UDim2.new(0, 25, 0, 25)
close.Position = UDim2.new(1, -30, 0, 2)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
close.TextColor3 = Color3.new(1, 1, 1)
close.Font = Enum.Font.GothamBold
close.TextSize = 16
close.MouseButton1Click:Connect(function()
    stopAutoFarm1()
    stopAutoFarm2()
    screenGui:Destroy()
end)
