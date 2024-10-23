local player = game.Players.LocalPlayer
local mouse = game:GetService("UserInputService")
local camera = workspace.CurrentCamera
local playerTeamColor = player.TeamColor
local enemyTeamColors = {BrickColor.new("Really red"), BrickColor.new("Dark red"), BrickColor.new("Bright red"), BrickColor.new("Medium red"), BrickColor.new("Reddish brown"), BrickColor.new("Red"), BrickColor.new("Pastel red"), BrickColor.new("Dark stone grey"), BrickColor.new("Medium stone grey"), BrickColor.new("Stone grey"), BrickColor.new("Cyan")}
local enemyTag = "HumanoidRootPart"
local esp = false
local aimbot = false

function getEnemyPlayers()
    local enemyTable = {}
    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= player and v.TeamColor ~= playerTeamColor then
            enemyTable[#enemyTable + 1] = v
        end
    end
    return enemyTable
end

function drawESP()
    for _, enemy in ipairs(getEnemyPlayers()) do
        if enemy.Character then
            local enemyHead = enemy:FindFirstChild(enemyTag)
            if enemyHead then
                local headPosition = enemyHead.Position
                local distance = (player.Character.HumanoidRootPart.Position - headPosition).Magnitude
                if distance <= 100 then
                    local screenPosition, onScreen = camera:WorldToViewportPoint(headPosition)
                    if onScreen then
                        drawRect(screenPosition.X, screenPosition.Y, 3, 3, BrickColor.new("Institutional white"))
                    end
                end
            end
        end
    end
end

function aimbotAction()
    local enemies = getEnemyPlayers()
    if #enemies > 0 then
        for _, enemy in ipairs(enemies) do
            local enemyHead = enemy:FindFirstChild(enemyTag)
            if enemyHead and enemy.Character and enemy.Character:FindFirstChild("Humanoid") and enemy.Character.Humanoid.Health > 0 then
                local lookVector = (player.Character.HumanoidRootPart.Position - enemyHead.Position).Unit
                player.Character.HumanoidRootPart.CFrame = CFrame.new(player.Character.HumanoidRootPart.Position, enemyHead.Position + lookVector * 10)
                player.Character.HumanoidRootPart.LookAt(enemyHead)
                if player:GetMouseButtonpressed(1) then
                    player.Character:BreakJoints()
                    local weapon = player.Character:FindFirstChild("RightArm") or player.Character:FindFirstChild("LeftArm")
                    if weapon and weapon:FindFirstChild("Grip") then
                        for _, child in ipairs(weapon:GetChildren()) do
                            if child:IsA("MeshPart") and child.Name ~= "Handle" then
                                local hit, hitPos = workspace:FindPartOnRay(Ray.new(player.Character.HumanoidRootPart.Position, enemyHead.Position), enemy)
                                if hit then
                                    local distance = (hitPos - player.Character.HumanoidRootPart.Position).Magnitude
                                    if distance <= 100 then
                                        child:FireServer()
                                        wait(0.01)
                                    else
                                        player.Character.Humanoid:ChangeState(11)
                                    end
                                else
                                    player.Character.Humanoid:ChangeState(11)
                                end
                            end
                        end
                    end
                else
                    player.Character.Humanoid:ChangeState(11)
                end
            end
        end
    end
end

function main()
    while wait() do
        if esp then
            drawESP()
        end
        if aimbot then
            aimbotAction()
        end
    end
end

mouse.Button1Down:connect(function()
    if aimbot then
        aimbotAction()
    end
end)

game:GetService("UserInputService").InputBegan:connect(function(input)
    if input.KeyCode == Enum.KeyCode.E then
        esp = not esp
        if esp then
            print("ESP Ativado")
        else
            print("ESP Desativado")
        end
    elseif input.KeyCode == Enum.KeyCode.R then
        aimbot = not aimbot
        if aimbot then
            print("Aimbot Ativado")
        else
            print("Aimbot Desativado")
        end
    end
end)

main()
