--// Painel GUI Completo - Mobile

local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local uis = game:GetService("UserInputService")
local camera = game.Workspace.CurrentCamera
local TweenService = game:GetService("TweenService")

-- Velocidade padrão
local defaultWalkSpeed = player.Character and player.Character:FindFirstChildOfClass("Humanoid") and player.Character:FindFirstChildOfClass("Humanoid").WalkSpeed or 16
local speedClicks = 0

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false

-- Painel
local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 230, 0, 380)
main.Position = UDim2.new(0, 30, 0.2, 0)
main.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true

local UIListLayout = Instance.new("UIListLayout", main)
UIListLayout.Padding = UDim.new(0, 6)

local function criarBotao(nome, callback)
	local btn = Instance.new("TextButton", main)
	btn.Size = UDim2.new(1, -10, 0, 30)
	btn.Position = UDim2.new(0, 5, 0, 0)
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.GothamBold
	btn.Text = nome
	btn.TextSize = 14
	btn.BorderSizePixel = 0

	local ativado = false

	btn.MouseButton1Click:Connect(function()
		ativado = not ativado
		btn.BackgroundColor3 = ativado and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(50, 50, 50)
		callback(ativado)
	end)
end

-- Aimbot
local mira = Instance.new("Frame", gui)
mira.Size = UDim2.new(0, 80, 0, 80)
mira.Position = UDim2.new(0.5, -40, 0.5, -40)
mira.BackgroundTransparency = 1
mira.AnchorPoint = Vector2.new(0.5, 0.5)

local circle = Instance.new("UICorner", mira)
circle.CornerRadius = UDim.new(1, 0)
local border = Instance.new("Frame", mira)
border.Size = UDim2.new(1, 0, 1, 0)
border.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
border.BackgroundTransparency = 0.5
border.BorderSizePixel = 0
border.ZIndex = 2

criarBotao("Aimbot", function(on)
	local runService = game:GetService("RunService")
	local aiming = on

	local connection
	if aiming then
		connection = runService.RenderStepped:Connect(function()
			local closestMob
			local shortestDistance = math.huge

			for _, mob in pairs(workspace:GetDescendants()) do
				if mob:IsA("Model") and mob:FindFirstChild("Humanoid") and mob:FindFirstChild("Head") and mob ~= player.Character then
					local screenPos, onScreen = camera:WorldToScreenPoint(mob.Head.Position)
					if onScreen then
						local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
						if dist < shortestDistance then
							shortestDistance = dist
							closestMob = mob
						end
					end
				end
			end

			if closestMob then
				camera.CFrame = CFrame.new(camera.CFrame.Position, closestMob.Head.Position)
			end
		end)
	else
		if connection then connection:Disconnect() end
	end
end)

-- ESP geral
local function criarESP(tipo, cor)
	for _, obj in pairs(workspace:GetDescendants()) do
		if obj:IsA("Model") and obj:FindFirstChild("Head") and obj:FindFirstChildOfClass("Humanoid") and tipo == "Mob" then
			local bill = Instance.new("BillboardGui", obj.Head)
			bill.Size = UDim2.new(0, 100, 0, 40)
			bill.AlwaysOnTop = true

			local texto = Instance.new("TextLabel", bill)
			texto.Size = UDim2.new(1, 0, 1, 0)
			texto.BackgroundTransparency = 1
			texto.Text = obj.Name
			texto.TextColor3 = cor
			texto.TextScaled = true
		end
	end
end

criarBotao("ESP Mob", function(on)
	if on then
		criarESP("Mob", Color3.fromRGB(0, 255, 0))
	end
end)

-- Auto Coletar
criarBotao("Auto Coletar", function(on)
	if on then
		for _, prompt in ipairs(workspace:GetDescendants()) do
			if prompt:IsA("ProximityPrompt") and prompt.MaxActivationDistance > 0 then
				fireproximityprompt(prompt)
			end
		end
	end
end)

-- Auto Comprar
criarBotao("Auto Comprar", function(on)
	if on then
		for _, prompt in ipairs(workspace:GetDescendants()) do
			if prompt:IsA("ProximityPrompt") and string.lower(prompt.Parent.Name):find("purch") then
				fireproximityprompt(prompt)
			end
		end
	end
end)

-- Speed
criarBotao("Speed +1", function(on)
	if on then
		speedClicks += 1
		local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
		if humanoid then
			if speedClicks < 5 then
				humanoid.WalkSpeed += 1
			else
				humanoid.WalkSpeed = defaultWalkSpeed
				speedClicks = 0
			end
		end
	end
end)

-- Noclip
criarBotao("Noclip", function(on)
	if on then
		game:GetService("RunService").Stepped:Connect(function()
			if player.Character then
				for _, part in pairs(player.Character:GetDescendants()) do
					if part:IsA("BasePart") then
						part.CanCollide = false
					end
				end
			end
		end)
	end
end)

-- Anti-Void
criarBotao("Anti-Void", function(on)
	if on then
		game:GetService("RunService").Heartbeat:Connect(function()
			if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
				if player.Character.HumanoidRootPart.Position.Y < -10 then
					player.Character.HumanoidRootPart.CFrame = CFrame.new(0, 10, 0)
				end
			end
		end)
	end
end)

-- X-Ray
criarBotao("X-Ray", function(on)
	for _, part in ipairs(workspace:GetDescendants()) do
		if part:IsA("BasePart") and part.Transparency < 1 then
			part.LocalTransparencyModifier = on and 0.8 or 0
		end
	end
end)

-- WallWalk
criarBotao("WallWalk", function(on)
	if on then
		player.Character.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame + Vector3.new(0, 1, 0)
	end
end)

-- Anti-Lag
criarBotao("Anti-Lag", function(on)
	if on then
		for _, obj in ipairs(workspace:GetDescendants()) do
			if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Explosion") or obj:IsA("Decal") then
				obj:Destroy()
			end
		end
	end
end)
