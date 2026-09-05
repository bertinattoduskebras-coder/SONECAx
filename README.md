--// CHAT PERSONALIZADO
--// Coloque este LocalScript em StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local TextChatService = game:GetService("TextChatService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer



-- COLOQUE SEU USER ID AQUI
local OWNER_USER_ID = 10368308763

local CHAT_WIDTH = 520
local CHAT_HEIGHT = 300

-- Tempo entre as piscadas do DONO
local BLINK_TIME = 0.55



pcall(function()
	TextChatService.ChatWindowConfiguration.Enabled = false
	TextChatService.ChatInputBarConfiguration.Enabled = false
end)



local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomChat"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Main = Instance.new("Frame")
Main.Name = "Chat"
Main.Size = UDim2.fromOffset(CHAT_WIDTH, CHAT_HEIGHT)
Main.Position = UDim2.new(0, 25, 1, -CHAT_HEIGHT - 80)
Main.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Main.BackgroundTransparency = 0.72
Main.BorderSizePixel = 0
Main.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 10)
Corner.Parent = Main

local Messages = Instance.new("ScrollingFrame")
Messages.Name = "Messages"
Messages.Size = UDim2.new(1, -15, 1, -55)
Messages.Position = UDim2.fromOffset(8, 8)
Messages.BackgroundTransparency = 1
Messages.BorderSizePixel = 0
Messages.ScrollBarThickness = 3
Messages.CanvasSize = UDim2.new(0, 0, 0, 0)
Messages.AutomaticCanvasSize = Enum.AutomaticSize.Y
Messages.Parent = Main

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0, 3)
Layout.SortOrder = Enum.SortOrder.LayoutOrder
Layout.Parent = Messages

local Input = Instance.new("TextBox")
Input.Name = "Input"
Input.Size = UDim2.new(1, -16, 0, 38)
Input.Position = UDim2.new(0, 8, 1, -45)
Input.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Input.BackgroundTransparency = 0.45
Input.BorderSizePixel = 0
Input.ClearTextOnFocus = false
Input.Font = Enum.Font.Gotham
Input.TextSize = 15
Input.TextColor3 = Color3.fromRGB(255, 255, 255)
Input.PlaceholderColor3 = Color3.fromRGB(180, 180, 180)
Input.PlaceholderText = "Digite uma mensagem..."
Input.Text = ""
Input.TextXAlignment = Enum.TextXAlignment.Left
Input.Parent = Main

local InputPadding = Instance.new("UIPadding")
InputPadding.PaddingLeft = UDim.new(0, 12)
InputPadding.Parent = Input

local InputCorner = Instance.new("UICorner")
InputCorner.CornerRadius = UDim.new(0, 8)
InputCorner.Parent = Input

--------------------------------------------------
-- COR DO NOME
--------------------------------------------------

local function GetNameColor(name)
	-- A mesma pessoa sempre recebe a mesma cor.
	-- Se mudar o nome, a cor muda.

	local hash = 0

	for i = 1, #name do
		hash = (hash * 31 + string.byte(name, i)) % 360
	end

	local hue = hash / 360

	return Color3.fromHSV(hue, 0.75, 1)
end

--------------------------------------------------
-- CRIA MENSAGEM
--------------------------------------------------

local function AddMessage(message)
	local TextSource = message.TextSource

	if not TextSource then
		return
	end

	local player = Players:GetPlayerByUserId(TextSource.UserId)

	if not player then
		return
	end

	local messageText = message.Text
	local name = player.DisplayName

	local Line = Instance.new("TextLabel")
	Line.Name = "Message"
	Line.Size = UDim2.new(1, -5, 0, 24)
	Line.AutomaticSize = Enum.AutomaticSize.Y
	Line.BackgroundTransparency = 1
	Line.TextWrapped = true
	Line.RichText = true
	Line.Font = Enum.Font.Gotham
	Line.TextSize = 15
	Line.TextXAlignment = Enum.TextXAlignment.Left
	Line.TextYAlignment = Enum.TextYAlignment.Top
	Line.TextColor3 = Color3.fromRGB(255, 255, 255)

	local nameColor = GetNameColor(name)

	local r = math.floor(nameColor.R * 255)
	local g = math.floor(nameColor.G * 255)
	local b = math.floor(nameColor.B * 255)

	local hex = string.format("#%02X%02X%02X", r, g, b)

	--------------------------------------------------
	-- DONO
	--------------------------------------------------

	if player.UserId == OWNER_USER_ID then

		Line.Text =
			'<font color="#FFFFFF"><b>[DONO]</b></font> ' ..
			'<font color="' .. hex .. '"><b>' ..
			name ..
			'</b></font>: ' ..
			messageText

		Line.Parent = Messages

		-- Efeito piscando somente no DONO
		task.spawn(function()

			local visible = true

			while Line.Parent do

				visible = not visible

				local transparency

				if visible then
					transparency = 0
				else
					transparency = 0.45
				end

				local tween = TweenService:Create(
					Line,
					TweenInfo.new(BLINK_TIME, Enum.EasingStyle.Sine),
					{
						TextTransparency = transparency
					}
				)

				tween:Play()
				tween.Completed:Wait()
			end
		end)

	else

		Line.Text =
			'<font color="' .. hex .. '"><b>' ..
			name ..
			'</b></font>: ' ..
			messageText

		Line.Parent = Messages
	end

	--------------------------------------------------
	-- LIMITE DE MENSAGENS
	--------------------------------------------------

	local children = Messages:GetChildren()

	local messageCount = 0

	for _, child in ipairs(children) do
		if child:IsA("TextLabel") then
			messageCount += 1
		end
	end

	if messageCount > 60 then
		for _, child in ipairs(children) do
			if child:IsA("TextLabel") then
				child:Destroy()
				break
			end
		end
	end

	task.wait()

	Messages.CanvasPosition = Vector2.new(
		0,
		Messages.AbsoluteCanvasSize.Y
	)
end


end)

--------------------------------------------------
-- ENVIAR MENSAGEM
--------------------------------------------------

Input.FocusLost:Connect(function(enterPressed)

Event:FireServer(text)

	if not enterPressed then
		return
	end

	local text = Input.Text

	if text == "" then
		return
	end

	Input.Text = ""

	local TextChannels = TextChatService:WaitForChild("TextChannels")
	local General = TextChannels:FindFirstChild("RBXGeneral")

	if General then
		General:SendAsync(text)
	end
end)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local Event = ReplicatedStorage:WaitForChild("SonecaZChatEvent")

--------------------------------------------------
-- CONFIG
--------------------------------------------------

local OWNER_USER_ID = 10368308763

local CHAT_WIDTH = 520
local CHAT_HEIGHT = 300
local BLINK_TIME = 0.55

--------------------------------------------------
-- GUI
--------------------------------------------------

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomChat"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Main = Instance.new("Frame")
Main.Name = "Chat"
Main.Size = UDim2.fromOffset(CHAT_WIDTH, CHAT_HEIGHT)
Main.Position = UDim2.new(0, 25, 1, -CHAT_HEIGHT - 80)
Main.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Main.BackgroundTransparency = 0.28
Main.BorderSizePixel = 0
Main.Active = true
Main.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 10)
Corner.Parent = Main

--------------------------------------------------
-- MARCA D'ÁGUA
--------------------------------------------------

local Watermark = Instance.new("TextLabel")
Watermark.Size = UDim2.fromScale(1, 1)
Watermark.BackgroundTransparency = 1
Watermark.Text = "SonecaZ"
Watermark.Font = Enum.Font.GothamBlack
Watermark.TextSize = 55
Watermark.TextColor3 = Color3.fromRGB(255, 255, 255)
Watermark.TextTransparency = 0.92
Watermark.ZIndex = 0
Watermark.Parent = Main

--------------------------------------------------
-- MENSAGENS
--------------------------------------------------

local Messages = Instance.new("ScrollingFrame")
Messages.Name = "Messages"
Messages.Size = UDim2.new(1, -15, 1, -55)
Messages.Position = UDim2.fromOffset(8, 8)
Messages.BackgroundTransparency = 1
Messages.BorderSizePixel = 0
Messages.ScrollBarThickness = 3
Messages.AutomaticCanvasSize = Enum.AutomaticSize.Y
Messages.CanvasSize = UDim2.new(0, 0, 0, 0)
Messages.ZIndex = 2
Messages.Parent = Main

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0, 3)
Layout.SortOrder = Enum.SortOrder.LayoutOrder
Layout.Parent = Messages

--------------------------------------------------
-- INPUT
--------------------------------------------------

local Input = Instance.new("TextBox")
Input.Name = "Input"
Input.Size = UDim2.new(1, -16, 0, 38)
Input.Position = UDim2.new(0, 8, 1, -45)
Input.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Input.BackgroundTransparency = 0.45
Input.BorderSizePixel = 0
Input.ClearTextOnFocus = false
Input.Font = Enum.Font.Gotham
Input.TextSize = 15
Input.TextColor3 = Color3.fromRGB(255, 255, 255)
Input.PlaceholderColor3 = Color3.fromRGB(180, 180, 180)
Input.PlaceholderText = "Digite uma mensagem..."
Input.Text = ""
Input.TextXAlignment = Enum.TextXAlignment.Left
Input.ZIndex = 3
Input.Parent = Main

local InputPadding = Instance.new("UIPadding")
InputPadding.PaddingLeft = UDim.new(0, 12)
InputPadding.Parent = Input

local InputCorner = Instance.new("UICorner")
InputCorner.CornerRadius = UDim.new(0, 8)
InputCorner.Parent = Input

--------------------------------------------------
-- COR DO NOME
--------------------------------------------------

local function GetNameColor(name)
	local hash = 0

	for i = 1, #name do
		hash = (hash * 31 + string.byte(name, i)) % 360
	end

	return Color3.fromHSV(hash / 360, 0.75, 1)
end

--------------------------------------------------
-- ADICIONAR MENSAGEM
--------------------------------------------------

local function AddMessage(userId, name, messageText)

	local Line = Instance.new("TextLabel")

	Line.Name = "Message"
	Line.Size = UDim2.new(1, -5, 0, 24)
	Line.AutomaticSize = Enum.AutomaticSize.Y
	Line.BackgroundTransparency = 1
	Line.TextWrapped = true
	Line.RichText = true
	Line.Font = Enum.Font.Gotham
	Line.TextSize = 15
	Line.TextXAlignment = Enum.TextXAlignment.Left
	Line.TextYAlignment = Enum.TextYAlignment.Top
	Line.TextColor3 = Color3.fromRGB(255, 255, 255)
	Line.ZIndex = 3

	local nameColor = GetNameColor(name)

	local r = math.floor(nameColor.R * 255)
	local g = math.floor(nameColor.G * 255)
	local b = math.floor(nameColor.B * 255)

	local hex = string.format("#%02X%02X%02X", r, g, b)

	--------------------------------------------------
	-- DONO
	--------------------------------------------------

	if userId == OWNER_USER_ID then

		Line.Text =
			'<font color="#FFFFFF"><b>[DONO]</b></font> ' ..
			'<font color="' .. hex .. '"><b>' ..
			name ..
			'</b></font>: ' ..
			messageText

		Line.Parent = Messages

		task.spawn(function()

			local visible = true

			while Line.Parent do

				visible = not visible

				local transparency = visible and 0 or 0.45

				local tween = TweenService:Create(
					Line,
					TweenInfo.new(BLINK_TIME, Enum.EasingStyle.Sine),
					{
						TextTransparency = transparency
					}
				)

				tween:Play()
				tween.Completed:Wait()
			end
		end)

	else

		Line.Text =
			'<font color="' .. hex .. '"><b>' ..
			name ..
			'</b></font>: ' ..
			messageText

		Line.Parent = Messages
	end

	--------------------------------------------------
	-- LIMITE
	--------------------------------------------------

	local labels = {}

	for _, child in ipairs(Messages:GetChildren()) do
		if child:IsA("TextLabel") then
			table.insert(labels, child)
		end
	end

	if #labels > 60 then
		labels[1]:Destroy()
	end

	task.wait()

	Messages.CanvasPosition = Vector2.new(
		0,
		Messages.AbsoluteCanvasSize.Y
	)
end

--------------------------------------------------
-- RECEBER APENAS O CHAT SECUNDÁRIO
--------------------------------------------------

Event.OnClientEvent:Connect(function(userId, name, text)

	AddMessage(userId, name, text)

end)

--------------------------------------------------
-- ENVIAR
--------------------------------------------------

Input.FocusLost:Connect(function(enterPressed)

	if not enterPressed then
		return
	end

	local text = Input.Text

	if text == "" then
		return
	end

	Input.Text = ""

	Event:FireServer(text)

end)

--------------------------------------------------
-- ARRASTAR CHAT
--------------------------------------------------

local dragging = false
local dragStart
local startPosition

Main.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1 then

		dragging = true
		dragStart = input.Position
		startPosition = Main.Position

	end

end)

Main.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end

end)

game:GetService("UserInputService").InputChanged:Connect(function(input)

	if not dragging then
		return
	end

	if input.UserInputType == Enum.UserInputType.MouseMovement then

		local delta = input.Position - dragStart

		Main.Position = UDim2.new(
			startPosition.X.Scale,
			startPosition.X.Offset + delta.X,
			startPosition.Y.Scale,
			startPosition.Y.Offset + delta.Y
		)

	end

end)
