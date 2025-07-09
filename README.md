-- ✅ Serviços
local VirtualInputManager = game:GetService("VirtualInputManager")
local player = game.Players.LocalPlayer

-- 🧲 Anti-Cheat / Anti-Ban
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local oldNamecall = mt.__namecall

    mt.__namecall = newcclosure(function(self, ...)
        local args = {...}
        local method = getnamecallmethod()
        if tostring(self) == "Kick" or method == "Kick" then
            warn("🚫 Tentativa de Kick bloqueada!")
            return nil
        end
        if tostring(self) == "Ban" or method == "Ban" then
            warn("🚫 Tentativa de Ban detectada!")
            return nil
        end
        return oldNamecall(self, unpack(args))
    end)

    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")

    humanoid:GetPropertyChangedSignal("Health"):Connect(function()
        if humanoid.Health <= 0 then
            humanoid.Health = 100
            warn("❤️ Tentaram te matar — Vida restaurada")
        end
    end)

    humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
        if humanoid.WalkSpeed == 0 then
            humanoid.WalkSpeed = 16
            warn("🏃‍♂️ Tentaram travar sua velocidade — Corrigido")
        end
    end)
end)

-- 📌 Funções Utilitárias
local shownMessages = {}
local function notify(txt)
    if shownMessages[txt] then return end
    shownMessages[txt] = true
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "🚗 AutoFarm",
            Text = txt,
            Duration = 3
        })
    end)
end

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

local function isInCar()
    local char = player.Character
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    return humanoid and humanoid.SeatPart ~= nil
end

local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- 🖼️ MENU UI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaMenuUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 250, 0, 180)
mainFrame.Position = UDim2.new(0, 20, 0.4, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0

local header = Instance.new("TextLabel", mainFrame)
header.Size = UDim2.new(1, 0, 0, 30)
header.Text = "💖 MOD MENU | RSJGAMES"
header.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
header.TextColor3 = Color3.new(1, 1, 1)
header.Font = Enum.Font.GothamBold
header.TextSize = 14

-- 🖱️ Menu Arrastável
local dragging, dragStart, startPos = false
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
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
header.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- ❌ Fechar
local close = Instance.new("TextButton", mainFrame)
close.Size = UDim2.new(0, 25, 0, 25)
close.Position = UDim2.new(1, -30, 0, 2)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
close.TextColor3 = Color3.new(1, 1, 1)
close.Font = Enum.Font.GothamBold
close.TextSize = 16
close.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- 🔽 Minimizar
local minimize = Instance.new("TextButton", mainFrame)
minimize.Size = UDim2.new(0, 25, 0, 25)
minimize.Position = UDim2.new(1, -55, 0, 2)
minimize.Text = "-"
minimize.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.Font = Enum.Font.GothamBold
minimize.TextSize = 16
minimize.MouseButton1Click:Connect(function()
    local min = (mainFrame.Size.Y.Offset <= 40)
    mainFrame.Size = min and UDim2.new(0, 250, 0, 180) or UDim2.new(0, 250, 0, 35)
    button1.Visible = min
    button2.Visible = min
end)

-- 💸 AutoFarm 1
local button1 = Instance.new("TextButton", mainFrame)
button1.Size = UDim2.new(1, -20, 0, 40)
button1.Position = UDim2.new(0, 10, 0, 40)
button1.Text = "AutoFarm 1 OFF"
button1.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
button1.TextColor3 = Color3.new(1, 1, 1)
button1.Font = Enum.Font.GothamBold
button1.TextSize = 14

-- 💸 AutoFarm 2
local button2 = Instance.new("TextButton", mainFrame)
button2.Size = UDim2.new(1, -20, 0, 40)
button2.Position = UDim2.new(0, 10, 0, 90)
button2.Text = "AutoFarm 2 OFF"
button2.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
button2.TextColor3 = Color3.new(1, 1, 1)
button2.Font = Enum.Font.GothamBold
button2.TextSize = 14

-- 🚗 AutoFarm Config
local autoFarm1Running = false
local autoFarm2Running = false
local endZ1 = -2598
local endZ2 = -4750

local function startAutoFarm(cf, endZ, btn, stateVar)
    if not isInCar() then
        notify("🚫 Você não está no carro!")
        _G[stateVar] = false
        btn.Text = "AutoFarm OFF"
        return
    end
    local car = getCar()
    repeat wait() until car and car.PrimaryPart
    wait(0.3)
    car:SetPrimaryPartCFrame(cf)

    local drive = task.spawn(function()
        while _G[stateVar] do
            pressW(true)
            wait(10)
        end
    end)

    local tp = task.spawn(function()
        while _G[stateVar] do
            if not isInCar() then
                notify("⛔ Saiu do carro — AutoFarm parado")
                btn.Text = "AutoFarm OFF"
                _G[stateVar] = false
                task.cancel(drive)
                return
            end
            local car = getCar()
            if not car or not car.PrimaryPart then
                notify("🚫 Carro inválido — AutoFarm parado")
                btn.Text = "AutoFarm OFF"
                _G[stateVar] = false
                task.cancel(drive)
                return
            end
            if car.PrimaryPart.Position.Z <= endZ then
                car:SetPrimaryPartCFrame(cf)
                wait(1)
            end
            wait(0.1)
        end
    end)
end

-- ▶️ AutoFarm 1
button1.MouseButton1Click:Connect(function()
    autoFarm1Running = not autoFarm1Running
    button1.Text = autoFarm1Running and "AutoFarm 1 ON" or "AutoFarm 1 OFF"
    if autoFarm1Running then
        _G["autoFarm1Running"] = true
        startAutoFarm(CFrame.new(1920.9, 30.8, -1610.7) * CFrame.Angles(0, 0, 0), endZ1, button1, "autoFarm1Running")
    else
        _G["autoFarm1Running"] = false
        pressW(false)
    end
end)

-- ▶️ AutoFarm 2
button2.MouseButton1Click:Connect(function()
    autoFarm2Running = not autoFarm2Running
    button2.Text = autoFarm2Running and "AutoFarm 2 ON" or "AutoFarm 2 OFF"
    if autoFarm2Running then
        _G["autoFarm2Running"] = true
        startAutoFarm(CFrame.new(-84.2, 42.6, -4175.5) * CFrame.Angles(0, math.rad(90), 0), endZ2, button2, "autoFarm2Running")
    else
        _G["autoFarm2Running"] = false
        pressW(false)
    end
end)
