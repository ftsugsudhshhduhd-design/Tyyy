-- MARK HUB MOBILE FINAL (DRAG TOPBAR FIX)

local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")

-- SETTINGS
local lockEnabled = false
local showMarker = false
local showFOV = true
local uiVisible = true

local FOV = 150
local markerSize = 0.6
local aimPart = "HumanoidRootPart"

local target = nil

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "MARK_HUB"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,240,0,400)
frame.Position = UDim2.new(0.3,0,0.3,0)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)

-- 🔥 TOPBAR (ใช้ลาก)
local topbar = Instance.new("Frame", frame)
topbar.Size = UDim2.new(1,0,0,40)
topbar.BackgroundColor3 = Color3.fromRGB(15,15,15)

local title = Instance.new("TextLabel", topbar)
title.Size = UDim2.new(1,0,1,0)
title.Text = "MARK HUB"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.new(1,1,1)
title.TextScaled = true

-- ✅ DRAG (มือถือ + คอม ใช้ได้จริง)
local dragging = false
local dragInput, dragStart, startPos

topbar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then
		
		dragging = true
		dragInput = input
		dragStart = input.Position
		startPos = frame.Position
	end
end)

UIS.InputEnded:Connect(function(input)
	if input == dragInput then
		dragging = false
	end
end)

UIS.InputChanged:Connect(function(input)
	if dragging and input == dragInput then
		local delta = input.Position - dragStart
		
		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

-- UI FUNCTION
local function makeBtn(text, y)
	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(1,0,0,40)
	btn.Position = UDim2.new(0,0,0,y)
	btn.Text = text
	btn.BackgroundColor3 = Color3.fromRGB(30,30,30)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.TextScaled = true
	return btn
end

local lockBtn = makeBtn("ล็อกเป้า: ปิด",50)
local espBtn = makeBtn("ESP: ปิด",90)
local fovBtn = makeBtn("FOV: เปิด",130)
local aimBtn = makeBtn("ยิง: ตัว",170)
local fovPlus = makeBtn("FOV +",210)
local fovMinus = makeBtn("FOV -",250)
local espPlus = makeBtn("ESP ใหญ่ขึ้น",290)
local fpsBtn = makeBtn("FPS BOOST",330)
local hideBtn = makeBtn("ซ่อน UI",370)

-- FOV
local circle = Instance.new("Frame", gui)
circle.BackgroundTransparency = 1
circle.BorderSizePixel = 2
circle.BorderColor3 = Color3.fromRGB(255,255,255)
circle.AnchorPoint = Vector2.new(0.5,0.5)

-- MARKER
local marker = Instance.new("Part")
marker.Shape = Enum.PartType.Ball
marker.Anchored = true
marker.CanCollide = false
marker.Material = Enum.Material.Neon
marker.Color = Color3.fromRGB(255,0,0)
marker.Parent = workspace
marker.Transparency = 1

-- FUNCTIONS
local function updateFOV()
	circle.Size = UDim2.new(0, FOV*2, 0, FOV*2)
	circle.Position = UDim2.new(0.5, 0, 0.5, 0)
end

local function getClosestTarget()
	local closest = nil
	local shortest = FOV
	
	for _,v in pairs(game.Players:GetPlayers()) do
		if v ~= player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
			
			local pos, onScreen = camera:WorldToViewportPoint(v.Character.HumanoidRootPart.Position)
			
			if onScreen then
				local dist = (Vector2.new(pos.X,pos.Y) - Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)).Magnitude
				
				if dist < shortest then
					shortest = dist
					closest = v
				end
			end
		end
	end
	
	return closest
end

-- BUTTONS
lockBtn.MouseButton1Click:Connect(function()
	lockEnabled = not lockEnabled
	lockBtn.Text = "ล็อกเป้า: " .. (lockEnabled and "เปิด" or "ปิด")
	target = lockEnabled and getClosestTarget() or nil
end)

espBtn.MouseButton1Click:Connect(function()
	showMarker = not showMarker
	espBtn.Text = "ESP: " .. (showMarker and "เปิด" or "ปิด")
end)

fovBtn.MouseButton1Click:Connect(function()
	showFOV = not showFOV
	fovBtn.Text = "FOV: " .. (showFOV and "เปิด" or "ปิด")
	circle.Visible = showFOV
end)

aimBtn.MouseButton1Click:Connect(function()
	if aimPart == "HumanoidRootPart" then
		aimPart = "Head"
		aimBtn.Text = "ยิง: หัว"
	else
		aimPart = "HumanoidRootPart"
		aimBtn.Text = "ยิง: ตัว"
	end
end)

fovPlus.MouseButton1Click:Connect(function()
	FOV = FOV + 20
end)

fovMinus.MouseButton1Click:Connect(function()
	FOV = math.max(50, FOV - 20)
end)

espPlus.MouseButton1Click:Connect(function()
	markerSize = markerSize + 0.2
end)

fpsBtn.MouseButton1Click:Connect(function()
	Lighting.GlobalShadows = false
	settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
	
	for _,v in pairs(workspace:GetDescendants()) do
		if v:IsA("ParticleEmitter") or v:IsA("Trail") then
			v.Enabled = false
		end
	end
end)

hideBtn.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

-- LOOP
RunService.RenderStepped:Connect(function()
	updateFOV()

	if target and target.Character and target.Character:FindFirstChild(aimPart) then
		local pos = target.Character[aimPart].Position

		if lockEnabled then
			camera.CFrame = camera.CFrame:Lerp(
				CFrame.new(camera.CFrame.Position, pos),
				0.35
			)
		end

		if showMarker then
			marker.Size = Vector3.new(markerSize,markerSize,markerSize)
			marker.Position = pos + Vector3.new(0,2,0)
			marker.Transparency = 0
		else
			marker.Transparency = 1
		end
	else
		marker.Transparency = 1
	end
end)
