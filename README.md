local p=game:GetService("Players")
local r=game:GetService("RunService")
local u=game:GetService("UserInputService")
local ts=game:GetService("TweenService")
local pl=p.LocalPlayer
local pg=pl:WaitForChild("PlayerGui")

local sg=Instance.new("ScreenGui")
sg.Parent=pg
sg.ResetOnSpawn=false

local mf=Instance.new("Frame")
mf.Size=UDim2.new(0,200,0,200)
mf.Position=UDim2.new(1,-210,0.2,0)
mf.BackgroundColor3=Color3.fromRGB(20,10,35)
mf.BorderSizePixel=0
mf.Parent=sg

local c=Instance.new("UICorner")
c.CornerRadius=UDim.new(0,12)
c.Parent=mf

local s=Instance.new("UIStroke")
s.Thickness=3
s.Color=Color3.fromRGB(150,0,255)
s.Parent=mf

local tb=Instance.new("Frame")
tb.Size=UDim2.new(1,0,0,45)
tb.Position=UDim2.new(0,0,0,0)
tb.BackgroundColor3=Color3.fromRGB(35,20,60)
tb.BorderSizePixel=0
tb.Parent=mf

local tbc=Instance.new("UICorner")
tbc.CornerRadius=UDim.new(0,12)
tbc.Parent=tb

local t=Instance.new("TextLabel")
t.Size=UDim2.new(1,0,0.6,0)
t.Position=UDim2.new(0,0,0,0)
t.BackgroundTransparency=1
t.Text="NovaHub Float"
t.TextColor3=Color3.fromRGB(255,255,255)
t.TextSize=16
t.Font=Enum.Font.GothamBlack
t.Parent=tb

local st=Instance.new("TextLabel")
st.Size=UDim2.new(1,0,0.4,0)
st.Position=UDim2.new(0,0,0.6,0)
st.BackgroundTransparency=1
st.Text="TT: novahubbest"
st.TextColor3=Color3.fromRGB(150,0,255)
st.TextSize=12
st.Font=Enum.Font.GothamBold
st.Parent=tb

local savedPos=nil
local flyEnabled=false
local bodyVelocity,bodyGyro
local noclipEnabled=false

local function notify(msg)
    local notifGui=Instance.new("ScreenGui")
    notifGui.Parent=pg
    notifGui.ResetOnSpawn=false
    local notifFrame=Instance.new("Frame")
    notifFrame.Size=UDim2.new(0,200,0,50)
    notifFrame.Position=UDim2.new(1,-220,1,-60)
    notifFrame.BackgroundColor3=Color3.fromRGB(35,20,60)
    notifFrame.BackgroundTransparency=1
    notifFrame.Parent=notifGui
    
    local notifCorner=Instance.new("UICorner")
    notifCorner.CornerRadius=UDim.new(0,8)
    notifCorner.Parent=notifFrame
    
    local notifStroke=Instance.new("UIStroke")
    notifStroke.Thickness=2
    notifStroke.Color=Color3.fromRGB(150,0,255)
    notifStroke.Parent=notifFrame
    
    local notifText=Instance.new("TextLabel")
    notifText.Size=UDim2.new(1,0,1,0)
    notifText.BackgroundTransparency=1
    notifText.Text=msg
    notifText.TextColor3=Color3.fromRGB(255,255,255)
    notifText.TextSize=14
    notifText.Font=Enum.Font.GothamSemibold
    notifText.Parent=notifFrame
    
    ts:Create(notifFrame,TweenInfo.new(0.3),{BackgroundTransparency=0}):Play()
    ts:Create(notifFrame,TweenInfo.new(0.3),{Position=UDim2.new(1,-220,1,-120)}):Play()
    task.wait(3)
    ts:Create(notifFrame,TweenInfo.new(0.5),{BackgroundTransparency=1}):Play()
    task.wait(0.5)
    notifGui:Destroy()
end

local function toggleFly()
    local char=pl.Character
    if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    if not savedPos then
        notify("No position saved!")
        return
    end

    flyEnabled=not flyEnabled

    if flyEnabled then
        bodyVelocity=Instance.new("BodyVelocity")
        bodyVelocity.MaxForce=Vector3.new(10000,10000,10000)
        bodyVelocity.Parent=root

        bodyGyro=Instance.new("BodyGyro")
        bodyGyro.MaxTorque=Vector3.new(40000,40000,40000)
        bodyGyro.Parent=root

        char.Humanoid.PlatformStand=true
        noclipEnabled=true -- Автоматически включаем NoClip
        notify("Float enabled!")
    else
        flyEnabled=false
        noclipEnabled=false
        if bodyVelocity then bodyVelocity:Destroy() bodyVelocity=nil end
        if bodyGyro then bodyGyro:Destroy() bodyGyro=nil end
        char.Humanoid.PlatformStand=false
        notify("Float disabled!")
    end
end

local function savePosition()
    local char=pl.Character
    if char and char:FindFirstChild("HumanoidRootPart")then
        savedPos=char.HumanoidRootPart.Position
        notify("Position saved!")
    end
end

local function toggleNoClip()
    noclipEnabled=not noclipEnabled
    if noclipEnabled then
notify("NoClip enabled!")
    else
        notify("NoClip disabled!")
    end
end

r.Heartbeat:Connect(function()
    if flyEnabled and bodyVelocity and bodyGyro then
        local char=pl.Character
        if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart")
        if not root then return end

        if savedPos then
            local pos=root.Position
            local dist=(savedPos-pos).Magnitude
            local horizontalDist=Vector3.new(savedPos.X-pos.X,0,savedPos.Z-pos.Z).Magnitude
            
            if horizontalDist>10 then
                local highTarget=Vector3.new(savedPos.X,savedPos.Y+50,savedPos.Z)
                local dir=(highTarget-pos).Unit
                bodyGyro.CFrame=CFrame.lookAt(pos,pos+dir)
                bodyVelocity.Velocity=dir*50
            else
                local targetPos=Vector3.new(savedPos.X,pos.Y,savedPos.Z)
                local dir=(targetPos-pos).Unit
                bodyGyro.CFrame=CFrame.lookAt(pos,pos+dir)
                bodyVelocity.Velocity=dir*(horizontalDist*3)
                
                if horizontalDist<2 then
                    flyEnabled=false
                    noclipEnabled=false
                    bodyVelocity:Destroy()
                    bodyGyro:Destroy()
                    char.Humanoid.PlatformStand=false
                    notify("Destination reached!")
                end
            end
        end
    end
end)

r.Heartbeat:Connect(function()
    if noclipEnabled then
        local char=pl.Character
        if char then
            for _,part in pairs(char:GetDescendants())do
                if part:IsA("BasePart")then part.CanCollide=false end
            end
        end
    end
end)

local function createBtn(text,yPos)
    local b=Instance.new("TextButton")
    b.Size=UDim2.new(0.9,0,0,35)
    b.Position=UDim2.new(0.05,0,yPos,0)
    b.BackgroundColor3=Color3.fromRGB(50,25,85)
    b.Text=text
    b.TextColor3=Color3.fromRGB(255,255,255)
    b.TextSize=13
    b.Font=Enum.Font.GothamSemibold
    b.Parent=mf

    local bc=Instance.new("UICorner")
    bc.CornerRadius=UDim.new(0,8)
    bc.Parent=b

    local bs=Instance.new("UIStroke")
    bs.Thickness=2
    bs.Color=Color3.fromRGB(120,0,220)
    bs.Parent=b

    b.MouseButton1Click:Connect(function()
        if text=="Save Pos"then savePosition()
        elseif text=="Float"then toggleFly()
        elseif text=="NoClip"then toggleNoClip() end
    end)
end

createBtn("Save Pos",0.25)
createBtn("Float",0.5)
createBtn("NoClip",0.75)

local d=false
local di,ds,sp
tb.InputBegan:Connect(function(i)
    if i.UserInputType==Enum.UserInputType.MouseButton1 then
        d=true
        ds=i.Position
        sp=mf.Position
        i.Changed:Connect(function()
            if i.UserInputState==Enum.UserInputState.End then d=false end
        end)
    end
end)

tb.InputChanged:Connect(function(i)
    if i.UserInputType==Enum.UserInputType.MouseMovement then di=i end
end)

u.InputChanged:Connect(function(i)
    if i==di and d then
        local de=i.Position-ds
        mf.Position=UDim2.new(sp.X.Scale,sp.X.Offset+de.X,sp.Y.Scale,sp.Y.Offset+de.Y)
    end
end)

notify("NovaHub loaded!")
