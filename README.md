local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")

-- Config
local autoFarmRunning = false
local endZ = -5149.8

-- Notificação
local function notify(txt)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tianta AutoFarm",
            Text = txt,
            Duration = 3
        })
    end)
end

-- Detecta o carro atual
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

-- Simula tecla W
local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- UI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0, 20, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Parent = screenGui

-- Sistema manual de arrastar a janela
local dragging, dragInput, dragStart, startPos

frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Botão
local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1, -20, 0, 40)
button.Position = UDim2.new(0, 10, 0, 30)
button.Text = "AutoFarm OFF"
button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 16

-- Ativar/desativar AutoFarm
local autoDriveThread, teleportThread

button.MouseButton1Click:Connect(function()
    autoFarmRunning = not autoFarmRunning
    button.Text = autoFarmRunning and "AutoFarm ON" or "AutoFarm OFF"

    if autoFarmRunning then
        notify("AutoFarm iniciado")
        local car
        repeat
            car = getCar()
            if not car then
                notify("Entra no carro para iniciar")
                wait(1)
            end
        until car

        local rotationY = car.PrimaryPart.Orientation.Y
        local startCFrame = CFrame.new(18.4, 42.6, -4235.6) * CFrame.Angles(0, math.rad(rotationY), 0)

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
                local car = getCar()
                if car then
                    local pos = car.PrimaryPart.Position
                    if pos.Z <= endZ then
                        notify("Teleportando para início...")
                        local rotY = car.PrimaryPart.Orientation.Y
                        local fixedCFrame = CFrame.new(18.4, 42.6, -4235.6) * CFrame.Angles(0, math.rad(rotY), 0)
                        car:SetPrimaryPartCFrame(fixedCFrame)
                        wait(1)
                    end
                end
                wait(0.1)
            end
        end)
    else
        notify("AutoFarm parado")
        pressW(false)
        if autoDriveThread then task.cancel(autoDriveThread) end
        if teleportThread then task.cancel(teleportThread) end
    end
end)
