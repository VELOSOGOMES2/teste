local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

-- 🛡️ Anti-Cheat + Anti-Ban
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

-- ⚙️ Configuração
local autoFarmRunning1 = false
local autoFarmRunning2 = false
local shownMessages = {}

-- Coords V1
local startCFrame1 = CFrame.new(-84.2, 42.6, -4175.5)
local endZ1 = -4001.5

-- Coords V2
local startCFrame2 = CFrame.new(2978.8, 52.2, -1493.5)
local endZ2 = -2094.2

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

-- 🚘 Detecta o carro atual
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

-- 🧍‍♂️ Verifica se o jogador está no carro
local function isInCar()
    local char = player.Character
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    return humanoid and humanoid.SeatPart ~= nil
end

-- 🎮 Simula tecla W
local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- 🖼️ UI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaFarmUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 250, 0, 170)
mainFrame.Position = UDim2.new(0, 20, 0.4, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local header = Instance.new("TextLabel", mainFrame)
header.Size = UDim2.new(1, 0, 0, 30)
header.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
header.Text = "MOD MENU\nRSJGAMES"
header.TextColor3 = Color3.new(1, 1, 1)
header.Font = Enum.Font.GothamBold
header.TextSize = 14

-- 🖱️ Arrastar
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

-- 🔘 Botões
local button1 = Instance.new("TextButton", mainFrame)
button1.Size = UDim2.new(1, -20, 0, 30)
button1.Position = UDim2.new(0, 10, 0, 40)
button1.Text = "AutoFarm 1 OFF"
button1.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button1.TextColor3 = Color3.new(1, 1, 1)
button1.Font = Enum.Font.GothamBold
button1.TextSize = 16

local button2 = Instance.new("TextButton", mainFrame)
button2.Size = UDim2.new(1, -20, 0, 30)
button2.Position = UDim2.new(0, 10, 0, 80)
button2.Text = "AutoFarm 2 OFF"
button2.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button2.TextColor3 = Color3.new(1, 1, 1)
button2.Font = Enum.Font.GothamBold
button2.TextSize = 16

local close = Instance.new("TextButton", mainFrame)
close.Size = UDim2.new(0, 25, 0, 25)
close.Position = UDim2.new(1, -30, 0, 2)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
close.TextColor3 = Color3.new(1, 1, 1)
close.Font = Enum.Font.GothamBold
close.TextSize = 16

-- Threads
local driveThread1, teleportThread1, driveThread2, teleportThread2

local function stopFarm1()
    autoFarmRunning1 = false
    button1.Text = "AutoFarm 1 OFF"
    pressW(false)
    if driveThread1 then task.cancel(driveThread1) end
    if teleportThread1 then task.cancel(teleportThread1) end
end

local function stopFarm2()
    autoFarmRunning2 = false
    button2.Text = "AutoFarm 2 OFF"
    pressW(false)
    if driveThread2 then task.cancel(driveThread2) end
    if teleportThread2 then task.cancel(teleportThread2) end
end

-- AutoFarm 1
button1.MouseButton1Click:Connect(function()
    autoFarmRunning1 = not autoFarmRunning1
    button1.Text = autoFarmRunning1 and "AutoFarm 1 ON" or "AutoFarm 1 OFF"
    shownMessages = {}
    if autoFarmRunning1 then
        stopFarm2()
        notify("✅ AutoFarm 1 iniciado")
        repeat wait() until getCar()
        wait(0.5)
        getCar():SetPrimaryPartCFrame(startCFrame1)
        wait(1)
        driveThread1 = task.spawn(function()
            while autoFarmRunning1 do
                pressW(true)
                wait(10)
            end
        end)
        teleportThread1 = task.spawn(function()
            while autoFarmRunning1 do
                if not isInCar() or not getCar() then
                    stopFarm1()
                    notify("❌ AutoFarm 1 interrompido")
                    return
                end
                if getCar().PrimaryPart.Position.Z >= endZ1 then
                    getCar():SetPrimaryPartCFrame(startCFrame1)
                    wait(1)
                end
                wait(0.1)
            end
        end)
    else
        stopFarm1()
    end
end)

-- AutoFarm 2
button2.MouseButton1Click:Connect(function()
    autoFarmRunning2 = not autoFarmRunning2
    button2.Text = autoFarmRunning2 and "AutoFarm 2 ON" or "AutoFarm 2 OFF"
    shownMessages = {}
    if autoFarmRunning2 then
        stopFarm1()
        notify("✅ AutoFarm 2 iniciado")
        repeat wait() until getCar()
        wait(0.5)
        getCar():SetPrimaryPartCFrame(startCFrame2)
        wait(1)
        driveThread2 = task.spawn(function()
            while autoFarmRunning2 do
                pressW(true)
                wait(10)
            end
        end)
        teleportThread2 = task.spawn(function()
            while autoFarmRunning2 do
                if not isInCar() or not getCar() then
                    stopFarm2()
                    notify("❌ AutoFarm 2 interrompido")
                    return
                end
                if getCar().PrimaryPart.Position.Z <= endZ2 then
                    getCar():SetPrimaryPartCFrame(startCFrame2)
                    wait(1)
                end
                wait(0.1)
            end
        end)
    else
        stopFarm2()
    end
end)

-- Fechar
close.MouseButton1Click:Connect(function()
    stopFarm1()
    stopFarm2()
    mainFrame:Destroy()
    screenGui:Destroy()
end)
