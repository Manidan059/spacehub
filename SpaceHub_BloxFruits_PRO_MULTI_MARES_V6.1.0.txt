--//======================================================
--// SPACE HUB
--// PREMIUM ORBITAL INTERFACE
--// VERSION 4.0.0
--//======================================================

--//======================================================
--// SERVICES
--//======================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")

local LocalPlayer = Players.LocalPlayer

--//======================================================
--// STATE
--//======================================================

local Character
local Humanoid
local HRP

local walkSpeed = 16
local flightSpeed = 50

local flying = false
local infiniteJump = false
local noclip = false
local fullbright = false

local espEnabled = false

local aimbotEnabled = false
local aimbotFOVEnabled = true
local aimbotFOV = 250
local aimbotSmoothness = 0.18
local teamCheck = false
local aimbotMaxDistance = 500
local aimbotPriority = "FOV"
local aimbotPart = "Head"
local aimbotVisibleCheck = false

local espShowName = true
local espShowHealth = true
local espShowDistance = true
local espTeamColors = true
local espMaxDistance = 1000

local jumpPower = 50
local hipHeight = 2
local customGravityEnabled = false
local customGravity = 196.2
local originalGravity = workspace.Gravity

local selectedPlayer = nil
local spectating = false
local selectedWaypoint = nil
local waypoints = {}
local previousPosition = nil

local dashboardLabels = {}

local flyConnection
local aimbotConnection
local noclipConnection
local jumpConnection
local fullbrightConnection

local espObjects = {}

local originalLighting = {
    Brightness = Lighting.Brightness,
    ClockTime = Lighting.ClockTime,
    FogEnd = Lighting.FogEnd,
    GlobalShadows = Lighting.GlobalShadows
}

--//======================================================
--// CHARACTER SYSTEM
--//======================================================

local function updateCharacter(character)

    Character = character

    Humanoid = character:WaitForChild(
        "Humanoid",
        10
    )

    HRP = character:WaitForChild(
        "HumanoidRootPart",
        10
    )

    if Humanoid then
        Humanoid.WalkSpeed = walkSpeed
        Humanoid.UseJumpPower = true
        Humanoid.JumpPower = jumpPower
        Humanoid.HipHeight = hipHeight
    end

end

if LocalPlayer.Character then
    updateCharacter(LocalPlayer.Character)
end

--//======================================================
--// RAYFIELD
--//======================================================

local RayfieldSource
local Rayfield

local ok, result = pcall(function()
    return game:HttpGet("https://sirius.menu/rayfield")
end)

if not ok or type(result) ~= "string" or result == "" then
    warn("[SPACE HUB] No se pudo descargar Rayfield.")
    return
end

local loadOk, loaded = pcall(loadstring, result)

if not loadOk or type(loaded) ~= "function" then
    warn("[SPACE HUB] No se pudo cargar Rayfield.")
    return
end

local initOk, initResult = pcall(loaded)

if not initOk or type(initResult) ~= "table" then
    warn("[SPACE HUB] Rayfield devolvió un resultado inválido.")
    return
end

Rayfield = initResult

--//======================================================
--// WINDOW
--//======================================================

local SpaceTheme = {
    TextColor = Color3.fromRGB(232, 244, 255),
    Background = Color3.fromRGB(8, 12, 22),
    Topbar = Color3.fromRGB(11, 18, 32),
    Shadow = Color3.fromRGB(0, 0, 0),

    NotificationBackground = Color3.fromRGB(13, 22, 38),
    NotificationActionsBackground = Color3.fromRGB(22, 34, 54),

    TabBackground = Color3.fromRGB(12, 20, 34),
    TabStroke = Color3.fromRGB(28, 45, 67),
    TabBackgroundSelected = Color3.fromRGB(24, 82, 112),
    TabTextColor = Color3.fromRGB(145, 169, 194),
    SelectedTabTextColor = Color3.fromRGB(235, 250, 255),

    ElementBackground = Color3.fromRGB(13, 21, 35),
    ElementBackgroundHover = Color3.fromRGB(18, 31, 49),
    SecondaryElementBackground = Color3.fromRGB(9, 16, 28),
    ElementStroke = Color3.fromRGB(27, 48, 70),
    SecondaryElementStroke = Color3.fromRGB(20, 36, 54),

    SliderBackground = Color3.fromRGB(23, 48, 70),
    SliderProgress = Color3.fromRGB(0, 190, 255),
    SliderStroke = Color3.fromRGB(73, 215, 255),

    ToggleBackground = Color3.fromRGB(17, 28, 43),
    ToggleEnabled = Color3.fromRGB(0, 170, 230),
    ToggleDisabled = Color3.fromRGB(57, 72, 91),
    ToggleEnabledStroke = Color3.fromRGB(72, 220, 255),
    ToggleDisabledStroke = Color3.fromRGB(74, 91, 112),
    ToggleEnabledOuterStroke = Color3.fromRGB(30, 88, 113),
    ToggleDisabledOuterStroke = Color3.fromRGB(36, 49, 66),

    DropdownSelected = Color3.fromRGB(20, 36, 55),
    DropdownUnselected = Color3.fromRGB(12, 21, 34),

    InputBackground = Color3.fromRGB(11, 20, 33),
    InputStroke = Color3.fromRGB(34, 58, 81),
    PlaceholderColor = Color3.fromRGB(108, 132, 158)
}

local stopUltimate
local startUltimate

local Window = Rayfield:CreateWindow({

    Name = "SPACE HUB  //  ORBITAL",

    Icon = "orbit",

    LoadingTitle = "SPACE HUB",

    LoadingSubtitle =
        "Interfaz Orbital Premium  •  v6.0.0",

    ShowText = "SPACE HUB",

    ToggleUIKeybind = "K",

    Theme = SpaceTheme,

    DisableRayfieldPrompts = false,

    DisableBuildWarnings = false,

    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "SpaceHub"
    },

    Discord = {
        Enabled = false,
        Invite = "",
        RememberJoins = true
    },

    KeySystem = true,

    KeySettings = {

        Title = "SPACE HUB",

        Subtitle = "Orbital Access",

        Note =
            "Enter your Space Hub access key",

        FileName = "SpaceHubKey",

        SaveKey = false,

        GrabKeyFromSite = false,

        Key = {
            "spacehub1254"
        }

    }

})

--//======================================================
--// TABS
--//======================================================

local DashboardTab =
    Window:CreateTab(
        "Dashboard",
        "layout-dashboard"
    )

local UniversalTab =
    Window:CreateTab(
        "Universal",
        "move"
    )

local GameTab =
    Window:CreateTab(
        "Game",
        "crosshair"
    )

local TeleportTab =
    Window:CreateTab(
        "Teleport",
        "map-pin"
    )

local WaypointsTab =
    Window:CreateTab(
        "Waypoints",
        "bookmark"
    )

local PlayerManagerTab =
    Window:CreateTab(
        "Player Manager",
        "users"
    )

local ConfigurationTab =
    Window:CreateTab(
        "Opciones",
        "settings-2"
    )


ConfigurationTab:CreateParagraph({
    Title = "⚙️ OPCIONES / COMMAND SETTINGS",
    Content =
        "Centro de configuración del hub. Los ajustes persistentes se guardan con la configuración de Rayfield."
})

ConfigurationTab:CreateSection("IDIOMA")

--//======================================================
--// LANGUAGE SYSTEM
--//======================================================
local LanguageSystem = {
    Current = "Español",
    Languages = {
        "Español",
        "English",
        "Português",
        "Français",
        "Deutsch",
        "Italiano",
        "Türkçe",
        "Русский",
        "简体中文",
        "日本語"
    }
}

local LanguageNames = {
    ["Español"] = "🇪🇸 Español",
    ["English"] = "🇺🇸 English",
    ["Português"] = "🇧🇷 Português",
    ["Français"] = "🇫🇷 Français",
    ["Deutsch"] = "🇩🇪 Deutsch",
    ["Italiano"] = "🇮🇹 Italiano",
    ["Türkçe"] = "🇹🇷 Türkçe",
    ["Русский"] = "🇷🇺 Русский",
    ["简体中文"] = "🇨🇳 简体中文",
    ["日本語"] = "🇯🇵 日本語"
}

local LanguageText = {
    ["Español"] = {selected="Idioma seleccionado", restart="Algunos textos se aplicarán al reiniciar el hub.", saved="Idioma guardado"},
    ["English"] = {selected="Language selected", restart="Some texts will be applied after restarting the hub.", saved="Language saved"},
    ["Português"] = {selected="Idioma selecionado", restart="Alguns textos serão aplicados ao reiniciar o hub.", saved="Idioma salvo"},
    ["Français"] = {selected="Langue sélectionnée", restart="Certains textes seront appliqués après le redémarrage du hub.", saved="Langue enregistrée"},
    ["Deutsch"] = {selected="Sprache ausgewählt", restart="Einige Texte werden nach dem Neustart des Hubs angewendet.", saved="Sprache gespeichert"},
    ["Italiano"] = {selected="Lingua selezionata", restart="Alcuni testi verranno applicati dopo il riavvio dell'hub.", saved="Lingua salvata"},
    ["Türkçe"] = {selected="Dil seçildi", restart="Bazı metinler hub yeniden başlatıldıktan sonra uygulanır.", saved="Dil kaydedildi"},
    ["Русский"] = {selected="Язык выбран", restart="Некоторые тексты применятся после перезапуска хаба.", saved="Язык сохранён"},
    ["简体中文"] = {selected="已选择语言", restart="部分文本将在重新启动 Hub 后应用。", saved="语言已保存"},
    ["日本語"] = {selected="言語を選択しました", restart="一部のテキストはHub再起動後に適用されます。", saved="言語を保存しました"}
}

local function normalizeLanguage(option)
    local value = typeof(option) == "table" and option[1] or option
    if LanguageText[value] then
        return value
    end
    return "Español"
end

local function getLanguageText(key)
    local pack = LanguageText[LanguageSystem.Current] or LanguageText["Español"]
    return pack[key] or LanguageText["Español"][key] or key
end

local LanguageStatusLabel = ConfigurationTab:CreateLabel(
    "🌐 Idioma actual • Español",
    "languages"
)

ConfigurationTab:CreateDropdown({
    Name = "🌐 Seleccionar Idioma",
    Options = LanguageSystem.Languages,
    CurrentOption = {LanguageSystem.Current},
    MultipleOptions = false,
    Flag = "SpaceHubLanguage",
    Callback = function(option)
        LanguageSystem.Current = normalizeLanguage(option)
        LanguageStatusLabel:Set("🌐 " .. getLanguageText("selected") .. " • " .. LanguageSystem.Current)
        safeNotify(
            getLanguageText("selected"),
            getLanguageText("restart"),
            "languages"
        )
    end
})

ConfigurationTab:CreateParagraph({
    Title = "🌍 IDIOMAS DISPONIBLES",
    Content =
        "Español • English • Português • Français • Deutsch • Italiano • Türkçe • Русский • 简体中文 • 日本語\n" ..
        "El idioma seleccionado queda guardado mediante la configuración del hub."
})

ConfigurationTab:CreateSection("INTERFAZ")
ConfigurationTab:CreateParagraph({
    Title = "TECLA DE INTERFAZ",
    Content =
        "La tecla global de mostrar/ocultar la interfaz es administrada por Rayfield. " ..
        "En esta versión el valor inicial es K; si tu build de Rayfield expone el selector global, " ..
        "úsalo para cambiarla sin modificar el código."
})

ConfigurationTab:CreateSection("SEGURIDAD Y RECUPERACIÓN")
ConfigurationTab:CreateButton({
    Name = "🛑 Parada de Emergencia Global",
    Callback = function()
        pcall(stopBloxAutoFarm)
        pcall(stopUltimate)
        bfTarget = nil
        safeNotify("EMERGENCY STOP", "Automatizaciones principales detenidas.", "circle-stop")
    end
})

ConfigurationTab:CreateButton({
    Name = "🔄 Resincronizar Hub",
    Callback = function()
        pcall(updateCharacter, LocalPlayer.Character)
        pcall(updateSeaAdapter)
        bfTarget = nil
        bfLastTargetSearch = 0
        safeNotify("SYNC", "Estado del hub resincronizado.", "refresh-cw")
    end
})

--//======================================================
--// DIAGNÓSTICO
--//======================================================

local function SpaceHubDiagnostic()
    local problems = {}

    if not Rayfield then
        table.insert(problems, "Rayfield")
    end

    if not LocalPlayer then
        table.insert(problems, "Jugador")
    end

    if not Character then
        table.insert(problems, "Personaje")
    end

    if not workspace then
        table.insert(problems, "Workspace")
    end

    return #problems == 0, problems
end

--//======================================================
--// DASHBOARD
--//======================================================

DashboardTab:CreateParagraph({
    Title = "✦ SPACE HUB  /  COMMAND DECK",
    Content =
        "Live orbital control center for movement, visuals, targeting and transportation.\n" ..
        "All status panels below update automatically."
})

DashboardTab:CreateDivider()
DashboardTab:CreateSection("LIVE SYSTEM STATUS")

local SystemStatusLabel = DashboardTab:CreateLabel("●  SYSTEM ONLINE", "circle-check")
local PlayerStatusLabel = DashboardTab:CreateLabel("Operator  •  " .. LocalPlayer.DisplayName .. "  @" .. LocalPlayer.Name, "user")
local CharacterStatusLabel = DashboardTab:CreateLabel("Character  •  Synchronizing...", "scan")
local RuntimeStatusLabel = DashboardTab:CreateLabel("Runtime  •  Initializing...", "activity")

DashboardTab:CreateDivider()
DashboardTab:CreateSection("LIVE TELEMETRY")

local FPSLabel = DashboardTab:CreateLabel("FPS  •  --", "gauge")
local PingLabel = DashboardTab:CreateLabel("Ping  •  -- ms", "wifi")
local PlayersLabel = DashboardTab:CreateLabel("Players  •  --", "users")
local PositionLabel = DashboardTab:CreateLabel("Position  •  --", "map-pin")

DashboardTab:CreateDivider()
DashboardTab:CreateSection("ACTIVE MODULES")

local MovementStatusLabel = DashboardTab:CreateLabel("Movement  •  STANDBY", "move")
local VisualStatusLabel = DashboardTab:CreateLabel("Visuals  •  STANDBY", "eye")
local TargetStatusLabel = DashboardTab:CreateLabel("Targeting  •  STANDBY", "crosshair")
local PhysicsStatusLabel = DashboardTab:CreateLabel("Physics  •  DEFAULT", "orbit")
local WaypointStatusLabel = DashboardTab:CreateLabel("Waypoints  •  0 SAVED", "bookmark")
local SeaStatusDashboardLabel =
    DashboardTab:CreateLabel(
        "Mar  •  detectando...",
        "waves"
    )

DashboardTab:CreateDivider()
DashboardTab:CreateSection("INTERFACE")

DashboardTab:CreateParagraph({
    Title = "KEYBOARD",
    Content =
        "Tecla de interfaz: configúrala desde Opciones.\n" ..
        "Flight:  W A S D  •  SPACE  •  LEFT CTRL"
})

DashboardTab:CreateButton({
    Name = "Re-Synchronize Character",
    Callback = function()
        if LocalPlayer.Character then
            updateCharacter(LocalPlayer.Character)
            Rayfield:Notify({
                Title = "SYSTEM",
                Content = "Character systems synchronized.",
                Duration = 3,
                Image = "refresh-cw"
            })
        end
    end
})

task.spawn(function()
    local frames = 0
    local last = os.clock()

    RunService.RenderStepped:Connect(function()
        frames += 1
        local now = os.clock()
        if now - last >= 0.5 then
            local fps = math.floor(frames / (now - last) + 0.5)
            frames = 0
            last = now

            local characterReady = Character and Humanoid and HRP and "READY" or "WAITING"
            local hp = Humanoid and math.floor(Humanoid.Health + 0.5) or 0
            local maxHp = Humanoid and math.floor(Humanoid.MaxHealth + 0.5) or 0

            CharacterStatusLabel:Set(
                "Character  •  " .. characterReady ..
                "  •  HP " .. tostring(hp) .. "/" .. tostring(maxHp),
                "scan"
            )

            local ping = "--"
            pcall(function()
                local stats = game:GetService("Stats")
                local network = stats:FindFirstChild("Network")
                local serverStats = network and network:FindFirstChild("ServerStatsItem")
                local dataPing = serverStats and serverStats:FindFirstChild("Data Ping")
                if dataPing then
                    ping = tostring(math.floor(dataPing:GetValue() + 0.5))
                end
            end)

            FPSLabel:Set("FPS  •  " .. tostring(fps), "gauge")
            PingLabel:Set("Ping  •  " .. tostring(ping) .. " ms", "wifi")
            PlayersLabel:Set("Players  •  " .. tostring(#Players:GetPlayers()), "users")

            if HRP then
                local p = HRP.Position
                PositionLabel:Set(
                    string.format("Position  •  X %.1f  Y %.1f  Z %.1f", p.X, p.Y, p.Z),
                    "map-pin"
                )
            else
                PositionLabel:Set("Position  •  --", "map-pin")
            end

            MovementStatusLabel:Set(
                "Movement  •  " ..
                (flying and "FLIGHT ONLINE" or ("WALKSPEED " .. tostring(math.floor(walkSpeed)))),
                "move"
            )

            VisualStatusLabel:Set(
                "Visuals  •  " ..
                (espEnabled and "ESP ONLINE" or "STANDBY"),
                "eye"
            )

            TargetStatusLabel:Set(
                "Targeting  •  " ..
                (aimbotEnabled and ("LOCKED / " .. aimbotPriority) or "STANDBY"),
                "crosshair"
            )

            PhysicsStatusLabel:Set(
                "Physics  •  " ..
                (customGravityEnabled and ("GRAVITY " .. tostring(math.floor(customGravity))) or "DEFAULT"),
                "orbit"
            )

            local waypointCount = 0
            for _ in pairs(waypoints) do
                waypointCount += 1
            end
            WaypointStatusLabel:Set(
                "Waypoints  •  " .. tostring(waypointCount) .. " SAVED",
                "bookmark"
            )

            if SeaStatusDashboardLabel and SeaAdapter then
                SeaStatusDashboardLabel:Set(
                    "Mar  •  " .. tostring(SeaAdapter.Name),
                    "waves"
                )
            end

            RuntimeStatusLabel:Set(
                "Runtime  •  " .. string.format("%.1fs", os.clock()),
                "activity"
            )
        end
    end)
end)

--//======================================================
--// UNIVERSAL HEADER
--//======================================================

UniversalTab:CreateParagraph({

    Title = "✦ ORBITAL CONTROL  /  MOVEMENT",

    Content =
        "Universal movement and player utilities.\n" ..
        "Configure your personal movement systems below."

})

--//======================================================
--// MOVEMENT
--//======================================================

UniversalTab:CreateSection(
    "MOVEMENT  /  CORE"
)

UniversalTab:CreateSlider({

    Name = "WalkSpeed",

    Range = {
        16,
        250
    },

    Increment = 1,

    Suffix = " SPD",

    CurrentValue = 16,

    Flag = "WalkSpeed",

    Callback = function(value)

        walkSpeed = value

        if Humanoid
            and Humanoid.Parent then

            Humanoid.WalkSpeed =
                value

        end

    end

})

--//======================================================
--// FLIGHT
--//======================================================

UniversalTab:CreateSection(
    "MOVEMENT  /  FLIGHT"
)

local function removeFlightObjects()

    if not HRP then
        return
    end

    local velocity =
        HRP:FindFirstChild(
            "SpaceHub_FlightVelocity"
        )

    if velocity then
        velocity:Destroy()
    end

    local attachment =
        HRP:FindFirstChild(
            "SpaceHub_FlightAttachment"
        )

    if attachment then
        attachment:Destroy()
    end

end

local function stopFlying()

    flying = false

    if flyConnection then

        flyConnection:Disconnect()

        flyConnection = nil

    end

    removeFlightObjects()

    if Humanoid then
        Humanoid.PlatformStand = false
    end

end

local function startFlying()

    if not HRP
        or not Humanoid then

        return

    end

    stopFlying()

    flying = true

    local attachment =
        Instance.new("Attachment")

    attachment.Name =
        "SpaceHub_FlightAttachment"

    attachment.Parent =
        HRP

    local velocity =
        Instance.new("LinearVelocity")

    velocity.Name =
        "SpaceHub_FlightVelocity"

    velocity.Attachment0 =
        attachment

    velocity.MaxForce =
        math.huge

    velocity.RelativeTo =
        Enum.ActuatorRelativeTo.World

    velocity.VectorVelocity =
        Vector3.zero

    velocity.Parent =
        HRP

    Humanoid.PlatformStand =
        true

    flyConnection =
        RunService.RenderStepped:Connect(
            function()

                if not flying then
                    return
                end

                if not HRP
                    or not HRP.Parent then

                    stopFlying()

                    return

                end

                local camera =
                    workspace.CurrentCamera

                if not camera then
                    return
                end

                local direction =
                    Vector3.zero

                if UserInputService:IsKeyDown(
                    Enum.KeyCode.W
                ) then

                    direction +=
                        camera.CFrame.LookVector

                end

                if UserInputService:IsKeyDown(
                    Enum.KeyCode.S
                ) then

                    direction -=
                        camera.CFrame.LookVector

                end

                if UserInputService:IsKeyDown(
                    Enum.KeyCode.A
                ) then

                    direction -=
                        camera.CFrame.RightVector

                end

                if UserInputService:IsKeyDown(
                    Enum.KeyCode.D
                ) then

                    direction +=
                        camera.CFrame.RightVector

                end

                if UserInputService:IsKeyDown(
                    Enum.KeyCode.Space
                ) then

                    direction +=
                        Vector3.yAxis

                end

                if UserInputService:IsKeyDown(
                    Enum.KeyCode.LeftControl
                ) then

                    direction -=
                        Vector3.yAxis

                end

                if direction.Magnitude > 0 then

                    direction =
                        direction.Unit
                        * flightSpeed

                end

                velocity.VectorVelocity =
                    direction

            end
        )

end

UniversalTab:CreateSlider({

    Name = "Flight Speed",

    Range = {
        10,
        3000
    },

    Increment = 10,

    Suffix = " SPD",

    CurrentValue = 10,

    Flag = "FlightSpeed",

    Callback = function(value)

        flightSpeed =
            value

    end

})

UniversalTab:CreateToggle({

    Name = "Flight",

    CurrentValue = false,

    Flag = "Flight",

    Callback = function(enabled)

        if enabled then
            startFlying()
        else
            stopFlying()
        end

    end

})

UniversalTab:CreateParagraph({

    Title = "FLIGHT CONTROLS",

    Content =
        "W A S D  →  Navigation\n" ..
        "SPACE  →  Ascend\n" ..
        "LEFT CTRL  →  Descend"

})

--//======================================================
--// PLAYER UTILITIES
--//======================================================

UniversalTab:CreateSection(
    "PLAYER  /  UTILITIES"
)

UniversalTab:CreateToggle({

    Name = "Infinite Jump",

    CurrentValue = false,

    Flag = "InfiniteJump",

    Callback = function(enabled)

        infiniteJump =
            enabled

        if jumpConnection then

            jumpConnection:Disconnect()

            jumpConnection = nil

        end

        if not enabled then
            return
        end

        jumpConnection =
            UserInputService.JumpRequest:Connect(
                function()

                    if Humanoid then

                        Humanoid:ChangeState(
                            Enum.HumanoidStateType.Jumping
                        )

                    end

                end
            )

    end

})

--//======================================================
--// PLAYER CONTROLS
--//======================================================

UniversalTab:CreateSection("PLAYER  /  ADVANCED CONTROLS")

UniversalTab:CreateSlider({
    Name = "JumpPower",
    Range = {0, 250},
    Increment = 1,
    Suffix = " JP",
    CurrentValue = 50,
    Flag = "JumpPower",
    Callback = function(value)
        jumpPower = value
        if Humanoid then
            Humanoid.UseJumpPower = true
            Humanoid.JumpPower = value
        end
    end
})

UniversalTab:CreateSlider({
    Name = "HipHeight",
    Range = {0, 10},
    Increment = 0.1,
    Suffix = " HH",
    CurrentValue = 2,
    Flag = "HipHeight",
    Callback = function(value)
        hipHeight = value
        if Humanoid then
            Humanoid.HipHeight = value
        end
    end
})

UniversalTab:CreateSection("PHYSICS  /  LOCAL")

UniversalTab:CreateToggle({
    Name = "Custom Gravity",
    CurrentValue = false,
    Flag = "CustomGravity",
    Callback = function(enabled)
        customGravityEnabled = enabled
        workspace.Gravity = enabled and customGravity or originalGravity
    end
})

UniversalTab:CreateSlider({
    Name = "Gravity",
    Range = {0, 500},
    Increment = 1,
    Suffix = " G",
    CurrentValue = 196,
    Flag = "Gravity",
    Callback = function(value)
        customGravity = value
        if customGravityEnabled then
            workspace.Gravity = value
        end
    end
})

--//======================================================
--// NOCLIP
--//======================================================

UniversalTab:CreateToggle({

    Name = "Noclip",

    CurrentValue = false,

    Flag = "Noclip",

    Callback = function(enabled)

        noclip =
            enabled

        if noclipConnection then

            noclipConnection:Disconnect()

            noclipConnection = nil

        end

        if not enabled
            and Character then

            for _, part in ipairs(
                Character:GetDescendants()
            ) do

                if part:IsA("BasePart") then
                    part.CanCollide = true
                end

            end

            return

        end

        noclipConnection =
            RunService.Stepped:Connect(
                function()

                    if not noclip
                        or not Character then

                        return

                    end

                    for _, part in ipairs(
                        Character:GetDescendants()
                    ) do

                        if part:IsA("BasePart") then
                            part.CanCollide = false
                        end

                    end

                end
            )

    end

})

--//======================================================
--// FULLBRIGHT
--//======================================================

UniversalTab:CreateToggle({

    Name = "Fullbright",

    CurrentValue = false,

    Flag = "Fullbright",

    Callback = function(enabled)

        fullbright =
            enabled

        if fullbrightConnection then

            fullbrightConnection:Disconnect()

            fullbrightConnection = nil

        end

        if enabled then

            fullbrightConnection =
                RunService.RenderStepped:Connect(
                    function()

                        Lighting.Brightness = 2
                        Lighting.ClockTime = 14
                        Lighting.FogEnd = 100000
                        Lighting.GlobalShadows = false

                    end
                )

        else

            Lighting.Brightness =
                originalLighting.Brightness

            Lighting.ClockTime =
                originalLighting.ClockTime

            Lighting.FogEnd =
                originalLighting.FogEnd

            Lighting.GlobalShadows =
                originalLighting.GlobalShadows

        end

    end

})

--//======================================================
--// GAME HEADER
--//======================================================

GameTab:CreateParagraph({

    Title = "✦ TARGETING & VISUALS  /  COMBAT",

    Content =
        "Advanced player visualization and targeting controls."

})

--//======================================================
--// ESP / ADVANCED VISUALS
--//======================================================

GameTab:CreateSection("VISUALS  /  ADVANCED ESP")

local function getTeamColor(player)
    if espTeamColors and player.Team then
        return player.Team.TeamColor.Color
    end
    return Color3.fromRGB(0, 190, 255)
end

local function removeESP(player)
    local data = espObjects[player]
    if not data then return end
    if data.Highlight then data.Highlight:Destroy() end
    if data.Billboard then data.Billboard:Destroy() end
    espObjects[player] = nil
end

local function createESP(player)
    if player == LocalPlayer then return end
    removeESP(player)
    if not espEnabled then return end

    local character = player.Character
    local head = character and character:FindFirstChild("Head")
    if not character or not head then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "SpaceHub_ESP"
    highlight.Adornee = character
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.FillTransparency = 0.72
    highlight.OutlineTransparency = 0
    highlight.FillColor = getTeamColor(player)
    highlight.OutlineColor = getTeamColor(player)
    highlight.Parent = character

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "SpaceHub_PlayerInfo"
    billboard.Adornee = head
    billboard.Size = UDim2.fromOffset(260, 72)
    billboard.StudsOffset = Vector3.new(0, 3.2, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = head

    local label = Instance.new("TextLabel")
    label.Name = "Info"
    label.Size = UDim2.fromScale(1, 1)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.GothamBold
    label.TextSize = 13
    label.TextWrapped = true
    label.TextColor3 = getTeamColor(player)
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.fromRGB(5, 10, 20)
    label.Parent = billboard

    espObjects[player] = {
        Highlight = highlight,
        Billboard = billboard,
        Label = label
    }
end

local function refreshESP()
    for player in pairs(espObjects) do
        if not player.Parent or not espEnabled or not player.Character then
            removeESP(player)
        end
    end
    if not espEnabled then return end
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            createESP(player)
        end
    end
end

GameTab:CreateToggle({
    Name = "Player ESP",
    CurrentValue = false,
    Flag = "ESP",
    Callback = function(enabled)
        espEnabled = enabled
        refreshESP()
    end
})

GameTab:CreateToggle({
    Name = "Show Name",
    CurrentValue = true,
    Flag = "ESPName",
    Callback = function(value) espShowName = value end
})

GameTab:CreateToggle({
    Name = "Show Health",
    CurrentValue = true,
    Flag = "ESPHealth",
    Callback = function(value) espShowHealth = value end
})

GameTab:CreateToggle({
    Name = "Show Distance",
    CurrentValue = true,
    Flag = "ESPDistance",
    Callback = function(value) espShowDistance = value end
})

GameTab:CreateToggle({
    Name = "Team Colors",
    CurrentValue = true,
    Flag = "ESPTeamColors",
    Callback = function(value)
        espTeamColors = value
        refreshESP()
    end
})

GameTab:CreateSlider({
    Name = "ESP Maximum Distance",
    Range = {100, 5000},
    Increment = 50,
    Suffix = " studs",
    CurrentValue = 1000,
    Flag = "ESPMaxDistance",
    Callback = function(value) espMaxDistance = value end
})

GameTab:CreateButton({
    Name = "Refresh Player Visuals",
    Callback = function()
        refreshESP()
        Rayfield:Notify({
            Title = "VISUAL SYSTEM",
            Content = "Advanced player visuals synchronized.",
            Duration = 3,
            Image = "sparkles"
        })
    end
})

task.spawn(function()
    while task.wait(0.15) do
        if espEnabled then
            for player, data in pairs(espObjects) do
                local character = player.Character
                local root = character and character:FindFirstChild("HumanoidRootPart")
                local humanoid = character and character:FindFirstChildOfClass("Humanoid")

                if not player.Parent or not character or not root or not humanoid then
                    removeESP(player)
                elseif data.Label and data.Highlight then
                    local distance = HRP and (HRP.Position - root.Position).Magnitude or math.huge
                    local visible = distance <= espMaxDistance
                    data.Billboard.Enabled = visible
                    data.Highlight.Enabled = visible
                    data.Label.TextColor3 = getTeamColor(player)

                    local lines = {}
                    if espShowName then
                        table.insert(lines, player.DisplayName .. "  @" .. player.Name)
                    end
                    if espShowHealth then
                        table.insert(lines, "♥ " .. math.floor(humanoid.Health + 0.5) .. " / " .. math.floor(humanoid.MaxHealth + 0.5))
                    end
                    if espShowDistance then
                        table.insert(lines, math.floor(distance) .. " studs")
                    end
                    data.Label.Text = table.concat(lines, "\n")
                end
            end
        end
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        if espEnabled then createESP(player) end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    removeESP(player)
end)

--//======================================================
--// AIMBOT / ADVANCED TARGETING
--//======================================================

GameTab:CreateSection("TARGETING  /  ADVANCED AIM")

local function getTargetPart(character)
    if not character then return nil end

    local names = {
        Head = "Head",
        Torso = "UpperTorso",
        Root = "HumanoidRootPart"
    }

    local preferred = character:FindFirstChild(names[aimbotPart] or "Head")
    if preferred and preferred:IsA("BasePart") then
        return preferred
    end

    return character:FindFirstChild("Head")
        or character:FindFirstChild("HumanoidRootPart")
end

local function isVisible(camera, targetPart, character)
    if not aimbotVisibleCheck then return true end
    local origin = camera.CFrame.Position
    local direction = targetPart.Position - origin

    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {Character, camera}

    local result = workspace:Raycast(origin, direction, params)
    return not result or result.Instance:IsDescendantOf(character)
end

local function getTargetScore(player, camera)
    if not HRP then return nil end
    if player == LocalPlayer then return nil end
    if teamCheck and player.Team == LocalPlayer.Team then return nil end

    local character = player.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    local root = character and character:FindFirstChild("HumanoidRootPart")
    local part = getTargetPart(character)

    if not character or not humanoid or humanoid.Health <= 0 or not root or not part then
        return nil
    end

    local worldDistance = (HRP.Position - root.Position).Magnitude
    if worldDistance > aimbotMaxDistance then return nil end

    local screen, onScreen = camera:WorldToViewportPoint(part.Position)
    if not onScreen then return nil end
    if not isVisible(camera, part, character) then return nil end

    local center = camera.ViewportSize / 2
    local fovDistance = (Vector2.new(screen.X, screen.Y) - center).Magnitude

    if aimbotFOVEnabled and fovDistance > aimbotFOV then
        return nil
    end

    if aimbotPriority == "Closest" then
        return worldDistance
    elseif aimbotPriority == "Lowest Health" then
        return humanoid.Health
    else
        return fovDistance
    end
end

local function getBestTarget(camera)
    local bestPlayer
    local bestScore = math.huge

    for _, player in ipairs(Players:GetPlayers()) do
        local score = getTargetScore(player, camera)
        if score and score < bestScore then
            bestScore = score
            bestPlayer = player
        end
    end

    return bestPlayer
end

local function stopAimbot()
    if aimbotConnection then
        aimbotConnection:Disconnect()
        aimbotConnection = nil
    end
end

local function startAimbot()
    stopAimbot()

    aimbotConnection = RunService.RenderStepped:Connect(function()
        if not aimbotEnabled then return end

        local camera = workspace.CurrentCamera
        if not camera or not HRP then return end

        local target = getBestTarget(camera)
        if not target then return end

        local part = getTargetPart(target.Character)
        if not part then return end

        local targetCFrame = CFrame.lookAt(camera.CFrame.Position, part.Position)
        camera.CFrame = camera.CFrame:Lerp(targetCFrame, aimbotSmoothness)
    end)
end

GameTab:CreateToggle({
    Name = "Aimbot",
    CurrentValue = false,
    Flag = "Aimbot",
    Callback = function(enabled)
        aimbotEnabled = enabled
        if enabled then
            startAimbot()
            Rayfield:Notify({
                Title = "OBJETIVOS",
                Content = "Advanced targeting system online.",
                Duration = 3,
                Image = "crosshair"
            })
        else
            stopAimbot()
        end
    end
})

GameTab:CreateDropdown({
    Name = "Target Part",
    Options = {"Head", "Torso", "Root"},
    CurrentOption = {"Head"},
    MultipleOptions = false,
    Flag = "AimbotPart",
    Callback = function(option)
        aimbotPart = typeof(option) == "table" and option[1] or option
    end
})

GameTab:CreateDropdown({
    Name = "Target Priority",
    Options = {"FOV", "Closest", "Lowest Health"},
    CurrentOption = {"FOV"},
    MultipleOptions = false,
    Flag = "AimbotPriority",
    Callback = function(option)
        aimbotPriority = typeof(option) == "table" and option[1] or option
    end
})

GameTab:CreateToggle({
    Name = "FOV Limiter",
    CurrentValue = true,
    Flag = "AimbotFOVEnabled",
    Callback = function(enabled) aimbotFOVEnabled = enabled end
})

GameTab:CreateSlider({
    Name = "Aimbot FOV",
    Range = {50, 1000},
    Increment = 10,
    Suffix = " PX",
    CurrentValue = 250,
    Flag = "AimbotFOV",
    Callback = function(value) aimbotFOV = value end
})

GameTab:CreateSlider({
    Name = "Maximum Distance",
    Range = {50, 5000},
    Increment = 50,
    Suffix = " studs",
    CurrentValue = 500,
    Flag = "AimbotMaxDistance",
    Callback = function(value) aimbotMaxDistance = value end
})

GameTab:CreateSlider({
    Name = "Smoothness",
    Range = {0.05, 1},
    Increment = 0.05,
    Suffix = "",
    CurrentValue = 0.18,
    Flag = "AimbotSmoothness",
    Callback = function(value) aimbotSmoothness = value end
})

GameTab:CreateToggle({
    Name = "Team Check",
    CurrentValue = false,
    Flag = "TeamCheck",
    Callback = function(enabled) teamCheck = enabled end
})

GameTab:CreateToggle({
    Name = "Visible Check",
    CurrentValue = false,
    Flag = "AimbotVisibleCheck",
    Callback = function(enabled) aimbotVisibleCheck = enabled end
})

GameTab:CreateParagraph({
    Title = "ESTADO DEL OBJETIVO",
    Content =
        "Part  →  " .. aimbotPart .. "\n" ..
        "Priority  →  " .. aimbotPriority .. "\n" ..
        "Range  →  " .. tostring(aimbotMaxDistance) .. " studs\n" ..
        "FOV  →  " .. tostring(aimbotFOV) .. " px"
})


--//======================================================
--// BLOX FRUITS  /  AUTO FARM
--//======================================================

local BloxFruitsTab = Window:CreateTab(
    "Blox Fruits",
    "swords"
)

local HubRuntime = {
    Version = "6.1.0",
    StartedAt = os.clock(),
    Stopped = false,
    FruitNotifier = true,
    PerformanceMode = false,
    LastFruitNotice = {},
    Connections = {},
}

local function safeNotify(title, content, image)
    pcall(function()
        Rayfield:Notify({
            Title = title or "SPACE HUB",
            Content = content or "",
            Duration = 3,
            Image = image or "info"
        })
    end)
end


--//======================================================
--// GESTOR MULTI-MARES
--//======================================================

-- El Blox Fruits oficial actualmente tiene Mar 1, Mar 2 y Mar 3.
-- El perfil Mar 4 queda preparado como compatibilidad futura/personalizada.
local SeaAdapter = {
    AutoDetect = true,
    ManualSea = 0,
    CurrentSea = 0,
    Name = "Desconocido"
}

local SeaProfiles = {
    [1] = {
        Name = "Mar 1",
        SearchDistance = 300,
        FarmDistance = 7
    },

    [2] = {
        Name = "Mar 2",
        SearchDistance = 400,
        FarmDistance = 8
    },

    [3] = {
        Name = "Mar 3",
        SearchDistance = 500,
        FarmDistance = 9
    },

    [4] = {
        Name = "Mar 4 • Futuro",
        SearchDistance = 600,
        FarmDistance = 10
    }
}

-- PlaceIds conocidos de los tres mares oficiales.
local KnownSeaPlaces = {
    [2753915549] = 1,
    [4442272183] = 2,
    [7449423635] = 3
}

local function detectBloxSea()
    local detected = KnownSeaPlaces[game.PlaceId]

    if detected then
        return detected
    end

    -- Compatibilidad con experiencias/actualizaciones que expongan
    -- el mar mediante atributos.
    local candidates = {
        workspace:GetAttribute("Sea"),
        workspace:GetAttribute("World"),
        LocalPlayer:GetAttribute("Sea"),
        LocalPlayer:GetAttribute("World")
    }

    for _, value in ipairs(candidates) do
        if typeof(value) == "number" and value >= 1 and value <= 4 then
            return math.floor(value)
        end

        if typeof(value) == "string" then
            local number = tonumber(value:match("%d+"))

            if number and number >= 1 and number <= 4 then
                return math.floor(number)
            end
        end
    end

    return 0
end

local function updateSeaAdapter()
    local sea

    if SeaAdapter.AutoDetect then
        sea = detectBloxSea()
    else
        sea = SeaAdapter.ManualSea
    end

    SeaAdapter.CurrentSea = sea

    local profile = SeaProfiles[sea]

    SeaAdapter.Name =
        profile and profile.Name
        or "Mar no identificado"

    return sea
end

updateSeaAdapter()

BloxFruitsTab:CreateSection("🌊 MAR / COMPATIBILIDAD")

local SeaStatusLabel = BloxFruitsTab:CreateLabel(
    "Mar detectado • " .. SeaAdapter.Name,
    "waves"
)

BloxFruitsTab:CreateToggle({
    Name = "Detección Automática del Mar",
    CurrentValue = true,
    Flag = "SeaAutoDetect",
    Callback = function(value)
        SeaAdapter.AutoDetect = value
        updateSeaAdapter()

        SeaStatusLabel:Set(
            "Mar detectado • " .. SeaAdapter.Name,
            "waves"
        )
    end
})

BloxFruitsTab:CreateDropdown({
    Name = "Perfil de Mar",
    Options = {
        "AUTO",
        "Mar 1",
        "Mar 2",
        "Mar 3",
        "Mar 4 • Futuro"
    },
    CurrentOption = {"AUTO"},
    MultipleOptions = false,
    Flag = "SeaProfile",
    Callback = function(option)
        local value = option

        if typeof(option) == "table" then
            value = option[1]
        end

        value = value or "AUTO"

        if value == "AUTO" then
            SeaAdapter.AutoDetect = true
        else
            SeaAdapter.AutoDetect = false
            SeaAdapter.ManualSea =
                tonumber(value:match("%d+")) or 0
        end

        updateSeaAdapter()

        SeaStatusLabel:Set(
            "Mar detectado • " .. SeaAdapter.Name,
            "waves"
        )
    end
})

BloxFruitsTab:CreateButton({
    Name = "Actualizar Mar",
    Callback = function()
        updateSeaAdapter()

        local profile = SeaProfiles[SeaAdapter.CurrentSea]

        if profile and SeaAdapter.AutoDetect then
            bfMaxDistance = profile.SearchDistance
            bfFarmDistance = profile.FarmDistance
        end

        SeaStatusLabel:Set(
            "Mar detectado • " .. SeaAdapter.Name,
            "waves"
        )

        Rayfield:Notify({
            Title = "MAR",
            Content = "Perfil activo: " .. SeaAdapter.Name,
            Duration = 3,
            Image = "waves"
        })
    end
})

BloxFruitsTab:CreateParagraph({
    Title = "Compatibilidad",
    Content =
        "El juego oficial actualmente utiliza Mar 1, Mar 2 y Mar 3. " ..
        "El perfil Mar 4 es un adaptador futuro/personalizado; no significa " ..
        "que exista actualmente un cuarto mar oficial."
})


local bfAutoFarm = false
local bfAutoAttack = true
local bfFarmDistance = 7
local bfAttackInterval = 0.12
local bfTargetName = "Nearest"
local bfFarmConnection
local bfTarget
-- Runtime settings are declared before functions that close over them.
local bfAutoHaki = false
local bfBringMobs = false
local bfFastAttack = false
local bfAntiAFK = false
local bfSelectedWeapon = "Auto"
local bfMaxDistance = 250
local bfTargetRefresh = 0.75
local bfLastTargetSearch = 0
local bfLastAttack = 0
local bfFarmState = "IDLE"
local bfStateSince = 0
local bfScanCache = {}
local bfScanCacheAt = 0
local bfScanCacheTTL = 0.35
local bfMovementFailures = 0
local bfLastError = ""
local bfCycleCount = 0
local bfSessionStartedAt = os.clock()
local bfSessionKills = 0
local bfSessionRecoveries = 0
local bfLastState = "IDLE"
local bfSmartMode = false
local bfSmartPriority = "XP"
local bfLowHealthGuard = true
local bfLowHealthPercent = 20
local bfAutoResume = true
local bfAdaptiveInterval = true
local bfBaseScanTTL = 0.35
local bfDynamicScanTTL = 0.35
local bfLastHealth = nil
local bfLastTargetHealth = nil
local bfTargetStartedAt = 0
local bfTargetSwitches = 0
local bfStateChanges = 0


-- Adaptive Engine V5: prioridad, telemetría y recuperación sin crear un segundo controlador.
local bfTargetMode = "Nearest"
local bfPreferBosses = false
local bfLockTarget = false
local bfLastPosition = nil
local bfStuckSince = 0
local bfRecoveryCount = 0
local bfKillsObserved = 0
local bfLastKillAt = 0
local bfControllerHeartbeat = 0
local bfTelemetry = {fps = 0, distance = 0, targetHP = 0, npcCount = 0}
local bfPerformanceMode = false
local bfScanInterval = 0.35

local function setFarmState(state, detail)
    if bfFarmState ~= state then
        bfStateChanges += 1
        bfLastState = state
    end
    bfFarmState = state
    bfStateSince = os.clock()
    if detail then
        bfLastError = tostring(detail)
    end
end

local function farmFail(reason)
    bfMovementFailures = bfMovementFailures + 1
    bfLastError = tostring(reason or "unknown")
    if bfMovementFailures >= 8 then
        setFarmState("RECOVERY", bfLastError)
        bfTarget = nil
        bfMovementFailures = 0
    end
end

BloxFruitsTab:CreateParagraph({
    Title = "✦ BLOX FRUITS  /  AUTO FARM",
    Content =
        "Experimental PvE farming module for Blox Fruits.\nState machine centralizada, caché de NPCs y recuperación automática.\n" ..
        "It searches Workspace.Enemies, moves your character near a living NPC and activates an equipped tool.\n" ..
        "Quest progression is intentionally left manual because quest names and locations change between updates."
})

BloxFruitsTab:CreateSection("AUTO FARM  /  CORE")

local function scanBloxEnemies()
    local enemies = {}
    local seen = {}

    local function addEnemy(model)
        if seen[model] or not model:IsA("Model") then
            return
        end

        local hum = model:FindFirstChildOfClass("Humanoid")
        local root =
            model:FindFirstChild("HumanoidRootPart")
            or model.PrimaryPart

        if hum and root and hum.Health > 0 then
            seen[model] = true
            table.insert(enemies, model)
        end
    end

    local enemiesFolder = workspace:FindFirstChild("Enemies")

    if enemiesFolder then
        for _, model in ipairs(enemiesFolder:GetChildren()) do
            addEnemy(model)
        end

        if #enemies > 0 then
            return enemies
        end
    end

    -- Fallback para layouts personalizados/actualizados.
    -- Se limita a hijos directos de Workspace para evitar recorrer
    -- miles de piezas del mapa.
    for _, object in ipairs(workspace:GetChildren()) do
        addEnemy(object)
    end

    return enemies
end

local function getBloxEnemies(forceRefresh)
    local now = os.clock()
    if not forceRefresh and (now - bfScanCacheAt) < bfScanCacheTTL and #bfScanCache > 0 then
        return bfScanCache
    end
    bfScanCache = scanBloxEnemies()
    bfScanCacheAt = now
    return bfScanCache
end

local function scoreBloxTarget(enemy, root)
    if not enemy or not root then return -math.huge end
    local hum = enemy:FindFirstChildOfClass("Humanoid")
    local enemyRoot = enemy:FindFirstChild("HumanoidRootPart") or enemy.PrimaryPart
    if not hum or not enemyRoot or hum.Health <= 0 then return -math.huge end
    local distance = (root.Position - enemyRoot.Position).Magnitude
    if distance > bfMaxDistance then return -math.huge end

    local name = string.lower(enemy.Name)
    local boss = name:find("boss") ~= nil or hum.MaxHealth >= 5000
    local score = -distance
    if bfPreferBosses and boss then score = score + 1000 end
    if bfTargetMode == "Bosses" and not boss then return -math.huge end
    if bfTargetMode == "Nearest" then return score end
    if bfTargetMode == "Low HP" then
        local hpRatio = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
        return (1 - hpRatio) * 1000 - distance
    end
    return score
end

local function chooseAdaptiveTarget()
    local root = (Character and Character:FindFirstChild("HumanoidRootPart")) or HRP
    if not root then return nil end
    local best, bestScore = nil, -math.huge
    for _, enemy in ipairs(getBloxEnemies()) do
        local score = scoreBloxTarget(enemy, root)
        if score > bestScore then bestScore, best = score, enemy end
    end
    return best
end

local function getNearestBloxEnemy()
    if not HRP then
        return nil
    end

    updateSeaAdapter()

    local profile = SeaProfiles[SeaAdapter.CurrentSea]
    local maxDistance =
        profile and profile.SearchDistance
        or bfMaxDistance

    local nearest
    local nearestDistance = maxDistance

    for _, enemy in ipairs(getBloxEnemies()) do
        local root =
            enemy:FindFirstChild("HumanoidRootPart")
            or enemy.PrimaryPart

        if root then
            local distance = (HRP.Position - root.Position).Magnitude

            if distance < nearestDistance then
                nearestDistance = distance
                nearest = enemy
            end
        end
    end

    return nearest
end

local function equipCombatTool()
    if not Character or not Humanoid then
        return nil
    end

    local equipped = Character:FindFirstChildOfClass("Tool")
    if equipped then
        return equipped
    end

    local backpack = LocalPlayer:FindFirstChildOfClass("Backpack")
    if not backpack then
        return nil
    end

    -- Prefer melee/sword-type tools commonly used in Blox Fruits.
    local preferred
    for _, tool in ipairs(backpack:GetChildren()) do
        if tool:IsA("Tool") then
            local name = string.lower(tool.Name)

            local matchesSelected = true

            if bfSelectedWeapon == "Melee" then
                matchesSelected = name:find("combat") ~= nil or name:find("fighting") ~= nil
            elseif bfSelectedWeapon == "Sword" then
                matchesSelected = name:find("sword") ~= nil or name:find("katana") ~= nil or name:find("blade") ~= nil
            end

            if matchesSelected then
                if name:find("combat") or name:find("fighting")
                    or name:find("sword") or name:find("katana")
                    or name:find("blade") then
                    preferred = tool
                    break
                end
                preferred = preferred or tool
            end
        end
    end

    if preferred then
        Humanoid:EquipTool(preferred)
    end

    return Character:FindFirstChildOfClass("Tool")
end

local function smartDirectorTick()
    if not bfSmartMode or not bfAutoFarm then
        return
    end

    if bfAdaptiveInterval then
        local fps = 60
        -- RenderStepped data is intentionally not required; use a conservative adaptive TTL.
        if HubRuntime.PerformanceMode or bfPerformanceMode then
            bfDynamicScanTTL = math.clamp(bfBaseScanTTL * 2, 0.35, 1.2)
        else
            bfDynamicScanTTL = math.clamp(bfBaseScanTTL, 0.25, 0.8)
        end
        bfScanCacheTTL = bfDynamicScanTTL
    end

    if bfLowHealthGuard and Humanoid and Humanoid.MaxHealth > 0 then
        local percent = (Humanoid.Health / Humanoid.MaxHealth) * 100
        if percent <= bfLowHealthPercent and bfFarmState ~= "RECOVERING" then
            bfLastError = "Protección de vida activa"
            bfTarget = nil
            bfRecoveryCount += 1
            bfSessionRecoveries += 1
            setFarmState("RECOVERING")
            return
        elseif percent > (bfLowHealthPercent + 10) and bfFarmState == "RECOVERING" and bfAutoResume then
            bfLastError = ""
            setFarmState("SEARCHING")
        end
    end

    if bfSmartPriority == "Boss" then
        bfTargetMode = "Bosses"
        bfPreferBosses = true
    elseif bfSmartPriority == "XP" then
        bfTargetMode = "Nearest"
        bfPreferBosses = false
    elseif bfSmartPriority == "Seguridad" then
        bfTargetMode = "Nearest"
        bfPreferBosses = false
        bfFarmDistance = math.max(bfFarmDistance, 10)
    elseif bfSmartPriority == "Equilibrado" then
        bfTargetMode = "Nearest"
        bfPreferBosses = true
    end
end

local function stopBloxAutoFarm()
    -- Single global stop flag used by every Blox Fruits farm loop.
    bfAutoFarm = false
    bfTarget = nil
    bfLastTargetSearch = 0
    bfLastAttack = 0
    bfFarmState = "IDLE"
    bfStateSince = os.clock()
    bfScanCache = {}
    bfScanCacheAt = 0
    bfMovementFailures = 0
    bfStuckSince = 0
    bfLastPosition = nil

    if bfFarmConnection then
        bfFarmConnection:Disconnect()
        bfFarmConnection = nil
    end
end

local function startBloxAutoFarm()
    stopBloxAutoFarm()
    if not HRP or not Humanoid then
        Rayfield:Notify({Title="BLOX FRUITS",Content="El personaje todavía no está listo.",Duration=3,Image="circle-alert"})
        return
    end
    bfAutoFarm = true
    bfLastTargetSearch = 0
    bfTarget = nil
    setFarmState("SEARCHING")
    Rayfield:Notify({Title="BLOX FRUITS",Content="Granja Automática activada • controlador central.",Duration=3,Image="swords"})
end

BloxFruitsTab:CreateToggle({
    Name = "Granja Automática",
    CurrentValue = false,
    Flag = "BloxAutoFarm",
    Callback = function(enabled)
        if enabled then
            startBloxAutoFarm()
        else
            stopBloxAutoFarm()

            Rayfield:Notify({
                Title = "BLOX FRUITS",
                Content = "Granja Automática desactivada.",
                Duration = 2,
                Image = "circle-stop"
            })
        end
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Ataque Automático",
    CurrentValue = true,
    Flag = "BloxAutoAttack",
    Callback = function(enabled)
        bfAutoAttack = enabled
    end
})

BloxFruitsTab:CreateSlider({
    Name = "Distancia de Farmeo",
    Range = {3, 15},
    Increment = 1,
    Suffix = " studs",
    CurrentValue = 7,
    Flag = "BloxFarmDistance",
    Callback = function(value)
        bfFarmDistance = value
    end
})

BloxFruitsTab:CreateSlider({
    Name = "Intervalo de Ataque",
    Range = {0.05, 0.5},
    Increment = 0.05,
    Suffix = " sec",
    CurrentValue = 0.12,
    Flag = "BloxAttackInterval",
    Callback = function(value)
        bfAttackInterval = value
    end
})

BloxFruitsTab:CreateButton({
    Name = "Actualizar Objetivo",
    Callback = function()
        bfTarget = getNearestBloxEnemy()

        Rayfield:Notify({
            Title = "BLOX FRUITS",
            Content =
                bfTarget
                and ("Objetivo  •  " .. bfTarget.Name)
                or "No living NPC found in Workspace.Enemies.",
            Duration = 3,
            Image = "crosshair"
        })
    end
})

local BloxTargetLabel =
    BloxFruitsTab:CreateLabel(
        "Objetivo  •  Ninguno",
        "target"
    )

local BloxStateLabel = BloxFruitsTab:CreateLabel("Estado • IDLE", "activity")
local BloxErrorLabel = BloxFruitsTab:CreateLabel("Diagnóstico • OK", "circle-check")

task.spawn(function()
    while task.wait(0.2) do
        local age = math.floor(os.clock() - bfStateSince)
        BloxStateLabel:Set("Estado • " .. bfFarmState .. "  " .. tostring(age) .. "s", "activity")
        if bfLastError ~= "" then
            BloxErrorLabel:Set("Diagnóstico • " .. bfLastError, "triangle-alert")
        else
            BloxErrorLabel:Set("Diagnóstico • OK  | ciclos " .. tostring(bfCycleCount), "circle-check")
        end
    end
end)

task.spawn(function()
    while task.wait(0.25) do
        if bfAutoFarm then
            if bfTarget
                and bfTarget.Parent then
                local hum =
                    bfTarget:FindFirstChildOfClass("Humanoid")

                BloxTargetLabel:Set(
                    "Objetivo  •  " ..
                    bfTarget.Name ..
                    (hum
                        and ("  •  HP " ..
                            math.floor(hum.Health + 0.5) ..
                            "/" ..
                            math.floor(hum.MaxHealth + 0.5))
                        or ""),
                    "target"
                )
            else
                BloxTargetLabel:Set(
                    "Objetivo  •  Buscando...",
                    "search"
                )
            end
        else
            BloxTargetLabel:Set(
                "Objetivo  •  En espera",
                "target"
            )
        end
    end
end)


--//======================================================
--// BLOX FRUITS / SISTEMAS AVANZADOS
--//======================================================

BloxFruitsTab:CreateSection("SMART CORE  /  DIRECTOR ADAPTATIVO")

local SmartModeLabel = BloxFruitsTab:CreateLabel("Director • MANUAL", "brain")
local SmartStatsLabel = BloxFruitsTab:CreateLabel("Sesión • 0m | objetivos 0 | recuperaciones 0", "chart-no-axes-combined")

BloxFruitsTab:CreateToggle({
    Name = "Director Inteligente",
    CurrentValue = false,
    Flag = "SmartDirector",
    Callback = function(value)
        bfSmartMode = value
        SmartModeLabel:Set("Director • " .. (value and "AUTO-ADAPTATIVO" or "MANUAL"), value and "brain" or "hand")
        if value then
            safeNotify("SMART CORE", "El director adaptativo está supervisando prioridad, rendimiento y recuperación.", "brain")
        end
    end
})

BloxFruitsTab:CreateDropdown({
    Name = "Prioridad del Director",
    Options = {"XP", "Boss", "Seguridad", "Equilibrado"},
    CurrentOption = {"XP"},
    MultipleOptions = false,
    Flag = "SmartPriority",
    Callback = function(value)
        bfSmartPriority = value
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Protección de Vida",
    CurrentValue = true,
    Flag = "SmartLowHealthGuard",
    Callback = function(value)
        bfLowHealthGuard = value
    end
})

BloxFruitsTab:CreateSlider({
    Name = "Vida mínima de seguridad",
    Range = {5, 60},
    Increment = 5,
    Suffix = "%",
    CurrentValue = 20,
    Flag = "SmartLowHealthPercent",
    Callback = function(value)
        bfLowHealthPercent = value
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Reanudar después de recuperación",
    CurrentValue = true,
    Flag = "SmartAutoResume",
    Callback = function(value)
        bfAutoResume = value
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Optimización dinámica",
    CurrentValue = true,
    Flag = "SmartAdaptiveInterval",
    Callback = function(value)
        bfAdaptiveInterval = value
    end
})

BloxFruitsTab:CreateParagraph({
    Title = "DIRECTOR ADAPTATIVO",
    Content =
        "El núcleo observa estado, vida, objetivo, recuperaciones y rendimiento. " ..
        "No crea un segundo Auto Farm: modifica las prioridades del controlador existente."
})

BloxFruitsTab:CreateSection("ADAPTIVE ENGINE  /  INTELIGENCIA")

BloxFruitsTab:CreateDropdown({
    Name = "Estrategia de Objetivos",
    Options = {"Nearest", "Low HP", "Bosses"},
    CurrentOption = {"Nearest"},
    MultipleOptions = false,
    Flag = "BloxTargetMode",
    Callback = function(option)
        bfTargetMode = typeof(option) == "table" and (option[1] or "Nearest") or (option or "Nearest")
        bfTarget = nil
        if bfAutoFarm then setFarmState("SEARCHING") end
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Priorizar Bosses",
    CurrentValue = false,
    Flag = "BloxPreferBosses",
    Callback = function(value)
        bfPreferBosses = value
        bfTarget = nil
        if bfAutoFarm then setFarmState("SEARCHING") end
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Bloquear Objetivo",
    CurrentValue = false,
    Flag = "BloxLockTarget",
    Callback = function(value)
        bfLockTarget = value
    end
})

local BloxTelemetryLabel = BloxFruitsTab:CreateLabel("Telemetría • iniciando...", "activity")
BloxFruitsTab:CreateParagraph({
    Title = "Motor Adaptativo V5",
    Content = "El sistema puntúa objetivos por distancia, HP y tipo de enemigo. Recalcula cuando un objetivo desaparece, muere o queda inválido. El modo rendimiento reduce el coste de escaneo."
})

BloxFruitsTab:CreateSection("COMBATE  /  SEGURIDAD")

local function getCharacterRoot()
    if Character and Character.Parent then
        return Character:FindFirstChild("HumanoidRootPart")
    end
end

local function isAlive()
    return Humanoid and Humanoid.Parent and Humanoid.Health > 0
end

local function activateTool()
    local tool = Character and Character:FindFirstChildOfClass("Tool")
    if not tool then
        tool = equipCombatTool()
    end

    if tool then
        pcall(function()
            tool:Activate()
        end)
    end

    return tool
end

local lastHakiAttempt = 0

local function enableHaki()
    if not bfAutoHaki then
        return
    end

    if os.clock() - lastHakiAttempt < 2 then
        return
    end

    lastHakiAttempt = os.clock()

    pcall(function()
        local vim = game:GetService("VirtualInputManager")
        vim:SendKeyEvent(true, Enum.KeyCode.J, false, game)
        task.wait(0.05)
        vim:SendKeyEvent(false, Enum.KeyCode.J, false, game)
    end)
end

BloxFruitsTab:CreateToggle({
    Name = "Haki Automático",
    CurrentValue = false,
    Flag = "BloxAutoHaki",
    Callback = function(enabled)
        bfAutoHaki = enabled
        if enabled then
            enableHaki()
        end
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Ataque Rápido",
    CurrentValue = false,
    Flag = "BloxFastAttack",
    Callback = function(enabled)
        bfFastAttack = enabled
        if enabled then
            bfAttackInterval = math.min(bfAttackInterval, 0.08)
        end
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Atraer Enemigos",
    CurrentValue = false,
    Flag = "BloxBringMobs",
    Callback = function(enabled)
        bfBringMobs = enabled
    end
})

BloxFruitsTab:CreateDropdown({
    Name = "Arma",
    Options = {
        "Auto",
        "Melee",
        "Sword"
    },
    CurrentOption = {"Auto"},
    MultipleOptions = false,
    Flag = "BloxWeapon",
    Callback = function(option)
        if typeof(option) == "table" then
            bfSelectedWeapon = option[1] or "Auto"
        else
            bfSelectedWeapon = option or "Auto"
        end
    end
})

BloxFruitsTab:CreateSlider({
    Name = "Distancia de Búsqueda",
    Range = {50, 2500},
    Increment = 50,
    Suffix = " studs",
    CurrentValue = 250,
    Flag = "BloxTargetDistance",
    Callback = function(value)
        bfMaxDistance = value
    end
})

BloxFruitsTab:CreateSlider({
    Name = "Actualización del Objetivo",
    Range = {0.25, 3},
    Increment = 0.25,
    Suffix = " sec",
    CurrentValue = 0.75,
    Flag = "BloxTargetRefresh",
    Callback = function(value)
        bfTargetRefresh = value
    end
})

local function findNearestSafeEnemy()
    return chooseAdaptiveTarget()
end
-- Centralized state machine: one controller owns target acquisition, movement and attack.
task.spawn(function()
    while task.wait(0.04) do
        pcall(smartDirectorTick)
        if not bfAutoFarm then
            if bfFarmState ~= "IDLE" then setFarmState("IDLE") end
            continue
        end

        if not isAlive() or not getCharacterRoot() then
            setFarmState("WAIT_CHARACTER")
            continue
        end

        local now = os.clock()
        if bfFarmState == "IDLE" or bfFarmState == "WAIT_CHARACTER" or bfFarmState == "RECOVERY" then
            bfTarget = nil
            bfScanCacheAt = 0
            bfMovementFailures = 0
            setFarmState("SEARCHING")
        elseif bfFarmState == "SEARCHING" then
            if now - bfLastTargetSearch >= bfTargetRefresh then
                bfTarget = (bfLockTarget and bfTarget and bfTarget.Parent) and bfTarget or findNearestSafeEnemy()
                bfLastTargetSearch = now
                if bfTarget then
                    bfCycleCount = bfCycleCount + 1
                    bfTargetSwitches += 1
                    bfTargetStartedAt = now
                    setFarmState("MOVING")
                end
            end
        elseif bfFarmState == "MOVING" then
            if not bfTarget or not bfTarget.Parent then
                setFarmState("SEARCHING")
                continue
            end
            local enemyHumanoid = bfTarget:FindFirstChildOfClass("Humanoid")
            local enemyRoot = bfTarget:FindFirstChild("HumanoidRootPart") or bfTarget.PrimaryPart
            local root = getCharacterRoot()
            if not enemyHumanoid or enemyHumanoid.Health <= 0 or not enemyRoot then
                bfTarget = nil
                setFarmState("SEARCHING")
                continue
            end
            local profile = SeaProfiles[SeaAdapter.CurrentSea]
            local activeDistance = profile and profile.FarmDistance or bfFarmDistance
            local distance = (root.Position - enemyRoot.Position).Magnitude
            if distance > bfMaxDistance then
                bfTarget = nil
                setFarmState("SEARCHING")
                continue
            end
            local previousPosition = bfLastPosition
            local ok = pcall(function()
                root.CFrame = CFrame.lookAt(enemyRoot.Position - enemyRoot.CFrame.LookVector * activeDistance, enemyRoot.Position)
            end)
            bfLastPosition = root.Position
            if previousPosition and (root.Position - previousPosition).Magnitude < 0.05 then
                if bfStuckSince == 0 then bfStuckSince = now end
                if now - bfStuckSince > 2 then
                    bfRecoveryCount = bfRecoveryCount + 1
                    bfTarget = nil
                    bfStuckSince = 0
                    setFarmState("RECOVERY", "Movimiento bloqueado; reintentando")
                    continue
                end
            else
                bfStuckSince = 0
            end
            if ok then
                bfMovementFailures = 0
                setFarmState("ATTACKING")
            else
                farmFail("No se pudo posicionar el personaje")
            end
        elseif bfFarmState == "ATTACKING" then
            if not bfTarget or not bfTarget.Parent then
                bfTarget = nil
                setFarmState("SEARCHING")
                continue
            end
            local enemyHumanoid = bfTarget:FindFirstChildOfClass("Humanoid")
            local enemyRoot = bfTarget:FindFirstChild("HumanoidRootPart") or bfTarget.PrimaryPart
            local root = getCharacterRoot()
            if not enemyHumanoid or enemyHumanoid.Health <= 0 or not enemyRoot or not root then
                if enemyHumanoid and enemyHumanoid.Health <= 0 then
                    bfKillsObserved += 1
                    bfSessionKills += 1
                    bfLastKillAt = now
                end
                bfTarget = nil
                setFarmState("SEARCHING")
                continue
            end
            local distance = (root.Position - enemyRoot.Position).Magnitude
            if distance > (bfFarmDistance + 12) then
                setFarmState("MOVING")
                continue
            end
            enableHaki()
            if bfBringMobs then
                for _, other in ipairs(getBloxEnemies()) do
                    if other ~= bfTarget then
                        local otherRoot = other:FindFirstChild("HumanoidRootPart") or other.PrimaryPart
                        local otherHum = other:FindFirstChildOfClass("Humanoid")
                        if otherRoot and otherHum and otherHum.Health > 0 and (otherRoot.Position-enemyRoot.Position).Magnitude < 35 then
                            pcall(function() otherRoot.CFrame = enemyRoot.CFrame * CFrame.new(0,0,4) end)
                        end
                    end
                end
            end
            if bfAutoAttack and now - bfLastAttack >= bfAttackInterval then
                bfLastAttack = now
                local ok = pcall(activateTool)
                if not ok then farmFail("Fallo al activar herramienta") end
            end
            if enemyHumanoid.Health <= 0 then
                bfTarget = nil
                setFarmState("SEARCHING")
            end
        end
    end
end)


task.spawn(function()
    while task.wait(bfPerformanceMode and 1 or 0.5) do
        local root = getCharacterRoot()
        local target = bfTarget
        local distance = 0
        local hp = 0
        if root and target and target.Parent then
            local tr = target:FindFirstChild("HumanoidRootPart") or target.PrimaryPart
            local th = target:FindFirstChildOfClass("Humanoid")
            if tr then distance = (root.Position - tr.Position).Magnitude end
            if th then hp = math.floor(th.Health + 0.5) end
        end
        bfTelemetry.distance = distance
        bfTelemetry.targetHP = hp
        bfTelemetry.npcCount = #getBloxEnemies()
        BloxTelemetryLabel:Set(
            "Telemetría • " .. bfFarmState ..
            " | NPC " .. tostring(bfTelemetry.npcCount) ..
            " | Dist " .. tostring(math.floor(distance)) ..
            " | HP " .. tostring(hp) ..
            " | REC " .. tostring(bfRecoveryCount),
            "activity"
        )
    end
end)

BloxFruitsTab:CreateSection("JUGADOR  /  CALIDAD DE VIDA")

BloxFruitsTab:CreateToggle({
    Name = "Anti-Inactividad",
    CurrentValue = false,
    Flag = "BloxAntiAFK",
    Callback = function(enabled)
        bfAntiAFK = enabled
    end
})

-- Anti-AFK without changing the player's movement settings.
local VirtualUser = game:GetService("VirtualUser")

LocalPlayer.Idled:Connect(function()
    if bfAntiAFK then
        pcall(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new(0, 0))
        end)
    end
end)

BloxFruitsTab:CreateButton({
    Name = "Volver a Buscar NPC",
    Callback = function()
        bfTarget = findNearestSafeEnemy()

        Rayfield:Notify({
            Title = "BLOX FRUITS",
            Content =
                bfTarget
                and ("Nuevo objetivo  •  " .. bfTarget.Name)
                or "No se encontró ningún NPC válido.",
            Duration = 3,
            Image = "target"
        })
    end
})

BloxFruitsTab:CreateButton({
    Name = "Detener Todo Blox Fruits",
    Callback = function()
        stopBloxAutoFarm()
        bfAutoHaki = false
        bfBringMobs = false
        bfFastAttack = false
        bfTarget = nil
        bfAntiAFK = false

        Rayfield:Notify({
            Title = "BLOX FRUITS",
            Content = "Toda la automatización de Blox Fruits fue detenida.",
            Duration = 3,
            Image = "circle-stop"
        })
    end
})

BloxFruitsTab:CreateParagraph({
    Title = "IMPORTANTE",
    Content =
        "The advanced module intentionally avoids direct server-side RemoteEvent abuse. " ..
        "It uses the game's visible NPCs, character movement and equipped Tool activation. " ..
        "Quest progression, fruit purchasing and server hopping are not forced automatically."
})

--//======================================================
--// BLOX FRUITS / MONITOR Y UTILIDADES
--//======================================================

BloxFruitsTab:CreateSection("MONITOR  /  BÚSQUEDAS")

local BloxMonitorLabel = BloxFruitsTab:CreateLabel("Monitor • listo", "activity")
local BloxBossLabel = BloxFruitsTab:CreateLabel("Boss • ninguno detectado", "crown")
local BloxFruitLabel = BloxFruitsTab:CreateLabel("Fruta • esperando aparición", "apple")

local function findBosses()
    local bosses = {}
    for _, enemy in ipairs(getBloxEnemies()) do
        local hum = enemy:FindFirstChildOfClass("Humanoid")
        local name = string.lower(enemy.Name)
        if hum and hum.Health > 0 and (name:find("boss") or hum.MaxHealth >= 5000) then
            table.insert(bosses, enemy)
        end
    end
    table.sort(bosses, function(a, b)
        local ar = a:FindFirstChild("HumanoidRootPart") or a.PrimaryPart
        local br = b:FindFirstChild("HumanoidRootPart") or b.PrimaryPart
        if not HRP or not ar or not br then return false end
        return (HRP.Position-ar.Position).Magnitude < (HRP.Position-br.Position).Magnitude
    end)
    return bosses
end

BloxFruitsTab:CreateButton({
    Name = "Buscar Bosses Cercanos",
    Callback = function()
        local bosses = findBosses()
        if #bosses > 0 then
            local boss = bosses[1]
            BloxBossLabel:Set("Boss • " .. boss.Name, "crown")
            safeNotify("BOSS SCANNER", "Detectado: " .. boss.Name, "crown")
        else
            BloxBossLabel:Set("Boss • ninguno detectado", "search-x")
            safeNotify("BOSS SCANNER", "No se detectaron bosses válidos cerca.", "search-x")
        end
    end
})

BloxFruitsTab:CreateButton({
    Name = "Escanear NPCs Ahora",
    Callback = function()
        local enemies = getBloxEnemies()
        BloxMonitorLabel:Set("Monitor • " .. tostring(#enemies) .. " NPCs vivos", "scan")
        safeNotify("NPC SCANNER", "NPCs válidos encontrados: " .. tostring(#enemies), "scan")
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Notificador de Frutas",
    CurrentValue = true,
    Flag = "FruitNotifier",
    Callback = function(value)
        HubRuntime.FruitNotifier = value
    end
})

BloxFruitsTab:CreateToggle({
    Name = "Modo Rendimiento",
    CurrentValue = false,
    Flag = "PerformanceMode",
    Callback = function(value)
        HubRuntime.PerformanceMode = value
        if value then
            safeNotify("RENDIMIENTO", "Escaneos pesados reducidos. ESP y automatizaciones siguen bajo tu control.", "gauge")
        else
            safeNotify("RENDIMIENTO", "Modo rendimiento desactivado.", "gauge")
        end
    end
})

task.spawn(function()
    while task.wait(HubRuntime.PerformanceMode and 1.5 or 0.5) do
        if HubRuntime.PerformanceMode then
            -- Deliberately avoid full Workspace scans here.
            if bfAutoFarm and bfTarget then
                BloxMonitorLabel:Set("Monitor • farmeando " .. bfTarget.Name, "activity")
            else
                BloxMonitorLabel:Set("Monitor • rendimiento activo", "gauge")
            end
        elseif bfAutoFarm then
            local count = #getBloxEnemies()
            BloxMonitorLabel:Set("Monitor • " .. tostring(count) .. " NPCs visibles", "activity")
        else
            BloxMonitorLabel:Set("Monitor • en espera", "activity")
        end
    end
end)

-- Detects newly spawned fruit-like objects without polling the whole map.
workspace.DescendantAdded:Connect(function(obj)
    if not HubRuntime.FruitNotifier or not obj then return end
    local name = string.lower(obj.Name or "")
    if not (name:find("fruit") or name:find("fruta")) then return end
    if not (obj:IsA("Tool") or obj:IsA("Model") or obj:IsA("BasePart")) then return end

    local now = os.clock()
    local last = HubRuntime.LastFruitNotice[name] or 0
    if now - last < 10 then return end
    HubRuntime.LastFruitNotice[name] = now

    BloxFruitLabel:Set("Fruta • " .. obj.Name, "apple")
    safeNotify("FRUTA DETECTADA", obj.Name, "apple")
end)

BloxFruitsTab:CreateParagraph({
    Title = "DIAGNÓSTICO",
    Content =
        "El monitor usa escaneos acotados y eventos cuando es posible. " ..
        "Si un sistema deja de responder, usa la Parada de Emergencia y vuelve a sincronizar el personaje."
})

BloxFruitsTab:CreateButton({
    Name = "Reiniciar Estado del Farm",
    Callback = function()
        bfTarget = nil
        bfScanCache = {}
        bfScanCacheAt = 0
        bfLastTargetSearch = 0
        bfMovementFailures = 0
        bfLastError = ""
        setFarmState(bfAutoFarm and "SEARCHING" or "IDLE")
        safeNotify("AUTO FARM", "Estado interno reiniciado.", "refresh-cw")
    end
})

BloxFruitsTab:CreateButton({
    Name = "Diagnóstico Completo",
    Callback = function()
        local enemies = getBloxEnemies(true)
        local alive = isAlive() and "sí" or "no"
        safeNotify("DIAGNÓSTICO", "Mar: " .. SeaAdapter.Name .. " | NPC: " .. tostring(#enemies) .. " | Vivo: " .. alive .. " | Estado: " .. bfFarmState, "stethoscope")
    end
})

--//======================================================
--// BLOX FRUITS / SOPORTE DE RENACIMIENTO
--//======================================================

LocalPlayer.CharacterAdded:Connect(function()
    pcall(updateSeaAdapter)
    if bfAutoFarm then
        task.wait(1)
        bfTarget = nil
    end
end)

--//======================================================
--// SMART PROFILES
--//======================================================

local SmartProfilesTab = Window:CreateTab(
    "Perfiles",
    "sliders-horizontal"
)

SmartProfilesTab:CreateParagraph({
    Title = "✦ SPACE HUB / SMART PROFILES",
    Content =
        "Perfiles rápidos para cambiar la filosofía del hub sin tocar decenas de opciones. " ..
        "Cada perfil utiliza los módulos ya existentes."
})

local ProfileLabel = SmartProfilesTab:CreateLabel("Perfil • Personalizado", "layers")

local function applySmartProfile(name)
    if name == "XP RÁPIDO" then
        bfTargetMode = "Nearest"
        bfPreferBosses = false
        bfFarmDistance = 7
        bfAttackInterval = 0.12
        bfPerformanceMode = false
        ProfileLabel:Set("Perfil • XP RÁPIDO", "zap")
    elseif name == "BOSS" then
        bfTargetMode = "Bosses"
        bfPreferBosses = true
        bfFarmDistance = 9
        bfAttackInterval = 0.14
        ProfileLabel:Set("Perfil • BOSS", "crown")
    elseif name == "SEGURO" then
        bfTargetMode = "Nearest"
        bfPreferBosses = false
        bfFarmDistance = 11
        bfAttackInterval = 0.20
        bfPerformanceMode = true
        bfLowHealthGuard = true
        ProfileLabel:Set("Perfil • SEGURO", "shield-check")
    elseif name == "RENDIMIENTO" then
        bfTargetMode = "Nearest"
        bfPreferBosses = false
        bfFarmDistance = 8
        bfAttackInterval = 0.16
        bfPerformanceMode = true
        bfAdaptiveInterval = true
        ProfileLabel:Set("Perfil • RENDIMIENTO", "gauge")
    end
    bfLastTargetSearch = 0
    bfTarget = nil
    safeNotify("PERFIL", "Aplicado: " .. name, "layers")
end

SmartProfilesTab:CreateButton({Name = "⚡ XP RÁPIDO", Callback = function() applySmartProfile("XP RÁPIDO") end})
SmartProfilesTab:CreateButton({Name = "👑 BOSS", Callback = function() applySmartProfile("BOSS") end})
SmartProfilesTab:CreateButton({Name = "🛡️ SEGURO", Callback = function() applySmartProfile("SEGURO") end})
SmartProfilesTab:CreateButton({Name = "📈 RENDIMIENTO", Callback = function() applySmartProfile("RENDIMIENTO") end})

SmartProfilesTab:CreateSection("ANALÍTICA DE SESIÓN")
local SessionLabel = SmartProfilesTab:CreateLabel("Sesión • iniciando...", "timer")
local SessionTargetLabel = SmartProfilesTab:CreateLabel("Objetivos • 0", "target")
local SessionRecoveryLabel = SmartProfilesTab:CreateLabel("Recuperaciones • 0", "rotate-ccw")

SmartProfilesTab:CreateButton({
    Name = "Restablecer estadísticas de sesión",
    Callback = function()
        bfSessionStartedAt = os.clock()
        bfSessionKills = 0
        bfKillsObserved = 0
        bfSessionRecoveries = 0
        bfTargetSwitches = 0
        safeNotify("SESIÓN", "Estadísticas reiniciadas.", "rotate-ccw")
    end
})

task.spawn(function()
    while task.wait(1) do
        local mins = math.floor((os.clock() - bfSessionStartedAt) / 60)
        SessionLabel:Set("Sesión • " .. tostring(mins) .. " min", "timer")
        SessionTargetLabel:Set("Objetivos • " .. tostring(bfTargetSwitches) .. " | bajas " .. tostring(bfKillsObserved), "target")
        SessionRecoveryLabel:Set("Recuperaciones • " .. tostring(bfRecoveryCount) .. " | estado " .. tostring(bfFarmState), "rotate-ccw")
        SmartStatsLabel:Set(
            "Sesión • " .. tostring(mins) .. "m | objetivos " .. tostring(bfTargetSwitches) ..
            " | bajas " .. tostring(bfKillsObserved) .. " | recuperaciones " .. tostring(bfRecoveryCount),
            "chart-no-axes-combined"
        )
    end
end)

--//======================================================
--// TELEPORT
--//======================================================

TeleportTab:CreateParagraph({

    Title = "✦ ORBITAL TELEPORT  /  TRANSPORT",

    Content =
        "Player and world transportation systems."

})

--//======================================================
--// PLAYER DESTINATIONS
--//======================================================

TeleportTab:CreateSection(
    "TELEPORT  /  PLAYERS"
)

local function teleportToPlayer(player)

    if not HRP then
        return
    end

    local character =
        player.Character

    if not character then
        return
    end

    local targetHRP =
        character:FindFirstChild(
            "HumanoidRootPart"
        )

    if not targetHRP then
        return
    end

    HRP.CFrame =
        targetHRP.CFrame
        * CFrame.new(
            3,
            0,
            0
        )

    Rayfield:Notify({

        Title = "TELEPORT",

        Content =
            "Arrived near "
            .. player.DisplayName,

        Duration = 2,

        Image = "sparkles"

    })

end

local function createTeleportButton(player)

    if player == LocalPlayer then
        return
    end

    TeleportTab:CreateButton({

        Name =
            "TP  •  "
            .. player.Name,

        Callback = function()

            teleportToPlayer(
                player
            )

        end

    })

end

for _, player in ipairs(
    Players:GetPlayers()
) do

    createTeleportButton(player)

end

Players.PlayerAdded:Connect(
    function(player)

        task.wait(0.5)

        createTeleportButton(player)

    end
)

--//======================================================
--// SYSTEM DESTINATIONS
--//======================================================

TeleportTab:CreateSection(
    "TELEPORT  /  SYSTEM"
)

TeleportTab:CreateButton({

    Name = "Teleport To Spawn",

    Callback = function()

        if not HRP then
            return
        end

        local spawn =
            workspace:FindFirstChild(
                "SpawnLocation",
                true
            )

        if spawn
            and spawn:IsA("BasePart") then

            HRP.CFrame =
                spawn.CFrame
                * CFrame.new(
                    0,
                    5,
                    0
                )

        else

            Rayfield:Notify({

                Title = "TELEPORT",

                Content =
                    "No SpawnLocation found.",

                Duration = 3,

                Image = "sparkles"

            })

        end

    end

})

--//======================================================
--// MASS TELEPORT
--//======================================================

TeleportTab:CreateSection(
    "TRANSPORT  /  MASS"
)

TeleportTab:CreateParagraph({

    Title = "◈ MASS TRANSPORT",

    Content =
        "Searches Workspace for valid humanoid models.\n" ..
        "Models with Humanoid + HumanoidRootPart will be moved near you."

})

local function teleportAllHumanoids()

    if not HRP then

        Rayfield:Notify({

            Title = "TRANSPORT  /  MASS",

            Content =
                "Your character is not ready.",

            Duration = 3,

            Image = "sparkles"

        })

        return

    end

    local moved = 0

    local baseCFrame =
        HRP.CFrame

    for _, object in ipairs(
        workspace:GetDescendants()
    ) do

        if object:IsA("Model")
            and object ~= Character then

            local humanoid =
                object:FindFirstChildOfClass(
                    "Humanoid"
                )

            local root =
                object:FindFirstChild(
                    "HumanoidRootPart"
                )

            if humanoid
                and root
                and root:IsA("BasePart")
                and humanoid.Health > 0 then

                local angle =
                    moved
                    * math.rad(45)

                local radius =
                    5
                    + (
                        (moved % 4)
                        * 2
                    )

                local offset =
                    Vector3.new(
                        math.cos(angle)
                        * radius,

                        0,

                        math.sin(angle)
                        * radius
                    )

                pcall(function()

                    root.CFrame =
                        CFrame.new(
                            baseCFrame.Position
                            + offset
                        )
                        * CFrame.Angles(
                            0,
                            baseCFrame:
                                ToEulerAnglesYXZ()
                        )

                    moved += 1

                end)

            end

        end

    end

    Rayfield:Notify({

        Title = "TRANSPORT  /  MASS",

        Content =
            "Transported "
            .. tostring(moved)
            .. " humanoid(s).",

        Duration = 4,

        Image = "sparkles"

    })

end

TeleportTab:CreateButton({

    Name =
        "Teleport All Humanoids",

    Callback = function()

        teleportAllHumanoids()

    end

})

--//======================================================
--// MASS TELEPORT - PLAYERS ONLY
--//======================================================

TeleportTab:CreateButton({

    Name =
        "Teleport All Players",

    Callback = function()

        if not HRP then
            return
        end

        local baseCFrame =
            HRP.CFrame

        local moved = 0

        for _, player in ipairs(
            Players:GetPlayers()
        ) do

            if player ~= LocalPlayer then

                local character =
                    player.Character

                if character then

                    local humanoid =
                        character:FindFirstChildOfClass(
                            "Humanoid"
                        )

                    local root =
                        character:FindFirstChild(
                            "HumanoidRootPart"
                        )

                    if humanoid
                        and root
                        and humanoid.Health > 0 then

                        local angle =
                            moved
                            * math.rad(45)

                        local radius =
                            5
                            + (
                                (moved % 4)
                                * 2
                            )

                        local offset =
                            Vector3.new(
                                math.cos(angle)
                                * radius,

                                0,

                                math.sin(angle)
                                * radius
                            )

                        pcall(function()

                            root.CFrame =
                                CFrame.new(
                                    baseCFrame.Position
                                    + offset
                                )

                        end)

                        moved += 1

                    end

                end

            end

        end

        Rayfield:Notify({

            Title = "TRANSPORT  /  MASS",

            Content =
                "Transported "
                .. tostring(moved)
                .. " player(s).",

            Duration = 4,

            Image = "sparkles"

        })

    end

})

--//======================================================
--// PHYSICS CONTROL
--//======================================================

TeleportTab:CreateSection(
    "PHYSICS  /  CONTROL"
)

TeleportTab:CreateParagraph({

    Title = "◈ ANCHORED OBJECTS",

    Content =
        "Releases every anchored BasePart in Workspace.\n" ..
        "Anchored objects will become physically movable."

})

TeleportTab:CreateButton({

    Name = "Unanchor All Objects",

    Callback = function()

        local count = 0

        for _, object in ipairs(
            workspace:GetDescendants()
        ) do

            if object:IsA("BasePart")
                and object.Anchored then

                local success = pcall(function()

                    object.Anchored = false

                end)

                if success then
                    count += 1
                end

            end

        end

        Rayfield:Notify({

            Title = "PHYSICS  /  CONTROL",

            Content =
                "Released "
                .. tostring(count)
                .. " anchored object(s).",

            Duration = 4,

            Image = "sparkles"

        })

    end

})

--//======================================================
--// WAYPOINTS
--//======================================================

WaypointsTab:CreateParagraph({
    Title = "✦ WAYPOINT NETWORK  /  POSITION MANAGER",
    Content =
        "Save locations, return to previous positions and manage your personal navigation points."
})

local waypointFile = "SpaceHub_Waypoints.json"

local function saveWaypointsToDisk()
    if not (writefile and readfile and isfile) then return false end
    local ok = pcall(function()
        writefile(waypointFile, HttpService:JSONEncode(waypoints))
    end)
    return ok
end

local function loadWaypointsFromDisk()
    if not (writefile and readfile and isfile) then return end
    if not isfile(waypointFile) then return end

    pcall(function()
        local decoded = HttpService:JSONDecode(readfile(waypointFile))
        if typeof(decoded) == "table" then
            waypoints = decoded
        end
    end)
end

local function waypointNames()
    local names = {}
    for name in pairs(waypoints) do
        table.insert(names, name)
    end
    table.sort(names)
    if #names == 0 then
        names = {"No waypoints"}
    end
    return names
end

local WaypointDropdown = WaypointsTab:CreateDropdown({
    Name = "Selected Waypoint",
    Options = waypointNames(),
    CurrentOption = {waypointNames()[1]},
    MultipleOptions = false,
    Flag = "SelectedWaypoint",
    Callback = function(option)
        selectedWaypoint = typeof(option) == "table" and option[1] or option
        if selectedWaypoint == "No waypoints" then
            selectedWaypoint = nil
        end
    end
})

local function refreshWaypointDropdown()
    local names = waypointNames()
    pcall(function()
        WaypointDropdown:Refresh(names, true)
    end)
    if selectedWaypoint and waypoints[selectedWaypoint] then
        pcall(function()
            WaypointDropdown:Set({selectedWaypoint})
        end)
    end
end

local function saveWaypoint(name)
    if not HRP then
        Rayfield:Notify({
            Title = "WAYPOINTS",
            Content = "Character is not ready.",
            Duration = 3,
            Image = "circle-alert"
        })
        return
    end

    local pos = HRP.Position
    local look = HRP.CFrame.LookVector

    waypoints[name] = {
        x = pos.X,
        y = pos.Y,
        z = pos.Z,
        lx = look.X,
        ly = look.Y,
        lz = look.Z
    }

    selectedWaypoint = name
    saveWaypointsToDisk()
    refreshWaypointDropdown()

    Rayfield:Notify({
        Title = "WAYPOINT SAVED",
        Content = name,
        Duration = 3,
        Image = "bookmark"
    })
end

local function teleportToWaypoint(name)
    if not HRP or not name or not waypoints[name] then return end

    local data = waypoints[name]
    previousPosition = HRP.CFrame

    HRP.CFrame = CFrame.lookAt(
        Vector3.new(data.x, data.y, data.z),
        Vector3.new(data.x + data.lx, data.y + data.ly, data.z + data.lz)
    )

    Rayfield:Notify({
        Title = "WAYPOINT",
        Content = "Arrived at  •  " .. name,
        Duration = 2,
        Image = "navigation"
    })
end

WaypointsTab:CreateInput({
    Name = "Waypoint Name",
    PlaceholderText = "Example: Base",
    RemoveTextAfterFocusLost = false,
    Callback = function(value)
        if value and value ~= "" then
            saveWaypoint(value)
        end
    end
})

WaypointsTab:CreateButton({
    Name = "Save Current Position",
    Callback = function()
        local name = "Waypoint " .. tostring(#waypointNames() + 1)
        saveWaypoint(name)
    end
})

WaypointsTab:CreateButton({
    Name = "Teleport To Selected",
    Callback = function()
        teleportToWaypoint(selectedWaypoint)
    end
})

WaypointsTab:CreateButton({
    Name = "Return To Previous Position",
    Callback = function()
        if HRP and previousPosition then
            local current = HRP.CFrame
            HRP.CFrame = previousPosition
            previousPosition = current
        else
            Rayfield:Notify({
                Title = "WAYPOINTS",
                Content = "No previous position is available.",
                Duration = 3,
                Image = "circle-alert"
            })
        end
    end
})

WaypointsTab:CreateButton({
    Name = "Delete Selected Waypoint",
    Callback = function()
        if selectedWaypoint and waypoints[selectedWaypoint] then
            local deleted = selectedWaypoint
            waypoints[selectedWaypoint] = nil
            selectedWaypoint = nil
            saveWaypointsToDisk()
            refreshWaypointDropdown()

            Rayfield:Notify({
                Title = "WAYPOINT DELETED",
                Content = deleted,
                Duration = 2,
                Image = "trash-2"
            })
        end
    end
})

WaypointsTab:CreateButton({
    Name = "Clear All Waypoints",
    Callback = function()
        waypoints = {}
        selectedWaypoint = nil
        saveWaypointsToDisk()
        refreshWaypointDropdown()

        Rayfield:Notify({
            Title = "WAYPOINTS",
            Content = "All saved waypoints cleared.",
            Duration = 3,
            Image = "trash-2"
        })
    end
})

WaypointsTab:CreateButton({
    Name = "Quickpoint  •  Save Current",
    Callback = function()
        saveWaypoint("Quickpoint")
    end
})

loadWaypointsFromDisk()
refreshWaypointDropdown()

--//======================================================
--// PLAYER MANAGER
--//======================================================

PlayerManagerTab:CreateParagraph({
    Title = "✦ PLAYER MANAGER  /  OPERATOR CONSOLE",
    Content =
        "Select a player to inspect their live information, teleport to them or spectate their character."
})

local function playerNames()
    local names = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(names, player.Name)
        end
    end
    table.sort(names)
    if #names == 0 then names = {"No players"} end
    return names
end

local PlayerDropdown = PlayerManagerTab:CreateDropdown({
    Name = "Select Player",
    Options = playerNames(),
    CurrentOption = {playerNames()[1]},
    MultipleOptions = false,
    Flag = "SelectedPlayer",
    Callback = function(option)
        local name = typeof(option) == "table" and option[1] or option
        selectedPlayer = Players:FindFirstChild(name)
        if name == "No players" then selectedPlayer = nil end
    end
})

local function refreshPlayerDropdown()
    local names = playerNames()
    pcall(function()
        PlayerDropdown:Refresh(names, true)
    end)
end

PlayerManagerTab:CreateSection("PLAYER INFORMATION")

local PMNameLabel = PlayerManagerTab:CreateLabel("Player  •  None", "user")
local PMHealthLabel = PlayerManagerTab:CreateLabel("Health  •  --", "heart")
local PMDistanceLabel = PlayerManagerTab:CreateLabel("Distance  •  --", "ruler")
local PMTeamLabel = PlayerManagerTab:CreateLabel("Team  •  --", "shield")

PlayerManagerTab:CreateSection("ACTIONS")

local function teleportToSelected()
    if selectedPlayer then
        teleportToPlayer(selectedPlayer)
    end
end

PlayerManagerTab:CreateButton({
    Name = "Teleport To Selected",
    Callback = teleportToSelected
})

PlayerManagerTab:CreateButton({
    Name = "Spectate Selected",
    Callback = function()
        if selectedPlayer and selectedPlayer.Character then
            local hum = selectedPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum and workspace.CurrentCamera then
                workspace.CurrentCamera.CameraSubject = hum
                spectating = true
                Rayfield:Notify({
                    Title = "PLAYER MANAGER",
                    Content = "Spectating  •  " .. selectedPlayer.DisplayName,
                    Duration = 2,
                    Image = "eye"
                })
            end
        end
    end
})

PlayerManagerTab:CreateButton({
    Name = "Stop Spectating",
    Callback = function()
        if Humanoid and workspace.CurrentCamera then
            workspace.CurrentCamera.CameraSubject = Humanoid
        end
        spectating = false
    end
})

PlayerManagerTab:CreateButton({
    Name = "Highlight Selected",
    Callback = function()
        if not selectedPlayer or not selectedPlayer.Character then return end

        local existing = selectedPlayer.Character:FindFirstChild("SpaceHub_Selected")
        if existing then
            existing:Destroy()
            return
        end

        local highlight = Instance.new("Highlight")
        highlight.Name = "SpaceHub_Selected"
        highlight.FillTransparency = 0.65
        highlight.OutlineTransparency = 0
        highlight.FillColor = Color3.fromRGB(255, 210, 80)
        highlight.OutlineColor = Color3.fromRGB(255, 245, 180)
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Parent = selectedPlayer.Character
    end
})

task.spawn(function()
    while task.wait(0.25) do
        if selectedPlayer and selectedPlayer.Parent then
            local character = selectedPlayer.Character
            local hum = character and character:FindFirstChildOfClass("Humanoid")
            local root = character and character:FindFirstChild("HumanoidRootPart")

            PMNameLabel:Set(
                "Player  •  " .. selectedPlayer.DisplayName .. "  @" .. selectedPlayer.Name,
                "user"
            )

            PMHealthLabel:Set(
                "Health  •  " ..
                (hum and (math.floor(hum.Health + 0.5) .. " / " .. math.floor(hum.MaxHealth + 0.5)) or "--"),
                "heart"
            )

            local distance = HRP and root and (HRP.Position - root.Position).Magnitude
            PMDistanceLabel:Set(
                "Distance  •  " .. (distance and (math.floor(distance) .. " studs") or "--"),
                "ruler"
            )

            PMTeamLabel:Set(
                "Team  •  " .. (selectedPlayer.Team and selectedPlayer.Team.Name or "Neutral"),
                "shield"
            )
        else
            PMNameLabel:Set("Player  •  None", "user")
            PMHealthLabel:Set("Health  •  --", "heart")
            PMDistanceLabel:Set("Distance  •  --", "ruler")
            PMTeamLabel:Set("Team  •  --", "shield")
        end
    end
end)

Players.PlayerAdded:Connect(function()
    task.wait(0.5)
    refreshPlayerDropdown()
end)

Players.PlayerRemoving:Connect(function(player)
    if selectedPlayer == player then
        selectedPlayer = nil
    end
    refreshPlayerDropdown()
end)

--//======================================================
--// CONFIGURACIÓN
--//======================================================

ConfigurationTab:CreateParagraph({

    Title = "✦ SYSTEM CONFIGURATION  /  PREFERENCES",

    Content =
        "Tune movement, interface and targeting preferences."

})

ConfigurationTab:CreateSection(
    "PARAMETERS  /  MOVEMENT"
)

ConfigurationTab:CreateSlider({

    Name = "Velocidad de Caminar",

    Range = {
        1,
        250
    },

    Increment = 1,

    Suffix = " SPD",

    CurrentValue = 16,

    Flag = "ConfigWalkSpeed",

    Callback = function(value)

        walkSpeed =
            value

        if Humanoid then
            Humanoid.WalkSpeed =
                value
        end

    end

})

ConfigurationTab:CreateSlider({

    Name = "Velocidad de Vuelo",

    Range = {
        10,
        300
    },

    Increment = 5,

    Suffix = " SPD",

    CurrentValue = 50,

    Flag = "ConfigFlightSpeed",

    Callback = function(value)

        flightSpeed =
            value

    end

})

--//======================================================
--// TEMAS
--//======================================================

ConfigurationTab:CreateSection(
    "INTERFACE  /  CORE"
)

ConfigurationTab:CreateDropdown({

    Name = "Tema de Interfaz",

    Options = {

        "Default",
        "DarkBlue",
        "Ocean",
        "Amethyst",
        "Bloom",
        "Serenity",
        "Green",
        "AmberGlow",
        "Light",
        "Orbital"

    },

    CurrentOption = {
        "Orbital"
    },

    MultipleOptions = false,

    Flag = "Theme",

    Callback = function(option)

        local theme

        if typeof(option) == "table" then
            theme = option[1]
        else
            theme = option
        end

        if theme then

            pcall(function()

                if theme == "Orbital" then
                    Window:ModifyTheme(SpaceTheme)
                else
                    Window:ModifyTheme(theme)
                end

            end)

        end

    end

})

ConfigurationTab:CreateParagraph({

    Title = "SPACE HUB  •  6.0.0  /  ORBITAL EDITION",

    Content =
        "Interfaz Orbital Premium\n\n" ..
        "Núcleo de Movimiento\n" ..
        "Núcleo de Objetivos\n" ..
        "Sistemas Visuales\n" ..
        "Red de Teletransportes\n" ..
        "Transporte Masivo\n" ..
        "Núcleo de Configuración
" ..
        "Monitor de NPCs / Bosses / Frutas
" ..
        "Gestor Multi-Mares + Diagnóstico"

})


--//======================================================
--// SPACE HUB  /  ULTIMATE CONTROL LAYER
--//======================================================

-- Central state manager. Existing systems remain intact; these controls provide
-- a single place to enable/disable the Blox Fruits automation safely.
local Ultimate = {
    Enabled = false,
    AutoFarm = false,
    AutoAttack = true,
    AutoHaki = false,
    BringMobs = false,
    AntiAFK = false,
    SafeMode = true,
    MaxTargetDistance = 250,
    FarmDistance = 7,
    AttackDelay = 0.12,
    Target = nil
}

local function notifyUltimate(title, content, duration, image)
    pcall(function()
        Rayfield:Notify({
            Title = title or "SPACE HUB",
            Content = content or "",
            Duration = duration or 3,
            Image = image or "sparkles"
        })
    end)
end

local function getRoot(character)
    character = character or LocalPlayer.Character
    return character and (
        character:FindFirstChild("HumanoidRootPart")
        or character.PrimaryPart
    )
end

local function getHumanoid(character)
    character = character or LocalPlayer.Character
    return character and character:FindFirstChildOfClass("Humanoid")
end

local function alive(character)
    local hum = getHumanoid(character)
    return hum and hum.Health > 0
end

local function getEnemies()
    local folder = workspace:FindFirstChild("Enemies")
    if not folder then
        return {}
    end

    local result = {}

    for _, model in ipairs(folder:GetChildren()) do
        if model:IsA("Model") then
            local hum = model:FindFirstChildOfClass("Humanoid")
            local root = getRoot(model)

            if hum and root and hum.Health > 0 then
                table.insert(result, model)
            end
        end
    end

    return result
end

local function nearestEnemy(maxDistance)
    local root = getRoot()
    if not root then
        return nil
    end

    local best
    local bestDistance = maxDistance or math.huge

    for _, enemy in ipairs(getEnemies()) do
        local enemyRoot = getRoot(enemy)
        local enemyHum = getHumanoid(enemy)

        if enemyRoot and enemyHum and enemyHum.Health > 0 then
            local distance = (root.Position - enemyRoot.Position).Magnitude

            if distance < bestDistance then
                bestDistance = distance
                best = enemy
            end
        end
    end

    return best
end

local function getTool()
    local character = LocalPlayer.Character
    if not character then
        return nil
    end

    local equipped = character:FindFirstChildOfClass("Tool")
    if equipped then
        return equipped
    end

    local backpack = LocalPlayer:FindFirstChildOfClass("Backpack")
    if not backpack then
        return nil
    end

    local preferred

    for _, item in ipairs(backpack:GetChildren()) do
        if item:IsA("Tool") then
            local n = item.Name:lower()

            if n:find("combat")
                or n:find("fighting")
                or n:find("sword")
                or n:find("katana")
                or n:find("blade") then
                preferred = item
                break
            end

            preferred = preferred or item
        end
    end

    if preferred then
        local hum = getHumanoid(character)

        if hum then
            pcall(function()
                hum:EquipTool(preferred)
            end)
        end
    end

    return character:FindFirstChildOfClass("Tool")
end

local function attack()
    if not Ultimate.AutoAttack then
        return
    end

    local tool = getTool()
    if tool then
        pcall(function()
            tool:Activate()
        end)
    end
end

local function resetUltimateTarget()
    Ultimate.Target = nil
end

stopUltimate = function()
    Ultimate.Enabled = false
    Ultimate.AutoFarm = false
    Ultimate.Target = nil
    bfAutoHaki = false
    bfBringMobs = false
    bfTarget = nil

    pcall(function()
        stopBloxAutoFarm()
    end)
end

startUltimate = function()
    Ultimate.Enabled = true
    Ultimate.AutoFarm = true

    -- Use the existing, single Blox Fruits controller.
    startBloxAutoFarm()
end

-- Master Blox Fruits tab.
local UltimateTab = Window:CreateTab(
    "Blox Fruits • Definitivo",
    "zap"
)

UltimateTab:CreateParagraph({
    Title = "✦ SPACE HUB  /  ULTIMATE EDITION",
    Content =
        "Sistema centralizado de control para Blox Fruits.\n\n" ..
        "Mantiene el hub existente y añade un control centralizado para las opciones de " ..
        "farmeo, combate y seguridad."
})

UltimateTab:CreateSection("CONTROL PRINCIPAL")

UltimateTab:CreateToggle({
    Name = "Granja Automática Definitiva",
    CurrentValue = false,
    Flag = "UltimateAutoFarm",
    Callback = function(value)
        if value then
            startUltimate()
            notifyUltimate(
                "ULTIMATE",
                "Granja Automática activada.",
                3,
                "zap"
            )
        else
            stopUltimate()
            notifyUltimate(
                "ULTIMATE",
                "Toda la granja automática fue detenida.",
                3,
                "circle-stop"
            )
        end
    end
})

UltimateTab:CreateToggle({
    Name = "Ataque Automático",
    CurrentValue = true,
    Flag = "UltimateAutoAttack",
    Callback = function(value)
        Ultimate.AutoAttack = value
        bfAutoAttack = value
    end
})

UltimateTab:CreateToggle({
    Name = "Modo Seguro",
    CurrentValue = true,
    Flag = "UltimateSafeMode",
    Callback = function(value)
        Ultimate.SafeMode = value
    end
})

UltimateTab:CreateSection("OBJETIVOS")

UltimateTab:CreateSlider({
    Name = "Distancia Máxima del Objetivo",
    Range = {50, 2500},
    Increment = 50,
    Suffix = " studs",
    CurrentValue = 250,
    Flag = "UltimateTargetDistance",
    Callback = function(value)
        Ultimate.MaxTargetDistance = value
    end
})

UltimateTab:CreateSlider({
    Name = "Distancia de Farmeo",
    Range = {3, 20},
    Increment = 1,
    Suffix = " studs",
    CurrentValue = 7,
    Flag = "UltimateFarmDistance",
    Callback = function(value)
        Ultimate.FarmDistance = value
        bfFarmDistance = value
    end
})

UltimateTab:CreateSlider({
    Name = "Retraso de Ataque",
    Range = {0.05, 0.5},
    Increment = 0.05,
    Suffix = " sec",
    CurrentValue = 0.12,
    Flag = "UltimateAttackDelay",
    Callback = function(value)
        Ultimate.AttackDelay = value
        bfAttackInterval = value
    end
})

local UltimateTargetLabel =
    UltimateTab:CreateLabel(
        "Objetivo  •  Ninguno",
        "target"
    )

UltimateTab:CreateButton({
    Name = "Buscar NPC Más Cercano",
    Callback = function()
        Ultimate.Target =
            nearestEnemy(Ultimate.MaxTargetDistance)

        if Ultimate.Target then
            UltimateTargetLabel:Set(
                "Objetivo  •  " .. Ultimate.Target.Name,
                "target"
            )

            notifyUltimate(
                "OBJETIVO ENCONTRADO",
                Ultimate.Target.Name,
                3,
                "crosshair"
            )
        else
            UltimateTargetLabel:Set(
                "Objetivo  •  Ninguno",
                "target"
            )

            notifyUltimate(
                "OBJETIVOS",
                "No se encontró ningún NPC válido.",
                3,
                "search"
            )
        end
    end
})

UltimateTab:CreateButton({
    Name = "Reiniciar Objetivo",
    Callback = function()
        resetUltimateTarget()

        UltimateTargetLabel:Set(
            "Objetivo  •  Ninguno",
            "target"
        )
    end
})

UltimateTab:CreateSection("COMBATE")

UltimateTab:CreateToggle({
    Name = "Haki Automático",
    CurrentValue = false,
    Flag = "UltimateAutoHaki",
    Callback = function(value)
        Ultimate.AutoHaki = value
        bfAutoHaki = value
        if value then
            enableHaki()
        end
    end
})

UltimateTab:CreateToggle({
    Name = "Atraer Enemigos",
    CurrentValue = false,
    Flag = "UltimateBringMobs",
    Callback = function(value)
        Ultimate.BringMobs = value
        bfBringMobs = value
    end
})

UltimateTab:CreateSection("SEGURIDAD DEL JUGADOR")

UltimateTab:CreateToggle({
    Name = "Anti-Inactividad",
    CurrentValue = false,
    Flag = "UltimateAntiAFK",
    Callback = function(value)
        Ultimate.AntiAFK = value
    end
})

UltimateTab:CreateButton({
    Name = "PARADA DE EMERGENCIA",
    Callback = function()
        stopUltimate()
        Ultimate.Enabled = false
        Ultimate.AutoFarm = false

        -- Also stop the earlier Blox Fruits module if it exists.
        pcall(function()
            stopBloxAutoFarm()
        end)

        notifyUltimate(
            "PARADA DE EMERGENCIA",
            "Toda la automatización de Blox Fruits fue detenida.",
            4,
            "circle-stop"
        )
    end
})

UltimateTab:CreateParagraph({
    Title = "ESTADO DEL SISTEMA",
    Content =
        "Ultimate layer loaded.\n" ..
        "The original Space Hub systems are preserved.\n" ..
        "Usa la Parada de Emergencia para detener la automatización."
})

-- Ultimate status loop. Movement/attack is handled only by the Blox controller above.
task.spawn(function()
    while task.wait(0.15) do
        if Ultimate.Enabled and Ultimate.AutoFarm and bfAutoFarm and alive() then
            local target = bfTarget

            if target and target.Parent then
                local hum = target:FindFirstChildOfClass("Humanoid")
                local targetRoot = target:FindFirstChild("HumanoidRootPart") or target.PrimaryPart

                if hum and targetRoot and hum.Health > 0 then
                    local root = getRoot()
                    local distance = root and (root.Position - targetRoot.Position).Magnitude or math.huge

                    UltimateTargetLabel:Set(
                        "Objetivo  •  " .. target.Name ..
                        "  •  HP " .. math.floor(hum.Health + 0.5) ..
                        "  •  " .. math.floor(distance) .. " studs",
                        "target"
                    )
                end
            else
                UltimateTargetLabel:Set("Objetivo  •  Buscando...", "search")
            end
        elseif not Ultimate.AutoFarm then
            UltimateTargetLabel:Set("Objetivo  •  En espera", "target")
        end
    end
end)

-- Optional Anti-AFK handler for the Ultimate layer.
local UltimateVirtualUser = game:GetService("VirtualUser")

LocalPlayer.Idled:Connect(function()
    if Ultimate.AntiAFK then
        pcall(function()
            UltimateVirtualUser:CaptureController()
            UltimateVirtualUser:ClickButton2(Vector2.new(0, 0))
        end)
    end
end)


--//======================================================
--// MARINA  /  CAZADOR DE RECOMPENSAS ASISTIDO
--//======================================================

local MarinaTab = Window:CreateTab(
    "Marina • Recompensas",
    "anchor"
)

local Marina = {
    Enabled = false,
    ESP = false,
    Selected = nil,
    MaxDistance = 2500,
    SafeZoneOnly = true,
    ShowHealth = true,
    ShowDistance = true,
    MarkerFolder = nil
}

MarinaTab:CreateParagraph({
    Title = "⚓ MARINA / CAZADOR DE RECOMPENSAS",
    Content =
        "Panel PvP asistido.\n\n" ..
        "Detecta jugadores, muestra su posición, vida y distancia, " ..
        "y permite seleccionar un objetivo manualmente. " ..
        "El ataque permanece bajo tu control."
})

MarinaTab:CreateSection("DETECCIÓN DE JUGADORES")

local function marinaCharacter(player)
    return player.Character
end

local function marinaRoot(player)
    local character = marinaCharacter(player)
    return character and (
        character:FindFirstChild("HumanoidRootPart")
        or character.PrimaryPart
    )
end

local function marinaHumanoid(player)
    local character = marinaCharacter(player)
    return character and character:FindFirstChildOfClass("Humanoid")
end

local function marinaAlive(player)
    local hum = marinaHumanoid(player)
    return hum and hum.Health > 0
end

local function marinaDistance(player)
    local myRoot = getRoot()
    local theirRoot = marinaRoot(player)

    if not myRoot or not theirRoot then
        return math.huge
    end

    return (myRoot.Position - theirRoot.Position).Magnitude
end

local function marinaIsSafe(player)
    local character = marinaCharacter(player)
    if not character then
        return true
    end

    -- Common Roblox/experience convention: a character attribute can mark
    -- safe-zone state. If the experience exposes one, respect it.
    local safe =
        character:GetAttribute("InSafeZone")
        or character:GetAttribute("SafeZone")
        or character:GetAttribute("IsSafe")

    if safe == true then
        return true
    end

    local playerSafe =
        player:GetAttribute("InSafeZone")
        or player:GetAttribute("SafeZone")
        or player:GetAttribute("IsSafe")

    return playerSafe == true
end

local function marinaEligible(player)
    if player == LocalPlayer then
        return false
    end

    if not marinaAlive(player) then
        return false
    end

    local distance = marinaDistance(player)

    if distance > Marina.MaxDistance then
        return false
    end

    if Marina.SafeZoneOnly and marinaIsSafe(player) then
        return false
    end

    return true
end

local function marinaFindNearest()
    local nearest
    local nearestDistance = Marina.MaxDistance

    for _, player in ipairs(Players:GetPlayers()) do
        if marinaEligible(player) then
            local distance = marinaDistance(player)

            if distance < nearestDistance then
                nearestDistance = distance
                nearest = player
            end
        end
    end

    return nearest
end

local MarinaTargetLabel =
    MarinaTab:CreateLabel(
        "Objetivo • Ninguno",
        "target"
    )

local MarinaInfoLabel =
    MarinaTab:CreateLabel(
        "Estado • En espera",
        "info"
    )

local function marinaUpdateLabels()
    local target = Marina.Selected

    if not target or not target.Parent or not marinaAlive(target) then
        MarinaTargetLabel:Set(
            "Objetivo • Ninguno",
            "target"
        )

        MarinaInfoLabel:Set(
            "Estado • Sin objetivo",
            "info"
        )

        return
    end

    local hum = marinaHumanoid(target)
    local distance = marinaDistance(target)
    local safe = marinaIsSafe(target)

    local hpText = ""
    local distanceText = ""

    if Marina.ShowHealth and hum then
        hpText =
            " • Vida " ..
            math.floor(hum.Health + 0.5) ..
            "/" ..
            math.floor(hum.MaxHealth + 0.5)
    end

    if Marina.ShowDistance and distance < math.huge then
        distanceText =
            " • " ..
            math.floor(distance) ..
            " studs"
    end

    MarinaTargetLabel:Set(
        "Objetivo • " .. target.Name,
        "target"
    )

    MarinaInfoLabel:Set(
        "Estado • " ..
        (safe and "Zona segura" or "Fuera de zona segura") ..
        hpText ..
        distanceText,
        safe and "shield" or "crosshair"
    )
end

MarinaTab:CreateToggle({
    Name = "Activar Detector",
    CurrentValue = false,
    Flag = "MarinaDetector",
    Callback = function(value)
        Marina.Enabled = value
    end
})

MarinaTab:CreateToggle({
    Name = "Mostrar ESP",
    CurrentValue = false,
    Flag = "MarinaESP",
    Callback = function(value)
        Marina.ESP = value
    end
})

MarinaTab:CreateToggle({
    Name = "Ignorar Zonas Seguras",
    CurrentValue = false,
    Flag = "MarinaSafeZoneFilter",
    Callback = function(value)
        Marina.SafeZoneOnly = not value
        marinaUpdateLabels()
    end
})

MarinaTab:CreateToggle({
    Name = "Mostrar Vida",
    CurrentValue = true,
    Flag = "MarinaHealth",
    Callback = function(value)
        Marina.ShowHealth = value
        marinaUpdateLabels()
    end
})

MarinaTab:CreateToggle({
    Name = "Mostrar Distancia",
    CurrentValue = true,
    Flag = "MarinaDistance",
    Callback = function(value)
        Marina.ShowDistance = value
        marinaUpdateLabels()
    end
})

MarinaTab:CreateSlider({
    Name = "Distancia Máxima",
    Range = {100, 5000},
    Increment = 100,
    Suffix = " studs",
    CurrentValue = 2500,
    Flag = "MarinaMaxDistance",
    Callback = function(value)
        Marina.MaxDistance = value
    end
})

MarinaTab:CreateButton({
    Name = "Buscar Objetivo Cercano",
    Callback = function()
        Marina.Selected = marinaFindNearest()
        marinaUpdateLabels()

        if Marina.Selected then
            Rayfield:Notify({
                Title = "MARINA",
                Content =
                    "Objetivo encontrado: " ..
                    Marina.Selected.Name,
                Duration = 3,
                Image = "target"
            })
        else
            Rayfield:Notify({
                Title = "MARINA",
                Content =
                    "No se encontró un jugador válido fuera de zona segura.",
                Duration = 3,
                Image = "search"
            })
        end
    end
})

MarinaTab:CreateButton({
    Name = "Reiniciar Objetivo",
    Callback = function()
        Marina.Selected = nil
        marinaUpdateLabels()
    end
})

MarinaTab:CreateParagraph({
    Title = "⚠️ COMBATE ASISTIDO",
    Content =
        "El panel detecta y sigue información del jugador seleccionado. " ..
        "No ejecuta ataques automáticos contra otros jugadores ni decide por sí solo a quién atacar."
})

-- Lightweight ESP using BillboardGui, with cleanup.
local function marinaClearESP()
    if Marina.MarkerFolder then
        pcall(function()
            Marina.MarkerFolder:Destroy()
        end)
    end

    Marina.MarkerFolder = nil
end

local function marinaCreateMarker(player)
    if not Marina.ESP or not marinaEligible(player) then
        return
    end

    local character = marinaCharacter(player)
    local root = marinaRoot(player)

    if not character or not root then
        return
    end

    local gui = Instance.new("BillboardGui")
    gui.Name = "MarinaESP"
    gui.Size = UDim2.fromOffset(220, 60)
    gui.StudsOffset = Vector3.new(0, 4, 0)
    gui.AlwaysOnTop = true
    gui.Adornee = root
    gui.Parent = root

    local label = Instance.new("TextLabel")
    label.BackgroundTransparency = 1
    label.Size = UDim2.fromScale(1, 1)
    label.TextScaled = true
    label.Font = Enum.Font.GothamBold
    label.Text = player.Name
    label.Parent = gui
end

task.spawn(function()
    while task.wait(0.5) do
        if Marina.Enabled then
            if Marina.ESP then
                marinaClearESP()

                local folder = Instance.new("Folder")
                folder.Name = "MarinaESPContainer"
                folder.Parent = LocalPlayer:FindFirstChildOfClass("PlayerGui")
                    or workspace

                Marina.MarkerFolder = folder

                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer then
                        local character = marinaCharacter(player)
                        local root = marinaRoot(player)

                        if character and root and marinaEligible(player) then
                            local gui = Instance.new("BillboardGui")
                            gui.Name = "MarinaESP"
                            gui.Size = UDim2.fromOffset(220, 55)
                            gui.StudsOffset = Vector3.new(0, 4, 0)
                            gui.AlwaysOnTop = true
                            gui.Adornee = root
                            gui.Parent = folder

                            local label = Instance.new("TextLabel")
                            label.BackgroundTransparency = 1
                            label.Size = UDim2.fromScale(1, 1)
                            label.TextScaled = true
                            label.Font = Enum.Font.GothamBold

                            local hum = marinaHumanoid(player)
                            local dist = marinaDistance(player)

                            label.Text =
                                player.Name ..
                                (hum and
                                    ("\nHP: " ..
                                    math.floor(hum.Health + 0.5))
                                    or "") ..
                                "\n" ..
                                math.floor(dist) ..
                                " studs"

                            label.Parent = gui
                        end
                    end
                end
            else
                marinaClearESP()
            end

            if not Marina.Selected
                or not Marina.Selected.Parent
                or not marinaAlive(Marina.Selected) then

                Marina.Selected = marinaFindNearest()
            end

            marinaUpdateLabels()
        else
            marinaClearESP()
        end
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if Marina.Selected == player then
        Marina.Selected = nil
        marinaUpdateLabels()
    end
end)


--//======================================================
--// COFRES  /  TELETRANSPORTE
--//======================================================

local ChestsTab = Window:CreateTab(
    "Cofres • Teletransporte",
    "box"
)

local ChestSystem = {
    Enabled = false,
    AutoCollect = false,
    Selected = nil,
    MaxDistance = 5000,
    Delay = 0.8,
    LastAutoChest = nil
}

ChestsTab:CreateParagraph({
    Title = "📦 COFRES / TELETRANSPORTE",
    Content =
        "Busca cofres visibles en el mapa y permite " ..
        "teletransportarte al cofre seleccionado."
})

ChestsTab:CreateSection("BUSCADOR DE COFRES")

local ChestLabel = ChestsTab:CreateLabel(
    "Cofre • Ninguno",
    "box"
)

local ChestInfo = ChestsTab:CreateLabel(
    "Estado • En espera",
    "info"
)

local function getChestRoot(obj)
    if not obj then
        return nil
    end

    if obj:IsA("BasePart") then
        return obj
    end

    if obj:IsA("Model") then
        return obj.PrimaryPart
            or obj:FindFirstChildWhichIsA("BasePart", true)
    end

    return nil
end

local function isChest(obj)
    if not (obj:IsA("Model") or obj:IsA("BasePart")) then
        return false
    end

    local name = obj.Name:lower()

    return name:find("chest")
        or name:find("cofre")
        or name:find("treasure")
end

local function findChests()
    local results = {}

    for _, obj in ipairs(workspace:GetDescendants()) do
        if isChest(obj) then
            local root = getChestRoot(obj)

            if root then
                table.insert(results, {
                    Object = obj,
                    Root = root
                })
            end
        end
    end

    return results
end

local function findNearestChest()
    local myRoot = getRoot()

    if not myRoot then
        return nil
    end

    local nearest
    local nearestDistance = ChestSystem.MaxDistance

    for _, data in ipairs(findChests()) do
        if data.Root and data.Root.Parent then
            local distance =
                (myRoot.Position - data.Root.Position).Magnitude

            if distance < nearestDistance then
                nearestDistance = distance

                nearest = data
                nearest.Distance = distance
            end
        end
    end

    return nearest
end

local function teleportToChest(data)
    if not data or not data.Root then
        return false
    end

    local root = getRoot()

    if not root then
        return false
    end

    pcall(function()
        root.CFrame =
            CFrame.new(
                data.Root.Position + Vector3.new(0, 3, 0)
            )
    end)

    return true
end

local function updateChestInfo()
    if not ChestSystem.Selected then
        ChestLabel:Set(
            "Cofre • Ninguno",
            "box"
        )

        ChestInfo:Set(
            "Estado • En espera",
            "info"
        )

        return
    end

    local data = ChestSystem.Selected

    if not data.Object
        or not data.Object.Parent
        or not data.Root
        or not data.Root.Parent then

        ChestSystem.Selected = nil
        updateChestInfo()
        return
    end

    local root = getRoot()

    if root then
        local distance =
            (root.Position - data.Root.Position).Magnitude

        ChestLabel:Set(
            "Cofre • " .. data.Object.Name,
            "box"
        )

        ChestInfo:Set(
            "Distancia • " ..
            math.floor(distance) ..
            " studs",
            "map-pin"
        )
    end
end

ChestsTab:CreateToggle({
    Name = "Activar Sistema de Cofres",
    CurrentValue = false,
    Flag = "ChestSystemEnabled",
    Callback = function(value)
        ChestSystem.Enabled = value
    end
})

ChestsTab:CreateSlider({
    Name = "Distancia Máxima",
    Range = {100, 10000},
    Increment = 100,
    Suffix = " studs",
    CurrentValue = 5000,
    Flag = "ChestMaxDistance",
    Callback = function(value)
        ChestSystem.MaxDistance = value
    end
})

ChestsTab:CreateButton({
    Name = "Buscar Cofre Más Cercano",
    Callback = function()
        ChestSystem.Selected = findNearestChest()
        updateChestInfo()

        if ChestSystem.Selected then
            Rayfield:Notify({
                Title = "COFRES",
                Content =
                    "Encontrado: " ..
                    ChestSystem.Selected.Object.Name,
                Duration = 3,
                Image = "box"
            })
        else
            Rayfield:Notify({
                Title = "COFRES",
                Content = "No se encontró ningún cofre.",
                Duration = 3,
                Image = "search"
            })
        end
    end
})

ChestsTab:CreateButton({
    Name = "TP al Cofre Seleccionado",
    Callback = function()
        if ChestSystem.Selected then
            ChestSystem.LastAutoChest = nil
            if teleportToChest(ChestSystem.Selected) then
                Rayfield:Notify({
                    Title = "TELETRANSPORTE",
                    Content =
                        "Teletransportado al cofre.",
                    Duration = 2,
                    Image = "map-pin"
                })
            end
        else
            Rayfield:Notify({
                Title = "COFRES",
                Content =
                    "Primero selecciona un cofre.",
                Duration = 3,
                Image = "circle-alert"
            })
        end
    end
})

ChestsTab:CreateButton({
    Name = "TP al Cofre Más Cercano",
    Callback = function()
        local chest = findNearestChest()

        if chest then
            ChestSystem.Selected = chest
            ChestSystem.LastAutoChest = nil
            teleportToChest(chest)
            updateChestInfo()

            Rayfield:Notify({
                Title = "TELETRANSPORTE",
                Content =
                    "TP al cofre más cercano.",
                Duration = 2,
                Image = "map-pin"
            })
        else
            Rayfield:Notify({
                Title = "COFRES",
                Content = "No se encontró ningún cofre.",
                Duration = 3,
                Image = "circle-alert"
            })
        end
    end
})

ChestsTab:CreateToggle({
    Name = "Auto TP a Cofres",
    CurrentValue = false,
    Flag = "ChestAutoTeleport",
    Callback = function(value)
        ChestSystem.AutoCollect = value
    end
})

ChestsTab:CreateSlider({
    Name = "Espera entre Cofres",
    Range = {0.5, 5},
    Increment = 0.5,
    Suffix = " sec",
    CurrentValue = 0.8,
    Flag = "ChestTeleportDelay",
    Callback = function(value)
        ChestSystem.Delay = value
    end
})

ChestsTab:CreateButton({
    Name = "Limpiar Cofre Seleccionado",
    Callback = function()
        ChestSystem.Selected = nil
        updateChestInfo()
    end
})

task.spawn(function()
    while task.wait(0.5) do
        if ChestSystem.Enabled then
            if ChestSystem.AutoCollect then
                local chest = findNearestChest()

                if chest then
                    ChestSystem.Selected = chest

                    -- Avoid repeatedly teleporting to the same still-present chest.
                    if ChestSystem.LastAutoChest ~= chest.Object then
                        teleportToChest(chest)
                        ChestSystem.LastAutoChest = chest.Object
                    end

                    updateChestInfo()
                    task.wait(ChestSystem.Delay)
                else
                    ChestSystem.LastAutoChest = nil
                end
            else
                updateChestInfo()
            end
        end
    end
end)

--//======================================================
--// CHARACTER RESPAWN
--//======================================================

LocalPlayer.CharacterAdded:Connect(
    function(character)

        task.wait(0.5)

        updateCharacter(
            character
        )

        if Humanoid then
            Humanoid.WalkSpeed = walkSpeed
            Humanoid.UseJumpPower = true
            Humanoid.JumpPower = jumpPower
            Humanoid.HipHeight = hipHeight
        end

        if customGravityEnabled then
            workspace.Gravity = customGravity
        end

        if espEnabled then

            task.wait(0.2)

            refreshESP()

        end

        if flying then

            task.wait(0.2)

            startFlying()

        end

    end
)

--//======================================================
--// LOAD CONFIGURATION
--//======================================================

pcall(function()

    Rayfield:LoadConfiguration()

end)

--//======================================================
--// INICIO
--//======================================================

Rayfield:Notify({

    Title = "SPACE HUB",

    Content =
        "Orbital command deck online  •  v6.0.0  •  Smart Core activo.",

    Duration = 5,

    Image = "sparkles"

})
