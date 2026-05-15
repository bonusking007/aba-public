-- ===== CONFIG =====
_G.main = false
_G.alt  = true
-- ==================

setfpscap(30)
local Players = game:GetService("Players")
local VirtualInputManager = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")
local HttpService = game:GetService("HttpService")
local UIS = game:GetService("UserInputService")
local LP = Players.LocalPlayer

local altCFrame = CFrame.new(20000, 2000, 20000)
local pauseCFrame = CFrame.new(124, 5, -23)
local baseName = "WWHub_BasePlate"

local tpDistanceLimit = 18
local pauseTpDistanceLimit = 8
local tpWatchdogDelay = 0.35

local m1Hit = CFrame.new(
	97.64178466796875, 497.5, -602.8313598632812,
	0.9989567399024963, 0.006808227859437466, -0.045158419758081436,
	4.656613428188905e-10, 0.9888255000114441, 0.14907847344875336,
	0.04566875472664833, -0.14892295002937317, 0.9877936840057373
)

local loopAlt       = false
local loopMain      = false
local jugResetting  = false
local starting      = false
local selectingTeam = false
local pointsCapped  = false
local roundPaused   = false
local roundPauseReason = nil
local roundResetting = false
local handledChar   = nil
local pressedKChar  = nil
local gui = nil

local pointCapLimit = 10000

local _request
if not pcall(function()
	_request = request or http_request or http.request
end) then
	warn("REQUEST NOT SUPPORTED")
	_request = function() end
end

local WebhookURL = "https://discord.com/api/webhooks/1453628734090514533/ddACObJX5Iuv966TcspBAEmkd5Er2ZfiVCMdoHzyONWLJ1CoqlDaAn3vg9D1GiZkvPoR"

local function sendWebhook(label)
	task.spawn(function()
		pcall(function()
			local moneyText, lvlText, pts = "N/A", "N/A", "N/A"
			pcall(function()
				moneyText = tostring(LP:WaitForChild("ReplicatedStats"):WaitForChild("Gold").Value)
				lvlText   = LP.PlayerGui.HUD.RightBotCorner.Line2.Lvl.Text
				pts       = tostring(LP.leaderstats.Points.Value)
			end)
			_request({
				Url = WebhookURL,
				Method = "POST",
				Headers = { ["Content-Type"] = "application/json" },
				Body = HttpService:JSONEncode({
					embeds = {{
						title = "⚡ WWHub — " .. label,
						color = 0x46C864,
						fields = {
							{ name = "Player", value = LP.Name,   inline = true },
							{ name = "Money",  value = moneyText, inline = true },
							{ name = "Level",  value = lvlText,   inline = true },
							{ name = "Points", value = pts,       inline = true },
						},
						footer = { text = os.date("%d/%m/%Y %H:%M:%S") }
					}}
				})
			})
		end)
	end)
end

local function getChar() return LP.Character end
local function getHRP()
	local char = getChar()
	if not char then return nil end
	return char:FindFirstChild("HumanoidRootPart")
end
local function getInput()
	local backpack = LP:FindFirstChild("Backpack")
	local char = getChar()
	return (backpack and backpack:FindFirstChild("Input"))
		or (char and char:FindFirstChild("Input"))
		or LP:FindFirstChild("Input")
end
local function fireInput(action, data)
	local input = getInput()
	if not input then return end
	pcall(function()
		if data then input:FireServer(action, data)
		else input:FireServer(action) end
	end)
end
local function selectIchigoPlay()
	fireInput("CharacterButton", "Ichigo")
	task.wait(0.2)
	fireInput("ClickPlay")
end
local function makeBase()
	local base = workspace:FindFirstChild(baseName)
	if not base then
		base = Instance.new("Part")
		base.Name = baseName
		base.Anchored = true
		base.Color = Color3.fromRGB(35, 35, 35)
		base.Material = Enum.Material.SmoothPlastic
		base.Parent = workspace
	end
	base.Size = Vector3.new(500, 0.4, 500)
	base.CFrame = CFrame.new(altCFrame.Position - Vector3.new(0, 3, 0))
	base.Anchored = true
end
local function fireM1()
	fireInput("M1", {
		air = false, skeyreal = false, skeydown = true,
		mousehit = m1Hit, md = Vector3.new(0, 0, 0)
	})
end
local function pressKey(key)
	pcall(function()
		VirtualInputManager:SendKeyEvent(true, key, false, game)
		task.wait(0.05)
		VirtualInputManager:SendKeyEvent(false, key, false, game)
	end)
end

local function fireSkills()
	local input = getInput()
	if not input then return end
	-- Skill 1
	pcall(function() input:FireServer("UseMove", { air=false, running=false, neutral=true, range="1", ToolName="Getsuga Tensho", mousehit=m1Hit, camdir=vector.create(-0.83,-0.065,-0.55), campos=vector.create(3083.8,579.3,473.5) }) end)
	pressKey(Enum.KeyCode.One)
	task.wait(0.15)
	-- Skill 2
	pcall(function() input:FireServer("UseMove", { air=false, running=false, neutral=true, range="2", ToolName="Getsuga Slash", mousehit=m1Hit, camdir=vector.create(-0.82,-0.033,-0.57), campos=vector.create(3083.6,578.9,473.7) }) end)
	pressKey(Enum.KeyCode.Two)
	task.wait(0.15)
	-- Skill 3
	pcall(function() input:FireServer("UseMove", { air=false, running=false, neutral=true, range="3", ToolName="Multi-Cut", mousehit=m1Hit, camdir=vector.create(-0.91,-0.095,-0.39), campos=vector.create(3047.1,579.7,438.5) }) end)
	pressKey(Enum.KeyCode.Three)
	task.wait(0.15)
	-- Skill 4
	pcall(function() input:FireServer("UseMove", { air=false, running=false, neutral=true, range="4", ToolName="Lunge", mousehit=m1Hit, camdir=vector.create(-0.75,-0.018,-0.65), campos=vector.create(3047.1,578.7,444.8) }) end)
	pressKey(Enum.KeyCode.Four)
	task.wait(0.15)
	pcall(function() input:FireServer("UseMode") end)
end
local function forceFieldOff() fireInput("ForceFieldOff") end
local function pressK()
	pcall(function()
		VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.K, false, game)
	end)
end

pcall(function()
	LP.Idled:Connect(function()
		pcall(function()
			VirtualUser:CaptureController()
			VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
			task.wait(0.1)
			VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
		end)
	end)
end)

local function fireRespawnDone()
	local pg = LP:WaitForChild("PlayerGui")
	local respawn = pg:FindFirstChild("Respawning")
	if respawn and respawn:FindFirstChild("Done") then
		pcall(function() respawn.Done:FireServer() end)
	end
end
local function afterCharacterLoaded(char)
	if not char or handledChar == char then return end
	handledChar = char
	pressedKChar = nil
	char:WaitForChild("HumanoidRootPart", 10)
	char:WaitForChild("Humanoid", 10)
	task.wait(1)
	fireRespawnDone()
	task.wait(0.2)
	forceFieldOff()
end
local function resetCharacter()
	local char = getChar()
	if char then
		local hum = char:FindFirstChildOfClass("Humanoid")
		if hum then hum.Health = 0 end
	end
	game:GetService("ReplicatedStorage"):WaitForChild("Loaded"):FireServer()
	task.wait(1)
	local newChar = getChar()
	if newChar then afterCharacterLoaded(newChar) end
end
local function getTeamPad()
	if loopAlt  then return workspace:FindFirstChild("Blue Team") end
	if loopMain then return workspace:FindFirstChild("Red Team")  end
	return nil
end
local function selectTeam()
	if selectingTeam then return end
	selectingTeam = true
	local pad = getTeamPad()
	if not pad then selectingTeam = false return end
	while workspace:FindFirstChild(pad.Name) and not roundPaused do
		local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
		if hrp then
			local padPart = pad:IsA("BasePart") and pad or pad:FindFirstChildWhichIsA("BasePart")
			if padPart then hrp.CFrame = padPart.CFrame + Vector3.new(0, 3, 0) end
		end
		task.wait(0.1)
	end
	task.wait(0.5)
	selectingTeam = false
end
local function getPoints()
	local ok, val = pcall(function() return LP.leaderstats.Points.Value end)
	return ok and val or 0
end
-- scan HUD ทั้งหมดหา TextLabel ที่เป็นตัวเลข (timer)
local function getTimerValue()
	local best = 0
	pcall(function()
		local hud = LP.PlayerGui:FindFirstChild("HUD")
		if not hud then return end
		for _, obj in ipairs(hud:GetDescendants()) do
			if obj:IsA("TextLabel") or obj:IsA("TextBox") then
				local n = tonumber(obj.Text)
				if n and n > best and n < 600 then
					best = n
				end
			end
		end
	end)
	return best
end

local timerTpDone = false -- กัน TP ซ้ำในตานั้น

-- timer watchdog: เช็คทุก 0.3 วิ, TP pause + หยุด skill เมื่อ timer ≤ 10
task.spawn(function()
	while true do
		task.wait(0.3)
		if loopMain then
			local pts   = getPoints()
			local timer = getTimerValue()

			-- point cap
			if not pointsCapped and pts >= pointCapLimit then
				pointsCapped = true
			end
			if pointsCapped and timer > 30 and pts < pointCapLimit then
				pointsCapped = false
			end

			-- timer ≤ 10 → TP ไปจุดพัก + หยุด skill
			if timer > 0 and timer <= 10 and not timerTpDone and not roundPaused then
				timerTpDone = true
				-- TP loop ซ้ำจน timer หมด
				task.spawn(function()
					for _ = 1, 8 do
						local hrp = getHRP()
						if hrp then
							pcall(function()
								hrp.AssemblyLinearVelocity = Vector3.new(0,0,0)
								hrp.AssemblyAngularVelocity = Vector3.new(0,0,0)
								hrp.CFrame = pauseCFrame
							end)
						end
						task.wait(0.5)
					end
				end)
			end

			-- reset เมื่อตาใหม่ (timer กลับมาสูง)
			if timer > 30 and timerTpDone then
				timerTpDone = false
			end
		else
			pointsCapped = false
			timerTpDone  = false
		end
	end
end)

local startFarm

local function getBlockedFarmMode()
	local pg = LP:FindFirstChild("PlayerGui")
	if not pg then return nil end
	local lb = pg:FindFirstChild("CustomLeaderboard")
	if not lb then return nil end
	local m = lb:FindFirstChild("Main") or lb
	if m:FindFirstChild("Juggernaut", true) then return "Juggernaut" end
	if m:FindFirstChild("Lives", true) then return "Lives" end
	for _, obj in ipairs(m:GetDescendants()) do
		local name = string.lower(obj.Name or "")
		if string.find(name, "juggernaut") then return "Juggernaut" end
		if string.find(name, "lives") then return "Lives" end
		if obj:IsA("TextLabel") or obj:IsA("TextBox") or obj:IsA("TextButton") then
			local text = string.lower(obj.Text or "")
			if string.find(text, "juggernaut") then return "Juggernaut" end
			if string.find(text, "lives") then return "Lives" end
		end
	end
	return nil
end

local tpPauseAndVerify

-- ดึงค่า lives จาก StockCount ("Lives: 5" → 5, "0" → 0)
local function getLives()
	local ok, val = pcall(function()
		local t = LP.PlayerGui.HUD.StockCount.Text
		-- รองรับ "Lives: 5" และ "5"
		return tonumber(t:match("%d+")) or 0
	end)
	return ok and val or 0
end

-- drain lives: reset ตัวซ้ำจนกว่า lives จะเป็น 0
local function drainLives()
	local lives = getLives()
	if lives <= 0 then return end
	print("Draining lives: " .. lives)
	while gui and gui.Parent and lives > 0 and (loopMain or loopAlt) do
		-- reset character
		local char = getChar()
		if char then
			local hum = char:FindFirstChildOfClass("Humanoid")
			if hum and hum.Health > 0 then
				hum.Health = 0
			end
		end
		task.wait(2) -- รอ respawn
		-- หลัง respawn แจ้ง done + ปิด forcefield
		local newChar = getChar()
		if newChar then
			afterCharacterLoaded(newChar)
		end
		task.wait(0.5)
		lives = getLives()
		print("Lives remaining: " .. lives)
	end
	print("Lives drained — waiting for next round")
end

local function pauseFarmForRound(reason)
	if roundPaused then return end
	roundPaused = true
	roundPauseReason = reason or "Blocked Mode"
	pointsCapped = false
	sendWebhook("Farm Paused — " .. roundPauseReason .. " Mode")
	if roundResetting then return end
	roundResetting = true
	task.spawn(function()
		-- drain lives ก่อน
		drainLives()
		-- TP ไป safe zone รอตาใหม่
		tpPauseAndVerify()
		-- รอจนกว่า blocked mode จะหายไป (ตาใหม่)
		local clearChecks = 0
		while gui and gui.Parent and (loopMain or loopAlt) and roundPaused do
			local blockedMode = getBlockedFarmMode()
			if blockedMode then
				clearChecks = 0
				tpPauseAndVerify()
				task.wait(1)
			else
				clearChecks += 1
				if clearChecks >= 3 then break end
				task.wait(1)
			end
		end
		local lastReason = roundPauseReason or "Blocked Mode"
		roundPaused = false
		roundPauseReason = nil
		roundResetting = false
		if gui and gui.Parent and (loopMain or loopAlt) then
			sendWebhook("Farm Resumed — Next Round after " .. lastReason)
		end
	end)
end

startFarm = function(mode)
	if starting then return end
	starting = true
	loopAlt  = false
	loopMain = false
	makeBase()
	selectIchigoPlay()
	task.wait(2.5)
	resetCharacter()
	task.wait(2.5)
	selectIchigoPlay()
	task.wait(2.5)
	roundPaused = false
	roundPauseReason = nil
	roundResetting = false
	if mode == "alt" then loopAlt = true
	else loopMain = true end
	if mode == "main" then sendWebhook("Main Farm Started") end
	starting = false
end

local function getMainCFrame()
	local altPos = altCFrame.Position
	return CFrame.lookAt(altPos + Vector3.new(0,0,-1), altPos)
end
local function getFarmTargetCFrame()
	if loopAlt  then return altCFrame end
	if loopMain then return getMainCFrame() end
	return nil
end
local function isNearCFrame(cf, limit)
	local hrp = getHRP()
	if not hrp or not cf then return false end
	return (hrp.Position - cf.Position).Magnitude <= (limit or tpDistanceLimit)
end
local function isNearFarm() return isNearCFrame(altCFrame, tpDistanceLimit) end
local function tpToCFrame(cf)
	if not cf then return false end
	makeBase()
	local hrp = getHRP()
	if not hrp then return false end
	pcall(function()
		hrp.AssemblyLinearVelocity = Vector3.new(0,0,0)
		hrp.AssemblyAngularVelocity = Vector3.new(0,0,0)
		hrp.CFrame = cf
	end)
	return true
end
local function tpAndVerify(cf)
	if not tpToCFrame(cf) then return false end
	task.delay(tpWatchdogDelay, function()
		if gui and gui.Parent and not selectingTeam and (loopAlt or loopMain) and not isNearCFrame(cf, tpDistanceLimit) then
			tpToCFrame(cf)
		end
	end)
	return true
end
local function tpToPauseCFrame()
	local hrp = getHRP()
	if not hrp then return false end
	pcall(function()
		hrp.AssemblyLinearVelocity = Vector3.new(0,0,0)
		hrp.AssemblyAngularVelocity = Vector3.new(0,0,0)
		hrp.CFrame = pauseCFrame
	end)
	return true
end
tpPauseAndVerify = function()
	if not tpToPauseCFrame() then return false end
	task.delay(tpWatchdogDelay, function()
		if gui and gui.Parent and roundPaused and loopMain and not isNearCFrame(pauseCFrame, pauseTpDistanceLimit) then
			tpToPauseCFrame()
		end
	end)
	return true
end
local function pressKAfterTP()
	local char = LP.Character
	if not char then return end
	if pressedKChar == char then return end
	if not isNearFarm() then return end
	pressedKChar = char
	task.wait(0.35)
	if isNearFarm() then pressK() end
end
local function tpAlt()  tpAndVerify(altCFrame) end
local function tpMain() tpAndVerify(getMainCFrame()) end

-- ===== NEW UI =====
gui = Instance.new("ScreenGui")
gui.Name = "WWHub_GUI"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = game.CoreGui

-- Toggle button (ด้านซ้าย กด toggle panel)
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 42, 0, 42)
toggleBtn.Position = UDim2.new(0, 10, 0.5, -21)
toggleBtn.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
toggleBtn.Text = "⚡"
toggleBtn.TextSize = 20
toggleBtn.TextColor3 = Color3.fromRGB(150, 70, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.ZIndex = 10
toggleBtn.Parent = gui
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 10)
local tStroke = Instance.new("UIStroke", toggleBtn)
tStroke.Color = Color3.fromRGB(110, 40, 200)
tStroke.Thickness = 2

-- Panel หลัก
local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 280, 0, 320)
panel.Position = UDim2.new(0.5, -140, 0.5, -160)
panel.BackgroundColor3 = Color3.fromRGB(14, 14, 22)
panel.BorderSizePixel = 0
panel.Active = true
panel.Parent = gui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 14)
local pStroke = Instance.new("UIStroke", panel)
pStroke.Color = Color3.fromRGB(100, 35, 190)
pStroke.Thickness = 2

-- Drag (PC + Mobile)
local dragging, dragStart, startPos
panel.InputBegan:Connect(function(inp)
	if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
		dragging = true; dragStart = inp.Position; startPos = panel.Position
	end
end)
panel.InputEnded:Connect(function(inp)
	if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)
UIS.InputChanged:Connect(function(inp)
	if dragging and (inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch) then
		local d = inp.Position - dragStart
		panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
	end
end)

-- Header
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 48)
header.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
header.BorderSizePixel = 0
header.Parent = panel
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 14)

local titleLbl = Instance.new("TextLabel")
titleLbl.Size = UDim2.new(1, -50, 1, 0)
titleLbl.Position = UDim2.new(0, 12, 0, 0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "⚡ WW Hub"
titleLbl.TextColor3 = Color3.fromRGB(155, 80, 255)
titleLbl.TextSize = 19
titleLbl.Font = Enum.Font.GothamBold
titleLbl.TextXAlignment = Enum.TextXAlignment.Left
titleLbl.Parent = header

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -40, 0, 8)
closeBtn.BackgroundColor3 = Color3.fromRGB(190, 35, 55)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextSize = 15
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = header
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)

-- Status label
local statusLbl = Instance.new("TextLabel")
statusLbl.Size = UDim2.new(1, -20, 0, 20)
statusLbl.Position = UDim2.new(0, 10, 0, 54)
statusLbl.BackgroundTransparency = 1
statusLbl.Text = "📊 Idle"
statusLbl.TextColor3 = Color3.fromRGB(140, 140, 165)
statusLbl.TextSize = 13
statusLbl.Font = Enum.Font.Gotham
statusLbl.TextXAlignment = Enum.TextXAlignment.Left
statusLbl.Parent = panel

-- Divider
local div = Instance.new("Frame")
div.Size = UDim2.new(1, -20, 0, 1)
div.Position = UDim2.new(0, 10, 0, 78)
div.BackgroundColor3 = Color3.fromRGB(60, 35, 110)
div.BorderSizePixel = 0
div.Parent = panel

-- Helper สร้างปุ่ม
local function mkBtn(txt, col, yPos, w, xOff)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(w or 1, -20, 0, 52)
	b.Position = UDim2.new(0, xOff or 10, 0, yPos)
	b.BackgroundColor3 = col
	b.Text = txt
	b.TextColor3 = Color3.fromRGB(255,255,255)
	b.TextSize = 20
	b.Font = Enum.Font.GothamBold
	b.Parent = panel
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 10)
	return b
end

local altBtn  = mkBtn("💀 ALT",  Color3.fromRGB(50, 110, 240),  88, 0.47, 10)
local mainBtn = mkBtn("🎮 MAIN", Color3.fromRGB(50, 185, 90),   88, 0.47, nil)
mainBtn.Position = UDim2.new(0.53, -8, 0, 88)

local stopBtn = mkBtn("⏹ STOP", Color3.fromRGB(190, 50, 50), 150, 1, 10)

-- Point cap display
local capLbl = Instance.new("TextLabel")
capLbl.Size = UDim2.new(1, -20, 0, 20)
capLbl.Position = UDim2.new(0, 10, 0, 212)
capLbl.BackgroundTransparency = 1
capLbl.Text = "🎯 Point Cap: " .. pointCapLimit
capLbl.TextColor3 = Color3.fromRGB(255, 200, 60)
capLbl.TextSize = 13
capLbl.Font = Enum.Font.GothamBold
capLbl.TextXAlignment = Enum.TextXAlignment.Left
capLbl.Parent = panel

-- + / - cap buttons
local capMinus = Instance.new("TextButton")
capMinus.Size = UDim2.new(0, 36, 0, 28)
capMinus.Position = UDim2.new(1, -90, 0, 208)
capMinus.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
capMinus.Text = "−"
capMinus.TextColor3 = Color3.fromRGB(255,255,255)
capMinus.TextSize = 18
capMinus.Font = Enum.Font.GothamBold
capMinus.Parent = panel
Instance.new("UICorner", capMinus).CornerRadius = UDim.new(0, 7)

local capPlus = Instance.new("TextButton")
capPlus.Size = UDim2.new(0, 36, 0, 28)
capPlus.Position = UDim2.new(1, -50, 0, 208)
capPlus.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
capPlus.Text = "+"
capPlus.TextColor3 = Color3.fromRGB(255,255,255)
capPlus.TextSize = 18
capPlus.Font = Enum.Font.GothamBold
capPlus.Parent = panel
Instance.new("UICorner", capPlus).CornerRadius = UDim.new(0, 7)

-- Render toggle
local renderEnabled = true
local renderBtn = mkBtn("👁 Render: ON", Color3.fromRGB(0, 120, 210), 248, 1, 10)
renderBtn.TextSize = 15

-- Destroy button
local destroyBtn = Instance.new("TextButton")
destroyBtn.Size = UDim2.new(1, -20, 0, 30)
destroyBtn.Position = UDim2.new(0, 10, 1, -38)
destroyBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
destroyBtn.Text = "❌ Destroy GUI"
destroyBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
destroyBtn.TextSize = 13
destroyBtn.Font = Enum.Font.GothamBold
destroyBtn.Parent = panel
Instance.new("UICorner", destroyBtn).CornerRadius = UDim.new(0, 8)

-- Status updater
local function setStatus(txt) statusLbl.Text = "📊 " .. txt end

-- Toggle panel
local panelVisible = true
toggleBtn.MouseButton1Click:Connect(function()
	panelVisible = not panelVisible
	panel.Visible = panelVisible
end)

-- ALT
altBtn.MouseButton1Click:Connect(function()
	setStatus("Starting ALT...")
	task.spawn(function() startFarm("alt") end)
	altBtn.BackgroundColor3 = Color3.fromRGB(40, 190, 100)
	altBtn.Text = "⏸ ALT"
	mainBtn.BackgroundColor3 = Color3.fromRGB(50, 185, 90)
	mainBtn.Text = "🎮 MAIN"
end)

-- MAIN
mainBtn.MouseButton1Click:Connect(function()
	setStatus("Starting MAIN...")
	task.spawn(function() startFarm("main") end)
	mainBtn.BackgroundColor3 = Color3.fromRGB(40, 190, 100)
	mainBtn.Text = "⏸ MAIN"
	altBtn.BackgroundColor3 = Color3.fromRGB(50, 110, 240)
	altBtn.Text = "💀 ALT"
end)

-- STOP
stopBtn.MouseButton1Click:Connect(function()
	loopAlt  = false
	loopMain = false
	altBtn.BackgroundColor3  = Color3.fromRGB(50, 110, 240)
	altBtn.Text  = "💀 ALT"
	mainBtn.BackgroundColor3 = Color3.fromRGB(50, 185, 90)
	mainBtn.Text = "🎮 MAIN"
	setStatus("Idle")
end)

-- Point cap buttons
capMinus.MouseButton1Click:Connect(function()
	pointCapLimit = math.max(500, pointCapLimit - 500)
	capLbl.Text = "🎯 Point Cap: " .. pointCapLimit
end)
capPlus.MouseButton1Click:Connect(function()
	pointCapLimit = pointCapLimit + 500
	capLbl.Text = "🎯 Point Cap: " .. pointCapLimit
end)

-- Render toggle
renderBtn.MouseButton1Click:Connect(function()
	renderEnabled = not renderEnabled
	game:GetService("RunService"):Set3dRenderingEnabled(renderEnabled)
	renderBtn.Text = renderEnabled and "👁 Render: ON" or "👁 Render: OFF"
	renderBtn.BackgroundColor3 = renderEnabled and Color3.fromRGB(0, 120, 210) or Color3.fromRGB(170, 35, 35)
end)

-- Close (ซ่อน)
closeBtn.MouseButton1Click:Connect(function()
	panel.Visible = false
	panelVisible = false
end)

-- Destroy
destroyBtn.MouseButton1Click:Connect(function()
	loopAlt  = false
	loopMain = false
	gui:Destroy()
end)

-- Auto-start จาก _G config
task.spawn(function()
	task.wait(0.5)
	if _G.main then
		task.spawn(function() startFarm("main") end)
		mainBtn.BackgroundColor3 = Color3.fromRGB(40, 190, 100)
		mainBtn.Text = "⏸ MAIN"
	elseif _G.alt then
		task.spawn(function() startFarm("alt") end)
		altBtn.BackgroundColor3 = Color3.fromRGB(40, 190, 100)
		altBtn.Text = "⏸ ALT"
	end
end)

-- Status sync loop
task.spawn(function()
	while gui and gui.Parent do
		if starting then setStatus("Starting...")
		elseif roundPaused then setStatus("⏸ Paused: " .. (roundPauseReason or "?"))
		elseif timerTpDone then setStatus("⏱ Timer TP — waiting...")
		elseif pointsCapped then setStatus("Cap! pts=" .. getPoints() .. " t=" .. getTimerValue())
		elseif loopAlt  then setStatus("💀 ALT | t=" .. getTimerValue())
		elseif loopMain then setStatus("🎮 MAIN | t=" .. getTimerValue())
		else setStatus("Idle") end
		task.wait(0.5)
	end
end)

-- ===== Main Loops (ไม่แตะ logic เลย) =====
task.spawn(function()
	makeBase()
	while gui.Parent do
		local char = LP.Character
		if char and char.Parent and char ~= handledChar then
			afterCharacterLoaded(char)
		end
		if not roundPaused then pressKAfterTP() end
		task.wait(0.25)
	end
end)

task.spawn(function()
	while gui.Parent do
		local pad = getTeamPad()
		if pad and not selectingTeam and not roundPaused and (loopAlt or loopMain) then
			task.spawn(selectTeam)
		end
		if not selectingTeam and not jugResetting and not starting and not roundPaused and not timerTpDone then
			local target = getFarmTargetCFrame()
			if target and not isNearCFrame(target, tpDistanceLimit) then
				tpAndVerify(target)
			elseif loopAlt  then tpAlt()
			elseif loopMain then tpMain() end
		end
		task.wait(0.08)
	end
end)

task.spawn(function()
	while gui.Parent do
		if loopMain and not jugResetting and not starting and not selectingTeam and not roundPaused and not pointsCapped and not timerTpDone then
			fireM1()
		end
		task.wait(0.12)
	end
end)

task.spawn(function()
	while gui.Parent do
		if loopMain and not jugResetting and not starting and not selectingTeam and not roundPaused and not pointsCapped and not timerTpDone then
			fireSkills()
		end
		task.wait(0.8)
	end
end)

task.spawn(function()
	while gui.Parent do
		if (loopMain or loopAlt) and not starting and not roundPaused then
			local blockedMode = getBlockedFarmMode()
			if blockedMode then pauseFarmForRound(blockedMode) end
		end
		task.wait(0.5)
	end
end)

task.spawn(function()
	local modes = {"Kills Team", "Team Battle", "3 Teams", "Free For All", "Kills FFA"}
	while gui.Parent do
		if (loopAlt or loopMain) and not roundPaused then
			for _, mode in ipairs(modes) do
				fireInput("mode", mode)
				task.wait(0.3)
			end
		end
		task.wait(1)
	end
end)

task.spawn(function()
	while gui.Parent do
		task.wait(30)
		if loopMain and not roundPaused then
			sendWebhook("Main Farm Report")
		end
	end
end)
