-- Tianta AutoFarm Menu (teleport por posição Z)
local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

-- Variáveis
local autoDrive = false
local teleportActive = false
local startPos = nil
local endPos = nil

-- Notificação
function notify(txt)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tianta Menu",
            Text = txt,
            Duration = 4
        })
    end)
end

-- Detecção de carro via assento
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

-- Criar interface
local screenGui = Instance.new("ScreenGui", game.CoreGui)
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 220, 0, 260)
frame.Position = UDim2.new(0, 10, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local function createButton(text, order, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, 10 + (order - 1) * 50)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.MouseButton1Click:Connect(callback)
end

-- Botões
createButton("Definir Ponto Inicial", 1, function()
    local car = getCar()
    if car then
        startPos = car.PrimaryPart.Position
        notify("Ponto inicial definido com sucesso.")
    else
        notify("Entra no carro antes de definir o ponto.")
    end
end)

createButton("Definir Ponto Final", 2, function()
    local car = getCar()
    if car then
        endPos = car.PrimaryPart.Position
        notify("Ponto final definido com sucesso.")
    else
        notify("Entra no carro antes de definir o ponto.")
    end
end)

createButton("Iniciar Auto Drive", 3, function()
    if autoDrive then
        autoDrive = false
        pressW(false)
        notify("Auto Drive desativado.")
    else
        autoDrive = true
        notify("Auto Drive ativado.")
        spawn(function()
            while autoDrive do
                pressW(true)
                wait(10)
            end
        end)
    end
end)

createButton("Ativar Teleport", 4, function()
    if teleportActive then
        teleportActive = false
        notify("Teleport desativado.")
    else
        if not startPos or not endPos then
            notify("Defina os dois pontos primeiro.")
            return
        end
        teleportActive = true
        notify("Teleport ativado.")
        spawn(function()
            while teleportActive do
                local car = getCar()
                if car then
                    local pos = car.PrimaryPart.Position
                    if pos.Z >= endPos.Z then
                        notify("Teleportando para o início.")
                        car:SetPrimaryPartCFrame(CFrame.new(startPos))
                        wait(1)
                    end
                end
                wait(0.1)
            end
        end)
    end
end)
