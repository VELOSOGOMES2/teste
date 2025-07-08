
local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

local autoDrive = false
local teleportActive = false
local startPos = Vector3.new(0, 0, 0)
local endZ = 3000 -- você pode ajustar aqui


function notify(txt)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tiânta Menu ",
            Text = txt,
            Duration = 4
        })
    end)
end


local screenGui = Instance.new("ScreenGui", game.CoreGui)
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 200, 0, 220)
frame.Position = UDim2.new(0, 10, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local function createButton(text, order, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, 10 + (order - 1) * 45)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.MouseButton1Click:Connect(callback)
end


local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

createButton(" Iniciar Auto Drive", 1, function()
    autoDrive = true
    notify("Auto Drive ON")
    spawn(function()
        while autoDrive do
            pressW(true)
            wait(10)
        end
    end)
end)

createButton(" Ativar Teleport", 2, function()
    teleportActive = true
    notify("Teleport ON ")
    spawn(function()
        while teleportActive do
            local car = workspace:FindFirstChild(player.Name)
            if car and car.PrimaryPart and car.PrimaryPart.Position.Z > endZ then
                notify("Voltando para o início da pista!")
                car:SetPrimaryPartCFrame(CFrame.new(startPos))
                wait(1)
            end
            wait(0.5)
        end
    end)
end)

createButton(" Definir Início Aqui", 3, function()
    local car = workspace:FindFirstChild(player.Name)
    if car and car.PrimaryPart then
        startPos = car.PrimaryPart.Position
        notify("Início definido! ")
    else
        notify("Entra no carro, amor ")
    end
end)

createButton(" Parar Tudo", 4, function()
    autoDrive = false
    teleportActive = false
    pressW(false)
    notify("Tudo parado. Te esperando ")
end)
