local players = game:GetService("Players")
local localPlayer = players.LocalPlayer
local camera = game:GetService("Workspace").CurrentCamera
local aimEnabled, espEnabled, aimToggle, tpEnabled = false, false, false, false
local trackedPlayer = nil  -- Игрок, за которым будет следить aimbot
local lines = {}  -- Для хранения линий ESP
local aimKey = Enum.KeyCode.LeftAlt
local aimRadius = 300  -- Радиус круга aimbot (по умолчанию 300)

-- Создание меню GUI
local screenGui = Instance.new("ScreenGui", localPlayer.PlayerGui)
screenGui.Name = "AimbotESPMenu"

-- Основное меню
local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 200, 0, 230)
mainFrame.Position = UDim2.new(0.5, -100, 0.4, -75)
mainFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
mainFrame.Visible = true
mainFrame.Active = true
mainFrame.Draggable = true

-- Заголовок меню
local titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.Size = UDim2.new(1, 0, 0, 30)
titleLabel.Position = UDim2.new(0, 0, 0, -30)
titleLabel.Text = "script by gimgo10"
titleLabel.BackgroundTransparency = 1
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Font = Enum.Font.SourceSans
titleLabel.TextSize = 18

-- Кнопка для включения и выключения Aimbot
local aimButton = Instance.new("TextButton", mainFrame)
aimButton.Size = UDim2.new(1, -20, 0, 30)
aimButton.Position = UDim2.new(0, 10, 0, 10)
aimButton.Text = "Toggle Aimbot (OFF)"
aimButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

-- Кнопка для включения и выключения ESP
local espButton = Instance.new("TextButton", mainFrame)
espButton.Size = UDim2.new(1, -20, 0, 30)
espButton.Position = UDim2.new(0, 10, 0, 50)
espButton.Text = "Toggle ESP (OFF)"
espButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

-- Ползунок для регулировки радиуса круга aimbot
local radiusSliderLabel = Instance.new("TextLabel", mainFrame)
radiusSliderLabel.Size = UDim2.new(1, -20, 0, 20)
radiusSliderLabel.Position = UDim2.new(0, 10, 0, 90)
radiusSliderLabel.Text = "Aimbot Radius: " .. aimRadius
radiusSliderLabel.BackgroundTransparency = 1
radiusSliderLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
radiusSliderLabel.Font = Enum.Font.SourceSans
radiusSliderLabel.TextSize = 14

-- Кнопка для увеличения радиуса круга
local increaseRadiusButton = Instance.new("TextButton", mainFrame)
increaseRadiusButton.Size = UDim2.new(0.5, -10, 0, 30)
increaseRadiusButton.Position = UDim2.new(0, 10, 0, 120)
increaseRadiusButton.Text = "+"
increaseRadiusButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

-- Кнопка для уменьшения радиуса круга
local decreaseRadiusButton = Instance.new("TextButton", mainFrame)
decreaseRadiusButton.Size = UDim2.new(0.5, -10, 0, 30)
decreaseRadiusButton.Position = UDim2.new(0.5, 0, 0, 120)
decreaseRadiusButton.Text = "-"
decreaseRadiusButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

-- Кнопка для переключения телепортации Ctrl + ЛКМ
local tpButton = Instance.new("TextButton", mainFrame)
tpButton.Size = UDim2.new(1, -20, 0, 30)
tpButton.Position = UDim2.new(0, 10, 0, 160)
tpButton.Text = "Toggle TP (OFF)"
tpButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

-- Кнопка для сворачивания меню
local closeButton = Instance.new("TextButton", mainFrame)
closeButton.Size = UDim2.new(1, -20, 0, 30)
closeButton.Position = UDim2.new(0, 10, 0, 200)
closeButton.Text = "Close Menu"
closeButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

-- Переключение aimbot через меню
aimButton.MouseButton1Click:Connect(function()
    aimToggle = not aimToggle
    aimButton.Text = "Toggle Aimbot (" .. (aimToggle and "ON" or "OFF") .. ")"
end)

-- Переключение ESP через меню
espButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    espButton.Text = "Toggle ESP (" .. (espEnabled and "ON" or "OFF") .. ")"
end)

-- Включение и выключение телепортации по Ctrl + ЛКМ
tpButton.MouseButton1Click:Connect(function()
    tpEnabled = not tpEnabled
    tpButton.Text = "Toggle TP (" .. (tpEnabled and "ON" or "OFF") .. ")"
end)

-- Закрытие меню
closeButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
end)

-- Изменение радиуса круга
increaseRadiusButton.MouseButton1Click:Connect(function()
    aimRadius = aimRadius + 50
    radiusSliderLabel.Text = "Aimbot Radius: " .. aimRadius
end)

decreaseRadiusButton.MouseButton1Click:Connect(function()
    if aimRadius > 50 then
        aimRadius = aimRadius - 50
        radiusSliderLabel.Text = "Aimbot Radius: " .. aimRadius
    end
end)

-- Функция для прицеливания на середину головы с небольшим смещением вниз
local function aimAtHead(target)
    if target and target:FindFirstChild("Head") then
        local head = target.Head
        local adjustedPosition = head.Position + Vector3.new(0, head.Size.Y / 3, 0)
        camera.CFrame = CFrame.new(camera.CFrame.Position, adjustedPosition)
    end
end

-- Нахождение ближайшего игрока в радиусе круга
local function getClosestPlayerInRadius()
    local closestPlayer = nil
    local shortestDistance = math.huge
    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local screenPoint, onScreen = camera:WorldToScreenPoint(player.Character.Head.Position)
            local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)).Magnitude
            if onScreen and distance < shortestDistance and distance <= aimRadius then
                closestPlayer = player
                shortestDistance = distance
            end
        end
    end
    return closestPlayer
end

-- Управление активацией aimbot с помощью Left Alt
game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.KeyCode == aimKey and aimToggle then
        if trackedPlayer then
            trackedPlayer = nil
            aimEnabled = false
        else
            trackedPlayer = getClosestPlayerInRadius()
            aimEnabled = trackedPlayer ~= nil
        end
    end
end)

-- Телепортация при Ctrl + ЛКМ, если включено
local ctrlPressed = false

game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.LeftControl then
        ctrlPressed = true
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.LeftControl then
        ctrlPressed = false
    end
end)

localPlayer:GetMouse().Button1Down:Connect(function()
    if ctrlPressed and tpEnabled then
        local mousePos = localPlayer:GetMouse().Hit.Position
        localPlayer.Character:MoveTo(mousePos)
    end
end)

-- Обновление каждый кадр
game:GetService("RunService").RenderStepped:Connect(function()
    if aimEnabled and trackedPlayer and trackedPlayer.Character then
        aimAtHead(trackedPlayer.Character)
    end
    if espEnabled then
        -- вызов функций для ESP здесь
    end
end)
