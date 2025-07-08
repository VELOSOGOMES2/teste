local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

-- Configuração
local teleportActive = false
local startCFrame = nil
local endPos = nil
local shownMessages = {}

-- Notificação
local function notify(txt)
    if shownMessages[txt] then return end
    shownMessages[txt] = true
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tianta Menu",
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

-- Criar GUI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaTeleportUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 280, 0, 280)
mainFrame.Position = UDim2.new(0, 20, 0.3, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.Active = true
mainFrame.Draggable = true

local header = Instance.new("TextLabel", mainFrame)
header.Size = UDim2.new(1, 0, 0, 30)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
header.Text = "Tianta Teleport Menu"
header.TextColor3 = Color3.new(1, 1, 1)
header.Font = Enum.Font.GothamBold
header.TextSize = 14

-- Arrastar
local dragging, dragStart, startPos
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

-- Minimizar
local minimize = Instance.new("TextButton", mainFrame)
minimize.Size = UDim2.new(0, 25, 0, 25)
minimize.Position = UDim2.new(1, -55, 0, 2)
minimize.Text = "-"
minimize.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.Font = Enum.Font.GothamBold
minimize.TextSize = 16

-- Fechar
local close = Instance.new("TextButton", mainFrame)
close.Size = UDim2.new(0, 25, 0, 25)
close.Position = UDim2.new(1, -30, 0, 2)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
close.TextColor3 = Color3.new(1, 1, 1)
close.Font = Enum.Font.GothamBold
close.TextSize = 16

-- Coords display
local coordsLabel = Instance.new("TextLabel", mainFrame)
coordsLabel.Size = UDim2.new(1, -20, 0, 60)
coordsLabel.Position = UDim2.new(0, 10, 0, 190)
coordsLabel.Text = "Coords:\nInício: --\nFim: --"
coordsLabel.BackgroundTransparency = 1
coordsLabel.TextColor3 = Color3.new(1, 1, 1)
coordsLabel.TextWrapped = true
coordsLabel.Font = Enum.Font.Gotham
coordsLabel.TextSize = 14
coordsLabel.TextXAlignment = Enum.TextXAlignment.Left

local function updateCoordsText()
    local s = startCFrame and string.format("%.1f, %.1f, %.1f", startCFrame.Position.X, startCFrame.Position.Y, startCFrame.Position.Z) or "--"
    local e = endPos and string.format("%.1f, %.1f, %.1f", endPos.X, endPos.Y, endPos.Z) or "--"
    coordsLabel.Text = "Coords:\nInício: " .. s .. "\nFim: " .. e
end

-- Criação de botão
local function createButton(text, order, callback)
    local btn = Instance.new("TextButton", mainFrame)
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, 35 + (order - 1) * 50)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 15
    btn.MouseButton1Click:Connect(callback)
end

-- Botão: Definir Início
createButton("Definir Início", 1, function()
    local car = getCar()
    if car then
        startCFrame = car.PrimaryPart.CFrame
        notify("Ponto inicial definido!")
        updateCoordsText()
    else
        notify("Entra no carro antes.")
    end
end)

-- Botão: Definir Fim
createButton("Definir Fim", 2, function()
    local car = getCar()
    if car then
        endPos = car.PrimaryPart.Position
        notify("Ponto final definido!")
        updateCoordsText()
    else
        notify("Entra no carro antes.")
    end
end)

-- Botão: Teleport
createButton("Ativar Teleport", 3, function()
    if teleportActive then
        teleportActive = false
        notify("Teleport desligado.")
    else
        if not startCFrame or not endPos then
            notify("Define os dois pontos primeiro.")
            return
        end
        teleportActive = true
        notify("Teleport ligado.")
        spawn(function()
            while teleportActive do
                local car = getCar()
                if car and car.PrimaryPart.Position.Z >= endPos.Z then
                    car:SetPrimaryPartCFrame(startCFrame)
                    wait(1)
                end
                wait(0.1)
            end
        end)
    end
end)

-- Minimizar e Fechar
minimize.MouseButton1Click:Connect(function()
    local collapsed = mainFrame.Size.Y.Offset <= 35
    coordsLabel.Visible = collapsed
    for _, obj in pairs(mainFrame:GetChildren()) do
        if obj:IsA("TextButton") and obj ~= close and obj ~= minimize then
            obj.Visible = collapsed
        end
    end
    mainFrame.Size = collapsed and UDim2.new(0, 280, 0, 280) or UDim2.new(0, 280, 0, 35)
end)

close.MouseButton1Click:Connect(function()
    teleportActive = false
    screenGui:Destroy()
end)
