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

-- ⚙️ Configurações
local autoFarmRunning = false
local startCFrame = CFrame.new(1920.9, 30.8, -1610.7) * CFrame.Angles(0, math.rad(0), 0)
local endZ = -2598.0
local shownMessages = {}

-- 🔔 Notificação
local function notify(txt)
    if shownMessages[txt] then return end
    shownMessages[txt] = true
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "🚗 Tianta AutoFarm",
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
mainFrame.Size = UDim2.new(0, 250, 0, 130)
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

-- 🖱️ Arrastar menu
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
local button = Instance.new("TextButton", mainFrame)
button.Size = UDim2.new(1, -20, 0, 40)
button.Position = UDim2.new(0, 10, 0, 40)
button.Text = "AutoFarm OFF"
button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 16

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

local function stopAutoFarm(reason)
    autoFarmRunning = false
    button.Text = "AutoFarm OFF"
    pressW(false)
    if autoDriveThread then task.cancel(autoDriveThread) end
    if teleportThread then task.cancel(teleportThread) end
    if reason then notify(reason) end
end

-- 🔄 Sistema de AutoFarm
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

                if car.PrimaryPart.Position.Z <= endZ then
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

-- 🔽 Minimizar
minimize.MouseButton1Click:Connect(function()
    local min = (mainFrame.Size.Y.Offset <= 40)
    button.Visible = min
    mainFrame.Size = min and UDim2.new(0, 250, 0, 130) or UDim2.new(0, 250, 0, 35)
end)

-- ❌ Fechar
close.MouseButton1Click:Connect(function()
    stopAutoFarm()
    mainFrame:Destroy()
    screenGui:Destroy()
end)
