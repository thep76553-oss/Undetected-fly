-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

-- GUI Setup
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
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

-- Variables
local boxOn = false
local moveSpeed = 50
local boxParts = {} -- will hold 6 walls

-- Toggle
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
			
			-- Floor
			local floor = Instance.new("Part")
			floor.Size = Vector3.new(boxSize.X, thickness, boxSize.Z)
			floor.Position = pos - Vector3.new(0, boxSize.Y/2, 0)
			floor.Anchored = true
			floor.CanCollide = true
			floor.Transparency = 0.5
			floor.Parent = workspace
			table.insert(boxParts, floor)
			
			-- Ceiling
			local ceiling = Instance.new("Part")
			ceiling.Size = Vector3.new(boxSize.X, thickness, boxSize.Z)
			ceiling.Position = pos + Vector3.new(0, boxSize.Y/2, 0)
			ceiling.Anchored = true
			ceiling.CanCollide = true
			ceiling.Transparency = 0.5
			ceiling.Parent = workspace
			table.insert(boxParts, ceiling)
			
			-- Walls
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
				wall.CanCollide = true
				wall.Transparency = 0.5
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

-- Movement Loop
RunService.RenderStepped:Connect(function(delta)
	if boxOn and #boxParts > 0 then
		-- Move box with player
		local pos = hrp.Position
		local boxSize = Vector3.new(8,6,8)
		local thickness = 0.5
		
		-- Floor and ceiling
		boxParts[1].Position = pos - Vector3.new(0, boxSize.Y/2, 0)
		boxParts[2].Position = pos + Vector3.new(0, boxSize.Y/2, 0)
		
		-- Walls
		boxParts[3].Position = pos + Vector3.new(boxSize.X/2, 0, 0)
		boxParts[4].Position = pos - Vector3.new(boxSize.X/2, 0, 0)
		boxParts[5].Position = pos + Vector3.new(0, 0, boxSize.Z/2)
		boxParts[6].Position = pos - Vector3.new(0, 0, boxSize.Z/2)
		
		-- Push player in look direction
		local cam = workspace.CurrentCamera
		local lookVector = cam.CFrame.LookVector
		hrp.CFrame = hrp.CFrame + lookVector * (moveSpeed * delta)
	end
end)
