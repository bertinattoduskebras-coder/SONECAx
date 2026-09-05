local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Event = ReplicatedStorage:WaitForChild("SonecaZChatEvent")

--------------------------------------------------
-- CONFIGURAÇÕES
--------------------------------------------------

local OWNER_USER_ID = 10368308763

local CHAT_WIDTH = 520
local CHAT_HEIGHT = 300

local CHAT_TRANSPARENCY = 0.28
local INPUT_TRANSPARENCY = 0.45

local BLINK_TIME = 0.55

--------------------------------------------------
-- GUI PRINCIPAL
--------------------------------------------------

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SonecaZCustomChat"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

--------------------------------------------------
-- CHAT
--------------------------------------------------

local Main = Instance.new("Frame")
Main.Name = "SecondaryChat"
Main.Size = UDim2.fromOffset(CHAT_WIDTH, CHAT_HEIGHT)
Main.Position = UDim2.new(0, 25, 1, -CHAT_HEIGHT - 80)
Main.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Main.BackgroundTransparency = CHAT_TRANSPARENCY
Main.BorderSizePixel = 0
Main.Active = true
Main.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 10)
Corner.Parent = Main

--------------------------------------------------
-- MARCA D'ÁGUA SONECAZ
--------------------------------------------------

local Watermark = Instance.new("TextLabel")
Watermark.Name = "Watermark"
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
Input.BackgroundTransparency = INPUT_TRANSPARENCY
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
-- BOTÃO MOSTRAR / ESCONDER
--------------------------------------------------

local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleChat"
ToggleButton.Size = UDim2.fromOffset(115, 35)
ToggleButton.Position = UDim2.new(0, 25, 1, -40)
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
ToggleButton.BackgroundTransparency = 0.25
ToggleButton.BorderSizePixel = 0
ToggleButton.Text = "CHAT"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 14
ToggleButton.Parent = ScreenGui

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 8)
ToggleCorner.Parent = ToggleButton

local ChatVisible = true

ToggleButton.MouseButton1Click:Connect(function()
	ChatVisible = not ChatVisible
	Main.Visible = ChatVisible
end)

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
	-- LIMITE DE 60 MENSAGENS
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
-- RECEBER SOMENTE O CHAT SECUNDÁRIO
--------------------------------------------------

Event.OnClientEvent:Connect(function(userId, name, text)
	AddMessage(userId, name, text)
end)

--------------------------------------------------
-- ENVIAR SOMENTE PARA O CHAT SECUNDÁRIO
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

	-- NÃO USA RBXGeneral
	-- NÃO USA TextChatService.MessageReceived
	Event:FireServer(text)
end)

--------------------------------------------------
-- ARRASTAR O CHAT
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

UserInputService.InputChanged:Connect(function(input)
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
