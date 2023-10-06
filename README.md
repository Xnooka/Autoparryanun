-- สร้างตัวแปร workspace และ RunService
local wsp = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local isRunning = true

local function checkGameState()
    if isRunning and localplayer and workspace:FindFirstChild("Alive") then
        -- ตรวจสอบสถานะตัวละคร
        local character = localplayer.Character
        if not character or not character.Parent then
            -- ถ้าตัวละครไม่มีหรือไม่อยู่ใน workspace ให้หยุดการทำงาน
            isRunning = false
            print("Script stopped because character is not found.")
        end
    end
    
    if isRunning then
        -- ตรวจสอบสถานะเกมและตัวละครเรียกฟังก์ชันนี้อีกครั้งในอนาคต
        RunService.Heartbeat:Wait()
        checkGameState()
    end
end

-- เริ่มตรวจสอบสถานะเกมและตัวละคร
checkGameState()

-- สร้างตัวแปร Players และ Local
local Players = game:GetService("Players")
local local = Players.LocalPlayer

-- สร้างตัวแปร Camera และ Balls
local Camera = wsp.CurrentCamera
local Balls = wsp:WaitForChild("Balls")

-- สร้างตัวแปร Signal เพื่อใช้เก็บค่าตัวแปร Signal
getgenv().Signal = Signal or {}

-- ฟังก์ชัน PlayerPoints เพื่อหาตำแหน่งของผู้เล่นในจอ
function PlayerPoints()
    local tbl = {}
    for i, v in pairs(Players:GetPlayers()) do
        local UserId, HumanoidRootPart = tostring(v.UserId), v.Character and v.Character:FindFirstChild("HumanoidRootPart")
        if HumanoidRootPart and v == localplayer then
            warn(v)
            tbl[UserId] = Camera:WorldToScreenPoint(HumanoidRootPart.Position)
        end
    end
    
    print(unpack(tbl))
    table.foreach(tbl, print)
    return tbl
end

-- ฟังก์ชัน Parry เพื่อแยกยิงลูกบอล
function Parry()
    if Local.Character then
        local Remote = game:GetService("ReplicatedStorage"):WaitForChild("Remotes"):WaitForChild("ParryAttempt")
        local WorldToScreenPoint = Camera:WorldToScreenPoint(Local.Character.HumanoidRootPart.Position)
        local args = {
            [1] = 0.5,
            [2] = wsp.CurrentCamera.CFrame,
            [3] = PlayerPoints(),
            [4] = {
                [1] = WorldToScreenPoint.X,
                [2] = WorldToScreenPoint.Y
            }
        }
        
        warn("Players:", unpack(args[3]))
        Remote:FireServer(unpack(args))
    end
end

-- ฟังก์ชัน Anticipate เพื่อตรวจสอบการยิงลูกบอล
local Debounce, LastTime = false
function Anticipate(Time)
    if Debounce then return end
    
    if LastTime then
        local Sum = (Time - oldTime)
        if (Sum >= -25 and Sum <= 25) then
            print("Anticipated Time:", Sum, "Time:", Time, "LastTime:", LastTime)
            if Sum >= 25 or Sum <= -25 then
                return true
            end
        end
    end
    
    LastTime = Time
end

-- ฟังก์ชันคำนวณเวลาที่ลูกบอลใช้ในการเดินทางถึงเป้าหมาย
function calculateProjectileTime(initialPosition, targetPosition, initialVelocity)
    local distance = (targetPosition - initialPosition).Magnitude
    local time = distance / initialVelocity.Magnitude
    return time
end

-- ฟังก์ชันคำนวณระยะห่างระหว่างลูกบอลและวัตถุ
function calculateDistance(projectilePosition, objectPosition)
    return math.abs((projectilePosition - objectPosition).Magnitude)
end

-- ฟังก์ชันตรวจสอบว่าวัตถุสามารถป้องกัน (Parry) ลูกบอลได้หรือไม่
function canObjectParry(projectilePosition, objectPosition, projectileVelocity, objectVelocity)
    local timeToIntercept = calculateProjectileTime(projectilePosition, objectPosition, projectileVelocity)
    local distanceToIntercept = calculateDistance(projectilePosition + projectileVelocity * timeToIntercept, objectPosition + objectVelocity * timeToIntercept)
    local Anticipate = Anticipate(timeToIntercept)
    
    print("CanParry:", distanceToIntercept, timeToIntercept, Anticipate)
    
    local conditions = {
        (Anticipate and distanceToIntercept <= 75);
        (distanceToIntercept >= 35 and distanceToIntercept <= 50 and timeToIntercept <= 0.6);
        (distanceToIntercept >= 35 and distanceToIntercept <= 50 and timeToIntercept <= 0.5);
        (distanceToIntercept >= 35 and distanceToIntercept <= 50 and timeToIntercept <= 0.4);
        (distanceToIntercept >= 20 and distanceToIntercept <= 30 and timeToIntercept <= 0.4 and timeToIntercept <= 0.55);
        (distanceToIntercept >= 50 and distanceToIntercept <= 75 and timeToIntercept >= 0.6 and timeToIntercept <= 0.75);
        (distanceToIntercept <= 35 and timeToIntercept <= 0.5);
        (distanceToIntercept >= 20 and timeToInterept <= 0.4);
        (distanceToIntercept <= 12.5 and timeToIntercept >= 0.5 and timeToIntercept <= 0.75);
        (distanceToIntercept >= 5 and timeToIntercept <= 1 and timeToIntercept <= 0.85);
        (distanceToIntercept >= 0.025 and timeToIntercept <= 0.50);
        (distanceToIntercept <= 0.025 and timeToIntercept <= 0.75);
        (distanceToIntercept <= 0.025 and timeToIntercept <= 0.25);
        (distanceToIntercept <= 0.025 and timeToIntercept <= 0.10);
        (distanceToIntercept >= 75 and distanceToIntercept <= 100 and timeToIntercept <= 0.5);
        (distanceToIntercept >= 75 and distanceToIntercept <= 100 and timeToIntercept <= 0.4);
        (distanceToIntercept >= 75 and distanceToIntercept <= 100 and timeToIntercept <= 0.3);
        (distanceToIntercept >= 75 and distanceToIntercept <= 100 and timeToIntercept <= 0.2);
    }
    
    local r
    for i, v in pairs(conditions) do
        if v == true then
            warn(i,
            
