local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

-- Configuração
local autoFarmRunning = false
local startCFrame = CFrame.new(18.4, 42.6, -4235.6) * CFrame.Angles(0, math.rad(0), 0)
local endZ = -5149.8
local shownMessages = {}

-- Notificação
local function notify(txt)
    if shownMessages[txt] then return end
    shownMessages[txt] = true
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tianta AutoFarm",
            Text = txt,
            Duration = 3
        })
    end)
end

-- Pegar carro atual
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

-- Simular tecla W
local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- Criar UI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaFarmUI"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 250, 0, 130)
frame.Position = UDim2.new(0, 20, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Active = true
frame.Draggable = true

-- Botão principal
local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1, -20, 0, 40)
button.Position = UDim2.new(0, 10, 0, 40)
button.Text = "AutoFarm OFF"
button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 16

-- Botão minimizar
local minimize = Instance.new("TextButton", frame)
minimize.Size = UDim2.new(0, 25, 0, 25)
minimize.Position = UDim2.new(1, -55, 0, 5)
minimize.Text = "-"
minimize.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.Font = Enum.Font.GothamBold
minimize.TextSize = 16

-- Botão fechar
local close = Instance.new("TextButton", frame)
close.Size = UDim2.new(0, 25, 0, 25)
close.Position = UDim2.new(1, -30, 0, 5)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
close.TextColor3 = Color3.new(1, 1, 1)
close.Font = Enum.Font.GothamBold
close.TextSize = 16

-- Threads
local autoDriveThread, teleportThread

-- Botão ON/OFF
button.MouseButton1Click:Connect(function()
    autoFarmRunning = not autoFarmRunning
    button.Text = autoFarmRunning and "AutoFarm ON" or "AutoFarm OFF"
    shownMessages = {}

    if autoFarmRunning then
        notify("AutoFarm iniciado")

        local car = getCar()
        if not car then
            notify("Entra no carro para iniciar")
        end

        repeat
            car = getCar()
            wait(1)
        until car

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
                        car:SetPrimaryPartCFrame(startCFrame)
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

-- Minimizar/Restaurar
local minimized = false
minimize.MouseButton1Click:Connect(function()
    minimized = not minimized
    button.Visible = not minimized
    frame.Size = minimized and UDim2.new(0, 250, 0, 35) or UDim2.new(0, 250, 0, 130)
end)

-- Fechar completamente
close.MouseButton1Click:Connect(function()
    autoFarmRunning = false
    pressW(false)
    if autoDriveThread then task.cancel(autoDriveThread) end
    if teleportThread then task.cancel(teleportThread) end
    frame:Destroy()
    screenGui:Destroy()
end)
