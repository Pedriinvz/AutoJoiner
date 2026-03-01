local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local remote = ReplicatedStorage:WaitForChild("ListaPetsFortes")
local player = Players.LocalPlayer

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

-- BOTÃO GRANDE (MOBILE FRIENDLY)
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 180, 0, 50)
toggleButton.Position = UDim2.new(0, 20, 0.5, -260)
toggleButton.BackgroundColor3 = Color3.fromRGB(40,40,40)
toggleButton.TextColor3 = Color3.new(1,1,1)
toggleButton.TextScaled = true
toggleButton.Text = "📋 Mostrar Lista"
toggleButton.Parent = screenGui

-- FRAME PRINCIPAL
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 320, 0, 420)
frame.Position = UDim2.new(0, 20, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.Visible = false
frame.Parent = screenGui

-- TÍTULO
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(40,40,40)
title.TextColor3 = Color3.new(1,1,1)
title.TextScaled = true
title.Text = "🔥 Pets acima de 300M (0)"
title.Parent = frame

-- LISTA
local listFrame = Instance.new("Frame")
listFrame.Size = UDim2.new(1, 0, 1, -50)
listFrame.Position = UDim2.new(0, 0, 0, 50)
listFrame.BackgroundTransparency = 1
listFrame.Parent = frame

-- ABRIR / FECHAR
toggleButton.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
	
	if frame.Visible then
		toggleButton.Text = "❌ Fechar Lista"
	else
		toggleButton.Text = "📋 Mostrar Lista"
	end
end)

-- SISTEMA DRAG (PC + MOBILE)
local dragging = false
local dragStart
local startPos

local function update(input)
	local delta = input.Position - dragStart
	
	frame.Position = UDim2.new(
		startPos.X.Scale,
		math.clamp(startPos.X.Offset + delta.X, 0, screenGui.AbsoluteSize.X - frame.AbsoluteSize.X),
		startPos.Y.Scale,
		math.clamp(startPos.Y.Offset + delta.Y, 0, screenGui.AbsoluteSize.Y - frame.AbsoluteSize.Y)
	)
end

frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
		
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

frame.InputChanged:Connect(function(input)
	if dragging then
		update(input)
	end
end)

-- ATUALIZAR LISTA
remote.OnClientEvent:Connect(function(lista)

	listFrame:ClearAllChildren()
	title.Text = "🔥 Pets acima de 300M ("..#lista..")"
	
	for i, info in pairs(lista) do
		local label = Instance.new("TextLabel")
		label.Size = UDim2.new(1, 0, 0, 35)
		label.Position = UDim2.new(0, 0, 0, (i-1)*35)
		label.BackgroundTransparency = 1
		label.TextColor3 = Color3.fromRGB(0,255,0)
		label.TextScaled = true
		label.Text = "⚡ "..info.Nome.." - "..string.format("%,d", info.Valor)
		label.Parent = listFrame
	end
end)
