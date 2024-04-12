local isvoid = game.PlaceId==11879754496

local optimize = false

if game.CoreGui:FindFirstChild("killmenow") then
	game.CoreGui:FindFirstChild("killmenow"):Destroy()
end

local players = game:GetService("Players")
local plr = players.LocalPlayer
local mouse = plr:GetMouse()
local char = plr.Character
local root:Part = if char then char:FindFirstChild("HumanoidRootPart") else nil
local hum:Humanoid = if char then char:FindFirstChild("Humanoid") else nil
plr.CharacterAdded:Connect(function()
	char = plr.Character
	root = char:WaitForChild("HumanoidRootPart")
	hum = char:WaitForChild("Humanoid")
end)
local rep = game:GetService("ReplicatedStorage")
local rs = game:GetService("RunService")
local input = game:GetService("UserInputService")
local keycodes = Enum.KeyCode:GetEnumItems()
local ts = game:GetService("TeleportService")
local cam = workspace.CurrentCamera

--FUCK STREAMING ENABLED

local packets = not isvoid and require(rep.Modules.Packets) or {}
if isvoid then
	for i,v in pairs(rep:WaitForChild("Events"):GetChildren()) do
		if v:IsA("RemoteEvent") then
			packets[v.Name]={send=function(...) v:FireServer(...) end}
		end
	end
end
local bytenet:RemoteEvent = if not isvoid then rep:FindFirstChild("ByteNetReliable") else nil

local offloaded:Instance = not isvoid and game:GetService("ReplicatedFirst").Animals.Offloaded or Instance.new("Folder")
local statsgui = plr.PlayerGui:WaitForChild("MainGui"):WaitForChild("Panels"):WaitForChild("Stats")
local inventorygui = plr.PlayerGui:WaitForChild("MainGui"):WaitForChild("RightPanel"):WaitForChild("Inventory"):WaitForChild("List")

local uilib = loadstring(game:HttpGet("https://github.com/huhthatsnice/pro/raw/main/uilib.lua"))()

local window = uilib:CreateWindow("Erudites Private V1")
window.ScreenGui.Name="killmenow"

--functions

local veltoggle = Instance.new("BodyVelocity")
veltoggle.P=math.huge
veltoggle.Velocity=Vector3.new()
veltoggle.MaxForce=Vector3.new(math.huge,math.huge,math.huge)
task.spawn(function()
	while true do rs.PreSimulation:Wait()
		if veltoggle.Parent and veltoggle.Parent:IsA("BasePart") then
			veltoggle.Parent.Velocity=Vector3.new(veltoggle.MaxForce.X==0 and veltoggle.Parent.Velocity.X or 0,veltoggle.MaxForce.Y==0 and veltoggle.Parent.Velocity.Y or 0,veltoggle.MaxForce.Z==0 and veltoggle.Parent.Velocity.Z or 0)
		end
	end
end)

local function merge(t1,...)
	for i,t2 in pairs({...}) do
		table.move(t2,1,#t2,#t1+1,t1)
	end
	return t1
end
local function remove(t1,find)
	return table.remove(t1,table.find(t1,find))
end
local function getClose(t:{Instance},range,ffunc:(i,v)->BasePart)
	if not root then return end
	ffunc=ffunc or function(i,v) return v end
	local ret = {}
	for i,v in pairs(t) do
		v=ffunc(i,v)
		if (v.Position-root.Position).Magnitude<range then
			ret[v]=i
		end
	end
	return ret
end
local rainparts = {}
for i,v in pairs(workspace:GetChildren()) do
	if v.Name=="RainPart" then
		table.insert(rainparts,v)
	end
end
workspace.ChildAdded:Connect(function(v)
	if v.Name=="RainPart" then
		table.insert(rainparts,v)
	end
end)
local plrchars = {}
for i,v in pairs(game:GetService("Players"):GetPlayers()) do
	plrchars[v]=v.Character
	v.CharacterAdded:Connect(function(c)
		plrchars[v]=v.Character
		local char = v.Character
		char:GetPropertyChangedSignal("Parent"):Connect(function()
			if char.Parent==nil then
				plrchars[v]=nil
			end
		end)
	end)
	if not v.Character then continue end
	local char = v.Character
	char:GetPropertyChangedSignal("Parent"):Connect(function()
		if char.Parent==nil then
			plrchars[v]=nil
		end
	end)
end
local function getMover(part)
	for i,v in pairs(part:GetDescendants()) do
		if not v:IsA("BasePart") then continue end
		local ocf = v.CFrame
		v.CFrame=CFrame.new()
		if v.CFrame==CFrame.new() then
			v.CFrame=ocf
			return v
		end
	end
end
local function getMovePart():BasePart
	if not root then return nil end
	if not (hum and root and hum.SeatPart and hum.SeatPart.Parent) then return root end
	return getMover(hum.SeatPart.Parent) or root
end
local function moveTo(pos:CFrame|Vector3)
	if not getMovePart() then return end
	if typeof(pos)=="Vector3" then pos = CFrame.new(pos) end
	local move=getMovePart()
	local dif = (move.CFrame.Position-root.CFrame.Position)
	move.CFrame = pos+dif
end
local function getMovementRaycastParams()
	local rp = RaycastParams.new()
	rp.IgnoreWater=true
	rp.FilterType=Enum.RaycastFilterType.Exclude
	local filt=merge({workspace:FindFirstChild("Items"),getMovePart() and getMovePart().Parent},rainparts)
	for i,v in pairs(game:GetService("Players"):GetPlayers()) do
		table.insert(filt,v.Character)
	end
	rp.FilterDescendantsInstances=filt
	return rp
end
local vels = {}
local parts = {}
local function disableBoat()
	if not getMovePart() then return end
	for i,v in pairs(getMovePart().Parent:GetDescendants()) do
		if v~=veltoggle and (v:IsA("BodyVelocity") or v:IsA("BodyPosition")) then
			vels[v]=v.MaxForce
			v.MaxForce=Vector3.new()
		elseif v:IsA("BasePart") then
			table.insert(parts,v)
			v.CanCollide=false
			v.Massless=true
		end
	end
	veltoggle.Parent=getMovePart()
	veltoggle.MaxForce=Vector3.new(math.huge,math.huge,math.huge)
end
local function enableBoat()
	for i,v in pairs(vels) do
		i.MaxForce=v
	end
	for i,v in pairs(parts) do
		v.CanCollide=true
		v.Massless=false
	end
	table.clear(vels)
	table.clear(parts)
	veltoggle.Parent=nil
end
local function teleportStepToward(pos,rate,step,height)
	local posflat=Vector3.new(pos.X,0,pos.Z)
	local cposflat=Vector3.new(root.CFrame.Position.X,0,root.CFrame.Position.Z)
	local dir = (posflat-cposflat).Unit
	local dist = (posflat-cposflat).Magnitude
	if dir.X~=dir.X then
		dir=Vector3.new()
	end
	cposflat+=dir*math.clamp((step or rs.PreSimulation:Wait())*rate,0,dist)
	local ray = workspace:Raycast(cposflat+Vector3.new(0,getMovePart().Position.Y+25,0),Vector3.new(0,-10000,0),getMovementRaycastParams())
	if ray then
		moveTo(ray.Position+Vector3.new(0,height or 3.5,0))
	end
end
local function teleportTo(pos:Vector3,rate:number,reenable:boolean,validator:()->boolean,height)
	local posflat=Vector3.new(pos.X,0,pos.Z)
	local cposflat=Vector3.new(root.CFrame.Position.X,0,root.CFrame.Position.Z)
	local dir = (posflat-cposflat).Unit

	disableBoat()
	while getMovePart() and validator() do
		local step = rate*rs.PreSimulation:Wait()
		if (cposflat-posflat).Magnitude<step then
			moveTo(pos)
			break
		else
			cposflat+=dir*step
			local ray = workspace:Raycast(cposflat+Vector3.new(0,1000,0),Vector3.new(0,-2000,0),getMovementRaycastParams())
			if ray then
				moveTo(ray.Position+Vector3.new(0,height or 5,0))
			end
		end
	end
	if reenable==nil or reenable then
		enableBoat()
	end
end
local function getSlot(name)
	if inventorygui:FindFirstChild(name) and not inventorygui[name]:IsA("UILayout") then
		return inventorygui[name].LayoutOrder
	end
end
local function getCount(name)
	if inventorygui:FindFirstChild(name) and not inventorygui[name]:IsA("UILayout") and inventorygui[name]:FindFirstChild("QuantityText",true) then
		return tonumber(inventorygui[name]:FindFirstChild("QuantityText",true).Text) or 0
	end
	return 0
end
local function trim(str)
	return string.gsub(str, '^%s*(.-)%s*$', '%1')
end
local function getServers(id)
	return game:GetService("HttpService"):JSONDecode(request({Url=`https://games.roblox.com/v1/games/{id}/servers/0?sortOrder=2&excludeFullGames=true&limit=100`}).Body)
end
--devs fucking implemented buffers so now i gotta fucking make custom functions for the simplest shit --nevermind i figured it out :3

local itemids
local itemdata
for i,v in pairs(getreg()) do
	if type(v)=="table" then
		if not itemids and v[1]=="Wood" then
			itemids=v
		elseif not itemdata and type(v.Wood)=="table" and v.Wood.itemType then
			itemdata=v
		elseif itemids and itemdata then
			break
		end
	end
end
local function getItemId(name)
	if isvoid then return name end
	return table.find(itemids,name)
end
local function hit(parts)
	packets.SwingTool.send(parts)
end
local function useSlot(slot)
	packets.UseBagItem.send(slot)
end
local pickupbuf = buffer.create(2)
buffer.writeu8(pickupbuf,0,58)
buffer.writeu8(pickupbuf,1,1)
local function pickup(part)
	if isvoid or not optimize then
		packets.Pickup.send(part)
	else
		bytenet:FireServer(pickupbuf,{part})
	end
end
local function plant(box,plant)
	if not isvoid then
		packets.InteractStructure.send({structure=box,itemID=plant})
	else
		packets.InteractStructure.send(box,plant)
	end
end
local function grab(item)
	packets.ForceInteract.send(item)
end
local function craft(item)
	packets.CraftItem.send(item)
end
local function touch(p1,p2) --pray this doesn't crash
	pcall(function()
		firetouchinterest(p1,p2,1)
		firetouchinterest(p1,p2,0)
	end)
end
local function place(name,rot,pos)
	if not isvoid then
		packets.PlaceStructure.send({
			buildingName=name,
			yrot=rot,
			vec=pos,
			isMobile=false,
		})
	else
		packets.PlaceStructure.send(
			pos,
			name,
			rot,
			false
		)
	end
end
local pressbuf = buffer.create(4)
buffer.writeu8(pressbuf,0,19)
buffer.writeu8(pressbuf,1,1)
buffer.writeu16(pressbuf,2,if not isvoid then getItemId("Gold") else 0)
local function press(press)
	if isvoid or not optimize then return plant(press,"Gold") end
	bytenet:FireServer(pressbuf,{press})
end


--acctual script begin
local combat = window:AddTab("Combat")
local movement = window:AddTab("Movement")
local resource = window:AddTab("Resource")
local utility = window:AddTab("Utility")
local visuals = window:AddTab("Visuals")

local speed = movement:AddSection("Speed")
speed_enabled=speed:AddSetting("Enabled","Toggle")
speed_keybind=speed:AddSetting("Keybind","String","Y")
speed_speed=speed:AddSetting("Speed","Slider",23.5,0,25,0.1)
speed_boatspeed=speed:AddSetting("Boat Speed","Slider",65,0,75,0.1)
speed_floatheight=speed:AddSetting("Float Height","Slider",3,0,15,0.1)

input.InputBegan:Connect(function(inp)
	if not window.ScreenGui.Parent or input:GetFocusedTextBox() then return end
	if speed_keybind.Value:lower()==inp.KeyCode.Name:lower() then
		speed:UpdateSettingValue("Enabled",not speed_enabled.Value)
	end
end)
local lsp:BasePart
speed:ConnectSettingUpdate("Enabled",function()
	if not window.ScreenGui.Parent or input:GetFocusedTextBox() then return end
	if not speed_enabled.Value and lsp and lsp.Parent then
		enableBoat()
		lsp = nil
	end
end)

task.spawn(function()
	while window.ScreenGui.Parent do
		local t = rs.PreSimulation:Wait()
		if autochasing or autofarm_enabled.Value then continue end
		if speed_enabled.Value and hum and root then
			if hum.SeatPart then
				disableBoat()
				lsp = hum.SeatPart
				local ray = workspace:Raycast(root.Position+(hum.MoveDirection*speed_boatspeed.Value*t)+Vector3.new(0,10,0),Vector3.new(0,-10000,0),getMovementRaycastParams())
				if ray then
					moveTo(getMovePart().CFrame.Rotation+ray.Position+Vector3.new(0,speed_floatheight.Value,0))
				else
					moveTo(getMovePart().CFrame+(hum.MoveDirection*speed_boatspeed.Value*t))
				end
			else
				if lsp and lsp.Parent then
					enableBoat()
				end
				veltoggle.Parent=root
				veltoggle.MaxForce=Vector3.new(math.huge,0,math.huge)
				root.CFrame+=hum.MoveDirection*speed_speed.Value*t
			end
		else
			enableBoat()
			veltoggle.Parent=nil
		end
	end
	enableBoat()
end)

local killaura = combat:AddSection("Killaura")
killaura_enabled = killaura:AddSetting("Enabled","Toggle")
killaura_keybind = killaura:AddSetting("Keybind","String","R")
killaura_range = killaura:AddSetting("Range","Slider",25,0,25,0.1)
killaura_rate = killaura:AddSetting("Rate","Slider",30,0,120,1)

input.InputBegan:Connect(function(inp)
	if not window.ScreenGui.Parent or input:GetFocusedTextBox() then return end
	if killaura_keybind.Value:lower()==inp.KeyCode.Name:lower() then
		killaura:UpdateSettingValue("Enabled",not killaura_enabled.Value)
	end
end)

local lasthit=0
task.spawn(function()
	while window.ScreenGui.Parent do
		local t = rs.PostSimulation:Wait()
		if killaura_enabled.Value and char and root and tick()-lasthit>(1/killaura_rate.Value) then
			local plrs = game:GetService("Players"):GetPlayers()
			local hits = {}
			for i,v in pairs(plrs) do
				if v and v ~= plr and (plr.Team.Name=="NoTribe" or plr.Neutral or plr.Team~=v.Team) and v.Character then
					local vroot:Part=v.Character:FindFirstChild("HumanoidRootPart")
					local vhum:Humanoid=v.Character:FindFirstChild("Humanoid")
					if vroot and vhum and (vroot.Position-root.Position).Magnitude<killaura_range.Value and vhum.Health>0 then
						for i,v in pairs(v.Character:GetChildren()) do
							if v:IsA("Part") then
								table.insert(hits,v)
							end
						end
					end
				end
			end
			for i,v in pairs(workspace.Critters:GetChildren()) do
				local vroot:Part=v:FindFirstChild("HumanoidRootPart")
				if vroot and (vroot.Position-root.Position).Magnitude<killaura_range.Value then
					for i,v in pairs(v:GetChildren()) do
						if v:IsA("Part") then
							table.insert(hits,v)
						end
					end
				end
			end
			if isvoid then
				for i,v in pairs(workspace:GetChildren()) do
					if v.Name=="Void Ant" then
						local vroot:Part=v:FindFirstChild("HumanoidRootPart")
						if vroot and (vroot.Position-root.Position).Magnitude<killaura_range.Value then
							for i,v in pairs(v:GetChildren()) do
								if v:IsA("Part") then
									table.insert(hits,v)
								end
							end
						end
					end
				end
			end
			if not isvoid then
				for i,v in pairs(workspace.HumanoidCritters:GetChildren()) do
					local vroot:Part=v:FindFirstChild("HumanoidRootPart")
					local vhum:Humanoid=v:FindFirstChild("Humanoid")
					if vroot and vhum and vhum.Health>0 and (vroot.Position-root.Position).Magnitude<killaura_range.Value then
						for i,v in pairs(v:GetChildren()) do
							if v:IsA("Part") then
								table.insert(hits,v)
							end
						end
					end
				end
			end
			if #hits>0 then
				hit(hits)
				lasthit=tick()
			end
		end
	end
end)

local clipup = movement:AddSection("ClipUp")
clipup_enabled=clipup:AddSetting("Activate","Button")
clipup_keybind=clipup:AddSetting("Keybind","String","E")

input.InputBegan:Connect(function(inp)
	if not window.ScreenGui.Parent or input:GetFocusedTextBox() then return end
	if clipup_keybind.Value:lower()==inp.KeyCode.Name:lower() then
		clipup:UpdateSettingValue("Activate")
	end
end)

clipup:ConnectSettingUpdate("Activate",function()
	if root and hum then
		local ray = workspace:Raycast(root.Position+Vector3.new(0,5000,0),Vector3.new(0,-10000),getMovementRaycastParams())
		if ray then
			moveTo(getMovePart().CFrame.Rotation+ray.Position+Vector3.new(0,2,0))
		end
	end
end)

local autograb = resource:AddSection("AutoGrab")
autograb_enabled=autograb:AddSetting("Enabled","Toggle")
autograb_keybind=autograb:AddSetting("Keybind","String","")
autograb_range=autograb:AddSetting("Range","Slider",25,0,25,0.1)
autograb_whitelistenabled=autograb:AddSetting("WhitelistEnabled","Toggle",true)
autograb_whitelist=autograb:AddSetting("Whitelist","String","Gold, Crystal Chunk, Void Shard, Essence, Emerald, Pink Diamond, Coin2, Coin, Magnetite, Spirit Key")

input.InputBegan:Connect(function(inp)
	if not window.ScreenGui.Parent or input:GetFocusedTextBox() then return end
	if autograb_keybind.Value:lower()==inp.KeyCode.Name:lower() then
		autograb:UpdateSettingValue("Enabled")
	end
end)

local grabbed = {}
local chests = {}
for i,v in pairs(workspace.Deployables:GetChildren()) do
	if v.Name:lower():find("chest") then
		table.insert(chests,v:FindFirstChild("Contents"))
	end
end
workspace.Deployables.ChildAdded:Connect(function(v)
	if v.Name:lower():find("chest") then
		table.insert(chests,v:FindFirstChild("Contents"))
	end
end)

task.spawn(function()
	while window.ScreenGui.Parent do rs.RenderStepped:Wait()
		if autograb_enabled.Value and root and hum and hum.Health>0 then
			local children = workspace.Items:GetChildren()
			for i,v in pairs(chests) do
				for i,v in pairs(v:GetChildren()) do
					table.insert(children,v)
				end
			end
			local whitelist = {}
			if autograb_whitelistenabled.Value then
				for i,v in pairs(autograb_whitelist.Value:split(",")) do
					whitelist[trim(v)]=true
				end
			end
			for i,v:Instance in pairs(children) do
				if grabbed[v] then continue end
				if autograb_whitelistenabled.Value and not whitelist[v.Name] then continue end
				if v:IsA("Model") then
					if #v:GetChildren()>2 and (v:GetPivot().Position-root.Position).Magnitude<autograb_range.Value then
						print("grab")
						pickup(v)
						grabbed[v]=true
						task.spawn(function()
							task.wait(1)
							grabbed[v]=nil
						end)
					end
				elseif v:IsA("BasePart") then
					if (v.Position-root.Position).Magnitude<autograb_range.Value then
						print("grab")
						pickup(v)
						grabbed[v]=true
						task.spawn(function()
							task.wait(1)
							grabbed[v]=nil
						end)
					end
				end
			end
		end
	end
end)

local autochest = resource:AddSection("Auto Chest")
autochest_enabled=autochest:AddSetting("Enabled","Toggle")
autochest_keybind=autochest:AddSetting("Keybind","String","")
autochest_range=autochest:AddSetting("Range","Slider",25,0,25,0.1)
autochest_whitelistenabled=autochest:AddSetting("WhitelistEnabled","Toggle",false)
autochest_whitelist=autochest:AddSetting("Whitelist","String","Gold, Crystal Chunk, Void Shard, Essence, Emerald, Pink Diamond")
autochest_bind=autochest:AddSetting("Bind","Button")

local chest 

autochest:ConnectSettingUpdate("Bind",function()
	mouse.Button1Down:Wait()
	chest = mouse.Target and mouse.Target.Parent and mouse.Target.Parent:FindFirstChild("Base")
end)

input.InputBegan:Connect(function(inp)
	if not window.ScreenGui.Parent or input:GetFocusedTextBox() then return end
	if autochest_keybind.Value:lower()==inp.KeyCode.Name:lower() then
		autochest:UpdateSettingValue("Enabled")
	end
end)

local waitingforchest=false
task.spawn(function()
	while window.ScreenGui.Parent do rs.RenderStepped:Wait()
		if autochest_enabled.Value and root and hum and hum.Health>0 and chest then
			local closest
			local closestmag = autochest_range.Value
			local whitelist = autochest_whitelist.Value:split(",")
			for i,v in pairs(whitelist) do
				whitelist[i]=trim(v)
			end
			for i,v:Instance in pairs(workspace.Items:GetChildren()) do
				if grabbed[v] then continue end
				if autochest_whitelistenabled.Value then
					if not table.find(whitelist,v.Name) then
						continue
					end
				end
				local pos = if v:IsA("BasePart") then v.Position elseif v:IsA("Model") then v:GetPivot().Position else nil
				if pos and (pos-root.Position).Magnitude<closestmag then
					closest = v
					closestmag = (pos-root.Position).Magnitude
				end
			end
			if not closest then continue end
			if closest:IsA("Model") then
				local mover = getMover(closest)
				if mover then
					task.spawn(function()
						waitingforchest=true
						grab(closest)
						task.wait(plr:GetNetworkPing()+0.1)
						touch(mover,chest)
						grab()
						grabbed[v]=true
						task.spawn(function()
							task.wait(1)
							grabbed[v]=nil
						end)
						waitingforchest=false
					end)
				end
			elseif closest:IsA("BasePart") then
				task.spawn(function()
					waitingforchest=true
					grab(closest)
					task.wait(plr:GetNetworkPing()+0.1)
					touch(closest,chest)
					grab()
					grabbed[v]=true
					task.spawn(function()
						task.wait(1)
						grabbed[v]=nil
					end)
					waitingforchest=false
				end)
			end
		end
	end
end)


local autofarm = resource:AddSection("Auto Farm")
autofarm_enabled=autofarm:AddSetting("Enabled","Toggle")
autofarm_antrange=autofarm:AddSetting("Avoid Ant Range","Slider",100,0,100)
autofarm_resources=autofarm:AddSetting("Resources","String","Gold Node")
autofarm_usechest=autofarm:AddSetting("Use Chest","Toggle")
autofarm_speed=autofarm:AddSetting("Speed","Slider",50,0,250)

local tppos
local itemcon:RBXScriptConnection
local function getAntsDistance(pos)
	if isvoid then return math.huge end
	local closestmag = math.huge
	for i,v in pairs(workspace.Mounds:GetChildren()) do
		local dist = (v:GetPivot().Position-pos).Magnitude
		if dist<closestmag then
			closestmag=dist
		end
	end
	return closestmag
end
local function validgold()
	local search = autofarm_resources.Value:split(",")
	for i,v in pairs(search) do
		search[i]=trim(v)
	end
	local closest
	local closestmag = math.huge
	local frp = root.Position-Vector3.new(0,root.Position.Y,0)
	for i,v:Instance in pairs(workspace.Resources:GetChildren()) do
		if v:IsA("Model") and table.find(search,v.Name) and v:GetPivot().Position~=Vector3.new() then
			local fpos = v:GetPivot().Position-Vector3.new(0,v:GetPivot().Position.Y,0)
			local dist =(fpos-frp).Magnitude
			if dist<closestmag then
				local cdist = getAntsDistance(v:GetPivot().Position)
				if cdist>autofarm_antrange.Value then
					closest=v
					closestmag=dist
				end
			end
		end
	end
	return closest==nil
end

local waiting = false
local defaulttppos
task.spawn(function()
	while not defaulttppos do rs.PostSimulation:Wait()
		tppos = workspace:Raycast(Vector3.new(-217,1000,-678),Vector3.new(0,-10000,0),getMovementRaycastParams())
		if tppos then
			defaulttppos=tppos.Position
		end
	end
end)
autofarm:ConnectSettingUpdate("Enabled",function()
	if not autofarm_enabled.Value then return end
	tppos=root.Position
	hum:SetStateEnabled(Enum.HumanoidStateType.Jumping,false)
	while window.ScreenGui.Parent and autofarm_enabled.Value do rs.PostSimulation:Wait()
		disableBoat()
		local search = autofarm_resources.Value:split(",")
		for i,v in pairs(search) do
			search[i]=trim(v)
		end
		local closest
		local closestmag = math.huge
		local frp = root.Position-Vector3.new(0,root.Position.Y,0)
		for i,v:Instance in pairs(workspace.Resources:GetChildren()) do
			if v:IsA("Model") and table.find(search,v.Name) and v:GetPivot().Position~=Vector3.new() then
				local fpos = v:GetPivot().Position-Vector3.new(0,v:GetPivot().Position.Y,0)
				local dist =(fpos-frp).Magnitude
				if dist<closestmag then
					local cdist = getAntsDistance(v:GetPivot().Position)
					if cdist>autofarm_antrange.Value then
						closest=v
						closestmag=dist
					end
				end
			end
		end
		if closest then
			waiting=false
			teleportTo(closest:GetPivot().Position,autofarm_speed.Value,false,function()
				return getAntsDistance(closest:GetPivot().Position)>autofarm_antrange.Value and autofarm_enabled.Value
			end)
			disableBoat()
			itemcon = workspace.Items.ChildAdded:Connect(function(v:Instance)
				if not autofarm_usechest.Value then
					if v:IsA("Model") then
						if #v:GetChildren()>2 and (v:GetPivot().Position-root.Position).Magnitude<autograb_range.Value then
							pickup(v)
							grabbed[v]=true
							task.spawn(function()
								task.wait(1)
								grabbed[v]=nil
							end)
						end
					elseif v:IsA("BasePart") then
						if (v.Position-root.Position).Magnitude<autograb_range.Value then
							pickup(v)
							grabbed[v]=true
							task.spawn(function()
								task.wait(1)
								grabbed[v]=nil
							end)
						end
					end
				end
			end)
			local rp = RaycastParams.new()
			rp.FilterType=Enum.RaycastFilterType.Include
			rp.FilterDescendantsInstances={closest}
			tppos = workspace:Raycast(closest:GetPivot().Position+Vector3.new(0,50,0),Vector3.new(0,-100,0),rp) or closest:GetPivot()
			tppos=tppos.Position
			local closeparts={}
			for i,v in pairs(closest:GetDescendants()) do
				if v:IsA("BasePart") then
					table.insert(closeparts,v)
				end
			end
			while tppos and autofarm_enabled.Value and closest.Parent and getAntsDistance(closest:GetPivot().Position)>autofarm_antrange.Value do rs.RenderStepped:Wait()
				disableBoat()
				moveTo(tppos+Vector3.new(0,1.5,0))
				hit(closeparts)
			end
			tppos = workspace:Raycast(tppos,Vector3.new(0,-100,0),getMovementRaycastParams()) or root.CFrame
			tppos=tppos.Position
			local t = tick()
			local lgrabbed = {}
			local waittime = plr:GetNetworkPing()+0.1
			while tppos and autofarm_enabled.Value and (tick()-t<waittime or autofarm_usechest.Value) and getAntsDistance(closest:GetPivot().Position)>autofarm_antrange.Value do rs.PostSimulation:Wait()
				if not autofarm_usechest.Value  then
					for i,v:Instance in pairs(workspace.Items:GetChildren()) do
						if lgrabbed[v] then continue end
						if v:IsA("Model") then
							if #v:GetChildren()>2 and (v:GetPivot().Position-root.Position).Magnitude<autograb_range.Value then
								pickup(v)
								lgrabbed[v]=true
								task.spawn(function()
									task.wait(1)
									lgrabbed[v]=nil
								end)
							end
						elseif v:IsA("BasePart") then
							if (v.Position-root.Position).Magnitude<autograb_range.Value then
								pickup(v)
								lgrabbed[v]=true
								task.spawn(function()
									task.wait(1)
									lgrabbed[v]=nil
								end)
							end
						end
					end
				else
					local closest
					local closestmag = autochest_range.Value
					for i,v:Instance in pairs(workspace.Items:GetChildren()) do
						if grabbed[v] then continue end
						local pos = if v:IsA("BasePart") then v.Position elseif v:IsA("Model") then v:GetPivot().Position else nil
						if pos and (pos-root.Position).Magnitude<closestmag then
							closest = v
							closestmag = (pos-root.Position).Magnitude
						end
					end
					if not closest then
						if tick()-t<0.5 then continue end
						break
					end
					if closest:IsA("Model") then
						local mover = getMover(closest)
						if mover then
							grab(closest)
							task.wait(plr:GetNetworkPing()+0.1)
							touch(closest,chest)
							grab()
							grabbed[closest]=true
							task.spawn(function()
								task.wait(1)
								grabbed[closest]=nil
							end)
						end
					elseif closest:IsA("BasePart") then
						grab(closest)
						task.wait(plr:GetNetworkPing()+0.1)
						touch(closest,chest)
						grab()
						grabbed[closest]=true
						task.spawn(function()
							task.wait(1)
							grabbed[closest]=nil
						end)
					end
				end
				disableBoat()
				moveTo(tppos+Vector3.new(0,3,0))
			end
			itemcon:Disconnect()
			itemcon=nil
		else
			if defaulttppos then
				if not waiting then
					waiting=true
					teleportTo(defaulttppos+Vector3.new(0,3,0),autofarm_speed.Value,false,function()
						return autofarm_enabled.Value and validgold()
					end)
				else
					moveTo(defaulttppos+Vector3.new(0,3,0))
				end
			end
		end
	end
	waiting=false
	if itemcon then	
		itemcon:Disconnect()
		itemcon=nil
	end
	hum:SetStateEnabled(Enum.HumanoidStateType.Jumping,true)
	enableBoat()
	tppos=nil
end)

local resourceaura = resource:AddSection("ResourceAura")
resourceaura_enabled = resourceaura:AddSetting("Enabled","Toggle")
resourceaura_keybind = resourceaura:AddSetting("Keybind","String","")
resourceaura_range = resourceaura:AddSetting("Range","Slider",25,0,25)
resourceaura_rate = resourceaura:AddSetting("Rate","Slider",30,0,120)

input.InputBegan:Connect(function(inp)
	if not window.ScreenGui.Parent or input:GetFocusedTextBox() then return end
	if resourceaura_keybind.Value==inp.KeyCode.Name then
		resourceaura_enabled:UpdateSettingValue("Enabled",not resourceaura_enabled.Value)
	end
end)

local lasthit=0
task.spawn(function()
	while window.ScreenGui.Parent do
		local t = rs.PostSimulation:Wait()
		if resourceaura_enabled.Value and char and root and tick()-lasthit>(1/resourceaura_rate.Value) then
			local resources:{Instance}=workspace.Resources:GetChildren()
			local hits = {}
			for i,v in pairs(resources) do
				if v:IsA("Model") then
					local dist = (v:GetPivot().Position-root.Position).Magnitude
					if dist<resourceaura_range.Value then
						for i,v in pairs(v:GetDescendants()) do
							if v:IsA("BasePart") then
								table.insert(hits,v)
							end
						end
					end
				end
			end
			if #hits>0 then
				hit(hits)
				lasthit=tick()
			end
		end
	end
end)
