-- LocalScript dentro de StarterGui
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Workspace = game:GetService("Workspace")

-- Criar ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = Player:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

-- Painel flutuante
local Painel = Instance.new("Frame")
Painel.Size = UDim2.new(0, 150, 0, 50)
Painel.Position = UDim2.new(0.8, 0, 0.8, 0)
Painel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Painel.BorderSizePixel = 0
Painel.Parent = ScreenGui
Painel.Active = true
Painel.Draggable = true

-- Título do Painel
local Titulo = Instance.new("TextButton")
Titulo.Size = UDim2.new(1, 0, 1, 0)
Titulo.BackgroundTransparency = 1
Titulo.Text = "Macaco Hub"
Titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
Titulo.Font = Enum.Font.SourceSansBold
Titulo.TextSize = 18
Titulo.Parent = Painel

-- Catálogo oculto
local Catalogo = Instance.new("Frame")
Catalogo.Size = UDim2.new(0, 150, 0, 220)
Catalogo.Position = UDim2.new(0, 0, -4, -50)
Catalogo.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Catalogo.BorderSizePixel = 0
Catalogo.Visible = false
Catalogo.Parent = Painel

-- Layout automático
local UIListLayout = Instance.new("UIListLayout")
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 5)
UIListLayout.Parent = Catalogo

-- Função para criar botões
local function criarBotao(nome, cor, callback)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(1, -10, 0, 40)
    botao.BackgroundColor3 = cor
    botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    botao.Font = Enum.Font.SourceSansBold
    botao.TextSize = 16
    botao.Text = nome
    botao.Parent = Catalogo
    botao.MouseButton1Click:Connect(callback)
    return botao
end

-- =========================
-- Variáveis gerais
-- =========================
local plataformaAtiva = false
local plataforma
local conexaoPlataforma

local hitboxAtiva = false
local hitboxes = {}

-- =========================
-- 1. Macaco Plataforma
-- =========================
criarBotao("Macaco Plataforma", Color3.fromRGB(0, 200, 255), function()
	local Char = Player.Character or Player.CharacterAdded:Wait()
	local HRP = Char:WaitForChild("HumanoidRootPart")

	if not plataformaAtiva then
		plataforma = Instance.new("Part")
		plataforma.Size = Vector3.new(6,1,6)
		plataforma.Anchored = true
		plataforma.Color = Color3.fromRGB(0, 200, 255)
		plataforma.Material = Enum.Material.Neon
		plataforma.CanCollide = true
		plataforma.Parent = workspace

		conexaoPlataforma = RunService.RenderStepped:Connect(function()
			if plataforma and HRP then
				plataforma.CFrame = HRP.CFrame * CFrame.new(0, -3, 0)
			end
		end)

		plataformaAtiva = true
	else
		if conexaoPlataforma then conexaoPlataforma:Disconnect() end
		if plataforma then plataforma:Destroy() end
		plataformaAtiva = false
	end
end)

-- =========================
-- 2. Macaco Teleporte
-- =========================
criarBotao("Macaco Teleporte", Color3.fromRGB(255, 50, 50), function()
	for _, plr in pairs(Players:GetPlayers()) do
		if plr ~= Player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
			plr.Character.HumanoidRootPart.CFrame = HumanoidRootPart.CFrame + Vector3.new(2,0,0)
		end
	end
end)

-- =========================
-- 3. Macaco Hitbox (do próprio player)
-- =========================
criarBotao("Macaco Hitbox", Color3.fromRGB(50, 255, 50), function()
	if HumanoidRootPart then
		HumanoidRootPart.Size = Vector3.new(1, 1, 1)
	end
end)

-- =========================
-- 4. Macaco Velocidade
-- =========================
criarBotao("Macaco Velocidade", Color3.fromRGB(150, 50, 255), function()
	local humanoid = Character:FindFirstChild("Humanoid")
	if humanoid then
		humanoid.WalkSpeed = 100
	end
end)

-- =========================
-- 5. Mostrar Hitbox de todos os players
-- =========================
criarBotao("Mostrar Hitbox", Color3.fromRGB(255, 165, 0), function()
	hitboxAtiva = not hitboxAtiva

	-- Limpar hitboxes existentes
	for _, hb in pairs(hitboxes) do
		if hb and hb.Parent then
			hb:Destroy()
		end
	end
	hitboxes = {}

	if hitboxAtiva then
		for _, plr in pairs(Players:GetPlayers()) do
			if plr ~= Player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
				local char = plr.Character
				local hrp = char:FindFirstChild("HumanoidRootPart")
				if hrp then
					local hb = Instance.new("Part")
					hb.Size = hrp.Size
					hb.CFrame = hrp.CFrame
					hb.Transparency = 0.5
					hb.Anchored = true
					hb.CanCollide = false
					hb.Color = Color3.fromRGB(255, 0, 0)
					hb.Material = Enum.Material.Neon
					hb.Parent = workspace
					table.insert(hitboxes, {Part = hb, Player = plr})
				end
			end
		end

		RunService.RenderStepped:Connect(function()
			if hitboxAtiva then
				for _, info in pairs(hitboxes) do
					if info.Part and info.Player.Character and info.Player.Character:FindFirstChild("HumanoidRootPart") then
						info.Part.CFrame = info.Player.Character.HumanoidRootPart.CFrame
					end
				end
			end
		end)
	else
		-- Desativar e remover hitboxes
		for _, info in pairs(hitboxes) do
			if info.Part and info.Part.Parent then
				info.Part:Destroy()
			end
		end
		hitboxes = {}
	end
end)

-- =========================
-- Abrir/fechar catálogo
-- =========================
local aberto = false
Titulo.MouseButton1Click:Connect(function()
	aberto = not aberto
	Catalogo.Visible = aberto
end)
