local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

local autoFarmRunning = false
local startCFrame = CFrame.new(18.4, 42.6, -4235.6)
local endZ = -5149.8

-- Notificação
local function notify(txt)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tianta AutoFarm",
            Text = txt,
            Duration = 4
        })
    end)
end

-- Detecta o carro
local function getCar()
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.SeatPart then
        local seat = humanoid.SeatPart
        local car = seat:FindFirstAncestorOfClass("Model")
        if car and car.PrimaryPart then
            return car
        end
    end
    return nil
end

-- Pressionar tecla W
local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- Interface
local screenGui = Instance.new("ScreenGui", game.CoreGui)
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0, 20, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1, -20, 0, 40)
button.Position = UDim2.new(0, 10, 0, 30)
button.Text = "AutoFarm OFF"
button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 16

local farmLoop, driveLoop

button.MouseButton1Click:Connect(function()
    autoFarmRunning = not autoFarmRunning
    if autoFarmRunning then
        button.Text = "AutoFarm ON"
        notify("AutoFarm iniciado")

        local car
        repeat
            car = getCar()
            if not car then
                notify("Entra no carro para iniciar o AutoFarm.")
                wait(1)
            end
        until car

        wait(0.5)
        car:SetPrimaryPartCFrame(startCFrame)
        wait(1)

        -- Acelerando
        driveLoop = task.spawn(function()
            while autoFarmRunning do
                pressW(true)
                wait(10)
            end
        end)

        -- Teleporte automático
        farmLoop = task.spawn(function()
            while autoFarmRunning do
                local car = getCar()
                if car then
                    local pos = car.PrimaryPart.Position
                    if pos.Z <= endZ then
                        notify("Voltando ao início...")
                        car:SetPrimaryPartCFrame(startCFrame)
                        wait(1)
                    end
                end
                wait(0.1)
            end
        end)

    else
        -- Desligar
        notify("AutoFarm parado")
        button.Text = "AutoFarm OFF"
        pressW(false)
        if farmLoop then task.cancel(farmLoop) end
        if driveLoop then task.cancel(driveLoop) end
    end
end)
