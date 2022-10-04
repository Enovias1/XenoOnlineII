if not game:IsLoaded() then
    game.Loaded:Wait()
end

local Player = game.Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Mouse = Player:GetMouse()

local CoreGui = game:GetService('CoreGui')
local UserInputService = game:GetService('UserInputService')
local RunService = game:GetService('RunService')
local Workspace = game:GetService('Workspace')
local Players = game.Players


CheckForVelocity = function()
     if Player.Character then
        if Player.Character:FindFirstChild('HumanoidRootPart') then
        if not Player.Character.HumanoidRootPart:FindFirstChild('VelocityAttachment') then
             local Velocity = Instance.new('LinearVelocity')
             local Attachment = Instance.new('Attachment')
             Velocity.MaxForce = math.huge
             Velocity.VectorVelocity = Vector3.new(0,0,0)
             Attachment.Parent = Player.Character.HumanoidRootPart
             Velocity.Parent = Attachment
             Attachment.Name = 'VelocityAttachment'
             Velocity.Attachment0 = Attachment  
         end
      end          
   end
end

RemoveVelocity = function()
     if Player.Character then
        if Player.Character:FindFirstChild('HumanoidRootPart') then
          if  Player.Character.HumanoidRootPart:FindFirstChild('VelocityAttachment') then
                Player.Character.HumanoidRootPart.VelocityAttachment:Destroy()
           end
       end
    end
end

Punch = function()
    pcall(function()
         Character.Client.Events.LightAttack:FireServer('ForTheKiddies')
       local args = {
        [1] = "Recover"
        }

         game:GetService("ReplicatedStorage").TurboRemotes.Mobility:FireServer(unpack(args))
    end)
end

local function getPlayer(shortcut)
	local player = nil
	local g = game.Players:GetPlayers()
	for i = 1, #g do
		if string.lower(string.sub(g[i].Name, 1, string.len(shortcut))) == string.lower(shortcut) or string.lower(string.sub(g[i].DisplayName, 1, string.len(shortcut))) == string.lower(shortcut) then
			player = g[i]
			break
		end
	end
	return player
end

TpToPlayer = function(plr)
    local Player = getPlayer(plr)
    if Player then
         if Player.Character and Player.Character:FindFirstChild('HumanoidRootPart') then
           CheckForVelocity()
           wait(.2)
           Character:MoveTo(Player.Character.HumanoidRootPart.CFrame.Position)
           wait(3)
           RemoveVelocity()
        end
    end
end

HookValueFalse = function(value)
local instance = value
local mt = getrawmetatable(game)
local old = mt.__index
setreadonly(mt, false)

mt.__index = newcclosure(function(table, index)
   if table == instance and index == "Value" then
       return false
   end
   return old(table, index)
end)    
end

HookValueTrue = function(value)
local instance = value
local mt = getrawmetatable(game)
local old = mt.__index
setreadonly(mt, false)

mt.__index = newcclosure(function(table, index)
   if table == instance and index == "Value" then
       return True
   end
   return old(table, index)
end)    
end

local PlaceID = game.PlaceId
local AllIDs = {}
local foundAnything = ""
local actualHour = os.date("!*t").hour
local Deleted = false
local File = pcall(function()
    AllIDs = game:GetService('HttpService'):JSONDecode(readfile("NotSameServers.json"))
end)
if not File then
    table.insert(AllIDs, actualHour)
    writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
end
function TPReturner()
    local Site;
    if foundAnything == "" then
        Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
    else
        Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
    end
    local ID = ""
    if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
        foundAnything = Site.nextPageCursor
    end
    local num = 0;
    for i,v in pairs(Site.data) do
        local Possible = true
        ID = tostring(v.id)
        if tonumber(v.maxPlayers) > tonumber(v.playing) then
            for _,Existing in pairs(AllIDs) do
                if num ~= 0 then
                    if ID == tostring(Existing) then
                        Possible = false
                    end
                else
                    if tonumber(actualHour) ~= tonumber(Existing) then
                        local delFile = pcall(function()
                            delfile("NotSameServers.json")
                            AllIDs = {}
                            table.insert(AllIDs, actualHour)
                        end)
                    end
                end
                num = num + 1
            end
            if Possible == true then
                table.insert(AllIDs, ID)
                wait()
                pcall(function()
                    writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
                    wait()
                    game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, ID, game.Players.LocalPlayer)
                end)
                wait(4)
            end
        end
    end
end

function Teleport()
    while wait() do
        pcall(function()
            TPReturner()
            if foundAnything ~= "" then
                TPReturner()
            end
        end)
    end
end


local Config,BattlePower;
StartCharacterFunctions = function()
    Config = Character:WaitForChild('Config')
    BattlePower = Config:WaitForChild('BattlePower')
   
   local KiSenseIcon = Character.HumanoidRootPart:FindFirstChild('KiSenseIcon')
   if KiSenseIcon then
       KiSenseIcon:Destroy()
   end
   
    
   --[[Auto - Refresh]]--
   local Health = Character:WaitForChild('Health')
   Health:GetPropertyChangedSignal('Value'):Connect(function()
       if Health.Value <= 25 and Settings.AutoRefresh == true then
           Character.Humanoid:Destroy()
       end
   end)
end

StartCharacterFunctions()

ModifyConfig = function()
    --[[Config Value Modifier]]--
   Config = Character:WaitForChild('Config')
   HookValueFalse(Config.Stunned)
   HookValueFalse(Config.FullyStunned)
   HookValueFalse(Config.AttackCooldown)
   HookValueFalse(Config.CAttackCooldown)
   HookValueFalse(Config.NoAttack)
   HookValueFalse(Config.NoBlock)
   
   HookValueTrue(Config.CanBlock)
   HookValueTrue(Config.CanFly)
   HookValueTrue(Config.CanRecover)
end

SpawnShadow = function()
    local args = {
    [1] = "ShadowSpar"
    }

   game:GetService("ReplicatedStorage").TurboRemotes.Generic:FireServer(unpack(args))
end

Rest = function()
    local args = {
    [1] = "Rest"
    }

   game:GetService("ReplicatedStorage").TurboRemotes.Generic:FireServer(unpack(args))
end

Player.CharacterAdded:connect(function(NewCharacter)
    Character = NewCharacter
    StartCharacterFunctions()
end)


StartShadowFarm = function()
--[[Checks]]--
if Character.Config.Resting.Value == true then
    for i = 1,999 do
        wait(1.3)
            if Character.Config.Resting.Value == true then
               Rest()
            else
            break
        end 
    end
end



local Spot;
spawn(function()
CheckForVelocity()
wait()
Character:MoveTo(Character.HumanoidRootPart.Position + Vector3.new(0,100000,0))
Spot = Character.HumanoidRootPart.CFrame.Position
if not workspace:FindFirstChild(Player.Name.."'s Shadow Image") then
    wait(5)
    for i = 1,999 do
    wait(1.3)
        if not workspace:FindFirstChild(Player.Name.."'s Shadow Image") then
            SpawnShadow()
                else
            break
        end
    end
end
end)


local NextToShadow;
spawn(function()
    while task.wait(.1) do
        if NextToShadow then
           Punch()
        end
    end
end)


--[[Check For Shadow Death]]--
    workspace.ChildRemoved:Connect(function(child)
        if child.Name == (Player.Name.."'s Shadow Image") then
                delay(2,function()
                    Character:MoveTo(Spot)
                    wait(2)
                    for i = 1,999 do
                        wait(1.3)
                        if Character.Config.Resting.Value == false then
                            Rest()
                            else
                            break
                        end
                    end
                end)
                wait(120)
               for i = 1,999 do
                        wait(1.3)
                        if not workspace:FindFirstChild(Player.Name.."'s Shadow Image") then
                            SpawnShadow()
                            else
                        break
                    end
                end
                wait(2)
                for i = 1,999 do
                    wait(1.3)
                     if Character.Config.Resting.Value == true then
                     Rest()
                    else
                break
              end
           end
        end
    end)
    
    --[[else
    workspace.ChildAdded:Connect(function(child)
        if child.Name == (Player.Name.."'s Shadow Image") then
            workspace.ChildRemoved:Connect(function(child2)
                if child2.Name == child.Name then   
                    delay(2,function()
                    Character:MoveTo(Spot)
                    wait(2)
                    for i = 1,3 do
                        wait(1.3)
                        if Character.Config.Resting.Value == false then
                            Rest()
                        else
                        break
                        end
                    end
                    end)
                   wait(250)     
                    for i = 1,3 do
                        wait(1.3)
                        if not workspace:FindFirstChild(Player.Name.."'s Shadow Image") then
                            SpawnShadow()
                            else
                            break
                        end
                    end
                    wait(2)
                    for i = 1,3 do
                        wait(1.3)
                        if Character.Config.Resting.Value == true then
                            Rest()
                            else
                            break
                        end
                    end
                end
            end)
        end
    end)
end]]


--[[Teleport]]--
local ShadowFarm; ShadowFarm = RunService.Heartbeat:connect(function()
    CheckForVelocity()
    
    if workspace:FindFirstChild(Player.Name.."'s Shadow Image") and Character then
        local Shadow = workspace:FindFirstChild(Player.Name.."'s Shadow Image")
        if Shadow:FindFirstChild('HumanoidRootPart') then
           Character.HumanoidRootPart.CFrame = Shadow.HumanoidRootPart.CFrame * CFrame.new(0,0,2.95)
            NextToShadow = true
        end         
    end
end)
end




local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window1 = Library.CreateLib("Xeno Online II  |  Credits: JBZ#1894", "Ocean")

--JBZ#1894






--[[Tabs]]--
local Main = Window1:NewTab("Main")
local Utilites = Window1:NewTab('Utilites')
local Settins = Window1:NewTab('Settings')

--[[Shadow Farm]]--
local FarmSection = Main:NewSection("Farm")
FarmSection:NewButton("Shadow Farm (Fists)", "Farm Shadow With M1s (Non Toggleable)", function()
    StartShadowFarm()
end)


--[[CharacterTab]]--
local CharacterSection = Main:NewSection("Character")


--[[Auto-ReFresh]]--
--[[CharacterSection:NewToggle("AutoRefresh", "Automatically Refreshes Character When Low", function(Active)
    Setttings.AutoRefresh = not Settings.AutoRefresh
end)]]

CharacterSection:NewButton("Refresh", "Refresh Your Character", function()
    pcall(function ()
        Character.Humanoid:Destroy()
    end)
end)

--[[Config Modifier]]--
CharacterSection:NewButton("Config Modifier", "Removes Cooldowns On Some Things", function()
    ModifyConfig()
end)

--[[Utilites]]--
local stuff = Utilites:NewSection("Stuff")
stuff:NewTextBox("Teleport To Player", "Teleport To Player", function(txt)
	TpToPlayer(txt)
end)

stuff:NewButton("ServerHop", "h", function()
    Teleport()
end)



local Settings = Settins:NewSection('Settings')
Settings:NewKeybind("Toggle Ui", "g", Enum.KeyCode.RightControl, function()
	pcall(function()
    Library:ToggleUI()
    end)
end)





