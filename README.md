--// CONFIG
local UpdateDelay = 0.5
local MaxDataPoints = 30

--// SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

--// DATA
local FPSData = {}
local PingData = {}

local CurrentFPS = 0

--// GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PerformanceGraph"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Container = Instance.new("Frame")
Container.Size = UDim2.new(0, 350, 0, 180)
Container.Position = UDim2.new(0, 20, 0, 20)
Container.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
Container.Parent = ScreenGui

Container.BorderSizePixel = 0
Container.BackgroundTransparency = 0

local UICorner = Instance.new("UICorner", Container)
UICorner.CornerRadius = UDim.new(0, 12)

local UIStroke = Instance.new("UIStroke", Container)
UIStroke.Thickness = 1
UIStroke.Color = Color3.fromRGB(60, 60, 70)

local Title = Instance.new("TextLabel")
Title.Parent = Container
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Text = "Performance Monitor"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextColor3 = Color3.fromRGB(230,230,255)

local GraphFrame = Instance.new("Frame")
GraphFrame.Parent = Container
GraphFrame.Position = UDim2.new(0, 10, 0, 40)
GraphFrame.Size = UDim2.new(1, -20, 1, -50)
GraphFrame.BackgroundTransparency = 1

--// FPS CALC
RunService.RenderStepped:Connect(function(dt)
	if dt > 0 then
		CurrentFPS = math.floor(1 / dt)
	end
end)

--// PING
local function GetPing()
	local ping = LocalPlayer:GetNetworkPing()
	return math.round(ping * 1000) -- convert to ms
end

--// ADD DATA
local function AddData(tbl, value)
	if #tbl >= MaxDataPoints then
		table.remove(tbl, 1)
	end
	table.insert(tbl, value)
end

--// DRAW GRAPH
local function DrawGraph()
	GraphFrame:ClearAllChildren()

	local width = GraphFrame.AbsoluteSize.X
	local height = GraphFrame.AbsoluteSize.Y

	local maxValue = 0

	for _, v in ipairs(FPSData) do
		maxValue = math.max(maxValue, v)
	end
	for _, v in ipairs(PingData) do
		maxValue = math.max(maxValue, v)
	end

	if maxValue == 0 then return end

	local function DrawLine(data, color)
		local lastPos

		for i, value in ipairs(data) do
			local x = (i / MaxDataPoints) * width
			local y = height - ((value / maxValue) * height)

			local point = Instance.new("Frame")
			point.Size = UDim2.new(0, 4, 0, 4)
			point.Position = UDim2.new(0, x, 0, y)
			point.BackgroundColor3 = color
			point.BorderSizePixel = 0
			point.Parent = GraphFrame

			local corner = Instance.new("UICorner", point)
			corner.CornerRadius = UDim.new(1,0)

			if lastPos then
				local line = Instance.new("Frame")
				line.BorderSizePixel = 0
				line.BackgroundColor3 = color
				line.AnchorPoint = Vector2.new(0.5,0.5)

				local dist = (Vector2.new(x,y) - lastPos).Magnitude
				line.Size = UDim2.new(0, dist, 0, 2)

				line.Position = UDim2.new(0,(x+lastPos.X)/2,0,(y+lastPos.Y)/2)
				line.Rotation = math.deg(math.atan2(y-lastPos.Y,x-lastPos.X))
				line.Parent = GraphFrame
			end

			lastPos = Vector2.new(x,y)
		end
	end

	DrawLine(FPSData, Color3.fromRGB(0,255,140))
	DrawLine(PingData, Color3.fromRGB(255,100,100))
end

--// UPDATE LOOP
task.spawn(function()
	while ScreenGui.Parent do
		AddData(FPSData, CurrentFPS)
		AddData(PingData, GetPing())

		DrawGraph()

		task.wait(UpdateDelay)
	end
end)
