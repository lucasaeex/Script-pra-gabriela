-- FRONTLINES ESP + AIMBOT + TELEPORT MENU
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local ESPEnabled = false
local AimbotEnabled = false
local Aiming = false
local FOV = 100

-- GUI
local GUI = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", GUI)
Frame.Size = UDim2.new(0, 160, 0, 150)
Frame.Position = UDim2.new(0, 10, 0, 100)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.Draggable = true
Frame.Active = true

local function criarBotao(texto, posY)
	local btn = Instance.new("TextButton", Frame)
	btn.Size = UDim2.new(1, 0, 0, 35)
	btn.Position = UDim2.new(0, 0, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.SourceSans
	btn.TextSize = 14
	btn.Text = texto
	return btn
end

local btnESP = criarBotao("Ativar ESP", 0)
local btnAimbot = criarBotao("Ativar Aimbot", 35)
local btnTP = criarBotao("Teleportar", 70)

-- Verifica se é inimigo
function isInimigo(player)
	if player and player ~= LocalPlayer and player.Character then
		if player.Character:FindFirstChild("friendly_marker") then
			return false
		end
	end
	return true
end

-- ESP
function criarESP(player)
	local txt = Drawing.new("Text")
	txt.Size = 14
	txt.Center = true
	txt.Outline = true
	txt.Color = Color3.fromRGB(255, 0, 0)
	txt.Visible = false

	RunService.RenderStepped:Connect(function()
		if ESPEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and isInimigo(player) then
			local pos, vis = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
			if vis then
				txt.Position = Vector2.new(pos.X, pos.Y)
				txt.Text = player.Name
				txt.Visible = true
			else
				txt.Visible = false
			end
		else
			txt.Visible = false
		end
	end)
end

-- Aimbot alvo mais próximo
function getAlvo()
	local closest = nil
	local shortest = FOV
	local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

	for _, p in pairs(Players:GetPlayers()) do
		if isInimigo(p) and p.Character and p.Character:FindFirstChild("Head") then
			local pos, vis = Camera:WorldToViewportPoint(p.Character.Head.Position)
			if vis then
				local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
				if dist < shortest then
					shortest = dist
					closest = p
				end
			end
		end
	end
	return closest
end

UIS.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		Aiming = true
	end
end)
UIS.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		Aiming = false
	end
end)

RunService.RenderStepped:Connect(function()
	if AimbotEnabled and Aiming then
		local target = getAlvo()
		if target and target.Character and target.Character:FindFirstChild("Head") then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
		end
	end
end)

-- Teleportar
function teleportar()
	local alvo = getAlvo()
	if alvo and alvo.Character and alvo.Character:FindFirstChild("HumanoidRootPart") then
		LocalPlayer.Character:SetPrimaryPartCFrame(alvo.Character.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0))
	end
end

-- Botões
btnESP.MouseButton1Click:Connect(function()
	ESPEnabled = not ESPEnabled
	btnESP.Text = ESPEnabled and "Desativar ESP" or "Ativar ESP"
	if ESPEnabled then
		for _, p in pairs(Players:GetPlayers()) do
			if isInimigo(p) then criarESP(p) end
		end
	end
end)

btnAimbot.MouseButton1Click:Connect(function()
	AimbotEnabled = not AimbotEnabled
	btnAimbot.Text = AimbotEnabled and "Desativar Aimbot" or "Ativar Aimbot"
end)

btnTP.MouseButton1Click:Connect(function()
	teleportar()
end)

print("script deu certo!!")
