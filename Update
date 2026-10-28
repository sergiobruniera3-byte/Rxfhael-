-- Rayfield Library Load
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "RX HUB",
   LoadingTitle = "RX HUB Loading...",
   LoadingSubtitle = "by Rxfhael",
   ConfigurationSaving = { Enabled = false }
})

-- Services & Variables
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

--------------------------------------------------------------------------------
-- SETTINGS & CONFIG
--------------------------------------------------------------------------------
local Config = {
    ESP = {
        Enabled = false,
        CornerBox = false,
        HealthBar = false,
        Skeleton = false,
        Name = false,
        Tracer = false,
        MaxDistance = 1000
    },
    Aimbot = {
        Enabled = false,
        Smoothness = 0.3,
        TeamCheck = false,
        Whitelist = "", -- Nicks separados por vírgula, ex: Amigo1, Amigo2
        WallCheck = false,
        ShowFOV = false,
        FOVSize = 180,
        TargetPart = "Head"
    }
}

--------------------------------------------------------------------------------
-- FOV CIRCLE DRAWING
--------------------------------------------------------------------------------
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1
FOVCircle.NumSides = 64
FOVCircle.Filled = false
FOVCircle.Visible = false

RunService.RenderStepped:Connect(function()
    FOVCircle.Visible = Config.Aimbot.ShowFOV and Config.Aimbot.Enabled
    FOVCircle.Radius = Config.Aimbot.FOVSize
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
end)

--------------------------------------------------------------------------------
-- AIMBOT SYSTEM (MOBILE ULTIMATE + TEAM/WHITELIST CHECK)
--------------------------------------------------------------------------------
local function IsWhitelisted(playerName)
    if Config.Aimbot.Whitelist == "" then return false end
    for name in string.gmatch(Config.Aimbot.Whitelist, '([^,]+)') do
        local cleanName = name:match("^%s*(.-)%s*$") -- Remove espaços
        if string.lower(playerName) == string.lower(cleanName) then
            return true
        end
    end
    return false
end

local function IsVisible(targetPart)
    if not Config.Aimbot.WallCheck then return true end
    local origin = Camera.CFrame.Position
    local direction = (targetPart.Position - origin)
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = RaycastFilterType.Exclude
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, Camera}
    
    local result = Workspace:Raycast(origin, direction, raycastParams)
    if result and result.Instance:IsDescendantOf(targetPart.Parent) then
        return true
    end
    return false
end

local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = Config.Aimbot.FOVSize
    local centerScreen = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            -- Team Check
            local sameTeam = false
            if Config.Aimbot.TeamCheck and player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then
                sameTeam = true
            end

            -- Whitelist Check (Ignorar amigos por nick)
            local whitelisted = IsWhitelisted(player.Name) or (player.DisplayName and IsWhitelisted(player.DisplayName))

            if not sameTeam and not whitelisted then
                local hum = player.Character:FindFirstChildOfClass("Humanoid")
                local targetPart = player.Character:FindFirstChild(Config.Aimbot.TargetPart)
                
                if hum and hum.Health > 0 and targetPart then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)

                    if onScreen then
                        local mouseDist = (Vector2.new(screenPos.X, screenPos.Y) - centerScreen).Magnitude
                        if mouseDist < shortestDistance then
                            if IsVisible(targetPart) then
                                shortestDistance = mouseDist
                                closestPlayer = player
                            end
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

RunService.RenderStepped:Connect(function()
    if Config.Aimbot.Enabled then
        local target = GetClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild(Config.Aimbot.TargetPart) then
            local targetPart = target.Character[Config.Aimbot.TargetPart]
            local camPos = Camera.CFrame.Position
            local targetCFrame = CFrame.new(camPos, targetPart.Position)
            Camera.CFrame = Camera.CFrame:Lerp(targetCFrame, Config.Aimbot.Smoothness)
        end
    end
end)

--------------------------------------------------------------------------------
-- ESP SYSTEM (IMPROVED SKELETON)
--------------------------------------------------------------------------------
local ESPObjects = {}

local function CreateESP(player)
    if player == LocalPlayer then return end
    
    local drawings = {
        CornerTopLeftHor = Drawing.new("Line"), CornerTopLeftVert = Drawing.new("Line"),
        CornerTopRightHor = Drawing.new("Line"), CornerTopRightVert = Drawing.new("Line"),
        CornerBottomLeftHor = Drawing.new("Line"), CornerBottomLeftVert = Drawing.new("Line"),
        CornerBottomRightHor = Drawing.new("Line"), CornerBottomRightVert = Drawing.new("Line"),
        HealthBarBG = Drawing.new("Line"), HealthBar = Drawing.new("Line"),
        Name = Drawing.new("Text"),
        Tracer = Drawing.new("Line"),
        Skeleton = {}
    }

    -- Conexões de esqueleto atualizadas para melhor precisão R15
    local connections = {
        {"Head", "UpperTorso"}, 
        {"UpperTorso", "LowerTorso"},
        {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
        {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
        {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
        {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}
    }
    
    for i = 1, #connections do
        local line = Drawing.new("Line")
        line.Color = Color3.fromRGB(255, 255, 255)
        line.Thickness = 1.2
        table.insert(drawings.Skeleton, {Line = line, Connection = connections[i]})
    end

    drawings.Name.Center = true
    drawings.Name.Outline = true
    drawings.Name.Size = 14
    drawings.Name.Color = Color3.fromRGB(255, 255, 255)

    drawings.Tracer.Color = Color3.fromRGB(255, 0, 0)
    drawings.Tracer.Thickness = 1

    ESPObjects[player] = drawings
end

local function RemoveESP(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do
            if type(obj) == "table" then
                for _, subObj in pairs(obj) do subObj.Line:Remove() end
            else
                obj:Remove()
            end
        end
        ESPObjects[player] = nil
    end
end

for _, p in ipairs(Players:GetPlayers()) do CreateESP(p) end
Players.PlayerAdded:Connect(CreateESP)
Players.PlayerRemoving:Connect(RemoveESP)

RunService.RenderStepped:Connect(function()
    for player, esp in pairs(ESPObjects) do
        local char = player.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local hrp = char and char:FindFirstChild("HumanoidRootPart")

        if Config.ESP.Enabled and char and hum and hrp and hum.Health > 0 then
            local dist = (hrp.Position - Camera.CFrame.Position).Magnitude
            local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

            if onScreen and dist <= Config.ESP.MaxDistance then
                local head = char:FindFirstChild("Head")
                local headPos = head and Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0)) or pos
                local legPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))

                local height = math.abs(headPos.Y - legPos.Y)
                local width = height / 2
                local x, y = pos.X - width / 2, pos.Y - height / 2

                -- Corner Box
                if Config.ESP.CornerBox then
                    local cornerLen = width / 3
                    local color = Color3.fromRGB(0, 255, 150)
                    
                    esp.CornerTopLeftHor.From, esp.CornerTopLeftHor.To = Vector2.new(x, y), Vector2.new(x + cornerLen, y)
                    esp.CornerTopLeftVert.From, esp.CornerTopLeftVert.To = Vector2.new(x, y), Vector2.new(x, y + cornerLen)
                    
                    esp.CornerTopRightHor.From, esp.CornerTopRightHor.To = Vector2.new(x + width, y), Vector2.new(x + width - cornerLen, y)
                    esp.CornerTopRightVert.From, esp.CornerTopRightVert.To = Vector2.new(x + width, y), Vector2.new(x + width, y + cornerLen)
                    
                    esp.CornerBottomLeftHor.From, esp.CornerBottomLeftHor.To = Vector2.new(x, y + height), Vector2.new(x + cornerLen, y + height)
                    esp.CornerBottomLeftVert.From, esp.CornerBottomLeftVert.To = Vector2.new(x, y + height), Vector2.new(x, y + height - cornerLen)
                    
                    esp.CornerBottomRightHor.From, esp.CornerBottomRightHor.To = Vector2.new(x + width, y + height), Vector2.new(x + width - cornerLen, y + height)
                    esp.CornerBottomRightVert.From, esp.CornerBottomRightVert.To = Vector2.new(x + width, y + height), Vector2.new(x + width, y + height - cornerLen)

                    for _, lineKey in ipairs({"CornerTopLeftHor", "CornerTopLeftVert", "CornerTopRightHor", "CornerTopRightVert", "CornerBottomLeftHor", "CornerBottomLeftVert", "CornerBottomRightHor", "CornerBottomRightVert"}) do
                        esp[lineKey].Color = color
                        esp[lineKey].Visible = true
                    end
                else
                    for _, lineKey in ipairs({"CornerTopLeftHor", "CornerTopLeftVert", "CornerTopRightHor", "CornerTopRightVert", "CornerBottomLeftHor", "CornerBottomLeftVert", "CornerBottomRightHor", "CornerBottomRightVert"}) do
                        esp[lineKey].Visible = false
                    end
                end

                -- Health Bar
                if Config.ESP.HealthBar then
                    local hpPercent = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
                    local barX = x - 5
                    
                    esp.HealthBarBG.From = Vector2.new(barX, y)
                    esp.HealthBarBG.To = Vector2.new(barX, y + height)
                    esp.HealthBarBG.Color = Color3.fromRGB(50, 50, 50)
                    esp.HealthBarBG.Thickness = 2
                    esp.HealthBarBG.Visible = true

                    esp.HealthBar.From = Vector2.new(barX, y + height)
                    esp.HealthBar.To = Vector2.new(barX, y + height - (height * hpPercent))
                    esp.HealthBar.Color = Color3.fromRGB(255 - (255 * hpPercent), 255 * hpPercent, 0)
                    esp.HealthBar.Thickness = 2
                    esp.HealthBar.Visible = true
                else
                    esp.HealthBarBG.Visible = false
                    esp.HealthBar.Visible = false
                end

                -- Name
                if Config.ESP.Name then
                    esp.Name.Position = Vector2.new(pos.X, y - 16)
                    esp.Name.Text = player.Name .. " [" .. math.floor(dist) .. "m]"
                    esp.Name.Visible = true
                else
                    esp.Name.Visible = false
                end

                -- Tracer
                if Config.ESP.Tracer then
                    esp.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, 0)
                    esp.Tracer.To = Vector2.new(pos.X, pos.Y)
                    esp.Tracer.Visible = true
                else
                    esp.Tracer.Visible = false
                end

                -- Skeleton (Melhorado e Otimizado)
                if Config.ESP.Skeleton then
                    for _, skel in ipairs(esp.Skeleton) do
                        local partA = char:FindFirstChild(skel.Connection[1])
                        local partB = char:FindFirstChild(skel.Connection[2])

                        if partA and partB then
                            local posA, visA = Camera:WorldToViewportPoint(partA.Position)
                            local posB, visB = Camera:WorldToViewportPoint(partB.Position)

                            if visA and visB then
                                skel.Line.From = Vector2.new(posA.X, posA.Y)
                                skel.Line.To = Vector2.new(posB.X, posB.Y)
                                skel.Line.Visible = true
                            else
                                skel.Line.Visible = false
                            end
                        else
                            skel.Line.Visible = false
                        end
                    end
                else
                    for _, skel in ipairs(esp.Skeleton) do skel.Line.Visible = false end
                end

            else
                for k, v in pairs(esp) do
                    if k == "Skeleton" then
                        for _, skel in ipairs(v) do skel.Line.Visible = false end
                    else
                        v.Visible = false
                    end
                end
            end
        else
            for k, v in pairs(esp) do
                if k == "Skeleton" then
                    for _, skel in ipairs(v) do skel.Line.Visible = false end
                else
                    v.Visible = false
                end
            end
        end
    end
end)

--------------------------------------------------------------------------------
-- RAYFIELD UI INTERFACE SETUP
--------------------------------------------------------------------------------
local PlayerTab = Window:CreateTab("Player", 4483362458)
local PVPTab = Window:CreateTab("PVP", 4483362458)

-- ESP Section
PlayerTab:CreateSection("ESP Professional")

PlayerTab:CreateToggle({
   Name = "Master Toggle ESP",
   CurrentValue = false,
   Callback = function(Value) Config.ESP.Enabled = Value end,
})

PlayerTab:CreateToggle({
   Name = "Corner Box",
   CurrentValue = false,
   Callback = function(Value) Config.ESP.CornerBox = Value end,
})

PlayerTab:CreateToggle({
   Name = "Health Bar",
   CurrentValue = false,
   Callback = function(Value) Config.ESP.HealthBar = Value end,
})

PlayerTab:CreateToggle({
   Name = "Skeleton (Esqueleto R15)",
   CurrentValue = false,
   Callback = function(Value) Config.ESP.Skeleton = Value end,
})

PlayerTab:CreateToggle({
   Name = "Player Name",
   CurrentValue = false,
   Callback = function(Value) Config.ESP.Name = Value end,
})

PlayerTab:CreateToggle({
   Name = "Tracer (Topo da Tela)",
   CurrentValue = false,
   Callback = function(Value) Config.ESP.Tracer = Value end,
})

PlayerTab:CreateSlider({
   Name = "Distância Máxima ESP",
   Range = {50, 5000},
   Increment = 50,
   Suffix = "m",
   CurrentValue = 1000,
   Callback = function(Value) Config.ESP.MaxDistance = Value end,
})

-- Aimbot Section (Mobile + Anti-Amigo)
PVPTab:CreateSection("Aimbot Mobile")

PVPTab:CreateToggle({
   Name = "Ativar Aimbot",
   CurrentValue = false,
   Callback = function(Value) Config.Aimbot.Enabled = Value end,
})

PVPTab:CreateToggle({
   Name = "Team Check (Ignorar Time)",
   CurrentValue = false,
   Callback = function(Value) Config.Aimbot.TeamCheck = Value end,
})

PVPTab:CreateInput({
   Name = "Ignorar Nicks (Ex: Nick1, Nick2)",
   PlaceholderText = "Separe por vírgula",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      Config.Aimbot.Whitelist = Text
   end,
})

PVPTab:CreateDropdown({
   Name = "Mira Alvo",
   Options = {"Head", "HumanoidRootPart"},
   CurrentOption = "Head",
   Callback = function(Option) Config.Aimbot.TargetPart = Option end,
})

PVPTab:CreateSlider({
   Name = "Suavidade (Smoothness)",
   Range = {1, 10},
   Increment = 1,
   Suffix = "",
   CurrentValue = 3,
   Callback = function(Value) Config.Aimbot.Smoothness = Value / 10 end,
})

PVPTab:CreateToggle({
   Name = "WallCheck (Linha de Visão)",
   CurrentValue = false,
   Callback = function(Value) Config.Aimbot.WallCheck = Value end,
})

PVPTab:CreateToggle({
   Name = "Mostrar FOV",
   CurrentValue = false,
   Callback = function(Value) Config.Aimbot.ShowFOV = Value end,
})

PVPTab:CreateSlider({
   Name = "Tamanho do FOV",
   Range = {50, 400},
   Increment = 10,
   Suffix = "px",
   CurrentValue = 180,
   Callback = function(Value) Config.Aimbot.FOVSize = Value end,
})
