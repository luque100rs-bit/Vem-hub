local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local LocalPlayer = Players.LocalPlayer
    local speedConn
    local baseSpeed = 27
    local active = false
 
    local function GetCharacter()
        local Char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local HRP = Char:WaitForChild("HumanoidRootPart")
        local Hum = Char:FindFirstChildOfClass("Humanoid")
        return Char, HRP, Hum
    end
 
    local function getMovementInput()
        local Char, HRP, Hum = GetCharacter()
        if not Char or not HRP or not Hum then return Vector3.new(0,0,0) end
        local moveVector = Hum.MoveDirection
        if moveVector.Magnitude > 0.1 then
            return Vector3.new(moveVector.X, 0, moveVector.Z).Unit
        end
        return Vector3.new(0,0,0)
    end
 
    local function startSpeedControl()
        if speedConn then return end
        speedConn = RunService.Heartbeat:Connect(function()
            local Char, HRP, Hum = GetCharacter()
            if not Char or not HRP or not Hum then return end
            local inputDirection = getMovementInput()
            if inputDirection.Magnitude > 0 then
                HRP.AssemblyLinearVelocity = Vector3.new(
                    inputDirection.X*baseSpeed,
                    HRP.AssemblyLinearVelocity.Y,
                    inputDirection.Z*baseSpeed
                )
            else
                HRP.AssemblyLinearVelocity = Vector3.new(0, HRP.AssemblyLinearVelocity.Y, 0)
            end
        end)
    end
 
    local function stopSpeedControl()
        if speedConn then speedConn:Disconnect() speedConn = nil end
        local Char, HRP = GetCharacter()
        if HRP then HRP.AssemblyLinearVelocity = Vector3.new(0, HRP.AssemblyLinearVelocity.Y, 0) end
    end
 
    speedButton.MouseButton1Click:Connect(function()
        active = not active
        if active then
            startSpeedControl()
            speedButton.BackgroundColor3 = Color3.fromRGB(0,170,0)
        else
            stopSpeedControl()
            speedButton.BackgroundColor3 = Color3.fromRGB(220,20,60)
        end
    end)

end
