local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

local autoFarmRunning = false

local START_CFRAME = CFrame.new(-1.0, 42.6, -4241.4)
local END_Z = -7245.7

function notify(txt)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Tianta AutoFarm",
            Text = txt,
            Duration = 4
        })
    end)
end

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

local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- Interface simples com botão AutoFarm
local screenGui = Instance.new("ScreenGui", game.CoreGui)
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0, 20, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1, -20, 0, 40)
button.Position = UDim2.new(0, 10, 0, 30)
button.Text = "Iniciar AutoFarm"
button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 16

button.MouseButton1Click:Connect(function()
    if autoFarmRunning then return end
    autoFarmRunning = true
    notify("AutoFarm iniciado")

    local car = nil
    repeat
        car = getCar()
        if not car then
            notify("Entra no carro para iniciar.")
            wait(1)
        end
    until car

    wait(0.5)
    car:SetPrimaryPartCFrame(START_CFRAME)
    wait(1)

    spawn(function()
        while autoFarmRunning do
            pressW(true)
            wait(10)
        end
    end)

    spawn(function()
        while autoFarmRunning do
            local car = getCar()
            if car then
                local pos = car.PrimaryPart.Position
                if pos.Z <= END_Z then
                    notify("Teleportando...")
                    car:SetPrimaryPartCFrame(START_CFRAME)
                    wait(1)
                end
            end
            wait(0.1)
        end
    end)
end)
