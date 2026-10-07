--[[
    ═══════════════════════════════════════════════════════
    🎯 UNIVERSAL SHOT ASSIST  •  PURPLE EDITION
    - Detecta quando você atira
    - Se tem alguém no FOV, o tiro vai nele
    - Universal (funciona em qualquer FPS do Roblox)
    - Sem hook de metatable (mais seguro)
    - UI móvel/minimizável
    ═══════════════════════════════════════════════════════
]]

-- ==================== SERVIÇOS ====================
local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local Workspace        = game:GetService("Workspace")

local LP     = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- ==================== TEMA ====================
local T = {
    Bg=Color3.fromRGB(18,12,28), Panel=Color3.fromRGB(28,18,45),
    Btn=Color3.fromRGB(45,25,70), BtnHov=Color3.fromRGB(70,40,110),
    Acc=Color3.fromRGB(170,90,255), AccB=Color3.fromRGB(210,140,255),
    Text=Color3.fromRGB(235,220,255), Green=Color3.fromRGB(140,255,180),
    Red=Color3.fromRGB(255,90,120), Yellow=Color3.fromRGB(255,220,90),
}

-- ==================== ESTADO ====================
local State = {
    Enabled       = false,
    FOV           = 150,
    Smooth        = 0.15,
    MaxDist       = 500,
    TeamCheck     = true,
    WallCheck     = false,
    AimPart       = "Head",
    ShowFOV       = true,
    AutoFire      = false,   -- se true, atira sozinho quando tem alvo
    FireDelay     = 0.15,
    SnapOnClick   = true,    -- move a câmera 1 frame ao clicar (método safe)
    HoldKey       = Enum.KeyCode.LeftAlt, -- segura essa tecla pra ativar
}

local Keybinds = {
    ToggleUI    = Enum.KeyCode.RightShift,
    ToggleAim   = Enum.KeyCode.C,
    Panic       = Enum.KeyCode.F,
}

-- ==================== GUI ====================
local SG=Instance.new("ScreenGui")
SG.Name="UniversalShotAssist"; SG.ResetOnSpawn=false; SG.IgnoreGuiInset=true
SG.ZIndexBehavior=Enum.ZIndexBehavior.Sibling
SG.Parent=LP:WaitForChild("PlayerGui")

local OpenBtn=Instance.new("TextButton")
OpenBtn.Size=UDim2.new(0,55,0,55); OpenBtn.Position=UDim2.new(0,20,0,20)
OpenBtn.BackgroundColor3=T.Panel; OpenBtn.Text="🎯"; OpenBtn.TextSize=26
OpenBtn.TextColor3=T.AccB; OpenBtn.BorderSizePixel=0; OpenBtn.ZIndex=100
OpenBtn.Parent=SG
Instance.new("UICorner",OpenBtn).CornerRadius=UDim.new(1,0)
local OS=Instance.new("UIStroke",OpenBtn); OS.Color=T.Acc; OS.Thickness=2

local Frame=Instance.new("Frame")
Frame.Size=UDim2.new(0,440,0,500)
Frame.Position=UDim2.new(0.5,0,0,20)
Frame.AnchorPoint=Vector2.new(0.5,0)
Frame.BackgroundColor3=T.Bg; Frame.BorderSizePixel=0; Frame.Visible=false
Frame.ZIndex=100; Frame.Parent=SG
Instance.new("UICorner",Frame).CornerRadius=UDim.new(0,12)
local FS=Instance.new("UIStroke",Frame); FS.Color=T.Acc; FS.Thickness=2

-- Título
local TB=Instance.new("Frame")
TB.Size=UDim2.new(1,0,0,42); TB.BackgroundColor3=T.Panel; TB.BorderSizePixel=0
TB.ZIndex=101; TB.Parent=Frame
Instance.new("UICorner",TB).CornerRadius=UDim.new(0,12)
local Mask=Instance.new("Frame")
Mask.Size=UDim2.new(1,0,0,12); Mask.Position=UDim2.new(0,0,1,-12)
Mask.BackgroundColor3=T.Panel; Mask.BorderSizePixel=0; Mask.ZIndex=102; Mask.Parent=TB
local TL=Instance.new("TextLabel")
TL.Size=UDim2.new(1,-110,1,0); TL.Position=UDim2.new(0,15,0,0); TL.BackgroundTransparency=1
TL.Text="🎯 SHOT ASSIST"; TL.TextColor3=T.AccB; TL.TextSize=17
TL.Font=Enum.Font.GothamBold; TL.TextXAlignment=Enum.TextXAlignment.Left
TL.ZIndex=103; TL.Parent=TB

local MinBtn=Instance.new("TextButton")
MinBtn.Size=UDim2.new(0,28,0,28); MinBtn.Position=UDim2.new(1,-70,0,7)
MinBtn.BackgroundColor3=T.Yellow; MinBtn.Text="—"; MinBtn.TextColor3=Color3.new(0,0,0)
MinBtn.TextSize=18; MinBtn.Font=Enum.Font.GothamBold; MinBtn.BorderSizePixel=0
MinBtn.ZIndex=103; MinBtn.Parent=TB
Instance.new("UICorner",MinBtn).CornerRadius=UDim.new(0,6)

local CB=Instance.new("TextButton")
CB.Size=UDim2.new(0,28,0,28); CB.Position=UDim2.new(1,-36,0,7)
CB.BackgroundColor3=T.Red; CB.Text="✕"; CB.TextColor3=Color3.new(1,1,1)
CB.TextSize=16; CB.Font=Enum.Font.GothamBold; CB.BorderSizePixel=0
CB.ZIndex=103; CB.Parent=TB
Instance.new("UICorner",CB).CornerRadius=UDim.new(0,6)

local StatusBar=Instance.new("Frame")
StatusBar.Size=UDim2.new(1,-20,0,26); StatusBar.Position=UDim2.new(0,10,0,50)
StatusBar.BackgroundColor3=T.Panel; StatusBar.BorderSizePixel=0
StatusBar.ZIndex=102; StatusBar.Parent=Frame
Instance.new("UICorner",StatusBar).CornerRadius=UDim.new(0,6)
local StatusLabel=Instance.new("TextLabel")
StatusLabel.Size=UDim2.new(1,-16,1,0); StatusLabel.Position=UDim2.new(0,8,0,0)
StatusLabel.BackgroundTransparency=1
StatusLabel.Text="🎯 Pronto"
StatusLabel.TextColor3=T.AccB; StatusLabel.TextSize=12
StatusLabel.Font=Enum.Font.GothamMedium
StatusLabel.TextXAlignment=Enum.TextXAlignment.Left
StatusLabel.ZIndex=103; StatusLabel.Parent=StatusBar

local Sc=Instance.new("ScrollingFrame")
Sc.Size=UDim2.new(1,-20,1,-90); Sc.Position=UDim2.new(0,10,0,82)
Sc.BackgroundTransparency=1; Sc.BorderSizePixel=0; Sc.ScrollBarThickness=4
Sc.ScrollBarImageColor3=T.Acc; Sc.CanvasSize=UDim2.new(0,0,0,0)
Sc.ZIndex=101; Sc.Parent=Frame
local LL=Instance.new("UIListLayout",Sc)
LL.Padding=UDim.new(0,6); LL.SortOrder=Enum.SortOrder.LayoutOrder

-- ==================== HELPERS UI ====================
local function MakeBtn(txt,cor,fn)
    local b=Instance.new("TextButton")
    b.Size=UDim2.new(1,-10,0,34); b.BackgroundColor3=cor or T.Btn
    b.Text="  "..txt; b.TextColor3=T.Text; b.TextSize=14
    b.Font=Enum.Font.GothamMedium; b.TextXAlignment=Enum.TextXAlignment.Left
    b.BorderSizePixel=0; b.ZIndex=102; b.Parent=Sc
    Instance.new("UICorner",b).CornerRadius=UDim.new(0,7)
    local s=Instance.new("UIStroke",b); s.Color=T.Acc; s.Thickness=1; s.Transparency=0.6
    b.MouseEnter:Connect(function()
        TweenService:Create(b,TweenInfo.new(0.15),{BackgroundColor3=T.BtnHov}):Play()
    end)
    b.MouseLeave:Connect(function()
        TweenService:Create(b,TweenInfo.new(0.15),{BackgroundColor3=cor or T.Btn}):Play()
    end)
    b.MouseButton1Click:Connect(function() fn(b) end)
    return b
end

local function MakeToggle(txt,key,callback)
    local b=MakeBtn("🔴 "..txt,nil,function(btn)
        State[key]=not State[key]
        local on=State[key]
        btn.Text=(on and "  🟢 " or "  🔴 ")..txt
        btn.TextColor3=on and T.Green or T.Text
        if callback then callback(on) end
    end)
    return b
end

local function Section(txt)
    local l=Instance.new("TextLabel")
    l.Size=UDim2.new(1,-10,0,26); l.BackgroundTransparency=1
    l.Text="  ── "..txt.." ──"; l.TextColor3=T.Acc; l.TextSize=13
    l.Font=Enum.Font.GothamBold; l.TextXAlignment=Enum.TextXAlignment.Left
    l.ZIndex=102; l.Parent=Sc
end

local function SetStatus(t) StatusLabel.Text = t end

-- ==================== FOV CIRCLE ====================
local FovCircle = Instance.new("Frame")
FovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
FovCircle.Size = UDim2.new(0, State.FOV * 2, 0, State.FOV * 2)
FovCircle.BackgroundTransparency = 1
FovCircle.BorderSizePixel = 0
FovCircle.ZIndex = 5
FovCircle.Visible = false
FovCircle.Parent = SG
Instance.new("UICorner", FovCircle).CornerRadius = UDim.new(1, 0)
local FovStroke = Instance.new("UIStroke", FovCircle)
FovStroke.Color = T.Acc
FovStroke.Thickness = 2
FovStroke.Transparency = 0.3

RunService.RenderStepped:Connect(function()
    FovCircle.Visible = State.ShowFOV and State.Enabled
    FovCircle.Size = UDim2.new(0, State.FOV * 2, 0, State.FOV * 2)
    FovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
    FovStroke.Color = currentTarget and T.Green or T.Acc
end)

-- ==================== LÓGICA DO ALVO ====================
local currentTarget = nil
local aiming = false

-- Verifica se é um alvo válido
local function IsValidTarget(plr)
    if plr == LP then return false end
    if State.TeamCheck and plr.Team == LP.Team then return false end
    local c = plr.Character
    if not c then return false end
    local hum = c:FindFirstChildOfClass("Humanoid")
    local hrp = c:FindFirstChild("HumanoidRootPart")
    if not hum or not hrp or hum.Health <= 0 then return false end
    return true
end

-- Verifica se está no FOV
local function InFOV(worldPos)
    local sp, on = Camera:WorldToViewportPoint(worldPos)
    if not on then return false, math.huge end
    local ctr = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local d = (Vector2.new(sp.X, sp.Y) - ctr).Magnitude
    return d <= State.FOV, d
end

-- Wall check
local function IsVisible(part)
    if not State.WallCheck then return true end
    local origin = Camera.CFrame.Position
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LP.Character, Camera}
    local result = Workspace:Raycast(origin, part.Position - origin, params)
    if result and result.Instance then
        return result.Instance:IsDescendantOf(part.Parent)
    end
    return true
end

-- Acha o melhor alvo dentro do FOV
local function FindTarget()
    local best, bestD = nil, math.huge
    for _, plr in pairs(Players:GetPlayers()) do
        if IsValidTarget(plr) then
            local c = plr.Character
            local part = c and c:FindFirstChild(State.AimPart)
            if part then
                local dist3D = (part.Position - Camera.CFrame.Position).Magnitude
                if dist3D <= State.MaxDist then
                    local inFov, fovD = InFOV(part.Position)
                    if inFov and fovD < bestD and IsVisible(part) then
                        bestD = fovD
                        best = part
                    end
                end
            end
        end
    end
    return best
end

-- Atualiza alvo continuamente
RunService.RenderStepped:Connect(function()
    if not State.Enabled then
        currentTarget = nil
        SetStatus("🔴 Desligado")
        return
    end

    currentTarget = FindTarget()

    if currentTarget then
        SetStatus("🎯 Alvo: "..currentTarget.Parent.Name)
    else
        SetStatus("⚪ Sem alvo no FOV")
    end
end)

-- ==================== SHOT ASSIST (o core) ====================
-- Quando você atira (clica esquerdo), se tem alvo no FOV,
-- a câmera dá um snap rápido pro alvo por 2 frames,
-- depois volta. Isso faz o tiro sair no alvo.

local snapActive = false
local snapFrame = 0
local savedCF = nil

UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end

    -- Segurar tecla = ativar aim
    if input.KeyCode == State.HoldKey then
        aiming = true
    end

    -- Clicou esquerdo = atirar
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        if State.Enabled and State.SnapOnClick and currentTarget then
            snapActive = true
            snapFrame = 0
            savedCF = Camera.CFrame
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == State.HoldKey then
        aiming = false
    end
end)

-- Loop do snap
RunService.RenderStepped:Connect(function()
    if not snapActive then return end
    if not currentTarget then
        if savedCF then Camera.CFrame = savedCF end
        snapActive = false
        return
    end

    if snapFrame < 2 then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, currentTarget.Position)
        snapFrame = snapFrame + 1
    else
        if savedCF then Camera.CFrame = savedCF end
        snapActive = false
        savedCF = nil
    end
end)

-- ==================== AUTO FIRE (opcional) ====================
local lastFire = 0
RunService.RenderStepped:Connect(function()
    if not State.AutoFire or not State.Enabled then return end
    if not currentTarget then return end
    if os.clock() - lastFire < State.FireDelay then return end
    lastFire = os.clock()

    pcall(function()
        local vim = game:GetService("VirtualInputManager")
        vim:SendMouseButtonEvent(0, 0, 0, true, game, 1)
        task.wait(0.02)
        vim:SendMouseButtonEvent(0, 0, 0, false, game, 1)
    end)
end)

-- ==================== AIMGUIDO (opcional — segurando tecla) ====================
RunService:BindToRenderStep("ShotAssistAim", Enum.RenderPriority.Camera.Value + 1, function()
    if not State.Enabled then return end
    if not aiming then return end
    if not currentTarget then return end

    local tcf = CFrame.new(Camera.CFrame.Position, currentTarget.Position)
    if State.Smooth <= 0 then
        Camera.CFrame = tcf
    else
        local alpha = math.clamp(1 - State.Smooth, 0.05, 1)
        Camera.CFrame = Camera.CFrame:Lerp(tcf, alpha)
    end
end)

-- ==================== SEÇÕES ====================
Section("🎯 SHOT ASSIST")
MakeToggle("Ativar Assist", "Enabled")
MakeToggle("Snap no Clique (recomendado)", "SnapOnClick")
MakeToggle("Auto Fire (atira sozinho)", "AutoFire")
MakeToggle("Mostrar FOV", "ShowFOV")
MakeToggle("Team Check", "TeamCheck")
MakeToggle("Wall Check", "WallCheck")

MakeBtn("🎯 FOV: 150", T.Panel, function(b)
    State.FOV = State.FOV + 50
    if State.FOV > 500 then State.FOV = 50 end
    b.Text = "  🎯 FOV: " .. State.FOV
end)

MakeBtn("💧 Smooth: 0.15", T.Panel, function(b)
    State.Smooth = State.Smooth + 0.05
    if State.Smooth > 1 then State.Smooth = 0.05 end
    b.Text = "  💧 Smooth: " .. string.format("%.2f", State.Smooth)
end)

MakeBtn("📏 Dist: 500", T.Panel, function(b)
    State.MaxDist = State.MaxDist + 250
    if State.MaxDist > 2000 then State.MaxDist = 250 end
    b.Text = "  📏 Dist: " .. State.MaxDist
end)

MakeBtn("🎯 Mira: Head", T.Panel, function(b)
    local list = {"Head","HumanoidRootPart","UpperTorso"}
    local i = table.find(list, State.AimPart) or 1
    i = i % #list + 1
    State.AimPart = list[i]
    b.Text = "  🎯 Mira: " .. State.AimPart
end)

MakeBtn("⏱️ AutoFire Delay: 0.15", T.Panel, function(b)
    State.FireDelay = State.FireDelay + 0.05
    if State.FireDelay > 0.5 then State.FireDelay = 0.05 end
    b.Text = "  ⏱️ AutoFire Delay: " .. string.format("%.2f", State.FireDelay)
end)

Section("⌨️ KEYBINDS")
MakeBtn("RightShift = Abrir UI", T.Panel, function() end)
MakeBtn("LeftAlt (segurar) = Aim assist", T.Panel, function() end)
MakeBtn("C = Toggle Assist", T.Panel, function() end)
MakeBtn("F = Panic", T.Panel, function() end)

Section("⚙️ SISTEMA")
MakeBtn("🔄 Resetar Tudo", T.Red, function()
    State.Enabled = false
    State.AutoFire = false
    currentTarget = nil
    snapActive = false
    print("🔄 Resetado")
end)

-- ==================== KEYBINDS ====================
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Keybinds.ToggleUI then
        Frame.Visible = not Frame.Visible
    elseif input.KeyCode == Keybinds.ToggleAim then
        State.Enabled = not State.Enabled
    elseif input.KeyCode == Keybinds.Panic then
        State.Enabled = false
        State.AutoFire = false
        currentTarget = nil
        print("[PANIC] Desligado")
    end
end)

-- ==================== DRAG ====================
local function MakeDrag(frame, handle)
    local dr, ds, sp
    handle.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dr = true; ds = i.Position; sp = frame.Position
        end
    end)
    handle.InputChanged:Connect(function(i)
        if dr and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - ds
            frame.Position = UDim2.new(sp.X.Scale, sp.X.Offset+d.X, sp.Y.Scale, sp.Y.Offset+d.Y)
        end
    end)
    handle.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dr = false
        end
    end)
end
MakeDrag(Frame, TB)

local bdr, bs, bsp, bmv
OpenBtn.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        bdr = true; bmv = false; bs = i.Position; bsp = OpenBtn.Position
    end
end)
OpenBtn.InputChanged:Connect(function(i)
    if bdr and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - bs
        if math.abs(d.X) > 5 or math.abs(d.Y) > 5 then bmv = true end
        if bmv then
            OpenBtn.Position = UDim2.new(bsp.X.Scale, bsp.X.Offset+d.X, bsp.Y.Scale, bsp.Y.Offset+d.Y)
        end
    end
end)
OpenBtn.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        if not bmv then Frame.Visible = not Frame.Visible end
        bdr = false; bmv = false
    end
end)

MinBtn.MouseButton1Click:Connect(function() Frame.Visible = not Frame.Visible end)
CB.MouseButton1Click:Connect(function() Frame.Visible = false end)

LL:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Sc.CanvasSize = UDim2.new(0, 0, 0, LL.AbsoluteContentSize.Y + 10)
end)

-- ==================== NOTIFY ====================
local N = Instance.new("TextLabel", SG)
N.Size = UDim2.new(0, 480, 0, 40)
N.Position = UDim2.new(0.5, 0, 0, 90)
N.AnchorPoint = Vector2.new(0.5, 0)
N.BackgroundColor3 = T.Panel
N.TextColor3 = T.AccB
N.Text = "🎯 SHOT ASSIST carregado! Atira no alvo dentro do FOV"
N.TextSize = 15
N.Font = Enum.Font.GothamBold
N.BorderSizePixel = 0
N.ZIndex = 200
Instance.new("UICorner", N).CornerRadius = UDim.new(0, 8)
task.wait(3.5)
TweenService:Create(N, TweenInfo.new(0.5), {BackgroundTransparency=1, TextTransparency=1}):Play()
task.wait(0.6)
N:Destroy()

print("🎯 SHOT ASSIST carregado!")
print("💡 Ativa, aponta pra alguém dentro do FOV e ATIRA — o tiro vai nele")
