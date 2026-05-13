--[[
    🍎 APPLE Hub | Beta v1.7
    - Sistema de Seleção Inicial
    - Carregamento de Listas Externas (Web)
    - Auto Appraiser Automatizado (Moosewood)
    - Auto Equip Rod Inteligente
]]

-- CONFIGURAÇÃO DE LINKS (Substitua pelos seus links RAW do Pastebin)
local RODS_DATA_URL = "https://pastebin.com/raw/hXj02GTr"
local MUTATIONS_DATA_URL = "https://pastebin.com/raw/7NeWF36Y"

-- Variáveis Globais
local WalkSpeedValue = 16
local AutoEquipRod = false
local SkipCutscenes = false
local InfiniteJumpEnabled = false

-- Variáveis Auto Appraiser
local AutoAppraiserEnabled = false
local SelectedFish = ""
local SelectedMutation = "None"
local StopAtShiny = false
local StopAtSparkling = false

-- Tabelas de Dados
local AllRods = {}
local AllMutations = {"None"}

-- Carregamento de Dados Externos
local function FetchData(url, targetTable)
    local success, content = pcall(function() return game:HttpGet(url) end)
    if success and content then
        for item in string.gmatch(content, "[^,]+") do
            table.insert(targetTable, string.gsub(item, "^%s*(.-)%s*$", "%1"))
        end
    end
end

FetchData(RODS_DATA_URL, AllRods)
FetchData(MUTATIONS_DATA_URL, AllMutations)

-- Funções Utilitárias
local function GetInventoryFish()
    local fish = {}
    local player = game.Players.LocalPlayer
    local backpack = player:FindFirstChild("Backpack")
    local char = player.Character
    
    local function scan(container)
        if not container then return end
        for _, item in pairs(container:GetChildren()) do
            if item:IsA("Tool") and not item.Name:lower():find("rod") then
                table.insert(fish, item.Name)
            end
        end
    end
    
    scan(backpack)
    scan(char)
    return (#fish > 0) and fish or {"Nenhum peixe encontrado"}
end

local function IsARod(tool)
    local name = tool.Name:lower()
    if name:find("rod") then return true end
    -- Check if it's in the Rods list from web
    for _, rodName in pairs(AllRods) do
        if name == rodName:lower() then return true end
    end
    return false
end

-- UI DE SELEÇÃO INICIAL (SCREEN GUI)
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 300, 0, 220)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -110)
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Instance.new("UICorner", MainFrame)

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.Text = "🍎 APPLE HUB - SELECIONE"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16

local function CreateBtn(name, pos, color, callback)
    local btn = Instance.new("TextButton", MainFrame)
    btn.Name = name
    btn.Size = UDim2.new(0.8, 0, 0, 45)
    btn.Position = pos
    btn.BackgroundColor3 = color
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.GothamSemibold
    Instance.new("UICorner", btn)
    btn.MouseButton1Click:Connect(callback)
end

-- CARREGAMENTO DO RAYFIELD
local function StartHub(mode)
    ScreenGui:Destroy()
    local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
    
    local Window = Rayfield:CreateWindow({
        Name = "🍎 APPLE Hub | " .. mode,
        LoadingTitle = "Carregando Scripts...",
        LoadingSubtitle = "by Gemini (Beta)",
        ConfigurationSaving = { Enabled = false },
        KeySystem = false
    })

    local MainTab = Window:CreateTab("Main", 4483362458)
    
    if mode == "Fisch" then
        -- CATEGORIA MAIN
        MainTab:CreateSection("Main")
        MainTab:CreateToggle({
            Name = "Auto Equip Rod",
            CurrentValue = false,
            Callback = function(V) AutoEquipRod = V end,
        })
        MainTab:CreateToggle({
            Name = "Skip Cutscenes",
            CurrentValue = false,
            Callback = function(V) SkipCutscenes = V end,
        })

        -- CATEGORIA HELPER FEATURE (BETA)
        MainTab:CreateSection("-- Helper Feature (BETA)")
        MainTab:CreateLabel("Funções experimentais e correções.")

        -- ABA AUTO (NOVA)
        local AutoTab = Window:CreateTab("Auto", 4483362458)
        AutoTab:CreateSection("-- Auto Appraiser (BETA)")
        
        local AppToggle = AutoTab:CreateToggle({
            Name = "Auto Appraiser [Enabled]",
            CurrentValue = false,
            Callback = function(Value) AutoAppraiserEnabled = Value end,
        })

        local FishDrop = AutoTab:CreateDropdown({
            Name = "Fish (Inventário):",
            Options = GetInventoryFish(),
            CurrentOption = "",
            Callback = function(Option) SelectedFish = Option end,
        })

        AutoTab:CreateButton({
            Name = "Atualizar Inventário",
            Callback = function() FishDrop:Refresh(GetInventoryFish()) end,
        })

        AutoTab:CreateDropdown({
            Name = "Mutation:",
            Options = AllMutations,
            CurrentOption = "None",
            Callback = function(Option) SelectedMutation = Option end,
        })

        AutoTab:CreateToggle({Name = "Until Shiny", CurrentValue = false, Callback = function(V) StopAtShiny = V end})
        AutoTab:CreateToggle({Name = "Until Sparkling", CurrentValue = false, Callback = function(V) StopAtSparkling = V end})
    end

    -- ABA MOVIMENTAÇÃO
    local MoveTab = Window:CreateTab("Movimentação", 4483362458)
    MoveTab:CreateSlider({
        Name = "Velocidade",
        Range = {16, 10000},
        Increment = 100,
        CurrentValue = 16,
        Callback = function(V) WalkSpeedValue = V end,
    })
    MoveTab:CreateToggle({
        Name = "Infinite Jump",
        CurrentValue = false,
        Callback = function(V) InfiniteJumpEnabled = V end,
    })

    -- LOGICA DO AUTO APPRAISER
    task.spawn(function()
        while task.wait(0.4) do
            if AutoAppraiserEnabled and SelectedFish ~= "" and SelectedFish ~= "Nenhum peixe encontrado" then
                pcall(function()
                    local player = game.Players.LocalPlayer
                    local char = player.Character
                    
                    -- 1. Equipar Peixe
                    local tool = char:FindFirstChild(SelectedFish) or player.Backpack:FindFirstChild(SelectedFish)
                    if tool and tool.Parent == player.Backpack then
                        char.Humanoid:EquipTool(tool)
                    end

                    -- 2. Localizar NPC em Moosewood
                    local npc = nil
                    for _, v in pairs(game.Workspace:GetDescendants()) do
                        if v.Name == "Appraiser" and v:IsA("Model") and (v.Parent.Name:find("Moosewood") or v.Parent.Parent.Name:find("Moosewood")) then
                            npc = v break
                        end
                    end

                    if npc then
                        char.HumanoidRootPart.CFrame = npc.HumanoidRootPart.CFrame * CFrame.new(0, 0, -3)
                        task.wait(0.1)
                        fireproximityprompt(npc:FindFirstChildOfClass("ProximityPrompt"))
                        
                        -- Clica no "1" instantaneamente
                        local gui = player.PlayerGui:FindFirstChild("DialogueGui") or player.PlayerGui:FindFirstChild("AppraiserGui")
                        if gui then
                            for _, b in pairs(gui:GetDescendants()) do
                                if (b:IsA("TextButton") or b:IsA("ImageButton")) and (b.Text == "1" or b.Name:find("1")) then
                                    b:Activate()
                                end
                            end
                        end

                        -- Verificar Condição
                        local n = tool.Name:lower()
                        if (SelectedMutation ~= "None" and n:find(SelectedMutation:lower())) or (StopAtShiny and n:find("shiny")) or (StopAtSparkling and n:find("sparkling")) then
                            AutoAppraiserEnabled = false
                            AppToggle:Set(false)
                            Rayfield:Notify({Title = "Sucesso!", Content = "Peixe desejado obtido!", Duration = 5})
                        end
                    end
                end)
            end
        end
    end)

    -- LOGICA AUTO EQUIP ROD
    task.spawn(function()
        while task.wait(0.8) do
            if AutoEquipRod then
                pcall(function()
                    local char = game.Players.LocalPlayer.Character
                    local has = false
                    for _, i in pairs(char:GetChildren()) do if i:IsA("Tool") and IsARod(i) then has = true end end
                    if not has then
                        for _, i in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
                            if i:IsA("Tool") and IsARod(i) then char.Humanoid:EquipTool(i) break end
                        end
                    end
                end)
            end
        end
    end)

    -- LOOP VELOCIDADE
    task.spawn(function()
        while task.wait(0.5) do
            pcall(function() game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = WalkSpeedValue end)
        end
    end)
    
    -- INFINITE JUMP
    game:GetService("UserInputService").JumpRequest:Connect(function()
        if InfiniteJumpEnabled then
            pcall(function() game.Players.LocalPlayer.Character.Humanoid:ChangeState("Jumping") end)
        end
    end)
end

CreateBtn("Fisch Mode 🎣", UDim2.new(0.1, 0, 0.3, 0), Color3.fromRGB(40, 100, 200), function() StartHub("Fisch") end)
CreateBtn("Universal Mode 🌐", UDim2.new(0.1, 0, 0.6, 0), Color3.fromRGB(80, 80, 80), function() StartHub("Universal") end)
