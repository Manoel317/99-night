-- Mod Menu Profissional para Roblox (Servidor Privado)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local config = {
    godMode = false,
    espEnabled = false,
    killAura = false,
    autoRegen = false,
    dynamicTime = false,
    timePerUnit = 15
}

local screenGui = Instance.new("ScreenGui", PlayerGui)
screenGui.Name = "ModMenu"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 250, 0, 330)
frame.Position = UDim2.new(0, 20, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0

local function createToggle(name, yPos, key)
    local button = Instance.new("TextButton", frame)
    button.Size = UDim2.new(0, 230, 0, 30)
    button.Position = UDim2.new(0, 10, 0, yPos)
    button.Text = name .. ": OFF"
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.MouseButton1Click:Connect(function()
        config[key] = not config[key]
        button.Text = name .. ": " .. (config[key] and "ON" or "OFF")
    end)
end

createToggle("God Mode", 10, "godMode")
createToggle("ESP Itens", 50, "espEnabled")
createToggle("Kill Aura", 90, "killAura")
createToggle("Auto Regen", 130, "autoRegen")
createToggle("Tempo Dinâmico", 170, "dynamicTime")

local organizeButton = Instance.new("TextButton", frame)
organizeButton.Size = UDim2.new(0, 230, 0, 30)
organizeButton.Position = UDim2.new(0, 10, 0, 210)
organizeButton.Text = "Organizar Recursos"
organizeButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
organizeButton.TextColor3 = Color3.new(1, 1, 1)
organizeButton.MouseButton1Click:Connect(function()
    organizeResources()
end)

local tpItemsButton = Instance.new("TextButton", frame)
tpItemsButton.Size = UDim2.new(0, 230, 0, 30)
tpItemsButton.Position = UDim2.new(0, 10, 0, 250)
tpItemsButton.Text = "Teleportar Itens para Jogador"
tpItemsButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
tpItemsButton.TextColor3 = Color3.new(1, 1, 1)
tpItemsButton.MouseButton1Click:Connect(function()
    teleportAllItems()
end)

local openChestsButton = Instance.new("TextButton", frame)
openChestsButton.Size = UDim2.new(0, 230, 0, 30)
openChestsButton.Position = UDim2.new(0, 10, 0, 290)
openChestsButton.Text = "Abrir Todos os Baús"
openChestsButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
openChestsButton.TextColor3 = Color3.new(1, 1, 1)
openChestsButton.MouseButton1Click:Connect(function()
    openAllChests()
end)

local function applyGodMode()
    if config.godMode and LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.Health = humanoid.MaxHealth
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        end
    end
end

local function applyESP()
    if not config.espEnabled then return end
    for _, item in pairs(workspace:GetChildren()) do
        if item:IsA("Tool") and not item:FindFirstChild("ESP") then
            local billboard = Instance.new("BillboardGui", item)
            billboard.Name = "ESP"
            billboard.Size = UDim2.new(0, 100, 0, 40)
            billboard.AlwaysOnTop = true
            billboard.Adornee = item

            local label = Instance.new("TextLabel", billboard)
            label.Size = UDim2.new(1, 0, 1, 0)
            label.BackgroundTransparency = 1
            label.Text = item.Name
            label.TextColor3 = Color3.new(0, 1, 0)
        end
    end
end

local function applyKillAura()
    if not config.killAura or not LocalPlayer.Character then return end
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") and not Players:GetPlayerFromCharacter(obj) then
            local humanoid = obj:FindFirstChildOfClass("Humanoid")
            local root = obj:FindFirstChild("HumanoidRootPart")
            local myRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and root and myRoot and (root.Position - myRoot.Position).Magnitude < 10 then
                humanoid.Health = 0
            end
        end
    end
end

local function applyAutoRegen()
    if not config.autoRegen or not LocalPlayer.Character then return end
    local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.Health < humanoid.MaxHealth then
        humanoid.Health = math.min(humanoid.MaxHealth, humanoid.Health + 1)
    end
end

local function applyDynamicTime()
    if not config.dynamicTime then return end
    local beds = workspace:FindFirstChild("Beds")
    local children = workspace:FindFirstChild("Children")

    local bedCount = beds and #beds:GetChildren() or 0
    local childCount = children and #children:GetChildren() or 0

    local multiplier = math.max(1, (bedCount + childCount) * config.timePerUnit)
    if workspace:FindFirstChild("GameClock") then
        workspace.GameClock.TimeScale = multiplier
    end
end

function organizeResources()
    local fogueira = workspace:FindFirstChild("Fogueira")
    local bancada = workspace:FindFirstChild("BancadaTriturador")
    if not fogueira or not bancada then return end

    for _, item in pairs(workspace:GetDescendants()) do
        if item:IsA("Tool") or item:IsA("Part") then
            local name = item.Name:lower()
            if (name:find("combustível") or name:find("carvão") or name:find("óleo")) and not name:find("madeira") then
                item.Position = fogueira.Position + Vector3.new(0, 2, 0)
            elseif name:find("madeira") or name:find("metal") or name:find("gema de cultista")
