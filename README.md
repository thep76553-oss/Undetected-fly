local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local boxOn = false
local moveSpeed = 50
local boxParts = {} -- will hold 6 walls

-- Function to reinitialize after respawn
local function reinitializeCharacter()
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    
    -- Create GUI setup again (to prevent issues on respawn)
    local screenGui = player:FindFirstChild("PlayerGui"):FindFirstChild("MoveBoxGUI")
    if screenGui then screenGui:Destroy() end -- Remove old GUI if any

    screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
    screenGui.Name = "MoveBoxGUI"

    local frame = Instance.new("Frame", screenGui)
    frame.Size = UDim2.new(0, 200, 0, 100)
    frame.Position = UDim2.new(0.5, -100, 0.5, -50)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    frame.Active = true
    frame.Draggable = true

    local toggleBtn = Instance.new("TextButton", frame)
    toggleBtn.Size = UDim2.new(0, 180, 0, 30)
    toggleBtn.Position = UDim2.new(0, 10, 0, 10)
    toggleBtn.Text = "Toggle Box: OFF"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)

    local speedBox = Instance.new("TextBox", frame)
    speedBox.Size = UDim2.new(0, 180, 0, 30)
    speedBox.Position = UDim2.new(0, 10, 0, 50)
    speedBox.PlaceholderText = "Speed (default 50)"
    speedBox.Text = ""
    speedBox.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    speedBox.ClearTextOnFocus = false

    -- Toggle button action
    toggleBtn.MouseButton1Click:Connect(function()
        boxOn = not boxOn
        toggleBtn.Text = "Toggle Box: " .. (boxOn and "ON" or "OFF")
        
        if boxOn then
            -- Find part under player
            local rayOrigin = hrp.Position
            local rayDir = Vector3.new(0, -10, 0)
            local rayResult = workspace:Raycast(rayOrigin, rayDir)
            if rayResult then
                local hitPart = rayResult.Instance
                
                -- Create 6 hollow walls using the hitPart's position
                local boxSize = Vector3.new(8, 6, 8)
                local pos = hitPart.Position + Vector3.new(0, 3, 0)
                local thickness = 0.5
                
                -- Floor (invisible but still here)
                local floor = Instance.new("Part")
                floor.Size = Vector3.new(boxSize.X, thickness, boxSize.Z)
                floor.Position = pos - Vector3.new(0, boxSize.Y/2, 0)
                floor.Anchored = true
                floor.CanCollide = false
                floor.Transparency = 1 -- Completely invisible
                floor.Parent = workspace
                table.insert(boxParts, floor)
                
                -- Ceiling (invisible)
                local ceiling = Instance.new("Part")
                ceiling.Size = Vector3.new(boxSize.X, thickness, boxSize.Z)
                ceiling.Position = pos + Vector3.new(0, boxSize.Y/2, 0)
                ceiling.Anchored = true
                ceiling.CanCollide = false
                ceiling.Transparency = 1 -- Completely invisible
                ceiling.Parent = workspace
                table.insert(boxParts, ceiling)
                
                -- Walls (invisible)
                local wallPositions = {
                    Vector3.new(pos.X + boxSize.X/2, pos.Y, pos.Z),
                    Vector3.new(pos.X - boxSize.X/2, pos.Y, pos.Z),
                    Vector3.new(pos.X, pos.Y, pos.Z + boxSize.Z/2),
                    Vector3.new(pos.X, pos.Y, pos.Z - boxSize.Z/2),
                }
                local wallSizes = {
                    Vector3.new(thickness, boxSize.Y, boxSize.Z),
                    Vector3.new(thickness, boxSize.Y, boxSize.Z),
                    Vector3.new(boxSize.X, boxSize.Y, thickness),
                    Vector3.new(boxSize.X, boxSize.Y, thickness),
                }
                
                for i = 1, 4 do
                    local wall = Instance.new("Part")
                    wall.Size = wallSizes[i]
                    wall.Position = wallPositions[i]
                    wall.Anchored = true
                    wall.CanCollide = false
                    wall.Transparency = 1 -- Completely invisible
                    wall.Parent = workspace
                    table.insert(boxParts, wall)
                end
            end
        else
            -- Remove box
            for _, p in pairs(boxParts) do
                if p then p:Destroy() end
            end
            boxParts = {}
        end
    end)

    -- Update speed
    speedBox.FocusLost:Connect(function()
        local num = tonumber(speedBox.Text)
        if num then moveSpeed = num end
    end)

    -- Movement loop (after respawn)
    RunService.RenderStepped:Connect(function(delta)
        if boxOn and #boxParts > 0 then
            local pos = hrp.Position
            local boxSize = Vector3.new(8, 6, 8)
            local thickness = 0.5
            
            -- Floor and ceiling (still invisible)
            boxParts[1].Position = pos - Vector3.new(0, boxSize.Y/2, 0)
            boxParts[2].Position = pos + Vector3.new(0, boxSize.Y/2, 0)
            
            -- Walls (still invisible)
            boxParts[3].Position = pos + Vector3.new(boxSize.X/2, 0, 0)
            boxParts[4].Position = pos - Vector3.new(boxSize.X/2, 0, 0)
            boxParts[5].Position = pos + Vector3.new(0, 0, boxSize.Z/2)
            boxParts[6].Position = pos - Vector3.new(0, 0, boxSize.Z/2)
            
            -- Smooth movement
            local cam = workspace.CurrentCamera
            local lookVector = cam.CFrame.LookVector
            local targetPosition = hrp.Position + lookVector * (moveSpeed * delta)
            
            -- Smooth movement: interpolate position using Lerp
            hrp.CFrame = CFrame.new(hrp.Position:Lerp(targetPosition, 0.2)) -- Lerp for smooth movement
            
            local velocity = (targetPosition - hrp.Position).Unit * moveSpeed
            hrp.Velocity = velocity
        end
    end)
end

-- Reinitialize character when they respawn
player.CharacterAdded:Connect(function()
    reinitializeCharacter()
end)

-- Initial setup when the script first runs
reinitializeCharacter()
