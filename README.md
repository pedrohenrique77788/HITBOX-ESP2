-- Carrega RedzhubUI
local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/tbao143/Library-ui/refs/heads/main/Redzhubui"))()

-- Cria a janela principal
local Window = redzlib:MakeWindow({
    Title = "Ghost Hub",
    SubTitle = "Hitbox + ESP",
    SaveFolder = "GhostHub"
})

-----------------------------------
-- 🌐 Aba de Hitbox
-----------------------------------
local TabHitbox = Window:MakeTab({"Hitbox", "⚔️"})

local Players = game:GetService('Players')
local RunService = game:GetService('RunService')
local LocalPlayer = Players.LocalPlayer

-- Configurações Hitbox
local hitboxEnabled = false
local BodySize = 20
local IsTeamCheckEnabled = false 
local hitboxesParts = {"Head","HumanoidRootPart","Torso","UpperTorso","LowerTorso","LeftArm","RightArm","LeftLeg","RightLeg"}

-- Função para aplicar hitbox
local function ApplyHitbox(character)
    if not character then return end
    for _, partName in ipairs(hitboxesParts) do
        local part = character:FindFirstChild(partName)
        if part then
            part.Size = Vector3.new(BodySize, BodySize, BodySize)
            part.Transparency = 0.7
            part.BrickColor = BrickColor.new("Really red")
            part.Material = Enum.Material.Neon
            part.CanCollide = false
        end
    end
end

RunService.RenderStepped:Connect(function()
    if hitboxEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and (not IsTeamCheckEnabled or player.Team ~= LocalPlayer.Team) then
                if player.Character then
                    ApplyHitbox(player.Character)
                end
            end
        end
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        if hitboxEnabled then
            ApplyHitbox(char)
        end
    end)
end)

-- Controles Hitbox na interface
TabHitbox:AddToggle({
    Name = "Ativar Hitbox",
    Default = false,
    Callback = function(Value)
        hitboxEnabled = Value
    end
})

TabHitbox:AddSlider({
    Name = "Tamanho da Hitbox",
    Min = 5,
    Max = 50,
    Increase = 1,
    Default = 20,
    Callback = function(value)
        BodySize = value
    end
})

TabHitbox:AddToggle({
    Name = "Ignorar Time",
    Default = true,
    Callback = function(Value)
        IsTeamCheckEnabled = not Value
    end
})

-----------------------------------
-- 🎯 Aba de ESP
-----------------------------------
local TabESP = Window:MakeTab({"ESP", "👁️"})

local Camera = workspace.CurrentCamera
local ESPEnabled = false
local ESPConnections = {}

local function createESP(player)
    if player == LocalPlayer then return end

    local esp = {}
    esp.Box = Drawing.new("Square")
    esp.Box.Thickness = 1.5
    esp.Box.Color = Color3.fromRGB(255, 255, 255)
    esp.Box.Filled = false
    esp.Box.Transparency = 0.8

    esp.HealthBar = Drawing.new("Square")
    esp.HealthBar.Thickness = 0
    esp.HealthBar.Filled = true

    esp.Text = Drawing.new("Text")
    esp.Text.Color = Color3.fromRGB(255, 255, 255)
    esp.Text.Center = true
    esp.Text.Size = 15
    esp.Text.Outline = true

    local connection
    connection = RunService.RenderStepped:Connect(function()
        if ESPEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = player.Character.HumanoidRootPart
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")

            local ok, cframe, size = pcall(function()
                return player.Character:GetBoundingBox()
            end)
            if not ok or size.Y == 0 then return end

            local topPos, topOnScreen = Camera:WorldToViewportPoint((cframe * CFrame.new(0, size.Y/2, 0)).Position)
            local bottomPos, bottomOnScreen = Camera:WorldToViewportPoint((cframe * CFrame.new(0, -size.Y/2, 0)).Position)

            if topPos.Z > 0 and bottomPos.Z > 0 then
                local height = math.abs(topPos.Y - bottomPos.Y)
                local width = height / 2
                if height > 0 then
                    local x = topPos.X - width/2
                    local y = topPos.Y

                    esp.Box.Size = Vector2.new(width, height)
                    esp.Box.Position = Vector2.new(x, y)
                    esp.Box.Visible = true

                    if humanoid then
                        local healthPercent = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
                        local barHeight = height * healthPercent
                        barHeight = math.clamp(barHeight, 1, height)
                        esp.HealthBar.Size = Vector2.new(4, barHeight)
                        esp.HealthBar.Position = Vector2.new(x - 6, y + (height - barHeight))
                        esp.HealthBar.Color = Color3.fromRGB(255 - (255 * healthPercent), 255 * healthPercent, 0)
                        esp.HealthBar.Visible = true
                    else
                        esp.HealthBar.Visible = false
                    end

                    local distance = (hrp.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                    esp.Text.Text = string.format("%s | %.0f studs", player.Name, distance)
                    esp.Text.Position = Vector2.new(topPos.X, y + height + 15)
                    esp.Text.Visible = true
                end
            else
                esp.Box.Visible = false
                esp.HealthBar.Visible = false
                esp.Text.Visible = false
            end
        else
            esp.Box.Visible = false
            esp.HealthBar.Visible = false
            esp.Text.Visible = false
        end
    end)

    player.AncestryChanged:Connect(function(_, parent)
        if not parent then
            esp.Box:Remove()
            esp.HealthBar:Remove()
            esp.Text:Remove()
            connection:Disconnect()
        end
    end)

    ESPConnections[player] = connection
end

for _, p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        createESP(p)
    end
end
Players.PlayerAdded:Connect(function(p) createESP(p) end)

-- Toggle do ESP
TabESP:AddToggle({
    Name = "Ativar ESP",
    Default = false,
    Callback = function(Value)
        ESPEnabled = Value
    end
})

-----------------------------------
-- Botão minimizar
-----------------------------------
Window:AddMinimizeButton({
    Button = { Image = "rbxassetid://71014873973869", BackgroundTransparency = 0 },
    Corner = { CornerRadius = UDim.new(35, 1) }
})
