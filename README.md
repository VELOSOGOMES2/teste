-- Tianta AutoFarm Menu (com coords visíveis)
local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

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

-- Detecção via assento
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

-- Criar GUI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 250, 0, 340)
frame.Position = UDim2.new(0, 10, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

-- Textos de coordenadas
local coordsLabel = Instance.new("TextLabel", frame)
coordsLabel.Size = UDim2.new(1, -20, 0, 80)
coordsLabel.Position = UDim2.new(0, 10, 0, 210)
coordsLabel.Text = "📍 Coords:\nInício: --\nFim: --"
coordsLabel.BackgroundTransparency = 1
coordsLabel.TextColor3 = Color3.new(1, 1, 1)
coordsLabel.TextWrapped = true
coordsLabel.Font = Enum.Font.Gotham
coordsLabel.TextSize = 14
coordsLabel.TextXAlignment = Enum.TextXAlignment.Left

local function updateCoordsText()
    local s = startPos and string.format("%.1f, %.1f, %.1f", startPos.X, startPos.Y, startPos.Z) or "--"
    local e = endPos and string.format("%.1f, %.1f, %.1f", endPos.X, endPos.Y, endPos.Z) or "--"
    coordsLabel.Text = "Coords:\nInício: " .. s .. "\nFim: " .. e
end

-- Criar botão
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
        notify("Ponto inicial definido.")
        updateCoordsText()
    else
        notify("Entra no carro antes.")
    end
end)

createButton("Definir Ponto Final", 2, function()
    local car = getCar()
    if car then
        endPos = car.PrimaryPart.Position
        notify("Ponto final definido.")
        updateCoordsText()
    else
        notify("Entra no carro antes.")
    end
end)

createButton("Iniciar Auto Drive", 3, function()
    if autoDrive then
        autoDrive = false
        pressW(false)
        notify("Auto Drive parado.")
    else
        autoDrive = true
        notify("Auto Drive ligado.")
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
        notify("Teleport desligado.")
    else
        if not startPos or not endPos then
            notify("Define os pontos primeiro.")
            return
        end
        teleportActive = true
        notify("Teleport ligado.")
        spawn(function()
            while teleportActive do
                local car = getCar()
                if car then
                    local pos = car.PrimaryPart.Position
                    if pos.Z >= endPos.Z then
                        notify("Teleportando...")
                        car:SetPrimaryPartCFrame(CFrame.new(startPos))
                        wait(1)
                    end
                end
                wait(0.1)
            end
        end)
    end
end)
