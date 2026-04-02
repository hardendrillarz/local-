--// HARDEN V1 FINAL

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LP = Players.LocalPlayer

-- CONFIG
local HEAD_SIZE = 3
local enabled = true

-- limitar 2–5
if HEAD_SIZE > 5 then HEAD_SIZE = 5 end
if HEAD_SIZE < 2 then HEAD_SIZE = 2 end

-- GUI
local PlayerGui = LP:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui", PlayerGui)
screenGui.Name = "HARDEN_V1"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0,160,0,90)
frame.Position = UDim2.new(0,50,0,50)
frame.BackgroundColor3 = Color3.fromRGB(10,10,20)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0.3,0)
title.BackgroundTransparency = 1
title.Text = "HARDEN V1"
title.TextColor3 = Color3.fromRGB(0,170,255)
title.TextScaled = true

local label = Instance.new("TextLabel", frame)
label.Position = UDim2.new(0,0,0.3,0)
label.Size = UDim2.new(1,0,0.3,0)
label.BackgroundTransparency = 1
label.Text = "Head: "..HEAD_SIZE
label.TextColor3 = Color3.fromRGB(255,255,255)
label.TextScaled = true

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1,0,0.4,0)
button.Position = UDim2.new(0,0,0.6,0)
button.Text = "ON"
button.BackgroundColor3 = Color3.fromRGB(0,170,0)
button.TextScaled = true

button.MouseButton1Click:Connect(function()
	enabled = not enabled
	button.Text = enabled and "ON" or "OFF"
	button.BackgroundColor3 = enabled and Color3.fromRGB(0,170,0) or Color3.fromRGB(170,0,0)
end)

-- RAINBOW
local function getRainbow()
	return Color3.fromHSV((tick()*0.2)%1,1,1)
end

-- HITBOX
local function applyHitbox(player)
	if player == LP then return end
	
	local function onCharacter(char)
		local head = char:FindFirstChild("Head")
		local torso = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
		
		if enabled then
			if head then
				head.Size = Vector3.new(HEAD_SIZE,HEAD_SIZE,HEAD_SIZE)
				head.Transparency = 0.3
				head.Material = Enum.Material.Neon
				head.CanCollide = false
			end
			
			if torso then
				torso.Size = Vector3.new(4,4,2)
				torso.Transparency = 0.5
				torso.Material = Enum.Material.Neon
				torso.CanCollide = false
			end
		end
	end
	
	if player.Character then
		onCharacter(player.Character)
	end
	
	player.CharacterAdded:Connect(onCharacter)
end

-- ESP + LINEAS
local esp = {}

local function createESP(player)
	if player == LP then return end
	
	local line = Drawing.new("Line")
	line.Thickness = 2
	
	local text = Drawing.new("Text")
	text.Size = 16
	text.Center = true
	text.Outline = true
	
	esp[player] = {line = line, text = text}
end

for _,p in pairs(Players:GetPlayers()) do
	createESP(p)
end

Players.PlayerAdded:Connect(createESP)

RunService.RenderStepped:Connect(function()
	for player,data in pairs(esp) do
		local char = player.Character
		
		if char and char:FindFirstChild("Head") and LP.Character and enabled then
			local headPos, visible = Camera:WorldToViewportPoint(char.Head.Position)
			local myPos = Camera:WorldToViewportPoint(LP.Character.Head.Position)
			
			if visible then
				local color = getRainbow()
				
				data.text.Position = Vector2.new(headPos.X, headPos.Y)
				data.text.Text = player.Name
				data.text.Color = color
				data.text.Visible = true
				
				data.line.From = Vector2.new(myPos.X, myPos.Y)
				data.line.To = Vector2.new(headPos.X, headPos.Y)
				data.line.Color = color
				data.line.Visible = true
			else
				data.text.Visible = false
				data.line.Visible = false
			end
		else
			if data then
				data.text.Visible = false
				data.line.Visible = false
			end
		end
	end
end)

-- SETUP
for _,p in pairs(Players:GetPlayers()) do
	applyHitbox(p)
end

Players.PlayerAdded:Connect(applyHitbox)