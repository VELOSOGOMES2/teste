-- Tianta AutoFarm Menu (com definicao de pontos)
local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

-- Variáveis
local autoDrive = false
local teleportActive = false
local startPos = nil
local endPos = nil

-- Função de notificação
function notify(txt)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tianta Menu",
            Text = txt,
            Duration = 4
        })
    end)
end

-- UI
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

-- Pressionar ou soltar tecla W
local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- Função para obter carro
local function getCar()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Engine") and obj.Name == player.Name then
            return obj
        end
    end
    return nil
end

-- Botão 1: Definir ponto inicial
createButton("Definir Ponto Inicial", 1, function()
    local car = getCar()
    if car and car.PrimaryPart then
        startPos = car.PrimaryPart.Position
        notify("Ponto inicial definido com sucesso.")
    else
        notify("Entra no carro antes de definir o ponto.")
    end
end)

-- Botão 2: Definir ponto final
createButton("Definir Ponto Final", 2, function()
    local car = getCar()
    if car and car.PrimaryPart then
        endPos = car.PrimaryPart.Position
        notify("Ponto final definido com sucesso.")
    else
        notify("Entra no carro antes de definir o ponto.")
    end
end)

-- Botão 3: Iniciar Auto Drive
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

-- Botão 4: Ativar Teleport
createButton("Ativar Teleport", 4, function()
    if teleportActive then
        teleportActive = false
        notify("Teleport desativado.")
    else
        if not startPos or not endPos then
            notify("Define os dois pontos primeiro.")
            return
        end
        teleportActive = true
        notify("Teleport ativado.")
        spawn(function()
            while teleportActive do
                local car = getCar()
                if car and car.PrimaryPart then
                    local pos = car.PrimaryPart.Position
                    local dist = (pos - endPos).Magnitude
                    if dist < 30 then -- tolerância de distância do final
                        notify("Teleportando de volta ao início.")
                        car:SetPrimaryPartCFrame(CFrame.new(startPos))
                        wait(1)
                    end
                end
                wait(0.5)
            end
        end)
    end
end)
