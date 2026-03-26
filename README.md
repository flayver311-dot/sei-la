local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
 
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

local HttpService = game:GetService("HttpService")
local CONFIG_FILE = "WaveDuels_Config.json"

local sliderValues = {
    autoSpeed = 45,
    stealSpeed = 27,
    infJump = 60,
    gravity = 0,
    spin = 0
}

local toggleStates = {}

local function tryReadFile(name)
    if type(readfile) ~= "function" then return nil end
    local ok, content = pcall(function() return readfile(name) end)
    if not ok then return nil end
    return content
end

local function tryWriteFile(name, content)
    if type(writefile) ~= "function" then return false end
    local ok, err = pcall(function() writefile(name, content) end)
    return ok, err
end

local function SaveConfigs()
    local data = {
        saveEnabled = (toggleStates["SAVE CONFIG"] == true),
        sliders = sliderValues,
        toggles = toggleStates
    }
    if toggleStates["SAVE CONFIG"] == true then
        pcall(function() tryWriteFile(CONFIG_FILE, HttpService:JSONEncode(data)) end)
    end
end

local function LoadConfigs()
    local content = tryReadFile(CONFIG_FILE)
    if not content then return end
    local ok, data = pcall(function() return HttpService:JSONDecode(content) end)
    if not ok or type(data) ~= "table" then return end
    if data.saveEnabled == true then
        if type(data.sliders) == "table" then
            for k,v in pairs(data.sliders) do sliderValues[k] = v end
        end
        if type(data.toggles) == "table" then
            for k,v in pairs(data.toggles) do toggleStates[k] = v end
        end
    end
end

LoadConfigs()

if PlayerGui:FindFirstChild("StoryGUI") then
    PlayerGui:FindFirstChild("StoryGUI"):Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "StoryGUI"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true

-- ==================== IMPROVED DRAG SYSTEM ====================
local draggedFrames = {}

local function createImprovedDrag(frame, onDragStart, onDragEnd)
    local dragging = false
    local dragInput = nil
    local dragStart = nil
    local startPos = nil

    frame.InputBegan:Connect(function(input, gp)
        if gp or dragging then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragInput = input
            dragStart = input.Position
            startPos = frame.Position
            draggedFrames[input] = frame
            if onDragStart then onDragStart() end
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if not dragging or dragInput ~= input or not dragStart then return end
        if input.Position then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if dragInput == input then
            dragging = false
            dragInput = nil
            draggedFrames[input] = nil
            if onDragEnd then onDragEnd() end
        end
    end)
end

-- FRAME PRINCIPAL
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 320, 0, 380)
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -190)
MainFrame.BackgroundColor3 = Color3.fromRGB(8, 20, 50)
MainFrame.BackgroundTransparency = 0.1
MainFrame.Parent = ScreenGui
MainFrame.Active = true

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 20)

local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Color = Color3.fromRGB(0,140,255)
Stroke.Transparency = 0.35
Stroke.Thickness = 2

local UIScale = Instance.new("UIScale", MainFrame)
UIScale.Scale = 0

local function openGui()
    MainFrame.Visible = true
    TweenService:Create(UIScale, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
    TweenService:Create(MainFrame, TweenInfo.new(0.25), {BackgroundTransparency = 0.08}):Play()
end

local function closeGui()
    local tweenMain = TweenService:Create(UIScale, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Scale = 0})
    local tweenBg = TweenService:Create(MainFrame, TweenInfo.new(0.25), {BackgroundTransparency = 0.1})
    tweenMain:Play()
    tweenBg:Play()

    tweenMain.Completed:Connect(function()
        MainFrame.Visible = false
    end)

    -- Close settings if visible (don't rely only on settingsOpened flag)
    if SettingsFrame and SettingsFrame.Visible then
        -- Use the existing closeSettingsGui to animate hide
        closeSettingsGui()
        if settingsOpened then
            TweenService:Create(GearButton, TweenInfo.new(0.5, Enum.EasingStyle.Linear), {Rotation = GearButton.Rotation + 360}):Play()
            settingsOpened = false
        end
    end
end

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,45)
Title.BackgroundTransparency = 1
Title.Text = "WAVE DUELS 1.0"
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
Title.TextColor3 = Color3.new(1,1,1)
Title.Parent = MainFrame

local Underline = Instance.new("Frame")
Underline.Size = UDim2.new(0.5, 0, 0, 2)
Underline.Position = UDim2.new(0.25, 0, 1, 0)
Underline.BackgroundColor3 = Color3.fromRGB(0, 140, 255)
Underline.Parent = Title

local UnderlineStroke = Instance.new("UIStroke", Underline)
UnderlineStroke.Color = Color3.fromRGB(0, 140, 255)
UnderlineStroke.Transparency = 0.5
UnderlineStroke.Thickness = 1

local GearButton = Instance.new("TextButton")
GearButton.Size = UDim2.new(0, 30, 0, 30)
GearButton.Position = UDim2.new(1, -40, 0, 7)
GearButton.BackgroundTransparency = 1
GearButton.Text = "⚙️"
GearButton.Font = Enum.Font.SourceSans
GearButton.TextSize = 24
GearButton.TextColor3 = Color3.new(1,1,1)
GearButton.Parent = MainFrame

-- POTATO SYSTEM
local PotatoEnabled = false
local partData = {}
local emitterData = {}

local Lighting = game:GetService("Lighting")
local Terrain = workspace:FindFirstChildOfClass("Terrain")

local originalLighting = {
    GlobalShadows = Lighting.GlobalShadows,
    FogEnd = Lighting.FogEnd,
    FogStart = Lighting.FogStart
}

local function EnablePotato()
    pcall(function()
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    end)
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    Lighting.FogStart = 9e9
    if Terrain then
        pcall(function()
            Terrain.WaterWaveSize = 0
            Terrain.WaterWaveSpeed = 0
            Terrain.WaterReflectance = 0
            Terrain.WaterTransparency = 1
        end)
    end
    for _,obj in ipairs(workspace:GetDescendants()) do
        pcall(function()
            if obj:IsA("BasePart") then
                partData[obj] = {
                    Material = obj.Material,
                    Reflectance = obj.Reflectance,
                    CastShadow = obj.CastShadow
                }
                obj.CastShadow = false
                obj.Material = Enum.Material.Plastic
                obj.Reflectance = 0
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then
                emitterData[obj] = obj.Enabled
                obj.Enabled = false
            end
        end)
    end
end

local function DisablePotato()
    pcall(function()
        settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
    end)
    Lighting.GlobalShadows = originalLighting.GlobalShadows
    Lighting.FogEnd = originalLighting.FogEnd
    Lighting.FogStart = originalLighting.FogStart
    for part,data in pairs(partData) do
        if part and part.Parent then
            part.Material = data.Material
            part.Reflectance = data.Reflectance
            part.CastShadow = data.CastShadow
        end
    end
    for emitter,state in pairs(emitterData) do
        if emitter and emitter.Parent then
            emitter.Enabled = state
        end
    end
end

-- Ghost implementation for Potato Mode
local GHOST_TRANSPARENCY = 0.7
local GHOST_COLOR = Color3.fromRGB(0, 255, 255)
local ghostOriginals = setmetatable({}, { __mode = "k" }) -- weak keys
local ghostPlotsListener = nil

local function storeOriginal(inst, key, value)
    local data = ghostOriginals[inst]
    if not data then
        data = {}
        ghostOriginals[inst] = data
    end
    if data[key] == nil then
        data[key] = value
    end
end

local function applyGhostToObject(obj)
    pcall(function()
        if obj:IsA("BasePart") then
            storeOriginal(obj, "Transparency", obj.Transparency)
            storeOriginal(obj, "Color", obj.Color)
            if obj.LocalTransparencyModifier ~= nil then
                storeOriginal(obj, "LocalTransparencyModifier", obj.LocalTransparencyModifier)
            end
            storeOriginal(obj, "CanCollide", obj.CanCollide)
            obj.Transparency = GHOST_TRANSPARENCY
            obj.Color = GHOST_COLOR
            -- LocalTransparencyModifier can be set if available
            pcall(function() obj.LocalTransparencyModifier = GHOST_TRANSPARENCY end)
            obj.CanCollide = true
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            storeOriginal(obj, "Transparency", obj.Transparency)
            obj.Transparency = GHOST_TRANSPARENCY
        elseif obj:IsA("SurfaceGui") or obj:IsA("BillboardGui") then
            storeOriginal(obj, "Enabled", obj.Enabled)
            obj.Enabled = false
        end
    end)
end

local function processDecorations(decorations)
    for _, obj in ipairs(decorations:GetDescendants()) do
        applyGhostToObject(obj)
    end
end

local function applyGhostToPlot(plot)
    if not plot or not plot:IsA("Model") then return end
    local decorations = plot:FindFirstChild("Decorations")
    if decorations then
        processDecorations(decorations)
    end
end

local function applyGhostToAllPlots()
    local plotsFolder = Workspace:FindFirstChild("Plots")
    if not plotsFolder then return end
    for _, plot in ipairs(plotsFolder:GetChildren()) do
        applyGhostToPlot(plot)
    end
end

local function ensurePlotsListener()
    if ghostPlotsListener and ghostPlotsListener.Connected then return end
    local plotsFolder = Workspace:FindFirstChild("Plots")
    if not plotsFolder then return end
    ghostPlotsListener = plotsFolder.ChildAdded:Connect(function(newPlot)
        -- small delay to let contents initialize
        task.wait(0.05)
        applyGhostToPlot(newPlot)
    end)
end

local function disconnectPlotsListener()
    if ghostPlotsListener then
        pcall(function() ghostPlotsListener:Disconnect() end)
        ghostPlotsListener = nil
    end
end

local function restoreGhosts()
    for inst, data in pairs(ghostOriginals) do
        if inst and typeof(inst) == "Instance" and inst.Parent then
            pcall(function()
                if data.Transparency ~= nil and inst:IsA("BasePart") then inst.Transparency = data.Transparency end
                if data.Color ~= nil and inst:IsA("BasePart") then inst.Color = data.Color end
                if data.LocalTransparencyModifier ~= nil and inst:IsA("BasePart") and inst.LocalTransparencyModifier ~= nil then inst.LocalTransparencyModifier = data.LocalTransparencyModifier end
                if data.CanCollide ~= nil and inst:IsA("BasePart") then inst.CanCollide = data.CanCollide end
                if data.Transparency ~= nil and (inst:IsA("Decal") or inst:IsA("Texture")) then inst.Transparency = data.Transparency end
                if data.Enabled ~= nil and (inst:IsA("SurfaceGui") or inst:IsA("BillboardGui")) then inst.Enabled = data.Enabled end
            end)
        end
    end
    ghostOriginals = setmetatable({}, { __mode = "k" })
end

-- SETTINGS FRAME
local SettingsFrame = Instance.new("Frame")
SettingsFrame.Size = UDim2.new(0, 300, 0, 420)
SettingsFrame.Position = UDim2.new(0.5, 180, 0.5, -210)
SettingsFrame.BackgroundColor3 = Color3.fromRGB(8,20,50)
SettingsFrame.BackgroundTransparency = 0.08
SettingsFrame.Parent = ScreenGui
SettingsFrame.Active = true
SettingsFrame.ClipsDescendants = false
Instance.new("UICorner", SettingsFrame).CornerRadius = UDim.new(0,20)

local settingsStroke = Instance.new("UIStroke", SettingsFrame)
settingsStroke.Color = Color3.fromRGB(0,140,255)
settingsStroke.Transparency = 0.35
settingsStroke.Thickness = 2

local SettingsUIScale = Instance.new("UIScale", SettingsFrame)
SettingsUIScale.Scale = 0.8

SettingsFrame.Visible = false
task.wait()
SettingsFrame.Visible = true

TweenService:Create(SettingsUIScale, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()

-- TITLE BAR
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1,0,0,50)
TitleBar.BackgroundTransparency = 1
TitleBar.Parent = SettingsFrame

local settingsTitle = Instance.new("TextLabel")
settingsTitle.Size = UDim2.new(1,0,0,40)
settingsTitle.Position = UDim2.new(0,0,0,5)
settingsTitle.BackgroundTransparency = 1
settingsTitle.Text = "DUELS CONFIG"
settingsTitle.Font = Enum.Font.GothamBold
settingsTitle.TextSize = 20
settingsTitle.TextColor3 = Color3.new(1,1,1)
settingsTitle.Parent = TitleBar

-- DRAG PARA SETTINGS
local dragging, dragInput, dragStart, startPos

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragInput = input
        dragStart = input.Position
        startPos = SettingsFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        local delta = input.Position - dragStart
        SettingsFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input == dragInput then
        dragging = false
    end
end)

-- SCROLL
local Scroll = Instance.new("ScrollingFrame")
Scroll.Size = UDim2.new(1,-10,1,-60)
Scroll.Position = UDim2.new(0,5,0,55)
Scroll.CanvasSize = UDim2.new(0,0,0,0)
Scroll.ScrollBarThickness = 5
Scroll.BackgroundTransparency = 1
Scroll.Parent = SettingsFrame

local layout = Instance.new("UIListLayout",Scroll)
layout.Padding = UDim.new(0,22)

layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Scroll.CanvasSize = UDim2.new(0,0,0,layout.AbsoluteContentSize.Y + 10)
end)

-- SLIDER FUNCTION
local currentlyDraggingSlider = nil

local function CreateSlider(text,min,max,default)
    local holder = Instance.new("Frame")
    holder.Size = UDim2.new(1,-15,0,70)
    holder.BackgroundTransparency = 1
    holder.Parent = Scroll

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1,0,0,20)
    label.BackgroundTransparency = 1
    label.Text = text
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14
    label.TextColor3 = Color3.new(1,1,1)
    label.TextXAlignment = Enum.TextXAlignment.Center
    label.Parent = holder

    local valueLabel = Instance.new("TextLabel")
    valueLabel.Size = UDim2.new(1,0,0,18)
    valueLabel.Position = UDim2.new(0,0,0,20)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.TextSize = 13
    valueLabel.TextColor3 = Color3.fromRGB(150,220,255)
    valueLabel.TextXAlignment = Enum.TextXAlignment.Center
    valueLabel.Parent = holder

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(1,0,0,12)
    bar.Position = UDim2.new(0,0,0,45)
    bar.BackgroundColor3 = Color3.fromRGB(30,50,80)
    bar.Parent = holder
    Instance.new("UICorner",bar).CornerRadius = UDim.new(1,0)

    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(0,0,1,0)
    fill.BackgroundColor3 = Color3.fromRGB(0,122,204)
    fill.Parent = bar
    Instance.new("UICorner",fill).CornerRadius = UDim.new(1,0)

    local thumb = Instance.new("Frame")
    thumb.Size = UDim2.new(0,18,0,18)
    thumb.AnchorPoint = Vector2.new(0.5,0.5)
    thumb.Position = UDim2.new(0,0,0.5,0)
    thumb.BackgroundColor3 = Color3.fromRGB(0,122,204)
    thumb.Parent = bar
    thumb.ZIndex = 5
    thumb.Active = true
    Instance.new("UICorner",thumb).CornerRadius = UDim.new(1,0)

    -- Local state for THIS slider only
    local dragging = false
    local dragInput = nil

    local function setPercent(p)
        p = math.clamp(p,0,1)
        fill.Size = UDim2.new(p,0,1,0)
        thumb.Position = UDim2.new(p,0,0.5,0)
        local value = min + (max-min)*p
        valueLabel.Text = string.format("%.1f", value)
        
        if text == "AUTO SPEED" then
            sliderValues.autoSpeed = value
        elseif text == "STEAL SPEED" then
            sliderValues.stealSpeed = value
        elseif text == "INF JUMP" then
            sliderValues.infJump = value
        elseif text == "GRAVITY" then
            sliderValues.gravity = value
        elseif text == "SPIN" then
            sliderValues.spin = value
        end
        if initialized then
            pcall(SaveConfigs)
        end
    end

    local function update(x)
        local absPos = bar.AbsolutePosition.X
        local absSize = bar.AbsoluteSize.X
        if absSize > 0 then
            setPercent((x-absPos)/absSize)
        end
    end

    thumb.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            if currentlyDraggingSlider ~= nil then return end
            dragging = true
            dragInput = input
            currentlyDraggingSlider = thumb
        end
    end)

    bar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            if currentlyDraggingSlider ~= nil then return end
            dragging = true
            dragInput = input
            currentlyDraggingSlider = thumb
            update(input.Position.X)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            if input.Position then
                update(input.Position.X)
            end
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input == dragInput then
            dragging = false
            dragInput = nil
            currentlyDraggingSlider = nil
        end
    end)

    local initialized = false
    setPercent((default-min)/(max-min))
    initialized = true
end

-- TOGGLE FUNCTION
local function CreateToggle(text,callback)
    local holder = Instance.new("Frame")
    holder.Size = UDim2.new(1,-15,0,50)
    holder.BackgroundTransparency = 1
    holder.Parent = Scroll

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.65,0,1,0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14
    label.TextColor3 = Color3.new(1,1,1)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = holder

    local toggle = Instance.new("Frame")
    toggle.Size = UDim2.new(0,50,0,26)
    toggle.Position = UDim2.new(1,-55,0.5,-13)
    toggle.BackgroundColor3 = Color3.fromRGB(40,60,90)
    toggle.Parent = holder
    Instance.new("UICorner",toggle).CornerRadius = UDim.new(1,0)

    local circle = Instance.new("Frame")
    circle.Size = UDim2.new(0,22,0,22)
    circle.Position = UDim2.new(0,2,0.5,-11)
    circle.BackgroundColor3 = Color3.new(1,1,1)
    circle.Parent = toggle
    Instance.new("UICorner",circle).CornerRadius = UDim.new(1,0)

    local state = false
    if toggleStates[text] ~= nil then
        state = toggleStates[text]
    end

    if state then
        toggle.BackgroundColor3 = Color3.fromRGB(0,140,255)
        circle.Position = UDim2.new(1,-24,0.5,-11)
    else
        toggle.BackgroundColor3 = Color3.fromRGB(40,60,90)
        circle.Position = UDim2.new(0,2,0.5,-11)
    end

    toggle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            state = not state
            if state then
                TweenService:Create(toggle,TweenInfo.new(0.25),{BackgroundColor3 = Color3.fromRGB(0,140,255)}):Play()
                TweenService:Create(circle,TweenInfo.new(0.25),{Position = UDim2.new(1,-24,0.5,-11)}):Play()
            else
                TweenService:Create(toggle,TweenInfo.new(0.25),{BackgroundColor3 = Color3.fromRGB(40,60,90)}):Play()
                TweenService:Create(circle,TweenInfo.new(0.25),{Position = UDim2.new(0,2,0.5,-11)}):Play()
            end
            if callback then callback(state) end
            toggleStates[text] = state
            SaveConfigs()
        end
    end)
    if state and callback then
        task.defer(function() pcall(callback, state) end)
    end
end

CreateSlider("AUTO SPEED",30,60,sliderValues.autoSpeed)
CreateSlider("STEAL SPEED",25,29.5,sliderValues.stealSpeed)
CreateSlider("INF JUMP",50,53.7,sliderValues.infJump)
CreateSlider("GRAVITY",0,30.5,sliderValues.gravity)
CreateSlider("SPIN",0,50,sliderValues.spin)

CreateToggle("SAVE CONFIG")

CreateToggle("POTATO MODE",function(state)
    PotatoEnabled = state
    if state then
        EnablePotato()
        -- aplica ghost nas plots quando Potato for ligado
        pcall(function()
            applyGhostToAllPlots()
            ensurePlotsListener()
        end)
    else
        -- desconecta listener e restaura ghosts, depois desliga potato
        pcall(function()
            disconnectPlotsListener()
            restoreGhosts()
        end)
        DisablePotato()
    end
end)

local function openSettingsGui()
    SettingsFrame.Visible = true
    TweenService:Create(SettingsUIScale, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
    TweenService:Create(SettingsFrame, TweenInfo.new(0.25), {BackgroundTransparency = 0.08}):Play()
end

local function closeSettingsGui()
    local tween = TweenService:Create(SettingsUIScale, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Scale = 0})
    tween:Play()
    tween.Completed:Connect(function()
        SettingsFrame.Visible = false
    end)
end

local settingsOpened = false
GearButton.MouseButton1Click:Connect(function()
    TweenService:Create(GearButton, TweenInfo.new(0.5, Enum.EasingStyle.Linear), {Rotation = GearButton.Rotation + 360}):Play()
    settingsOpened = not settingsOpened
    if settingsOpened then
        openSettingsGui()
    else
        closeSettingsGui()
    end
end)

local Container = Instance.new("Frame")
Container.Size = UDim2.new(1,-20,1,-70)
Container.Position = UDim2.new(0,10,0,60)
Container.BackgroundTransparency = 1
Container.Parent = MainFrame

local Grid = Instance.new("UIGridLayout")
Grid.CellSize = UDim2.new(0.48,0,0,40)
Grid.CellPadding = UDim2.new(0.04,0,0.04,0)
Grid.Parent = Container

-- LEFT PANEL
local LeftPanel = Instance.new("Frame")
LeftPanel.Name = "LeftPanel"
LeftPanel.Size = UDim2.new(0,180,0,120)
LeftPanel.Position = UDim2.new(0,12,0.5,-60)
LeftPanel.AnchorPoint = Vector2.new(0,0.5)
LeftPanel.BackgroundColor3 = Color3.fromRGB(8, 20, 50)
LeftPanel.BackgroundTransparency = 0.12
LeftPanel.Parent = ScreenGui
LeftPanel.Visible = false
Instance.new("UICorner", LeftPanel).CornerRadius = UDim.new(0,10)

local LeftStroke = Instance.new("UIStroke", LeftPanel)
LeftStroke.Color = Color3.fromRGB(40,40,40)
LeftStroke.Transparency = 0.3
LeftStroke.Thickness = 1

local LeftPanelScale = Instance.new("UIScale", LeftPanel)
LeftPanelScale.Scale = 0

local LeftTitle = Instance.new("TextLabel")
LeftTitle.Size = UDim2.new(1,0,0,32)
LeftTitle.Position = UDim2.new(0,0,0,8)
LeftTitle.BackgroundTransparency = 1
LeftTitle.Text = "LEFT"
LeftTitle.TextSize = 17
LeftTitle.Font = Enum.Font.GothamBold
LeftTitle.TextColor3 = Color3.fromRGB(100,180,255)
LeftTitle.TextTransparency = 0.05
LeftTitle.Parent = LeftPanel

local LeftActivate = Instance.new("TextButton")
LeftActivate.Size = UDim2.new(0.9,0,0,38)
LeftActivate.Position = UDim2.new(0.05,0,0,65)
LeftActivate.BackgroundColor3 = Color3.fromRGB(0,122,204)
LeftActivate.TextColor3 = Color3.new(1,1,1)
LeftActivate.Text = "Activate"
LeftActivate.Font = Enum.Font.GothamBold
LeftActivate.TextSize = 15
LeftActivate.Parent = LeftPanel
Instance.new("UICorner", LeftActivate).CornerRadius = UDim.new(0,8)

-- RIGHT PANEL
local RightPanel = Instance.new("Frame")
RightPanel.Name = "RightPanel"
RightPanel.Size = UDim2.new(0,180,0,120)
RightPanel.Position = UDim2.new(1,-10,0.5,-60)
RightPanel.AnchorPoint = Vector2.new(1,0.5)
RightPanel.BackgroundColor3 = Color3.fromRGB(8, 20, 50)
RightPanel.BackgroundTransparency = 0.12
RightPanel.Parent = ScreenGui
RightPanel.Visible = false
Instance.new("UICorner", RightPanel).CornerRadius = UDim.new(0,10)

local RightStroke = Instance.new("UIStroke", RightPanel)
RightStroke.Color = Color3.fromRGB(40,40,40)
RightStroke.Transparency = 0.3
RightStroke.Thickness = 1

local RightPanelScale = Instance.new("UIScale", RightPanel)
RightPanelScale.Scale = 0

local RightTitle = Instance.new("TextLabel")
RightTitle.Size = UDim2.new(1,0,0,32)
RightTitle.Position = UDim2.new(0,0,0,8)
RightTitle.BackgroundTransparency = 1
RightTitle.Text = "RIGHT"
RightTitle.TextSize = 17
RightTitle.Font = Enum.Font.GothamBold
RightTitle.TextColor3 = Color3.fromRGB(100,180,255)
RightTitle.TextTransparency = 0.05
RightTitle.Parent = RightPanel

local RightActivate = Instance.new("TextButton", RightPanel)
RightActivate.Size = UDim2.new(0.9,0,0,38)
RightActivate.Position = UDim2.new(0.05,0,0,65)
RightActivate.BackgroundColor3 = Color3.fromRGB(0,122,204)
RightActivate.TextColor3 = Color3.new(1,1,1)
RightActivate.Text = "Activate"
RightActivate.Font = Enum.Font.GothamBold
RightActivate.TextSize = 15
Instance.new("UICorner", RightActivate).CornerRadius = UDim.new(0,8)

local function openLeftPanel()
    LeftPanel.Visible = true
    TweenService:Create(LeftPanelScale, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
    TweenService:Create(LeftPanel, TweenInfo.new(0.22), {BackgroundTransparency = 0.08}):Play()
end

local function closeLeftPanel()
    local t = TweenService:Create(LeftPanelScale, TweenInfo.new(0.22, Enum.EasingStyle.Quad), {Scale = 0})
    t:Play()
    t.Completed:Wait()
    LeftPanel.Visible = false
end

local function openRightPanel()
    RightPanel.Visible = true
    TweenService:Create(RightPanelScale, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
    TweenService:Create(RightPanel, TweenInfo.new(0.22), {BackgroundTransparency = 0.08}):Play()
end

local function closeRightPanel()
    local t = TweenService:Create(RightPanelScale, TweenInfo.new(0.22, Enum.EasingStyle.Quad), {Scale = 0})
    t:Play()
    t.Completed:Wait()
    RightPanel.Visible = false
end

-- Jump system variables
local character, hrp, hum
local jumpActive = false

local function setupCharacter(char)
    character = char
    hrp = char:WaitForChild("HumanoidRootPart")
    hum = char:WaitForChild("Humanoid")
end

if player.Character then setupCharacter(player.Character) end
player.CharacterAdded:Connect(setupCharacter)

UserInputService.JumpRequest:Connect(function()
    if jumpActive and hrp then
        pcall(function()
            hrp.AssemblyLinearVelocity = Vector3.new(
                hrp.AssemblyLinearVelocity.X,
                sliderValues.infJump,
                hrp.AssemblyLinearVelocity.Z
            )
        end)
    end
end)

-- ==================== GRAVITY CONTROLLER LOGIC ====================
local DEFAULT_GRAVITY = 196.2
local HOP_POWER = 35
local HOP_COOLDOWN = 0.08

local galaxyVectorForce = nil
local galaxyAttachment = nil
local galaxyEnabled = false
local hopsEnabled = false
local lastHopTime = 0
local spaceHeld = false
local originalJumpPower = 50

local function setupGalaxyForce()
    pcall(function()
        local c = player.Character
        if not c then return end
        local h = c:FindFirstChild("HumanoidRootPart")
        if not h then return end
        if galaxyVectorForce then galaxyVectorForce:Destroy() end
        if galaxyAttachment then galaxyAttachment:Destroy() end
        galaxyAttachment = Instance.new("Attachment")
        galaxyAttachment.Parent = h
        galaxyVectorForce = Instance.new("VectorForce")
        galaxyVectorForce.Attachment0 = galaxyAttachment
        galaxyVectorForce.ApplyAtCenterOfMass = true
        galaxyVectorForce.RelativeTo = Enum.ActuatorRelativeTo.World
        galaxyVectorForce.Force = Vector3.new(0, 0, 0)
        galaxyVectorForce.Parent = h
    end)
end

local function updateGalaxyForce()
    if not galaxyEnabled or not galaxyVectorForce then return end
    local c = player.Character
    if not c then return end
    local mass = 0
    for _, p in ipairs(c:GetDescendants()) do
        if p:IsA("BasePart") then
            mass = mass + p:GetMass()
        end
    end
    -- Slider 0-50: 0 = 100% gravidade (normal), 50 = mínima gravidade (leve)
    -- Quanto MAIOR o slider, MENOR a gravidade
    local percentGravity = (50 - sliderValues.gravity) / 50
    local targetG = DEFAULT_GRAVITY * percentGravity
    local compensate = mass * (DEFAULT_GRAVITY - targetG)
    
    galaxyVectorForce.Force = Vector3.new(0, compensate * 0.96, 0)
end

local function adjustGalaxyJump()
    pcall(function()
        local c = player.Character
        if not c then return end
        local hum = c:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        if not galaxyEnabled then
            hum.JumpPower = originalJumpPower
            return
        end
        local percentGravity = (50 - sliderValues.gravity) / 50
        local ratio = math.sqrt((DEFAULT_GRAVITY * percentGravity) / DEFAULT_GRAVITY)
        hum.JumpPower = originalJumpPower * ratio
    end)
end

local function doMiniHop()
    if not hopsEnabled then return end
    pcall(function()
        local c = player.Character
        if not c then return end
        local h = c:FindFirstChild("HumanoidRootPart")
        local hum = c:FindFirstChildOfClass("Humanoid")
        if not h or not hum then return end
        if tick() - lastHopTime < HOP_COOLDOWN then return end
        lastHopTime = tick()
        if hum.FloorMaterial == Enum.Material.Air then
            h.AssemblyLinearVelocity = Vector3.new(h.AssemblyLinearVelocity.X, HOP_POWER, h.AssemblyLinearVelocity.Z)
        end
    end)
end

local function captureJumpPower()
    local c = player.Character
    if c then
        local hum = c:FindFirstChildOfClass("Humanoid")
        if hum and hum.JumpPower > 0 then
            originalJumpPower = hum.JumpPower
        end
    end
end

local function enableGravity()
    galaxyEnabled = true
    hopsEnabled = true
    setupGalaxyForce()
    adjustGalaxyJump()
end

local function disableGravity()
    galaxyEnabled = false
    hopsEnabled = false
    if galaxyVectorForce then
        galaxyVectorForce:Destroy()
        galaxyVectorForce = nil
    end
    if galaxyAttachment then
        galaxyAttachment:Destroy()
        galaxyAttachment = nil
    end
    adjustGalaxyJump()
end

task.spawn(function()
    task.wait(1)
    captureJumpPower()
end)

player.CharacterAdded:Connect(function(char)
    task.wait(1)
    captureJumpPower()
    if galaxyEnabled then
        task.wait(0.1)
        setupGalaxyForce()
        adjustGalaxyJump()
    end
end)

RunService.Heartbeat:Connect(function()
    if hopsEnabled and spaceHeld then
        doMiniHop()
    end
    if galaxyEnabled then
        updateGalaxyForce()
    end
end)

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.Space then
        spaceHeld = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Space then
        spaceHeld = false
    end
end)

-- XRAY
local xrayEnabled = false
local xrayGen = 0
local xrayOriginalTransparency = setmetatable({}, { __mode = "k" })
local xrayRestoring = false
local xrayLastToggle = 0

local function applyXrayForGen(gen)
    if gen ~= xrayGen or xrayRestoring then return end
    local Plots = Workspace:FindFirstChild("Plots")
    if not Plots then return end
    for _, Plot in ipairs(Plots:GetChildren()) do
        if gen ~= xrayGen or xrayRestoring then return end
        if Plot:IsA("Model") and Plot:FindFirstChild("Decorations") then
            for _, Part in ipairs(Plot.Decorations:GetDescendants()) do
                if gen ~= xrayGen or xrayRestoring then return end
                if Part:IsA("BasePart") then
                    if xrayOriginalTransparency[Part] == nil then
                        xrayOriginalTransparency[Part] = Part.Transparency
                    end
                    pcall(function() Part.Transparency = 0.8 end)
                end
            end
        end
    end
end

RunService.Heartbeat:Connect(function()
    if xrayEnabled and not xrayRestoring then
        local gen = xrayGen
        applyXrayForGen(gen)
    end
end)

local function restoreNow()
    if xrayRestoring then return end
    xrayRestoring = true
    xrayGen = xrayGen + 1
    xrayEnabled = false
    for part, value in pairs(xrayOriginalTransparency) do
        if part and typeof(part) == "Instance" and part:IsA("BasePart") then
            pcall(function() part.Transparency = value end)
        end
    end
    xrayOriginalTransparency = setmetatable({}, { __mode = "k" })
    xrayRestoring = false
end

-- UNWALK
local unwalkEnabled = false
local unwalkSavedAnims = {}
local unwalkWatcher = nil

local function saveAndRemoveAnimationInstance(anim)
    if not anim or not anim:IsA("Animation") then return end
    for _, v in ipairs(unwalkSavedAnims) do
        if v.instance == anim then return end
    end
    table.insert(unwalkSavedAnims, {instance = anim, id = anim.AnimationId})
    pcall(function() anim.AnimationId = "" end)
end

local function stopPlayingTracks(hum)
    if not hum then return end
    for _, t in ipairs(hum:GetPlayingAnimationTracks()) do
        pcall(function() t:Stop() end)
    end
end

local function scanAndRemoveAllAnimations(character)
    if not character then return end
    for _, desc in ipairs(character:GetDescendants()) do
        if desc:IsA("Animation") then
            saveAndRemoveAnimationInstance(desc)
        end
    end
    local humLocal = character:FindFirstChildOfClass("Humanoid")
    if humLocal then
        stopPlayingTracks(humLocal)
    end
end

local function onDescendantAdded(desc)
    if unwalkEnabled and desc and desc:IsA("Animation") then
        saveAndRemoveAnimationInstance(desc)
    end
end

local function enableUnwalk()
    if unwalkEnabled then return end
    unwalkSavedAnims = {}
    local char = player.Character
    if char then
        scanAndRemoveAllAnimations(char)
        if unwalkWatcher then unwalkWatcher:Disconnect() unwalkWatcher = nil end
        unwalkWatcher = char.DescendantAdded:Connect(onDescendantAdded)
    end
    unwalkEnabled = true
end

local function disableUnwalk()
    if unwalkWatcher then unwalkWatcher:Disconnect() unwalkWatcher = nil end
    for _, v in ipairs(unwalkSavedAnims) do
        if v.instance and v.id then
            pcall(function() v.instance.AnimationId = v.id end)
        end
    end
    unwalkSavedAnims = {}
    unwalkEnabled = false
end

player.CharacterAdded:Connect(function(c)
    if unwalkEnabled then
        task.spawn(function()
            scanAndRemoveAllAnimations(c)
            if unwalkWatcher then unwalkWatcher:Disconnect() end
            unwalkWatcher = c.DescendantAdded:Connect(onDescendantAdded)
        end)
    end
end)

-- ESP PLAYER
local espConnections = {}
local espEnabled = false

local function createESP(plr)
    if plr == player then return end
    if not plr.Character then return end
    if plr.Character:FindFirstChild("ESP_Hitbox") then return end

    local char = plr.Character
    local hrpLocal = char:FindFirstChild("HumanoidRootPart")
    local head = char:FindFirstChild("Head")
    if not (hrpLocal and head) then return end

    local hitbox = Instance.new("BoxHandleAdornment")
    hitbox.Name = "ESP_Hitbox"
    hitbox.Adornee = hrpLocal
    hitbox.Size = Vector3.new(4, 6, 2)
    hitbox.Color3 = Color3.fromRGB(0, 0, 128)
    hitbox.Transparency = 0.6
    hitbox.ZIndex = 10
    hitbox.AlwaysOnTop = true
    hitbox.Parent = char

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Name"
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 150, 0, 30)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = char

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = plr.DisplayName or plr.Name
    label.TextColor3 = Color3.fromRGB(0, 0, 128)
    label.Font = Enum.Font.GothamBold
    label.TextScaled = true
    label.TextSize = 16
    label.TextStrokeTransparency = 0.5
    label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    label.Parent = billboard
end

local function removeESP(plr)
    if not plr.Character then return end
    local hitbox = plr.Character:FindFirstChild("ESP_Hitbox")
    local nameGui = plr.Character:FindFirstChild("ESP_Name")
    if hitbox then hitbox:Destroy() end
    if nameGui then nameGui:Destroy() end
end

local function enableESP()
    espEnabled = true
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= player then
            if plr.Character then
                createESP(plr)
            end
            local conn = plr.CharacterAdded:Connect(function()
                task.wait(0.1)
                if espEnabled then
                    createESP(plr)
                end
            end)
            table.insert(espConnections, conn)
        end
    end
    local playerAddedConn = Players.PlayerAdded:Connect(function(plr)
        if plr == player then return end
        local charAddedConn = plr.CharacterAdded:Connect(function()
            task.wait(0.1)
            if espEnabled then
                createESP(plr)
            end
        end)
        table.insert(espConnections, charAddedConn)
    end)
    table.insert(espConnections, playerAddedConn)
end

local function disableESP()
    espEnabled = false
    for _, plr in ipairs(Players:GetPlayers()) do
        removeESP(plr)
    end
    for _, conn in ipairs(espConnections) do
        if conn and conn.Connected then
            conn:Disconnect()
        end
    end
    espConnections = {}
end

-- ANTI RAGDOLL
local AntiRagdollEnabled = false
local antiConnection = nil
local antiKeysPressed = {W=false,A=false,S=false,D=false,Space=false}
local antiLastSafePos = nil
local antiLastSafeTime = 0

local function antiIsBadState(state)
    return state == Enum.HumanoidStateType.Ragdoll
        or state == Enum.HumanoidStateType.Physics
        or state == Enum.HumanoidStateType.FallingDown
        or state == Enum.HumanoidStateType.PlatformStanding
end

local function antiRemoveRagdollInstant()
    player:SetAttribute("RagdollEndTime", 0)
    
    local char = player.Character
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid then
        pcall(function()
            humanoid.PlatformStand = false
            humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
            task.wait()
            humanoid:ChangeState(Enum.HumanoidStateType.Running)
        end)
    end
    
    local camera = workspace.CurrentCamera
    if camera and humanoid and camera.CameraSubject ~= humanoid then
        pcall(function()
            camera.CameraSubject = humanoid
            camera.CameraType = Enum.CameraType.Custom
        end)
    end
end

local function antiStartSystem()
    if antiConnection then return end
    antiConnection = RunService.Heartbeat:Connect(function()
        if not AntiRagdollEnabled then return end
        
        local char = player.Character
        if not char then return end
        
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not humanoid or not root then return end
        
        local state = humanoid:GetState()
        local vel = root.AssemblyLinearVelocity
        
        -- salva posição segura
        if humanoid.FloorMaterial ~= Enum.Material.Air
            and not antiIsBadState(state)
            and vel.Magnitude < 25
            and tick() - antiLastSafeTime > 0.4 then
            
            antiLastSafePos = Vector3.new(root.Position.X, 0, root.Position.Z)
            antiLastSafeTime = tick()
        end
        
        -- remove ragdoll instantâneo
        if antiIsBadState(state) then
            antiRemoveRagdollInstant()
            
            -- reativa motores
            for _, m in char:GetDescendants() do
                if m:IsA("Motor6D") and not m.Enabled then
                    m.Enabled = true
                end
            end
        end
        
        -- anti launch forte
        if antiIsBadState(state) and antiLastSafePos and vel.Magnitude > 40 then
            local y = root.Position.Y
            pcall(function() root.CFrame = CFrame.new(antiLastSafePos.X, y, antiLastSafePos.Z) end)
            local newY = math.abs(vel.Y) > 40 and 0 or vel.Y
            pcall(function()
                root.AssemblyLinearVelocity = Vector3.new(0, newY, 0)
                root.AssemblyAngularVelocity = Vector3.zero
            end)
        end
        
        -- movimento forçado
        humanoid.PlatformStand = false
        humanoid.Sit = false
        
        local move = Vector3.new()
        if antiKeysPressed.W then move += Vector3.new(0,0,-1) end
        if antiKeysPressed.S then move += Vector3.new(0,0,1) end
        if antiKeysPressed.A then move += Vector3.new(-1,0,0) end
        if antiKeysPressed.D then move += Vector3.new(1,0,0) end
        
        if move.Magnitude > 0 then
            pcall(function() humanoid:Move(move.Unit, true) end)
        else
            pcall(function() humanoid:Move(Vector3.zero) end)
        end
        
        humanoid.Jump = antiKeysPressed.Space
        
        -- anti spin
        if root.AssemblyAngularVelocity.Magnitude > 5 then
            pcall(function() root.AssemblyAngularVelocity = Vector3.new(0, root.AssemblyAngularVelocity.Y, 0) end)
        end
    end)
end

local function antiStopSystem()
    if antiConnection then
        antiConnection:Disconnect()
        antiConnection = nil
    end
end

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    local kc = input.KeyCode
    if kc == Enum.KeyCode.W then antiKeysPressed.W = true
    elseif kc == Enum.KeyCode.A then antiKeysPressed.A = true
    elseif kc == Enum.KeyCode.S then antiKeysPressed.S = true
    elseif kc == Enum.KeyCode.D then antiKeysPressed.D = true
    elseif kc == Enum.KeyCode.Space then antiKeysPressed.Space = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    local kc = input.KeyCode
    if kc == Enum.KeyCode.W then antiKeysPressed.W = false
    elseif kc == Enum.KeyCode.A then antiKeysPressed.A = false
    elseif kc == Enum.KeyCode.S then antiKeysPressed.S = false
    elseif kc == Enum.KeyCode.D then antiKeysPressed.D = false
    elseif kc == Enum.KeyCode.Space then antiKeysPressed.Space = false
    end
end)

player.CharacterAdded:Connect(function()
    task.wait(0.6)
    if AntiRagdollEnabled then
        antiStartSystem()
    end
end)

-- WAVE BAT AUTO
local WaveBat = {}
WaveBat.SafeFollow = {ENABLED = false, FOLLOW_DISTANCE = 6}
WaveBat.Aimbot = {ENABLED = false, RANGE = 7.25}

local WAVE_ACTIVE_WALKSPEED = 50
local WAVE_CLOSE_DISTANCE = 1

local wave_speedConn = nil
local wave_hb = nil
local wave_lookState = {}
local wave_storedWalk = {}

local function wave_getNearestPlayer(hrp)
    local nearest, dist = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local other = p.Character.HumanoidRootPart
            local d = (Vector3.new(other.Position.X,0,other.Position.Z) - Vector3.new(hrp.Position.X,0,hrp.Position.Z)).Magnitude
            if d < dist then
                dist = d
                nearest = p.Character
            end
        end
    end
    return nearest, dist
end

local function wave_stopSpeed()
    if wave_speedConn then
        pcall(function() wave_speedConn:Disconnect() end)
        wave_speedConn = nil
    end
end

local function wave_startSpeedTowards(getRootFn, targetPosFn, speed)
    wave_stopSpeed()
    wave_speedConn = RunService.RenderStepped:Connect(function()
        local root = getRootFn()
        if not root or not root.Parent then
            wave_stopSpeed()
            return
        end
        local targetPos = targetPosFn()
        if not targetPos then
            root.AssemblyLinearVelocity = Vector3.zero
            return
        end
        local dir = targetPos - root.Position
        local flat = Vector3.new(dir.X, 0, dir.Z)
        if flat.Magnitude > 0.2 then
            local unit = flat.Unit
            root.AssemblyLinearVelocity = Vector3.new(unit.X * speed, root.AssemblyLinearVelocity.Y, unit.Z * speed)
        else
            root.AssemblyLinearVelocity = Vector3.zero
        end
        root.AssemblyAngularVelocity = Vector3.zero
    end)
end

local function wave_ensureLookSetup(char)
    if wave_lookState.align and wave_lookState.align.Parent and wave_lookState.lookPart and wave_lookState.lookPart.Parent then return end
    local hrpLocal = char:FindFirstChild("HumanoidRootPart")
    if not hrpLocal then return end
    local lp = Instance.new("Part")
    lp.Name = "WaveBat_LookPart"
    lp.Size = Vector3.new(0.2,0.2,0.2)
    lp.Transparency = 1
    lp.Anchored = true
    lp.CanCollide = false
    lp.Parent = workspace

    local att0 = Instance.new("Attachment", hrpLocal)
    att0.Name = "WaveBat_HRP_Att"
    att0.Position = Vector3.new(0,0,0)
    local att1 = Instance.new("Attachment", lp)
    att1.Name = "WaveBat_Target_Att"

    local align = Instance.new("AlignOrientation")
    align.Attachment0 = att0
    align.Attachment1 = att1
    align.RigidityEnabled = true
    align.MaxTorque = 9e8
    align.Responsiveness = 200
    align.Parent = hrpLocal

    wave_lookState.lookPart = lp
    wave_lookState.attHRP = att0
    wave_lookState.attTarget = att1
    wave_lookState.align = align
end

local function wave_cleanupLook()
    if wave_lookState.align then pcall(function() wave_lookState.align:Destroy() end) end
    if wave_lookState.attHRP then pcall(function() wave_lookState.attHRP:Destroy() end) end
    if wave_lookState.lookPart then pcall(function() wave_lookState.lookPart:Destroy() end) end
    wave_lookState = {}
end

local function wave_storeWalk(hum)
    if not hum then return end
    wave_storedWalk[hum] = wave_storedWalk[hum] or hum.WalkSpeed
end

local function wave_restoreWalk(hum)
    if not hum then return end
    if wave_storedWalk[hum] then
        pcall(function() hum.WalkSpeed = wave_storedWalk[hum] end)
        wave_storedWalk[hum] = nil
    end
end

local function wave_fastEquipAndActivateTool(char, humanoid)
    if not char then return end
    local tool = char:FindFirstChild("Bat") or char:FindFirstChild("Medusa")
    if not tool then
        local bp = player:FindFirstChild("Backpack")
        if bp then
            tool = bp:FindFirstChild("Bat") or bp:FindFirstChild("Medusa")
            if tool and humanoid and typeof(humanoid.EquipTool) == "function" then
                pcall(function() humanoid:EquipTool(tool) end)
                task.wait(0.01)
            end
        end
    end
    if tool and tool.Parent == char and typeof(tool.Activate) == "function" then
        pcall(function() tool:Activate() end)
    end
end

local function WaveBat_Start()
    WaveBat.SafeFollow.ENABLED = true
    WaveBat.Aimbot.ENABLED = true
    if wave_hb and wave_hb.Connected then return end
    wave_hb = RunService.Heartbeat:Connect(function()
        local char = player.Character
        if not char then
            wave_cleanupLook()
            wave_stopSpeed()
            return
        end
        local hrpLocal = char:FindFirstChild("HumanoidRootPart")
        local humLocal = char:FindFirstChildOfClass("Humanoid")
        if not (hrpLocal and humLocal) then
            wave_cleanupLook()
            wave_stopSpeed()
            return
        end

        pcall(function() if humLocal.Health <= 1 then humLocal.Health = humLocal.MaxHealth end end)

        if WaveBat.SafeFollow.ENABLED then
            local targetChar, dist = wave_getNearestPlayer(hrpLocal)
            if targetChar and targetChar:FindFirstChild("HumanoidRootPart") then
                local trg = targetChar.HumanoidRootPart
                local flatDir = Vector3.new(trg.Position.X - hrpLocal.Position.X, 0, trg.Position.Z - hrpLocal.Position.Z)
                local d = flatDir.Magnitude
                local desired = WAVE_CLOSE_DISTANCE

                local followPos = trg.Position - (flatDir.Magnitude > 0 and flatDir.Unit * desired or Vector3.new(0,0,0))

                wave_startSpeedTowards(
                    function() return hrpLocal end,
                    function() return followPos end,
                    WAVE_ACTIVE_WALKSPEED
                )

                wave_ensureLookSetup(char)
                if wave_lookState.lookPart and wave_lookState.lookPart.Parent then
                    local lookPos = Vector3.new(trg.Position.X, hrpLocal.Position.Y, trg.Position.Z)
                    wave_lookState.lookPart.CFrame = CFrame.new(hrpLocal.Position, lookPos)
                end

                if d <= desired + 0.2 then
                    hrpLocal.AssemblyLinearVelocity = Vector3.zero
                end
            else
                wave_stopSpeed()
                wave_restoreWalk(humLocal)
                wave_cleanupLook()
            end
        else
            wave_stopSpeed()
            wave_restoreWalk(humLocal)
            wave_cleanupLook()
        end

        if WaveBat.Aimbot.ENABLED then
            local targ, dmin = nil, WaveBat.Aimbot.RANGE
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    local d = (p.Character.HumanoidRootPart.Position - hrpLocal.Position).Magnitude
                    if d <= dmin then
                        targ = p.Character.HumanoidRootPart
                        dmin = d
                    end
                end
            end
            if targ then
                wave_ensureLookSetup(char)
                if wave_lookState.lookPart and wave_lookState.lookPart.Parent then
                    local lookPos = Vector3.new(targ.Position.X, hrpLocal.Position.Y, targ.Position.Z)
                    wave_lookState.lookPart.CFrame = CFrame.new(hrpLocal.Position, lookPos)
                end
                wave_fastEquipAndActivateTool(char, humLocal)
            end
        end
    end)
end

local function WaveBat_Stop()
    WaveBat.SafeFollow.ENABLED = false
    WaveBat.Aimbot.ENABLED = false
    wave_stopSpeed()
    wave_cleanupLook()
    if wave_hb then pcall(function() wave_hb:Disconnect() end) wave_hb = nil end
    wave_storedWalk = {}
end

player.CharacterAdded:Connect(function()
    task.wait(0.1)
    wave_storedWalk = {}
    wave_cleanupLook()
    wave_stopSpeed()
end)

player.AncestryChanged:Connect(function()
    if not player:IsDescendantOf(game) then
        if wave_hb and wave_hb.Connected then wave_hb:Disconnect() end
        wave_cleanupLook()
        wave_stopSpeed()
    end
end)

-- CAMERA FOLLOW (com rotação do player)
local camFollowEnabled = false
local camFollowConnection = nil
local cam_lookState = {}

local function getHRP()
    local c = player.Character
    return c and c:FindFirstChild("HumanoidRootPart")
end

local function getClosestPlayer()
    local root = getHRP()
    if not root then return nil end
    local closest, best = nil, math.huge
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (root.Position - p.Character.HumanoidRootPart.Position).Magnitude
            if dist < best then
                best = dist
                closest = p.Character.HumanoidRootPart
            end
        end
    end
    return closest
end

local function cam_ensureLookSetup(char)
    if cam_lookState.align and cam_lookState.align.Parent and cam_lookState.lookPart and cam_lookState.lookPart.Parent then return end
    local hrpLocal = char:FindFirstChild("HumanoidRootPart")
    if not hrpLocal then return end
    local lp = Instance.new("Part")
    lp.Name = "CamFollow_LookPart"
    lp.Size = Vector3.new(0.2,0.2,0.2)
    lp.Transparency = 1
    lp.Anchored = true
    lp.CanCollide = false
    lp.Parent = workspace

    local att0 = Instance.new("Attachment", hrpLocal)
    att0.Name = "CamFollow_HRP_Att"
    att0.Position = Vector3.new(0,0,0)
    local att1 = Instance.new("Attachment", lp)
    att1.Name = "CamFollow_Target_Att"

    local align = Instance.new("AlignOrientation")
    align.Attachment0 = att0
    align.Attachment1 = att1
    align.RigidityEnabled = true
    align.MaxTorque = 9e8
    align.Responsiveness = 200
    align.Parent = hrpLocal

    cam_lookState.lookPart = lp
    cam_lookState.attHRP = att0
    cam_lookState.attTarget = att1
    cam_lookState.align = align
end

local function cam_cleanupLook()
    if cam_lookState.align then pcall(function() cam_lookState.align:Destroy() end) end
    if cam_lookState.attHRP then pcall(function() cam_lookState.attHRP:Destroy() end) end
    if cam_lookState.lookPart then pcall(function() cam_lookState.lookPart:Destroy() end) end
    cam_lookState = {}
end

local function startCamFollow()
    if camFollowConnection then camFollowConnection:Disconnect() camFollowConnection = nil end
    camFollowConnection = RunService.Heartbeat:Connect(function()
        if not camFollowEnabled then return end
        local root = getHRP()
        if not root then return end
        local target = getClosestPlayer()
        if not target or not target.Parent then
            cam_cleanupLook()
            return
        end

        cam_ensureLookSetup(player.Character)
        if cam_lookState.lookPart and cam_lookState.lookPart.Parent then
            local lookPos = Vector3.new(target.Position.X, root.Position.Y, target.Position.Z)
            pcall(function()
                cam_lookState.lookPart.CFrame = CFrame.new(root.Position, lookPos)
            end)
        end
    end)
end

local function stopCamFollow()
    if camFollowConnection then camFollowConnection:Disconnect() camFollowConnection = nil end
    cam_cleanupLook()
end

-- SPIN logic
local spinActive = false

local function enableSpin()
    spinActive = true
end

local function disableSpin()
    spinActive = false
    local r = getHRP()
    if r then
        pcall(function() r.AssemblyAngularVelocity = Vector3.new(0,0,0) end)
    end
    if player.Character then
        local hh = player.Character:FindFirstChildOfClass("Humanoid")
        if hh then pcall(function() hh.AutoRotate = true end) end
    end
end

RunService.RenderStepped:Connect(function()
    if spinActive then
        local r = getHRP()
        if r then
            pcall(function()
                -- Multiplicar por 2 para aumentar intensidade: 0-50 slider = 0-100 rotação
                local intensity = sliderValues.spin * 2
                r.AssemblyAngularVelocity = Vector3.new(0, intensity, 0)
            end)
            local hh = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
            if hh then pcall(function() hh.AutoRotate = false end) end
        end
    end
end)

-- AIM ASSIST
local aimEnabled = false
local aim_lookState = {}
local aimConnection = nil

local function aim_ensureLookSetup(char)
    if aim_lookState.align and aim_lookState.align.Parent and aim_lookState.lookPart and aim_lookState.lookPart.Parent then return end
    local hrpLocal = char:FindFirstChild("HumanoidRootPart")
    if not hrpLocal then return end
    local lp = Instance.new("Part")
    lp.Name = "Aim_LookPart"
    lp.Size = Vector3.new(0.2,0.2,0.2)
    lp.Transparency = 1
    lp.Anchored = true
    lp.CanCollide = false
    lp.Parent = workspace

    local att0 = Instance.new("Attachment", hrpLocal)
    att0.Name = "Aim_HRP_Att"
    att0.Position = Vector3.new(0,0,0)
    local att1 = Instance.new("Attachment", lp)
    att1.Name = "Aim_Target_Att"

    local align = Instance.new("AlignOrientation")
    align.Attachment0 = att0
    align.Attachment1 = att1
    align.RigidityEnabled = true
    align.MaxTorque = 9e8
    align.Responsiveness = 200
    align.Parent = hrpLocal

    aim_lookState.lookPart = lp
    aim_lookState.attHRP = att0
    aim_lookState.attTarget = att1
    aim_lookState.align = align
end

local function aim_cleanupLook()
    if aim_lookState.align then pcall(function() aim_lookState.align:Destroy() end) end
    if aim_lookState.attHRP then pcall(function() aim_lookState.attHRP:Destroy() end) end
    if aim_lookState.lookPart then pcall(function() aim_lookState.lookPart:Destroy() end) end
    aim_lookState = {}
end

local function getClosestPlayer()
    local myChar = player.Character
    local myHead = myChar and myChar:FindFirstChild("Head")
    if not myHead then return nil end
    local closest, best = nil, math.huge
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= player and pl.Character and pl.Character:FindFirstChild("Head") and pl.Character:FindFirstChild("HumanoidRootPart") then
            local head = pl.Character.Head
            local dist = (myHead.Position - head.Position).Magnitude
            if dist < best then
                best = dist
                closest = pl
            end
        end
    end
    return closest
end

local function startAimAssist()
    if aimConnection then aimConnection:Disconnect() aimConnection = nil end
    aimConnection = RunService.Heartbeat:Connect(function()
        if not aimEnabled then return end
        if spinActive then return end
        local root = getHRP()
        if not root then return end
        local target = getClosestPlayer()
        if not target or not target.Character or not target.Character:FindFirstChild("Head") then
            local humLocal = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
            if humLocal then pcall(function() humLocal.AutoRotate = true end) end
            aim_cleanupLook()
            return
        end

        aim_ensureLookSetup(player.Character)
        if aim_lookState.lookPart and aim_lookState.lookPart.Parent then
            local targetPos = target.Character.Head.Position
            local lookAt = Vector3.new(targetPos.X, root.Position.Y, targetPos.Z)
            pcall(function()
                aim_lookState.lookPart.CFrame = CFrame.new(root.Position, lookAt)
            end)
        end
        local humLocal = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
        if humLocal then pcall(function() humLocal.AutoRotate = false end) end
    end)
end

local function stopAimAssist()
    if aimConnection then aimConnection:Disconnect() aimConnection = nil end
    local humLocal = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humLocal then pcall(function() humLocal.AutoRotate = true end) end
    aim_cleanupLook()
end

-- AUTO BAT GUI
local waveBatGui = Instance.new("ScreenGui")
waveBatGui.Name = "WaveBatUI"
waveBatGui.ResetOnSpawn = false
waveBatGui.Parent = PlayerGui

local AutoBatPanel = Instance.new("Frame")
AutoBatPanel.Size = UDim2.new(0, 200, 0, 80)
AutoBatPanel.Position = UDim2.new(0.5, -100, 0.5, -180)
AutoBatPanel.BackgroundColor3 = Color3.fromRGB(8, 20, 50)
AutoBatPanel.BorderSizePixel = 0
AutoBatPanel.Active = true
AutoBatPanel.Parent = waveBatGui
Instance.new("UICorner", AutoBatPanel).CornerRadius = UDim.new(0, 8)
AutoBatPanel.Visible = false
local panelStroke = Instance.new("UIStroke", AutoBatPanel)
panelStroke.Color = Color3.fromRGB(40,40,40)
panelStroke.Thickness = 1

local AutoBatTitle = Instance.new("TextLabel", AutoBatPanel)
AutoBatTitle.Size = UDim2.new(1, -12, 0, 28)
AutoBatTitle.Position = UDim2.new(0, 6, 0, 6)
AutoBatTitle.BackgroundTransparency = 1
AutoBatTitle.Text = "AUTO BAT"
AutoBatTitle.TextColor3 = Color3.fromRGB(100,180,255)
AutoBatTitle.Font = Enum.Font.GothamBold
AutoBatTitle.TextSize = 16

local AutoBatButton = Instance.new("TextButton", AutoBatPanel)
AutoBatButton.Size = UDim2.new(0.9, 0, 0, 32)
AutoBatButton.Position = UDim2.new(0.05, 0, 0, 40)
AutoBatButton.BackgroundColor3 = Color3.fromRGB(0,122,204)
AutoBatButton.TextColor3 = Color3.new(1,1,1)
AutoBatButton.Text = "Activate"
AutoBatButton.Font = Enum.Font.GothamBold
AutoBatButton.TextSize = 14
Instance.new("UICorner", AutoBatButton).CornerRadius = UDim.new(0,6)

createImprovedDrag(AutoBatPanel)

local autoBatRunning = false
AutoBatButton.MouseButton1Click:Connect(function()
    if not autoBatRunning then
        WaveBat_Start()
        autoBatRunning = true
        AutoBatButton.Text = "Stop"
        AutoBatButton.BackgroundColor3 = Color3.fromRGB(200,0,0)
    else
        WaveBat_Stop()
        autoBatRunning = false
        AutoBatButton.Text = "Activate"
        AutoBatButton.BackgroundColor3 = Color3.fromRGB(0,122,204)
    end
end)

-- AUTOBACK
local autoBackEnabled = false

-- Movement routes
local route1 = { pos1 = Vector3.new(-475.0, -7.0, 27.4), pos2 = Vector3.new(-485.0, -5.0, 22.0), after_pos1 = Vector3.new(-473.0, -7.0, 31.0), after_pos2 = Vector3.new(-473.0, -7.0, 100.0) }
local route2 = { pos1 = Vector3.new(-473.20, -7.00, 96.09), pos2 = Vector3.new(-483.0, -5.0, 101.0), after_pos1 = Vector3.new(-474.0, -7.00, 89.0), after_pos2 = Vector3.new(-471.0, -7.00, 25.0) }

-- AUTO STEAL
local AutoStealEnabled = true
local AutoStealRadius = 20
local AutoStealStealData = {}
local AutoStealIsStealing = false

local function AutoStealIsMyPlot(plotName)
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return false end
    local plot = plots:FindFirstChild(plotName)
    if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then
        local yb = sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then
            return yb.Enabled == true
        end
    end
    return false
end

local function AutoStealFindNearestPrompt()
    local h = getHRP()
    if not h then return nil, math.huge, nil end
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil, math.huge, nil end
    local np, nd, nn = nil, math.huge, nil
    for _, plot in ipairs(plots:GetChildren()) do
        if AutoStealIsMyPlot(plot.Name) then continue end
        local podiums = plot:FindFirstChild("AnimalPodiums") or plot:FindFirstChild("Podiums")
        if not podiums then continue end
        for _, pod in ipairs(podiums:GetChildren()) do
            local base = pod:FindFirstChild("Base")
            if not base then continue end
            local spawn = base:FindFirstChild("Spawn")
            if not spawn then continue end
            local dist = (spawn.Position - h.Position).Magnitude
            if dist < nd and dist <= AutoStealRadius then
                local att = spawn:FindFirstChild("PromptAttachment")
                if att then
                    for _, ch in ipairs(att:GetChildren()) do
                        if ch:IsA("ProximityPrompt") then
                            np, nd, nn = ch, dist, pod.Name
                            break
                        end
                    end
                end
            end
        end
    end
    return np, nd, nn
end

local function AutoStealExecuteSteal(prompt, name)
    if AutoStealIsStealing or not prompt then return end
    if not AutoStealStealData[prompt] then
        AutoStealStealData[prompt] = {hold = {}, trigger = {}, ready = true}
        if getconnections then
            pcall(function()
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do
                    if c.Function then table.insert(AutoStealStealData[prompt].hold, c.Function) end
                end
            end)
            pcall(function()
                for _, c in ipairs(getconnections(prompt.Triggered)) do
                    if c.Function then table.insert(AutoStealStealData[prompt].trigger, c.Function) end
                end
            end)
        end
    end
    local data = AutoStealStealData[prompt]
    if not data.ready then return end
    data.ready = false
    AutoStealIsStealing = true
    for _, f in ipairs(data.hold) do pcall(function() f() end) end
    for _, f in ipairs(data.trigger) do pcall(function() f() end) end
    data.ready = true
    AutoStealIsStealing = false
end

RunService.RenderStepped:Connect(function()
    if not AutoStealEnabled or AutoStealIsStealing then return end
    local p, _, n = AutoStealFindNearestPrompt()
    if p then AutoStealExecuteSteal(p, n) end
end)

-- ==================== TP RAGDOLL ====================
local tpAtivado       = false
local tp_ladoEsquerdo = true
local tp_ladoTravado  = false
local tp_isTeleporting= false
local tp_hasRecovered = true
local tp_currentZone  = nil
local tp_zoneEnterTime= 0

local tp_finalPosLeft    = Vector3.new(-483.51, -5.10, 18.89)
local tp_finalPosRight   = Vector3.new(-483.59, -5.04, 104.24)
local tp_checkpointA     = Vector3.new(-472.60, -7.00, 57.52)
local tp_checkpointBLeft = Vector3.new(-471.76, -7.00, 26.22)
local tp_checkpointBRight= Vector3.new(-472.65, -7.00, 95.69)
local TP_ZONE_LEFT       = Vector3.new(-466, -7, 7)
local TP_ZONE_RIGHT      = Vector3.new(-466, -7, 114)
local TP_DETECT_RADIUS   = 5
local TP_TIME_TO_CONFIRM = 2

local function tpMoveTo(pos)
    local char = player.Character
    if not char then return end
    char:PivotTo(CFrame.new(pos))
    local hrpTp = char:FindFirstChild("HumanoidRootPart")
    if hrpTp then
        hrpTp.AssemblyLinearVelocity = Vector3.zero
    end
end

local function tpExecuteSequence()
    if tp_isTeleporting then return end
    tp_isTeleporting = true
    tp_hasRecovered  = false
    local targetB     = tp_ladoEsquerdo and tp_checkpointBLeft  or tp_checkpointBRight
    local targetFinal = tp_ladoEsquerdo and tp_finalPosLeft     or tp_finalPosRight
    tpMoveTo(tp_checkpointA)  task.wait(0.12)
    tpMoveTo(targetB)          task.wait(0.12)
    tpMoveTo(targetFinal)
    tp_isTeleporting = false
end

-- Detects which side the player is on
RunService.Heartbeat:Connect(function()
    if tp_ladoTravado then return end
    local char = player.Character
    local hrpTp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrpTp then return end
    local flatPos   = Vector3.new(hrpTp.Position.X, -7, hrpTp.Position.Z)
    local distLeft  = (flatPos - TP_ZONE_LEFT).Magnitude
    local distRight = (flatPos - TP_ZONE_RIGHT).Magnitude
    if distLeft < TP_DETECT_RADIUS then
        if tp_currentZone ~= "left" then tp_currentZone = "left" tp_zoneEnterTime = tick() end
        if tick() - tp_zoneEnterTime >= TP_TIME_TO_CONFIRM then
            tp_ladoEsquerdo = false
            tp_ladoTravado  = true
        end
    elseif distRight < TP_DETECT_RADIUS then
        if tp_currentZone ~= "right" then tp_currentZone = "right" tp_zoneEnterTime = tick() end
        if tick() - tp_zoneEnterTime >= TP_TIME_TO_CONFIRM then
            tp_ladoEsquerdo = true
            tp_ladoTravado  = true
        end
    else
        tp_currentZone   = nil
        tp_zoneEnterTime = 0
    end
end)

-- Detects ragdoll state and fires TP sequence
RunService.Heartbeat:Connect(function()
    if not tpAtivado then return end
    local char = player.Character
    if not char then return end
    local humTp = char:FindFirstChild("Humanoid")
    if not humTp then return end
    local st = humTp:GetState()
    local isRagdoll = (
        st == Enum.HumanoidStateType.Physics or
        st == Enum.HumanoidStateType.Ragdoll or
        st == Enum.HumanoidStateType.FallingDown
    )
    if not isRagdoll then tp_hasRecovered = true end
    if isRagdoll and tp_hasRecovered and not tp_isTeleporting then
        task.spawn(tpExecuteSequence)
    end
end)

player.CharacterAdded:Connect(function()
    tp_isTeleporting = false
    tp_hasRecovered  = true
    tp_ladoTravado   = false
end)

-- Toggles
local function createToggle(text)
    local Holder = Instance.new("Frame")
    Holder.Size = UDim2.new(1,0,0,36)
    Holder.BackgroundTransparency = 1

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.65,0,1,0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.Font = Enum.Font.GothamMedium
    Label.TextSize = 15
    Label.TextColor3 = Color3.fromRGB(230,230,230)
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Holder

    local Toggle = Instance.new("Frame")
    Toggle.Size = UDim2.new(0,44,0,20)
    Toggle.Position = UDim2.new(1,-48,0.5,-8)
    Toggle.BackgroundColor3 = Color3.fromRGB(60,60,60)
    Toggle.Parent = Holder
    Instance.new("UICorner", Toggle).CornerRadius = UDim.new(1,0)

    local Circle = Instance.new("Frame")
    Circle.Size = UDim2.new(0,16,0,16)
    Circle.Position = UDim2.new(0,2,0.5,-8)
    Circle.BackgroundColor3 = Color3.new(1,1,1)
    Circle.Parent = Toggle
    Instance.new("UICorner", Circle).CornerRadius = UDim.new(1,0)

    local enabled = false
    if toggleStates[text] ~= nil then
        enabled = toggleStates[text]
    end

    if enabled then
        Toggle.BackgroundColor3 = Color3.fromRGB(0,140,255)
        Circle.Position = UDim2.new(1,-18,0.5,-8)
    else
        Toggle.BackgroundColor3 = Color3.fromRGB(60,60,60)
        Circle.Position = UDim2.new(0,2,0.5,-8)
    end

    local function animate(state)
        if state then
            TweenService:Create(Toggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0,140,255)}):Play()
            TweenService:Create(Circle, TweenInfo.new(0.2), {Position = UDim2.new(1,-18,0.5,-8)}):Play()
        else
            TweenService:Create(Toggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60,60,60)}):Play()
            TweenService:Create(Circle, TweenInfo.new(0.2), {Position = UDim2.new(0,2,0.5,-8)}):Play()
        end
    end
    local function applyFeature(state)
        if text == "LEFT" then
            if state then openLeftPanel() else closeLeftPanel() end
        end
        if text == "RIGHT" then
            if state then openRightPanel() else closeRightPanel() end
        end
        if text == "AUTOBAT" then
            AutoBatPanel.Visible = state
            if not state then
                if autoBatRunning then
                    WaveBat_Stop()
                    autoBatRunning = false
                    AutoBatButton.Text = "Activate"
                    AutoBatButton.BackgroundColor3 = Color3.fromRGB(0,122,204)
                end
            end
        end
        if text == "AUTOBACK" then autoBackEnabled = state end
        if text == "TP" then tpAtivado = state end
        if text == "INFJUMP" then jumpActive = state end
        if text == "SPIN" then
            if state then enableSpin() else disableSpin() end
        end
        if text == "ANTIRAGDOLL" then
            AntiRagdollEnabled = state
            if state then
                pcall(function() antiStartSystem() antiRemoveRagdollInstant() end)
            else
                pcall(function() antiStopSystem() end)
            end
        end
        if text == "ESPPLAYER" then
            if state then enableESP() else disableESP() end
        end
        if text == "UNWALK" then
            if state then enableUnwalk() else disableUnwalk() end
        end
        if text == "CAMFOLLOW" then
            if state then camFollowEnabled = true startCamFollow()
            else camFollowEnabled = false stopCamFollow() end
        end
        if text == "FLOAT" then
            if state then enableGravity() else disableGravity() end
        end
    end

    Toggle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            enabled = not enabled
            animate(enabled)
            applyFeature(enabled)
            toggleStates[text] = enabled
            SaveConfigs()
        end
    end)

    if enabled then
        task.defer(function() applyFeature(true) end)
    end

    return Holder
end

local options = {"LEFT","RIGHT","AUTOBAT","AUTOBACK","TP","INFJUMP","SPIN","ANTIRAGDOLL","ESPPLAYER","CAMFOLLOW","UNWALK","FLOAT"}

for _, name in pairs(options) do
    local t = createToggle(name)
    t.Parent = Container
end

-- Improved drag para SettingsFrame
createImprovedDrag(SettingsFrame, function()
    TweenService:Create(SettingsUIScale, TweenInfo.new(0.15), {Scale = 0.97}):Play()
end, function()
    TweenService:Create(SettingsUIScale, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Scale = 1}):Play()
end)

createImprovedDrag(MainFrame, function()
    TweenService:Create(UIScale, TweenInfo.new(0.15), {Scale = 0.97}):Play()
end, function()
    TweenService:Create(UIScale, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Scale = 1}):Play()
end)

-- Bubble
local Bubble = Instance.new("Frame")
Bubble.Size = UDim2.new(0,55,0,55)
Bubble.Position = UDim2.new(0,100,0,200)
Bubble.BackgroundColor3 = Color3.fromRGB(8,20,50)
Bubble.Parent = ScreenGui
Bubble.Active = true
Bubble.ClipsDescendants = true

Instance.new("UICorner", Bubble).CornerRadius = UDim.new(1,0)

local BubbleScale = Instance.new("UIScale", Bubble)

local BubbleImage = Instance.new("ImageLabel")
BubbleImage.Size = UDim2.new(1,0,1,0)
BubbleImage.BackgroundTransparency = 1
BubbleImage.Image = "rbxassetid://82405507602856"
BubbleImage.ScaleType = Enum.ScaleType.Fit
BubbleImage.Parent = Bubble

local bubbleOpened = false
Bubble.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        bubbleOpened = not bubbleOpened
        if bubbleOpened then openGui() else closeGui() end
    end
end)

createImprovedDrag(Bubble, function()
    TweenService:Create(BubbleScale, TweenInfo.new(0.15), {Scale = 0.9}):Play()
end, function()
    TweenService:Create(BubbleScale, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Scale = 1}):Play()
end)

MainFrame.Visible = false
SettingsFrame.Visible = false
LeftPanel.Visible = false
RightPanel.Visible = false
AutoBatPanel.Visible = false

-- MOVEMENT
local MOVE_TIMEOUT = 8
local ARRIVE_THRESHOLD = 1.0

local running = false
local speedConnection = nil
local activeRoute = nil

local function startSpeed(target, speed)
    if speedConnection then pcall(function() speedConnection:Disconnect() end) speedConnection = nil end
    speedConnection = RunService.RenderStepped:Connect(function()
        local rootLocal = getHRP()
        if not rootLocal then return end
        local direction = (target - rootLocal.Position)
        if direction.Magnitude > ARRIVE_THRESHOLD then
            direction = direction.Unit
            rootLocal.AssemblyLinearVelocity = Vector3.new(direction.X * speed, rootLocal.AssemblyLinearVelocity.Y, direction.Z * speed)
        else
            rootLocal.AssemblyLinearVelocity = Vector3.zero
        end
    end)
end

local function stopSpeed()
    if speedConnection then pcall(function() speedConnection:Disconnect() end) speedConnection = nil end
end

local function zeroVelocity()
    local rootLocal = getHRP()
    if rootLocal then
        pcall(function() rootLocal.AssemblyLinearVelocity = Vector3.zero end)
    end
end

local function moveTo(target)
    local rootLocal = getHRP()
    if not rootLocal then return false end
    local startTime = tick()
    while tick() - startTime < MOVE_TIMEOUT do
        if not running then pcall(function() rootLocal.AssemblyLinearVelocity = Vector3.zero end) return false end
        local dist = (rootLocal.Position - target).Magnitude
        if dist <= ARRIVE_THRESHOLD then
            pcall(function() rootLocal.CFrame = CFrame.new(target) end)
            pcall(function() rootLocal.AssemblyLinearVelocity = Vector3.zero end)
            return true
        end
        task.wait(0.03)
    end
    return false
end

local function walkPath(route)
    if running then return end
    running = true
    
    local fastSpeed = sliderValues.autoSpeed
    local slowSpeed = sliderValues.stealSpeed
    
    startSpeed(route.pos1, fastSpeed)
    if not moveTo(route.pos1) then stopSpeed() zeroVelocity() running = false return end
    
    startSpeed(route.pos2, fastSpeed)
    if not moveTo(route.pos2) then stopSpeed() zeroVelocity() running = false return end
    
    if not autoBackEnabled then
        stopSpeed()
        zeroVelocity()
        running = false
        activeRoute = nil
        return
    end
    
    task.wait(0.6)
    
    startSpeed(route.after_pos1, slowSpeed)
    if not moveTo(route.after_pos1) then stopSpeed() zeroVelocity() running = false return end
    
    startSpeed(route.after_pos2, slowSpeed)
    if not moveTo(route.after_pos2) then stopSpeed() zeroVelocity() running = false return end
    
    stopSpeed()
    zeroVelocity()
    running = false
    activeRoute = nil
end

local function stopCurrentWalk()
    activeRoute = nil
    running = false
    stopSpeed()
    zeroVelocity()
end

RightActivate.MouseButton1Click:Connect(function()
    if activeRoute == "route1" then
        stopCurrentWalk()
        RightActivate.Text = "Activate"
        LeftActivate.Text = "Activate"
        return
    end
    activeRoute = "route1"
    RightActivate.Text = "Stop"
    LeftActivate.Text = "Activate"
    spawn(function()
        while activeRoute == "route1" do
            if not running then walkPath(route1) end
            task.wait(0.25)
        end
        RightActivate.Text = "Activate"
        LeftActivate.Text = "Activate"
        stopCurrentWalk()
    end)
end)

LeftActivate.MouseButton1Click:Connect(function()
    if activeRoute == "route2" then
        stopCurrentWalk()
        LeftActivate.Text = "Activate"
        RightActivate.Text = "Activate"
        return
    end
    activeRoute = "route2"
    LeftActivate.Text = "Stop"
    RightActivate.Text = "Activate"
    spawn(function()
        while activeRoute == "route2" do
            if not running then walkPath(route2) end
            task.wait(0.25)
        end
        LeftActivate.Text = "Activate"
        RightActivate.Text = "Activate"
        stopCurrentWalk()
    end)
end)

local settingsOpened2 = false
GearButton.MouseButton1Click:Connect(function()
    TweenService:Create(GearButton, TweenInfo.new(0.5, Enum.EasingStyle.Linear), {Rotation = GearButton.Rotation + 360}):Play()
    settingsOpened2 = not settingsOpened2
    if settingsOpened2 then
        SettingsFrame.Visible = true
        TweenService:Create(SettingsUIScale, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
        TweenService:Create(SettingsFrame, TweenInfo.new(0.25), {BackgroundTransparency = 0.08}):Play()
    else
        local t = TweenService:Create(SettingsUIScale, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Scale = 0})
        t:Play()
        t.Completed:Wait()
        SettingsFrame.Visible = false
    end
end)

print("✓ Script completo com SLIDERS LIGADOS!")
print("✓ AUTO SPEED (30-60) - controla velocidade")
print("✓ STEAL SPEED (25-29.5) - velocidade real")
print("✓ INF JUMP (50-65.8) - altura do pulo + INFJUMP TOGGLE")
print("✓ GRAVITY (0-50) - 0=normal, 50=2x gravidade")
print("✓ SPIN (0-50) - velocidade de rotação com BodyAngularVelocity")
