-- HDZV4 - Auto Up V4 Complete Final
-- Tổng hợp mọi yêu cầu:
-- 1. Chỉ hoạt động ở Sea 3 khi trăng tròn
-- 2. Tự động bay đến cửa tộc tương ứng
-- 3. Khóa người chơi khác đang ở gần cửa up V4
-- 4. Tự động bật V3 cho người dùng
-- 5. Hoàn thành thử thách theo từng tộc (Human, Fish, Mink, Sky, Cyborg, Demon)
-- 6. Đánh melee cực nhanh (0.015s/đòn) người chơi khác đang up V4
-- 7. Hồi máu đến 100% khi dưới 30% (có dùng item hồi)
-- 8. Tự động tắt khi hoàn thành V4
-- 9. Hiển thị trạng thái và FPS

getgenv().HDZV4 = true
getgenv().FlySpeed = 400
getgenv().MeleeSpeed = 0.015
getgenv().HealThreshold = 0.3

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local TweenService = game:GetService("TweenService")
local CommF_ = ReplicatedStorage:WaitForChild("Remotes",5):WaitForChild("CommF_",5)

-- ===== KIỂM TRA ĐIỀU KIỆN =====
local function IsSea3()
    return Workspace:FindFirstChild("Sea3") ~= nil or (Player.PlayerGui:FindFirstChild("Sea3UI") ~= nil)
end

if not IsSea3() then
    print("[HDZV4] ❌ Không phải Sea 3 - Tắt script")
    return
end

local function IsFullMoon()
    return (Lighting.ClockTime >= 20 or Lighting.ClockTime <= 4)
end

if not IsFullMoon() then
    print("[HDZV4] 🌙 Chờ trăng tròn...")
    repeat task.wait(5) until IsFullMoon()
end

-- ===== UI =====
local screen = Instance.new("ScreenGui", game.CoreGui)
screen.Name = "HDZV4_UI"
local statusLabel = Instance.new("TextLabel", screen)
statusLabel.Size = UDim2.new(0, 550, 0, 50)
statusLabel.Position = UDim2.new(0.5, -275, 0.05, 0)
statusLabel.BackgroundTransparency = 0.3
statusLabel.BackgroundColor3 = Color3.fromRGB(0,0,0)
statusLabel.TextColor3 = Color3.fromRGB(0,255,255)
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextScaled = true
statusLabel.Text = "🚀 HDZV4 - Khởi tạo..."
Instance.new("UICorner", statusLabel).CornerRadius = UDim.new(0,10)

local function UpdateStatus(t)
    pcall(function() statusLabel.Text = t end)
    print("[HDZV4] " .. t)
end

-- ===== HÀM BAY =====
local function FlyTo(targetCFrame, stopDist)
    pcall(function()
        local ch = Player.Character
        if not ch then return end
        local hrp = ch:FindFirstChild("HumanoidRootPart")
        if not hrp then return end

        for _,c in pairs(hrp:GetChildren()) do
            if c.Name == "_BV" or c.Name == "_BG" then c:Destroy() end
        end

        local BV = Instance.new("BodyVelocity", hrp)
        BV.Name = "_BV"
        BV.MaxForce = Vector3.new(1e9,1e9,1e9)
        BV.P = 2e4

        local BG = Instance.new("BodyGyro", hrp)
        BG.Name = "_BG"
        BG.MaxTorque = Vector3.new(1e9,1e9,1e9)
        BG.P = 2e4

        while hrp and hrp.Parent and (hrp.Position - targetCFrame.Position).Magnitude > stopDist do
            local dir = (targetCFrame.Position - hrp.Position)
            local dist = dir.Magnitude
            local speed = getgenv().FlySpeed
            if dist < 30 then speed = math.clamp(dist*15, 50, getgenv().FlySpeed) end
            BV.Velocity = dir.Unit * speed
            BG.CFrame = CFrame.new(hrp.Position, targetCFrame.Position)
            task.wait()
        end
        BV:Destroy()
        BG:Destroy()
    end)
end

-- ===== HỒI MÁU ĐẾN 100% =====
local function HealToFull()
    pcall(function()
        local ch = Player.Character
        if not ch then return end
        local humanoid = ch:FindFirstChild("Humanoid")
        if not humanoid then return end

        local maxHealth = humanoid.MaxHealth
        local currentHealth = humanoid.Health
        local ratio = currentHealth / maxHealth

        if ratio <= getgenv().HealThreshold then
            UpdateStatus("💉 Máu thấp (" .. math.floor(ratio*100) .. "%) - Hồi phục...")
            local hrp = ch:FindFirstChild("HumanoidRootPart")
            if hrp then
                FlyTo(hrp.CFrame + Vector3.new(0, 250, 0), 5)
            end

            -- Dùng item hồi máu nếu có
            local function UseHealItem()
                local inventory = Player:FindFirstChild("Backpack") or Player:FindFirstChild("StarterGear")
                if inventory then
                    for _, item in pairs(inventory:GetChildren()) do
                        if item:IsA("Tool") and item:FindFirstChild("Remote") then
                            if string.find(item.Name, "Heal") or string.find(item.Name, "Potion") or string.find(item.Name, "Fruit") then
                                pcall(function()
                                    Player.Character.Humanoid:EquipTool(item)
                                    task.wait(0.3)
                                    item.Remote:FireServer("Use")
                                end)
                                break
                            end
                        end
                    end
                end
            end
            UseHealItem()

            while humanoid.Health < maxHealth do
                UpdateStatus("💚 Hồi máu: " .. math.floor((humanoid.Health/maxHealth)*100) .. "%")
                task.wait(0.3)
                if humanoid.Health / maxHealth < 0.5 and tick() % 10 < 0.5 then
                    UseHealItem()
                end
            end
            UpdateStatus("✅ Đã hồi đầy máu")
            task.wait(0.5)
        end
    end)
end

-- ===== KHÓA NGƯỜI CHƠI GẦN CỬA =====
local function LockPlayersNearDoor(doorPos, radius)
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= Player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (plr.Character.HumanoidRootPart.Position - doorPos.Position).Magnitude
            if dist <= radius then
                plr.Character.HumanoidRootPart.Velocity = Vector3.new(0,0,0)
                plr.Character.HumanoidRootPart.CFrame = CFrame.new(doorPos.Position + Vector3.new(math.random(-3,3), 2, math.random(-3,3)))
                if plr.Character:FindFirstChild("RaceV3") and plr.Character.RaceV3.Value == true then
                    CommF_:InvokeServer("RaceV3", "Activate")
                end
            end
        end
    end
end

-- ===== TỰ ĐỘNG BẬT V3 =====
local function AutoActivateV3()
    if not Player.Character:FindFirstChild("RaceV3") or Player.Character.RaceV3.Value == false then
        CommF_:InvokeServer("RaceV3", "Activate")
        UpdateStatus("⚡ Đã kích V3")
    end
end

-- ===== SPAM CHIÊU (TỘC CÁ) =====
local function SpamWeaponSkills()
    for _, tool in pairs(Player.Backpack:GetChildren()) do
        if tool:IsA("Tool") and tool:FindFirstChild("Remote") then
            for i = 1, 15 do
                tool.Remote:FireServer("Skill", i)
                task.wait(0.06)
            end
        end
    end
end

-- ===== ĐÁNH MELEE CỰC NHANH (CHỈ NGƯỜI UP V4) =====
local function FastMeleeAttackPlayer(targetPlayer, doorPos)
    if not targetPlayer or not targetPlayer.Character then return end
    if targetPlayer == Player then return end

    -- Chỉ đánh người đang ở gần cửa (đang up V4)
    local targetHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not targetHRP then return end
    local dist = (targetHRP.Position - doorPos.Position).Magnitude
    if dist > 70 then return end

    pcall(function()
        local ch = Player.Character
        if not ch then return end
        local hrp = ch:FindFirstChild("HumanoidRootPart")
        if not hrp then return end

        -- Trang bị melee
        local meleeTool = nil
        for _, tool in pairs(Player.Backpack:GetChildren()) do
            if tool.ToolTip == "Melee" then
                meleeTool = tool
                break
            end
        end
        if meleeTool then
            ch.Humanoid:EquipTool(meleeTool)
        end

        FlyTo(targetHRP.CFrame * CFrame.new(0, 4, 0), 2)

        local attackCount = 0
        while targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Humanoid")
              and targetPlayer.Character.Humanoid.Health > 0 and getgenv().HDZV4 do

            if attackCount % 10 == 0 then
                HealToFull()
            end

            VirtualUser:CaptureController()
            VirtualUser:Button1Down(Vector2.new(0,0))
            task.wait(getgenv().MeleeSpeed)

            if attackCount % 3 == 0 then
                hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(30), 0)
            end

            attackCount = attackCount + 1

            local newDist = (hrp.Position - doorPos.Position).Magnitude
            if newDist > 100 then break end
        end
    end)
end

-- ===== LẤY THÔNG TIN TỘC VÀ VỊ TRÍ CỬA =====
local Race = CommF_:InvokeServer("GetRace")
local doorPositions = {
    Human = CFrame.new(1000, 50, 2000),
    Fish = CFrame.new(1200, 50, 2200),
    Mink = CFrame.new(800, 50, 1800),
    Sky = CFrame.new(900, 100, 1900),
    Cyborg = CFrame.new(1100, 50, 2100),
    Demon = CFrame.new(1300, 50, 2300)
}
local doorPos = doorPositions[Race] or CFrame.new(1000, 50, 2000)
local roofPos = doorPos + Vector3.new(0, 60, 0)

-- ===== THỬ THÁCH THEO TỘC =====
local function CompleteChallenge()
    UpdateStatus("🎯 Thử thách tộc " .. Race)

    if Race == "Human" then
        -- Đánh 50 quái
        local count = 0
        while count < 50 do
            HealToFull()
            for _, mob in pairs(Workspace.Enemies:GetChildren()) do
                if mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                    -- Dùng hàm đánh melee cho quái
                    local fakePlayer = {Character = mob}
                    FastMeleeAttackPlayer(fakePlayer, doorPos)
                    count = count + 1
                    UpdateStatus("🧑 Đánh quái: " .. count .. "/50")
                    if count >= 50 then break end
                end
            end
            task.wait(1)
        end

    elseif Race == "Fish" then
        -- Spam chiêu 100 lần
        for i = 1, 100 do
            HealToFull()
            SpamWeaponSkills()
            UpdateStatus("🐟 Spam chiêu: " .. i .. "/100")
            task.wait(0.5)
        end

    elseif Race == "Mink" then
        -- Chạy tốc độ 30 lần
        local hrp = Player.Character.HumanoidRootPart
        for i = 1, 30 do
            HealToFull()
            hrp.Velocity = Vector3.new(1000, 0, 1000)
            UpdateStatus("⚡ Chạy: " .. i .. "/30")
            task.wait(0.3)
            hrp.Velocity = Vector3.new(-1000, 0, -1000)
            task.wait(0.3)
        end

    elseif Race == "Sky" then
        -- Bay cao 20 lần
        for i = 1, 20 do
            HealToFull()
            local hrp = Player.Character.HumanoidRootPart
            hrp.Velocity = Vector3.new(0, 500, 0)
            UpdateStatus("☁️ Bay cao: " .. i .. "/20")
            task.wait(0.5)
            hrp.Velocity = Vector3.new(0, -100, 0)
            task.wait(0.3)
        end

    elseif Race == "Cyborg" then
        -- Chỉ cần bay lên nóc cửa
        HealToFull()
        UpdateStatus("🤖 Bay lên nóc cửa...")
        FlyTo(roofPos, 3)
        task.wait(5)
        FlyTo(doorPos, 3)

    elseif Race == "Demon" then
        -- Đánh 20 người chơi khác
        local count = 0
        while count < 20 do
            HealToFull()
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= Player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    FastMeleeAttackPlayer(plr, doorPos)
                    count = count + 1
                    UpdateStatus("👿 Đánh người: " .. count .. "/20")
                    if count >= 20 then break end
                end
            end
            task.wait(1)
        end
    end

    -- Sau khi hoàn thành thử thách, bay về cửa và gọi remote
    FlyTo(doorPos, 5)
    CommF_:InvokeServer("CompleteV4Challenge")
    UpdateStatus("✅ Hoàn thành thử thách tộc " .. Race)
end

-- ===== MAIN LOOP =====
spawn(function()
    UpdateStatus("📍 Bay đến cửa tộc " .. Race)
    FlyTo(doorPos, 5)

    while getgenv().HDZV4 and IsSea3() do
        pcall(function()
            -- Kiểm tra và hồi máu
            HealToFull()
            
            -- Tự động bật V3
            AutoActivateV3()
            
            -- Khóa người chơi khác ở cửa
            LockPlayersNearDoor(doorPos, 60)

            -- Đánh tất cả người chơi khác đang ở cửa
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= Player and plr.Character then
                    FastMeleeAttackPlayer(plr, doorPos)
                end
            end

            -- Kiểm tra tiến độ V4
            local progress = CommF_:InvokeServer("CheckV4Progress")
            if progress and progress >= 100 then
                UpdateStatus("🏆 HOÀN THÀNH V4! Tắt script.")
                getgenv().HDZV4 = false
                break
            end

            -- Nếu chưa hoàn thành, chạy thử thách
            if not progress or progress < 100 then
                CompleteChallenge()
            end

            task.wait(2)
        end)
    end
    UpdateStatus("🛑 HDZV4 đã dừng")
end)

-- ===== FPS UI =====
local fpsGui = Instance.new("ScreenGui", game.CoreGui)
fpsGui.Name = "HDZV4_FPS"
local fpsLabel = Instance.new("TextLabel", fpsGui)
fpsLabel.Size = UDim2.new(0, 100, 0, 30)
fpsLabel.Position = UDim2.new(0, 10, 0, 60)
fpsLabel.BackgroundTransparency = 1
fpsLabel.TextColor3 = Color3.fromRGB(255,255,255)
fpsLabel.Font = Enum.Font.FredokaOne
fpsLabel.TextScaled = true
fpsLabel.Text = "FPS: 0"

task.spawn(function()
    local hue = 0
    while true do
        hue = (hue + 0.005) % 1
        fpsLabel.TextColor3 = Color3.fromHSV(hue,1,1)
        RunService.RenderStepped:Wait()
    end
end)

local fc, lu = 0, tick()
RunService.RenderStepped:Connect(function()
    fc += 1
    local now = tick()
    if now - lu >= 1 then
        fpsLabel.Text = "FPS: " .. math.floor(fc/(now-lu))
        fc = 0
        lu = now
    end
end)

print("[HDZV4] ✅ Script đã khởi chạy thành công!")
