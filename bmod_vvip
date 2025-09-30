_G.TeamCheck            = true
_G.ESPEnabled           = true
_G.BoxEnabled           = true
_G.NameEnabled          = true
_G.ShowHPBar            = true
_G.ShowLine             = true
_G.AimbotEnabled        = true
_G.DisableCameraOverride= false
_G.FOVRadius            = 120
_G.ESPColor             = Color3.fromRGB(0,255,180)
_G.RainbowESP           = false
_G.RainbowSpeed         = 0.6
_G.BoxThickness         = 1
_G.ESPAlpha             = 1
_G.AimMode              = "HARD"
_G.AimSmoothing         = 0.65
_G.AimOffset            = 0

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local ESP = {}

local FOV = Drawing.new("Circle")
FOV.NumSides = 100
FOV.Thickness = 1
FOV.Radius = _G.FOVRadius
FOV.Filled = false
FOV.Color = Color3.fromRGB(255,30,80)
FOV.Visible = true
FOV.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

local function clamp(v,a,b) return math.max(a, math.min(b, v)) end

local function safeRaycast(origin, dir, blacklist)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Blacklist
    params.FilterDescendantsInstances = blacklist or {}
    local ok, res = pcall(function() return workspace:Raycast(origin, dir, params) end)
    if not ok then return nil end
    return res
end

local function IsVisible(part)
    if not part or not part.Parent then return false end
    local origin = Camera.CFrame.Position
    local dir = (part.Position - origin)
    if dir.Magnitude <= 0.1 then return true end
    local blacklist = {}
    if LocalPlayer and LocalPlayer.Character then table.insert(blacklist, LocalPlayer.Character) end
    local res = safeRaycast(origin, dir, blacklist)
    if not res then return true end
    return res.Instance and res.Instance:IsDescendantOf(part.Parent)
end

local function hsvColor(h) return Color3.fromHSV((h%1), 0.9, 0.95) end

local COLOR_PRESETS = {
    {name="NeonCyan", color=Color3.fromRGB(0,255,180)},
    {name="NeonPink", color=Color3.fromRGB(255,90,190)},
    {name="NeonGreen", color=Color3.fromRGB(0,255,100)},
    {name="NeonYellow", color=Color3.fromRGB(255,220,80)},
    {name="NeonMagenta", color=Color3.fromRGB(200,0,200)},
    {name="NeonBlue", color=Color3.fromRGB(80,140,255)}
}

local function CreateESP(plr)
    if ESP[plr] then return end
    ESP[plr] = {
        Box = Drawing.new("Square"),
        Name = Drawing.new("Text"),
        HPBG = Drawing.new("Square"),
        HP = Drawing.new("Square"),
        Line = Drawing.new("Line")
    }
    local e = ESP[plr]
    e.Box.Thickness = _G.BoxThickness
    e.Box.Color = _G.ESPColor
    e.Box.Filled = false
    e.Box.Transparency = _G.ESPAlpha

    e.Name.Size = 14
    e.Name.Center = true
    e.Name.Outline = true
    e.Name.Color = e.Box.Color
    e.Name.Transparency = _G.ESPAlpha

    e.HPBG.Filled = true
    e.HPBG.Color = Color3.fromRGB(0,0,0)
    e.HPBG.Transparency = _G.ESPAlpha

    e.HP.Filled = true
    e.HP.Color = Color3.fromRGB(0,255,0)
    e.HP.Transparency = _G.ESPAlpha

    e.Line.Thickness = 1
    e.Line.Color = Color3.fromRGB(255,255,255)
    e.Line.Transparency = _G.ESPAlpha
end

local function RemoveESP(plr)
    if not ESP[plr] then return end
    for _,v in pairs(ESP[plr]) do
        pcall(function() if v and v.Remove then v:Remove() end end)
    end
    ESP[plr] = nil
end

Players.PlayerRemoving:Connect(function(plr) RemoveESP(plr) end)

local function GetBox(plr)
    if not plr or not plr.Character then return end
    local char = plr.Character
    local head = char:FindFirstChild("Head")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not (head and hrp and humanoid) then return end
    if humanoid.Health <= 0 then return end
    local screenPos, vis = Camera:WorldToViewportPoint(hrp.Position)
    if not vis then return end
    local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
    local scale = clamp(2000 / math.max(distance, 0.1), 3, 650)
    local height = 6 * scale
    local width = 4 * scale
    local topLeft = Vector2.new(screenPos.X - width/2, screenPos.Y - height/2)
    return topLeft, Vector2.new(width, height), humanoid, head, screenPos
end

local function IsValidTarget(plr)
    if not plr then return false end
    if plr == LocalPlayer then return false end
    if _G.TeamCheck and plr.Team == LocalPlayer.Team then return false end
    if not plr.Character then return false end
    return true
end

local function GetClosestHeadInFOV()
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local bestHead = nil
    local bestDist = _G.FOVRadius
    for _, plr in ipairs(Players:GetPlayers()) do
        if IsValidTarget(plr) then
            local _, _, humanoid, head = GetBox(plr)
            if head and humanoid and humanoid.Health > 0 then
                local headScr, vis = Camera:WorldToViewportPoint(head.Position)
                if vis then
                    local dist = (Vector2.new(headScr.X, headScr.Y) - center).Magnitude
                    if dist <= _G.FOVRadius and IsVisible(head) then
                        if dist < bestDist then
                            bestDist = dist
                            bestHead = head
                        end
                    end
                end
            end
        end
    end
    return bestHead
end

RunService.RenderStepped:Connect(function()
    FOV.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    FOV.Radius = _G.FOVRadius

    local anyVisible = false
    local now = tick()

    for _, plr in ipairs(Players:GetPlayers()) do
        if IsValidTarget(plr) and _G.ESPEnabled then
            local boxPos, size, humanoid, head, screenPos = GetBox(plr)
            if boxPos and size then
                if not ESP[plr] then CreateESP(plr) end
                local e = ESP[plr]
                local color = _G.RainbowESP and hsvColor(now * _G.RainbowSpeed) or _G.ESPColor
                e.Box.Color = color
                e.Box.Thickness = _G.BoxThickness
                e.Box.Transparency = _G.ESPAlpha
                e.Name.Color = color
                e.Name.Transparency = _G.ESPAlpha
                e.HP.Transparency = _G.ESPAlpha
                e.HPBG.Transparency = _G.ESPAlpha
                e.Line.Transparency = _G.ESPAlpha

                if _G.BoxEnabled then
                    e.Box.Visible = true
                    e.Box.Position = boxPos
                    e.Box.Size = size
                else
                    e.Box.Visible = false
                end

                if _G.NameEnabled then
                    e.Name.Visible = true
                    e.Name.Position = Vector2.new(boxPos.X + size.X/2, boxPos.Y - 15)
                    e.Name.Text = plr.Name
                else
                    e.Name.Visible = false
                end

                if _G.ShowHPBar then
                    local hpPerc = clamp(humanoid.Health / math.max(humanoid.MaxHealth,1), 0, 1)
                    e.HPBG.Visible = true
                    e.HPBG.Size = Vector2.new(4, size.Y)
                    e.HPBG.Position = Vector2.new(boxPos.X - 6, boxPos.Y)
                    e.HP.Visible = true
                    e.HP.Size = Vector2.new(4, size.Y * hpPerc)
                    e.HP.Position = Vector2.new(boxPos.X - 6, boxPos.Y + size.Y * (1 - hpPerc))
                    if hpPerc <= 0.3 then e.HP.Color = Color3.fromRGB(255,0,0)
                    elseif hpPerc <= 0.6 then e.HP.Color = Color3.fromRGB(255,255,0)
                    else e.HP.Color = Color3.fromRGB(0,255,0) end
                else
                    e.HP.Visible = false
                    e.HPBG.Visible = false
                end

                if _G.ShowLine and screenPos then
                    e.Line.Visible = true
                    e.Line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                    e.Line.To = Vector2.new(screenPos.X, screenPos.Y)
                else
                    e.Line.Visible = false
                end

                local headScr, headVis = Camera:WorldToViewportPoint(head.Position)
                if headVis and (Vector2.new(headScr.X, headScr.Y) - FOV.Position).Magnitude <= _G.FOVRadius and IsVisible(head) then
                    anyVisible = true
                end
            else
                if ESP[plr] then
                    for _,v in pairs(ESP[plr]) do
                        if v and v.Visible ~= nil then v.Visible = false end
                    end
                end
            end
        else
            RemoveESP(plr)
        end
    end

    FOV.Color = anyVisible and Color3.fromRGB(0,255,100) or Color3.fromRGB(255,30,80)

    if _G.AimbotEnabled then
        local targetHead = GetClosestHeadInFOV()
        if targetHead and targetHead.Position then
            local aimPos = targetHead.Position + Vector3.new(0, _G.AimOffset, 0)
            if not _G.DisableCameraOverride then
                if _G.AimMode == "HARD" then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, aimPos)
                else
                    local desired = CFrame.new(Camera.CFrame.Position, aimPos)
                    local s = clamp(_G.AimSmoothing, 0.01, 0.99)
                    Camera.CFrame = Camera.CFrame:Lerp(desired, s)
                end
            end
        end
    end
end)

local function createHUD()
    local parent
    local ok = pcall(function() parent = game:GetService("CoreGui") end)
    if not ok or not parent then parent = LocalPlayer:WaitForChild("PlayerGui") end
    if parent:FindFirstChild("BMOD_HUD") then return end

    local gui = Instance.new("ScreenGui")
    gui.Name = "BMOD_HUD"
    gui.ResetOnSpawn = false
    gui.Parent = parent

    local frame = Instance.new("Frame", gui)
    frame.Name = "HUD"
    frame.Size = UDim2.new(0, 180, 0, 42)
    frame.Position = UDim2.new(0, 12, 0, 8)
    frame.BackgroundColor3 = Color3.fromRGB(10,10,12)
    frame.BorderSizePixel = 0
    local corner = Instance.new("UICorner", frame); corner.CornerRadius = UDim.new(0,8)
    local stroke = Instance.new("UIStroke", frame); stroke.Color = Color3.fromRGB(0,255,160); stroke.Transparency = 0.9; stroke.Thickness = 1

    local icon = Instance.new("Frame", frame)
    icon.Size = UDim2.new(0,40,1, -8); icon.Position = UDim2.new(0,6,0,4)
    icon.BackgroundColor3 = Color3.fromRGB(0,255,160); icon.BorderSizePixel = 0
    local icorner = Instance.new("UICorner", icon); icorner.CornerRadius = UDim.new(0,6)
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, -56, 1, 0); label.Position = UDim2.new(0, 52, 0, 0)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14
    label.TextColor3 = Color3.fromRGB(230,230,230)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Text = "BMOD VVIP"

    local status = Instance.new("TextLabel", frame)
    status.Size = UDim2.new(1, -56, 0, 18)
    status.Position = UDim2.new(0, 52, 0, 18)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.SourceSans
    status.TextSize = 12
    status.TextColor3 = Color3.fromRGB(200,200,200)
    status.TextXAlignment = Enum.TextXAlignment.Left
    status.Text = ""

    spawn(function()
        while gui and gui.Parent do
            pcall(function()
                status.Text = "ESP: "..( _G.ESPEnabled and "ON" or "OFF").."  •  AIM: "..( _G.AimbotEnabled and (_G.AimMode=="HARD" and "HARD" or "SMOOTH") or "OFF")
                icon.BackgroundColor3 = (_G.ESPEnabled and Color3.fromRGB(0,255,160) or Color3.fromRGB(120,120,120))
            end)
            wait(0.25)
        end
    end)
end

pcall(createHUD)

local function createMenu()
    local parent
    local ok = pcall(function() parent = game:GetService("CoreGui") end)
    if not ok or not parent then parent = LocalPlayer:WaitForChild("PlayerGui") end
    if parent:FindFirstChild("BMOD_VVIP_Menu") then return end

    local gui = Instance.new("ScreenGui")
    gui.Name = "BMOD_VVIP_Menu"
    gui.ResetOnSpawn = false
    gui.Parent = parent

    local frame = Instance.new("Frame", gui)
    frame.Name = "Main"
    frame.Size = UDim2.new(0, 360, 0, 520)
    frame.Position = UDim2.new(0, 18, 0, 80)
    frame.BackgroundColor3 = Color3.fromRGB(10,10,14)
    frame.BorderSizePixel = 0
    frame.Active = true
    local corner = Instance.new("UICorner", frame); corner.CornerRadius = UDim.new(0,12)
    local stroke = Instance.new("UIStroke", frame); stroke.Color = Color3.fromRGB(0,255,160); stroke.Transparency = 0.92; stroke.Thickness = 1

    local header = Instance.new("Frame", frame)
    header.Size = UDim2.new(1,0,0,52)
    header.Position = UDim2.new(0,0,0,0)
    header.BackgroundTransparency = 1

    local title = Instance.new("TextLabel", header)
    title.Size = UDim2.new(1, -20, 1, 0)
    title.Position = UDim2.new(0, 12, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "「BMOD VVIP」 • CYBERPUNK"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 16
    title.TextColor3 = Color3.fromRGB(230,230,230)
    title.TextXAlignment = Enum.TextXAlignment.Left

    local ver = Instance.new("TextLabel", header)
    ver.Size = UDim2.new(0, 60, 1, 0)
    ver.Position = UDim2.new(1, -74, 0, 0)
    ver.BackgroundTransparency = 1
    ver.Text = "v3.1"
    ver.Font = Enum.Font.Gotham
    ver.TextSize = 12
    ver.TextColor3 = Color3.fromRGB(180,180,180)
    ver.TextXAlignment = Enum.TextXAlignment.Right

    local closeBtn = Instance.new("TextButton", header)
    closeBtn.Size = UDim2.new(0, 28, 0, 24)
    closeBtn.Position = UDim2.new(1, -38, 0, 14)
    closeBtn.Text = "✕"
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.TextSize = 14
    closeBtn.TextColor3 = Color3.fromRGB(220,220,220)
    closeBtn.BackgroundColor3 = Color3.fromRGB(18,18,20)
    closeBtn.BorderSizePixel = 0

    local minBtn = Instance.new("TextButton", header)
    minBtn.Size = UDim2.new(0, 28, 0, 24)
    minBtn.Position = UDim2.new(1, -72, 0, 14)
    minBtn.Text = "–"
    minBtn.Font = Enum.Font.GothamBold
    minBtn.TextSize = 14
    minBtn.TextColor3 = Color3.fromRGB(220,220,220)
    minBtn.BackgroundColor3 = Color3.fromRGB(18,18,20)
    minBtn.BorderSizePixel = 0

    local scroll = Instance.new("ScrollingFrame", frame)
    scroll.Size = UDim2.new(1, -12, 1, -72)
    scroll.Position = UDim2.new(0,6,0,64)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.CanvasSize = UDim2.new(0,0,2,0)
    scroll.ScrollBarThickness = 6

    local list = Instance.new("UIListLayout", scroll)
    list.SortOrder = Enum.SortOrder.LayoutOrder
    list.Padding = UDim.new(0,8)

    local function ToggleRow(text, getter, setter)
        local row = Instance.new("Frame", scroll)
        row.Size = UDim2.new(1, -12, 0, 40)
        row.BackgroundTransparency = 1

        local lbl = Instance.new("TextLabel", row)
        lbl.Size = UDim2.new(0.66, 0, 1, 0)
        lbl.Position = UDim2.new(0, 12, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.Font = Enum.Font.Gotham
        lbl.TextSize = 14
        lbl.TextColor3 = Color3.fromRGB(220,220,220)
        lbl.TextXAlignment = Enum.TextXAlignment.Left

        local sw = Instance.new("Frame", row)
        sw.Size = UDim2.new(0, 58, 0, 30)
        sw.Position = UDim2.new(1, -80, 0, 5)
        sw.BackgroundColor3 = getter() and Color3.fromRGB(0,255,160) or Color3.fromRGB(40,40,46)
        sw.BorderSizePixel = 0
        local swc = Instance.new("UICorner", sw); swc.CornerRadius = UDim.new(1,0)

        local knob = Instance.new("Frame", sw)
        knob.Size = UDim2.new(0, 26, 0, 26)
        knob.Position = getter() and UDim2.new(1, -30, 0, 2) or UDim2.new(0, 2, 0, 2)
        knob.BackgroundColor3 = Color3.fromRGB(14,14,16)
        local knc = Instance.new("UICorner", knob); knc.CornerRadius = UDim.new(1,0)

        local function toggle()
            local new = not getter()
            setter(new)
            TweenService:Create(sw, TweenInfo.new(0.12), {BackgroundColor3 = new and Color3.fromRGB(0,255,160) or Color3.fromRGB(40,40,46)}):Play()
            local tpos = new and UDim2.new(1, -30, 0, 2) or UDim2.new(0, 2, 0, 2)
            TweenService:Create(knob, TweenInfo.new(0.12), {Position = tpos}):Play()
        end

        sw.InputBegan:Connect(function(inp)
            if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
                toggle()
            end
        end)
    end

    local function SliderRow(text, getter, setter, minV, maxV)
        local row = Instance.new("Frame", scroll)
        row.Size = UDim2.new(1, -12, 0, 60)
        row.BackgroundTransparency = 1

        local lbl = Instance.new("TextLabel", row)
        lbl.Size = UDim2.new(0.6, 0, 0, 18)
        lbl.Position = UDim2.new(0, 12, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.Font = Enum.Font.Gotham
        lbl.TextSize = 13
        lbl.TextColor3 = Color3.fromRGB(220,220,220)
        lbl.TextXAlignment = Enum.TextXAlignment.Left

        local val = Instance.new("TextLabel", row)
        val.Size = UDim2.new(0.3, 0, 0, 18)
        val.Position = UDim2.new(1, -90, 0, 0)
        val.BackgroundTransparency = 1
        val.Text = tostring(getter())
        val.Font = Enum.Font.SourceSans
        val.TextSize = 13
        val.TextColor3 = Color3.fromRGB(200,200,200)
        val.TextXAlignment = Enum.TextXAlignment.Right

        local barBg = Instance.new("Frame", row)
        barBg.Size = UDim2.new(1, -24, 0, 12)
        barBg.Position = UDim2.new(0, 12, 0, 34)
        barBg.BackgroundColor3 = Color3.fromRGB(28,28,34)
        barBg.BorderSizePixel = 0
        local br = Instance.new("UICorner", barBg); br.CornerRadius = UDim.new(0,6)

        local fill = Instance.new("Frame", barBg)
        fill.Size = UDim2.new((getter() - minV)/(maxV - minV), 0, 1, 0)
        fill.BackgroundColor3 = Color3.fromRGB(0,255,160)
        fill.BorderSizePixel = 0
        local fr = Instance.new("UICorner", fill); fr.CornerRadius = UDim.new(0,6)

        local handle = Instance.new("Frame", barBg)
        handle.Size = UDim2.new(0, 14, 0, 14)
        handle.Position = UDim2.new((getter() - minV)/(maxV - minV), -7, 0, -1)
        handle.BackgroundColor3 = Color3.fromRGB(240,240,240)
        local hr = Instance.new("UICorner", handle); hr.CornerRadius = UDim.new(1,0)

        local dragging = false
        local function updateFromX(x)
            local rel = clamp((x - barBg.AbsolutePosition.X) / barBg.AbsoluteSize.X, 0, 1)
            local v = math.floor(minV + rel * (maxV - minV))
            setter(v)
            fill.Size = UDim2.new(rel, 0, 1, 0)
            handle.Position = UDim2.new(rel, -7, 0, -1)
            val.Text = tostring(v)
        end

        UserInputService.InputBegan:Connect(function(input, gp)
            if gp then return end
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                local pos = input.Position or UserInputService:GetMouseLocation()
                if pos.X >= barBg.AbsolutePosition.X and pos.X <= barBg.AbsolutePosition.X + barBg.AbsoluteSize.X
                and pos.Y >= barBg.AbsolutePosition.Y and pos.Y <= barBg.AbsolutePosition.Y + barBg.AbsoluteSize.Y then
                    dragging = true
                    updateFromX(pos.X)
                end
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if not dragging then return end
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                local pos = input.Position or UserInputService:GetMouseLocation()
                updateFromX(pos.X)
            end
        end)
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = false
            end
        end)
    end

    ToggleRow("ESP", function() return _G.ESPEnabled end, function(v) _G.ESPEnabled = v end)
    ToggleRow("Boxes", function() return _G.BoxEnabled end, function(v) _G.BoxEnabled = v end)
    ToggleRow("Names", function() return _G.NameEnabled end, function(v) _G.NameEnabled = v end)
    ToggleRow("HP Bar", function() return _G.ShowHPBar end, function(v) _G.ShowHPBar = v end)
    ToggleRow("Line", function() return _G.ShowLine end, function(v) _G.ShowLine = v end)
    ToggleRow("Aimbot", function() return _G.AimbotEnabled end, function(v) _G.AimbotEnabled = v end)
    ToggleRow("Team Check", function() return _G.TeamCheck end, function(v) _G.TeamCheck = v end)
    ToggleRow("Disable Cam Override", function() return _G.DisableCameraOverride end, function(v) _G.DisableCameraOverride = v end)
    ToggleRow("Rainbow ESP", function() return _G.RainbowESP end, function(v) _G.RainbowESP = v end)

    local presetRow = Instance.new("Frame", scroll)
    presetRow.Size = UDim2.new(1, -12, 0, 44); presetRow.BackgroundTransparency = 1
    local pl = Instance.new("TextLabel", presetRow)
    pl.Size = UDim2.new(0.5,0,0,18); pl.Position = UDim2.new(0,12,0,0); pl.BackgroundTransparency = 1
    pl.Text = "Color Presets"; pl.Font = Enum.Font.Gotham; pl.TextColor3 = Color3.fromRGB(220,220,220)

    local xpos = 12
    for i,p in ipairs(COLOR_PRESETS) do
        local b = Instance.new("TextButton", presetRow)
        b.Size = UDim2.new(0, 36, 0, 36)
        b.Position = UDim2.new(0, xpos, 0, 6)
        b.BackgroundColor3 = p.color
        b.BorderSizePixel = 0
        b.AutoButtonColor = false
        local bc = Instance.new("UICorner", b); bc.CornerRadius = UDim.new(0,6)
        b.MouseButton1Click:Connect(function()
            _G.ESPColor = p.color
            _G.RainbowESP = false
        end)
        xpos = xpos + 42
    end

    SliderRow("FOV Radius", function() return _G.FOVRadius end, function(v) _G.FOVRadius = v end, 20, 1000)
    SliderRow("Aim Offset (studs)", function() return _G.AimOffset end, function(v) _G.AimOffset = v end, -5, 10)
    SliderRow("Aim Smoothing (%)", function() return math.floor(_G.AimSmoothing * 100) end, function(v) _G.AimSmoothing = clamp(v/100, 0.01, 0.99) end, 1, 99)
    SliderRow("Rainbow Speed (%)", function() return math.floor(_G.RainbowSpeed * 100) end, function(v) _G.RainbowSpeed = clamp(v/100, 1, 400)/100 end, 1, 400)
    SliderRow("Box Thickness", function() return _G.BoxThickness end, function(v) _G.BoxThickness = clamp(v,1,8) end, 1, 8)
    SliderRow("ESP Alpha (%)", function() return math.floor(_G.ESPAlpha * 100) end, function(v) _G.ESPAlpha = clamp(v/100, 0, 1) end, 0, 100)

    local footer = Instance.new("Frame", scroll); footer.Size = UDim2.new(1,-12,0,44); footer.BackgroundTransparency = 1
    local reset = Instance.new("TextButton", footer)
    reset.Size = UDim2.new(0.46, -6, 1, 0); reset.Position = UDim2.new(0,12,0,0)
    reset.Text = "Reset"; reset.Font = Enum.Font.GothamBold; reset.TextColor3 = Color3.fromRGB(240,240,240)
    reset.BackgroundColor3 = Color3.fromRGB(20,20,24); reset.BorderSizePixel = 0
    reset.MouseButton1Click:Connect(function()
        _G.TeamCheck = true; _G.ESPEnabled=true; _G.BoxEnabled=true; _G.NameEnabled=true
        _G.ShowHPBar=true; _G.ShowLine=true; _G.AimbotEnabled=true; _G.DisableCameraOverride=false
        _G.FOVRadius=120; _G.ESPColor=Color3.fromRGB(0,255,180); _G.RainbowESP=false; _G.RainbowSpeed=0.6
        _G.BoxThickness=1; _G.ESPAlpha=1; _G.AimMode="HARD"; _G.AimSmoothing=0.65; _G.AimOffset=0
    end)

    local hide = Instance.new("TextButton", footer)
    hide.Size = UDim2.new(0.46, -6, 1, 0); hide.Position = UDim2.new(1, -(12 + 160), 0, 0)
    hide.Text = "Hide"; hide.Font = Enum.Font.GothamBold; hide.TextColor3 = Color3.fromRGB(240,240,240)
    hide.BackgroundColor3 = Color3.fromRGB(20,20,24); hide.BorderSizePixel = 0
    hide.MouseButton1Click:Connect(function() frame.Visible = false end)

    local aimModeRow = Instance.new("Frame", scroll); aimModeRow.Size = UDim2.new(1,-12,0,40); aimModeRow.BackgroundTransparency = 1
    local amLabel = Instance.new("TextLabel", aimModeRow); amLabel.Size = UDim2.new(0.6,0,1,0); amLabel.Position = UDim2.new(0,12,0,0); amLabel.BackgroundTransparency=1
    amLabel.Text = "Aim Mode"; amLabel.Font = Enum.Font.Gotham; amLabel.TextColor3 = Color3.fromRGB(220,220,220)
    local hardBtn = Instance.new("TextButton", aimModeRow); hardBtn.Size = UDim2.new(0,80,0,28); hardBtn.Position = UDim2.new(1, -190, 0, 6)
    hardBtn.Text = "HARD"; hardBtn.Font = Enum.Font.GothamBold; hardBtn.TextSize = 14; hardBtn.BackgroundColor3 = Color3.fromRGB(40,40,48)
    local smoothBtn = Instance.new("TextButton", aimModeRow); smoothBtn.Size = UDim2.new(0,80,0,28); smoothBtn.Position = UDim2.new(1, -100, 0, 6)
    smoothBtn.Text = "SMOOTH"; smoothBtn.Font = Enum.Font.GothamBold; smoothBtn.TextSize = 14; smoothBtn.BackgroundColor3 = Color3.fromRGB(40,40,48)
    local function updateAimButtons()
        if _G.AimMode == "HARD" then
            hardBtn.BackgroundColor3 = Color3.fromRGB(0,255,160)
            smoothBtn.BackgroundColor3 = Color3.fromRGB(40,40,48)
        else
            smoothBtn.BackgroundColor3 = Color3.fromRGB(0,255,160)
            hardBtn.BackgroundColor3 = Color3.fromRGB(40,40,48)
        end
    end
    hardBtn.MouseButton1Click:Connect(function() _G.AimMode="HARD"; updateAimButtons() end)
    smoothBtn.MouseButton1Click:Connect(function() _G.AimMode="SMOOTH"; updateAimButtons() end)
    updateAimButtons()

    closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)
    minBtn.MouseButton1Click:Connect(function()
        if scroll.Visible then
            scroll.Visible = false; frame.Size = UDim2.new(0,360,0,52); minBtn.Text = "+"
        else
            scroll.Visible = true; frame.Size = UDim2.new(0,360,0,520); minBtn.Text = "–"
        end
    end)

    local dragging = false
    local dragStart = nil
    local startPos = nil
    header.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = inp.Position
            startPos = frame.Position
            inp.Changed:Connect(function()
                if inp.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    UserInputService.InputChanged:Connect(function(inp)
        if not dragging then return end
        if inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch then
            local delta = inp.Position - dragStart
            frame.Position = UDim2.new(0, startPos.X.Offset + delta.X, 0, startPos.Y.Offset + delta.Y)
        end
    end)

    return gui
end

pcall(createMenu)

do
    local tapTimes = {}
    local function onTouch(input)
        if input.UserInputType == Enum.UserInputType.Touch then
            local now = tick()
            table.insert(tapTimes, now)
            while #tapTimes > 3 do table.remove(tapTimes,1) end
            if #tapTimes >= 2 and tapTimes[#tapTimes] - tapTimes[#tapTimes-1] <= 0.35 then
                local menu = nil
                pcall(function() menu = (game:GetService("CoreGui").BMOD_VVIP_Menu or (LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.PlayerGui:FindFirstChild("BMOD_VVIP_Menu"))) end)
                if menu and menu.Parent then
                    local mf = menu:FindFirstChild("Main")
                    if mf then mf.Visible = not mf.Visible end
                end
            end
        end
    end
    UserInputService.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.UserInputType == Enum.UserInputType.Touch then onTouch(input) end
    end)
end
