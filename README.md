-- [[
	WARNING: Executor-Ready Version. (Fixed & Optimized)
]]

-- 1. Đợi game tải xong hoàn toàn trước khi chạy script
if not game:IsLoaded() then
	game.Loaded:Wait()
end

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

-- FIX 1: Vòng lặp chờ chắc chắn LocalPlayer đã tồn tại rồi mới gán biến
local CoolPlr = Players.LocalPlayer
while not CoolPlr do
	task.wait()
	CoolPlr = Players.LocalPlayer
end

-- 2. Hệ thống tìm Parent an toàn để giấu GUI khỏi Anti-cheat
local function getSecureGuiParent()
	-- FIX 2: Kiểm tra nghiêm ngặt xem gethui() có thực sự trả về một Object hợp lệ không
	if gethui then
		local hui = gethui()
		if hui then return hui end
	end
	
	-- Ưu tiên 2: Giấu vào CoreGui nếu pcall thành công
	local success, core = pcall(function() return game:GetService("CoreGui") end)
	if success and core then return core end
	
	-- Ưu tiên 3: Bắt buộc dùng PlayerGui nếu các cách trên thất bại
	return CoolPlr:WaitForChild("PlayerGui")
end

-- Khởi tạo GUI
local Gui = Instance.new("ScreenGui")
Gui.Name = "cOOlGui_Protected"
Gui.IgnoreGuiInset = true
Gui.ResetOnSpawn = false
Gui.DisplayOrder = 100000

local CoolButton = Instance.new("TextButton")
CoolButton.Name = "CoolGuiButtonOpen"
CoolButton.Text = "cOOlGui"
CoolButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
CoolButton.Font = Enum.Font.Arcade
CoolButton.TextSize = 20
CoolButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CoolButton.RichText = true

-- FIX 3: Chuyển sang kích thước chuẩn (Offset) để tránh bug co giãn về 0 pixel
CoolButton.SizeConstraint = Enum.SizeConstraint.RelativeXY
CoolButton.Size = UDim2.new(0, 130, 0, 45) -- Kích thước rộng 130px, cao 45px cố định cực đẹp
CoolButton.Position = UDim2.new(0.02, 0, 0.45, 0)
CoolButton.Parent = Gui

local uistroke = Instance.new("UIStroke")
uistroke.Thickness = 3
uistroke.Color = Color3.fromRGB(255, 0, 0)
uistroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
uistroke.Parent = CoolButton

-- Gắn GUI vào vị trí an toàn đã tìm được
Gui.Parent = getSecureGuiParent()

-- Các biến trạng thái
local Turn = false
local LivingFolder = Workspace:WaitForChild("Living", 10) 

-- Hàm tìm kẻ địch ở khoảng cách gần nhất
local function getClosestEnemy()
	local myChar = CoolPlr.Character
	local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
	if not myHRP or not LivingFolder then return nil end

	local closestEnemy = nil
	local shortestDistance = math.huge

	for _, model in LivingFolder:GetChildren() do
		if model.Name ~= CoolPlr.Name and model:FindFirstChild("Humanoid") and model:FindFirstChild("AI") and model:FindFirstChild("HumanoidRootPart") then
			local enemyHumanoid = model.Humanoid
			if enemyHumanoid.Health > 0 then
				local enemyHRP = model.HumanoidRootPart
				local distance = (myHRP.Position - enemyHRP.Position).Magnitude
				
				if distance < shortestDistance then
					shortestDistance = distance
					closestEnemy = model
				end
			end
		end
	end

	return closestEnemy
end

-- Vòng lặp chính xử lý Teleport & Tấn công
local function mainLoop()
	while Turn do
		if not LivingFolder or not LivingFolder:FindFirstChild(CoolPlr.Name) then
			CoolButton.Text = "Canceled"
			task.wait(0.5)
			continue
		end

		CoolButton.Text = "Searching..."
		local target = getClosestEnemy()

		if not target then
			CoolButton.Text = "No Target"
			task.wait(0.5)
			continue
		end

		CoolButton.Text = "LOCKED"
		local targetHumanoid = target.Humanoid
		local targetHRP = target.HumanoidRootPart

		while Turn and targetHumanoid and targetHumanoid.Health > 0 and targetHRP do
			local myChar = CoolPlr.Character
			local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")

			if myHRP then
				CoolButton.Text = "KILL: " .. target.Name
				-- Đã thêm Offset nhẹ (lên trên 2 studs) để nhân vật của bạn không bị kẹt cứng vào giữa người quái
				myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 2, 0)
			end
			
			RunService.Heartbeat:Wait()
		end
		
		task.wait(0.1)
	end
	CoolButton.Text = "cOOlGui"
end

-- Lắng nghe sự kiện Bật/Tắt bằng chuột / chạm màn hình
CoolButton.MouseButton1Click:Connect(function()
	Turn = not Turn
	if Turn then
		task.spawn(mainLoop)
	end
end)
