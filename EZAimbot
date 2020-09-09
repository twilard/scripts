--[[
    EzAimbot apart of EzLibrary
    Developed by ToddDev
    Skid to your hearts content. That's what it's meant for!
]]--

--// Container

local EzAimbot = {}

--// Internal

local MainLoop = nil
local Camera = workspace.CurrentCamera
local Viewport = Camera.ViewportSize
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character
local Head = Character.Head
local Mouse = LocalPlayer:GetMouse()
local FOV = nil
local FriendlyFire = false
local RunService = game:GetService("RunService")
local InputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local FOVColor = Color3.fromRGB(0,0,0)
local MousePosition = function()
    return Vector2.new(Mouse.X,Mouse.Y)
end
local NotObstructing = function(destination, ignore) -- Rewrote by Stefanuk12
    local Origin = workspace.CurrentCamera.CFrame.p
    local CheckRay = Ray.new(Origin, destination- Origin)
    local Hit = workspace:FindPartOnRayWithIgnoreList(CheckRay, ignore)
    return Hit == nil
end
local ClosestPlayer = function() -- Most of this stuff was ripped right out of my upcoming hub
    local MousePos = MousePosition()
    local Radius = FOV.Radius
    local Closest = math.huge
    local Target = nil
    local function HandleTeam(player)
        local Team = LocalPlayer.Team
        local Result = true
        if player.Team == Team and FriendlyFire then
            Result = true
        elseif player.Team == Team and FriendlyFire == false then
            Result = false
        else
            Result = true
        end
        return Result
    end
    for k,v in pairs(Players:GetPlayers()) do
        pcall(function()
            if HandleTeam(v) then
                if v.Character:FindFirstChild("Head") and v ~= LocalPlayer then
                    local Point,OnScreen = Camera:WorldToViewportPoint(v.Character.Head.Position)
                    local Char = game.Players.LocalPlayer.Character
                    if OnScreen and NotObstructing(v.Character.Head.Position,{Char,v.Character}) then
                        local Distance = (Vector2.new(Point.X,Point.Y) - MousePosition()).magnitude
                        if Distance < math.min(Radius,Closest) then
                            Closest = Distance
                            Target = v
                        end
                    end
                end
            end
        end)
    end
    return Target
end
local RefreshInternals = function()
    Camera = workspace.CurrentCamera
    LocalPlayer = Players.LocalPlayer
    Character = LocalPlayer.Character
    Head = Character.Head
end
local HandleFOVColor = function(color)
    if type(color) == "string" then
        local function RainbowMath(R) -- Not mine, I don't really know what it does. But it works. The place I found it from: https://www.youtube.com/watch?v=C7V7-Sb1Qzk . I don't know if thats the original source however.
            return math.acos(math.cos(R*math.pi))/math.pi
        end
        local R = 0.002
        FOV.Color = Color3.fromHSV(R,R,0)
        local RainbowLoop = RunService.RenderStepped:Connect(function()
            FOV.Color = Color3.fromHSV(RainbowMath(R),1,1)
            R = R + 0.002
        end)
        coroutine.resume(coroutine.create(function()
            while wait() do
                if FOV == nil then RainbowLoop:Disconnect() break end
                if FOVColor ~= color then RainbowLoop:Disconnect() break end
            end
        end))
    end
end
local Key = ""
local Smooth = false

--// Main functions

EzAimbot.Disable = function()
    if MainLoop then
        MainLoop:Disconnect()
        MainLoop = nil
    end
    if FOV then
        FOV:Remove()
        FOV = nil
    end
    RefreshInternals()
end

EzAimbot.Enable = function(showfov,fovconfig,key,friendlyfire,smooth)
    assert(typeof(showfov)=="boolean","EzAimbot.Enable | Expected Boolean as argument #1")
    assert(typeof(fovconfig)=="table","EzAimbot.Enable | Expected Table as argument #2")
    assert(fovconfig["Size"],"EzAimbot.Enable | Expected Size in argument #2")
    assert(fovconfig["Sides"],"EzAimbot.Enable | Expected Sides in argument #2")
    assert(fovconfig["Color"],"EzAimbot.Enable | Expected Color in argument #2")
    assert(type(fovconfig["Size"])=="number","EzAimbot.Enable | Expected Size in argument #2")
    assert(type(fovconfig["Sides"])=="number","EzAimbot.Enable | Expected Sides in argument #2")
    assert(typeof(fovconfig["Color"])=="Color3"or fovconfig["Color"]=="Rainbow","EzAimbot.Enable | Expected Color or Rainbow in argument #2")
    assert(type(key)=="string","EzAimbot.Enable | Expected String as argument #3")
    assert(type(friendlyfire)=="boolean","EzAimbot.Enable | Expected Boolean as argument #4")
    if smooth == nil then Smooth = false end -- To be compatible with previous code
    if smooth ~= nil and type(smooth) ~= "boolean" then error("EzAimbot.Enable | Expected Boolean as argument #5") return end -- Too complicated for assert
    local Size = fovconfig["Size"]
    local Sides = fovconfig["Sides"]
    local Color = fovconfig["Color"]
    Key = key
    Smooth = smooth
    FriendlyFire = friendlyfire
    FOVColor = Color
    if showfov then
        FOV = Drawing.new("Circle")
        local FOV = FOV
        FOV.NumSides = Sides
        FOV.Position = MousePosition()
        FOV.Thickness = 2
        FOV.Radius = Size
        FOV.Color = Color ~= "Rainbow" and Color or Color3.fromRGB(0,0,0)
        HandleFOVColor(Color)
        FOV.Visible = true
    end
    MainLoop = RunService.RenderStepped:Connect(function()
        if FOV then
            FOV.Position = MousePosition()
        end
        if InputService:IsKeyDown(Enum.KeyCode[Key]) then
            local ClosestPlayer = ClosestPlayer()
            if ClosestPlayer then
                if Smooth then
                    local C = {}
                    local Info = TweenInfo.new(0.05,Enum.EasingStyle.Linear,Enum.EasingDirection.Out)
                    C.CFrame = CFrame.new(Camera.CFrame.p,ClosestPlayer.Character.Head.CFrame.p)
                    local Tween = TweenService:Create(Camera,Info,C)
                    Tween:Play()
                else
                    Camera.CFrame = CFrame.new(Camera.CFrame.p,ClosestPlayer.Character.Head.CFrame.p)
                end
            end
            RefreshInternals()
        end
    end)
end

EzAimbot.UpdateConfig = function(showfov,fovconfig,key,friendlyfire,smooth)
    assert(typeof(showfov)=="boolean","EzAimbot.UpdateConfig | Expected Boolean as argument #1")
    assert(typeof(fovconfig)=="table","EzAimbot.UpdateConfig | Expected Table as argument #2")
    assert(fovconfig["Size"],"EzAimbot.UpdateConfig | Expected Size in argument #2")
    assert(fovconfig["Sides"],"EzAimbot.UpdateConfig | Expected Sides in argument #2")
    assert(fovconfig["Color"],"EzAimbot.UpdateConfig | Expected Color in argument #2")
    assert(type(fovconfig["Size"])=="number","EzAimbot.UpdateConfig | Expected Size in argument #2")
    assert(type(fovconfig["Sides"])=="number","EzAimbot.UpdateConfig | Expected Sides in argument #2")
    assert(typeof(fovconfig["Color"])=="Color3"or fovconfig["Color"]=="Rainbow","EzAimbot.UpdateConfig | Expected Color or Rainbow in argument #2")
    assert(type(key)=="string","EzAimbot.UpdateConfig | Expected String as argument #3")
    assert(type(friendlyfire)=="boolean","EzAimbot.UpdateConfig | Expected Boolean as argument #4")
    if smooth == nil then Smooth = false end -- To be compatible with previous code
    if smooth ~= nil and type(smooth) ~= "boolean" then error("EzAimbot.UpdateConfig | Expected Boolean as argument #5") return end -- Too complicated for assert
    assert(FOV,"EzAimbot isn't enabled!")
    local Size = fovconfig["Size"]
    local Sides = fovconfig["Sides"]
    local Color = fovconfig["Color"]
    FOV.NumSides = Sides
    FOV.Radius = Size
    FOV.Visible = showfov
    FOV.Color = Color ~= "Rainbow" and Color or Color3.fromRGB(0,0,0)
    Key = key
    Smooth = smooth
    HandleFOVColor(Color)
    FriendlyFire = friendlyfire
    FOVColor = Color
end

--// Return

return EzAimbot
