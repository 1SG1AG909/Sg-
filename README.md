# ㅋ-
 -- =====================
-- Load Config (FILE)
-- =====================
local CONFIG = loadstring([[
local CONFIG = {}

CONFIG.TARGET_RANGE = 50
CONFIG.ATTACH_DISTANCE = 2.5
CONFIG.SKY_HEIGHT = 120
CONFIG.FALL_SPEED = 400
CONFIG.FOLLOW_TARGET = true
CONFIG.RETARGET_IF_FAR = true
CONFIG.MAX_TARGET_DISTANCE = 60
CONFIG.RESET_DELAY = 3
CONFIG.START_TEXT = "💣 START"
CONFIG.STOP_TEXT  = "🛑 STOP"

return CONFIG
]])()

-- =====================
-- Services
-- =====================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local lp = Players.LocalPlayer
local gui = Instance.new("ScreenGui", lp.PlayerGui)
gui.ResetOnSpawn = false

-- =====================
-- Button (Mobile OK)
-- =====================
local btn = Instance.new("TextButton", gui)
btn.Size = UDim2.fromScale(0.22, 0.08)
btn.Position = UDim2.fromScale(0.03, 0.45)
btn.TextScaled = true
btn.Text = CONFIG.START_TEXT
btn.BackgroundColor3 = Color3.fromRGB(25,25,25)
btn.TextColor3 = Color3.fromRGB(255,255,255)

local running = false
btn.MouseButton1Click:Connect(function()
	running = not running
	btn.Text = running and CONFIG.STOP_TEXT or CONFIG.START_TEXT
end)

-- =====================
-- Target finder
-- =====================
local function getTarget()
	local char = lp.Character
	if not char then return nil end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return nil end

	local list = {}
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local d = (hrp.Position - p.Character.HumanoidRootPart.Position).Magnitude
			if d <= CONFIG.TARGET_RANGE then
				table.insert(list, p)
			end
		end
	end
	return #list > 0 and list[math.random(#list)] or nil
end

-- =====================
-- State
-- =====================
local target = nil
local falling = false
local startTime = 0

-- =====================
-- Main loop (NO SHAKE)
-- =====================
RunService.Stepped:Connect(function(_, dt)
	if not running then return end

	local char = lp.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	-- 흔들림 완전 제거
	hrp.AssemblyLinearVelocity = Vector3.zero
	hrp.AssemblyAngularVelocity = Vector3.zero

	if not target or not target.Character then
		target = getTarget()
		falling = false
		return
	end

	local th = target.Character:FindFirstChild("HumanoidRootPart")
	if not th then
		target = nil
		return
	end

	-- 거리 멀어지면 타겟 변경
	if CONFIG.RETARGET_IF_FAR then
		if (hrp.Position - th.Position).Magnitude > CONFIG.MAX_TARGET_DISTANCE then
			target = getTarget()
			falling = false
			return
		end
	end

	-- 시작 위치 (하늘)
	if not falling then
		hrp.CFrame = CFrame.new(
			th.Position.X,
			th.Position.Y + CONFIG.SKY_HEIGHT,
			th.Position.Z
		)
		falling = true
	end

	-- 🚀 포탄처럼 수직 낙하 (완전 밀착)
	local newY = hrp.Position.Y - CONFIG.FALL_SPEED * dt
	local targetY = th.Position.Y

	if newY <= targetY + CONFIG.ATTACH_DISTANCE then
		-- 완전히 붙음
		hrp.CFrame = CFrame.new(th.Position)
		startTime += dt
		if startTime >= CONFIG.RESET_DELAY then
			target = nil
			falling = false
			startTime = 0
		end
	else
		hrp.CFrame = CFrame.new(th.Position.X, newY, th.Position.Z)
	end
end)
