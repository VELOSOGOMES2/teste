-- Tiânta Autofarm Menu (versão fixa)  
local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

local autoFarm = false
local car = nil
local startRotationY = 0

-- Coordenadas fixas (imagem que você enviou)
local startPos = Vector3.new(18.4, 42.6, -4235.6)
local endPos = Vector3.new(-191.4, 50.2, -5149.8)

-- Notificação
function notify(txt)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tiânta Menu",
            Text = txt,
            Duration = 4
        })
    end)
end

-- Detectar carro
function getCar()
    local character = player.Character
    if not character then return nil end
    local seatPart = character:FindFirstChildWhichIsA("BasePart")
    if not seatPart then return nil end

    local vehicle = seatPart:FindFirstAncestorOfClass("Model")
    if vehicle and vehicle:FindFirstChild("Humanoid") == nil then
        return vehicle
    end
    return nil
end

-- Criar botão
local screenGui = Instance.new("ScreenGui", game.CoreGui)
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0, 0.3, 0, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1, 0, 1, 0)
button.TextSize = 18
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
button.Font = Enum.Font.GothamBold
button.Text = "AutoFarm OFF"

-- AutoFarm Loop
function autoFarmLoop()
    while autoFarm do
        car = getCar()
        if car and car.PrimaryPart then
            -- Salvar rotação original
            local rot = car.PrimaryPart.Orientation.Y
            startRotationY = rot

            -- Teleportar para o início com rotação correta
            car:SetPrimaryPartCFrame(CFrame.new(startPos) * CFrame.Angles(0, math.rad(startRotationY), 0))
            wait(0.5)

            -- Dirigir até o fim
            for i = 1, 150 do
                if not autoFarm then break end
                VirtualInputManager:SendKeyEvent(true, "W", false, game)
                wait(0.03)
                VirtualInputManager:SendKeyEvent(false, "W", false, game)
                wait(0.02)
            end

            -- Teleportar para o fim
            car:SetPrimaryPartCFrame(CFrame.new(endPos) * CFrame.Angles(0, math.rad(startRotationY), 0))
            wait(0.5)
        else
            notify("Entra no carro primeiro!")
        end
        wait(1)
    end
end

-- Botão ON/OFF
button.MouseButton1Click:Connect(function()
    autoFarm = not autoFarm
    button.Text = autoFarm and "AutoFarm ON" or "AutoFarm OFF"
    notify(autoFarm and "AutoFarm Iniciado!" or "AutoFarm Parado.")
    if autoFarm then
        task.spawn(autoFarmLoop)
    end
end)
