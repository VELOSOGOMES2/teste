local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

-- 🛡️ Proteções Anti-Kick / Anti-Ban / Anti-Cheat
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local old = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if tostring(self) == "Kick" or method == "Kick" then
            warn("🚫 Kick bloqueado")
            return nil
        end
        if tostring(self) == "Ban" or method == "Ban" then
            warn("🚫 Ban detectado")
            return nil
        end
        return old(self, ...)
    end)

    local char = player.Character or player.CharacterAdded:Wait()
    local hum = char:WaitForChild("Humanoid")

    hum:GetPropertyChangedSignal("Health"):Connect(function()
        if hum.Health <= 0 then
            hum.Health = 100
            warn("❤️ Vida restaurada")
        end
    end)

    hum:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
        if hum.WalkSpeed == 0 then
            hum.WalkSpeed = 16
            warn("🏃 WalkSpeed travado - corrigido")
        end
    end)
end)

-- 📍 Coordenadas do AutoFarm V2
local startCFrame = CFrame.new(2978.8, 52.2, -1493.5) * CFrame.Angles(0, math.rad(90), 0)
local endZ = -2094.2
local autoFarmOn = false
local pressedKeys = {}
local shownNotify = {}

-- 🔄 Utilidades
local function notify(msg)
    if shownNotify[msg] then return end
    shownNotify[msg] = true
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "🚗 AutoFarm V2",
            Text = msg,
            Duration = 3
        })
    end)
end

local function pressKey(key, state)
    if pressedKeys[key] == state then return end
    pressedKeys[key] = state
    VirtualInputManager:SendKeyEvent(state, key, false, game)
end

local function getCar()
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum and hum.SeatPart then
        local car = hum.SeatPart:FindFirstAncestorOfClass("Model")
        if car and car.PrimaryPart then
            return car
        end
    end
    return nil
end

local function isInCar()
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    return hum and hum.SeatPart ~= nil
end

-- 🖼️ Interface Gráfica
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "TiantaAutoFarmV2"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 110)
frame.Position = UDim2.new(0, 15, 0.5, -55)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "AutoFarm V2 - RSJGAMES"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.Font = Enum.Font.GothamBold
title.TextSize = 14

local toggle = Instance.new("TextButton", frame)
toggle.Size = UDim2.new(1, -20, 0, 40)
toggle.Position = UDim2.new(0, 10, 0, 40)
toggle.Text = "AutoFarm: OFF"
toggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.Font = Enum.Font.Gotham
toggle.TextSize = 16

local close = Instance.new("TextButton", frame)
close.Size = UDim2.new(0, 25, 0, 25)
close.Position = UDim2.new(1, -30, 0, 2)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
close.TextColor3 = Color3.new(1, 1, 1)
close.Font = Enum.Font.GothamBold
close.TextSize = 16

-- 🖱️ Arrastar menu
local dragging, dragStart, startPos
title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)
title.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)
title.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- 🔁 Threads
local autoThread, teleportThread

local function stopAutoFarm(reason)
    autoFarmOn = false
    toggle.Text = "AutoFarm: OFF"
    pressKey("W", false)
    if autoThread then task.cancel(autoThread) end
    if teleportThread then task.cancel(teleportThread) end
    if reason then notify(reason) end
end

-- ▶️ Início do AutoFarm
toggle.MouseButton1Click:Connect(function()
    autoFarmOn = not autoFarmOn
    toggle.Text = autoFarmOn and "AutoFarm: ON" or "AutoFarm: OFF"
    shownNotify = {}

    if autoFarmOn then
        notify("✅ AutoFarm iniciado")
        local car = getCar()
        if not car then notify("❗ Entra no carro primeiro!") return end
        repeat car = getCar() wait(0.5) until car

        wait(0.5)
        car:SetPrimaryPartCFrame(startCFrame)
        wait(1)

        autoThread = task.spawn(function()
            while autoFarmOn do
                pressKey("W", true)
                wait(10)
            end
        end)

        teleportThread = task.spawn(function()
            while autoFarmOn do
                if not isInCar() then
                    stopAutoFarm("⛔ Saiu do carro")
                    return
                end
                local car = getCar()
                if not car or not car.PrimaryPart then
                    stopAutoFarm("🚫 Carro não encontrado")
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
        stopAutoFarm("🛑 AutoFarm desligado")
    end
end)

-- ❌ Botão fechar
close.MouseButton1Click:Connect(function()
    stopAutoFarm("⛔ Fechado manualmente")
    gui:Destroy()
end)
