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
local autoFarmRunning = false
local startCFrame = CFrame.new(-84.2, 42.6, -4175.5) * CFrame.Angles(0, math.rad(90), 0)
local endZ = -4001.5
local shownMessages = {}

-- 🔔 Notificação
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

-- 🚘 Detectar carro
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

-- 🧍‍♂️ Verifica se está no carro
local function isInCar()
    local char = player.Character
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    return humanoid and humanoid.SeatPart ~= nil
end

-- 🎮 Simula W
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
local button = Instance.new("TextButton", mainFrame)
button.Size = UDim2.new(1, -20, 0, 40)
button.Position = UDim2.new(0, 10, 0, 40)
button.Text = "AutoFarm OFF"
button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 16

-- 🔘 Botão AutoFarm V2
local button2 = Instance.new("TextButton", mainFrame)
button2.Size = UDim2.new(1, -20, 0, 40)
button2.Position = UDim2.new(0, 10, 0, 85)
button2.Text = "AutoFarm V2 OFF"
button2.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button2.TextColor3 = Color3.new(1, 1, 1)
button2.Font = Enum.Font.GothamBold
button2.TextSize = 16

-- 🔘 Minimizar e Fechar
local minimize = Instance.new("TextButton", mainFrame)
minimize.Size = UDim2.new(0, 25, 0, 25)
minimize.Position = UDim2.new(1, -55, 0, 2)
minimize.Text = "-"
minimize.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.Font = Enum.Font.GothamBold
minimize.TextSize = 16

local close = Instance.new("TextButton", mainFrame)
close.Size = UDim2.new(0, 25, 0, 25)
close.Position = UDim2.new(1, -30, 0, 2)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
close.TextColor3 = Color3.new(1, 1, 1)
close.Font = Enum.Font.GothamBold
close.TextSize = 16

-- 🔁 Threads
local autoDriveThread, teleportThread
local autoFarm2Running = false
local autoDriveThread2, teleportThread2

-- ⛔ Funções parar AutoFarm
local function stopAutoFarm(reason)
    autoFarmRunning = false
    button.Text = "AutoFarm OFF"
    pressW(false)
    if autoDriveThread then task.cancel(autoDriveThread) end
    if teleportThread then task.cancel(teleportThread) end
    if reason then notify(reason) end
end

local function stopAutoFarm2(reason)
    autoFarm2Running = false
    button2.Text = "AutoFarm V2 OFF"
    pressW(false)
    if autoDriveThread2 then task.cancel(autoDriveThread2) end
    if teleportThread2 then task.cancel(teleportThread2) end
    if reason then notify(reason) end
end

-- ▶️ AutoFarm 1
button.MouseButton1Click:Connect(function()
    autoFarmRunning = not autoFarmRunning
    button.Text = autoFarmRunning and "AutoFarm ON" or "AutoFarm OFF"
    shownMessages = {}

    if autoFarmRunning then
        notify("✅ AutoFarm iniciado")
        local car = getCar()
        if not car then notify("❗ Entra no carro para iniciar") end
        repeat car = getCar() wait(1) until car

        wait(0.5)
        car:SetPrimaryPartCFrame(startCFrame)
        wait(1)

        autoDriveThread = task.spawn(function()
            while autoFarmRunning do
                pressW(true)
                wait(10)
            end
        end)

        teleportThread = task.spawn(function()
            while autoFarmRunning do
                if not isInCar() then
                    stopAutoFarm("⛔ Saiu do carro, AutoFarm desligado")
                    return
                end

                local car = getCar()
                if not car or not car.Parent then
                    stopAutoFarm("🚫 Carro removido, AutoFarm desligado")
                    return
                end

                if car.PrimaryPart.Position.Z >= endZ then
                    car:SetPrimaryPartCFrame(startCFrame)
                    wait(1)
                end
                wait(0.1)
            end
        end)

    else
        stopAutoFarm("🛑 AutoFarm parado")
    end
end)

-- ▶️ AutoFarm V2
local startCFrame2 = CFrame.new(2930.6, -16.3, -4933.2) * CFrame.Angles(0, math.rad(0), 0)
local endZ2 = -5038.8


button2.MouseButton1Click:Connect(function()
    autoFarm2Running = not autoFarm2Running
    button2.Text = autoFarm2Running and "AutoFarm V2 ON" or "AutoFarm V2 OFF"
    shownMessages = {}

    if autoFarm2Running then
        notify("✅ AutoFarm V2 iniciado")
        local car = getCar()
        if not car then notify("❗ Entra no carro para iniciar") end
        repeat car = getCar() wait(1) until car

        wait(0.5)
        car:SetPrimaryPartCFrame(startCFrame2)
        wait(1)

        autoDriveThread2 = task.spawn(function()
            while autoFarm2Running do
                pressW(true)
                wait(10)
            end
        end)

        teleportThread2 = task.spawn(function()
            while autoFarm2Running do
                if not isInCar() then
                    stopAutoFarm2("⛔ Saiu do carro, AutoFarm V2 desligado")
                    return
                end

                local car = getCar()
                if not car or not car.Parent then
                    stopAutoFarm2("🚫 Carro removido, AutoFarm V2 desligado")
                    return
                end

                if car.PrimaryPart.Position.Z <= endZ2 then
                    car:SetPrimaryPartCFrame(startCFrame2)
                    wait(1)
                end
                wait(0.1)
            end
        end)

    else
        stopAutoFarm2("🛑 AutoFarm V2 parado")
    end
end)

-- 🔽 Minimizar
minimize.MouseButton1Click:Connect(function()
    local min = (mainFrame.Size.Y.Offset <= 40)
    button.Visible = min
    button2.Visible = min
    mainFrame.Size = min and UDim2.new(0, 250, 0, 180) or UDim2.new(0, 250, 0, 35)
end)

-- ❌ Fechar
close.MouseButton1Click:Connect(function()
    stopAutoFarm()
    stopAutoFarm2()
    mainFrame:Destroy()
    screenGui:Destroy()
end)
