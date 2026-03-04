--// CONFIG
local UpdateDelay = 0.4
local MaxPoints = 25

--// SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

--// DATA
local FPSData = table.create(MaxPoints, 0)
local CurrentFPS = 0
local Enabled = true
local DataIndex = 0

--// GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Container = Instance.new("Frame")
Container.Size = UDim2.new(0, 300, 0, 160)
Container.Position = UDim2.new(0, 30, 0, 30)
Container.BackgroundColor3 = Color3.fromRGB(18,18,22)
Container.BorderSizePixel = 0
Container.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,25)
Title.BackgroundTransparency = 1
Title.Text = "ULTRA LIGHT FPS"
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 15
Title.TextColor3 = Color3.fromRGB(200,200,220)
Title.Parent = Container

local Toggle = Instance.new("TextButton")
Toggle.Size = UDim2.new(0,80,0,22)
Toggle.Position = UDim2.new(1,-85,0,2)
Toggle.Text = "ON"
Toggle.BackgroundColor3 = Color3.fromRGB(0,170,0)
Toggle.TextColor3 = Color3.new(1,1,1)
Toggle.Font = Enum.Font.SourceSansBold
Toggle.TextSize = 13
Toggle.BorderSizePixel = 0
Toggle.Parent = Container

local GraphFrame = Instance.new("Frame")
GraphFrame.Position = UDim2.new(0,10,0,30)
GraphFrame.Size = UDim2.new(1,-20,1,-40)
GraphFrame.BackgroundColor3 = Color3.fromRGB(25,25,30)
GraphFrame.BorderSizePixel = 0
GraphFrame.Parent = Container

--// PRECRIAR PONTOS E LINHAS (REUTILIZADOS)
local Points = {}
local Lines = {}

for i = 1, MaxPoints do
	local point = Instance.new("Frame")
	point.Size = UDim2.new(0,5,0,5)
	point.BackgroundColor3 = Color3.fromRGB(0,200,255)
	point.BorderSizePixel = 0
	point.Parent = GraphFrame
	Points[i] = point

	if i > 1 then
		local line = Instance.new("Frame")
		line.BackgroundColor3 = Color3.fromRGB(0,200,255)
		line.BorderSizePixel = 0
		line.AnchorPoint = Vector2.new(0.5,0.5)
		line.Parent = GraphFrame
		Lines[i] = line
	end
end

--// FPS
RunService.RenderStepped:Connect(function(dt)
	if dt > 0 then
		CurrentFPS = math.floor(1/dt)
	end
end)

--// TOGGLE
Toggle.MouseButton1Click:Connect(function()
	Enabled = not Enabled
	if Enabled then
		Toggle.Text = "ON"
		Toggle.BackgroundColor3 = Color3.fromRGB(0,170,0)
		GraphFrame.Visible = true
	else
		Toggle.Text = "OFF"
		Toggle.BackgroundColor3 = Color3.fromRGB(170,0,0)
		GraphFrame.Visible = false
	end
end)

--// UPDATE LOOP
task.spawn(function()
	while ScreenGui.Parent do
		if Enabled then
			-- Atualiza índice circular
			DataIndex = (DataIndex % MaxPoints) + 1
			FPSData[DataIndex] = CurrentFPS

			local width = GraphFrame.AbsoluteSize.X
			local height = GraphFrame.AbsoluteSize.Y

			local maxValue = 1
			for i = 1, MaxPoints do
				maxValue = math.max(maxValue, FPSData[i])
			end

			local lastPos

			for i = 1, MaxPoints do
				local index = ((DataIndex + i - 1) % MaxPoints) + 1
				local value = FPSData[index]

				local x = (i / MaxPoints) * width
				local y = height - ((value / maxValue) * height)

				local point = Points[i]
				point.Position = UDim2.new(0, x-2, 0, y-2)

				if lastPos and Lines[i] then
					local line = Lines[i]

					local dist = (Vector2.new(x,y) - lastPos).Magnitude
					line.Size = UDim2.new(0, dist, 0, 2)
					line.Position = UDim2.new(0,(x+lastPos.X)/2,0,(y+lastPos.Y)/2)
					line.Rotation = math.deg(math.atan2(y-lastPos.Y,x-lastPos.X))
				end

				lastPos = Vector2.new(x,y)
			end
		end

		task.wait(UpdateDelay)
	end
end)
