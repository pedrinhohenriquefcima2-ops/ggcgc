local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local lp = Players.LocalPlayer

if not lp:GetAttribute("IsAdmin") then return end

local ESPEnabled = false
local AimlockEnabled = false
local AimlockKey = Enum.KeyCode.E
local FOV = 150
local ESPCache = {}
local Camera = workspace.CurrentCamera

local function addESP(plr)
	if ESPCache[plr] then return end
	local char = plr.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	local hum = char:FindFirstChild("Humanoid")
	if not hrp or not hum then return end

	local box = Instance.new("BoxHandleAdornment")
	box.Adornee = hrp
	box.Size = Vector3.new(2,3,1)
	box.Color3 = Color3.fromRGB(255,0,0)
	box.AlwaysOnTop = true
	box.ZIndex = 5
	box.Parent = hrp

	local billboard = Instance.new("BillboardGui")
	billboard.Adornee = hrp
	billboard.Size = UDim2.new(0,100,0,50)
	billboard.AlwaysOnTop = true
	billboard.StudsOffset = Vector3.new(0,3,0)
	billboard.Parent = hrp

	local label = Instance.new("TextLabel", billboard)
	label.Size = UDim2.fromScale(1,1)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1,1,1)
	label.Font = Enum.Font.GothamBold
	label.TextSize = 14

	local conn
	conn = RunService.RenderStepped:Connect(function()
		if not ESPEnabled or not char.Parent then
			conn:Disconnect()
			box:Destroy()
			billboard:Destroy()
			ESPCache[plr] = nil
			return
		end
		label.Text = plr.Name.." | "..math.floor(hum.Health).."/"..math.floor(hum.MaxHealth)
	end)

	ESPCache[plr] = {box=box, billboard=billboard, conn=conn}
end

local function removeESP()
	for _,v in pairs(ESPCache) do
		if v.box then v.box:Destroy() end
		if v.billboard then v.billboard:Destroy() end
		if v.conn then v.conn:Disconnect() end
	end
	ESPCache = {}
end

local function toggleESP(state)
	ESPEnabled = state
	if not state then removeESP() return end
	for _,plr in pairs(Players:GetPlayers()) do
		if plr ~= lp and plr.Team ~= lp.Team then
			addESP(plr)
		end
	end
end

Players.PlayerAdded:Connect(function(plr)
	plr.CharacterAdded:Connect(function()
		if ESPEnabled and plr ~= lp and plr.Team ~= lp.Team then
			task.wait(1)
			addESP(plr)
		end
	end)
end)

local FOVCircle = Instance.new("Frame")
FOVCircle.Size = UDim2.fromOffset(FOV*2, FOV*2)
FOVCircle.Position = UDim2.fromScale(0.5,0.5) - UDim2.fromOffset(FOV,FOV)
FOVCircle.BackgroundColor3 = Color3.fromRGB(0,170,255)
FOVCircle.BackgroundTransparency = 0.7
FOVCircle.BorderSizePixel = 0
FOVCircle.Visible = true
FOVCircle.AnchorPoint = Vector2.new(0.5,0.5)
FOVCircle.Parent = lp:WaitForChild("PlayerGui"):WaitForChild("ScreenGui") or Instance.new("ScreenGui", lp.PlayerGui)

local function getClosest()
	local closest, dist = nil, FOV
	for _,plr in pairs(Players:GetPlayers()) do
		if plr ~= lp and plr.Team ~= lp.Team and plr.Character then
			local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
			if hrp then
				local magnitude = (Camera.CFrame.Position - hrp.Position).Magnitude
				if magnitude < dist then
					dist = magnitude
					closest = hrp
				end
			end
		end
	end
	return closest
end

RunService.RenderStepped:Connect(function()
	if AimlockEnabled then
		local target = getClosest()
		if target then
			Camera.CFrame = Camera.CFrame:Lerp(
				CFrame.new(Camera.CFrame.Position, target.Position),
				0.08
			)
		end
	end
end)

UIS.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == AimlockKey then
		AimlockEnabled = not AimlockEnabled
	end
end)

local gui = Instance.new("ScreenGui", lp.PlayerGui)
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.fromScale(0.38,0.48)
main.Position = UDim2.fromScale(0.31,0.26)
main.BackgroundColor3 = Color3.fromRGB(20,20,20)
main.Active = true
main.Draggable = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0,16)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.fromScale(1,0.15)
title.Position = UDim2.fromScale(0,0)
title.Text = "PEDRO HUB"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 22

local Tabs = {"ESP","Aim","Visual"}
local Pages = {}

local tabFrame = Instance.new("Frame", main)
tabFrame.Size = UDim2.fromScale(1,0.12)
tabFrame.Position = UDim2.fromScale(0,0.15)
tabFrame.BackgroundTransparency = 1

local pageHolder = Instance.new("Frame", main)
pageHolder.Size = UDim2.fromScale(1,0.73)
pageHolder.Position = UDim2.fromScale(0,0.27)
pageHolder.BackgroundTransparency = 1

for i,tabName in ipairs(Tabs) do
	local btn = Instance.new("TextButton", tabFrame)
	btn.Size = UDim2.fromScale(1/#Tabs,1)
	btn.Position = UDim2.fromScale((i-1)/#Tabs,0)
	btn.Text = tabName
	btn.BackgroundColor3 = Color3.fromRGB(35,35,35)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	Instance.new("UICorner", btn)

	local page = Instance.new("Frame", pageHolder)
	page.Size = UDim2.fromScale(1,1)
	page.BackgroundTransparency = 1
	page.Visible = (i==1)
	Pages[tabName] = page

	btn.MouseButton1Click:Connect(function()
		for _,p in pairs(Pages) do p.Visible=false end
		page.Visible = true
	end)
end

local function createToggle(parent,text,posY,callback,iconId)
	local holder = Instance.new("Frame", parent)
	holder.Size = UDim2.fromScale(0.85,0.12)
	holder.Position = UDim2.fromScale(0.075,posY)
	holder.BackgroundColor3 = Color3.fromRGB(35,35,35)
	Instance.new("UICorner", holder)

	local icon = Instance.new("ImageLabel", holder)
	icon.Size = UDim2.fromScale(0.15,0.8)
	icon.Position = UDim2.fromScale(0.02,0.1)
	icon.BackgroundTransparency = 1
	icon.Image = iconId or ""
	icon.ScaleType = Enum.ScaleType.Fit

	local label = Instance.new("TextLabel", holder)
	label.Size = UDim2.fromScale(0.65,1)
	label.Position = UDim2.fromScale(0.2,0)
	label.BackgroundTransparency = 1
	label.Text = text
	label.TextColor3 = Color3.new(1,1,1)
	label.Font = Enum.Font.Gotham
	label.TextSize = 14

	local btn = Instance.new("TextButton", holder)
	btn.Size = UDim2.fromScale(0.2,0.6)
	btn.Position = UDim2.fromScale(0.75,0.2)
	btn.BackgroundColor3 = Color3.fromRGB(70,70,70)
	btn.Text = ""
	Instance.new("UICorner", btn)

	local state = false
	btn.MouseButton1Click:Connect(function()
		state = not state
		TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = state and Color3.fromRGB(0,170,255) or Color3.fromRGB(70,70,70)}):Play()
		callback(state)
	end)
end

createToggle(Pages["ESP"],"ESP",0.15,toggleESP,"rbxassetid://6031094676")
createToggle(Pages["Aim"],"Aimlock (E)",0.15,function(v) AimlockEnabled=v end,"rbxassetid://6031094786")
