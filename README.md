
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local ContentProvider = game:GetService("ContentProvider")
local LP = Players.LocalPlayer

local State = {                      local State = { repeat task.wait() until game:IsLoaded()
local Players,RunService,UIS,TS,Lighting,HS = game:GetService("Players"),game:GetService("RunService"),game:GetService("UserInputService"),game:GetService("TweenService"),game:GetService("Lighting"),game:GetService("HttpService")
local LP = Players.LocalPlayer
local NS,CS,DAS,DAD = 60,30,150,0.2
local LAGGER_SPEED = 15
local speedMode,antiRagdollEnabled,infJumpEnabled = false,false,false
local laggerToggled = false
local lastMoveDir = Vector3.new(0,0,0)
local MOVE_KEYS={[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,
	[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local desyncNormalSpeed = 60
local desyncCarrySpeed = 30
local desyncSpeedMode = false
local medusaCounterEnabled = false
local batCounterEnabled = false
local batCounterWasEnabled = false
local _batCounterV2Was = false
local batCounterV2Enabled = false
local setBatCounterV2Visual = nil
local batCounterV2Debounce = false
local batCounterV2LastUsed = 0
local batCounterV2Conns = {}
local unwalkEnabled = false
local lastHealth,medusaDebounce,medusaLastUsed,dropActive = 100,false,0,false
local autoLeftEnabled,autoRightEnabled = false,false
local autoLeftSetVisual,autoRightSetVisual = nil,nil
local speedLabel = nil
local medusaConns = {}
local batCounterConns = {}
local aimbotEnabled = false
local aimbotMoveEnabled = false
local aimbotSetVisual = nil
local autoSwingEnabled = false
local autoSwingSetVisual = nil
local antiLagEnabled = false
local removeAccessoriesEnabled = false
local descendantConnection = nil
local accessoryConnection = nil
local unwalkSavedAnimate = nil
local _anyKeyListening = false
local setK7AntiLagVisual = nil

-- âââ LOCK IN MODE (Tryhard Animation from Vyse) âââ
local lockInEnabled = false
local lockInHeartbeatConn = nil
local originalAnims = nil
local setLockInVisual = nil

local Anims = {
	idle1    = "rbxassetid://133806214992291",
	idle2    = "rbxassetid://94970088341563",
	walk     = "rbxassetid://707897309",
	run      = "rbxassetid://707861613",
	jump     = "rbxassetid://116936326516985",
	fall     = "rbxassetid://116936326516985",
	climb    = "rbxassetid://116936326516985",
	swim     = "rbxassetid://116936326516985",
	swimidle = "rbxassetid://116936326516985",
}

local function isPackAnim(id)
	if not id then return false end
	for _, v in pairs(Anims) do if v == id then return true end end
	return false
end

local function saveOriginalAnims(char)
	local animate = char:FindFirstChild("Animate")
	if not animate then return end
	local function g(obj) return obj and obj.AnimationId or nil end
	local ids = {
		idle1    = g(animate.idle     and animate.idle.Animation1),
		idle2    = g(animate.idle     and animate.idle.Animation2),
		walk     = g(animate.walk     and animate.walk.WalkAnim),
		run      = g(animate.run      and animate.run.RunAnim),
		jump     = g(animate.jump     and animate.jump.JumpAnim),
		fall     = g(animate.fall     and animate.fall.FallAnim),
		climb    = g(animate.climb    and animate.climb.ClimbAnim),
		swim     = g(animate.swim     and animate.swim.Swim),
		swimidle = g(animate.swimidle and animate.swimidle.SwimIdle),
	}
	if not isPackAnim(ids.walk) then originalAnims = ids end
end

local function applyAnimPack(char)
	local animate = char:FindFirstChild("Animate")
	if not animate then return end
	local function s(obj, id) if obj then obj.AnimationId = id end end
	s(animate.idle     and animate.idle.Animation1,     Anims.idle1)
	s(animate.idle     and animate.idle.Animation2,     Anims.idle2)
	s(animate.walk     and animate.walk.WalkAnim,       Anims.walk)
	s(animate.run      and animate.run.RunAnim,         Anims.run)
	s(animate.jump     and animate.jump.JumpAnim,       Anims.jump)
	s(animate.fall     and animate.fall.FallAnim,       Anims.fall)
	s(animate.climb    and animate.climb.ClimbAnim,     Anims.climb)
	s(animate.swim     and animate.swim.Swim,           Anims.swim)
	s(animate.swimidle and animate.swimidle.SwimIdle,   Anims.swimidle)
end

local function restoreOriginalAnims(char)
	if not originalAnims then return end
	local animate = char:FindFirstChild("Animate")
	if not animate then return end
	local function s(obj, id) if obj and id then obj.AnimationId = id end end
	s(animate.idle     and animate.idle.Animation1,     originalAnims.idle1)
	s(animate.idle     and animate.idle.Animation2,     originalAnims.idle2)
	s(animate.walk     and animate.walk.WalkAnim,       originalAnims.walk)
	s(animate.run      and animate.run.RunAnim,         originalAnims.run)
	s(animate.jump     and animate.jump.JumpAnim,       originalAnims.jump)
	s(animate.fall     and animate.fall.FallAnim,       originalAnims.fall)
	s(animate.climb    and animate.climb.ClimbAnim,     originalAnims.climb)
	s(animate.swim     and animate.swim.Swim,           originalAnims.swim)
	s(animate.swimidle and animate.swimidle.SwimIdle,   originalAnims.swimidle)
	local hum2 = char:FindFirstChildOfClass("Humanoid")
	if hum2 then
		for _, track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end
		hum2:ChangeState(Enum.HumanoidStateType.Running)
	end
end

local function startLockIn()
	if lockInHeartbeatConn then lockInHeartbeatConn:Disconnect(); lockInHeartbeatConn = nil end
	local char = LP.Character
	if char then
		saveOriginalAnims(char)
		applyAnimPack(char)
		local hum2 = char:FindFirstChildOfClass("Humanoid")
		if hum2 then
			for _, track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end
			hum2:ChangeState(Enum.HumanoidStateType.Running)
		end
	end
	lockInHeartbeatConn = RunService.Heartbeat:Connect(function()
		if not lockInEnabled then return end
		local c = LP.Character
		if c then applyAnimPack(c) end
	end)
end

local function stopLockIn()
	if lockInHeartbeatConn then lockInHeartbeatConn:Disconnect(); lockInHeartbeatConn = nil end
	local char = LP.Character
	if char then restoreOriginalAnims(char) end
end

-- âââ K7 ANTI-LAG (no GUI) âââ


-- âââ COLOR THEME STATE âââ
local THEME_MODE = "red" -- "red", "black", "white"

-- Theme palette builder
local function getThemePalette(mode)
	if mode == "black" then
		return {
			BG    = Color3.fromRGB(2, 2, 2),
			BG2   = Color3.fromRGB(6, 6, 6),
			CARD  = Color3.fromRGB(12, 12, 12),
			HOV   = Color3.fromRGB(22, 22, 22),
			ACCENT= Color3.fromRGB(180, 180, 180),
			ACCDIM= Color3.fromRGB(100, 100, 100),
			STROKE= Color3.fromRGB(80, 80, 80),
			W     = Color3.fromRGB(240,240,240),
			DIM   = Color3.fromRGB(70, 70, 70),
			INP   = Color3.fromRGB(8, 8, 8),
			OFF   = Color3.fromRGB(18, 18, 18),
			MB_C_OFF   = Color3.fromRGB(8, 8, 8),
			MB_C_ON    = Color3.fromRGB(90, 90, 90),
			MB_BRD_OFF = Color3.fromRGB(50, 50, 50),
			MB_BRD_ON  = Color3.fromRGB(200, 200, 200),
			MB_TXT_OFF = Color3.fromRGB(150, 150, 150),
			MB_TXT_ON  = Color3.fromRGB(255, 255, 255),
			PB_BG      = Color3.fromRGB(20, 20, 20),
			SECT_CLR   = Color3.fromRGB(160, 160, 160),
		}
	elseif mode == "white" then
		return {
			BG    = Color3.fromRGB(230, 230, 230),
			BG2   = Color3.fromRGB(215, 215, 215),
			CARD  = Color3.fromRGB(200, 200, 200),
			HOV   = Color3.fromRGB(185, 185, 185),
			ACCENT= Color3.fromRGB(30, 30, 30),
			ACCDIM= Color3.fromRGB(80, 80, 80),
			STROKE= Color3.fromRGB(60, 60, 60),
			W     = Color3.fromRGB(10, 10, 10),
			DIM   = Color3.fromRGB(100, 100, 100),
			INP   = Color3.fromRGB(190, 190, 190),
			OFF   = Color3.fromRGB(175, 175, 175),
			MB_C_OFF   = Color3.fromRGB(200, 200, 200),
			MB_C_ON    = Color3.fromRGB(80, 80, 80),
			MB_BRD_OFF = Color3.fromRGB(120, 120, 120),
			MB_BRD_ON  = Color3.fromRGB(20, 20, 20),
			MB_TXT_OFF = Color3.fromRGB(60, 60, 60),
			MB_TXT_ON  = Color3.fromRGB(10, 10, 10),
			PB_BG      = Color3.fromRGB(185, 185, 185),
			SECT_CLR   = Color3.fromRGB(50, 50, 50),
		}
	else -- red (default)
		return {
			BG    = Color3.fromRGB(6, 0, 0),
			BG2   = Color3.fromRGB(10, 1, 1),
			CARD  = Color3.fromRGB(18, 3, 3),
			HOV   = Color3.fromRGB(35, 5, 5),
			ACCENT= Color3.fromRGB(210, 25, 25),
			ACCDIM= Color3.fromRGB(150, 18, 18),
			STROKE= Color3.fromRGB(190, 25, 25),
			W     = Color3.fromRGB(240,240,240),
			DIM   = Color3.fromRGB(100,25,25),
			INP   = Color3.fromRGB(10, 1, 1),
			OFF   = Color3.fromRGB(30, 4, 4),
			MB_C_OFF   = Color3.fromRGB(8, 0, 0),
			MB_C_ON    = Color3.fromRGB(160, 15, 15),
			MB_BRD_OFF = Color3.fromRGB(100, 12, 12),
			MB_BRD_ON  = Color3.fromRGB(220, 40, 40),
			MB_TXT_OFF = Color3.fromRGB(200, 60, 60),
			MB_TXT_ON  = Color3.fromRGB(255, 255, 255),
			PB_BG      = Color3.fromRGB(20, 2, 2),
			SECT_CLR   = Color3.fromRGB(200, 50, 50),
		}
	end
end

local ICON_ID_RED   = "rbxassetid://124615334262719"
local ICON_ID_BLACK = "rbxassetid://98706655274368"
local ICON_ID_WHITE = "rbxassetid://104402968118489"
local function getIconId()
	if THEME_MODE == "black" then return ICON_ID_BLACK
	elseif THEME_MODE == "white" then return ICON_ID_WHITE
	else return ICON_ID_RED end
end
local ICON_ID = ICON_ID_RED

local POS = {
	L1=Vector3.new(-476.48,-6.28,92.73), L2=Vector3.new(-483.12,-4.95,94.80),
	R1=Vector3.new(-476.16,-6.52,25.62), R2=Vector3.new(-483.04,-5.09,23.14),
}
local AP_L_FACE = Vector3.new(-482.25,-4.96,92.09)
local AP_R_FACE = Vector3.new(-482.06,-6.93,35.47)

local KB = {
	DropBrainrot={kb=Enum.KeyCode.X,gp=nil},
	AutoLeft    ={kb=Enum.KeyCode.Z,gp=nil},
	AutoRight   ={kb=Enum.KeyCode.C,gp=nil},
	AimBot      ={kb=Enum.KeyCode.E,gp=nil},
	TPFloor     ={kb=Enum.KeyCode.F,gp=nil},
	GuiHide     ={kb=Enum.KeyCode.LeftControl,gp=nil},
	SpeedToggle ={kb=Enum.KeyCode.Q,gp=nil},
	LaggerToggle={kb=Enum.KeyCode.R,gp=nil}
}

local Steal = {
	AutoStealEnabled=false, StealRadius=20, StealDuration=0.25,
	Data={}, plotCache={}, plotCacheTime={}, cachedPrompts={}, promptCacheTime=0,
}
local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local Conns = {autoSteal=nil,antiRag=nil,anchor={},progress=nil,aimbot=nil,batCounter=nil}
local PLOT_CACHE_DURATION = 2
local PROMPT_CACHE_REFRESH = 0.15
local STEAL_COOLDOWN = 0.1
local MEDUSA_COOLDOWN = 25
local BAT_COUNTER_COOLDOWN = 0
local batCounterDebounce = false
local batCounterLastUsed = 0
local progressRadLbl,progressFill,progressPct
local modeValLbl
local _aimTarget = nil
local _aimLastScan = 0

local VYSE_AIMBOT_SPEED = 56.5
local VYSE_HIT_DIST = 5
local SWING_COOLDOWN = 0.08
local hittingCooldown = false

local desyncEnabled = false
local setDesyncVisual = nil
local desyncActive = false
local desyncSpeedConn = nil
local G_desyncAnimate = nil

local fpsBoostEnabled = false

-- âââ KYRIE DESYNC âââ
local function applyDesyncFFlags(normalSpd, carrySpd)
	pcall(function()
		local setff = syn and syn.set_fflag or (setfflag or nil)
		if setff then
			local flags = {"FFlagDisableGroupGameService","FFlagDisableGroupPassService","FFlagDisableGroupBadgeService",
				"FFlagDisableGroupCurrencyService","FFlagDisableGroupDataStoreService","FFlagDisableGroupAnalyticsService",
				"FFlagDisableGroupLogService","FFlagDisableGroupReportService","FFlagDisableGroupRankService",
				"FFlagDisableGroupMembershipService","FFlagDisableGroupWallService","FFlagDisableGroupAuditService",
				"FFlagDisableGroupRelationshipService","FFlagDisableGroupNotificationService","FFlagDisableGroupPermissionService",
				"FFlagDisableGroupRoleService","FFlagDisableGroupSearchService","FFlagDisableGroupEventService",
				"FFlagDisableGroupInviteService","FFlagDisableGroupShoutService"}
			for _,f in ipairs(flags) do pcall(function() setff(f,"true") end) end
			pcall(function() setff("DFIntUserCameraMaxZoomDistance","2000") end)
		end
	end)
	if desyncSpeedConn then desyncSpeedConn:Disconnect(); desyncSpeedConn = nil end
	desyncSpeedConn = RunService.Heartbeat:Connect(function()
		if not desyncEnabled then if desyncSpeedConn then desyncSpeedConn:Disconnect(); desyncSpeedConn=nil end; return end
		local c = LP.Character; if not c then return end
		local hrp = c:FindFirstChild("HumanoidRootPart"); if not hrp then return end
		local hum = c:FindFirstChildOfClass("Humanoid"); if not hum then return end
		local md = hum.MoveDirection
		local targetSpd = desyncSpeedMode and desyncCarrySpeed or desyncNormalSpeed
		if md.Magnitude > 0 then hrp.AssemblyLinearVelocity = Vector3.new(md.X*targetSpd, hrp.AssemblyLinearVelocity.Y, md.Z*targetSpd) end
	end)
end

local function enableDesync()
	pcall(function() if raknet then raknet.desync(true) end end)
	local c = LP.Character; if not c then return end
	local hum = c:FindFirstChildOfClass("Humanoid")
	if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end; hum.WalkSpeed=0; hum.JumpPower=0 end
	local anim = c:FindFirstChild("Animate")
	if anim then G_desyncAnimate = anim:Clone(); anim:Destroy() end
	applyDesyncFFlags(desyncNormalSpeed, desyncCarrySpeed)
end

local function disableDesync()
	pcall(function() if raknet then raknet.desync(false) end end)
	if desyncSpeedConn then desyncSpeedConn:Disconnect(); desyncSpeedConn = nil end
	local c = LP.Character
	if c then
		local hum = c:FindFirstChildOfClass("Humanoid"); if hum then hum.WalkSpeed=16; hum.JumpPower=50 end
		if G_desyncAnimate then G_desyncAnimate:Clone().Parent=c; G_desyncAnimate=nil end
	end
end

-- âââ COLT INSTA STEAL âââ
local function isMyPlotByName(plotName)
	local ct = tick()
	if Steal.plotCache[plotName] and (ct-(Steal.plotCacheTime[plotName] or 0))<PLOT_CACHE_DURATION then return Steal.plotCache[plotName] end
	local plots = workspace:FindFirstChild("Plots")
	if not plots then Steal.plotCache[plotName]=false;Steal.plotCacheTime[plotName]=ct;return false end
	local plot = plots:FindFirstChild(plotName)
	if not plot then Steal.plotCache[plotName]=false;Steal.plotCacheTime[plotName]=ct;return false end
	local sign = plot:FindFirstChild(">Z-J('")
	if sign then
		local yb = sign:FindFirstChild("YourBase")
		if yb and yb:IsA("BillboardGui") then
			local r = yb.Enabled==true;Steal.plotCache[plotName]=r;Steal.plotCacheTime[plotName]=ct;return r
		end
	end
	Steal.plotCache[plotName]=false;Steal.plotCacheTime[plotName]=ct;return false
end

local function findNearestPrompt()
	local char = LP.Character;if not char then return nil end
	local root = char:FindFirstChild("HumanoidRootPart");if not root then return nil end
	local ct = tick()
	if ct-Steal.promptCacheTime<PROMPT_CACHE_REFRESH and #Steal.cachedPrompts>0 then
		local np,nd = nil,math.huge
		for _,data in ipairs(Steal.cachedPrompts) do
			if data.spawn then
				local dist = (data.spawn.Position-root.Position).Magnitude
				if dist<=Steal.StealRadius and dist<nd then np=data.prompt;nd=dist end
			end
		end
		if np then return np end
	end
	Steal.cachedPrompts={};Steal.promptCacheTime=ct
	local plots = workspace:FindFirstChild("Plots");if not plots then return nil end
	local np,nd = nil,math.huge
	for _,plot in ipairs(plots:GetChildren()) do
		if isMyPlotByName(plot.Name) then continue end
		local pods = plot:FindFirstChild("AnimalPodiums");if not pods then continue end
		for _,pod in ipairs(pods:GetChildren()) do
			pcall(function()
				local base = pod:FindFirstChild("Base");local sp = base and base:FindFirstChild("Spawn")
				if sp then
					local att = sp:FindFirstChild("PromptAttachment")
					if att then
						for _,child in ipairs(att:GetChildren()) do
							if child:IsA("ProximityPrompt") then
								local dist = (sp.Position-root.Position).Magnitude
								table.insert(Steal.cachedPrompts,{prompt=child,spawn=sp,name=pod.Name})
								if dist<=Steal.StealRadius and dist<nd then np=child;nd=dist end
								break
							end
						end
					end
				end
			end)
		end
	end
	return np
end

local function resetProgressBar()
	if progressPct then progressPct.Text="0%" end
	if progressFill then progressFill.Size=UDim2.new(0,0,1,0) end
end

local function executeSteal(prompt)
	local ct = tick()
	if ct-lastStealTick<STEAL_COOLDOWN then return end
	if isStealing then return end
	if not Steal.Data[prompt] then
		Steal.Data[prompt]={hold={},trigger={},ready=true}
		pcall(function()
			if getconnections then
				for _,c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(Steal.Data[prompt].hold,c.Function) end end
				for _,c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(Steal.Data[prompt].trigger,c.Function) end end
			else Steal.Data[prompt].useFallback=true end
		end)
	end
	local data = Steal.Data[prompt];if not data.ready then return end
	data.ready=false;isStealing=true;stealStartTime=ct;lastStealTick=ct
	if Conns.progress then Conns.progress:Disconnect() end
	Conns.progress = RunService.Heartbeat:Connect(function()
		if not isStealing then Conns.progress:Disconnect();return end
		local prog = math.clamp((tick()-stealStartTime)/Steal.StealDuration,0,1)
		if progressFill then progressFill.Size=UDim2.new(prog,0,1,0) end
		if progressPct then progressPct.Text=math.floor(prog*100).."%"  end
	end)
	task.spawn(function()
		local ok = false
		pcall(function()
			if not data.useFallback then
				for _,fn in ipairs(data.hold) do task.spawn(fn) end
				task.wait(Steal.StealDuration)
				for _,fn in ipairs(data.trigger) do task.spawn(fn) end
				ok=true
			end
		end)
		if not ok and fireproximityprompt then pcall(function() fireproximityprompt(prompt);ok=true end) end
		if not ok then pcall(function() prompt:InputHoldBegin();task.wait(Steal.StealDuration);prompt:InputHoldEnd() end) end
		task.wait(Steal.StealDuration*0.3)
		if Conns.progress then Conns.progress:Disconnect() end
		resetProgressBar()
		task.wait(0.05);data.ready=true;isStealing=false
	end)
end

local function startAutoSteal()
	if Conns.autoSteal then return end
	Conns.autoSteal = RunService.Heartbeat:Connect(function()
		if not Steal.AutoStealEnabled or isStealing then return end
		local p = findNearestPrompt();if p then executeSteal(p) end
	end)
end

local function stopAutoSteal()
	if Conns.autoSteal then Conns.autoSteal:Disconnect();Conns.autoSteal=nil end
	isStealing=false;lastStealTick=0
	Steal.plotCache={};Steal.plotCacheTime={};Steal.cachedPrompts={}
	resetProgressBar()
end

RunService.Stepped:Connect(function()
	for _,p in ipairs(Players:GetPlayers()) do
		if p~=LP and p.Character then
			for _,part in ipairs(p.Character:GetDescendants()) do
				if part:IsA("BasePart") then part.CanCollide=false end
			end
		end
	end
end)

RunService.RenderStepped:Connect(function()
	local char=LP.Character;if not char then return end
	local hum=char:FindFirstChildOfClass("Humanoid")
	local hrp=char:FindFirstChild("HumanoidRootPart")
	if not hum or not hrp then return end
	if not desyncEnabled then
		if not(autoLeftEnabled or autoRightEnabled) and not(aimbotEnabled and aimbotMoveEnabled) then
			local md=hum.MoveDirection
			local spd=laggerToggled and LAGGER_SPEED or (speedMode and CS or NS)
			if md.Magnitude>0 then
				lastMoveDir=md
				hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
			elseif antiRagdollEnabled and lastMoveDir.Magnitude>0 then
				local anyHeld=false
				for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true;break end end
				if anyHeld then hrp.Velocity=Vector3.new(lastMoveDir.X*spd,hrp.Velocity.Y,lastMoveDir.Z*spd) end
			end
		end
	end
	if speedLabel then speedLabel.Text=string.format("Speed: %.1f",Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude) end
end)

local alConn,arConn = nil,nil
local alPhase,arPhase = 1,1

local function stopAutoLeft()
	if alConn then alConn:Disconnect();alConn=nil end;alPhase=1
	local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
end
local function stopAutoRight()
	if arConn then arConn:Disconnect();arConn=nil end;arPhase=1
	local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
end

local function startAutoLeft()
	if alConn then alConn:Disconnect() end;alPhase=1
	alConn=RunService.Heartbeat:Connect(function()
		if not autoLeftEnabled then return end
		local char=LP.Character;if not char then return end
		local hrp=char:FindFirstChild("HumanoidRootPart")
		local hum=char:FindFirstChildOfClass("Humanoid")
		if not hrp or not hum then return end
		local spd=NS
		if alPhase==1 then
			local tgt=Vector3.new(POS.L1.X,hrp.Position.Y,POS.L1.Z)
			if (tgt-hrp.Position).Magnitude<1 then alPhase=2; return end
			local d=POS.L1-hrp.Position; local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
		elseif alPhase==2 then
			local tgt=Vector3.new(POS.L2.X,hrp.Position.Y,POS.L2.Z)
			if (tgt-hrp.Position).Magnitude<1 then
				hum:Move(Vector3.zero,false);hrp.Velocity=Vector3.zero
				autoLeftEnabled=false;if alConn then alConn:Disconnect();alConn=nil end
				alPhase=1;if autoLeftSetVisual then autoLeftSetVisual(false) end
				local dir=Vector3.new(AP_L_FACE.X,hrp.Position.Y,AP_L_FACE.Z)-hrp.Position
				if dir.Magnitude>0.01 then hrp.CFrame=CFrame.new(hrp.Position,hrp.Position+Vector3.new(dir.X,0,dir.Z).Unit) end
				return
			end
			local d=POS.L2-hrp.Position; local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
		end
	end)
end

local function startAutoRight()
	if arConn then arConn:Disconnect() end;arPhase=1
	arConn=RunService.Heartbeat:Connect(function()
		if not autoRightEnabled then return end
		local char=LP.Character;if not char then return end
		local hrp=char:FindFirstChild("HumanoidRootPart")
		local hum=char:FindFirstChildOfClass("Humanoid")
		if not hrp or not hum then return end
		local spd=NS
		if arPhase==1 then
			local tgt=Vector3.new(POS.R1.X,hrp.Position.Y,POS.R1.Z)
			if (tgt-hrp.Position).Magnitude<1 then arPhase=2; return end
			local d=POS.R1-hrp.Position; local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
		elseif arPhase==2 then
			local tgt=Vector3.new(POS.R2.X,hrp.Position.Y,POS.R2.Z)
			if (tgt-hrp.Position).Magnitude<1 then
				hum:Move(Vector3.zero,false);hrp.Velocity=Vector3.zero
				autoRightEnabled=false;if arConn then arConn:Disconnect();arConn=nil end
				arPhase=1;if autoRightSetVisual then autoRightSetVisual(false) end
				local dir=Vector3.new(AP_R_FACE.X,hrp.Position.Y,AP_R_FACE.Z)-hrp.Position
				if dir.Magnitude>0.01 then hrp.CFrame=CFrame.new(hrp.Position,hrp.Position+Vector3.new(dir.X,0,dir.Z).Unit) end
				return
			end
			local d=POS.R2-hrp.Position; local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
		end
	end)
end

local function getSpeedLabelColor()
	if THEME_MODE == "black" then return Color3.fromRGB(200,200,200)
	elseif THEME_MODE == "white" then return Color3.fromRGB(30,30,30)
	else return Color3.fromRGB(180,20,40) end
end

local function setupSpeedIndicator(char)
	local head = char:WaitForChild("Head",5);if not head then return end
	local oldBB = head:FindFirstChild("ColtSpeedBB"); if oldBB then oldBB:Destroy() end
	local bb = Instance.new("BillboardGui",head)
	bb.Name = "ColtSpeedBB"
	bb.Size=UDim2.new(0,140,0,25);bb.StudsOffset=Vector3.new(0,3,0);bb.AlwaysOnTop=true
	speedLabel = Instance.new("TextLabel",bb)
	speedLabel.Size=UDim2.new(1,0,1,0);speedLabel.BackgroundTransparency=1
	speedLabel.Text="Speed: 0";speedLabel.TextColor3=getSpeedLabelColor()
	speedLabel.Font=Enum.Font.GothamBold;speedLabel.TextScaled=true
	speedLabel.TextStrokeTransparency=0;speedLabel.TextStrokeColor3=Color3.fromRGB(0,0,0)
end

local otherSpeedLabels = {}
local function setupOtherSpeedLabel(player)
	if player == LP then return end
	local function onChar(char)
		task.spawn(function()
			local head = char:WaitForChild("Head", 5)
			if not head then return end
			local oldBB = head:FindFirstChild("CursedSpeedBB")
			if oldBB then oldBB:Destroy() end
			local bb = Instance.new("BillboardGui", head)
			bb.Name = "CursedSpeedBB"
			bb.Size = UDim2.new(0, 120, 0, 20)
			bb.StudsOffset = Vector3.new(0, 3.2, 0)
			bb.AlwaysOnTop = true
			local lbl = Instance.new("TextLabel", bb)
			lbl.Size = UDim2.new(1, 0, 1, 0)
			lbl.BackgroundTransparency = 1
			lbl.Text = "0"
			lbl.TextColor3 = getSpeedLabelColor()
			lbl.Font = Enum.Font.GothamBold
			lbl.TextScaled = true
			lbl.TextStrokeTransparency = 0
			lbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
			otherSpeedLabels[player] = {lbl = lbl, char = char}
		end)
	end
	if player.Character then onChar(player.Character) end
	player.CharacterAdded:Connect(onChar)
end

Players.PlayerAdded:Connect(function(p) if p ~= LP then setupOtherSpeedLabel(p) end end)
Players.PlayerRemoving:Connect(function(p) otherSpeedLabels[p] = nil end)
for _, p in ipairs(Players:GetPlayers()) do if p ~= LP then setupOtherSpeedLabel(p) end end

RunService.Heartbeat:Connect(function()
	local now2=tick()
	for p, data in pairs(otherSpeedLabels) do
		if data.lbl and data.lbl.Parent and data.char then
			local hrp = data.char:FindFirstChild("HumanoidRootPart")
			if hrp then
				local dt=now2-(data.lastTick or now2)
				if dt>0 and data.lastPos then
					local delta=hrp.Position-data.lastPos
					local spd=Vector3.new(delta.X,0,delta.Z).Magnitude/dt
					data.lbl.Text=string.format("%.1f",spd)
				end
				data.lastPos=hrp.Position
				data.lastTick=now2
			end
		else
			otherSpeedLabels[p] = nil
		end
	end
end)

local function startAntiRagdoll()
	if Conns.antiRag then return end
	Conns.antiRag = RunService.Heartbeat:Connect(function()
		local char = LP.Character;if not char then return end
		local hum = char:FindFirstChildOfClass("Humanoid");local root=char:FindFirstChild("HumanoidRootPart")
		if hum then
			local st = hum:GetState()
			if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then
				hum:ChangeState(Enum.HumanoidStateType.Running)
				workspace.CurrentCamera.CameraSubject=hum
				pcall(function() local pm=LP.PlayerScripts:FindFirstChild(">Vz(v^");if pm then require(pm:FindFirstChild("ControlModule")):Enable() end end)
				if root then root.Velocity=Vector3.zero;root.RotVelocity=Vector3.zero end
			end
		end
		for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled=true end end
	end)
end

local function stopAntiRagdoll()
	if Conns.antiRag then Conns.antiRag:Disconnect();Conns.antiRag=nil end
end

local IJ_Conn = nil
local function startInfiniteJump()
	if IJ_Conn then IJ_Conn:Disconnect() end
	IJ_Conn = RunService.Heartbeat:Connect(function()
		if not infJumpEnabled then return end
		local char=LP.Character; if not char then return end
		local root=char:FindFirstChild("HumanoidRootPart")
		local hum=char:FindFirstChildOfClass("Humanoid")
		if not root or not hum then return end
		if root.AssemblyLinearVelocity.Y<-120 then
			root.AssemblyLinearVelocity=Vector3.new(root.AssemblyLinearVelocity.X,-120,root.AssemblyLinearVelocity.Z)
		end
		if hum.FloorMaterial==Enum.Material.Air then return end
		if not (UIS:IsKeyDown(Enum.KeyCode.Space) or UIS:IsKeyDown(Enum.KeyCode.ButtonA)) then return end
		local st=hum:GetState()
		if st==Enum.HumanoidStateType.Jumping or st==Enum.HumanoidStateType.Freefall then return end
		root.AssemblyLinearVelocity=Vector3.new(root.AssemblyLinearVelocity.X,55,root.AssemblyLinearVelocity.Z)
	end)
	UIS.JumpRequest:Connect(function()
		if not infJumpEnabled then return end
		local char=LP.Character; if not char then return end
		local root=char:FindFirstChild("HumanoidRootPart")
		if root then root.AssemblyLinearVelocity=Vector3.new(root.AssemblyLinearVelocity.X,55,root.AssemblyLinearVelocity.Z) end
	end)
end

local function stopInfiniteJump()
	if IJ_Conn then IJ_Conn:Disconnect(); IJ_Conn=nil end
end

local function startUnwalk()
	local c=LP.Character;if not c then return end
	local hum=c:FindFirstChildOfClass("Humanoid")
	if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end
	local anim=c:FindFirstChild("Animate")
	if anim then unwalkSavedAnimate=anim:Clone();anim:Destroy() end
end

local function stopUnwalk()
	local c=LP.Character
	if c and unwalkSavedAnimate then unwalkSavedAnimate:Clone().Parent=c;unwalkSavedAnimate=nil end
end

local function runTPFloor()
	pcall(function()
		local char=LP.Character;if not char then return end
		local hrp=char:FindFirstChild("HumanoidRootPart");if not hrp then return end
		local rp=RaycastParams.new();rp.FilterDescendantsInstances={char};rp.FilterType=Enum.RaycastFilterType.Exclude
		local res=workspace:Raycast(hrp.Position,Vector3.new(0,-1000,0),rp)
		if res then hrp.CFrame=CFrame.new(res.Position+Vector3.new(0,hrp.Size.Y/2+0.5,0)); hrp.AssemblyLinearVelocity=Vector3.zero end
	end)
end

-- Instant Reset removed

local _dropConns = {}
local dropBrainrotActive = false
local DROP_AUTO_OFF_DELAY = 0.15

local function stopDropBrainrot()
	dropBrainrotActive = false
	for _,cn in ipairs(_dropConns) do pcall(function() cn:Disconnect() end) end; _dropConns = {}
end

local function runDrop()
	if dropBrainrotActive then return end; dropBrainrotActive = true
	task.spawn(function()
		local colConn = RunService.Stepped:Connect(function()
			if not dropBrainrotActive then return end
			for _,p in ipairs(Players:GetPlayers()) do
				if p~=LP and p.Character then
					for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end
				end
			end
		end)
		table.insert(_dropConns, colConn)
		task.spawn(function()
			while dropBrainrotActive do
				RunService.Heartbeat:Wait()
				local c=LP.Character; local root=c and c:FindFirstChild("HumanoidRootPart")
				if not root then continue end
				local vel=root.Velocity
				root.Velocity=vel*10000+Vector3.new(0,10000,0)
				RunService.RenderStepped:Wait()
				if root and root.Parent then root.Velocity=vel end
				RunService.Stepped:Wait()
				if root and root.Parent then root.Velocity=vel+Vector3.new(0,0.1,0) end
			end
		end)
		task.wait(DROP_AUTO_OFF_DELAY); stopDropBrainrot()
	end)
end

local function applyFPSBoost()
	pcall(function() setfpscap(999999999) end)
	local function pO(v) pcall(function()
		if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic
		elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance
		elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0
		elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1
		elseif v:IsA("SpecialMesh") then v.TextureId=""
		elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false
		elseif v:IsA("SurfaceAppearance") or v:IsA("1^&Uj{") then v:Destroy()
		elseif v:IsA("Attachment") then v.Visible=false end
	end) end
	for _,v in pairs(workspace:GetDescendants()) do pO(v) end
	pcall(function()
		local L=game:GetService("Lighting")
		for _,v in pairs(L:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end
		pcall(function() sethiddenproperty(L,"Technology",Enum.Technology.Legacy) end)
		L.GlobalShadows=false; L.FogEnd=9e9; L.Brightness=0
		local ter=workspace:FindFirstChildOfClass("Terrain")
		if ter then pcall(function() sethiddenproperty(ter,"Decoration",false) end); ter.WaterReflectance=0; ter.WaterTransparency=0.7; ter.WaterWaveSize=0; ter.WaterWaveSpeed=0 end
	end)
	workspace.DescendantAdded:Connect(function(v) if fpsBoostEnabled then task.spawn(pO,v) end end)
end

local function findAnyTool()
	local c=LP.Character
	if c then for _,v in ipairs(c:GetChildren()) do if v:IsA("Tool") then return v end end end
	local bp=LP:FindFirstChildOfClass("Backpack")
	if bp then for _,v in ipairs(bp:GetChildren()) do if v:IsA("Tool") then return v end end end
	return nil
end

local function getClosestPlayer()
	local char=LP.Character; if not char then return nil,math.huge end
	local hrp=char:FindFirstChild("HumanoidRootPart"); if not hrp then return nil,math.huge end
	local cp,cd=nil,math.huge
	for _,p in pairs(Players:GetPlayers()) do
		if p~=LP and p.Character then
			local tr=p.Character:FindFirstChild("HumanoidRootPart")
			local ph=p.Character:FindFirstChildOfClass("Humanoid")
			if tr and ph and ph.Health>0 then
				local d=(hrp.Position-tr.Position).Magnitude
				if d<cd then cd=d; cp=p end
			end
		end
	end
	return cp,cd
end

local function tryHitBat()
	if hittingCooldown then return end; hittingCooldown=true
	pcall(function()
		local c=LP.Character; if not c then return end
		local hum2=c:FindFirstChildOfClass("Humanoid")
		local tool=findAnyTool()
		if tool then
			if tool.Parent~=c and hum2 then pcall(function() hum2:EquipTool(tool) end) end
			local remote=tool:FindFirstChildOfClass("Eé¨µ/z{")
			if remote then pcall(function() remote:FireServer() end)
			else pcall(function() tool:Activate() end) end
		end
	end)
	task.delay(SWING_COOLDOWN, function() hittingCooldown=false end)
end

local function startBatAimbot()
	if Conns.aimbot then return end
	Conns.aimbot = RunService.Heartbeat:Connect(function()
		if not aimbotEnabled then return end
		local c=LP.Character; if not c then return end
		local root=c:FindFirstChild("HumanoidRootPart"); if not root then return end
		local hum2=c:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
		local target,dist=getClosestPlayer()
		if target and target.Character then
			local tr=target.Character:FindFirstChild("HumanoidRootPart")
			if tr then
				local fp=tr.Position+tr.CFrame.LookVector*1.5
				local dir=(fp-root.Position).Unit
				root.AssemblyLinearVelocity=Vector3.new(dir.X*VYSE_AIMBOT_SPEED,dir.Y*VYSE_AIMBOT_SPEED,dir.Z*VYSE_AIMBOT_SPEED)
				if dist<=VYSE_HIT_DIST and autoSwingEnabled then tryHitBat() end
			end
		else root.AssemblyLinearVelocity=Vector3.zero end
	end)
end

local function stopBatAimbot()
	if Conns.aimbot then Conns.aimbot:Disconnect(); Conns.aimbot=nil end
	local c=LP.Character; local root=c and c:FindFirstChild("HumanoidRootPart")
	if root then root.AssemblyLinearVelocity=Vector3.zero end; hittingCooldown=false
end

local function enableAimbot()
	if autoLeftEnabled then autoLeftEnabled=false;if autoLeftSetVisual then autoLeftSetVisual(false) end;stopAutoLeft() end
	if autoRightEnabled then autoRightEnabled=false;if autoRightSetVisual then autoRightSetVisual(false) end;stopAutoRight() end
	local char=LP.Character
	if char then
		local hum=char:FindFirstChildOfClass("Humanoid")
		local bp=LP:FindFirstChild("Backpack")
		local bat=(bp and bp:FindFirstChild("Bat")) or char:FindFirstChild("Bat")
		if bat and hum then hum:EquipTool(bat) end
	end
	aimbotMoveEnabled=true
	startBatAimbot()
end

local function disableAimbot(autoSwingRow)
	aimbotMoveEnabled=false
	stopBatAimbot()
	local char=LP.Character
	local hum=char and char:FindFirstChildOfClass("Humanoid")
	local root=char and char:FindFirstChild("HumanoidRootPart")
	if hum then hum.AutoRotate=true end
	if root then
		root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0)
	end
	autoSwingEnabled=false
	if autoSwingSetVisual then autoSwingSetVisual(false) end
	if autoSwingRow then autoSwingRow.Visible=false end
end

local BAT_COUNTER_SLAP_LIST={"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
local function findBatForCounter()
	local c=LP.Character; if not c then return nil end
	local bp=LP:FindFirstChildOfClass("Backpack")
	for _,name in ipairs(BAT_COUNTER_SLAP_LIST) do
		local t=c:FindFirstChild(name) or (bp and bp:FindFirstChild(name))
		if t then return t end
	end
	for _,ch in ipairs(c:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end
	if bp then for _,ch in ipairs(bp:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end end
	return nil
end

local function swingBatForCounter(bat,char)
	local hum2=char:FindFirstChildOfClass("Humanoid")
	if bat.Parent~=char then if hum2 then pcall(function() hum2:EquipTool(bat) end) end; task.wait(0.05) end
	local remote=bat:FindFirstChildOfClass("Eé¨µ/z{") or bat:FindFirstChildOfClass("RemoteFunction")
	if remote and remote:IsA("Eé¨µ/z{") then
		pcall(function() remote:FireServer() end); task.wait(0.15); pcall(function() remote:FireServer() end)
	else pcall(function() bat:Activate() end); task.wait(0.15); pcall(function() bat:Activate() end) end
end

local setBatCounterVisual = nil

local function startBatCounter()
	if Conns.batCounter then return end
	Conns.batCounter = RunService.Heartbeat:Connect(function()
		if not batCounterEnabled then return end
		if batCounterDebounce then return end
		local char=LP.Character; if not char then return end
		local hum2=char:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
		local st=hum2:GetState()
		local isRagdolled=st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown
		if isRagdolled then
			batCounterDebounce=true
			task.spawn(function()
				local bat=findBatForCounter()
				if bat then swingBatForCounter(bat,char) end
				task.wait(0.5); batCounterDebounce=false
			end)
		end
	end)
end

local function stopBatCounter()
	if Conns.batCounter then Conns.batCounter:Disconnect(); Conns.batCounter=nil end
	batCounterDebounce=false
end

local function findMedusa()
	local char = LP.Character;if not char then return nil end
	for _,t in ipairs(char:GetChildren()) do if t:IsA("Tool") and t.Name:lower():find("medusa") then return t end end
	local bp = LP:FindFirstChild("Backpack")
	if bp then for _,t in ipairs(bp:GetChildren()) do if t:IsA("Tool") and t.Name:lower():find("medusa") then return t end end end
end

local function useMedusa()
	if medusaDebounce or tick()-medusaLastUsed<MEDUSA_COOLDOWN then return end
	local char = LP.Character;if not char then return end
	medusaDebounce=true
	local med = findMedusa()
	if med then
		if med.Parent~=char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:EquipTool(med) end end
		pcall(function() med:Activate() end)
		medusaLastUsed=tick()
	end
	medusaDebounce=false
end

local function onAnchorChanged(part)
	return part:GetPropertyChangedSignal("Anchored"):Connect(function()
		if part.Anchored and part.Transparency==1 and medusaCounterEnabled then useMedusa() end
	end)
end

local function setupMedusa(char)
	for _,c in pairs(medusaConns) do pcall(function() c:Disconnect() end) end;medusaConns={}
	if not char then return end
	for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(medusaConns,onAnchorChanged(part)) end end
	table.insert(medusaConns,char.DescendantAdded:Connect(function(part)
		if part:IsA("BasePart") then table.insert(medusaConns,onAnchorChanged(part)) end
	end))
end

local autoTPEnabled=false
local autoTPHeight=20
local autoTPConn=nil
local setAutoTPVisual=nil

local function startAutoTP()
	if autoTPConn then task.cancel(autoTPConn);autoTPConn=nil end
	autoTPConn=task.spawn(function()
		while autoTPEnabled do
			task.wait(0.1)
			pcall(function()
				local char=LP.Character;if not char then return end
				local hrp=char:FindFirstChild("HumanoidRootPart");if not hrp then return end
				local hum2=char:FindFirstChildOfClass("Humanoid");if not hum2 then return end
				if hum2.FloorMaterial~=Enum.Material.Air then return end
				if hrp.Position.Y<autoTPHeight then return end
				hrp.CFrame=CFrame.new(hrp.Position.X,-8.80,hrp.Position.Z)
				hrp.AssemblyLinearVelocity=Vector3.zero
			end)
		end
	end)
end

local function stopAutoTP()
	autoTPEnabled=false
	if autoTPConn then task.cancel(autoTPConn);autoTPConn=nil end
end

local optimizerThreads = {}
local optimizerConns   = {}
local BK_FFLAGS = {
	["DFIntTaskSchedulerTargetFps"]=999,
	["DFIntTaskSchedulerTargetFpsMax"]=999,
	["FFlagDebugGraphicsPreferVulkan"]=true,
	["FFlagDisablePostFx"]=true,
	["FIntRenderShadowIntensity"]=0,
	["DFIntParticleMaxCount"]=0,
	["DFIntGlobalPointLightMaxCount"]=0,
}

local bkCleanedChars = {}
local function bkCleanChar(char)
	if not char then return end
	pcall(function()
		for _,child in ipairs(char:GetChildren()) do
			if child:IsA("Accessory") or child:IsA("Hat") then child:Destroy()
			elseif child:IsA("Shirt") or child:IsA("Pants") or child:IsA("ShirtGraphic") then child:Destroy()
			elseif child:IsA("BodyColors") then child:Destroy()
			elseif child:IsA("CharacterMesh") then child:Destroy()
			end
		end
		for _,child in ipairs(char:GetDescendants()) do
			pcall(function()
				if child:IsA("ParticleEmitter") or child:IsA("Trail") or child:IsA("Beam") then child:Destroy()
				elseif child:IsA("PointLight") or child:IsA("SpotLight") or child:IsA("SurfaceLight") then child:Destroy()
				elseif child:IsA("Fire") or child:IsA("Smoke") or child:IsA("Sparkles") then child:Destroy()
				elseif child:IsA("BasePart") then
					child.CastShadow=false;child.Material=Enum.Material.Plastic;child.Reflectance=0
				end
			end)
		end
	end)
	bkCleanedChars[char]=true
end

local function bkCleanBackpack(player)
	pcall(function()
		local bp=player:FindFirstChild("Backpack"); if not bp then return end
		for _,tool in ipairs(bp:GetChildren()) do
			if tool:IsA("Tool") then
				for _,desc in ipairs(tool:GetDescendants()) do
					pcall(function()
						if desc:IsA("ParticleEmitter") or desc:IsA("Trail") or desc:IsA("Beam")
							or desc:IsA("PointLight") or desc:IsA("SpotLight") or desc:IsA("SurfaceLight")
							or desc:IsA("Fire") or desc:IsA("Smoke") or desc:IsA("Sparkles") then
							desc:Destroy()
						end
					end)
				end
			end
		end
	end)
end

local function enableAntiLag()
	if antiLagEnabled then return end
	antiLagEnabled=true
	bkCleanedChars={}
	pcall(function() for flag,val in pairs(BK_FFLAGS) do pcall(function() setfflag(flag,tostring(val)) end) end end)
	pcall(function() local r=settings().Rendering; r.QualityLevel=Enum.QualityLevel.Level01 end)
	pcall(function()
		Lighting.GlobalShadows=false;Lighting.Brightness=3;Lighting.FogEnd=9e9
		for _,e in ipairs(Lighting:GetChildren()) do
			if e:IsA("PostEffect") or e:IsA("Atmosphere") then pcall(function() e:Destroy() end) end
		end
	end)
	pcall(function() setfpscap(999) end)
	for _,p in ipairs(Players:GetPlayers()) do
		bkCleanChar(p.Character)
		bkCleanBackpack(p)
	end
end

local function disableAntiLag()
	antiLagEnabled=false
	bkCleanedChars={}
	for _,c in ipairs(optimizerConns) do pcall(function() c:Disconnect() end) end
	optimizerConns={}
	for _,t in ipairs(optimizerThreads) do pcall(function() task.cancel(t) end) end
	optimizerThreads={}
	if descendantConnection then descendantConnection:Disconnect();descendantConnection=nil end
end

local function enableRemoveAccessories()
	removeAccessoriesEnabled=true
	for _,p in pairs(Players:GetPlayers()) do
		if p.Character then for _,obj in ipairs(p.Character:GetDescendants()) do if obj:IsA("Accessory") or obj:IsA("Hat") then obj:Destroy() end end end
	end
end

local function disableRemoveAccessories()
	removeAccessoriesEnabled=false
	if accessoryConnection then accessoryConnection:Disconnect();accessoryConnection=nil end
end

LP.CharacterAdded:Connect(function(char)
	lastHealth=100;task.wait(0.5)
	setupSpeedIndicator(char)
	if medusaCounterEnabled then setupMedusa(char) end
	if batCounterEnabled then startBatCounter() end
	if unwalkEnabled then task.wait(0.5);startUnwalk() end
	if lockInEnabled then
		task.wait(0.3)
		saveOriginalAnims(char)
		applyAnimPack(char)
	end
end)
if LP.Character then setupSpeedIndicator(LP.Character) end

local function saveConfig()
	local function ks(e) return {kb=e.kb and e.kb.Name or nil,gp=e.gp and e.gp.Name or nil} end
	local cfg = {
		normalSpeed=NS,carrySpeed=CS,
		dropBrainrotKey=ks(KB.DropBrainrot),autoLeftKey=ks(KB.AutoLeft),autoRightKey=ks(KB.AutoRight),
		aimbotKey=ks(KB.AimBot),laggerToggleKey=ks(KB.LaggerToggle),tpFloorKey=ks(KB.TPFloor),guiHideKey=ks(KB.GuiHide),
		speedToggleKey=ks(KB.SpeedToggle),
		grabRadius=Steal.StealRadius,stealDuration=Steal.StealDuration,
		antiRagdoll=antiRagdollEnabled,autoStealEnabled=Steal.AutoStealEnabled,
		infiniteJump=infJumpEnabled,medusaCounter=medusaCounterEnabled,
		batCounter=batCounterEnabled,
		carryMode=speedMode,laggerMode=laggerToggled,laggerSpeed=LAGGER_SPEED,
		autoTPEnabled=autoTPEnabled,autoTPHeight=autoTPHeight,
		aimbot=aimbotEnabled,aimbotMove=aimbotMoveEnabled,autoSwing=autoSwingEnabled,
		unwalkEnabled=unwalkEnabled,
		antiLag=antiLagEnabled,removeAccessories=removeAccessoriesEnabled,
		lockInEnabled=lockInEnabled,
		themeMode=THEME_MODE,
	}
	if writefile then pcall(function() writefile("ColtHubPC.json",HS:JSONEncode(cfg)) end) end
end

task.spawn(function() while task.wait(5) do saveConfig() end end)

local setInstaGrab,setInfJumpVisual,setAntiRagVisual,setMedusaVisual
local setUnwalkVisual,setAntiLagVisual,setRemoveAccVisual,setFpsBoostVisual
local normalBox,carryBox,laggerBox,radInput,floatHeightBox,stealDurBox

-- âââ THEME APPLY FUNCTION (wires closed over in buildGui) âââ
local buildGui  -- forward declare so rebuildWithTheme can reference it

local function rebuildWithTheme(mode)
	THEME_MODE = mode
	ICON_ID = getIconId()
	saveConfig()
	-- Recolor own speed billboard
	if speedLabel then speedLabel.TextColor3 = getSpeedLabelColor() end
	-- Recolor other players' speed billboards
	for _, data in pairs(otherSpeedLabels) do
		if data.lbl then data.lbl.TextColor3 = getSpeedLabelColor() end
	end
	local old = game:GetService("CoreGui"):FindFirstChild("ColtHub")
	if old then old:Destroy() end
	local pg = LP:FindFirstChild("PlayerGui")
	if pg then
		local o = pg:FindFirstChild("ColtHub"); if o then o:Destroy() end
		local m = pg:FindFirstChild("ColtMobileGui"); if m then m:Destroy() end
	end
	buildGui()
end

buildGui = function()
	local P = getThemePalette(THEME_MODE)
	local BG    = P.BG
	local BG2   = P.BG2
	local CARD  = P.CARD
	local HOV   = P.HOV
	local RED   = P.ACCENT
	local REDDIM= P.ACCDIM
	local STROKE= P.STROKE
	local W     = P.W
	local DIM   = P.DIM
	local INP   = P.INP
	local OFF   = P.OFF
	local MB_C_OFF  = P.MB_C_OFF
	local MB_C_ON   = P.MB_C_ON
	local MB_BRD_OFF= P.MB_BRD_OFF
	local MB_BRD_ON = P.MB_BRD_ON
	local MB_TXT_OFF= P.MB_TXT_OFF
	local MB_TXT_ON = P.MB_TXT_ON

	local old=game:GetService("CoreGui"):FindFirstChild("ColtHub");if old then old:Destroy() end
	local pg=LP:FindFirstChild("PlayerGui");if pg then local o=pg:FindFirstChild("ColtHub");if o then o:Destroy() end end
	local gui=Instance.new("ScreenGui")
	gui.Name="ColtHub";gui.ResetOnSpawn=false;gui.DisplayOrder=10;gui.IgnoreGuiInset=true
	pcall(function() if syn and syn.protect_gui then syn.protect_gui(gui) end end)
	if not pcall(function() gui.Parent=game:GetService("CoreGui") end) then gui.Parent=LP:WaitForChild("PlayerGui") end

	local main=Instance.new("Frame",gui)
	main.Size=UDim2.new(0,300,0,510);main.Position=UDim2.new(0,20,0,20)
	main.BackgroundColor3=BG;main.BorderSizePixel=0;main.ClipsDescendants=false
	Instance.new("UICorner",main).CornerRadius=UDim.new(0,22)
	local mainStroke=Instance.new("UIStroke",main)
	mainStroke.Color=STROKE;mainStroke.Thickness=2

	local bgImg=Instance.new("ImageLabel",main)
	bgImg.Size=UDim2.new(1,0,1,0);bgImg.Position=UDim2.new(0,0,0,0)
	bgImg.BackgroundTransparency=1;bgImg.Image=getIconId()
	bgImg.ImageTransparency=0.88;bgImg.ScaleType=Enum.ScaleType.Crop
	bgImg.ZIndex=0
	Instance.new("UICorner",bgImg).CornerRadius=UDim.new(0,12)

	local function drag(f)
		local dn,ds,sp,di=false
		f.InputBegan:Connect(function(i)
			if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
				dn=true;ds=i.Position;sp=f.Position
				i.Changed:Connect(function() if i.UserInputState==Enum.UserInputState.End then dn=false end end)
			end
		end)
		f.InputChanged:Connect(function(i)
			if i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch then di=i end
		end)
		UIS.InputChanged:Connect(function(i)
			if i==di and dn then
				local nX=sp.X.Offset+(i.Position.X-ds.X)
				local nY=sp.Y.Offset+(i.Position.Y-ds.Y)
				f.Position=UDim2.new(sp.X.Scale,nX,sp.Y.Scale,nY)
			end
		end)
	end
	drag(main)

	local hdr=Instance.new("Frame",main)
	hdr.Size=UDim2.new(1,0,0,52);hdr.BackgroundColor3=BG2;hdr.BorderSizePixel=0
	Instance.new("UICorner",hdr).CornerRadius=UDim.new(0,12)

	local hdrIcon=Instance.new("ImageLabel",hdr)
	hdrIcon.Size=UDim2.new(0,26,0,26);hdrIcon.Position=UDim2.new(0,8,0.5,-13)
	hdrIcon.BackgroundColor3=RED;hdrIcon.BackgroundTransparency=0
	hdrIcon.Image=getIconId();hdrIcon.ImageTransparency=0.2;hdrIcon.ScaleType=Enum.ScaleType.Crop
	hdrIcon.BorderSizePixel=0;hdrIcon.ZIndex=3
	Instance.new("UICorner",hdrIcon).CornerRadius=UDim.new(0,5)

	local ttl=Instance.new("TextLabel",hdr)
	ttl.Size=UDim2.new(1,-80,1,0);ttl.Position=UDim2.new(0,42,0,0)
	ttl.BackgroundTransparency=1;ttl.Text="COLT HUB"
	ttl.TextColor3=Color3.fromRGB(235,235,235);ttl.Font=Enum.Font.GothamBlack;ttl.TextSize=15
	ttl.TextXAlignment=Enum.TextXAlignment.Left;ttl.ZIndex=3
	local sub=Instance.new("TextLabel",hdr)
	sub.Size=UDim2.new(1,-80,0,12);sub.Position=UDim2.new(0,42,0,28)
	sub.BackgroundTransparency=1;sub.Text="crimson edition"
	sub.TextColor3=REDDIM;sub.Font=Enum.Font.Gotham;sub.TextSize=9
	sub.TextXAlignment=Enum.TextXAlignment.Left;sub.ZIndex=3

	local closeBtn=Instance.new("TextButton",hdr)
	closeBtn.Size=UDim2.new(0,28,0,28);closeBtn.Position=UDim2.new(1,-34,0.5,-14)
	closeBtn.BackgroundColor3=BG2;closeBtn.BorderSizePixel=0
	closeBtn.Text="-";closeBtn.TextColor3=REDDIM;closeBtn.Font=Enum.Font.GothamBold;closeBtn.TextSize=22;closeBtn.ZIndex=4
	Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(0,6)

	local miniBtn=Instance.new("Frame",gui)
	miniBtn.Size=UDim2.new(0,44,0,44);miniBtn.Position=UDim2.new(0,20,0,20)
	miniBtn.BackgroundColor3=RED;miniBtn.BorderSizePixel=0
	miniBtn.ZIndex=20;miniBtn.Visible=false;miniBtn.Active=true
	Instance.new("UICorner",miniBtn).CornerRadius=UDim.new(1,0)
	local miniStroke=Instance.new("UIStroke",miniBtn);miniStroke.Color=STROKE;miniStroke.Thickness=1.5

	local miniIcon=Instance.new("ImageLabel",miniBtn)
	miniIcon.Size=UDim2.new(1,0,1,0);miniIcon.Position=UDim2.new(0,0,0,0)
	miniIcon.BackgroundTransparency=1;miniIcon.Image=getIconId()
	miniIcon.ImageTransparency=0.1;miniIcon.ScaleType=Enum.ScaleType.Crop;miniIcon.ZIndex=21
	Instance.new("UICorner",miniIcon).CornerRadius=UDim.new(1,0)

	local miniClk=Instance.new("TextButton",miniBtn)
	miniClk.Size=UDim2.new(1,0,1,0);miniClk.BackgroundTransparency=1;miniClk.Text="";miniClk.ZIndex=22

	local function showGui() main.Visible=true;miniBtn.Visible=false end
	local function hideGui() main.Visible=false;miniBtn.Visible=true end

	do
		local dn,ds,sp,di=false,nil,nil,nil;local moved=false
		miniClk.InputBegan:Connect(function(i)
			if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
				dn=true;moved=false;ds=i.Position;sp=miniBtn.Position
				i.Changed:Connect(function() if i.UserInputState==Enum.UserInputState.End then
					if not moved then showGui() end
					dn=false;moved=false
				end end)
			end
		end)
		miniClk.InputChanged:Connect(function(i)
			if i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch then di=i end
		end)
		UIS.InputChanged:Connect(function(i)
			if i==di and dn then
				local dx=i.Position.X-ds.X;local dy=i.Position.Y-ds.Y
				if math.abs(dx)>4 or math.abs(dy)>4 then moved=true end
				if moved then miniBtn.Position=UDim2.new(sp.X.Scale,sp.X.Offset+dx,sp.Y.Scale,sp.Y.Offset+dy) end
			end
		end)
	end
	closeBtn.MouseButton1Click:Connect(hideGui)

	local sf=Instance.new("ScrollingFrame",main)
	sf.Size=UDim2.new(1,0,1,-52);sf.Position=UDim2.new(0,0,0,52)
	sf.BackgroundTransparency=1;sf.BorderSizePixel=0;sf.ClipsDescendants=true
	sf.ScrollBarThickness=3;sf.ScrollBarImageColor3=RED
	sf.CanvasSize=UDim2.new(0,0,0,0);sf.AutomaticCanvasSize=Enum.AutomaticSize.Y
	local ll=Instance.new("UIListLayout",sf);ll.SortOrder=Enum.SortOrder.LayoutOrder;ll.Padding=UDim.new(0,2)
	local pad=Instance.new("UIPadding",sf)
	pad.PaddingLeft=UDim.new(0,7);pad.PaddingRight=UDim.new(0,7)
	pad.PaddingTop=UDim.new(0,7);pad.PaddingBottom=UDim.new(0,10)
	local lo=0
	local function LO() lo=lo+1;return lo end

	local stackDefs, stackWrappers, getDefaultStackPos

	local function mkSect(txt)
		local f=Instance.new("Frame",sf);f.Size=UDim2.new(1,0,0,24);f.BackgroundTransparency=1;f.BorderSizePixel=0;f.LayoutOrder=LO()
		local bar=Instance.new("Frame",f);bar.Size=UDim2.new(0,3,0,14);bar.Position=UDim2.new(0,0,0.5,-7)
		bar.BackgroundColor3=RED;bar.BorderSizePixel=0
		Instance.new("UICorner",bar).CornerRadius=UDim.new(0,2)
		local l=Instance.new("TextLabel",f);l.Size=UDim2.new(1,-14,1,0);l.Position=UDim2.new(0,10,0,0)
		l.BackgroundTransparency=1;l.Text=txt:upper();l.TextColor3=P.SECT_CLR
		l.Font=Enum.Font.GothamBlack;l.TextSize=9;l.TextXAlignment=Enum.TextXAlignment.Left
		local sep=Instance.new("Frame",f);sep.Size=UDim2.new(0.6,0,0,1);sep.Position=UDim2.new(0.4,0,0.5,0)
		sep.BackgroundColor3=CARD;sep.BorderSizePixel=0
	end

	local function mkRow(h)
		local f=Instance.new("Frame",sf);f.Size=UDim2.new(1,0,0,h or 34)
		f.BackgroundColor3=CARD;f.BorderSizePixel=0;f.LayoutOrder=LO()
		Instance.new("UICorner",f).CornerRadius=UDim.new(0,10)
		local rowStroke=Instance.new("UIStroke",f);rowStroke.Color=OFF;rowStroke.Thickness=1
		f.MouseEnter:Connect(function() TS:Create(f,TweenInfo.new(0.08),{BackgroundColor3=HOV}):Play() end)
		f.MouseLeave:Connect(function() TS:Create(f,TweenInfo.new(0.08),{BackgroundColor3=CARD}):Play() end)
		return f
	end

	local function mkLabel(row,txt)
		local l=Instance.new("TextLabel",row);l.Size=UDim2.new(0.58,0,1,0);l.Position=UDim2.new(0,11,0,0)
		l.BackgroundTransparency=1;l.Text=txt;l.TextColor3=W
		l.Font=Enum.Font.GothamBold;l.TextSize=11;l.TextXAlignment=Enum.TextXAlignment.Left
	end

	local function mkPill(row,offset)
		local pill=Instance.new("Frame",row);pill.Size=UDim2.new(0,38,0,20)
		pill.Position=UDim2.new(1,-(offset or 44),0.5,-10)
		pill.BackgroundColor3=OFF;pill.BorderSizePixel=0;pill.ZIndex=3
		Instance.new("UICorner",pill).CornerRadius=UDim.new(1,0)
		local ps=Instance.new("UIStroke",pill);ps.Color=STROKE;ps.Thickness=1
		local dot=Instance.new("Frame",pill);dot.Size=UDim2.new(0,14,0,14)
		dot.Position=UDim2.new(0,3,0.5,-7);dot.BackgroundColor3=DIM
		dot.BorderSizePixel=0;dot.ZIndex=4
		Instance.new("UICorner",dot).CornerRadius=UDim.new(1,0)
		return pill,dot,ps
	end

	local function animPill(pill,on,dot,ps)
		TS:Create(pill,TweenInfo.new(0.2,Enum.EasingStyle.Quad),{BackgroundColor3=on and RED or OFF}):Play()
		if dot then
			TS:Create(dot,TweenInfo.new(0.2,Enum.EasingStyle.Back),{
				Position=on and UDim2.new(1,-17,0.5,-7) or UDim2.new(0,3,0.5,-7),
				BackgroundColor3=on and W or DIM
			}):Play()
		end
		if ps then ps.Color=on and STROKE or OFF end
	end

	local function mkToggle(txt,cb)
		local row=mkRow(34);mkLabel(row,txt)
		local pill,dot,ps=mkPill(row,46)
		local on=false
		local function sv(s) on=s;animPill(pill,s,dot,ps) end
		local clk=Instance.new("TextButton",pill);clk.Size=UDim2.new(1,0,1,0);clk.BackgroundTransparency=1;clk.Text="";clk.ZIndex=5
		clk.MouseButton1Click:Connect(function() on=not on;sv(on);cb(on) end)
		pill.ZIndex=3
		return sv
	end

	local function mkBox(parent,default,w,xOff,cb)
		local tb=Instance.new("TextBox",parent)
		tb.Size=UDim2.new(0,w or 50,0,22);tb.Position=UDim2.new(1,-(xOff or 56),0.5,-11)
		tb.BackgroundColor3=INP;tb.BorderSizePixel=0;tb.Text=tostring(default);tb.TextColor3=W
		tb.Font=Enum.Font.GothamBold;tb.TextSize=11;tb.ClearTextOnFocus=false;tb.ZIndex=5
		Instance.new("UICorner",tb).CornerRadius=UDim.new(0,5)
		local bs=Instance.new("UIStroke",tb);bs.Color=DIM;bs.Thickness=1
		tb.Focused:Connect(function() TS:Create(bs,TweenInfo.new(0.12),{Color=RED}):Play() end)
		tb.FocusLost:Connect(function()
			TS:Create(bs,TweenInfo.new(0.12),{Color=DIM}):Play()
			if cb then local n=tonumber(tb.Text);if n then cb(n) else tb.Text=tostring(default) end end
		end)
		return tb
	end

	local function kbMatch(entry,kc) return kc==entry.kb or (entry.gp and kc==entry.gp) end

	local function mkKB(parent,kbEntry,cb)
		local btn=Instance.new("TextButton",parent)
		btn.Size=UDim2.new(0,46,0,22);btn.Position=UDim2.new(1,-50,0.5,-11)
		btn.BackgroundColor3=INP;btn.BorderSizePixel=0
		btn.Text=(kbEntry.kb or Enum.KeyCode.Unknown).Name;btn.TextColor3=W
		btn.Font=Enum.Font.GothamBold;btn.TextSize=9;btn.ZIndex=5
		Instance.new("UICorner",btn).CornerRadius=UDim.new(0,5)
		local li=false;local lc;local pv=btn.Text
		btn.MouseButton1Click:Connect(function()
			if li then li=false;_anyKeyListening=false;if lc then lc:Disconnect();lc=nil end;btn.Text=pv;btn.TextColor3=W;return end
			pv=btn.Text;li=true;_anyKeyListening=true;btn.Text="...";btn.TextColor3=W
			lc=UIS.InputBegan:Connect(function(inp)
				if not li then return end
				if inp.UserInputType~=Enum.UserInputType.Keyboard and inp.UserInputType~=Enum.UserInputType.Gamepad1 then return end
				if inp.KeyCode==Enum.KeyCode.Escape then li=false;_anyKeyListening=false;if lc then lc:Disconnect();lc=nil end;btn.Text=pv;btn.TextColor3=W;return end
				btn.Text=inp.KeyCode.Name;pv=inp.KeyCode.Name;btn.TextColor3=W
				li=false;_anyKeyListening=false;if lc then lc:Disconnect();lc=nil end
				if cb then cb(inp.KeyCode,inp.UserInputType==Enum.UserInputType.Gamepad1) end
			end)
		end)
		return btn
	end

	local function mkToggleKB(txt,kbEntry,onToggle,onKB)
		local row=mkRow(34);mkLabel(row,txt)
		if kbEntry then mkKB(row,kbEntry,function(k,isGp)
			if isGp then kbEntry.gp=k;kbEntry.kb=nil else kbEntry.kb=k;kbEntry.gp=nil end
			if onKB then onKB(k,isGp) end
		end) end
		local pill,dot,ps=mkPill(row,kbEntry and 106 or 46)
		local on=false
		local function sv(s) on=s;animPill(pill,s,dot,ps) end
		local clk=Instance.new("TextButton",pill);clk.Size=UDim2.new(1,0,1,0);clk.BackgroundTransparency=1;clk.Text="";clk.ZIndex=5
		clk.MouseButton1Click:Connect(function() if _anyKeyListening then return end;on=not on;sv(on);if onToggle then onToggle(on) end end)
		pill.ZIndex=3
		return sv
	end

	-- Steal progress bar
	local pbFrame=Instance.new("Frame",gui)
	pbFrame.Size=UDim2.new(0,280,0,50);pbFrame.Position=UDim2.new(0.5,-140,1,-66)
	pbFrame.BackgroundColor3=BG2;pbFrame.BorderSizePixel=0;pbFrame.Active=true;pbFrame.ClipsDescendants=false
	Instance.new("UICorner",pbFrame).CornerRadius=UDim.new(0,9)
	local pbStroke=Instance.new("UIStroke",pbFrame);pbStroke.Color=STROKE;pbStroke.Thickness=1
	local pbImg=Instance.new("ImageLabel",pbFrame)
	pbImg.Size=UDim2.new(1,0,1,0);pbImg.Position=UDim2.new(0,0,0,0)
	pbImg.BackgroundTransparency=1;pbImg.Image=getIconId()
	pbImg.ImageTransparency=0.82;pbImg.ScaleType=Enum.ScaleType.Crop;pbImg.ZIndex=0
	Instance.new("UICorner",pbImg).CornerRadius=UDim.new(0,9)
	do
		local pbDn,pbDs,pbSp,pbDi=false
		pbFrame.InputBegan:Connect(function(i)
			if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
				pbDn=true;pbDs=i.Position;pbSp=pbFrame.Position
				i.Changed:Connect(function() if i.UserInputState==Enum.UserInputState.End then pbDn=false end end)
			end
		end)
		pbFrame.InputChanged:Connect(function(i)
			if i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch then pbDi=i end
		end)
		UIS.InputChanged:Connect(function(i)
			if i==pbDi and pbDn then
				local vp=workspace.CurrentCamera.ViewportSize
				local pw=280;local ph=50
				local nx=math.clamp(pbSp.X.Offset+(i.Position.X-pbDs.X),0,vp.X-pw)
				local ny=math.clamp(pbSp.Y.Offset+(i.Position.Y-pbDs.Y),0,vp.Y-ph)
				pbFrame.Position=UDim2.new(0,nx,0,ny)
			end
		end)
	end
	progressPct=Instance.new("TextLabel",pbFrame)
	progressPct.Size=UDim2.new(0,44,0,16);progressPct.Position=UDim2.new(0,9,0,7)
	progressPct.BackgroundTransparency=1;progressPct.Text="0%";progressPct.TextColor3=W
	progressPct.Font=Enum.Font.GothamBold;progressPct.TextSize=11;progressPct.TextXAlignment=Enum.TextXAlignment.Left
	progressRadLbl=Instance.new("TextLabel",pbFrame)
	progressRadLbl.Size=UDim2.new(0,104,0,16);progressRadLbl.Position=UDim2.new(1,-112,0,7)
	progressRadLbl.BackgroundTransparency=1;progressRadLbl.Text=string.format("Radius: %.2g",Steal.StealRadius)
	progressRadLbl.TextColor3=W;progressRadLbl.Font=Enum.Font.GothamBold;progressRadLbl.TextSize=11;progressRadLbl.TextXAlignment=Enum.TextXAlignment.Right
	local pbg=Instance.new("Frame",pbFrame)
	pbg.Size=UDim2.new(1,-18,0,11);pbg.Position=UDim2.new(0,9,0,30)
	pbg.BackgroundColor3=P.PB_BG;pbg.BorderSizePixel=0
	Instance.new("UICorner",pbg).CornerRadius=UDim.new(1,0)
	progressFill=Instance.new("Frame",pbg)
	progressFill.Size=UDim2.new(0,0,1,0);progressFill.BackgroundColor3=RED;progressFill.BorderSizePixel=0
	Instance.new("UICorner",progressFill).CornerRadius=UDim.new(1,0)

	local function mkCircleBtn(parent, color, activeColor, xOff, tip)
		local btn=Instance.new("TextButton",parent)
		btn.Size=UDim2.new(0,18,0,18);btn.Position=UDim2.new(1,-xOff,0.5,-9)
		btn.BackgroundColor3=color;btn.BorderSizePixel=0;btn.Text="";btn.ZIndex=5
		Instance.new("UICorner",btn).CornerRadius=UDim.new(1,0)
		local bs=Instance.new("UIStroke",btn);bs.Color=activeColor;bs.Thickness=1.5
		local function setActive(on)
			TS:Create(btn,TweenInfo.new(0.15),{BackgroundColor3=on and activeColor or color}):Play()
		end
		return btn,setActive
	end

	mkSect("Speed")
	local activeSpeedMode = "normal"
	local setNormCircle, setCarryCircle, setDesyncCircle

	do
		local row=mkRow(32);mkLabel(row,"Normal Speed")
		normalBox=mkBox(row,NS,50,74,function(v) if v>0 and v<=500 then NS=v end;saveConfig() end)
		local normBtn,normSet=mkCircleBtn(row,OFF,RED,130)
		setNormCircle=normSet
		normBtn.MouseButton1Click:Connect(function()
			if desyncEnabled then return end
			activeSpeedMode="normal";speedMode=false;laggerToggled=false
			setNormCircle(true);setCarryCircle(false)
			if modeValLbl then modeValLbl.Text="Normal" end;saveConfig()
		end)
	end
	do
		local row=mkRow(32);mkLabel(row,"Carry Speed")
		carryBox=mkBox(row,CS,50,74,function(v) if v>0 and v<=500 then CS=v end;saveConfig() end)
		local carryBtn,carrySet=mkCircleBtn(row,OFF,RED,130)
		setCarryCircle=carrySet
		carryBtn.MouseButton1Click:Connect(function()
			if desyncEnabled then return end
			activeSpeedMode="carry";speedMode=true;laggerToggled=false
			setNormCircle(false);setCarryCircle(true)
			if modeValLbl then modeValLbl.Text="Carry" end;saveConfig()
		end)
	end
	do
		local row=mkRow(32);mkLabel(row,"Lagger Speed")
		laggerBox=mkBox(row,LAGGER_SPEED,50,48,function(v) if v>0 and v<=500 then LAGGER_SPEED=v end;saveConfig() end)
	end
	do
		local row=mkRow(32);mkLabel(row,"Mode")
		modeValLbl=Instance.new("TextLabel",row)
		modeValLbl.Size=UDim2.new(0,90,1,0);modeValLbl.Position=UDim2.new(1,-94,0,0)
		modeValLbl.BackgroundTransparency=1;modeValLbl.Text="Normal";modeValLbl.TextColor3=RED
		modeValLbl.Font=Enum.Font.GothamBlack;modeValLbl.TextSize=11;modeValLbl.TextXAlignment=Enum.TextXAlignment.Right
		local clk=Instance.new("TextButton",row);clk.Size=UDim2.new(1,0,1,0);clk.BackgroundTransparency=1;clk.Text="";clk.ZIndex=2
		clk.MouseButton1Click:Connect(function()
			if _anyKeyListening or desyncEnabled then return end
			speedMode=not speedMode
			if speedMode then laggerToggled=false;activeSpeedMode="carry";setNormCircle(false);setCarryCircle(true)
			else activeSpeedMode="normal";setNormCircle(true);setCarryCircle(false) end
			modeValLbl.Text=laggerToggled and "Lagger" or (speedMode and "Carry" or "Normal")
			saveConfig()
		end)
	end
	task.defer(function() if setNormCircle then setNormCircle(true) end end)

	mkSect("Keybinds")
	do local row=mkRow(32);mkLabel(row,"Speed Key");mkKB(row,KB.SpeedToggle,function(k) KB.SpeedToggle.kb=k;saveConfig() end) end
	do local row=mkRow(32);mkLabel(row,"Lagger Key");mkKB(row,KB.LaggerToggle,function(k) KB.LaggerToggle.kb=k;saveConfig() end) end

	mkSect("Desync")
	do
		local desRow=mkRow(32);mkLabel(desRow,"Desync")
		local desModeCircle,desModeSet=mkCircleBtn(desRow,OFF,RED,130)
		setDesyncCircle=desModeSet
		local desPill,desDot,desPs=mkPill(desRow,98)
		local desOn=false
		local function svDes(s)
			desOn=s;animPill(desPill,s,desDot,desPs)
			if s then
				setNormCircle(false);setCarryCircle(false);setDesyncCircle(not desyncSpeedMode)
				if modeValLbl then modeValLbl.Text=desyncSpeedMode and "D-Carry" or "D-Normal" end
			else
				local isCarry=speedMode and not laggerToggled
				setNormCircle(not isCarry);setCarryCircle(isCarry);setDesyncCircle(false)
				if modeValLbl then modeValLbl.Text=laggerToggled and "Lagger" or (speedMode and "Carry" or "Normal") end
			end
		end
		setDesyncVisual=svDes
		desModeCircle.MouseButton1Click:Connect(function()
			if not desyncEnabled then return end
			desyncSpeedMode=not desyncSpeedMode
			setDesyncCircle(not desyncSpeedMode)
			if modeValLbl then modeValLbl.Text=desyncSpeedMode and "D-Carry" or "D-Normal" end
			applyDesyncFFlags(desyncNormalSpeed,desyncCarrySpeed)
		end)
		local desClk=Instance.new("TextButton",desPill);desClk.Size=UDim2.new(1,0,1,0);desClk.BackgroundTransparency=1;desClk.Text="";desClk.ZIndex=5
		desClk.MouseButton1Click:Connect(function()
			desyncEnabled=not desyncEnabled
			svDes(desyncEnabled)
			if desyncEnabled then enableDesync() else disableDesync() end
			saveConfig()
		end)
		desPill.ZIndex=3
	end
	do
		local row=mkRow(32);mkLabel(row,"Desync Normal")
		mkBox(row,desyncNormalSpeed,50,48,function(v) if v>0 and v<=500 then desyncNormalSpeed=v;if desyncEnabled then applyDesyncFFlags(desyncNormalSpeed,desyncCarrySpeed) end end;saveConfig() end)
	end
	do
		local row=mkRow(32);mkLabel(row,"Desync Carry")
		mkBox(row,desyncCarrySpeed,50,48,function(v) if v>0 and v<=500 then desyncCarrySpeed=v;if desyncEnabled then applyDesyncFFlags(desyncNormalSpeed,desyncCarrySpeed) end end;saveConfig() end)
	end

	mkSect("Combat")
	do
		local autoSwingRow = nil
		local abRow=mkRow(32);mkLabel(abRow,"Aimbot")
		mkKB(abRow,KB.AimBot,function(k,isGp)
			if isGp then KB.AimBot.gp=k;KB.AimBot.kb=nil else KB.AimBot.kb=k;KB.AimBot.gp=nil end
			saveConfig()
		end)
		local abPill,abDot,abPs=mkPill(abRow,102)
		abPill.ZIndex=3
		local abOn=false
		local function svAimbot(s) abOn=s;animPill(abPill,s,abDot,abPs) end
		aimbotSetVisual=svAimbot
		autoSwingRow=mkRow(32)
		autoSwingRow.Visible=false
		mkLabel(autoSwingRow,"Auto Swing")
		local pill2,dot2,ps2=mkPill(autoSwingRow,42)
		local swingOn=false
		local function svSwing(s) swingOn=s;autoSwingEnabled=s;animPill(pill2,s,dot2,ps2);saveConfig() end
		autoSwingSetVisual=svSwing
		local clkSwing=Instance.new("TextButton",pill2);clkSwing.Size=UDim2.new(1,0,1,0);clkSwing.BackgroundTransparency=1;clkSwing.Text="";clkSwing.ZIndex=5
		clkSwing.MouseButton1Click:Connect(function() swingOn=not swingOn;svSwing(swingOn) end)
		local holdActive=false;local holdFired=false;local holdStart=0
		local abClk=Instance.new("TextButton",abRow);abClk.Size=UDim2.new(1,0,1,0);abClk.BackgroundTransparency=1;abClk.Text="";abClk.ZIndex=6
		abClk.InputBegan:Connect(function(inp)
			if inp.UserInputType~=Enum.UserInputType.MouseButton1 and inp.UserInputType~=Enum.UserInputType.Touch then return end
			holdActive=true;holdFired=false;holdStart=tick()
			task.delay(0.5,function()
				if holdActive and tick()-holdStart>=0.5 then
					holdFired=true
					autoSwingRow.Visible=not autoSwingRow.Visible
				end
			end)
		end)
		abClk.InputEnded:Connect(function(inp)
			if inp.UserInputType~=Enum.UserInputType.MouseButton1 and inp.UserInputType~=Enum.UserInputType.Touch then return end
			if holdActive and not holdFired then
				holdActive=false
				if _anyKeyListening then return end
				abOn=not abOn;svAimbot(abOn)
				aimbotEnabled=abOn
				if abOn then enableAimbot() else disableAimbot(autoSwingRow) end
				saveConfig()
			else holdActive=false end
		end)
		abPill.ZIndex=3;abClk.ZIndex=2
	end

	mkSect("Steal")
	do
		local row=mkRow(32);mkLabel(row,"Radius")
		radInput=mkBox(row,Steal.StealRadius,50,56,function(v)
			if v>=0.5 and v<=300 then Steal.StealRadius=v;if progressRadLbl then progressRadLbl.Text=string.format("Radius: %.2g",Steal.StealRadius) end end;saveConfig()
		end)
	end
	do
		local stealRow=mkRow(32);mkLabel(stealRow,"Insta Steal")
		local pill,sdot,sps=mkPill(stealRow,42)
		local on=false
		local function sv(s) on=s;animPill(pill,s,sdot,sps) end
		setInstaGrab=sv
		local durRow=mkRow(32);mkLabel(durRow,"Steal Duration")
		stealDurBox=mkBox(durRow,Steal.StealDuration,50,56,function(v) if v>0 and v<=60 then Steal.StealDuration=v end;saveConfig() end)
		local holdTimer=nil;local holding=false;local holdFired=false
		local clk=Instance.new("TextButton",pill);clk.Size=UDim2.new(1,0,1,0);clk.BackgroundTransparency=1;clk.Text="";clk.ZIndex=5
		clk.InputBegan:Connect(function(inp)
			if inp.UserInputType~=Enum.UserInputType.MouseButton1 and inp.UserInputType~=Enum.UserInputType.Touch then return end
			holding=true;holdFired=false
			holdTimer=task.delay(0.5,function()
				if not holding then return end
				holdFired=true;durRow.Visible=not durRow.Visible
			end)
		end)
		clk.InputEnded:Connect(function(inp)
			if inp.UserInputType~=Enum.UserInputType.MouseButton1 and inp.UserInputType~=Enum.UserInputType.Touch then return end
			if not holding then return end;holding=false
			if task.cancel then pcall(function() task.cancel(holdTimer) end) end
			if not holdFired then
				on=not on;sv(on);Steal.AutoStealEnabled=on
				if on then if not pcall(startAutoSteal) then Steal.AutoStealEnabled=false;sv(false) end else stopAutoSteal() end
				saveConfig()
			end
		end)
		pill.ZIndex=3
	end

	mkSect("Misc")
	setInfJumpVisual=mkToggle("Infinite Jump",function(on) infJumpEnabled=on;if on then startInfiniteJump() else stopInfiniteJump() end end)
	setAntiRagVisual=mkToggle("Anti Ragdoll",function(on) antiRagdollEnabled=on;if on then startAntiRagdoll() else stopAntiRagdoll() end end)
	setMedusaVisual=mkToggle("Medusa Counter",function(on)
		medusaCounterEnabled=on
		if on then setupMedusa(LP.Character)
		else for _,c in pairs(medusaConns) do pcall(function() c:Disconnect() end) end;medusaConns={} end
	end)
	setBatCounterVisual=mkToggle("Bat Counter",function(on)
		batCounterEnabled=on
		if on then startBatCounter() else stopBatCounter() end
	end)
	setUnwalkVisual=mkToggle("Unwalk",function(on) unwalkEnabled=on;if on then startUnwalk() else stopUnwalk() end end)

	-- Lock In Mode
	setLockInVisual=mkToggle("Lock In Animation",function(on)
		lockInEnabled=on
		if on then startLockIn() else stopLockIn() end
		saveConfig()
	end)

	mkSect("Visual")
	setAntiLagVisual=mkToggle("Anti Lag",function(on) if on then enableAntiLag() else disableAntiLag() end end)
	setFpsBoostVisual=mkToggle("FPS Boost",function(on)
		fpsBoostEnabled=on; if on then pcall(applyFPSBoost) end
	end)
	setRemoveAccVisual=mkToggle("Remove Accessories",function(on) if on then enableRemoveAccessories() else disableRemoveAccessories() end end)

	-- Color Theme
	mkSect("Color Theme")
	do
		local row=mkRow(34);mkLabel(row,"Theme")
		-- Three theme buttons: RED / BLACK / WHITE
		local function mkThemeBtn(parent, label, mode, xOff)
			local btn=Instance.new("TextButton",parent)
			btn.Size=UDim2.new(0,42,0,22);btn.Position=UDim2.new(1,-xOff,0.5,-11)
			btn.BackgroundColor3=INP;btn.BorderSizePixel=0
			btn.Text=label;btn.TextColor3=W
			btn.Font=Enum.Font.GothamBold;btn.TextSize=9;btn.ZIndex=5
			Instance.new("UICorner",btn).CornerRadius=UDim.new(0,5)
			local s=Instance.new("UIStroke",btn);s.Color=STROKE;s.Thickness=1
			if THEME_MODE==mode then
				btn.BackgroundColor3=RED
			end
			btn.MouseButton1Click:Connect(function()
				THEME_MODE=mode
				rebuildWithTheme(mode)
			end)
			return btn
		end
		mkThemeBtn(row,"RED","red",140)
		mkThemeBtn(row,"BLACK","black",94)
		mkThemeBtn(row,"WHITE","white",48)
	end

	mkSect("Movement")
	do
		local sv=mkToggleKB("Auto Left",KB.AutoLeft,
			function(on)
				autoLeftEnabled=on
				if on then
					if autoRightEnabled then autoRightEnabled=false;if autoRightSetVisual then autoRightSetVisual(false) end;stopAutoRight() end
					if aimbotEnabled then aimbotEnabled=false;aimbotMoveEnabled=false;stopBatAimbot();if aimbotSetVisual then aimbotSetVisual(false) end end
					startAutoLeft()
				else stopAutoLeft() end
			end,
			function(k) KB.AutoLeft.kb=k;saveConfig() end)
		autoLeftSetVisual=sv
	end
	do
		local sv=mkToggleKB("Auto Right",KB.AutoRight,
			function(on)
				autoRightEnabled=on
				if on then
					if autoLeftEnabled then autoLeftEnabled=false;if autoLeftSetVisual then autoLeftSetVisual(false) end;stopAutoLeft() end
					if aimbotEnabled then aimbotEnabled=false;aimbotMoveEnabled=false;stopBatAimbot();if aimbotSetVisual then aimbotSetVisual(false) end end
					startAutoRight()
				else stopAutoRight() end
			end,
			function(k) KB.AutoRight.kb=k;saveConfig() end)
		autoRightSetVisual=sv
	end
	do
		local row=mkRow(32);mkLabel(row,"TP Down")
		mkKB(row,KB.TPFloor,function(k) KB.TPFloor.kb=k;saveConfig() end)
		local clk=Instance.new("TextButton",row);clk.Size=UDim2.new(0.58,0,1,0);clk.BackgroundTransparency=1;clk.Text="";clk.ZIndex=2
		clk.MouseButton1Click:Connect(function() runTPFloor() end)
	end
	do
		local row=mkRow(32);mkLabel(row,"Drop Brainrot")
		mkKB(row,KB.DropBrainrot,function(k) KB.DropBrainrot.kb=k;saveConfig() end)
		local clk=Instance.new("TextButton",row);clk.Size=UDim2.new(0.58,0,1,0);clk.BackgroundTransparency=1;clk.Text="";clk.ZIndex=2
		clk.MouseButton1Click:Connect(function() runDrop() end)
	end

	mkSect("Interface")
	do
		local row=mkRow(32);mkLabel(row,"Hide UI")
		mkKB(row,KB.GuiHide,function(k) KB.GuiHide.kb=k;saveConfig() end)
	end

	UIS.InputBegan:Connect(function(input,gpe)
		if gpe then return end
		if _anyKeyListening then return end
		if input.UserInputType~=Enum.UserInputType.Keyboard and input.UserInputType~=Enum.UserInputType.Gamepad1 then return end
		local kc=input.KeyCode
		if kbMatch(KB.LaggerToggle,kc) then
			laggerToggled=not laggerToggled
			if laggerToggled then speedMode=false else speedMode=true end
			if modeValLbl then modeValLbl.Text=laggerToggled and "Lagger" or (speedMode and "Carry" or "Normal") end
			saveConfig()
		elseif kbMatch(KB.SpeedToggle,kc) then
			speedMode=not speedMode
			if speedMode then laggerToggled=false end
			if modeValLbl then modeValLbl.Text=laggerToggled and "Lagger" or (speedMode and "Carry" or "Normal") end
			saveConfig()
		elseif kbMatch(KB.DropBrainrot,kc) then runDrop()
		elseif kbMatch(KB.TPFloor,kc) then runTPFloor()
		elseif kbMatch(KB.AutoLeft,kc) then
			autoLeftEnabled=not autoLeftEnabled
			if autoLeftEnabled then
				if autoRightEnabled then autoRightEnabled=false;if autoRightSetVisual then autoRightSetVisual(false) end;stopAutoRight() end
				if aimbotEnabled then aimbotEnabled=false;aimbotMoveEnabled=false;stopBatAimbot();if aimbotSetVisual then aimbotSetVisual(false) end end
				startAutoLeft()
			else stopAutoLeft() end
			if autoLeftSetVisual then autoLeftSetVisual(autoLeftEnabled) end
		elseif kbMatch(KB.AutoRight,kc) then
			autoRightEnabled=not autoRightEnabled
			if autoRightEnabled then
				if autoLeftEnabled then autoLeftEnabled=false;if autoLeftSetVisual then autoLeftSetVisual(false) end;stopAutoLeft() end
				if aimbotEnabled then aimbotEnabled=false;aimbotMoveEnabled=false;stopBatAimbot();if aimbotSetVisual then aimbotSetVisual(false) end end
				startAutoRight()
			else stopAutoRight() end
			if autoRightSetVisual then autoRightSetVisual(autoRightEnabled) end
		elseif kbMatch(KB.AimBot,kc) then
			aimbotEnabled=not aimbotEnabled
			if aimbotEnabled then enableAimbot() else disableAimbot(nil) end
			if aimbotSetVisual then aimbotSetVisual(aimbotEnabled) end
		elseif kbMatch(KB.GuiHide,kc) then if main.Visible then hideGui() else showGui() end
		end
	end)

	-- ââââ MOBILE BUTTONS ââââ
	local MB_CIRCLE = false
	local MB_CIRCLE_SIZE = 56
	local MB_BW = 60
	local MB_BH = 50
	local MB_GAP = 6
	local MB_COLS = 2
	local MB_ROWS = 4

	local mbDefs = {
		{top="DROP", bot="BR",     fk="drop",      isT=false, fn=function() runDrop() end},
		{top="AUTO", bot="LEFT",   fk="autoLeft",  isT=true},
		{top="AIMBOT",bot="",      fk="aimbot",    isT=true},
		{top="AUTO", bot="RIGHT",  fk="autoRight", isT=true},
		{top="TP",   bot="DOWN",   fk="tpDown",    isT=false, fn=function() runTPFloor() end},
		{top="CARRY",bot="SPD",    fk="carrySpeed",isT=true},
		{top="LAGGER",bot="MODE",  fk="lagger",    isT=true},
		{top="BAT",  bot="CTR",    fk="batCtr",    isT=true},
	}

	local function getMBDefaultPos(i)
		local bw = MB_CIRCLE and MB_CIRCLE_SIZE or MB_BW
		local bh = MB_CIRCLE and MB_CIRCLE_SIZE or MB_BH
		local totW = MB_COLS*bw + (MB_COLS-1)*MB_GAP
		local totH = MB_ROWS*bh + (MB_ROWS-1)*MB_GAP
		local col = (i-1) % MB_COLS
		local row = math.floor((i-1)/MB_COLS)
		return UDim2.new(1, -(totW+14) + col*(bw+MB_GAP), 0.5, -(totH/2) + row*(bh+MB_GAP))
	end

	getDefaultStackPos = getMBDefaultPos

	local mbGui = Instance.new("ScreenGui", LP:WaitForChild("PlayerGui"))
	mbGui.Name = "ColtMobileGui"; mbGui.ResetOnSpawn = false
	mbGui.DisplayOrder = 18; mbGui.IgnoreGuiInset = true

	local mbStates = {}
	local mbFrames = {}
	stackWrappers = mbFrames

	-- mbLocked: when true, buttons cannot be dragged
	local mbLocked = false

	local function rebuildMobileButtons()
		for _,ch in ipairs(mbGui:GetChildren()) do pcall(function() ch:Destroy() end) end
		mbFrames = {}; stackWrappers = mbFrames

		for i, bd in ipairs(mbDefs) do
			local bw = MB_CIRCLE and MB_CIRCLE_SIZE or MB_BW
			local bh = MB_CIRCLE and MB_CIRCLE_SIZE or MB_BH
			local corner = MB_CIRCLE and UDim.new(1,0) or UDim.new(0,10)

			local wrap = Instance.new("Frame", mbGui)
			wrap.Name = "MB_"..bd.fk
			wrap.Size = UDim2.new(0, bw, 0, bh)
			wrap.Position = getMBDefaultPos(i)
			wrap.BackgroundColor3 = MB_C_OFF
			wrap.BorderSizePixel = 0
			wrap.ZIndex = 20; wrap.Active = true
			Instance.new("UICorner", wrap).CornerRadius = corner
			local bStr = Instance.new("UIStroke", wrap)
			bStr.Color = MB_BRD_OFF; bStr.Thickness = 1.2
			bStr.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
			mbFrames[bd.fk] = wrap

			local topL = Instance.new("TextLabel", wrap)
			topL.Size = UDim2.new(1,-4, 0, 22); topL.Position = UDim2.new(0,2, 0.5,-22)
			topL.BackgroundTransparency = 1; topL.Text = bd.top
			topL.TextColor3 = MB_TXT_OFF
			topL.Font = Enum.Font.GothamBlack; topL.TextSize = 12; topL.ZIndex = 21
			topL.TextXAlignment = Enum.TextXAlignment.Center

			local botL = Instance.new("TextLabel", wrap)
			botL.Size = UDim2.new(1,-4, 0, 16); botL.Position = UDim2.new(0,2, 0.5,1)
			botL.BackgroundTransparency = 1; botL.Text = bd.bot
			botL.TextColor3 = REDDIM
			botL.Font = Enum.Font.GothamBold; botL.TextSize = 10; botL.ZIndex = 21
			botL.TextXAlignment = Enum.TextXAlignment.Center

			if not mbStates[bd.fk] then mbStates[bd.fk] = false end
			local isActive = mbStates[bd.fk]

			local function setActive(v)
				mbStates[bd.fk] = v; isActive = v
				TS:Create(wrap,TweenInfo.new(0.15),{BackgroundColor3 = v and MB_C_ON or MB_C_OFF}):Play()
				bStr.Color = v and MB_BRD_ON or MB_BRD_OFF
				topL.TextColor3 = v and MB_TXT_ON or MB_TXT_OFF
				botL.TextColor3 = v and Color3.fromRGB(255,200,200) or REDDIM
			end

			if isActive then setActive(true) end

			local pressing, pressStart, moved2 = false, nil, false
			wrap.InputBegan:Connect(function(inp)
				local isT = inp.UserInputType==Enum.UserInputType.Touch
				local isM = inp.UserInputType==Enum.UserInputType.MouseButton1
				if not isT and not isM then return end
				if pressing then return end
				pressing=true; moved2=false; pressStart=inp.Position
			end)
			wrap.InputChanged:Connect(function(inp)
				if not pressing then return end
				if mbLocked then return end  -- locked = no drag
				local isT=inp.UserInputType==Enum.UserInputType.Touch
				local isM=inp.UserInputType==Enum.UserInputType.MouseMovement
				if not isT and not isM then return end
				if pressStart and (inp.Position-pressStart).Magnitude>8 then
					moved2=true
					wrap.Position=UDim2.new(wrap.Position.X.Scale, wrap.Position.X.Offset+(inp.Position.X-pressStart.X),
						wrap.Position.Y.Scale, wrap.Position.Y.Offset+(inp.Position.Y-pressStart.Y))
					pressStart=inp.Position
				end
			end)
			wrap.InputEnded:Connect(function(inp)
				local isT=inp.UserInputType==Enum.UserInputType.Touch
				local isM=inp.UserInputType==Enum.UserInputType.MouseButton1
				if not isT and not isM then return end
				if not pressing then return end; pressing=false
				if moved2 then moved2=false; return end
				-- TAP LOGIC
				if bd.fk=="drop" then runDrop(); return end
				if bd.fk=="tpDown" then runTPFloor(); return end
				if bd.fk=="carrySpeed" then
					if desyncEnabled then
						desyncSpeedMode=not desyncSpeedMode; setActive(desyncSpeedMode)
						if setDesyncCircle then setDesyncCircle(not desyncSpeedMode) end
						if modeValLbl then modeValLbl.Text=desyncSpeedMode and "D-Carry" or "D-Normal" end
						applyDesyncFFlags(desyncNormalSpeed,desyncCarrySpeed)
					else
						speedMode=not speedMode; setActive(speedMode)
						if setNormCircle then setNormCircle(not speedMode) end
						if setCarryCircle then setCarryCircle(speedMode) end
						if modeValLbl then modeValLbl.Text=laggerToggled and "Lagger" or (speedMode and "Carry" or "Normal") end
					end; return
				end
				if bd.fk=="lagger" then
					laggerToggled=not laggerToggled; setActive(laggerToggled)
					if modeValLbl then modeValLbl.Text=laggerToggled and "Lagger" or (speedMode and "Carry" or "Normal") end
					return
				end
				if bd.fk=="batCtr" then
					batCounterEnabled=not batCounterEnabled; setActive(batCounterEnabled)
					if batCounterEnabled then startBatCounter() else stopBatCounter() end
					if setBatCounterVisual then setBatCounterVisual(batCounterEnabled) end
					saveConfig(); return
				end
				local ns = not mbStates[bd.fk]
				if bd.fk~="autoLeft" and bd.fk~="autoRight" then setActive(ns) end
				mbStates[bd.fk] = ns
				if bd.fk=="autoLeft" then
					autoLeftEnabled=ns
					if ns then
						if autoRightEnabled then autoRightEnabled=false;if autoRightSetVisual then autoRightSetVisual(false) end;stopAutoRight() end
						if aimbotEnabled then aimbotEnabled=false;aimbotMoveEnabled=false;stopBatAimbot();if aimbotSetVisual then aimbotSetVisual(false) end end
						startAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(true) end
					else stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
				elseif bd.fk=="autoRight" then
					autoRightEnabled=ns
					if ns then
						if autoLeftEnabled then autoLeftEnabled=false;if autoLeftSetVisual then autoLeftSetVisual(false) end;stopAutoLeft() end
						if aimbotEnabled then aimbotEnabled=false;aimbotMoveEnabled=false;stopBatAimbot();if aimbotSetVisual then aimbotSetVisual(false) end end
						startAutoRight(); if autoRightSetVisual then autoRightSetVisual(true) end
					else stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
				elseif bd.fk=="aimbot" then
					aimbotEnabled=ns
					if ns then
						if autoLeftEnabled then autoLeftEnabled=false;stopAutoLeft();if autoLeftSetVisual then autoLeftSetVisual(false) end end
						if autoRightEnabled then autoRightEnabled=false;stopAutoRight();if autoRightSetVisual then autoRightSetVisual(false) end end
						enableAimbot(); if aimbotSetVisual then aimbotSetVisual(true) end
					else disableAimbot(nil); if aimbotSetVisual then aimbotSetVisual(false) end end
				end
			end)
		end
	end

	rebuildMobileButtons()

	mkSect("Mobile Buttons")
	do
		-- Lock Mobile Buttons
		local lockRow=mkRow(32); mkLabel(lockRow,"Lock Buttons")
		local lockPill,lockDot,lockPs=mkPill(lockRow,46)
		local lockOn=false
		local function svLock(s)
			lockOn=s; mbLocked=s; animPill(lockPill,s,lockDot,lockPs)
		end
		local lClk=Instance.new("TextButton",lockPill); lClk.Size=UDim2.new(1,0,1,0); lClk.BackgroundTransparency=1; lClk.Text=""; lClk.ZIndex=5
		lClk.MouseButton1Click:Connect(function() lockOn=not lockOn; svLock(lockOn) end)
		lockPill.ZIndex=3

		-- Circle Toggle
		local circRow = mkRow(32); mkLabel(circRow, "Circle Toggle")
		local circPill,circDot,circPs = mkPill(circRow, 46)
		local circOn = false
		local function svCirc(s)
			circOn = s; animPill(circPill, s, circDot, circPs)
			MB_CIRCLE = s; rebuildMobileButtons()
		end
		local cClk = Instance.new("TextButton", circPill); cClk.Size=UDim2.new(1,0,1,0); cClk.BackgroundTransparency=1; cClk.Text=""; cClk.ZIndex=5
		cClk.MouseButton1Click:Connect(function() circOn=not circOn; svCirc(circOn) end)
		circPill.ZIndex=3
		local csRow = mkRow(32); mkLabel(csRow, "Circle Size")
		mkBox(csRow, MB_CIRCLE_SIZE, 50, 56, function(v)
			if v >= 30 and v <= 120 then MB_CIRCLE_SIZE = math.floor(v); rebuildMobileButtons() end
		end)
		local drRow = mkRow(26)
		local drLbl = Instance.new("TextLabel", drRow)
		drLbl.Size=UDim2.new(1,-8,1,0); drLbl.Position=UDim2.new(0,8,0,0)
		drLbl.BackgroundTransparency=1; drLbl.Text="Buttons draggable (lock to disable)"
		drLbl.TextColor3=DIM; drLbl.Font=Enum.Font.Gotham
		drLbl.TextSize=9; drLbl.TextXAlignment=Enum.TextXAlignment.Left
		local rstRow = mkRow(32); mkLabel(rstRow, "Reset Positions")
		local rClk = Instance.new("TextButton", rstRow)
		rClk.Size=UDim2.new(1,0,1,0); rClk.BackgroundTransparency=1; rClk.Text=""; rClk.ZIndex=2
		rClk.MouseButton1Click:Connect(function()
			for i, bd in ipairs(mbDefs) do
				local w = mbFrames[bd.fk]
				if w then TS:Create(w,TweenInfo.new(0.35,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=getMBDefaultPos(i)}):Play() end
			end
		end)
		local rstLbl = Instance.new("TextLabel", rstRow)
		rstLbl.Size=UDim2.new(0,60,1,0); rstLbl.Position=UDim2.new(1,-64,0,0)
		rstLbl.BackgroundTransparency=1; rstLbl.Text="RESET"; rstLbl.TextColor3=RED
		rstLbl.Font=Enum.Font.GothamBlack; rstLbl.TextSize=10; rstLbl.TextXAlignment=Enum.TextXAlignment.Right
	end
end

local function loadConfig()
	if not(isfile and isfile("ColtHubPC.json")) then return end
	local ok,cfg=pcall(function() return HS:JSONDecode(readfile("ColtHubPC.json")) end)
	if not ok or not cfg then return end
	local function lk(e,d) if type(d)~="table" then return end;if d.kb and Enum.KeyCode[d.kb] then e.kb=Enum.KeyCode[d.kb] end;if d.gp and Enum.KeyCode[d.gp] then e.gp=Enum.KeyCode[d.gp] end end
	if cfg.themeMode then THEME_MODE=cfg.themeMode end
	if cfg.normalSpeed then NS=cfg.normalSpeed;if normalBox then normalBox.Text=tostring(NS) end end
	if cfg.carrySpeed then CS=cfg.carrySpeed;if carryBox then carryBox.Text=tostring(CS) end end
	if cfg.grabRadius then Steal.StealRadius=cfg.grabRadius;if radInput then radInput.Text=tostring(cfg.grabRadius) end;if progressRadLbl then progressRadLbl.Text=string.format("Radius: %.2g",cfg.grabRadius) end end
	if cfg.stealDuration then Steal.StealDuration=cfg.stealDuration;if stealDurBox then stealDurBox.Text=tostring(cfg.stealDuration) end end
	lk(KB.DropBrainrot,cfg.dropBrainrotKey);lk(KB.AutoLeft,cfg.autoLeftKey);lk(KB.AutoRight,cfg.autoRightKey)
	lk(KB.AimBot,cfg.aimbotKey)
	lk(KB.LaggerToggle,cfg.laggerToggleKey)
	lk(KB.TPFloor,cfg.tpFloorKey);lk(KB.GuiHide,cfg.guiHideKey)
	lk(KB.SpeedToggle,cfg.speedToggleKey)
	task.spawn(function()
		task.wait(0.15)
		if cfg.antiRagdoll then antiRagdollEnabled=true;if setAntiRagVisual then setAntiRagVisual(true) end;startAntiRagdoll() end
		if cfg.autoStealEnabled then Steal.AutoStealEnabled=true;if setInstaGrab then setInstaGrab(true) end;pcall(startAutoSteal) end
		if cfg.infiniteJump then infJumpEnabled=true;if setInfJumpVisual then setInfJumpVisual(true) end;startInfiniteJump() end
		if cfg.medusaCounter then medusaCounterEnabled=true;if setMedusaVisual then setMedusaVisual(true) end;setupMedusa(LP.Character) end
		if cfg.batCounter then batCounterEnabled=true;if setBatCounterVisual then setBatCounterVisual(true) end;startBatCounter() end
		if cfg.laggerSpeed and type(cfg.laggerSpeed)=="number" then LAGGER_SPEED=cfg.laggerSpeed;if laggerBox then laggerBox.Text=tostring(LAGGER_SPEED) end end
		if cfg.laggerMode then laggerToggled=true end
		if cfg.carryMode then speedMode=true;if modeValLbl then modeValLbl.Text=laggerToggled and "Lagger" or "Carry" end end
		if cfg.aimbot then
			aimbotEnabled=true;aimbotMoveEnabled=true
			if aimbotSetVisual then aimbotSetVisual(true) end
			enableAimbot()
		end
		if cfg.autoSwing then autoSwingEnabled=true;if autoSwingSetVisual then autoSwingSetVisual(true) end end
		if cfg.unwalkEnabled then unwalkEnabled=true;if setUnwalkVisual then setUnwalkVisual(true) end;task.spawn(function() task.wait(0.5);startUnwalk() end) end
		if cfg.antiLag then antiLagEnabled=false;enableAntiLag();if setAntiLagVisual then setAntiLagVisual(true) end end
		if cfg.autoTPEnabled then autoTPEnabled=true;if setAutoTPVisual then setAutoTPVisual(true) end;startAutoTP() end
		if cfg.autoTPHeight and type(cfg.autoTPHeight)=="number" then autoTPHeight=cfg.autoTPHeight end
		if cfg.removeAccessories then enableRemoveAccessories();if setRemoveAccVisual then setRemoveAccVisual(true) end end
		if cfg.lockInEnabled then lockInEnabled=true;if setLockInVisual then setLockInVisual(true) end;startLockIn() end
	end)
end

buildGui()
loadConfig()
print("RGS HUB V2" Loaded")

    normalSpeed = 60, carrySpeed = 30, laggerSpeed = 13, laggerCarrySpeed = 13,
    speedType = "normal", laggerActive = false, autoBatToggled = false,
    hittingCooldown = false, infJumpEnabled = false, antiRagdollEnabled = false,
    fpsBoostEnabled = false, guiVisible = true, isStealing = false,
    stealStartTime = nil, lastStealTick = 0, medusaLastUsed = 0,
    medusaDebounce = false, medusaCounterEnabled = false, dropBrainrotActive = false,
    autoLeftEnabled = false, autoRightEnabled = false, autoLeftPhase = 1,
    autoRightPhase = 1, _tpInProgress = false, lastMoveDir = Vector3.new(0,0,0),
    animEnabled = false, unwalkEnabled = false, originalC0 = nil, crazyAngle = 0,
}

local Keys = {
    autoBat = Enum.KeyCode.E, speed = Enum.KeyCode.Q, lagger = Enum.KeyCode.C,
    guiHide = Enum.KeyCode.LeftControl, autoLeft = Enum.KeyCode.L,
    autoRight = Enum.KeyCode.R, dropBrainrot = Enum.KeyCode.H, tpDown = Enum.KeyCode.T,
}

local main, closeBtn, miniBtn
local toggleGuiVis

local MobileButtons = { Visible = true, Locked = false, BtnsObjects = {}, BtnRefs = {}, Buttons = {}, HideGuiBtn = nil }

local MB_BTN_W = 58
local MB_BTN_H = 48
local MB_COLS = 2
local MB_ROWS = 4
local MB_PAD = 3
local MB_CORNER = 14
local MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
local MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
local mbPanel = nil

local Theme = {
    Background      = Color3.fromRGB(8, 10, 20),
    BackgroundSecondary = Color3.fromRGB(12, 15, 28),
    Header          = Color3.fromRGB(5, 8, 18),
    Element         = Color3.fromRGB(14, 18, 35),
    ElementHover    = Color3.fromRGB(20, 26, 50),
    Text            = Color3.fromRGB(200, 220, 255),
    SubText         = Color3.fromRGB(80, 110, 160),
    Accent          = Color3.fromRGB(0, 180, 255),
    AccentDark      = Color3.fromRGB(0, 100, 200),
    AccentGlow      = Color3.fromRGB(0, 220, 255),
    AccentDim       = Color3.fromRGB(0, 60, 120),
    Border          = Color3.fromRGB(0, 60, 120),
    BorderLight     = Color3.fromRGB(0, 100, 180),
    Success         = Color3.fromRGB(0, 255, 160),
    Danger          = Color3.fromRGB(255, 50, 80),
    Keybind         = Color3.fromRGB(10, 20, 45),
    KeybindText     = Color3.fromRGB(0, 180, 255),
    ToggleOff       = Color3.fromRGB(18, 22, 45),
    Font            = Enum.Font.Gotham,
    FontBold        = Enum.Font.GothamBold,
    FontBlack       = Enum.Font.GothamBlack,
    FontMedium      = Enum.Font.GothamMedium,
}

local AutoSteal = {
    Enabled = false, Radius = 20, Duration = 1.3, IsStealing = false,
    Data = {}, ProgressFill = nil, ProgressText = nil,
}

local function isMyPlotByName(plotName)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return false end
    local plot = plots:FindFirstChild(plotName); if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then local yb = sign:FindFirstChild("YourBase"); if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end end
    return false
end

local function findNearestPrompt()
    local char = LP.Character; local root = char and char:FindFirstChild("HumanoidRootPart"); if not root then return nil end
    local plots = workspace:FindFirstChild("Plots"); if not plots then return nil end
    local bestPrompt, bestDist, bestName = nil, math.huge, nil
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local podiums = plot:FindFirstChild("AnimalPodiums"); if not podiums then continue end
        for _, pod in ipairs(podiums:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base"); local spawn = base and base:FindFirstChild("Spawn")
                if spawn then
                    local dist = (spawn.Position - root.Position).Magnitude
                    if dist < bestDist and dist <= AutoSteal.Radius then
                        local att = spawn:FindFirstChild("PromptAttachment")
                        if att then for _, child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") then bestPrompt, bestDist, bestName = child, dist, pod.Name; break end
                        end end
                    end
                end
            end)
        end
    end
    return bestPrompt, bestDist, bestName
end

local function executeSteal(prompt, animalName)
    if AutoSteal.IsStealing then return end
    if not AutoSteal.Data[prompt] then
        AutoSteal.Data[prompt] = {hold = {}, trigger = {}, ready = true}
        pcall(function()
            if getconnections then
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(AutoSteal.Data[prompt].hold, c.Function) end end
                for _, c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(AutoSteal.Data[prompt].trigger, c.Function) end end
            end
        end)
    end
    local data = AutoSteal.Data[prompt]; if not data.ready then return end
    data.ready = false; AutoSteal.IsStealing = true; local startTime = tick()
    local conn; conn = RunService.Heartbeat:Connect(function()
        if not AutoSteal.IsStealing then conn:Disconnect(); return end
        local prog = math.clamp((tick() - startTime) / AutoSteal.Duration, 0, 1)
        if AutoSteal.ProgressFill then AutoSteal.ProgressFill.Size = UDim2.new(prog, 0, 1, 0) end
    end)
    task.spawn(function()
        for _, f in ipairs(data.hold) do task.spawn(f) end
        task.wait(AutoSteal.Duration)
        for _, f in ipairs(data.trigger) do task.spawn(f) end
        AutoSteal.IsStealing = false; data.ready = true
        task.wait(0.6)
        if not AutoSteal.IsStealing and AutoSteal.ProgressFill then
            TweenService:Create(AutoSteal.ProgressFill, TweenInfo.new(0.4), {Size = UDim2.new(0,0,1,0)}):Play()
        end
    end)
end

local autoStealConnection = nil
local function startAutoSteal()
    if autoStealConnection then return end
    autoStealConnection = RunService.Heartbeat:Connect(function()
        if AutoSteal.Enabled and not AutoSteal.IsStealing then
            local p, _, name = findNearestPrompt(); if p then executeSteal(p, name) end
        end
    end)
end
local function stopAutoSteal()
    if autoStealConnection then autoStealConnection:Disconnect(); autoStealConnection = nil end
    AutoSteal.IsStealing = false
    for k, v in pairs(AutoSteal.Data) do if v.ready ~= nil then v.ready = true end end
end

local MOVE_KEYS = {[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local DROP_ASCEND_DURATION = 0.2; local DROP_ASCEND_SPEED = 150
local POS = {L1=Vector3.new(-476.48,-6.28,92.73),L2=Vector3.new(-483.12,-4.95,94.80),R1=Vector3.new(-476.16,-6.52,25.62),R2=Vector3.new(-483.04,-5.09,23.14)}
local Conns = {autoSteal=nil,antiRag=nil,autoLeft=nil,autoRight=nil,anchor={},progress=nil}
local h, hrp, speedLbl
local setAutoLeft, setAutoRight
local setInstaGrab, setAutoBat, setInfJump, setAntiRag, setFps, setMedusaCounter
local setAnimToggle, setUnwalkToggle
local setupMedusaCounter, stopMedusaCounter, startAntiRagdoll, stopAntiRagdoll, applyFPSBoost
local modeValLbl, normalBox, carryBox, laggerBox, carryLaggerBox
local autoBatKeyBtn, speedKeyBtn, laggerKeyBtn, autoLeftKeyBtn, autoRightKeyBtn, guiHideKeyBtn, dropBrainrotKeyBtn, tpDownKeyBtn
local setSpeedToggleUI, setLaggerToggleUI
local progressRadLbl
local startAutoLeft, stopAutoLeft, startAutoRight, stopAutoRight

local function getRootJoint()
    local char2 = LP.Character; local torso = char2 and char2:FindFirstChild("LowerTorso")
    return torso and torso:FindFirstChild("Root")
end
local function resetBend()
    local rj = getRootJoint()
    if rj and State.originalC0 then rj.C0 = State.originalC0; State.originalC0 = nil end
end
local function getBat()
    local char = LP.Character; if not char then return nil end
    local tool = char:FindFirstChild("Bat"); if tool then return tool end
    local bp2 = LP:FindFirstChild("Backpack")
    if bp2 then tool = bp2:FindFirstChild("Bat"); if tool then tool.Parent = char; return tool end end
    return nil
end
local function tryHitBat()
    if State.hittingCooldown then return end; State.hittingCooldown = true
    pcall(function()
        local bat = getBat(); if bat then bat:Activate(); local ev = bat:FindFirstChildWhichIsA("RemoteEvent"); if ev then ev:FireServer() end end
    end)
    task.delay(0.08, function() State.hittingCooldown = false end)
end
local function getClosestPlayer()
    if not hrp then return nil, math.huge end
    local cp, cd = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if tr then local d = (hrp.Position - tr.Position).Magnitude; if d < cd then cd = d; cp = p end end
        end
    end
    return cp, cd
end

local saveDebounce = false
local function autoSaveConfig()
    if saveDebounce then return end; saveDebounce = true
    task.delay(0.5, function()
        local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
        pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end); saveDebounce = false
    end)
end
local function forceSaveConfig()
    local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
    pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end)
end
localsteal(steal) local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local Conns = {autoSteal=nil,antiRag=nil,anchor={},progress=nil,aimbot=nil,batCounter=nil}
local PLOT_CACHE_DURATION = 2
local PROMPT_CACHE_REFRESH = 0.15
local STEAL_COOLDOWN = 0.1
local MEDUSA_COOLDOWN = 25
local BAT_COUNTER_COOLDOWN = 0
local batCounterDebounce = false
local batCounterLastUsed = 0
local progressRadLbl,progressFill,progressPct
local modeValLbl
local _aimTarget = nil
local _aimLastScan = 0

local VYSE_AIMBOT_SPEED = 56.5
local VYSE_HIT_DIST = 5
local SWING_COOLDOWN = 0.08
local hittingCooldown = false

local desyncEnabled = false
local setDesyncVisual = nil
local desyncActive = false
local desyncSpeedConn = nil
local G_desyncAnimate = nil

local fpsBoostEnabled = false

local function refreshUIToggles()
    if setSpeedToggleUI then setSpeedToggleUI(State.speedType == "carry") end
    if setLaggerToggleUI then setLaggerToggleUI(State.laggerActive) end
    if modeValLbl then
        if State.laggerActive then modeValLbl.Text = (State.speedType=="normal") and "â¡ Lagger Normal" or "â¡ Lagger Carry"
        else modeValLbl.Text = (State.speedType=="normal") and "Normal" or "Carry" end
    end
end
local function toggleSpeedType()
    State.speedType = (State.speedType=="normal") and "carry" or "normal"; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.carrySpeed then MobileButtons.BtnRefs.carrySpeed:SetAttribute("MB_On", State.speedType=="carry") end
end
local function toggleLagger()
    State.laggerActive = not State.laggerActive; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.lagger then MobileButtons.BtnRefs.lagger:SetAttribute("MB_On", State.laggerActive) end
end
local function getCurrentSpeed()
    if State.laggerActive then return State.speedType=="normal" and State.laggerSpeed or State.laggerCarrySpeed
    else return State.speedType=="normal" and State.normalSpeed or State.carrySpeed end
end
local function getAutoMoveSpeed()
    return State.laggerActive and State.laggerSpeed or State.normalSpeed
end
local function tpToGround()
    local char = LP.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
    local rp = RaycastParams.new(); rp.FilterType = Enum.RaycastFilterType.Exclude; rp.FilterDescendantsInstances = {char}
    local rr = workspace:Raycast(root.Position, Vector3.new(0,-500,0), rp)
    if rr then root.CFrame = CFrame.new(rr.Position + Vector3.new(0,3,0)) else root.CFrame = root.CFrame * CFrame.new(0,-20,0) end
end
local function runDropBrainrot()
    if State.dropBrainrotActive then return end
    local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if not root then return end
    State.dropBrainrotActive=true; local t0=tick(); local dc
    dc=RunService.Heartbeat:Connect(function()
        local r=char and char:FindFirstChild("HumanoidRootPart"); if not r then dc:Disconnect(); State.dropBrainrotActive=false; return end
        if tick()-t0>=DROP_ASCEND_DURATION then
            dc:Disconnect()
            local rp=RaycastParams.new(); rp.FilterDescendantsInstances={char}; rp.FilterType=Enum.RaycastFilterType.Exclude
            local rr=workspace:Raycast(r.Position,Vector3.new(0,-2000,0),rp)
            if rr then local hum2=char:FindFirstChildOfClass("Humanoid"); local off=(hum2 and hum2.HipHeight or 2)+(r.Size.Y/2); r.CFrame=CFrame.new(r.Position.X,rr.Position.Y+off,r.Position.Z); r.AssemblyLinearVelocity=Vector3.new(0,0,0) end
            State.dropBrainrotActive=false; return
        end
        r.AssemblyLinearVelocity=Vector3.new(r.AssemblyLinearVelocity.X,DROP_ASCEND_SPEED,r.AssemblyLinearVelocity.Z)
    end)
end

-- ========== MOBILE PANEL ==========
local function createMobilePanel()
    local panel = Instance.new("ScreenGui")
    panel.Name="OmniHubButtons"; panel.Parent=LP:WaitForChild("PlayerGui"); panel.ResetOnSpawn=false; panel.ZIndexBehavior=Enum.ZIndexBehavior.Sibling

    mbPanel = Instance.new("Frame", panel)
    mbPanel.Name = "MobilePanel"
    mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H)
    mbPanel.Position = UDim2.new(1, -(MB_TOTAL_W+8), 0.5, -(MB_TOTAL_H/2))
    mbPanel.BackgroundColor3 = Theme.Background
    mbPanel.BorderSizePixel = 0; mbPanel.Active = true; mbPanel.ZIndex = 50
    Instance.new("UICorner", mbPanel).CornerRadius = UDim.new(0, MB_CORNER+4)
    local mbStroke = Instance.new("UIStroke", mbPanel)
    mbStroke.Color = Theme.Accent; mbStroke.Thickness = 1.5; mbStroke.Transparency = 0.45

    local dragStart, startPos, moved = nil, nil, false
    local DRAG_THRESHOLD = 8
    mbPanel.InputBegan:Connect(function(inp)
        if MobileButtons.Locked then return end
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
            dragStart=inp.Position; startPos=mbPanel.Position; moved=false
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if MobileButtons.Locked or not dragStart then return end
        if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then
            local d=inp.Position-dragStart
            if not moved and (math.abs(d.X)+math.abs(d.Y))>DRAG_THRESHOLD then moved=true end
            if moved then mbPanel.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+d.X,startPos.Y.Scale,startPos.Y.Offset+d.Y) end
        end
    end)
    UIS.InputEnded:Connect(function(inp)
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragStart=nil; moved=false end
    end)

    local function makeMobileBtn(label, row, col, onClick)
        local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
        local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
        local btn = Instance.new("TextButton", mbPanel)
        btn.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
        btn.Position = UDim2.new(0, x, 0, y)
        btn.BackgroundColor3 = Theme.Element; btn.BorderSizePixel = 0
        btn.Text = ""; btn.AutoButtonColor = false; btn.ZIndex = 52
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, MB_CORNER)
        local bs = Instance.new("UIStroke", btn); bs.Color = Theme.Accent; bs.Thickness = 1; bs.Transparency = 0.7
        local lbl = Instance.new("TextLabel", btn)
        lbl.Size = UDim2.new(1,0,1,0); lbl.BackgroundTransparency = 1
        lbl.Text = label; lbl.TextColor3 = Theme.Accent; lbl.Font = Theme.FontBold
        lbl.TextSize = 9; lbl.TextWrapped = true; lbl.TextXAlignment = Enum.TextXAlignment.Center; lbl.TextYAlignment = Enum.TextYAlignment.Center; lbl.ZIndex = 56

        btn.MouseButton1Down:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.05), {BackgroundColor3=Theme.ElementHover}):Play()
            TweenService:Create(bs, TweenInfo.new(0.05), {Transparency=0}):Play()
        end)
        btn.MouseButton1Up:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.10), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.10), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseEnter:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.ElementHover}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.4}):Play()
            end
        end)
        btn.MouseLeave:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseButton1Click:Connect(function() if onClick then pcall(onClick) end end)

        btn:SetAttribute("MB_On", false)
        btn.AttributeChanged:Connect(function(a)
            if a == "MB_On" then
                local on = btn:GetAttribute("MB_On")
                if on then
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Accent}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(5,5,15)}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0, Color = Theme.AccentGlow}):Play()
                    bs.Thickness = 2
                else
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Element}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Theme.Accent}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0.7, Color = Theme.Accent}):Play()
                    bs.Thickness = 1
                end
            end
        end)
        table.insert(MobileButtons.BtnsObjects, btn)
        return btn
    end

    local alMbBtn = makeMobileBtn("AUTO\nLEFT", 1, 1, function()
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoLeftEnabled = not State.autoLeftEnabled
        if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end
        if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoLeft = alMbBtn

    local arMbBtn = makeMobileBtn("AUTO\nRIGHT", 1, 2, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoRightEnabled = not State.autoRightEnabled
        if setAutoRight then setAutoRight(State.autoRightEnabled) end
        if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoRight = arMbBtn

    local batMbBtn = makeMobileBtn("BAT", 2, 1, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        State.autoBatToggled = not State.autoBatToggled
        if not State.autoBatToggled then resetBend() end
        if setAutoBat then setAutoBat(State.autoBatToggled) end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoBat = batMbBtn

    makeMobileBtn("DROP", 2, 2, function() runDropBrainrot() end)

    local carryMbBtn = makeMobileBtn("CARRY\nSPD", 3, 1, function() toggleSpeedType() end)
    MobileButtons.BtnRefs.carrySpeed = carryMbBtn

    local lagMbBtn = makeMobileBtn("LAG", 3, 2, function() toggleLagger() end)
    MobileButtons.BtnRefs.lagger = lagMbBtn

    makeMobileBtn("TP\nDOWN", 4, 1, function() tpToGround() end)

    local guiMbBtn = makeMobileBtn("GUI", 4, 2, function() if toggleGuiVis then toggleGuiVis() end end)
    MobileButtons.HideGuiBtn = guiMbBtn

    task.spawn(function()
        while task.wait(0.2) do
            pcall(function()
                if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
                if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
                if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
                if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
                if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
            end)
        end
    end)

    task.spawn(function()
        task.wait(0.1)
        if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
        if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
        if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
        if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
        if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
    end)

    for _, btn in pairs(MobileButtons.BtnsObjects) do btn.Visible = MobileButtons.Visible end
    mbPanel.Visible = MobileButtons.Visible
end

local function repositionMbButtons()
    if not mbPanel then return end
    local idx = 0
    for _, child in ipairs(mbPanel:GetChildren()) do
        if child:IsA("TextButton") then
            local row = math.floor(idx / MB_COLS) + 1
            local col = (idx % MB_COLS) + 1
            local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
            local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
            child.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
            child.Position = UDim2.new(0, x, 0, y)
            idx = idx + 1
        end
    end
end

local function applyMbBtnSize(w, h2)
    w = math.clamp(w, 40, 120); h2 = math.clamp(h2, 36, 120)
    MB_BTN_W = w; MB_BTN_H = h2
    MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
    MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
    if mbPanel then mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H); repositionMbButtons() end
end

-- ========== ANIMATIONS ==========
local Anims = {idle1="rbxassetid://133806214992291",idle2="rbxassetid://94970088341563",walk="rbxassetid://707897309",run="rbxassetid://707861613",jump="rbxassetid://116936326516985",fall="rbxassetid://116936326516985",climb="rbxassetid://116936326516985",swim="rbxassetid://116936326516985",swimidle="rbxassetid://116936326516985"}
task.spawn(function() pcall(function() ContentProvider:PreloadAsync({Anims.idle1,Anims.idle2,Anims.walk,Anims.run,Anims.jump,Anims.fall,Anims.climb,Anims.swim,Anims.swimidle}) end) end)
local animHeartbeatConn=nil; local savedAnimate=nil; local originalAnims=nil
local function isPackAnim(id) if not id then return false end; for _,v in pairs(Anims) do if v==id then return true end end; return false end
local function saveOriginalAnims(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function g(obj) return obj and obj.AnimationId or nil end; local ids={idle1=g(animate.idle and animate.idle.Animation1),idle2=g(animate.idle and animate.idle.Animation2),walk=g(animate.walk and animate.walk.WalkAnim),run=g(animate.run and animate.run.RunAnim),jump=g(animate.jump and animate.jump.JumpAnim),fall=g(animate.fall and animate.fall.FallAnim),climb=g(animate.climb and animate.climb.ClimbAnim),swim=g(animate.swim and animate.swim.Swim),swimidle=g(animate.swimidle and animate.swimidle.SwimIdle)}; if not isPackAnim(ids.walk) then originalAnims=ids end end
local function applyAnimPack(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,Anims.idle1); s(animate.idle and animate.idle.Animation2,Anims.idle2); s(animate.walk and animate.walk.WalkAnim,Anims.walk); s(animate.run and animate.run.RunAnim,Anims.run); s(animate.jump and animate.jump.JumpAnim,Anims.jump); s(animate.fall and animate.fall.FallAnim,Anims.fall); s(animate.climb and animate.climb.ClimbAnim,Anims.climb); s(animate.swim and animate.swim.Swim,Anims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,Anims.swimidle) end
local function restoreOriginalAnims(char) if not originalAnims then return end; local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj and id then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,originalAnims.idle1); s(animate.idle and animate.idle.Animation2,originalAnims.idle2); s(animate.walk and animate.walk.WalkAnim,originalAnims.walk); s(animate.run and animate.run.RunAnim,originalAnims.run); s(animate.jump and animate.jump.JumpAnim,originalAnims.jump); s(animate.fall and animate.fall.FallAnim,originalAnims.fall); s(animate.climb and animate.climb.ClimbAnim,originalAnims.climb); s(animate.swim and animate.swim.Swim,originalAnims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,originalAnims.swimidle); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end
local function startAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then saveOriginalAnims(char); applyAnimPack(char); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end; animHeartbeatConn=RunService.Heartbeat:Connect(function() if not State.animEnabled then return end; local c=LP.Character; if c then applyAnimPack(c) end end) end
local function stopAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then restoreOriginalAnims(char) end end
local function startUnwalk() if State.unwalkEnabled then return end; State.unwalkEnabled=true; local c=LP.Character; if not c then return end; local hum=c:FindFirstChildOfClass("Humanoid"); if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end; local anim=c:FindFirstChild("Animate"); if anim then savedAnimate=anim:Clone(); anim:Destroy() end end
local function stopUnwalk() if not State.unwalkEnabled then return end; State.unwalkEnabled=false; local c=LP.Character; if c and savedAnimate then savedAnimate.Parent=c; savedAnimate.Disabled=false; savedAnimate=nil end; task.spawn(function() task.wait(0.15); local char=LP.Character; if not char then return end; if State.animEnabled then saveOriginalAnims(char); applyAnimPack(char) else restoreOriginalAnims(char) end end) end

-- ========== MAIN GUI ==========
local gui = Instance.new("ScreenGui")
gui.Name="OmniHubGUI"; gui.ResetOnSpawn=false; gui.DisplayOrder=10; gui.IgnoreGuiInset=true; gui.Parent=LP:WaitForChild("PlayerGui")

local stealProgressBar = Instance.new("Frame", gui)
stealProgressBar.Size=UDim2.new(0,280,0,6); stealProgressBar.Position=UDim2.new(0.5,-140,0.9,0)
stealProgressBar.BackgroundColor3=Color3.fromRGB(8,12,25); stealProgressBar.BackgroundTransparency=0; stealProgressBar.BorderSizePixel=0; stealProgressBar.ZIndex=100
Instance.new("UICorner",stealProgressBar).CornerRadius=UDim.new(1,0)
local barStroke=Instance.new("UIStroke",stealProgressBar); barStroke.Color=Theme.AccentDim; barStroke.Thickness=1
local barFill=Instance.new("Frame",stealProgressBar); barFill.Size=UDim2.new(0,0,1,0); barFill.BackgroundColor3=Theme.Accent; barFill.BorderSizePixel=0
Instance.new("UICorner",barFill).CornerRadius=UDim.new(1,0); AutoSteal.ProgressFill=barFill

local function makeStealBarDraggable(frame)
    local dragging=false; local dragInput,dragStart,startPos,activeDragConn
    local function stopDrag() dragging=false; dragInput=nil; dragStart=nil; startPos=nil; if activeDragConn then activeDragConn:Disconnect(); activeDragConn=nil end end
    frame.InputBegan:Connect(function(input) if MobileButtons.Locked then return end; if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=input.Position; startPos=frame.Position; input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End or input.UserInputState==Enum.UserInputState.Cancelled then stopDrag() end end) end end)
    frame.InputChanged:Connect(function(input) if not dragging then return end; if MobileButtons.Locked then stopDrag(); return end; if input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch then dragInput=input end end)
    UIS.InputChanged:Connect(function(input) if input==dragInput and dragging then if MobileButtons.Locked then stopDrag(); return end; local delta=input.Position-dragStart; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y) end end)
end
makeStealBarDraggable(stealProgressBar)
local function saveStealBarPosition() local pos=stealProgressBar.Position; pcall(function() writefile("JispiStealBarPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function loadStealBarPosition() local savedPos=nil; pcall(function() savedPos=readfile("JispiStealBarPos.txt") end); if savedPos and savedPos~="" then local parts={}; for part in string.gmatch(savedPos,"[^,]+") do table.insert(parts,part) end; if #parts>=4 then stealProgressBar.Position=UDim2.new(tonumber(parts[1]),tonumber(parts[2]),tonumber(parts[3]),tonumber(parts[4])) end end end
loadStealBarPosition()
local lastStealBarSave=0; stealProgressBar:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastStealBarSave>0.5 then lastStealBarSave=tick(); saveStealBarPosition() end end)

-- ========== MAIN FRAME ==========
main = Instance.new("Frame", gui)
main.Name="Main"; main.Size=UDim2.new(0,320,0,470); main.Position=UDim2.new(0,20,0,20)
main.BackgroundColor3=Theme.Background; main.BorderSizePixel=0; main.Active=true; main.ClipsDescendants=true
Instance.new("UICorner",main).CornerRadius=UDim.new(0,16)
local outerGlow=Instance.new("UIStroke",main); outerGlow.Color=Theme.Accent; outerGlow.Thickness=1.5; outerGlow.Transparency=0.3
local glowPulse=Instance.new("Frame",main); glowPulse.Size=UDim2.new(1,8,1,8); glowPulse.Position=UDim2.new(0,-4,0,-4)
glowPulse.BackgroundColor3=Theme.Accent; glowPulse.BackgroundTransparency=0.88; glowPulse.BorderSizePixel=0; glowPulse.ZIndex=-1
Instance.new("UICorner",glowPulse).CornerRadius=UDim.new(0,20)
task.spawn(function()
    while true do
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.78}):Play()
        task.wait(1.8)
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.92}):Play()
        task.wait(1.8)
    end
end)

local function saveMainPosition() local pos=main.Position; pcall(function() writefile("JispiHubGUIPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function saveMiniPosition() if miniBtn then local pos=miniBtn.Position; pcall(function() writefile("JispiMiniPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end end
local function loadMainPosition() local sp=nil; pcall(function() sp=readfile("JispiHubGUIPos.txt") end); if sp and sp~="" then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then main.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])); if closeBtn then closeBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2])+320-34,tonumber(p[3]),tonumber(p[4])+18) end end end end
local function loadMiniPosition() local sp=nil; pcall(function() sp=readfile("JispiMiniPos.txt") end); if sp and sp~="" and miniBtn then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then miniBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])) end end end

-- ========== HEADER ==========
local header=Instance.new("Frame",main)
header.Size=UDim2.new(1,0,0,52); header.BackgroundColor3=Theme.Header; header.BorderSizePixel=0; header.ZIndex=5
Instance.new("UICorner",header).CornerRadius=UDim.new(0,16)
local headerAccentLine=Instance.new("Frame",header)
headerAccentLine.Size=UDim2.new(1,0,0,2); headerAccentLine.Position=UDim2.new(0,0,1,-2)
headerAccentLine.BackgroundColor3=Theme.Accent; headerAccentLine.BorderSizePixel=0; headerAccentLine.ZIndex=6
local lineGlow=Instance.new("UIGradient",headerAccentLine)
lineGlow.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,30)),ColorSequenceKeypoint.new(0.5,Theme.AccentGlow),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,30))})
local titleLbl=Instance.new("TextLabel",header)
titleLbl.Size=UDim2.new(1,0,1,-4); titleLbl.Position=UDim2.new(0,0,0,0)
titleLbl.BackgroundTransparency=1; titleLbl.Text="â  RBG HUB  â"
titleLbl.TextColor3=Theme.AccentGlow; titleLbl.Font=Theme.FontBlack; titleLbl.TextSize=17
titleLbl.TextXAlignment=Enum.TextXAlignment.Center; titleLbl.ZIndex=6
local versionLbl=Instance.new("TextLabel",header)
versionLbl.Size=UDim2.new(0,40,0,16); versionLbl.Position=UDim2.new(1,-50,0.5,-8)
versionLbl.BackgroundTransparency=1; versionLbl.Text="v1.0"; versionLbl.TextColor3=Theme.AccentDim
versionLbl.Font=Theme.Font; versionLbl.TextSize=9; versionLbl.ZIndex=6

-- ========== CLOSE / MINI ==========
closeBtn=Instance.new("TextButton",gui)
closeBtn.Size=UDim2.new(0,28,0,28); closeBtn.BackgroundColor3=Color3.fromRGB(14,18,35)
closeBtn.BorderSizePixel=0; closeBtn.Text="â"; closeBtn.TextColor3=Theme.SubText
closeBtn.Font=Theme.FontBold; closeBtn.TextSize=14; closeBtn.ZIndex=50
Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(0,8)
local closeStroke=Instance.new("UIStroke",closeBtn); closeStroke.Color=Theme.Border; closeStroke.Thickness=1.5
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Danger,TextColor3=Color3.new(1,1,1)}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Danger}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Color3.fromRGB(14,18,35),TextColor3=Theme.SubText}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Border}):Play() end)
closeBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)

miniBtn=Instance.new("ImageButton",gui); miniBtn.Name="OmniMiniButton"; miniBtn.Size=UDim2.new(0,44,0,44)
miniBtn.Position=UDim2.new(0,20,0,100); miniBtn.BackgroundColor3=Theme.Background; miniBtn.Image=""; miniBtn.BorderSizePixel=0; miniBtn.Visible=false; miniBtn.ZIndex=50
Instance.new("UICorner",miniBtn).CornerRadius=UDim.new(0,12)
local miniStroke=Instance.new("UIStroke",miniBtn); miniStroke.Color=Theme.Accent; miniStroke.Thickness=1.8
local miniIcon=Instance.new("TextLabel",miniBtn); miniIcon.Size=UDim2.new(1,0,1,0); miniIcon.Text="â"; miniIcon.TextColor3=Theme.AccentGlow; miniIcon.Font=Theme.FontBlack; miniIcon.TextSize=20; miniIcon.BackgroundTransparency=1
miniBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)
miniBtn.MouseEnter:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Element}):Play() end)
miniBtn.MouseLeave:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Background}):Play() end)

local function makeMainDraggable(frame)
    local dragging,dragInput,dragStart,startPos,startCloseBtnPos=false,nil,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; if closeBtn then startCloseBtnPos=closeBtn.Position end; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy); if closeBtn and startCloseBtnPos then closeBtn.Position=UDim2.new(startCloseBtnPos.X.Scale,startCloseBtnPos.X.Offset+dx,startCloseBtnPos.Y.Scale,startCloseBtnPos.Y.Offset+dy) end end end)
end
local function makeMiniDraggable(frame)
    local dragging,dragInput,dragStart,startPos=false,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy) end end)
end
makeMainDraggable(main); makeMiniDraggable(miniBtn)
local lastSaveTime=0; main:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastSaveTime>0.5 then lastSaveTime=tick(); saveMainPosition() end end)
local lastMiniSaveTime=0; miniBtn:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastMiniSaveTime>0.5 then lastMiniSaveTime=tick(); saveMiniPosition() end end)
local function updateCloseButtonPosition() if main.Visible then local p=main.Position; closeBtn.Position=UDim2.new(p.X.Scale,p.X.Offset+320-34,p.Y.Scale,p.Y.Offset+18) end end
main:GetPropertyChangedSignal("Position"):Connect(updateCloseButtonPosition); main:GetPropertyChangedSignal("Visible"):Connect(updateCloseButtonPosition)

-- ========== TAB SYSTEM ==========
local tabFrame=Instance.new("Frame",main)
tabFrame.Size=UDim2.new(1,-16,0,30); tabFrame.Position=UDim2.new(0,8,0,56)
tabFrame.BackgroundTransparency=1; tabFrame.BorderSizePixel=0; tabFrame.ZIndex=3
local tabLayout=Instance.new("UIListLayout",tabFrame)
tabLayout.FillDirection=Enum.FillDirection.Horizontal; tabLayout.SortOrder=Enum.SortOrder.LayoutOrder
tabLayout.Padding=UDim.new(0,4); tabLayout.HorizontalAlignment=Enum.HorizontalAlignment.Center

local scroll=Instance.new("ScrollingFrame",main)
scroll.Size=UDim2.new(1,-16,1,-94); scroll.Position=UDim2.new(0,8,0,90)
scroll.BackgroundTransparency=1; scroll.BorderSizePixel=0; scroll.ScrollBarThickness=3
scroll.ScrollBarImageColor3=Theme.Accent; scroll.ScrollBarImageTransparency=0.4
scroll.AutomaticCanvasSize=Enum.AutomaticSize.Y; scroll.CanvasSize=UDim2.new(0,0,0,0); scroll.ZIndex=2
local listLayout=Instance.new("UIListLayout",scroll); listLayout.SortOrder=Enum.SortOrder.LayoutOrder; listLayout.Padding=UDim.new(0,5)
local pad=Instance.new("UIPadding",scroll); pad.PaddingLeft=UDim.new(0,0); pad.PaddingRight=UDim.new(0,0); pad.PaddingTop=UDim.new(0,2); pad.PaddingBottom=UDim.new(0,8)

local tabDefs = {{"Speed",1},{"Features",2},{"Settings",3}}
local tabBtns = {}; local tabPages = {}; local activeTab = "Speed"

for _,td in ipairs(tabDefs) do
    local btn=Instance.new("TextButton",tabFrame)
    btn.Size=UDim2.new(0,80,0,26); btn.BackgroundColor3=(td[1]==activeTab) and Theme.AccentDark or Theme.Element
    btn.BorderSizePixel=0; btn.Text=td[1]:upper(); btn.TextColor3=(td[1]==activeTab) and Theme.AccentGlow or Theme.SubText
    btn.Font=Theme.FontBold; btn.TextSize=9; btn.LayoutOrder=td[2]; btn.ZIndex=4; btn.AutoButtonColor=false
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,8)
    local ts=Instance.new("UIStroke",btn); ts.Color=(td[1]==activeTab) and Theme.Accent or Theme.Border; ts.Thickness=(td[1]==activeTab) and 1.5 or 1
    tabBtns[td[1]]={btn=btn,stroke=ts}
    local page=Instance.new("Frame",scroll)
    page.Size=UDim2.new(1,0,0,0); page.BackgroundTransparency=1; page.BorderSizePixel=0
    page.Visible=(td[1]==activeTab); page.ZIndex=2
    page.AutomaticSize=Enum.AutomaticSize.Y
    local pageLL=Instance.new("UIListLayout",page); pageLL.SortOrder=Enum.SortOrder.LayoutOrder; pageLL.Padding=UDim.new(0,5)
    local pagePad=Instance.new("UIPadding",page); pagePad.PaddingBottom=UDim.new(0,4)
    tabPages[td[1]]=page
    local pageLO=0
    btn.MouseButton1Click:Connect(function()
        if activeTab==td[1] then return end
        activeTab=td[1]
        for name,data in pairs(tabBtns) do
            local isA=(name==activeTab)
            TweenService:Create(data.btn,TweenInfo.new(0.15),{BackgroundColor3=isA and Theme.AccentDark or Theme.Element}):Play()
            TweenService:Create(data.btn,TweenInfo.new(0.15),{TextColor3=isA and Theme.AccentGlow or Theme.SubText}):Play()
            TweenService:Create(data.stroke,TweenInfo.new(0.15),{Color=isA and Theme.Accent or Theme.Border,Thickness=isA and 1.5 or 1}):Play()
            tabPages[name].Visible=isA
        end
    end)
    btn.MouseEnter:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end
    end)
    btn.MouseLeave:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.Element}):Play() end
    end)
end

-- ========== UI HELPERS (per-page) ==========
local pageLOs = {Speed=0, Features=0, Settings=0}
local function LO(tab) pageLOs[tab]=(pageLOs[tab] or 0)+1; return pageLOs[tab] end
local function pg(tab) return tabPages[tab] end

local function makeGap(tab,px) local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,0,0,px or 3); f.BackgroundTransparency=1; f.LayoutOrder=LO(tab) end
local function makeDivider(tab)
    local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,-8,0,1); f.BackgroundColor3=Theme.Border; f.BackgroundTransparency=0; f.BorderSizePixel=0; f.LayoutOrder=LO(tab)
    local g=Instance.new("UIGradient",f); g.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,20)),ColorSequenceKeypoint.new(0.5,Theme.Accent),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,20))})
end
local function makeSectionLabel(tab,text)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,26); row.BackgroundTransparency=1; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-8,1,0); lbl.BackgroundTransparency=1
    lbl.Text="  â "..text:upper(); lbl.TextColor3=Theme.Accent; lbl.Font=Theme.FontBold; lbl.TextSize=10; lbl.TextXAlignment=Enum.TextXAlignment.Left
end

local function makeInputRow(tab,label,default,onChange)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0,100,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.ZIndex=2
    local box=Instance.new("TextBox",row); box.Size=UDim2.new(0,60,0,26); box.Position=UDim2.new(1,-70,0.5,-13); box.BackgroundColor3=Theme.BackgroundSecondary; box.BorderSizePixel=0; box.Text=tostring(default); box.TextColor3=Theme.AccentGlow; box.Font=Theme.FontBold; box.TextSize=13; box.ClearTextOnFocus=false; box.ZIndex=3
    Instance.new("UICorner",box).CornerRadius=UDim.new(0,8)
    local boxStroke=Instance.new("UIStroke",box); boxStroke.Color=Theme.AccentDim; boxStroke.Thickness=1
    box.Focused:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.Element}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
    box.FocusLost:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.BackgroundSecondary}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); local num=tonumber(box.Text); if num~=nil then local fv=math.floor(math.clamp(num,0,500)); box.Text=tostring(fv); if onChange then onChange(tostring(fv)) end; autoSaveConfig() else box.Text=tostring(default) end end)
    return box
end

local function makeStatusRow(tab,label,valTxt)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,32); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0.5,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local val=Instance.new("TextLabel",row); val.Size=UDim2.new(0.45,-10,1,0); val.Position=UDim2.new(0.52,0,0,0); val.BackgroundTransparency=1; val.Text=valTxt; val.TextColor3=Theme.AccentGlow; val.Font=Theme.FontBold; val.TextSize=12; val.TextXAlignment=Enum.TextXAlignment.Right
    return val
end

local function makeKeybindRow(tab,label,currentKey,onChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local btn=Instance.new("TextButton",row); btn.Size=UDim2.new(0,70,0,24); btn.Position=UDim2.new(1,-80,0.5,-12); btn.BackgroundColor3=Theme.Keybind; btn.BorderSizePixel=0; btn.Text="["..currentKey.Name.."]"; btn.TextColor3=Theme.Accent; btn.Font=Theme.FontBold; btn.TextSize=10
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,7)
    local bs=Instance.new("UIStroke",btn); bs.Color=Theme.AccentDim; bs.Thickness=1
    local listening=false; local listenConn
    local function stopListen(key)
        listening=false; if listenConn then listenConn:Disconnect(); listenConn=nil end
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(btn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); btn.TextColor3=Theme.Accent
        if key then btn.Text="["..key.Name.."]"; if onChanged then onChanged(key) end; autoSaveConfig() end
    end
    btn.MouseButton1Click:Connect(function()
        if listening then stopListen(nil); return end
        listening=true; btn.Text="[Â·Â·Â·]"; btn.TextColor3=Theme.AccentGlow; btn.BackgroundColor3=Theme.Element
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
        listenConn=UIS.InputBegan:Connect(function(inp,gp)
            if not listening then return end
            if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode)
            elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode) end
        end)
    end)
    return btn
end

local function makeToggleRow(tab,label,defaultKey,defaultOn,onToggle,onKeyChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,40); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-120,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local keyBtn=nil
    if defaultKey then
        keyBtn=Instance.new("TextButton",row); keyBtn.Size=UDim2.new(0,50,0,22); keyBtn.Position=UDim2.new(1,-108,0.5,-11); keyBtn.BackgroundColor3=Theme.Keybind; keyBtn.BorderSizePixel=0; keyBtn.Text=defaultKey.Name; keyBtn.TextColor3=Theme.Accent; keyBtn.Font=Theme.FontBold; keyBtn.TextSize=9; keyBtn.ZIndex=5
        Instance.new("UICorner",keyBtn).CornerRadius=UDim.new(0,5)
        local ks=Instance.new("UIStroke",keyBtn); ks.Color=Theme.AccentDim; ks.Thickness=1
        local kListening=false; local kConn
        local function kStop(key) kListening=false; if kConn then kConn:Disconnect(); kConn=nil end; TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(keyBtn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); keyBtn.TextColor3=Theme.Accent; if key then keyBtn.Text=key.Name; if onKeyChanged then onKeyChanged(key) end; autoSaveConfig() end end
        keyBtn.MouseButton1Click:Connect(function()
            if kListening then kStop(nil); return end
            kListening=true; keyBtn.Text="Â·Â·Â·"; keyBtn.TextColor3=Theme.AccentGlow; keyBtn.BackgroundColor3=Theme.Element
            TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
            kConn=UIS.InputBegan:Connect(function(inp,gp)
                if not kListening then return end
                if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode)
                elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode) end
            end)
        end)
    end
    local toggleBg=Instance.new("TextButton",row); toggleBg.Size=UDim2.new(0,42,0,22); toggleBg.Position=UDim2.new(1,-48,0.5,-11)
    toggleBg.BackgroundColor3=defaultOn and Theme.AccentDark or Theme.ToggleOff; toggleBg.BorderSizePixel=0; toggleBg.Text=""; toggleBg.ZIndex=5; toggleBg.AutoButtonColor=false
    Instance.new("UICorner",toggleBg).CornerRadius=UDim.new(1,0)
    local tStroke=Instance.new("UIStroke",toggleBg); tStroke.Color=defaultOn and Theme.Accent or Theme.Border; tStroke.Thickness=1.5
    local knob=Instance.new("Frame",toggleBg); knob.Size=UDim2.new(0,17,0,17); knob.Position=defaultOn and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5)
    knob.BackgroundColor3=defaultOn and Theme.AccentGlow or Theme.SubText; knob.BorderSizePixel=0; knob.ZIndex=6
    Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
    local isOn=defaultOn or false
    local function setV(on)
        isOn=on
        TweenService:Create(toggleBg,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{BackgroundColor3=on and Theme.AccentDark or Theme.ToggleOff}):Play()
        TweenService:Create(tStroke,TweenInfo.new(0.22),{Color=on and Theme.Accent or Theme.Border}):Play()
        TweenService:Create(knob,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{Position=on and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=on and Theme.AccentGlow or Theme.SubText}):Play()
        TweenService:Create(rowStroke,TweenInfo.new(0.22),{Color=on and Theme.AccentDim or Theme.Border,Thickness=on and 1.5 or 1}):Play()
    end
    toggleBg.MouseButton1Click:Connect(function() isOn=not isOn; setV(isOn); if onToggle then pcall(onToggle,isOn) end; autoSaveConfig() end)
    toggleBg.MouseEnter:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
    toggleBg.MouseLeave:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)
    return setV, keyBtn
end

-- ===================== BUILD TABS =====================

-- ===== SPEED TAB =====
makeSectionLabel("Speed","Speed")
normalBox=makeInputRow("Speed","Normal",State.normalSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.normalSpeed=n end end)
carryBox=makeInputRow("Speed","Carry",State.carrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.carrySpeed=n end end)
setSpeedToggleUI,speedKeyBtn=makeToggleRow("Speed","Toggle",Keys.speed,false,function(on) toggleSpeedType() end,function(k) Keys.speed=k end)
modeValLbl=makeStatusRow("Speed","Mode","Normal")
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Lagger Speed")
laggerBox=makeInputRow("Speed","Normal",State.laggerSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerSpeed=n end end)
carryLaggerBox=makeInputRow("Speed","Carry",State.laggerCarrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerCarrySpeed=n end end)
setLaggerToggleUI,laggerKeyBtn=makeToggleRow("Speed","Toggle",Keys.lagger,false,function(on) toggleLagger() end,function(k) Keys.lagger=k end)
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Combat")
setAutoBat,autoBatKeyBtn=makeToggleRow("Speed","Bat Aimbot",Keys.autoBat,false,function(on) State.autoBatToggled=on; if not on then resetBend() end end,function(k) Keys.autoBat=k end)
makeGap("Speed",4)

-- ===== FEATURES TAB =====
makeSectionLabel("Features","Auto Steal")
local radiusBox=makeInputRow("Features","Grab Rad",AutoSteal.Radius,function(v) local n=tonumber(v); if n and n>=5 and n<=300 then AutoSteal.Radius=math.floor(n); if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end end)
setInstaGrab=makeToggleRow("Features","Auto Grab",nil,false,function(on) AutoSteal.Enabled=on; if on then startAutoSteal() else stopAutoSteal() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Active Toggles")
setInfJump=makeToggleRow("Features","Infinite Jump",nil,false,function(on) State.infJumpEnabled=on end)
setAntiRag=makeToggleRow("Features","Anti Ragdoll",nil,false,function(on) State.antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
setFps=makeToggleRow("Features","FPS Boost",nil,false,function(on) State.fpsBoostEnabled=on; if on then pcall(applyFPSBoost) end end)
setMedusaCounter=makeToggleRow("Features","Medusa Counter",nil,false,function(on) State.medusaCounterEnabled=on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
setAnimToggle=makeToggleRow("Features","Tryhard Anim",nil,false,function(on) State.animEnabled=on; if on then startAnimToggle() else stopAnimToggle() end end)
setUnwalkToggle=makeToggleRow("Features","Unwalk",nil,false,function(on) if on then startUnwalk() else stopUnwalk() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Movement")
tpDownKeyBtn=makeKeybindRow("Features","TP Down",Keys.tpDown,function(k) Keys.tpDown=k end)
dropBrainrotKeyBtn=makeKeybindRow("Features","Drop Brainrot",Keys.dropBrainrot,function(k) Keys.dropBrainrot=k end)
setAutoLeft,autoLeftKeyBtn=makeToggleRow("Features","Auto Left",Keys.autoLeft,false,function(on) State.autoLeftEnabled=on; if on then startAutoLeft() else stopAutoLeft() end end,function(k) Keys.autoLeft=k end)
setAutoRight,autoRightKeyBtn=makeToggleRow("Features","Auto Right",Keys.autoRight,false,function(on) State.autoRightEnabled=on; if on then startAutoRight() else stopAutoRight() end end,function(k) Keys.autoRight=k end)
makeGap("Features",4)

-- ===== SETTINGS TAB =====
makeSectionLabel("Settings","Interface")
guiHideKeyBtn=makeKeybindRow("Settings","Hide GUI",Keys.guiHide,function(k) Keys.guiHide=k end)
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

makeSectionLabel("Settings","Mobile Panel")
local showRow=Instance.new("Frame",pg("Settings")); showRow.Size=UDim2.new(1,0,0,38); showRow.BackgroundColor3=Theme.Element; showRow.BorderSizePixel=0; showRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",showRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",showRow).Color=Theme.Border
local showLbl=Instance.new("TextLabel",showRow); showLbl.Size=UDim2.new(1,-115,1,0); showLbl.Position=UDim2.new(0,12,0,0); showLbl.BackgroundTransparency=1; showLbl.Text="Show Buttons"; showLbl.TextColor3=Theme.Text; showLbl.Font=Theme.FontMedium; showLbl.TextSize=12; showLbl.TextXAlignment=Enum.TextXAlignment.Left
local showToggleBg=Instance.new("TextButton",showRow); showToggleBg.Size=UDim2.new(0,42,0,22); showToggleBg.Position=UDim2.new(1,-48,0.5,-11); showToggleBg.BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff; showToggleBg.BorderSizePixel=0; showToggleBg.Text=""; showToggleBg.ZIndex=5; showToggleBg.AutoButtonColor=false
Instance.new("UICorner",showToggleBg).CornerRadius=UDim.new(1,0)
local showKnob=Instance.new("Frame",showToggleBg); showKnob.Size=UDim2.new(0,17,0,17); showKnob.Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); showKnob.BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText; showKnob.BorderSizePixel=0; showKnob.ZIndex=6; Instance.new("UICorner",showKnob).CornerRadius=UDim.new(1,0)
showToggleBg.MouseButton1Click:Connect(function() MobileButtons.Visible=not MobileButtons.Visible; if mbPanel then mbPanel.Visible=MobileButtons.Visible end; TweenService:Create(showToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(showKnob,TweenInfo.new(0.22),{Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
showToggleBg.MouseEnter:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
showToggleBg.MouseLeave:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local lockRow=Instance.new("Frame",pg("Settings")); lockRow.Size=UDim2.new(1,0,0,38); lockRow.BackgroundColor3=Theme.Element; lockRow.BorderSizePixel=0; lockRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",lockRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",lockRow).Color=Theme.Border
local lockLbl=Instance.new("TextLabel",lockRow); lockLbl.Size=UDim2.new(1,-115,1,0); lockLbl.Position=UDim2.new(0,12,0,0); lockLbl.BackgroundTransparency=1; lockLbl.Text="Lock Buttons"; lockLbl.TextColor3=Theme.Text; lockLbl.Font=Theme.FontMedium; lockLbl.TextSize=12; lockLbl.TextXAlignment=Enum.TextXAlignment.Left
local lockToggleBg=Instance.new("TextButton",lockRow); lockToggleBg.Size=UDim2.new(0,42,0,22); lockToggleBg.Position=UDim2.new(1,-48,0.5,-11); lockToggleBg.BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff; lockToggleBg.BorderSizePixel=0; lockToggleBg.Text=""; lockToggleBg.ZIndex=5; lockToggleBg.AutoButtonColor=false
Instance.new("UICorner",lockToggleBg).CornerRadius=UDim.new(1,0)
local lockKnob=Instance.new("Frame",lockToggleBg); lockKnob.Size=UDim2.new(0,17,0,17); lockKnob.Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); lockKnob.BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText; lockKnob.BorderSizePixel=0; lockKnob.ZIndex=6; Instance.new("UICorner",lockKnob).CornerRadius=UDim.new(1,0)
lockToggleBg.MouseButton1Click:Connect(function() MobileButtons.Locked=not MobileButtons.Locked; TweenService:Create(lockToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(lockKnob,TweenInfo.new(0.22),{Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
lockToggleBg.MouseEnter:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
lockToggleBg.MouseLeave:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local sizeRow=Instance.new("Frame",pg("Settings")); sizeRow.Size=UDim2.new(1,0,0,38); sizeRow.BackgroundColor3=Theme.Element; sizeRow.BorderSizePixel=0; sizeRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",sizeRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",sizeRow).Color=Theme.Border
local sizeLbl=Instance.new("TextLabel",sizeRow); sizeLbl.Size=UDim2.new(0,100,1,0); sizeLbl.Position=UDim2.new(0,12,0,0); sizeLbl.BackgroundTransparency=1; sizeLbl.Text="Btn Size"; sizeLbl.TextColor3=Theme.Text; sizeLbl.Font=Theme.FontMedium; sizeLbl.TextSize=12; sizeLbl.TextXAlignment=Enum.TextXAlignment.Left
local sizeValLbl=Instance.new("TextLabel",sizeRow); sizeValLbl.Size=UDim2.new(0,40,0,16); sizeValLbl.Position=UDim2.new(0,112,0.5,-8); sizeValLbl.BackgroundTransparency=1; sizeValLbl.Text=MB_BTN_W.."px"; sizeValLbl.TextColor3=Theme.AccentGlow; sizeValLbl.Font=Theme.FontBold; sizeValLbl.TextSize=9; sizeValLbl.TextXAlignment=Enum.TextXAlignment.Center
local decBtn=Instance.new("TextButton",sizeRow); decBtn.Size=UDim2.new(0,24,0,24); decBtn.Position=UDim2.new(1,-78,0.5,-12); decBtn.BackgroundColor3=Theme.BackgroundSecondary; decBtn.BorderSizePixel=0; decBtn.Text="-"; decBtn.TextColor3=Theme.Text; decBtn.Font=Theme.FontBlack; decBtn.TextSize=14; decBtn.ZIndex=5
Instance.new("UICorner",decBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",decBtn).Color=Theme.Border
local incBtn=Instance.new("TextButton",sizeRow); incBtn.Size=UDim2.new(0,24,0,24); incBtn.Position=UDim2.new(1,-48,0.5,-12); incBtn.BackgroundColor3=Theme.BackgroundSecondary; incBtn.BorderSizePixel=0; incBtn.Text="+"; incBtn.TextColor3=Theme.Text; incBtn.Font=Theme.FontBlack; incBtn.TextSize=14; incBtn.ZIndex=5
Instance.new("UICorner",incBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",incBtn).Color=Theme.Border
decBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W-4,MB_BTN_H-3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
incBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W+4,MB_BTN_H+3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
for _,b in pairs({decBtn,incBtn}) do
    b.MouseEnter:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
    b.MouseLeave:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
end
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

local saveConfigRow=Instance.new("Frame",pg("Settings")); saveConfigRow.Size=UDim2.new(1,0,0,40); saveConfigRow.BackgroundColor3=Theme.Element; saveConfigRow.BorderSizePixel=0; saveConfigRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",saveConfigRow).CornerRadius=UDim.new(0,10)
local saveConfigBtn=Instance.new("TextButton",saveConfigRow); saveConfigBtn.Size=UDim2.new(1,-16,0,28); saveConfigBtn.Position=UDim2.new(0,8,0.5,-14)
saveConfigBtn.BackgroundColor3=Color3.fromRGB(0,40,80); saveConfigBtn.BorderSizePixel=0; saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; saveConfigBtn.Font=Theme.FontBold; saveConfigBtn.TextSize=12; saveConfigBtn.ZIndex=5; saveConfigBtn.AutoButtonColor=false
Instance.new("UICorner",saveConfigBtn).CornerRadius=UDim.new(0,9)
local saveBtnStroke=Instance.new("UIStroke",saveConfigBtn); saveBtnStroke.Color=Theme.Accent; saveBtnStroke.Thickness=1.5
saveConfigBtn.MouseEnter:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,60,120)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.AccentGlow}):Play() end)
saveConfigBtn.MouseLeave:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,40,80)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
saveConfigBtn.MouseButton1Click:Connect(function() forceSaveConfig(); saveConfigBtn.Text="â  SAVED!"; saveConfigBtn.TextColor3=Theme.Success; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Success}):Play(); task.wait(1.2); saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)

local resetMbRow=Instance.new("Frame",pg("Settings")); resetMbRow.Size=UDim2.new(1,0,0,36); resetMbRow.BackgroundColor3=Theme.Element; resetMbRow.BorderSizePixel=0; resetMbRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",resetMbRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",resetMbRow).Color=Theme.Border
local resetMbBtn=Instance.new("TextButton",resetMbRow); resetMbBtn.Size=UDim2.new(1,-16,0,24); resetMbBtn.Position=UDim2.new(0,8,0.5,-12)
resetMbBtn.BackgroundColor3=Theme.BackgroundSecondary; resetMbBtn.BorderSizePixel=0; resetMbBtn.Text="Reset Button Panel Position"; resetMbBtn.TextColor3=Theme.SubText; resetMbBtn.Font=Theme.FontMedium; resetMbBtn.TextSize=10; resetMbBtn.ZIndex=5; resetMbBtn.AutoButtonColor=false
Instance.new("UICorner",resetMbBtn).CornerRadius=UDim.new(0,7)
resetMbBtn.MouseEnter:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
resetMbBtn.MouseLeave:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
resetMbBtn.MouseButton1Click:Connect(function() if mbPanel then mbPanel.Position=UDim2.new(1,-(MB_TOTAL_W+8),0.5,-(MB_TOTAL_H/2)) end end)

makeGap("Settings",2)
local footerLbl=Instance.new("TextLabel",pg("Settings")); footerLbl.Size=UDim2.new(1,0,0,18); footerLbl.BackgroundTransparency=1; footerLbl.LayoutOrder=LO("Settings"); footerLbl.Text="â RBG â"; footerLbl.TextColor3=Theme.AccentDim; footerLbl.Font=Theme.FontBold; footerLbl.TextSize=9; footerLbl.TextXAlignment=Enum.TextXAlignment.Center

-- ========== TOGGLE GUI VISIBILITY ==========
toggleGuiVis=function()
    State.guiVisible=not State.guiVisible
    if main then main.Visible=State.guiVisible; closeBtn.Visible=State.guiVisible; if miniBtn then miniBtn.Visible=not State.guiVisible end end
end

-- ========== LOGIC ==========
local function findMedusa()
    local char=LP.Character; if not char then return nil end
    for _,tool in ipairs(char:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end
    local bp2=LP:FindFirstChild("Backpack"); if bp2 then for _,tool in ipairs(bp2:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end end
    return nil
end
local function useMedusaCounter()
    if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<25 then return end; local char=LP.Character; if not char then return end
    State.medusaDebounce=true; local med=findMedusa(); if not med then State.medusaDebounce=false; return end
    if med.Parent~=char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
end
local function onAnchorChanged(part) return part:GetPropertyChangedSignal("Anchored"):Connect(function() if part.Anchored and part.Transparency==1 then useMedusaCounter() end end) end
setupMedusaCounter=function(char) stopMedusaCounter(); if not char then return end; for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end; table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end)) end
stopMedusaCounter=function() for _,c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end; Conns.anchor={} end

startAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect() end; State.autoLeftPhase=1
    Conns.autoLeft=RunService.Heartbeat:Connect(function()
        if not State.autoLeftEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoLeftPhase==1 then
            local tgt=Vector3.new(POS.L1.X,root.Position.Y,POS.L1.Z); if (tgt-root.Position).Magnitude<1 then State.autoLeftPhase=2; return end
            local d=(POS.L1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoLeftPhase==2 then
            local tgt=Vector3.new(POS.L2.X,root.Position.Y,POS.L2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoLeftEnabled=false; if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1; if setAutoLeft then setAutoLeft(false) end; return end
            local d=(POS.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect() end; State.autoRightPhase=1
    Conns.autoRight=RunService.Heartbeat:Connect(function()
        if not State.autoRightEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoRightPhase==1 then
            local tgt=Vector3.new(POS.R1.X,root.Position.Y,POS.R1.Z); if (tgt-root.Position).Magnitude<1 then State.autoRightPhase=2; return end
            local d=(POS.R1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoRightPhase==2 then
            local tgt=Vector3.new(POS.R2.X,root.Position.Y,POS.R2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoRightEnabled=false; if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1; if setAutoRight then setAutoRight(false) end; return end
            local d=(POS.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAntiRagdoll=function()
    if Conns.antiRag then return end
    Conns.antiRag=RunService.Heartbeat:Connect(function()
        local char=LP.Character; if not char then return end; local hum2=char:FindFirstChildOfClass("Humanoid"); local root=char:FindFirstChild("HumanoidRootPart")
        if hum2 then local st=hum2:GetState(); if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then hum2:ChangeState(Enum.HumanoidStateType.Running); workspace.CurrentCamera.CameraSubject=hum2; pcall(function() local pm=LP.PlayerScripts:FindFirstChild("PlayerModule"); if pm then require(pm:FindFirstChild("ControlModule")):Enable() end end); if root then root.Velocity=Vector3.new(0,0,0); root.RotVelocity=Vector3.new(0,0,0) end end end
        for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled=true end end
    end)
end
stopAntiRagdoll=function() if Conns.antiRag then Conns.antiRag:Disconnect(); Conns.antiRag=nil end end
applyFPSBoost=function()
    pcall(function() setfpscap(999999999) end)
    local function processObj(v) pcall(function() if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0 elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1 elseif v:IsA("SpecialMesh") then v.TextureId="" elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false elseif v:IsA("SurfaceAppearance") or v:IsA("MaterialVariant") then v:Destroy() elseif v:IsA("Attachment") then v.Visible=false end end) end
    for _,v in pairs(workspace:GetDescendants()) do processObj(v) end
    pcall(function() local lighting=game:GetService("Lighting"); for _,v in pairs(lighting:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end; pcall(function() sethiddenproperty(game:GetService("Lighting"),"Technology",Enum.Technology.Legacy) end); local lighting2=game:GetService("Lighting"); lighting2.GlobalShadows=false; lighting2.FogEnd=9e9; lighting2.Brightness=0; local terrain=workspace:FindFirstChildOfClass("Terrain"); if terrain then pcall(function() sethiddenproperty(terrain,"Decoration",false) end); terrain.WaterReflectance=0; terrain.WaterTransparency=0.7; terrain.WaterWaveSize=0; terrain.WaterWaveSpeed=0 end end)
    workspace.DescendantAdded:Connect(function(v) if State.fpsBoostEnabled then task.spawn(processObj,v) end end)
end

-- ========== CHARACTER SETUP ==========
local function setupChar(char)
    task.wait(0.1); h=char:WaitForChild("Humanoid",5); hrp=char:WaitForChild("HumanoidRootPart",5); State.originalC0=nil
    if not h or not hrp then return end
    local head=char:FindFirstChild("Head")
    if head then
        local oldBB=head:FindFirstChild("SpeedBillboard"); if oldBB then oldBB:Destroy() end
        local bb=Instance.new("BillboardGui",head); bb.Name="SpeedBillboard"; bb.Size=UDim2.new(0,120,0,22); bb.StudsOffset=Vector3.new(0,3,0); bb.AlwaysOnTop=true
        speedLbl=Instance.new("TextLabel",bb); speedLbl.Size=UDim2.new(1,0,1,0); speedLbl.BackgroundTransparency=1; speedLbl.TextColor3=Theme.AccentGlow; speedLbl.Font=Theme.FontBold; speedLbl.TextScaled=true; speedLbl.TextStrokeTransparency=0.5; speedLbl.TextStrokeColor3=Color3.fromRGB(0,20,40)
    end
    if State.antiRagdollEnabled and not Conns.antiRag then task.wait(0.5); startAntiRagdoll() end
    if State.medusaCounterEnabled then setupMedusaCounter(char) end
    if State.animEnabled then task.wait(0.3); saveOriginalAnims(char); applyAnimPack(char) end
    if State.unwalkEnabled then State.unwalkEnabled=false; task.wait(0.3); startUnwalk() end
end
LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

RunService.Stepped:Connect(function()
    for _,p in ipairs(Players:GetPlayers()) do if p~=LP and p.Character then for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end end end
end)

UIS.JumpRequest:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root then root.Velocity=Vector3.new(root.Velocity.X,55,root.Velocity.Z) end
end)
RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root and root.Velocity.Y<-120 then root.Velocity=Vector3.new(root.Velocity.X,-120,root.Velocity.Z) end
end)

RunService.RenderStepped:Connect(function()
    if not (h and hrp) then return end; if State._tpInProgress then return end
    if State.autoLeftEnabled or State.autoRightEnabled then if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end; return end
    local md=h.MoveDirection; local spd=getCurrentSpeed()
    if md.Magnitude>0 then State.lastMoveDir=md; hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
    elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude>0 then local anyHeld=false; for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true; break end end; if anyHeld then hrp.Velocity=Vector3.new(State.lastMoveDir.X*spd,hrp.Velocity.Y,State.lastMoveDir.Z*spd) end end
    if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end
end)

RunService.Heartbeat:Connect(function(dt)
    if not (State.autoBatToggled and h and hrp) then resetBend(); return end
    local target,dist=getClosestPlayer(); if not (target and target.Character) then resetBend(); return end
    local tr=target.Character:FindFirstChild("HumanoidRootPart"); if not tr then resetBend(); return end
    local rj=getRootJoint(); if rj and not State.originalC0 then State.originalC0=rj.C0 end
    local yDiff=tr.Position.Y-hrp.Position.Y; local yVel=math.clamp(yDiff*20,-120,120)
    if dist>6 then
        local dir=(tr.Position-hrp.Position).Unit; hrp.AssemblyLinearVelocity=Vector3.new(dir.X*59,yVel,dir.Z*59)
        if rj then rj.C0=State.originalC0*CFrame.Angles(math.rad(40),0,0) end
    else
        State.crazyAngle=State.crazyAngle+dt*40
        local offsetX=math.cos(State.crazyAngle)*0.8; local offsetZ=math.sin(State.crazyAngle)*0.8
        local targetPos=tr.Position+Vector3.new(offsetX,0,offsetZ); local flatDir=Vector3.new(targetPos.X-hrp.Position.X,0,targetPos.Z-hrp.Position.Z)
        hrp.AssemblyLinearVelocity=Vector3.new(flatDir.Unit.X*200,yVel,flatDir.Unit.Z*200)
        if rj then local t=tick(); local fwdBack=math.sin(t*25)*math.rad(50); local sideways=math.cos(t*20)*math.rad(30); rj.C0=State.originalC0*CFrame.Angles(fwdBack,0,sideways) end
        tryHitBat()
    end
end)

UIS.InputBegan:Connect(function(inp,gp)
    if gp then return end
    if inp.UserInputType~=Enum.UserInputType.Keyboard and inp.UserInputType~=Enum.UserInputType.Gamepad1 then return end
    local kc=inp.KeyCode
    if (State.autoLeftEnabled or State.autoRightEnabled) then if MOVE_KEYS[kc] then return end end
    if kc==Keys.speed then toggleSpeedType()
    elseif kc==Keys.lagger then toggleLagger()
    elseif kc==Keys.autoBat then State.autoBatToggled=not State.autoBatToggled; if not State.autoBatToggled then resetBend() end; setAutoBat(State.autoBatToggled); autoSaveConfig()
    elseif kc==Keys.autoLeft then State.autoLeftEnabled=not State.autoLeftEnabled; if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end; if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    elseif kc==Keys.autoRight then State.autoRightEnabled=not State.autoRightEnabled; if setAutoRight then setAutoRight(State.autoRightEnabled) end; if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    elseif kc==Keys.dropBrainrot then task.spawn(runDropBrainrot)
    elseif kc==Keys.tpDown then tpToGround()
    elseif kc==Keys.guiHide then toggleGuiVis() end
end)

task.spawn(function() while task.wait(0.5) do pcall(function() if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end) end end)

local function loadConfig()
    local hasFile=false; pcall(function() hasFile=isfile("JispiHubConfig.json") end); if not hasFile then return end
    local ok,cfg=pcall(function() return HttpService:JSONDecode(readfile("JispiHubConfig.json")) end); if not ok or not cfg then return end
    if cfg.normalSpeed and type(cfg.normalSpeed)=="number" then State.normalSpeed=cfg.normalSpeed; normalBox.Text=tostring(cfg.normalSpeed) end
    if cfg.carrySpeed  and type(cfg.carrySpeed)=="number"  then State.carrySpeed=cfg.carrySpeed;   carryBox.Text=tostring(cfg.carrySpeed)   end
    if cfg.laggerSpeed and type(cfg.laggerSpeed)=="number" then State.laggerSpeed=cfg.laggerSpeed; laggerBox.Text=tostring(cfg.laggerSpeed) end
    if cfg.laggerCarrySpeed and type(cfg.laggerCarrySpeed)=="number" then State.laggerCarrySpeed=cfg.laggerCarrySpeed; carryLaggerBox.Text=tostring(cfg.laggerCarrySpeed) end
    if cfg.speedType=="normal" or cfg.speedType=="carry" then State.speedType=cfg.speedType end
    if type(cfg.laggerActive)=="boolean" then State.laggerActive=cfg.laggerActive end
    if cfg.autoBatKey  and Enum.KeyCode[cfg.autoBatKey]    then Keys.autoBat=Enum.KeyCode[cfg.autoBatKey];   if autoBatKeyBtn  then autoBatKeyBtn.Text="["..cfg.autoBatKey.."]"   end end
    if cfg.speedKey    and Enum.KeyCode[cfg.speedKey]      then Keys.speed=Enum.KeyCode[cfg.speedKey]        end
    if cfg.laggerKey   and Enum.KeyCode[cfg.laggerKey]     then Keys.lagger=Enum.KeyCode[cfg.laggerKey]      end
    if cfg.autoLeftKey  and Enum.KeyCode[cfg.autoLeftKey]  then Keys.autoLeft=Enum.KeyCode[cfg.autoLeftKey];   if autoLeftKeyBtn  then autoLeftKeyBtn.Text="["..cfg.autoLeftKey.."]"   end end
    if cfg.autoRightKey and Enum.KeyCode[cfg.autoRightKey] then Keys.autoRight=Enum.KeyCode[cfg.autoRightKey]; if autoRightKeyBtn then autoRightKeyBtn.Text="["..cfg.autoRightKey.."]" end end
    if cfg.tpDownKey    and Enum.KeyCode[cfg.tpDownKey]    then Keys.tpDown=Enum.KeyCode[cfg.tpDownKey];       if tpDownKeyBtn    then tpDownKeyBtn.Text="["..cfg.tpDownKey.."]"        end end
    if cfg.grabRadius and type(cfg.grabRadius)=="number" then AutoSteal.Radius=cfg.grabRadius; if progressRadLbl then progressRadLbl.Text="Radius: "..cfg.grabRadius end end
    if cfg.autoStealEnabled then AutoSteal.Enabled=true; setInstaGrab(true); pcall(startAutoSteal) end
    if cfg.infJump      then State.infJumpEnabled=true;      setInfJump(true)      end
    if cfg.antiRagdoll  then State.antiRagdollEnabled=true;  setAntiRag(true);     startAntiRagdoll() end
    if cfg.fpsBoost     then State.fpsBoostEnabled=true;     setFps(true);         applyFPSBoost()    end
    if cfg.medusaCounter then State.medusaCounterEnabled=true; setMedusaCounter(true); setupMedusaCounter(LP.Character) end
    if cfg.dropBrainrotKey and Enum.KeyCode[cfg.dropBrainrotKey] then Keys.dropBrainrot=Enum.KeyCode[cfg.dropBrainrotKey]; if dropBrainrotKeyBtn then dropBrainrotKeyBtn.Text="["..cfg.dropBrainrotKey.."]"                    bat:Activate() -- Ø¨ÙØ®
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local ContentProvider = game:GetService("ContentProvider")
local LP = Players.LocalPlayer

local State = {
    normalSpeed = 60, carrySpeed = 30, laggerSpeed = 13, laggerCarrySpeed = 13,
    speedType = "normal", laggerActive = false, autoBatToggled = false,
    hittingCooldown = false, infJumpEnabled = false, antiRagdollEnabled = false,
    fpsBoostEnabled = false, guiVisible = true, isStealing = false,
    stealStartTime = nil, lastStealTick = 0, medusaLastUsed = 0,
    medusaDebounce = false, medusaCounterEnabled = false, dropBrainrotActive = false,
    autoLeftEnabled = false, autoRightEnabled = false, autoLeftPhase = 1,
    autoRightPhase = 1, _tpInProgress = false, lastMoveDir = Vector3.new(0,0,0),
    animEnabled = false, unwalkEnabled = false, originalC0 = nil, crazyAngle = 0,
}

local Keys = {
    autoBat = Enum.KeyCode.E, speed = Enum.KeyCode.Q, lagger = Enum.KeyCode.C,
    guiHide = Enum.KeyCode.LeftControl, autoLeft = Enum.KeyCode.L,
    autoRight = Enum.KeyCode.R, dropBrainrot = Enum.KeyCode.H, tpDown = Enum.KeyCode.T,
}

local main, closeBtn, miniBtn
local toggleGuiVis

local MobileButtons = { Visible = true, Locked = false, BtnsObjects = {}, BtnRefs = {}, Buttons = {}, HideGuiBtn = nil }

local MB_BTN_W = 58
local MB_BTN_H = 48
local MB_COLS = 2
local MB_ROWS = 4
local MB_PAD = 3
local MB_CORNER = 14
local MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
local MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
local mbPanel = nil

local Theme = {
    Background      = Color3.fromRGB(8, 10, 20),
    BackgroundSecondary = Color3.fromRGB(12, 15, 28),
    Header          = Color3.fromRGB(5, 8, 18),
    Element         = Color3.fromRGB(14, 18, 35),
    ElementHover    = Color3.fromRGB(20, 26, 50),
    Text            = Color3.fromRGB(200, 220, 255),
    SubText         = Color3.fromRGB(80, 110, 160),
    Accent          = Color3.fromRGB(0, 180, 255),
    AccentDark      = Color3.fromRGB(0, 100, 200),
    AccentGlow      = Color3.fromRGB(0, 220, 255),
    AccentDim       = Color3.fromRGB(0, 60, 120),
    Border          = Color3.fromRGB(0, 60, 120),
    BorderLight     = Color3.fromRGB(0, 100, 180),
    Success         = Color3.fromRGB(0, 255, 160),
    Danger          = Color3.fromRGB(255, 50, 80),
    Keybind         = Color3.fromRGB(10, 20, 45),
    KeybindText     = Color3.fromRGB(0, 180, 255),
    ToggleOff       = Color3.fromRGB(18, 22, 45),
    Font            = Enum.Font.Gotham,
    FontBold        = Enum.Font.GothamBold,
    FontBlack       = Enum.Font.GothamBlack,
    FontMedium      = Enum.Font.GothamMedium,
}

local AutoSteal = {
    Enabled = false, Radius = 20, Duration = 1.3, IsStealing = false,
    Data = {}, ProgressFill = nil, ProgressText = nil,
}

local function isMyPlotByName(plotName)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return false end
    local plot = plots:FindFirstChild(plotName); if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then local yb = sign:FindFirstChild("YourBase"); if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end end
    return false
end

local function findNearestPrompt()
    local char = LP.Character; local root = char and char:FindFirstChild("HumanoidRootPart"); if not root then return nil end
    local plots = workspace:FindFirstChild("Plots"); if not plots then return nil end
    local bestPrompt, bestDist, bestName = nil, math.huge, nil
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local podiums = plot:FindFirstChild("AnimalPodiums"); if not podiums then continue end
        for _, pod in ipairs(podiums:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base"); local spawn = base and base:FindFirstChild("Spawn")
                if spawn then
                    local dist = (spawn.Position - root.Position).Magnitude
                    if dist < bestDist and dist <= AutoSteal.Radius then
                        local att = spawn:FindFirstChild("PromptAttachment")
                        if att then for _, child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") then bestPrompt, bestDist, bestName = child, dist, pod.Name; break end
                        end end
                    end
                end
            end)
        end
    end
    return bestPrompt, bestDist, bestName
end

local function executeSteal(prompt, animalName)
    if AutoSteal.IsStealing then return end
    if not AutoSteal.Data[prompt] then
        AutoSteal.Data[prompt] = {hold = {}, trigger = {}, ready = true}
        pcall(function()
            if getconnections then
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(AutoSteal.Data[prompt].hold, c.Function) end end
                for _, c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(AutoSteal.Data[prompt].trigger, c.Function) end end
            end
        end)
    end
    local data = AutoSteal.Data[prompt]; if not data.ready then return end
    data.ready = false; AutoSteal.IsStealing = true; local startTime = tick()
    local conn; conn = RunService.Heartbeat:Connect(function()
        if not AutoSteal.IsStealing then conn:Disconnect(); return end
        local prog = math.clamp((tick() - startTime) / AutoSteal.Duration, 0, 1)
        if AutoSteal.ProgressFill then AutoSteal.ProgressFill.Size = UDim2.new(prog, 0, 1, 0) end
    end)
    task.spawn(function()
        for _, f in ipairs(data.hold) do task.spawn(f) end
        task.wait(AutoSteal.Duration)
        for _, f in ipairs(data.trigger) do task.spawn(f) end
        AutoSteal.IsStealing = false; data.ready = true
        task.wait(0.6)
        if not AutoSteal.IsStealing and AutoSteal.ProgressFill then
            TweenService:Create(AutoSteal.ProgressFill, TweenInfo.new(0.4), {Size = UDim2.new(0,0,1,0)}):Play()
        end
    end)
end

local autoStealConnection = nil
local function startAutoSteal()
    if autoStealConnection then return end
    autoStealConnection = RunService.Heartbeat:Connect(function()
        if AutoSteal.Enabled and not AutoSteal.IsStealing then
            local p, _, name = findNearestPrompt(); if p then executeSteal(p, name) end
        end
    end)
end
local function stopAutoSteal()
    if autoStealConnection then autoStealConnection:Disconnect(); autoStealConnection = nil end
    AutoSteal.IsStealing = false
    for k, v in pairs(AutoSteal.Data) do if v.ready ~= nil then v.ready = true end end
end

local MOVE_KEYS = {[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local DROP_ASCEND_DURATION = 0.2; local DROP_ASCEND_SPEED = 150
local POS = {L1=Vector3.new(-476.48,-6.28,92.73),L2=Vector3.new(-483.12,-4.95,94.80),R1=Vector3.new(-476.16,-6.52,25.62),R2=Vector3.new(-483.04,-5.09,23.14)}
local Conns = {autoSteal=nil,antiRag=nil,autoLeft=nil,autoRight=nil,anchor={},progress=nil}
local h, hrp, speedLbl
local setAutoLeft, setAutoRight
local setInstaGrab, setAutoBat, setInfJump, setAntiRag, setFps, setMedusaCounter
local setAnimToggle, setUnwalkToggle
local setupMedusaCounter, stopMedusaCounter, startAntiRagdoll, stopAntiRagdoll, applyFPSBoost
local modeValLbl, normalBox, carryBox, laggerBox, carryLaggerBox
local autoBatKeyBtn, speedKeyBtn, laggerKeyBtn, autoLeftKeyBtn, autoRightKeyBtn, guiHideKeyBtn, dropBrainrotKeyBtn, tpDownKeyBtn
local setSpeedToggleUI, setLaggerToggleUI
local progressRadLbl
local startAutoLeft, stopAutoLeft, startAutoRight, stopAutoRight

local function getRootJoint()
    local char2 = LP.Character; local torso = char2 and char2:FindFirstChild("LowerTorso")
    return torso and torso:FindFirstChild("Root")
end
local function resetBend()
    local rj = getRootJoint()
    if rj and State.originalC0 then rj.C0 = State.originalC0; State.originalC0 = nil end
end
local function getBat()
    local char = LP.Character; if not char then return nil end
    local tool = char:FindFirstChild("Bat"); if tool then return tool end
    local bp2 = LP:FindFirstChild("Backpack")
    if bp2 then tool = bp2:FindFirstChild("Bat"); if tool then tool.Parent = char; return tool end end
    return nil
end
local function tryHitBat()
    if State.hittingCooldown then return end; State.hittingCooldown = true
    pcall(function()
        local bat = getBat(); if bat then bat:Activate(); local ev = bat:FindFirstChildWhichIsA("RemoteEvent"); if ev then ev:FireServer() end end
    end)
    task.delay(0.08, function() State.hittingCooldown = false end)
end
local function getClosestPlayer()
    if not hrp then return nil, math.huge end
    local cp, cd = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if tr then local d = (hrp.Position - tr.Position).Magnitude; if d < cd then cd = d; cp = p end end
        end
    end
    return cp, cd
end

local saveDebounce = false
local function autoSaveConfig()
    if saveDebounce then return end; saveDebounce = true
    task.delay(0.5, function()
        local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
        pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end); saveDebounce = false
    end)
end
local function forceSaveConfig()
    local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
    pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end)
end
localsteal(steal) local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local Conns = {autoSteal=nil,antiRag=nil,anchor={},progress=nil,aimbot=nil,batCounter=nil}
local PLOT_CACHE_DURATION = 2
local PROMPT_CACHE_REFRESH = 0.15
local STEAL_COOLDOWN = 0.1
local MEDUSA_COOLDOWN = 25
local BAT_COUNTER_COOLDOWN = 0
local batCounterDebounce = false
local batCounterLastUsed = 0
local progressRadLbl,progressFill,progressPct
local modeValLbl
local _aimTarget = nil
local _aimLastScan = 0

local VYSE_AIMBOT_SPEED = 56.5
local VYSE_HIT_DIST = 5
local SWING_COOLDOWN = 0.08
local hittingCooldown = false

local desyncEnabled = false
local setDesyncVisual = nil
local desyncActive = false
local desyncSpeedConn = nil
local G_desyncAnimate = nil

local fpsBoostEnabled = false

local function refreshUIToggles()
    if setSpeedToggleUI then setSpeedToggleUI(State.speedType == "carry") end
    if setLaggerToggleUI then setLaggerToggleUI(State.laggerActive) end
    if modeValLbl then
        if State.laggerActive then modeValLbl.Text = (State.speedType=="normal") and "â¡ Lagger Normal" or "â¡ Lagger Carry"
        else modeValLbl.Text = (State.speedType=="normal") and "Normal" or "Carry" end
    end
end
local function toggleSpeedType()
    State.speedType = (State.speedType=="normal") and "carry" or "normal"; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.carrySpeed then MobileButtons.BtnRefs.carrySpeed:SetAttribute("MB_On", State.speedType=="carry") end
end
local function toggleLagger()
    State.laggerActive = not State.laggerActive; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.lagger then MobileButtons.BtnRefs.lagger:SetAttribute("MB_On", State.laggerActive) end
end
local function getCurrentSpeed()
    if State.laggerActive then return State.speedType=="normal" and State.laggerSpeed or State.laggerCarrySpeed
    else return State.speedType=="normal" and State.normalSpeed or State.carrySpeed end
end
local function getAutoMoveSpeed()
    return State.laggerActive and State.laggerSpeed or State.normalSpeed
end
local function tpToGround()
    local char = LP.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
    local rp = RaycastParams.new(); rp.FilterType = Enum.RaycastFilterType.Exclude; rp.FilterDescendantsInstances = {char}
    local rr = workspace:Raycast(root.Position, Vector3.new(0,-500,0), rp)
    if rr then root.CFrame = CFrame.new(rr.Position + Vector3.new(0,3,0)) else root.CFrame = root.CFrame * CFrame.new(0,-20,0) end
end
local function runDropBrainrot()
    if State.dropBrainrotActive then return end
    local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if not root then return end
    State.dropBrainrotActive=true; local t0=tick(); local dc
    dc=RunService.Heartbeat:Connect(function()
        local r=char and char:FindFirstChild("HumanoidRootPart"); if not r then dc:Disconnect(); State.dropBrainrotActive=false; return end
        if tick()-t0>=DROP_ASCEND_DURATION then
            dc:Disconnect()
            local rp=RaycastParams.new(); rp.FilterDescendantsInstances={char}; rp.FilterType=Enum.RaycastFilterType.Exclude
            local rr=workspace:Raycast(r.Position,Vector3.new(0,-2000,0),rp)
            if rr then local hum2=char:FindFirstChildOfClass("Humanoid"); local off=(hum2 and hum2.HipHeight or 2)+(r.Size.Y/2); r.CFrame=CFrame.new(r.Position.X,rr.Position.Y+off,r.Position.Z); r.AssemblyLinearVelocity=Vector3.new(0,0,0) end
            State.dropBrainrotActive=false; return
        end
        r.AssemblyLinearVelocity=Vector3.new(r.AssemblyLinearVelocity.X,DROP_ASCEND_SPEED,r.AssemblyLinearVelocity.Z)
    end)
end

-- ========== MOBILE PANEL ==========
local function createMobilePanel()
    local panel = Instance.new("ScreenGui")
    panel.Name="OmniHubButtons"; panel.Parent=LP:WaitForChild("PlayerGui"); panel.ResetOnSpawn=false; panel.ZIndexBehavior=Enum.ZIndexBehavior.Sibling

    mbPanel = Instance.new("Frame", panel)
    mbPanel.Name = "MobilePanel"
    mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H)
    mbPanel.Position = UDim2.new(1, -(MB_TOTAL_W+8), 0.5, -(MB_TOTAL_H/2))
    mbPanel.BackgroundColor3 = Theme.Background
    mbPanel.BorderSizePixel = 0; mbPanel.Active = true; mbPanel.ZIndex = 50
    Instance.new("UICorner", mbPanel).CornerRadius = UDim.new(0, MB_CORNER+4)
    local mbStroke = Instance.new("UIStroke", mbPanel)
    mbStroke.Color = Theme.Accent; mbStroke.Thickness = 1.5; mbStroke.Transparency = 0.45

    local dragStart, startPos, moved = nil, nil, false
    local DRAG_THRESHOLD = 8
    mbPanel.InputBegan:Connect(function(inp)
        if MobileButtons.Locked then return end
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
            dragStart=inp.Position; startPos=mbPanel.Position; moved=false
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if MobileButtons.Locked or not dragStart then return end
        if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then
            local d=inp.Position-dragStart
            if not moved and (math.abs(d.X)+math.abs(d.Y))>DRAG_THRESHOLD then moved=true end
            if moved then mbPanel.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+d.X,startPos.Y.Scale,startPos.Y.Offset+d.Y) end
        end
    end)
    UIS.InputEnded:Connect(function(inp)
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragStart=nil; moved=false end
    end)

    local function makeMobileBtn(label, row, col, onClick)
        local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
        local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
        local btn = Instance.new("TextButton", mbPanel)
        btn.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
        btn.Position = UDim2.new(0, x, 0, y)
        btn.BackgroundColor3 = Theme.Element; btn.BorderSizePixel = 0
        btn.Text = ""; btn.AutoButtonColor = false; btn.ZIndex = 52
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, MB_CORNER)
        local bs = Instance.new("UIStroke", btn); bs.Color = Theme.Accent; bs.Thickness = 1; bs.Transparency = 0.7
        local lbl = Instance.new("TextLabel", btn)
        lbl.Size = UDim2.new(1,0,1,0); lbl.BackgroundTransparency = 1
        lbl.Text = label; lbl.TextColor3 = Theme.Accent; lbl.Font = Theme.FontBold
        lbl.TextSize = 9; lbl.TextWrapped = true; lbl.TextXAlignment = Enum.TextXAlignment.Center; lbl.TextYAlignment = Enum.TextYAlignment.Center; lbl.ZIndex = 56

        btn.MouseButton1Down:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.05), {BackgroundColor3=Theme.ElementHover}):Play()
            TweenService:Create(bs, TweenInfo.new(0.05), {Transparency=0}):Play()
        end)
        btn.MouseButton1Up:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.10), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.10), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseEnter:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.ElementHover}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.4}):Play()
            end
        end)
        btn.MouseLeave:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseButton1Click:Connect(function() if onClick then pcall(onClick) end end)

        btn:SetAttribute("MB_On", false)
        btn.AttributeChanged:Connect(function(a)
            if a == "MB_On" then
                local on = btn:GetAttribute("MB_On")
                if on then
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Accent}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(5,5,15)}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0, Color = Theme.AccentGlow}):Play()
                    bs.Thickness = 2
                else
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Element}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Theme.Accent}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0.7, Color = Theme.Accent}):Play()
                    bs.Thickness = 1
                end
            end
        end)
        table.insert(MobileButtons.BtnsObjects, btn)
        return btn
    end

    local alMbBtn = makeMobileBtn("AUTO\nLEFT", 1, 1, function()
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoLeftEnabled = not State.autoLeftEnabled
        if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end
        if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoLeft = alMbBtn

    local arMbBtn = makeMobileBtn("AUTO\nRIGHT", 1, 2, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoRightEnabled = not State.autoRightEnabled
        if setAutoRight then setAutoRight(State.autoRightEnabled) end
        if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoRight = arMbBtn

    local batMbBtn = makeMobileBtn("BAT", 2, 1, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        State.autoBatToggled = not State.autoBatToggled
        if not State.autoBatToggled then resetBend() end
        if setAutoBat then setAutoBat(State.autoBatToggled) end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoBat = batMbBtn

    makeMobileBtn("DROP", 2, 2, function() runDropBrainrot() end)

    local carryMbBtn = makeMobileBtn("CARRY\nSPD", 3, 1, function() toggleSpeedType() end)
    MobileButtons.BtnRefs.carrySpeed = carryMbBtn

    local lagMbBtn = makeMobileBtn("LAG", 3, 2, function() toggleLagger() end)
    MobileButtons.BtnRefs.lagger = lagMbBtn

    makeMobileBtn("TP\nDOWN", 4, 1, function() tpToGround() end)

    local guiMbBtn = makeMobileBtn("GUI", 4, 2, function() if toggleGuiVis then toggleGuiVis() end end)
    MobileButtons.HideGuiBtn = guiMbBtn

    task.spawn(function()
        while task.wait(0.2) do
            pcall(function()
                if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
                if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
                if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
                if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
                if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
            end)
        end
    end)

    task.spawn(function()
        task.wait(0.1)
        if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
        if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
        if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
        if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
        if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
    end)

    for _, btn in pairs(MobileButtons.BtnsObjects) do btn.Visible = MobileButtons.Visible end
    mbPanel.Visible = MobileButtons.Visible
end

local function repositionMbButtons()
    if not mbPanel then return end
    local idx = 0
    for _, child in ipairs(mbPanel:GetChildren()) do
        if child:IsA("TextButton") then
            local row = math.floor(idx / MB_COLS) + 1
            local col = (idx % MB_COLS) + 1
            local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
            local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
            child.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
            child.Position = UDim2.new(0, x, 0, y)
            idx = idx + 1
        end
    end
end

local function applyMbBtnSize(w, h2)
    w = math.clamp(w, 40, 120); h2 = math.clamp(h2, 36, 120)
    MB_BTN_W = w; MB_BTN_H = h2
    MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
    MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
    if mbPanel then mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H); repositionMbButtons() end
end

-- ========== ANIMATIONS ==========
local Anims = {idle1="rbxassetid://133806214992291",idle2="rbxassetid://94970088341563",walk="rbxassetid://707897309",run="rbxassetid://707861613",jump="rbxassetid://116936326516985",fall="rbxassetid://116936326516985",climb="rbxassetid://116936326516985",swim="rbxassetid://116936326516985",swimidle="rbxassetid://116936326516985"}
task.spawn(function() pcall(function() ContentProvider:PreloadAsync({Anims.idle1,Anims.idle2,Anims.walk,Anims.run,Anims.jump,Anims.fall,Anims.climb,Anims.swim,Anims.swimidle}) end) end)
local animHeartbeatConn=nil; local savedAnimate=nil; local originalAnims=nil
local function isPackAnim(id) if not id then return false end; for _,v in pairs(Anims) do if v==id then return true end end; return false end
local function saveOriginalAnims(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function g(obj) return obj and obj.AnimationId or nil end; local ids={idle1=g(animate.idle and animate.idle.Animation1),idle2=g(animate.idle and animate.idle.Animation2),walk=g(animate.walk and animate.walk.WalkAnim),run=g(animate.run and animate.run.RunAnim),jump=g(animate.jump and animate.jump.JumpAnim),fall=g(animate.fall and animate.fall.FallAnim),climb=g(animate.climb and animate.climb.ClimbAnim),swim=g(animate.swim and animate.swim.Swim),swimidle=g(animate.swimidle and animate.swimidle.SwimIdle)}; if not isPackAnim(ids.walk) then originalAnims=ids end end
local function applyAnimPack(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,Anims.idle1); s(animate.idle and animate.idle.Animation2,Anims.idle2); s(animate.walk and animate.walk.WalkAnim,Anims.walk); s(animate.run and animate.run.RunAnim,Anims.run); s(animate.jump and animate.jump.JumpAnim,Anims.jump); s(animate.fall and animate.fall.FallAnim,Anims.fall); s(animate.climb and animate.climb.ClimbAnim,Anims.climb); s(animate.swim and animate.swim.Swim,Anims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,Anims.swimidle) end
local function restoreOriginalAnims(char) if not originalAnims then return end; local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj and id then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,originalAnims.idle1); s(animate.idle and animate.idle.Animation2,originalAnims.idle2); s(animate.walk and animate.walk.WalkAnim,originalAnims.walk); s(animate.run and animate.run.RunAnim,originalAnims.run); s(animate.jump and animate.jump.JumpAnim,originalAnims.jump); s(animate.fall and animate.fall.FallAnim,originalAnims.fall); s(animate.climb and animate.climb.ClimbAnim,originalAnims.climb); s(animate.swim and animate.swim.Swim,originalAnims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,originalAnims.swimidle); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end
local function startAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then saveOriginalAnims(char); applyAnimPack(char); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end; animHeartbeatConn=RunService.Heartbeat:Connect(function() if not State.animEnabled then return end; local c=LP.Character; if c then applyAnimPack(c) end end) end
local function stopAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then restoreOriginalAnims(char) end end
local function startUnwalk() if State.unwalkEnabled then return end; State.unwalkEnabled=true; local c=LP.Character; if not c then return end; local hum=c:FindFirstChildOfClass("Humanoid"); if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end; local anim=c:FindFirstChild("Animate"); if anim then savedAnimate=anim:Clone(); anim:Destroy() end end
local function stopUnwalk() if not State.unwalkEnabled then return end; State.unwalkEnabled=false; local c=LP.Character; if c and savedAnimate then savedAnimate.Parent=c; savedAnimate.Disabled=false; savedAnimate=nil end; task.spawn(function() task.wait(0.15); local char=LP.Character; if not char then return end; if State.animEnabled then saveOriginalAnims(char); applyAnimPack(char) else restoreOriginalAnims(char) end end) end

-- ========== MAIN GUI ==========
local gui = Instance.new("ScreenGui")
gui.Name="OmniHubGUI"; gui.ResetOnSpawn=false; gui.DisplayOrder=10; gui.IgnoreGuiInset=true; gui.Parent=LP:WaitForChild("PlayerGui")

local stealProgressBar = Instance.new("Frame", gui)
stealProgressBar.Size=UDim2.new(0,280,0,6); stealProgressBar.Position=UDim2.new(0.5,-140,0.9,0)
stealProgressBar.BackgroundColor3=Color3.fromRGB(8,12,25); stealProgressBar.BackgroundTransparency=0; stealProgressBar.BorderSizePixel=0; stealProgressBar.ZIndex=100
Instance.new("UICorner",stealProgressBar).CornerRadius=UDim.new(1,0)
local barStroke=Instance.new("UIStroke",stealProgressBar); barStroke.Color=Theme.AccentDim; barStroke.Thickness=1
local barFill=Instance.new("Frame",stealProgressBar); barFill.Size=UDim2.new(0,0,1,0); barFill.BackgroundColor3=Theme.Accent; barFill.BorderSizePixel=0
Instance.new("UICorner",barFill).CornerRadius=UDim.new(1,0); AutoSteal.ProgressFill=barFill

local function makeStealBarDraggable(frame)
    local dragging=false; local dragInput,dragStart,startPos,activeDragConn
    local function stopDrag() dragging=false; dragInput=nil; dragStart=nil; startPos=nil; if activeDragConn then activeDragConn:Disconnect(); activeDragConn=nil end end
    frame.InputBegan:Connect(function(input) if MobileButtons.Locked then return end; if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=input.Position; startPos=frame.Position; input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End or input.UserInputState==Enum.UserInputState.Cancelled then stopDrag() end end) end end)
    frame.InputChanged:Connect(function(input) if not dragging then return end; if MobileButtons.Locked then stopDrag(); return end; if input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch then dragInput=input end end)
    UIS.InputChanged:Connect(function(input) if input==dragInput and dragging then if MobileButtons.Locked then stopDrag(); return end; local delta=input.Position-dragStart; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y) end end)
end
makeStealBarDraggable(stealProgressBar)
local function saveStealBarPosition() local pos=stealProgressBar.Position; pcall(function() writefile("JispiStealBarPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function loadStealBarPosition() local savedPos=nil; pcall(function() savedPos=readfile("JispiStealBarPos.txt") end); if savedPos and savedPos~="" then local parts={}; for part in string.gmatch(savedPos,"[^,]+") do table.insert(parts,part) end; if #parts>=4 then stealProgressBar.Position=UDim2.new(tonumber(parts[1]),tonumber(parts[2]),tonumber(parts[3]),tonumber(parts[4])) end end end
loadStealBarPosition()
local lastStealBarSave=0; stealProgressBar:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastStealBarSave>0.5 then lastStealBarSave=tick(); saveStealBarPosition() end end)

-- ========== MAIN FRAME ==========
main = Instance.new("Frame", gui)
main.Name="Main"; main.Size=UDim2.new(0,320,0,470); main.Position=UDim2.new(0,20,0,20)
main.BackgroundColor3=Theme.Background; main.BorderSizePixel=0; main.Active=true; main.ClipsDescendants=true
Instance.new("UICorner",main).CornerRadius=UDim.new(0,16)
local outerGlow=Instance.new("UIStroke",main); outerGlow.Color=Theme.Accent; outerGlow.Thickness=1.5; outerGlow.Transparency=0.3
local glowPulse=Instance.new("Frame",main); glowPulse.Size=UDim2.new(1,8,1,8); glowPulse.Position=UDim2.new(0,-4,0,-4)
glowPulse.BackgroundColor3=Theme.Accent; glowPulse.BackgroundTransparency=0.88; glowPulse.BorderSizePixel=0; glowPulse.ZIndex=-1
Instance.new("UICorner",glowPulse).CornerRadius=UDim.new(0,20)
task.spawn(function()
    while true do
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.78}):Play()
        task.wait(1.8)
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.92}):Play()
        task.wait(1.8)
    end
end)

local function saveMainPosition() local pos=main.Position; pcall(function() writefile("JispiHubGUIPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function saveMiniPosition() if miniBtn then local pos=miniBtn.Position; pcall(function() writefile("JispiMiniPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end end
local function loadMainPosition() local sp=nil; pcall(function() sp=readfile("JispiHubGUIPos.txt") end); if sp and sp~="" then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then main.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])); if closeBtn then closeBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2])+320-34,tonumber(p[3]),tonumber(p[4])+18) end end end end
local function loadMiniPosition() local sp=nil; pcall(function() sp=readfile("JispiMiniPos.txt") end); if sp and sp~="" and miniBtn then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then miniBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])) end end end

-- ========== HEADER ==========
local header=Instance.new("Frame",main)
header.Size=UDim2.new(1,0,0,52); header.BackgroundColor3=Theme.Header; header.BorderSizePixel=0; header.ZIndex=5
Instance.new("UICorner",header).CornerRadius=UDim.new(0,16)
local headerAccentLine=Instance.new("Frame",header)
headerAccentLine.Size=UDim2.new(1,0,0,2); headerAccentLine.Position=UDim2.new(0,0,1,-2)
headerAccentLine.BackgroundColor3=Theme.Accent; headerAccentLine.BorderSizePixel=0; headerAccentLine.ZIndex=6
local lineGlow=Instance.new("UIGradient",headerAccentLine)
lineGlow.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,30)),ColorSequenceKeypoint.new(0.5,Theme.AccentGlow),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,30))})
local titleLbl=Instance.new("TextLabel",header)
titleLbl.Size=UDim2.new(1,0,1,-4); titleLbl.Position=UDim2.new(0,0,0,0)
titleLbl.BackgroundTransparency=1; titleLbl.Text="â  RBG HUB  â"
titleLbl.TextColor3=Theme.AccentGlow; titleLbl.Font=Theme.FontBlack; titleLbl.TextSize=17
titleLbl.TextXAlignment=Enum.TextXAlignment.Center; titleLbl.ZIndex=6
local versionLbl=Instance.new("TextLabel",header)
versionLbl.Size=UDim2.new(0,40,0,16); versionLbl.Position=UDim2.new(1,-50,0.5,-8)
versionLbl.BackgroundTransparency=1; versionLbl.Text="v1.0"; versionLbl.TextColor3=Theme.AccentDim
versionLbl.Font=Theme.Font; versionLbl.TextSize=9; versionLbl.ZIndex=6

-- ========== CLOSE / MINI ==========
closeBtn=Instance.new("TextButton",gui)
closeBtn.Size=UDim2.new(0,28,0,28); closeBtn.BackgroundColor3=Color3.fromRGB(14,18,35)
closeBtn.BorderSizePixel=0; closeBtn.Text="â"; closeBtn.TextColor3=Theme.SubText
closeBtn.Font=Theme.FontBold; closeBtn.TextSize=14; closeBtn.ZIndex=50
Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(0,8)
local closeStroke=Instance.new("UIStroke",closeBtn); closeStroke.Color=Theme.Border; closeStroke.Thickness=1.5
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Danger,TextColor3=Color3.new(1,1,1)}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Danger}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Color3.fromRGB(14,18,35),TextColor3=Theme.SubText}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Border}):Play() end)
closeBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)

miniBtn=Instance.new("ImageButton",gui); miniBtn.Name="OmniMiniButton"; miniBtn.Size=UDim2.new(0,44,0,44)
miniBtn.Position=UDim2.new(0,20,0,100); miniBtn.BackgroundColor3=Theme.Background; miniBtn.Image=""; miniBtn.BorderSizePixel=0; miniBtn.Visible=false; miniBtn.ZIndex=50
Instance.new("UICorner",miniBtn).CornerRadius=UDim.new(0,12)
local miniStroke=Instance.new("UIStroke",miniBtn); miniStroke.Color=Theme.Accent; miniStroke.Thickness=1.8
local miniIcon=Instance.new("TextLabel",miniBtn); miniIcon.Size=UDim2.new(1,0,1,0); miniIcon.Text="â"; miniIcon.TextColor3=Theme.AccentGlow; miniIcon.Font=Theme.FontBlack; miniIcon.TextSize=20; miniIcon.BackgroundTransparency=1
miniBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)
miniBtn.MouseEnter:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Element}):Play() end)
miniBtn.MouseLeave:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Background}):Play() end)

local function makeMainDraggable(frame)
    local dragging,dragInput,dragStart,startPos,startCloseBtnPos=false,nil,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; if closeBtn then startCloseBtnPos=closeBtn.Position end; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy); if closeBtn and startCloseBtnPos then closeBtn.Position=UDim2.new(startCloseBtnPos.X.Scale,startCloseBtnPos.X.Offset+dx,startCloseBtnPos.Y.Scale,startCloseBtnPos.Y.Offset+dy) end end end)
end
local function makeMiniDraggable(frame)
    local dragging,dragInput,dragStart,startPos=false,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy) end end)
end
makeMainDraggable(main); makeMiniDraggable(miniBtn)
local lastSaveTime=0; main:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastSaveTime>0.5 then lastSaveTime=tick(); saveMainPosition() end end)
local lastMiniSaveTime=0; miniBtn:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastMiniSaveTime>0.5 then lastMiniSaveTime=tick(); saveMiniPosition() end end)
local function updateCloseButtonPosition() if main.Visible then local p=main.Position; closeBtn.Position=UDim2.new(p.X.Scale,p.X.Offset+320-34,p.Y.Scale,p.Y.Offset+18) end end
main:GetPropertyChangedSignal("Position"):Connect(updateCloseButtonPosition); main:GetPropertyChangedSignal("Visible"):Connect(updateCloseButtonPosition)

-- ========== TAB SYSTEM ==========
local tabFrame=Instance.new("Frame",main)
tabFrame.Size=UDim2.new(1,-16,0,30); tabFrame.Position=UDim2.new(0,8,0,56)
tabFrame.BackgroundTransparency=1; tabFrame.BorderSizePixel=0; tabFrame.ZIndex=3
local tabLayout=Instance.new("UIListLayout",tabFrame)
tabLayout.FillDirection=Enum.FillDirection.Horizontal; tabLayout.SortOrder=Enum.SortOrder.LayoutOrder
tabLayout.Padding=UDim.new(0,4); tabLayout.HorizontalAlignment=Enum.HorizontalAlignment.Center

local scroll=Instance.new("ScrollingFrame",main)
scroll.Size=UDim2.new(1,-16,1,-94); scroll.Position=UDim2.new(0,8,0,90)
scroll.BackgroundTransparency=1; scroll.BorderSizePixel=0; scroll.ScrollBarThickness=3
scroll.ScrollBarImageColor3=Theme.Accent; scroll.ScrollBarImageTransparency=0.4
scroll.AutomaticCanvasSize=Enum.AutomaticSize.Y; scroll.CanvasSize=UDim2.new(0,0,0,0); scroll.ZIndex=2
local listLayout=Instance.new("UIListLayout",scroll); listLayout.SortOrder=Enum.SortOrder.LayoutOrder; listLayout.Padding=UDim.new(0,5)
local pad=Instance.new("UIPadding",scroll); pad.PaddingLeft=UDim.new(0,0); pad.PaddingRight=UDim.new(0,0); pad.PaddingTop=UDim.new(0,2); pad.PaddingBottom=UDim.new(0,8)

local tabDefs = {{"Speed",1},{"Features",2},{"Settings",3}}
local tabBtns = {}; local tabPages = {}; local activeTab = "Speed"

for _,td in ipairs(tabDefs) do
    local btn=Instance.new("TextButton",tabFrame)
    btn.Size=UDim2.new(0,80,0,26); btn.BackgroundColor3=(td[1]==activeTab) and Theme.AccentDark or Theme.Element
    btn.BorderSizePixel=0; btn.Text=td[1]:upper(); btn.TextColor3=(td[1]==activeTab) and Theme.AccentGlow or Theme.SubText
    btn.Font=Theme.FontBold; btn.TextSize=9; btn.LayoutOrder=td[2]; btn.ZIndex=4; btn.AutoButtonColor=false
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,8)
    local ts=Instance.new("UIStroke",btn); ts.Color=(td[1]==activeTab) and Theme.Accent or Theme.Border; ts.Thickness=(td[1]==activeTab) and 1.5 or 1
    tabBtns[td[1]]={btn=btn,stroke=ts}
    local page=Instance.new("Frame",scroll)
    page.Size=UDim2.new(1,0,0,0); page.BackgroundTransparency=1; page.BorderSizePixel=0
    page.Visible=(td[1]==activeTab); page.ZIndex=2
    page.AutomaticSize=Enum.AutomaticSize.Y
    local pageLL=Instance.new("UIListLayout",page); pageLL.SortOrder=Enum.SortOrder.LayoutOrder; pageLL.Padding=UDim.new(0,5)
    local pagePad=Instance.new("UIPadding",page); pagePad.PaddingBottom=UDim.new(0,4)
    tabPages[td[1]]=page
    local pageLO=0
    btn.MouseButton1Click:Connect(function()
        if activeTab==td[1] then return end
        activeTab=td[1]
        for name,data in pairs(tabBtns) do
            local isA=(name==activeTab)
            TweenService:Create(data.btn,TweenInfo.new(0.15),{BackgroundColor3=isA and Theme.AccentDark or Theme.Element}):Play()
            TweenService:Create(data.btn,TweenInfo.new(0.15),{TextColor3=isA and Theme.AccentGlow or Theme.SubText}):Play()
            TweenService:Create(data.stroke,TweenInfo.new(0.15),{Color=isA and Theme.Accent or Theme.Border,Thickness=isA and 1.5 or 1}):Play()
            tabPages[name].Visible=isA
        end
    end)
    btn.MouseEnter:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end
    end)
    btn.MouseLeave:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.Element}):Play() end
    end)
end

-- ========== UI HELPERS (per-page) ==========
local pageLOs = {Speed=0, Features=0, Settings=0}
local function LO(tab) pageLOs[tab]=(pageLOs[tab] or 0)+1; return pageLOs[tab] end
local function pg(tab) return tabPages[tab] end

local function makeGap(tab,px) local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,0,0,px or 3); f.BackgroundTransparency=1; f.LayoutOrder=LO(tab) end
local function makeDivider(tab)
    local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,-8,0,1); f.BackgroundColor3=Theme.Border; f.BackgroundTransparency=0; f.BorderSizePixel=0; f.LayoutOrder=LO(tab)
    local g=Instance.new("UIGradient",f); g.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,20)),ColorSequenceKeypoint.new(0.5,Theme.Accent),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,20))})
end
local function makeSectionLabel(tab,text)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,26); row.BackgroundTransparency=1; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-8,1,0); lbl.BackgroundTransparency=1
    lbl.Text="  â "..text:upper(); lbl.TextColor3=Theme.Accent; lbl.Font=Theme.FontBold; lbl.TextSize=10; lbl.TextXAlignment=Enum.TextXAlignment.Left
end

local function makeInputRow(tab,label,default,onChange)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0,100,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.ZIndex=2
    local box=Instance.new("TextBox",row); box.Size=UDim2.new(0,60,0,26); box.Position=UDim2.new(1,-70,0.5,-13); box.BackgroundColor3=Theme.BackgroundSecondary; box.BorderSizePixel=0; box.Text=tostring(default); box.TextColor3=Theme.AccentGlow; box.Font=Theme.FontBold; box.TextSize=13; box.ClearTextOnFocus=false; box.ZIndex=3
    Instance.new("UICorner",box).CornerRadius=UDim.new(0,8)
    local boxStroke=Instance.new("UIStroke",box); boxStroke.Color=Theme.AccentDim; boxStroke.Thickness=1
    box.Focused:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.Element}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
    box.FocusLost:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.BackgroundSecondary}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); local num=tonumber(box.Text); if num~=nil then local fv=math.floor(math.clamp(num,0,500)); box.Text=tostring(fv); if onChange then onChange(tostring(fv)) end; autoSaveConfig() else box.Text=tostring(default) end end)
    return box
end

local function makeStatusRow(tab,label,valTxt)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,32); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0.5,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local val=Instance.new("TextLabel",row); val.Size=UDim2.new(0.45,-10,1,0); val.Position=UDim2.new(0.52,0,0,0); val.BackgroundTransparency=1; val.Text=valTxt; val.TextColor3=Theme.AccentGlow; val.Font=Theme.FontBold; val.TextSize=12; val.TextXAlignment=Enum.TextXAlignment.Right
    return val
end

local function makeKeybindRow(tab,label,currentKey,onChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local btn=Instance.new("TextButton",row); btn.Size=UDim2.new(0,70,0,24); btn.Position=UDim2.new(1,-80,0.5,-12); btn.BackgroundColor3=Theme.Keybind; btn.BorderSizePixel=0; btn.Text="["..currentKey.Name.."]"; btn.TextColor3=Theme.Accent; btn.Font=Theme.FontBold; btn.TextSize=10
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,7)
    local bs=Instance.new("UIStroke",btn); bs.Color=Theme.AccentDim; bs.Thickness=1
    local listening=false; local listenConn
    local function stopListen(key)
        listening=false; if listenConn then listenConn:Disconnect(); listenConn=nil end
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(btn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); btn.TextColor3=Theme.Accent
        if key then btn.Text="["..key.Name.."]"; if onChanged then onChanged(key) end; autoSaveConfig() end
    end
    btn.MouseButton1Click:Connect(function()
        if listening then stopListen(nil); return end
        listening=true; btn.Text="[Â·Â·Â·]"; btn.TextColor3=Theme.AccentGlow; btn.BackgroundColor3=Theme.Element
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
        listenConn=UIS.InputBegan:Connect(function(inp,gp)
            if not listening then return end
            if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode)
            elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode) end
        end)
    end)
    return btn
end

local function makeToggleRow(tab,label,defaultKey,defaultOn,onToggle,onKeyChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,40); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-120,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local keyBtn=nil
    if defaultKey then
        keyBtn=Instance.new("TextButton",row); keyBtn.Size=UDim2.new(0,50,0,22); keyBtn.Position=UDim2.new(1,-108,0.5,-11); keyBtn.BackgroundColor3=Theme.Keybind; keyBtn.BorderSizePixel=0; keyBtn.Text=defaultKey.Name; keyBtn.TextColor3=Theme.Accent; keyBtn.Font=Theme.FontBold; keyBtn.TextSize=9; keyBtn.ZIndex=5
        Instance.new("UICorner",keyBtn).CornerRadius=UDim.new(0,5)
        local ks=Instance.new("UIStroke",keyBtn); ks.Color=Theme.AccentDim; ks.Thickness=1
        local kListening=false; local kConn
        local function kStop(key) kListening=false; if kConn then kConn:Disconnect(); kConn=nil end; TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(keyBtn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); keyBtn.TextColor3=Theme.Accent; if key then keyBtn.Text=key.Name; if onKeyChanged then onKeyChanged(key) end; autoSaveConfig() end end
        keyBtn.MouseButton1Click:Connect(function()
            if kListening then kStop(nil); return end
            kListening=true; keyBtn.Text="Â·Â·Â·"; keyBtn.TextColor3=Theme.AccentGlow; keyBtn.BackgroundColor3=Theme.Element
            TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
            kConn=UIS.InputBegan:Connect(function(inp,gp)
                if not kListening then return end
                if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode)
                elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode) end
            end)
        end)
    end
    local toggleBg=Instance.new("TextButton",row); toggleBg.Size=UDim2.new(0,42,0,22); toggleBg.Position=UDim2.new(1,-48,0.5,-11)
    toggleBg.BackgroundColor3=defaultOn and Theme.AccentDark or Theme.ToggleOff; toggleBg.BorderSizePixel=0; toggleBg.Text=""; toggleBg.ZIndex=5; toggleBg.AutoButtonColor=false
    Instance.new("UICorner",toggleBg).CornerRadius=UDim.new(1,0)
    local tStroke=Instance.new("UIStroke",toggleBg); tStroke.Color=defaultOn and Theme.Accent or Theme.Border; tStroke.Thickness=1.5
    local knob=Instance.new("Frame",toggleBg); knob.Size=UDim2.new(0,17,0,17); knob.Position=defaultOn and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5)
    knob.BackgroundColor3=defaultOn and Theme.AccentGlow or Theme.SubText; knob.BorderSizePixel=0; knob.ZIndex=6
    Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
    local isOn=defaultOn or false
    local function setV(on)
        isOn=on
        TweenService:Create(toggleBg,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{BackgroundColor3=on and Theme.AccentDark or Theme.ToggleOff}):Play()
        TweenService:Create(tStroke,TweenInfo.new(0.22),{Color=on and Theme.Accent or Theme.Border}):Play()
        TweenService:Create(knob,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{Position=on and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=on and Theme.AccentGlow or Theme.SubText}):Play()
        TweenService:Create(rowStroke,TweenInfo.new(0.22),{Color=on and Theme.AccentDim or Theme.Border,Thickness=on and 1.5 or 1}):Play()
    end
    toggleBg.MouseButton1Click:Connect(function() isOn=not isOn; setV(isOn); if onToggle then pcall(onToggle,isOn) end; autoSaveConfig() end)
    toggleBg.MouseEnter:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
    toggleBg.MouseLeave:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)
    return setV, keyBtn
end

-- ===================== BUILD TABS =====================

-- ===== SPEED TAB =====
makeSectionLabel("Speed","Speed")
normalBox=makeInputRow("Speed","Normal",State.normalSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.normalSpeed=n end end)
carryBox=makeInputRow("Speed","Carry",State.carrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.carrySpeed=n end end)
setSpeedToggleUI,speedKeyBtn=makeToggleRow("Speed","Toggle",Keys.speed,false,function(on) toggleSpeedType() end,function(k) Keys.speed=k end)
modeValLbl=makeStatusRow("Speed","Mode","Normal")
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Lagger Speed")
laggerBox=makeInputRow("Speed","Normal",State.laggerSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerSpeed=n end end)
carryLaggerBox=makeInputRow("Speed","Carry",State.laggerCarrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerCarrySpeed=n end end)
setLaggerToggleUI,laggerKeyBtn=makeToggleRow("Speed","Toggle",Keys.lagger,false,function(on) toggleLagger() end,function(k) Keys.lagger=k end)
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Combat")
setAutoBat,autoBatKeyBtn=makeToggleRow("Speed","Bat Aimbot",Keys.autoBat,false,function(on) State.autoBatToggled=on; if not on then resetBend() end end,function(k) Keys.autoBat=k end)
makeGap("Speed",4)

-- ===== FEATURES TAB =====
makeSectionLabel("Features","Auto Steal")
local radiusBox=makeInputRow("Features","Grab Rad",AutoSteal.Radius,function(v) local n=tonumber(v); if n and n>=5 and n<=300 then AutoSteal.Radius=math.floor(n); if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end end)
setInstaGrab=makeToggleRow("Features","Auto Grab",nil,false,function(on) AutoSteal.Enabled=on; if on then startAutoSteal() else stopAutoSteal() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Active Toggles")
setInfJump=makeToggleRow("Features","Infinite Jump",nil,false,function(on) State.infJumpEnabled=on end)
setAntiRag=makeToggleRow("Features","Anti Ragdoll",nil,false,function(on) State.antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
setFps=makeToggleRow("Features","FPS Boost",nil,false,function(on) State.fpsBoostEnabled=on; if on then pcall(applyFPSBoost) end end)
setMedusaCounter=makeToggleRow("Features","Medusa Counter",nil,false,function(on) State.medusaCounterEnabled=on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
setAnimToggle=makeToggleRow("Features","Tryhard Anim",nil,false,function(on) State.animEnabled=on; if on then startAnimToggle() else stopAnimToggle() end end)
setUnwalkToggle=makeToggleRow("Features","Unwalk",nil,false,function(on) if on then startUnwalk() else stopUnwalk() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Movement")
tpDownKeyBtn=makeKeybindRow("Features","TP Down",Keys.tpDown,function(k) Keys.tpDown=k end)
dropBrainrotKeyBtn=makeKeybindRow("Features","Drop Brainrot",Keys.dropBrainrot,function(k) Keys.dropBrainrot=k end)
setAutoLeft,autoLeftKeyBtn=makeToggleRow("Features","Auto Left",Keys.autoLeft,false,function(on) State.autoLeftEnabled=on; if on then startAutoLeft() else stopAutoLeft() end end,function(k) Keys.autoLeft=k end)
setAutoRight,autoRightKeyBtn=makeToggleRow("Features","Auto Right",Keys.autoRight,false,function(on) State.autoRightEnabled=on; if on then startAutoRight() else stopAutoRight() end end,function(k) Keys.autoRight=k end)
makeGap("Features",4)

-- ===== SETTINGS TAB =====
makeSectionLabel("Settings","Interface")
guiHideKeyBtn=makeKeybindRow("Settings","Hide GUI",Keys.guiHide,function(k) Keys.guiHide=k end)
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

makeSectionLabel("Settings","Mobile Panel")
local showRow=Instance.new("Frame",pg("Settings")); showRow.Size=UDim2.new(1,0,0,38); showRow.BackgroundColor3=Theme.Element; showRow.BorderSizePixel=0; showRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",showRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",showRow).Color=Theme.Border
local showLbl=Instance.new("TextLabel",showRow); showLbl.Size=UDim2.new(1,-115,1,0); showLbl.Position=UDim2.new(0,12,0,0); showLbl.BackgroundTransparency=1; showLbl.Text="Show Buttons"; showLbl.TextColor3=Theme.Text; showLbl.Font=Theme.FontMedium; showLbl.TextSize=12; showLbl.TextXAlignment=Enum.TextXAlignment.Left
local showToggleBg=Instance.new("TextButton",showRow); showToggleBg.Size=UDim2.new(0,42,0,22); showToggleBg.Position=UDim2.new(1,-48,0.5,-11); showToggleBg.BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff; showToggleBg.BorderSizePixel=0; showToggleBg.Text=""; showToggleBg.ZIndex=5; showToggleBg.AutoButtonColor=false
Instance.new("UICorner",showToggleBg).CornerRadius=UDim.new(1,0)
local showKnob=Instance.new("Frame",showToggleBg); showKnob.Size=UDim2.new(0,17,0,17); showKnob.Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); showKnob.BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText; showKnob.BorderSizePixel=0; showKnob.ZIndex=6; Instance.new("UICorner",showKnob).CornerRadius=UDim.new(1,0)
showToggleBg.MouseButton1Click:Connect(function() MobileButtons.Visible=not MobileButtons.Visible; if mbPanel then mbPanel.Visible=MobileButtons.Visible end; TweenService:Create(showToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(showKnob,TweenInfo.new(0.22),{Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
showToggleBg.MouseEnter:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
showToggleBg.MouseLeave:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local lockRow=Instance.new("Frame",pg("Settings")); lockRow.Size=UDim2.new(1,0,0,38); lockRow.BackgroundColor3=Theme.Element; lockRow.BorderSizePixel=0; lockRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",lockRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",lockRow).Color=Theme.Border
local lockLbl=Instance.new("TextLabel",lockRow); lockLbl.Size=UDim2.new(1,-115,1,0); lockLbl.Position=UDim2.new(0,12,0,0); lockLbl.BackgroundTransparency=1; lockLbl.Text="Lock Buttons"; lockLbl.TextColor3=Theme.Text; lockLbl.Font=Theme.FontMedium; lockLbl.TextSize=12; lockLbl.TextXAlignment=Enum.TextXAlignment.Left
local lockToggleBg=Instance.new("TextButton",lockRow); lockToggleBg.Size=UDim2.new(0,42,0,22); lockToggleBg.Position=UDim2.new(1,-48,0.5,-11); lockToggleBg.BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff; lockToggleBg.BorderSizePixel=0; lockToggleBg.Text=""; lockToggleBg.ZIndex=5; lockToggleBg.AutoButtonColor=false
Instance.new("UICorner",lockToggleBg).CornerRadius=UDim.new(1,0)
local lockKnob=Instance.new("Frame",lockToggleBg); lockKnob.Size=UDim2.new(0,17,0,17); lockKnob.Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); lockKnob.BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText; lockKnob.BorderSizePixel=0; lockKnob.ZIndex=6; Instance.new("UICorner",lockKnob).CornerRadius=UDim.new(1,0)
lockToggleBg.MouseButton1Click:Connect(function() MobileButtons.Locked=not MobileButtons.Locked; TweenService:Create(lockToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(lockKnob,TweenInfo.new(0.22),{Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
lockToggleBg.MouseEnter:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
lockToggleBg.MouseLeave:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local sizeRow=Instance.new("Frame",pg("Settings")); sizeRow.Size=UDim2.new(1,0,0,38); sizeRow.BackgroundColor3=Theme.Element; sizeRow.BorderSizePixel=0; sizeRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",sizeRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",sizeRow).Color=Theme.Border
local sizeLbl=Instance.new("TextLabel",sizeRow); sizeLbl.Size=UDim2.new(0,100,1,0); sizeLbl.Position=UDim2.new(0,12,0,0); sizeLbl.BackgroundTransparency=1; sizeLbl.Text="Btn Size"; sizeLbl.TextColor3=Theme.Text; sizeLbl.Font=Theme.FontMedium; sizeLbl.TextSize=12; sizeLbl.TextXAlignment=Enum.TextXAlignment.Left
local sizeValLbl=Instance.new("TextLabel",sizeRow); sizeValLbl.Size=UDim2.new(0,40,0,16); sizeValLbl.Position=UDim2.new(0,112,0.5,-8); sizeValLbl.BackgroundTransparency=1; sizeValLbl.Text=MB_BTN_W.."px"; sizeValLbl.TextColor3=Theme.AccentGlow; sizeValLbl.Font=Theme.FontBold; sizeValLbl.TextSize=9; sizeValLbl.TextXAlignment=Enum.TextXAlignment.Center
local decBtn=Instance.new("TextButton",sizeRow); decBtn.Size=UDim2.new(0,24,0,24); decBtn.Position=UDim2.new(1,-78,0.5,-12); decBtn.BackgroundColor3=Theme.BackgroundSecondary; decBtn.BorderSizePixel=0; decBtn.Text="-"; decBtn.TextColor3=Theme.Text; decBtn.Font=Theme.FontBlack; decBtn.TextSize=14; decBtn.ZIndex=5
Instance.new("UICorner",decBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",decBtn).Color=Theme.Border
local incBtn=Instance.new("TextButton",sizeRow); incBtn.Size=UDim2.new(0,24,0,24); incBtn.Position=UDim2.new(1,-48,0.5,-12); incBtn.BackgroundColor3=Theme.BackgroundSecondary; incBtn.BorderSizePixel=0; incBtn.Text="+"; incBtn.TextColor3=Theme.Text; incBtn.Font=Theme.FontBlack; incBtn.TextSize=14; incBtn.ZIndex=5
Instance.new("UICorner",incBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",incBtn).Color=Theme.Border
decBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W-4,MB_BTN_H-3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
incBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W+4,MB_BTN_H+3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
for _,b in pairs({decBtn,incBtn}) do
    b.MouseEnter:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
    b.MouseLeave:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
end
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

local saveConfigRow=Instance.new("Frame",pg("Settings")); saveConfigRow.Size=UDim2.new(1,0,0,40); saveConfigRow.BackgroundColor3=Theme.Element; saveConfigRow.BorderSizePixel=0; saveConfigRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",saveConfigRow).CornerRadius=UDim.new(0,10)
local saveConfigBtn=Instance.new("TextButton",saveConfigRow); saveConfigBtn.Size=UDim2.new(1,-16,0,28); saveConfigBtn.Position=UDim2.new(0,8,0.5,-14)
saveConfigBtn.BackgroundColor3=Color3.fromRGB(0,40,80); saveConfigBtn.BorderSizePixel=0; saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; saveConfigBtn.Font=Theme.FontBold; saveConfigBtn.TextSize=12; saveConfigBtn.ZIndex=5; saveConfigBtn.AutoButtonColor=false
Instance.new("UICorner",saveConfigBtn).CornerRadius=UDim.new(0,9)
local saveBtnStroke=Instance.new("UIStroke",saveConfigBtn); saveBtnStroke.Color=Theme.Accent; saveBtnStroke.Thickness=1.5
saveConfigBtn.MouseEnter:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,60,120)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.AccentGlow}):Play() end)
saveConfigBtn.MouseLeave:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,40,80)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
saveConfigBtn.MouseButton1Click:Connect(function() forceSaveConfig(); saveConfigBtn.Text="â  SAVED!"; saveConfigBtn.TextColor3=Theme.Success; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Success}):Play(); task.wait(1.2); saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)

local resetMbRow=Instance.new("Frame",pg("Settings")); resetMbRow.Size=UDim2.new(1,0,0,36); resetMbRow.BackgroundColor3=Theme.Element; resetMbRow.BorderSizePixel=0; resetMbRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",resetMbRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",resetMbRow).Color=Theme.Border
local resetMbBtn=Instance.new("TextButton",resetMbRow); resetMbBtn.Size=UDim2.new(1,-16,0,24); resetMbBtn.Position=UDim2.new(0,8,0.5,-12)
resetMbBtn.BackgroundColor3=Theme.BackgroundSecondary; resetMbBtn.BorderSizePixel=0; resetMbBtn.Text="Reset Button Panel Position"; resetMbBtn.TextColor3=Theme.SubText; resetMbBtn.Font=Theme.FontMedium; resetMbBtn.TextSize=10; resetMbBtn.ZIndex=5; resetMbBtn.AutoButtonColor=false
Instance.new("UICorner",resetMbBtn).CornerRadius=UDim.new(0,7)
resetMbBtn.MouseEnter:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
resetMbBtn.MouseLeave:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
resetMbBtn.MouseButton1Click:Connect(function() if mbPanel then mbPanel.Position=UDim2.new(1,-(MB_TOTAL_W+8),0.5,-(MB_TOTAL_H/2)) end end)

makeGap("Settings",2)
local footerLbl=Instance.new("TextLabel",pg("Settings")); footerLbl.Size=UDim2.new(1,0,0,18); footerLbl.BackgroundTransparency=1; footerLbl.LayoutOrder=LO("Settings"); footerLbl.Text="â RBG â"; footerLbl.TextColor3=Theme.AccentDim; footerLbl.Font=Theme.FontBold; footerLbl.TextSize=9; footerLbl.TextXAlignment=Enum.TextXAlignment.Center

-- ========== TOGGLE GUI VISIBILITY ==========
toggleGuiVis=function()
    State.guiVisible=not State.guiVisible
    if main then main.Visible=State.guiVisible; closeBtn.Visible=State.guiVisible; if miniBtn then miniBtn.Visible=not State.guiVisible end end
end

-- ========== LOGIC ==========
local function findMedusa()
    local char=LP.Character; if not char then return nil end
    for _,tool in ipairs(char:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end
    local bp2=LP:FindFirstChild("Backpack"); if bp2 then for _,tool in ipairs(bp2:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end end
    return nil
end
local function useMedusaCounter()
    if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<25 then return end; local char=LP.Character; if not char then return end
    State.medusaDebounce=true; local med=findMedusa(); if not med then State.medusaDebounce=false; return end
    if med.Parent~=char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
end
local function onAnchorChanged(part) return part:GetPropertyChangedSignal("Anchored"):Connect(function() if part.Anchored and part.Transparency==1 then useMedusaCounter() end end) end
setupMedusaCounter=function(char) stopMedusaCounter(); if not char then return end; for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end; table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end)) end
stopMedusaCounter=function() for _,c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end; Conns.anchor={} end

startAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect() end; State.autoLeftPhase=1
    Conns.autoLeft=RunService.Heartbeat:Connect(function()
        if not State.autoLeftEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoLeftPhase==1 then
            local tgt=Vector3.new(POS.L1.X,root.Position.Y,POS.L1.Z); if (tgt-root.Position).Magnitude<1 then State.autoLeftPhase=2; return end
            local d=(POS.L1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoLeftPhase==2 then
            local tgt=Vector3.new(POS.L2.X,root.Position.Y,POS.L2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoLeftEnabled=false; if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1; if setAutoLeft then setAutoLeft(false) end; return end
            local d=(POS.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect() end; State.autoRightPhase=1
    Conns.autoRight=RunService.Heartbeat:Connect(function()
        if not State.autoRightEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoRightPhase==1 then
            local tgt=Vector3.new(POS.R1.X,root.Position.Y,POS.R1.Z); if (tgt-root.Position).Magnitude<1 then State.autoRightPhase=2; return end
            local d=(POS.R1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoRightPhase==2 then
            local tgt=Vector3.new(POS.R2.X,root.Position.Y,POS.R2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoRightEnabled=false; if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1; if setAutoRight then setAutoRight(false) end; return end
            local d=(POS.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAntiRagdoll=function()
    if Conns.antiRag then return end
    Conns.antiRag=RunService.Heartbeat:Connect(function()
        local char=LP.Character; if not char then return end; local hum2=char:FindFirstChildOfClass("Humanoid"); local root=char:FindFirstChild("HumanoidRootPart")
        if hum2 then local st=hum2:GetState(); if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then hum2:ChangeState(Enum.HumanoidStateType.Running); workspace.CurrentCamera.CameraSubject=hum2; pcall(function() local pm=LP.PlayerScripts:FindFirstChild("PlayerModule"); if pm then require(pm:FindFirstChild("ControlModule")):Enable() end end); if root then root.Velocity=Vector3.new(0,0,0); root.RotVelocity=Vector3.new(0,0,0) end end end
        for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled=true end end
    end)
end
stopAntiRagdoll=function() if Conns.antiRag then Conns.antiRag:Disconnect(); Conns.antiRag=nil end end
applyFPSBoost=function()
    pcall(function() setfpscap(999999999) end)
    local function processObj(v) pcall(function() if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0 elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1 elseif v:IsA("SpecialMesh") then v.TextureId="" elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false elseif v:IsA("SurfaceAppearance") or v:IsA("MaterialVariant") then v:Destroy() elseif v:IsA("Attachment") then v.Visible=false end end) end
    for _,v in pairs(workspace:GetDescendants()) do processObj(v) end
    pcall(function() local lighting=game:GetService("Lighting"); for _,v in pairs(lighting:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end; pcall(function() sethiddenproperty(game:GetService("Lighting"),"Technology",Enum.Technology.Legacy) end); local lighting2=game:GetService("Lighting"); lighting2.GlobalShadows=false; lighting2.FogEnd=9e9; lighting2.Brightness=0; local terrain=workspace:FindFirstChildOfClass("Terrain"); if terrain then pcall(function() sethiddenproperty(terrain,"Decoration",false) end); terrain.WaterReflectance=0; terrain.WaterTransparency=0.7; terrain.WaterWaveSize=0; terrain.WaterWaveSpeed=0 end end)
    workspace.DescendantAdded:Connect(function(v) if State.fpsBoostEnabled then task.spawn(processObj,v) end end)
end

-- ========== CHARACTER SETUP ==========
local function setupChar(char)
    task.wait(0.1); h=char:WaitForChild("Humanoid",5); hrp=char:WaitForChild("HumanoidRootPart",5); State.originalC0=nil
    if not h or not hrp then return end
    local head=char:FindFirstChild("Head")
    if head then
        local oldBB=head:FindFirstChild("SpeedBillboard"); if oldBB then oldBB:Destroy() end
        local bb=Instance.new("BillboardGui",head); bb.Name="SpeedBillboard"; bb.Size=UDim2.new(0,120,0,22); bb.StudsOffset=Vector3.new(0,3,0); bb.AlwaysOnTop=true
        speedLbl=Instance.new("TextLabel",bb); speedLbl.Size=UDim2.new(1,0,1,0); speedLbl.BackgroundTransparency=1; speedLbl.TextColor3=Theme.AccentGlow; speedLbl.Font=Theme.FontBold; speedLbl.TextScaled=true; speedLbl.TextStrokeTransparency=0.5; speedLbl.TextStrokeColor3=Color3.fromRGB(0,20,40)
    end
    if State.antiRagdollEnabled and not Conns.antiRag then task.wait(0.5); startAntiRagdoll() end
    if State.medusaCounterEnabled then setupMedusaCounter(char) end
    if State.animEnabled then task.wait(0.3); saveOriginalAnims(char); applyAnimPack(char) end
    if State.unwalkEnabled then State.unwalkEnabled=false; task.wait(0.3); startUnwalk() end
end
LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

RunService.Stepped:Connect(function()
    for _,p in ipairs(Players:GetPlayers()) do if p~=LP and p.Character then for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end end end
end)

UIS.JumpRequest:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root then root.Velocity=Vector3.new(root.Velocity.X,55,root.Velocity.Z) end
end)
RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root and root.Velocity.Y<-120 then root.Velocity=Vector3.new(root.Velocity.X,-120,root.Velocity.Z) end
end)

RunService.RenderStepped:Connect(function()
    if not (h and hrp) then return end; if State._tpInProgress then return end
    if State.autoLeftEnabled or State.autoRightEnabled then if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end; return end
    local md=h.MoveDirection; local spd=getCurrentSpeed()
    if md.Magnitude>0 then State.lastMoveDir=md; hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
    elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude>0 then local anyHeld=false; for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true; break end end; if anyHeld then hrp.Velocity=Vector3.new(State.lastMoveDir.X*spd,hrp.Velocity.Y,State.lastMoveDir.Z*spd) end end
    if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end
end)

RunService.Heartbeat:Connect(function(dt)
    if not (State.autoBatToggled and h and hrp) then resetBend(); return end
    local target,dist=getClosestPlayer(); if not (target and target.Character) then resetBend(); return end
    local tr=target.Character:FindFirstChild("HumanoidRootPart"); if not tr then resetBend(); return end
    local rj=getRootJoint(); if rj and not State.originalC0 then State.originalC0=rj.C0 end
    local yDiff=tr.Position.Y-hrp.Position.Y; local yVel=math.clamp(yDiff*20,-120,120)
    if dist>6 then
        local dir=(tr.Position-hrp.Position).Unit; hrp.AssemblyLinearVelocity=Vector3.new(dir.X*59,yVel,dir.Z*59)
        if rj then rj.C0=State.originalC0*CFrame.Angles(math.rad(40),0,0) end
    else
        State.crazyAngle=State.crazyAngle+dt*40
        local offsetX=math.cos(State.crazyAngle)*0.8; local offsetZ=math.sin(State.crazyAngle)*0.8
        local targetPos=tr.Position+Vector3.new(offsetX,0,offsetZ); local flatDir=Vector3.new(targetPos.X-hrp.Position.X,0,targetPos.Z-hrp.Position.Z)
        hrp.AssemblyLinearVelocity=Vector3.new(flatDir.Unit.X*200,yVel,flatDir.Unit.Z*200)
        if rj then local t=tick(); local fwdBack=math.sin(t*25)*math.rad(50); local sideways=math.cos(t*20)*math.rad(30); rj.C0=State.originalC0*CFrame.Angles(fwdBack,0,sideways) end
        tryHitBat()
    end
end)

UIS.InputBegan:Connect(function(inp,gp)
    if gp then return end
    if inp.UserInputType~=Enum.UserInputType.Keyboard and inp.UserInputType~=Enum.UserInputType.Gamepad1 then return end
    local kc=inp.KeyCode
    if (State.autoLeftEnabled or State.autoRightEnabled) then if MOVE_KEYS[kc] then return end end
    if kc==Keys.speed then toggleSpeedType()
    elseif kc==Keys.lagger then toggleLagger()
    elseif kc==Keys.autoBat then State.autoBatToggled=not State.autoBatToggled; if not State.autoBatToggled then resetBend() end; setAutoBat(State.autoBatToggled); autoSaveConfig()
    elseif kc==Keys.autoLeft then State.autoLeftEnabled=not State.autoLeftEnabled; if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end; if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    elseif kc==Keys.autoRight then State.autoRightEnabled=not State.autoRightEnabled; if setAutoRight then setAutoRight(State.autoRightEnabled) end; if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    elseif kc==Keys.dropBrainrot then task.spawn(runDropBrainrot)
    elseif kc==Keys.tpDown then tpToGround()
    elseif kc==Keys.guiHide then toggleGuiVis() end
end)

task.spawn(function() while task.wait(0.5) do pcall(function() if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end) end end)

local function loadConfig()
    local hasFile=false; pcall(function() hasFile=isfile("JispiHubConfig.json") end); if not hasFile then return end
    local ok,cfg=pcall(function() return HttpService:JSONDecode(readfile("JispiHubConfig.json")) end); if not ok or not cfg then return end
    if cfg.normalSpeed and type(cfg.normalSpeed)=="number" then State.normalSpeed=cfg.normalSpeed; normalBox.Text=tostring(cfg.normalSpeed) end
    if cfg.carrySpeed  and type(cfg.carrySpeed)=="number"  then State.carrySpeed=cfg.carrySpeed;   carryBox.Text=tostring(cfg.carrySpeed)   end
    if cfg.laggerSpeed and type(cfg.laggerSpeed)=="number" then State.laggerSpeed=cfg.laggerSpeed; laggerBox.Text=tostring(cfg.laggerSpeed) end
    if cfg.laggerCarrySpeed and type(cfg.laggerCarrySpeed)=="number" then State.laggerCarrySpeed=cfg.laggerCarrySpeed; carryLaggerBox.Text=tostring(cfg.laggerCarrySpeed) end
    if cfg.speedType=="normal" or cfg.speedType=="carry" then State.speedType=cfg.speedType end
    if type(cfg.laggerActive)=="boolean" then State.laggerActive=cfg.laggerActive end
    if cfg.autoBatKey  and Enum.KeyCode[cfg.autoBatKey]    then Keys.autoBat=Enum.KeyCode[cfg.autoBatKey];   if autoBatKeyBtn  then autoBatKeyBtn.Text="["..cfg.autoBatKey.."]"   end end
    if cfg.speedKey    and Enum.KeyCode[cfg.speedKey]      then Keys.speed=Enum.KeyCode[cfg.speedKey]        end
    if cfg.laggerKey   and Enum.KeyCode[cfg.laggerKey]     then Keys.lagger=Enum.KeyCode[cfg.laggerKey]      end
    if cfg.autoLeftKey  and Enum.KeyCode[cfg.autoLeftKey]  then Keys.autoLeft=Enum.KeyCode[cfg.autoLeftKey];   if autoLeftKeyBtn  then autoLeftKeyBtn.Text="["..cfg.autoLeftKey.."]"   end end
    if cfg.autoRightKey and Enum.KeyCode[cfg.autoRightKey] then Keys.autoRight=Enum.KeyCode[cfg.autoRightKey]; if autoRightKeyBtn then autoRightKeyBtn.Text="["..cfg.autoRightKey.."]" end end
    if cfg.tpDownKey    and Enum.KeyCode[cfg.tpDownKey]    then Keys.tpDown=Enum.KeyCode[cfg.tpDownKey];       if tpDownKeyBtn    then tpDownKeyBtn.Text="["..cfg.tpDownKey.."]"        end end
    if cfg.grabRadius and type(cfg.grabRadius)=="number" then AutoSteal.Radius=cfg.grabRadius; if progressRadLbl then progressRadLbl.Text="Radius: "..cfg.grabRadius end end
    if cfg.autoStealEnabled then AutoSteal.Enabled=true; setInstaGrab(true); pcall(startAutoSteal) end
    if cfg.infJump      then State.infJumpEnabled=true;      setInfJump(true)      end
    if cfg.antiRagdoll  then State.antiRagdollEnabled=true;  setAntiRag(true);     startAntiRagdoll() end
    if cfg.fpsBoost     then State.fpsBoostEnabled=true;     setFps(true);         applyFPSBoost()    end
    if cfg.medusaCounter then State.medusaCounterEnabled=true; setMedusaCounter(true); setupMedusaCounter(LP.Character) end
    if cfg.dropBrainrotKey and Enum.KeyCode[cfg.dropBrainrotKey] then Keys.dropBrainrot=Enum.KeyCode[cfg.dropBrainrotKey]; if dropBrainrotKeyBtn then dropBrainrotKeyBtn.Text="["..cfg.dropBrainrotKey.."]"                    bat:Activate() -- Ø¨ÙØ®
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local ContentProvider = game:GetService("ContentProvider")
local LP = Players.LocalPlayer

local State = {
    normalSpeed = 60, carrySpeed = 30, laggerSpeed = 13, laggerCarrySpeed = 13,
    speedType = "normal", laggerActive = false, autoBatToggled = false,
    hittingCooldown = false, infJumpEnabled = false, antiRagdollEnabled = false,
    fpsBoostEnabled = false, guiVisible = true, isStealing = false,
    stealStartTime = nil, lastStealTick = 0, medusaLastUsed = 0,
    medusaDebounce = false, medusaCounterEnabled = false, dropBrainrotActive = false,
    autoLeftEnabled = false, autoRightEnabled = false, autoLeftPhase = 1,
    autoRightPhase = 1, _tpInProgress = false, lastMoveDir = Vector3.new(0,0,0),
    animEnabled = false, unwalkEnabled = false, originalC0 = nil, crazyAngle = 0,
}

local Keys = {
    autoBat = Enum.KeyCode.E, speed = Enum.KeyCode.Q, lagger = Enum.KeyCode.C,
    guiHide = Enum.KeyCode.LeftControl, autoLeft = Enum.KeyCode.L,
    autoRight = Enum.KeyCode.R, dropBrainrot = Enum.KeyCode.H, tpDown = Enum.KeyCode.T,
}

local main, closeBtn, miniBtn
local toggleGuiVis

local MobileButtons = { Visible = true, Locked = false, BtnsObjects = {}, BtnRefs = {}, Buttons = {}, HideGuiBtn = nil }

local MB_BTN_W = 58
local MB_BTN_H = 48
local MB_COLS = 2
local MB_ROWS = 4
local MB_PAD = 3
local MB_CORNER = 14
local MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
local MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
local mbPanel = nil

local Theme = {
    Background      = Color3.fromRGB(8, 10, 20),
    BackgroundSecondary = Color3.fromRGB(12, 15, 28),
    Header          = Color3.fromRGB(5, 8, 18),
    Element         = Color3.fromRGB(14, 18, 35),
    ElementHover    = Color3.fromRGB(20, 26, 50),
    Text            = Color3.fromRGB(200, 220, 255),
    SubText         = Color3.fromRGB(80, 110, 160),
    Accent          = Color3.fromRGB(0, 180, 255),
    AccentDark      = Color3.fromRGB(0, 100, 200),
    AccentGlow      = Color3.fromRGB(0, 220, 255),
    AccentDim       = Color3.fromRGB(0, 60, 120),
    Border          = Color3.fromRGB(0, 60, 120),
    BorderLight     = Color3.fromRGB(0, 100, 180),
    Success         = Color3.fromRGB(0, 255, 160),
    Danger          = Color3.fromRGB(255, 50, 80),
    Keybind         = Color3.fromRGB(10, 20, 45),
    KeybindText     = Color3.fromRGB(0, 180, 255),
    ToggleOff       = Color3.fromRGB(18, 22, 45),
    Font            = Enum.Font.Gotham,
    FontBold        = Enum.Font.GothamBold,
    FontBlack       = Enum.Font.GothamBlack,
    FontMedium      = Enum.Font.GothamMedium,
}

local AutoSteal = {
    Enabled = false, Radius = 20, Duration = 1.3, IsStealing = false,
    Data = {}, ProgressFill = nil, ProgressText = nil,
}

local function isMyPlotByName(plotName)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return false end
    local plot = plots:FindFirstChild(plotName); if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then local yb = sign:FindFirstChild("YourBase"); if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end end
    return false
end

local function findNearestPrompt()
    local char = LP.Character; local root = char and char:FindFirstChild("HumanoidRootPart"); if not root then return nil end
    local plots = workspace:FindFirstChild("Plots"); if not plots then return nil end
    local bestPrompt, bestDist, bestName = nil, math.huge, nil
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local podiums = plot:FindFirstChild("AnimalPodiums"); if not podiums then continue end
        for _, pod in ipairs(podiums:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base"); local spawn = base and base:FindFirstChild("Spawn")
                if spawn then
                    local dist = (spawn.Position - root.Position).Magnitude
                    if dist < bestDist and dist <= AutoSteal.Radius then
                        local att = spawn:FindFirstChild("PromptAttachment")
                        if att then for _, child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") then bestPrompt, bestDist, bestName = child, dist, pod.Name; break end
                        end end
                    end
                end
            end)
        end
    end
    return bestPrompt, bestDist, bestName
end

local function executeSteal(prompt, animalName)
    if AutoSteal.IsStealing then return end
    if not AutoSteal.Data[prompt] then
        AutoSteal.Data[prompt] = {hold = {}, trigger = {}, ready = true}
        pcall(function()
            if getconnections then
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(AutoSteal.Data[prompt].hold, c.Function) end end
                for _, c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(AutoSteal.Data[prompt].trigger, c.Function) end end
            end
        end)
    end
    local data = AutoSteal.Data[prompt]; if not data.ready then return end
    data.ready = false; AutoSteal.IsStealing = true; local startTime = tick()
    local conn; conn = RunService.Heartbeat:Connect(function()
        if not AutoSteal.IsStealing then conn:Disconnect(); return end
        local prog = math.clamp((tick() - startTime) / AutoSteal.Duration, 0, 1)
        if AutoSteal.ProgressFill then AutoSteal.ProgressFill.Size = UDim2.new(prog, 0, 1, 0) end
    end)
    task.spawn(function()
        for _, f in ipairs(data.hold) do task.spawn(f) end
        task.wait(AutoSteal.Duration)
        for _, f in ipairs(data.trigger) do task.spawn(f) end
        AutoSteal.IsStealing = false; data.ready = true
        task.wait(0.6)
        if not AutoSteal.IsStealing and AutoSteal.ProgressFill then
            TweenService:Create(AutoSteal.ProgressFill, TweenInfo.new(0.4), {Size = UDim2.new(0,0,1,0)}):Play()
        end
    end)
end

local autoStealConnection = nil
local function startAutoSteal()
    if autoStealConnection then return end
    autoStealConnection = RunService.Heartbeat:Connect(function()
        if AutoSteal.Enabled and not AutoSteal.IsStealing then
            local p, _, name = findNearestPrompt(); if p then executeSteal(p, name) end
        end
    end)
end
local function stopAutoSteal()
    if autoStealConnection then autoStealConnection:Disconnect(); autoStealConnection = nil end
    AutoSteal.IsStealing = false
    for k, v in pairs(AutoSteal.Data) do if v.ready ~= nil then v.ready = true end end
end

local MOVE_KEYS = {[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local DROP_ASCEND_DURATION = 0.2; local DROP_ASCEND_SPEED = 150
local POS = {L1=Vector3.new(-476.48,-6.28,92.73),L2=Vector3.new(-483.12,-4.95,94.80),R1=Vector3.new(-476.16,-6.52,25.62),R2=Vector3.new(-483.04,-5.09,23.14)}
local Conns = {autoSteal=nil,antiRag=nil,autoLeft=nil,autoRight=nil,anchor={},progress=nil}
local h, hrp, speedLbl
local setAutoLeft, setAutoRight
local setInstaGrab, setAutoBat, setInfJump, setAntiRag, setFps, setMedusaCounter
local setAnimToggle, setUnwalkToggle
local setupMedusaCounter, stopMedusaCounter, startAntiRagdoll, stopAntiRagdoll, applyFPSBoost
local modeValLbl, normalBox, carryBox, laggerBox, carryLaggerBox
local autoBatKeyBtn, speedKeyBtn, laggerKeyBtn, autoLeftKeyBtn, autoRightKeyBtn, guiHideKeyBtn, dropBrainrotKeyBtn, tpDownKeyBtn
local setSpeedToggleUI, setLaggerToggleUI
local progressRadLbl
local startAutoLeft, stopAutoLeft, startAutoRight, stopAutoRight

local function getRootJoint()
    local char2 = LP.Character; local torso = char2 and char2:FindFirstChild("LowerTorso")
    return torso and torso:FindFirstChild("Root")
end
local function resetBend()
    local rj = getRootJoint()
    if rj and State.originalC0 then rj.C0 = State.originalC0; State.originalC0 = nil end
end
local function getBat()
    local char = LP.Character; if not char then return nil end
    local tool = char:FindFirstChild("Bat"); if tool then return tool end
    local bp2 = LP:FindFirstChild("Backpack")
    if bp2 then tool = bp2:FindFirstChild("Bat"); if tool then tool.Parent = char; return tool end end
    return nil
end
local function tryHitBat()
    if State.hittingCooldown then return end; State.hittingCooldown = true
    pcall(function()
        local bat = getBat(); if bat then bat:Activate(); local ev = bat:FindFirstChildWhichIsA("RemoteEvent"); if ev then ev:FireServer() end end
    end)
    task.delay(0.08, function() State.hittingCooldown = false end)
end
local function getClosestPlayer()
    if not hrp then return nil, math.huge end
    local cp, cd = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if tr then local d = (hrp.Position - tr.Position).Magnitude; if d < cd then cd = d; cp = p end end
        end
    end
    return cp, cd
end

local saveDebounce = false
local function autoSaveConfig()
    if saveDebounce then return end; saveDebounce = true
    task.delay(0.5, function()
        local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
        pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end); saveDebounce = false
    end)
end
local function forceSaveConfig()
    local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
    pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end)
end
localsteal(steal) local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local Conns = {autoSteal=nil,antiRag=nil,anchor={},progress=nil,aimbot=nil,batCounter=nil}
local PLOT_CACHE_DURATION = 2
local PROMPT_CACHE_REFRESH = 0.15
local STEAL_COOLDOWN = 0.1
local MEDUSA_COOLDOWN = 25
local BAT_COUNTER_COOLDOWN = 0
local batCounterDebounce = false
local batCounterLastUsed = 0
local progressRadLbl,progressFill,progressPct
local modeValLbl
local _aimTarget = nil
local _aimLastScan = 0

local VYSE_AIMBOT_SPEED = 56.5
local VYSE_HIT_DIST = 5
local SWING_COOLDOWN = 0.08
local hittingCooldown = false

local desyncEnabled = false
local setDesyncVisual = nil
local desyncActive = false
local desyncSpeedConn = nil
local G_desyncAnimate = nil

local fpsBoostEnabled = false

local function refreshUIToggles()
    if setSpeedToggleUI then setSpeedToggleUI(State.speedType == "carry") end
    if setLaggerToggleUI then setLaggerToggleUI(State.laggerActive) end
    if modeValLbl then
        if State.laggerActive then modeValLbl.Text = (State.speedType=="normal") and "â¡ Lagger Normal" or "â¡ Lagger Carry"
        else modeValLbl.Text = (State.speedType=="normal") and "Normal" or "Carry" end
    end
end
local function toggleSpeedType()
    State.speedType = (State.speedType=="normal") and "carry" or "normal"; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.carrySpeed then MobileButtons.BtnRefs.carrySpeed:SetAttribute("MB_On", State.speedType=="carry") end
end
local function toggleLagger()
    State.laggerActive = not State.laggerActive; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.lagger then MobileButtons.BtnRefs.lagger:SetAttribute("MB_On", State.laggerActive) end
end
local function getCurrentSpeed()
    if State.laggerActive then return State.speedType=="normal" and State.laggerSpeed or State.laggerCarrySpeed
    else return State.speedType=="normal" and State.normalSpeed or State.carrySpeed end
end
local function getAutoMoveSpeed()
    return State.laggerActive and State.laggerSpeed or State.normalSpeed
end
local function tpToGround()
    local char = LP.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
    local rp = RaycastParams.new(); rp.FilterType = Enum.RaycastFilterType.Exclude; rp.FilterDescendantsInstances = {char}
    local rr = workspace:Raycast(root.Position, Vector3.new(0,-500,0), rp)
    if rr then root.CFrame = CFrame.new(rr.Position + Vector3.new(0,3,0)) else root.CFrame = root.CFrame * CFrame.new(0,-20,0) end
end
local function runDropBrainrot()
    if State.dropBrainrotActive then return end
    local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if not root then return end
    State.dropBrainrotActive=true; local t0=tick(); local dc
    dc=RunService.Heartbeat:Connect(function()
        local r=char and char:FindFirstChild("HumanoidRootPart"); if not r then dc:Disconnect(); State.dropBrainrotActive=false; return end
        if tick()-t0>=DROP_ASCEND_DURATION then
            dc:Disconnect()
            local rp=RaycastParams.new(); rp.FilterDescendantsInstances={char}; rp.FilterType=Enum.RaycastFilterType.Exclude
            local rr=workspace:Raycast(r.Position,Vector3.new(0,-2000,0),rp)
            if rr then local hum2=char:FindFirstChildOfClass("Humanoid"); local off=(hum2 and hum2.HipHeight or 2)+(r.Size.Y/2); r.CFrame=CFrame.new(r.Position.X,rr.Position.Y+off,r.Position.Z); r.AssemblyLinearVelocity=Vector3.new(0,0,0) end
            State.dropBrainrotActive=false; return
        end
        r.AssemblyLinearVelocity=Vector3.new(r.AssemblyLinearVelocity.X,DROP_ASCEND_SPEED,r.AssemblyLinearVelocity.Z)
    end)
end

-- ========== MOBILE PANEL ==========
local function createMobilePanel()
    local panel = Instance.new("ScreenGui")
    panel.Name="OmniHubButtons"; panel.Parent=LP:WaitForChild("PlayerGui"); panel.ResetOnSpawn=false; panel.ZIndexBehavior=Enum.ZIndexBehavior.Sibling

    mbPanel = Instance.new("Frame", panel)
    mbPanel.Name = "MobilePanel"
    mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H)
    mbPanel.Position = UDim2.new(1, -(MB_TOTAL_W+8), 0.5, -(MB_TOTAL_H/2))
    mbPanel.BackgroundColor3 = Theme.Background
    mbPanel.BorderSizePixel = 0; mbPanel.Active = true; mbPanel.ZIndex = 50
    Instance.new("UICorner", mbPanel).CornerRadius = UDim.new(0, MB_CORNER+4)
    local mbStroke = Instance.new("UIStroke", mbPanel)
    mbStroke.Color = Theme.Accent; mbStroke.Thickness = 1.5; mbStroke.Transparency = 0.45

    local dragStart, startPos, moved = nil, nil, false
    local DRAG_THRESHOLD = 8
    mbPanel.InputBegan:Connect(function(inp)
        if MobileButtons.Locked then return end
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
            dragStart=inp.Position; startPos=mbPanel.Position; moved=false
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if MobileButtons.Locked or not dragStart then return end
        if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then
            local d=inp.Position-dragStart
            if not moved and (math.abs(d.X)+math.abs(d.Y))>DRAG_THRESHOLD then moved=true end
            if moved then mbPanel.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+d.X,startPos.Y.Scale,startPos.Y.Offset+d.Y) end
        end
    end)
    UIS.InputEnded:Connect(function(inp)
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragStart=nil; moved=false end
    end)

    local function makeMobileBtn(label, row, col, onClick)
        local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
        local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
        local btn = Instance.new("TextButton", mbPanel)
        btn.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
        btn.Position = UDim2.new(0, x, 0, y)
        btn.BackgroundColor3 = Theme.Element; btn.BorderSizePixel = 0
        btn.Text = ""; btn.AutoButtonColor = false; btn.ZIndex = 52
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, MB_CORNER)
        local bs = Instance.new("UIStroke", btn); bs.Color = Theme.Accent; bs.Thickness = 1; bs.Transparency = 0.7
        local lbl = Instance.new("TextLabel", btn)
        lbl.Size = UDim2.new(1,0,1,0); lbl.BackgroundTransparency = 1
        lbl.Text = label; lbl.TextColor3 = Theme.Accent; lbl.Font = Theme.FontBold
        lbl.TextSize = 9; lbl.TextWrapped = true; lbl.TextXAlignment = Enum.TextXAlignment.Center; lbl.TextYAlignment = Enum.TextYAlignment.Center; lbl.ZIndex = 56

        btn.MouseButton1Down:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.05), {BackgroundColor3=Theme.ElementHover}):Play()
            TweenService:Create(bs, TweenInfo.new(0.05), {Transparency=0}):Play()
        end)
        btn.MouseButton1Up:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.10), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.10), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseEnter:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.ElementHover}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.4}):Play()
            end
        end)
        btn.MouseLeave:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseButton1Click:Connect(function() if onClick then pcall(onClick) end end)

        btn:SetAttribute("MB_On", false)
        btn.AttributeChanged:Connect(function(a)
            if a == "MB_On" then
                local on = btn:GetAttribute("MB_On")
                if on then
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Accent}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(5,5,15)}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0, Color = Theme.AccentGlow}):Play()
                    bs.Thickness = 2
                else
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Element}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Theme.Accent}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0.7, Color = Theme.Accent}):Play()
                    bs.Thickness = 1
                end
            end
        end)
        table.insert(MobileButtons.BtnsObjects, btn)
        return btn
    end

    local alMbBtn = makeMobileBtn("AUTO\nLEFT", 1, 1, function()
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoLeftEnabled = not State.autoLeftEnabled
        if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end
        if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoLeft = alMbBtn

    local arMbBtn = makeMobileBtn("AUTO\nRIGHT", 1, 2, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoRightEnabled = not State.autoRightEnabled
        if setAutoRight then setAutoRight(State.autoRightEnabled) end
        if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoRight = arMbBtn

    local batMbBtn = makeMobileBtn("BAT", 2, 1, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        State.autoBatToggled = not State.autoBatToggled
        if not State.autoBatToggled then resetBend() end
        if setAutoBat then setAutoBat(State.autoBatToggled) end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoBat = batMbBtn

    makeMobileBtn("DROP", 2, 2, function() runDropBrainrot() end)

    local carryMbBtn = makeMobileBtn("CARRY\nSPD", 3, 1, function() toggleSpeedType() end)
    MobileButtons.BtnRefs.carrySpeed = carryMbBtn

    local lagMbBtn = makeMobileBtn("LAG", 3, 2, function() toggleLagger() end)
    MobileButtons.BtnRefs.lagger = lagMbBtn

    makeMobileBtn("TP\nDOWN", 4, 1, function() tpToGround() end)

    local guiMbBtn = makeMobileBtn("GUI", 4, 2, function() if toggleGuiVis then toggleGuiVis() end end)
    MobileButtons.HideGuiBtn = guiMbBtn

    task.spawn(function()
        while task.wait(0.2) do
            pcall(function()
                if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
                if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
                if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
                if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
                if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
            end)
        end
    end)

    task.spawn(function()
        task.wait(0.1)
        if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
        if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
        if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
        if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
        if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
    end)

    for _, btn in pairs(MobileButtons.BtnsObjects) do btn.Visible = MobileButtons.Visible end
    mbPanel.Visible = MobileButtons.Visible
end

local function repositionMbButtons()
    if not mbPanel then return end
    local idx = 0
    for _, child in ipairs(mbPanel:GetChildren()) do
        if child:IsA("TextButton") then
            local row = math.floor(idx / MB_COLS) + 1
            local col = (idx % MB_COLS) + 1
            local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
            local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
            child.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
            child.Position = UDim2.new(0, x, 0, y)
            idx = idx + 1
        end
    end
end

local function applyMbBtnSize(w, h2)
    w = math.clamp(w, 40, 120); h2 = math.clamp(h2, 36, 120)
    MB_BTN_W = w; MB_BTN_H = h2
    MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
    MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
    if mbPanel then mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H); repositionMbButtons() end
end

-- ========== ANIMATIONS ==========
local Anims = {idle1="rbxassetid://133806214992291",idle2="rbxassetid://94970088341563",walk="rbxassetid://707897309",run="rbxassetid://707861613",jump="rbxassetid://116936326516985",fall="rbxassetid://116936326516985",climb="rbxassetid://116936326516985",swim="rbxassetid://116936326516985",swimidle="rbxassetid://116936326516985"}
task.spawn(function() pcall(function() ContentProvider:PreloadAsync({Anims.idle1,Anims.idle2,Anims.walk,Anims.run,Anims.jump,Anims.fall,Anims.climb,Anims.swim,Anims.swimidle}) end) end)
local animHeartbeatConn=nil; local savedAnimate=nil; local originalAnims=nil
local function isPackAnim(id) if not id then return false end; for _,v in pairs(Anims) do if v==id then return true end end; return false end
local function saveOriginalAnims(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function g(obj) return obj and obj.AnimationId or nil end; local ids={idle1=g(animate.idle and animate.idle.Animation1),idle2=g(animate.idle and animate.idle.Animation2),walk=g(animate.walk and animate.walk.WalkAnim),run=g(animate.run and animate.run.RunAnim),jump=g(animate.jump and animate.jump.JumpAnim),fall=g(animate.fall and animate.fall.FallAnim),climb=g(animate.climb and animate.climb.ClimbAnim),swim=g(animate.swim and animate.swim.Swim),swimidle=g(animate.swimidle and animate.swimidle.SwimIdle)}; if not isPackAnim(ids.walk) then originalAnims=ids end end
local function applyAnimPack(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,Anims.idle1); s(animate.idle and animate.idle.Animation2,Anims.idle2); s(animate.walk and animate.walk.WalkAnim,Anims.walk); s(animate.run and animate.run.RunAnim,Anims.run); s(animate.jump and animate.jump.JumpAnim,Anims.jump); s(animate.fall and animate.fall.FallAnim,Anims.fall); s(animate.climb and animate.climb.ClimbAnim,Anims.climb); s(animate.swim and animate.swim.Swim,Anims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,Anims.swimidle) end
local function restoreOriginalAnims(char) if not originalAnims then return end; local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj and id then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,originalAnims.idle1); s(animate.idle and animate.idle.Animation2,originalAnims.idle2); s(animate.walk and animate.walk.WalkAnim,originalAnims.walk); s(animate.run and animate.run.RunAnim,originalAnims.run); s(animate.jump and animate.jump.JumpAnim,originalAnims.jump); s(animate.fall and animate.fall.FallAnim,originalAnims.fall); s(animate.climb and animate.climb.ClimbAnim,originalAnims.climb); s(animate.swim and animate.swim.Swim,originalAnims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,originalAnims.swimidle); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end
local function startAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then saveOriginalAnims(char); applyAnimPack(char); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end; animHeartbeatConn=RunService.Heartbeat:Connect(function() if not State.animEnabled then return end; local c=LP.Character; if c then applyAnimPack(c) end end) end
local function stopAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then restoreOriginalAnims(char) end end
local function startUnwalk() if State.unwalkEnabled then return end; State.unwalkEnabled=true; local c=LP.Character; if not c then return end; local hum=c:FindFirstChildOfClass("Humanoid"); if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end; local anim=c:FindFirstChild("Animate"); if anim then savedAnimate=anim:Clone(); anim:Destroy() end end
local function stopUnwalk() if not State.unwalkEnabled then return end; State.unwalkEnabled=false; local c=LP.Character; if c and savedAnimate then savedAnimate.Parent=c; savedAnimate.Disabled=false; savedAnimate=nil end; task.spawn(function() task.wait(0.15); local char=LP.Character; if not char then return end; if State.animEnabled then saveOriginalAnims(char); applyAnimPack(char) else restoreOriginalAnims(char) end end) end

-- ========== MAIN GUI ==========
local gui = Instance.new("ScreenGui")
gui.Name="OmniHubGUI"; gui.ResetOnSpawn=false; gui.DisplayOrder=10; gui.IgnoreGuiInset=true; gui.Parent=LP:WaitForChild("PlayerGui")

local stealProgressBar = Instance.new("Frame", gui)
stealProgressBar.Size=UDim2.new(0,280,0,6); stealProgressBar.Position=UDim2.new(0.5,-140,0.9,0)
stealProgressBar.BackgroundColor3=Color3.fromRGB(8,12,25); stealProgressBar.BackgroundTransparency=0; stealProgressBar.BorderSizePixel=0; stealProgressBar.ZIndex=100
Instance.new("UICorner",stealProgressBar).CornerRadius=UDim.new(1,0)
local barStroke=Instance.new("UIStroke",stealProgressBar); barStroke.Color=Theme.AccentDim; barStroke.Thickness=1
local barFill=Instance.new("Frame",stealProgressBar); barFill.Size=UDim2.new(0,0,1,0); barFill.BackgroundColor3=Theme.Accent; barFill.BorderSizePixel=0
Instance.new("UICorner",barFill).CornerRadius=UDim.new(1,0); AutoSteal.ProgressFill=barFill

local function makeStealBarDraggable(frame)
    local dragging=false; local dragInput,dragStart,startPos,activeDragConn
    local function stopDrag() dragging=false; dragInput=nil; dragStart=nil; startPos=nil; if activeDragConn then activeDragConn:Disconnect(); activeDragConn=nil end end
    frame.InputBegan:Connect(function(input) if MobileButtons.Locked then return end; if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=input.Position; startPos=frame.Position; input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End or input.UserInputState==Enum.UserInputState.Cancelled then stopDrag() end end) end end)
    frame.InputChanged:Connect(function(input) if not dragging then return end; if MobileButtons.Locked then stopDrag(); return end; if input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch then dragInput=input end end)
    UIS.InputChanged:Connect(function(input) if input==dragInput and dragging then if MobileButtons.Locked then stopDrag(); return end; local delta=input.Position-dragStart; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y) end end)
end
makeStealBarDraggable(stealProgressBar)
local function saveStealBarPosition() local pos=stealProgressBar.Position; pcall(function() writefile("JispiStealBarPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function loadStealBarPosition() local savedPos=nil; pcall(function() savedPos=readfile("JispiStealBarPos.txt") end); if savedPos and savedPos~="" then local parts={}; for part in string.gmatch(savedPos,"[^,]+") do table.insert(parts,part) end; if #parts>=4 then stealProgressBar.Position=UDim2.new(tonumber(parts[1]),tonumber(parts[2]),tonumber(parts[3]),tonumber(parts[4])) end end end
loadStealBarPosition()
local lastStealBarSave=0; stealProgressBar:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastStealBarSave>0.5 then lastStealBarSave=tick(); saveStealBarPosition() end end)

-- ========== MAIN FRAME ==========
main = Instance.new("Frame", gui)
main.Name="Main"; main.Size=UDim2.new(0,320,0,470); main.Position=UDim2.new(0,20,0,20)
main.BackgroundColor3=Theme.Background; main.BorderSizePixel=0; main.Active=true; main.ClipsDescendants=true
Instance.new("UICorner",main).CornerRadius=UDim.new(0,16)
local outerGlow=Instance.new("UIStroke",main); outerGlow.Color=Theme.Accent; outerGlow.Thickness=1.5; outerGlow.Transparency=0.3
local glowPulse=Instance.new("Frame",main); glowPulse.Size=UDim2.new(1,8,1,8); glowPulse.Position=UDim2.new(0,-4,0,-4)
glowPulse.BackgroundColor3=Theme.Accent; glowPulse.BackgroundTransparency=0.88; glowPulse.BorderSizePixel=0; glowPulse.ZIndex=-1
Instance.new("UICorner",glowPulse).CornerRadius=UDim.new(0,20)
task.spawn(function()
    while true do
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.78}):Play()
        task.wait(1.8)
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.92}):Play()
        task.wait(1.8)
    end
end)

local function saveMainPosition() local pos=main.Position; pcall(function() writefile("JispiHubGUIPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function saveMiniPosition() if miniBtn then local pos=miniBtn.Position; pcall(function() writefile("JispiMiniPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end end
local function loadMainPosition() local sp=nil; pcall(function() sp=readfile("JispiHubGUIPos.txt") end); if sp and sp~="" then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then main.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])); if closeBtn then closeBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2])+320-34,tonumber(p[3]),tonumber(p[4])+18) end end end end
local function loadMiniPosition() local sp=nil; pcall(function() sp=readfile("JispiMiniPos.txt") end); if sp and sp~="" and miniBtn then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then miniBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])) end end end

-- ========== HEADER ==========
local header=Instance.new("Frame",main)
header.Size=UDim2.new(1,0,0,52); header.BackgroundColor3=Theme.Header; header.BorderSizePixel=0; header.ZIndex=5
Instance.new("UICorner",header).CornerRadius=UDim.new(0,16)
local headerAccentLine=Instance.new("Frame",header)
headerAccentLine.Size=UDim2.new(1,0,0,2); headerAccentLine.Position=UDim2.new(0,0,1,-2)
headerAccentLine.BackgroundColor3=Theme.Accent; headerAccentLine.BorderSizePixel=0; headerAccentLine.ZIndex=6
local lineGlow=Instance.new("UIGradient",headerAccentLine)
lineGlow.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,30)),ColorSequenceKeypoint.new(0.5,Theme.AccentGlow),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,30))})
local titleLbl=Instance.new("TextLabel",header)
titleLbl.Size=UDim2.new(1,0,1,-4); titleLbl.Position=UDim2.new(0,0,0,0)
titleLbl.BackgroundTransparency=1; titleLbl.Text="â  RBG HUB  â"
titleLbl.TextColor3=Theme.AccentGlow; titleLbl.Font=Theme.FontBlack; titleLbl.TextSize=17
titleLbl.TextXAlignment=Enum.TextXAlignment.Center; titleLbl.ZIndex=6
local versionLbl=Instance.new("TextLabel",header)
versionLbl.Size=UDim2.new(0,40,0,16); versionLbl.Position=UDim2.new(1,-50,0.5,-8)
versionLbl.BackgroundTransparency=1; versionLbl.Text="v1.0"; versionLbl.TextColor3=Theme.AccentDim
versionLbl.Font=Theme.Font; versionLbl.TextSize=9; versionLbl.ZIndex=6

-- ========== CLOSE / MINI ==========
closeBtn=Instance.new("TextButton",gui)
closeBtn.Size=UDim2.new(0,28,0,28); closeBtn.BackgroundColor3=Color3.fromRGB(14,18,35)
closeBtn.BorderSizePixel=0; closeBtn.Text="â"; closeBtn.TextColor3=Theme.SubText
closeBtn.Font=Theme.FontBold; closeBtn.TextSize=14; closeBtn.ZIndex=50
Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(0,8)
local closeStroke=Instance.new("UIStroke",closeBtn); closeStroke.Color=Theme.Border; closeStroke.Thickness=1.5
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Danger,TextColor3=Color3.new(1,1,1)}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Danger}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Color3.fromRGB(14,18,35),TextColor3=Theme.SubText}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Border}):Play() end)
closeBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)

miniBtn=Instance.new("ImageButton",gui); miniBtn.Name="OmniMiniButton"; miniBtn.Size=UDim2.new(0,44,0,44)
miniBtn.Position=UDim2.new(0,20,0,100); miniBtn.BackgroundColor3=Theme.Background; miniBtn.Image=""; miniBtn.BorderSizePixel=0; miniBtn.Visible=false; miniBtn.ZIndex=50
Instance.new("UICorner",miniBtn).CornerRadius=UDim.new(0,12)
local miniStroke=Instance.new("UIStroke",miniBtn); miniStroke.Color=Theme.Accent; miniStroke.Thickness=1.8
local miniIcon=Instance.new("TextLabel",miniBtn); miniIcon.Size=UDim2.new(1,0,1,0); miniIcon.Text="â"; miniIcon.TextColor3=Theme.AccentGlow; miniIcon.Font=Theme.FontBlack; miniIcon.TextSize=20; miniIcon.BackgroundTransparency=1
miniBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)
miniBtn.MouseEnter:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Element}):Play() end)
miniBtn.MouseLeave:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Background}):Play() end)

local function makeMainDraggable(frame)
    local dragging,dragInput,dragStart,startPos,startCloseBtnPos=false,nil,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; if closeBtn then startCloseBtnPos=closeBtn.Position end; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy); if closeBtn and startCloseBtnPos then closeBtn.Position=UDim2.new(startCloseBtnPos.X.Scale,startCloseBtnPos.X.Offset+dx,startCloseBtnPos.Y.Scale,startCloseBtnPos.Y.Offset+dy) end end end)
end
local function makeMiniDraggable(frame)
    local dragging,dragInput,dragStart,startPos=false,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy) end end)
end
makeMainDraggable(main); makeMiniDraggable(miniBtn)
local lastSaveTime=0; main:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastSaveTime>0.5 then lastSaveTime=tick(); saveMainPosition() end end)
local lastMiniSaveTime=0; miniBtn:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastMiniSaveTime>0.5 then lastMiniSaveTime=tick(); saveMiniPosition() end end)
local function updateCloseButtonPosition() if main.Visible then local p=main.Position; closeBtn.Position=UDim2.new(p.X.Scale,p.X.Offset+320-34,p.Y.Scale,p.Y.Offset+18) end end
main:GetPropertyChangedSignal("Position"):Connect(updateCloseButtonPosition); main:GetPropertyChangedSignal("Visible"):Connect(updateCloseButtonPosition)

-- ========== TAB SYSTEM ==========
local tabFrame=Instance.new("Frame",main)
tabFrame.Size=UDim2.new(1,-16,0,30); tabFrame.Position=UDim2.new(0,8,0,56)
tabFrame.BackgroundTransparency=1; tabFrame.BorderSizePixel=0; tabFrame.ZIndex=3
local tabLayout=Instance.new("UIListLayout",tabFrame)
tabLayout.FillDirection=Enum.FillDirection.Horizontal; tabLayout.SortOrder=Enum.SortOrder.LayoutOrder
tabLayout.Padding=UDim.new(0,4); tabLayout.HorizontalAlignment=Enum.HorizontalAlignment.Center

local scroll=Instance.new("ScrollingFrame",main)
scroll.Size=UDim2.new(1,-16,1,-94); scroll.Position=UDim2.new(0,8,0,90)
scroll.BackgroundTransparency=1; scroll.BorderSizePixel=0; scroll.ScrollBarThickness=3
scroll.ScrollBarImageColor3=Theme.Accent; scroll.ScrollBarImageTransparency=0.4
scroll.AutomaticCanvasSize=Enum.AutomaticSize.Y; scroll.CanvasSize=UDim2.new(0,0,0,0); scroll.ZIndex=2
local listLayout=Instance.new("UIListLayout",scroll); listLayout.SortOrder=Enum.SortOrder.LayoutOrder; listLayout.Padding=UDim.new(0,5)
local pad=Instance.new("UIPadding",scroll); pad.PaddingLeft=UDim.new(0,0); pad.PaddingRight=UDim.new(0,0); pad.PaddingTop=UDim.new(0,2); pad.PaddingBottom=UDim.new(0,8)

local tabDefs = {{"Speed",1},{"Features",2},{"Settings",3}}
local tabBtns = {}; local tabPages = {}; local activeTab = "Speed"

for _,td in ipairs(tabDefs) do
    local btn=Instance.new("TextButton",tabFrame)
    btn.Size=UDim2.new(0,80,0,26); btn.BackgroundColor3=(td[1]==activeTab) and Theme.AccentDark or Theme.Element
    btn.BorderSizePixel=0; btn.Text=td[1]:upper(); btn.TextColor3=(td[1]==activeTab) and Theme.AccentGlow or Theme.SubText
    btn.Font=Theme.FontBold; btn.TextSize=9; btn.LayoutOrder=td[2]; btn.ZIndex=4; btn.AutoButtonColor=false
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,8)
    local ts=Instance.new("UIStroke",btn); ts.Color=(td[1]==activeTab) and Theme.Accent or Theme.Border; ts.Thickness=(td[1]==activeTab) and 1.5 or 1
    tabBtns[td[1]]={btn=btn,stroke=ts}
    local page=Instance.new("Frame",scroll)
    page.Size=UDim2.new(1,0,0,0); page.BackgroundTransparency=1; page.BorderSizePixel=0
    page.Visible=(td[1]==activeTab); page.ZIndex=2
    page.AutomaticSize=Enum.AutomaticSize.Y
    local pageLL=Instance.new("UIListLayout",page); pageLL.SortOrder=Enum.SortOrder.LayoutOrder; pageLL.Padding=UDim.new(0,5)
    local pagePad=Instance.new("UIPadding",page); pagePad.PaddingBottom=UDim.new(0,4)
    tabPages[td[1]]=page
    local pageLO=0
    btn.MouseButton1Click:Connect(function()
        if activeTab==td[1] then return end
        activeTab=td[1]
        for name,data in pairs(tabBtns) do
            local isA=(name==activeTab)
            TweenService:Create(data.btn,TweenInfo.new(0.15),{BackgroundColor3=isA and Theme.AccentDark or Theme.Element}):Play()
            TweenService:Create(data.btn,TweenInfo.new(0.15),{TextColor3=isA and Theme.AccentGlow or Theme.SubText}):Play()
            TweenService:Create(data.stroke,TweenInfo.new(0.15),{Color=isA and Theme.Accent or Theme.Border,Thickness=isA and 1.5 or 1}):Play()
            tabPages[name].Visible=isA
        end
    end)
    btn.MouseEnter:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end
    end)
    btn.MouseLeave:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.Element}):Play() end
    end)
end

-- ========== UI HELPERS (per-page) ==========
local pageLOs = {Speed=0, Features=0, Settings=0}
local function LO(tab) pageLOs[tab]=(pageLOs[tab] or 0)+1; return pageLOs[tab] end
local function pg(tab) return tabPages[tab] end

local function makeGap(tab,px) local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,0,0,px or 3); f.BackgroundTransparency=1; f.LayoutOrder=LO(tab) end
local function makeDivider(tab)
    local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,-8,0,1); f.BackgroundColor3=Theme.Border; f.BackgroundTransparency=0; f.BorderSizePixel=0; f.LayoutOrder=LO(tab)
    local g=Instance.new("UIGradient",f); g.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,20)),ColorSequenceKeypoint.new(0.5,Theme.Accent),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,20))})
end
local function makeSectionLabel(tab,text)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,26); row.BackgroundTransparency=1; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-8,1,0); lbl.BackgroundTransparency=1
    lbl.Text="  â "..text:upper(); lbl.TextColor3=Theme.Accent; lbl.Font=Theme.FontBold; lbl.TextSize=10; lbl.TextXAlignment=Enum.TextXAlignment.Left
end

local function makeInputRow(tab,label,default,onChange)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0,100,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.ZIndex=2
    local box=Instance.new("TextBox",row); box.Size=UDim2.new(0,60,0,26); box.Position=UDim2.new(1,-70,0.5,-13); box.BackgroundColor3=Theme.BackgroundSecondary; box.BorderSizePixel=0; box.Text=tostring(default); box.TextColor3=Theme.AccentGlow; box.Font=Theme.FontBold; box.TextSize=13; box.ClearTextOnFocus=false; box.ZIndex=3
    Instance.new("UICorner",box).CornerRadius=UDim.new(0,8)
    local boxStroke=Instance.new("UIStroke",box); boxStroke.Color=Theme.AccentDim; boxStroke.Thickness=1
    box.Focused:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.Element}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
    box.FocusLost:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.BackgroundSecondary}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); local num=tonumber(box.Text); if num~=nil then local fv=math.floor(math.clamp(num,0,500)); box.Text=tostring(fv); if onChange then onChange(tostring(fv)) end; autoSaveConfig() else box.Text=tostring(default) end end)
    return box
end

local function makeStatusRow(tab,label,valTxt)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,32); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0.5,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local val=Instance.new("TextLabel",row); val.Size=UDim2.new(0.45,-10,1,0); val.Position=UDim2.new(0.52,0,0,0); val.BackgroundTransparency=1; val.Text=valTxt; val.TextColor3=Theme.AccentGlow; val.Font=Theme.FontBold; val.TextSize=12; val.TextXAlignment=Enum.TextXAlignment.Right
    return val
end

local function makeKeybindRow(tab,label,currentKey,onChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local btn=Instance.new("TextButton",row); btn.Size=UDim2.new(0,70,0,24); btn.Position=UDim2.new(1,-80,0.5,-12); btn.BackgroundColor3=Theme.Keybind; btn.BorderSizePixel=0; btn.Text="["..currentKey.Name.."]"; btn.TextColor3=Theme.Accent; btn.Font=Theme.FontBold; btn.TextSize=10
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,7)
    local bs=Instance.new("UIStroke",btn); bs.Color=Theme.AccentDim; bs.Thickness=1
    local listening=false; local listenConn
    local function stopListen(key)
        listening=false; if listenConn then listenConn:Disconnect(); listenConn=nil end
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(btn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); btn.TextColor3=Theme.Accent
        if key then btn.Text="["..key.Name.."]"; if onChanged then onChanged(key) end; autoSaveConfig() end
    end
    btn.MouseButton1Click:Connect(function()
        if listening then stopListen(nil); return end
        listening=true; btn.Text="[Â·Â·Â·]"; btn.TextColor3=Theme.AccentGlow; btn.BackgroundColor3=Theme.Element
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
        listenConn=UIS.InputBegan:Connect(function(inp,gp)
            if not listening then return end
            if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode)
            elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode) end
        end)
    end)
    return btn
end

local function makeToggleRow(tab,label,defaultKey,defaultOn,onToggle,onKeyChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,40); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-120,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local keyBtn=nil
    if defaultKey then
        keyBtn=Instance.new("TextButton",row); keyBtn.Size=UDim2.new(0,50,0,22); keyBtn.Position=UDim2.new(1,-108,0.5,-11); keyBtn.BackgroundColor3=Theme.Keybind; keyBtn.BorderSizePixel=0; keyBtn.Text=defaultKey.Name; keyBtn.TextColor3=Theme.Accent; keyBtn.Font=Theme.FontBold; keyBtn.TextSize=9; keyBtn.ZIndex=5
        Instance.new("UICorner",keyBtn).CornerRadius=UDim.new(0,5)
        local ks=Instance.new("UIStroke",keyBtn); ks.Color=Theme.AccentDim; ks.Thickness=1
        local kListening=false; local kConn
        local function kStop(key) kListening=false; if kConn then kConn:Disconnect(); kConn=nil end; TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(keyBtn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); keyBtn.TextColor3=Theme.Accent; if key then keyBtn.Text=key.Name; if onKeyChanged then onKeyChanged(key) end; autoSaveConfig() end end
        keyBtn.MouseButton1Click:Connect(function()
            if kListening then kStop(nil); return end
            kListening=true; keyBtn.Text="Â·Â·Â·"; keyBtn.TextColor3=Theme.AccentGlow; keyBtn.BackgroundColor3=Theme.Element
            TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
            kConn=UIS.InputBegan:Connect(function(inp,gp)
                if not kListening then return end
                if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode)
                elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode) end
            end)
        end)
    end
    local toggleBg=Instance.new("TextButton",row); toggleBg.Size=UDim2.new(0,42,0,22); toggleBg.Position=UDim2.new(1,-48,0.5,-11)
    toggleBg.BackgroundColor3=defaultOn and Theme.AccentDark or Theme.ToggleOff; toggleBg.BorderSizePixel=0; toggleBg.Text=""; toggleBg.ZIndex=5; toggleBg.AutoButtonColor=false
    Instance.new("UICorner",toggleBg).CornerRadius=UDim.new(1,0)
    local tStroke=Instance.new("UIStroke",toggleBg); tStroke.Color=defaultOn and Theme.Accent or Theme.Border; tStroke.Thickness=1.5
    local knob=Instance.new("Frame",toggleBg); knob.Size=UDim2.new(0,17,0,17); knob.Position=defaultOn and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5)
    knob.BackgroundColor3=defaultOn and Theme.AccentGlow or Theme.SubText; knob.BorderSizePixel=0; knob.ZIndex=6
    Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
    local isOn=defaultOn or false
    local function setV(on)
        isOn=on
        TweenService:Create(toggleBg,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{BackgroundColor3=on and Theme.AccentDark or Theme.ToggleOff}):Play()
        TweenService:Create(tStroke,TweenInfo.new(0.22),{Color=on and Theme.Accent or Theme.Border}):Play()
        TweenService:Create(knob,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{Position=on and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=on and Theme.AccentGlow or Theme.SubText}):Play()
        TweenService:Create(rowStroke,TweenInfo.new(0.22),{Color=on and Theme.AccentDim or Theme.Border,Thickness=on and 1.5 or 1}):Play()
    end
    toggleBg.MouseButton1Click:Connect(function() isOn=not isOn; setV(isOn); if onToggle then pcall(onToggle,isOn) end; autoSaveConfig() end)
    toggleBg.MouseEnter:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
    toggleBg.MouseLeave:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)
    return setV, keyBtn
end

-- ===================== BUILD TABS =====================

-- ===== SPEED TAB =====
makeSectionLabel("Speed","Speed")
normalBox=makeInputRow("Speed","Normal",State.normalSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.normalSpeed=n end end)
carryBox=makeInputRow("Speed","Carry",State.carrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.carrySpeed=n end end)
setSpeedToggleUI,speedKeyBtn=makeToggleRow("Speed","Toggle",Keys.speed,false,function(on) toggleSpeedType() end,function(k) Keys.speed=k end)
modeValLbl=makeStatusRow("Speed","Mode","Normal")
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Lagger Speed")
laggerBox=makeInputRow("Speed","Normal",State.laggerSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerSpeed=n end end)
carryLaggerBox=makeInputRow("Speed","Carry",State.laggerCarrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerCarrySpeed=n end end)
setLaggerToggleUI,laggerKeyBtn=makeToggleRow("Speed","Toggle",Keys.lagger,false,function(on) toggleLagger() end,function(k) Keys.lagger=k end)
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Combat")
setAutoBat,autoBatKeyBtn=makeToggleRow("Speed","Bat Aimbot",Keys.autoBat,false,function(on) State.autoBatToggled=on; if not on then resetBend() end end,function(k) Keys.autoBat=k end)
makeGap("Speed",4)

-- ===== FEATURES TAB =====
makeSectionLabel("Features","Auto Steal")
local radiusBox=makeInputRow("Features","Grab Rad",AutoSteal.Radius,function(v) local n=tonumber(v); if n and n>=5 and n<=300 then AutoSteal.Radius=math.floor(n); if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end end)
setInstaGrab=makeToggleRow("Features","Auto Grab",nil,false,function(on) AutoSteal.Enabled=on; if on then startAutoSteal() else stopAutoSteal() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Active Toggles")
setInfJump=makeToggleRow("Features","Infinite Jump",nil,false,function(on) State.infJumpEnabled=on end)
setAntiRag=makeToggleRow("Features","Anti Ragdoll",nil,false,function(on) State.antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
setFps=makeToggleRow("Features","FPS Boost",nil,false,function(on) State.fpsBoostEnabled=on; if on then pcall(applyFPSBoost) end end)
setMedusaCounter=makeToggleRow("Features","Medusa Counter",nil,false,function(on) State.medusaCounterEnabled=on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
setAnimToggle=makeToggleRow("Features","Tryhard Anim",nil,false,function(on) State.animEnabled=on; if on then startAnimToggle() else stopAnimToggle() end end)
setUnwalkToggle=makeToggleRow("Features","Unwalk",nil,false,function(on) if on then startUnwalk() else stopUnwalk() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Movement")
tpDownKeyBtn=makeKeybindRow("Features","TP Down",Keys.tpDown,function(k) Keys.tpDown=k end)
dropBrainrotKeyBtn=makeKeybindRow("Features","Drop Brainrot",Keys.dropBrainrot,function(k) Keys.dropBrainrot=k end)
setAutoLeft,autoLeftKeyBtn=makeToggleRow("Features","Auto Left",Keys.autoLeft,false,function(on) State.autoLeftEnabled=on; if on then startAutoLeft() else stopAutoLeft() end end,function(k) Keys.autoLeft=k end)
setAutoRight,autoRightKeyBtn=makeToggleRow("Features","Auto Right",Keys.autoRight,false,function(on) State.autoRightEnabled=on; if on then startAutoRight() else stopAutoRight() end end,function(k) Keys.autoRight=k end)
makeGap("Features",4)

-- ===== SETTINGS TAB =====
makeSectionLabel("Settings","Interface")
guiHideKeyBtn=makeKeybindRow("Settings","Hide GUI",Keys.guiHide,function(k) Keys.guiHide=k end)
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

makeSectionLabel("Settings","Mobile Panel")
local showRow=Instance.new("Frame",pg("Settings")); showRow.Size=UDim2.new(1,0,0,38); showRow.BackgroundColor3=Theme.Element; showRow.BorderSizePixel=0; showRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",showRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",showRow).Color=Theme.Border
local showLbl=Instance.new("TextLabel",showRow); showLbl.Size=UDim2.new(1,-115,1,0); showLbl.Position=UDim2.new(0,12,0,0); showLbl.BackgroundTransparency=1; showLbl.Text="Show Buttons"; showLbl.TextColor3=Theme.Text; showLbl.Font=Theme.FontMedium; showLbl.TextSize=12; showLbl.TextXAlignment=Enum.TextXAlignment.Left
local showToggleBg=Instance.new("TextButton",showRow); showToggleBg.Size=UDim2.new(0,42,0,22); showToggleBg.Position=UDim2.new(1,-48,0.5,-11); showToggleBg.BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff; showToggleBg.BorderSizePixel=0; showToggleBg.Text=""; showToggleBg.ZIndex=5; showToggleBg.AutoButtonColor=false
Instance.new("UICorner",showToggleBg).CornerRadius=UDim.new(1,0)
local showKnob=Instance.new("Frame",showToggleBg); showKnob.Size=UDim2.new(0,17,0,17); showKnob.Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); showKnob.BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText; showKnob.BorderSizePixel=0; showKnob.ZIndex=6; Instance.new("UICorner",showKnob).CornerRadius=UDim.new(1,0)
showToggleBg.MouseButton1Click:Connect(function() MobileButtons.Visible=not MobileButtons.Visible; if mbPanel then mbPanel.Visible=MobileButtons.Visible end; TweenService:Create(showToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(showKnob,TweenInfo.new(0.22),{Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
showToggleBg.MouseEnter:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
showToggleBg.MouseLeave:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local lockRow=Instance.new("Frame",pg("Settings")); lockRow.Size=UDim2.new(1,0,0,38); lockRow.BackgroundColor3=Theme.Element; lockRow.BorderSizePixel=0; lockRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",lockRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",lockRow).Color=Theme.Border
local lockLbl=Instance.new("TextLabel",lockRow); lockLbl.Size=UDim2.new(1,-115,1,0); lockLbl.Position=UDim2.new(0,12,0,0); lockLbl.BackgroundTransparency=1; lockLbl.Text="Lock Buttons"; lockLbl.TextColor3=Theme.Text; lockLbl.Font=Theme.FontMedium; lockLbl.TextSize=12; lockLbl.TextXAlignment=Enum.TextXAlignment.Left
local lockToggleBg=Instance.new("TextButton",lockRow); lockToggleBg.Size=UDim2.new(0,42,0,22); lockToggleBg.Position=UDim2.new(1,-48,0.5,-11); lockToggleBg.BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff; lockToggleBg.BorderSizePixel=0; lockToggleBg.Text=""; lockToggleBg.ZIndex=5; lockToggleBg.AutoButtonColor=false
Instance.new("UICorner",lockToggleBg).CornerRadius=UDim.new(1,0)
local lockKnob=Instance.new("Frame",lockToggleBg); lockKnob.Size=UDim2.new(0,17,0,17); lockKnob.Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); lockKnob.BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText; lockKnob.BorderSizePixel=0; lockKnob.ZIndex=6; Instance.new("UICorner",lockKnob).CornerRadius=UDim.new(1,0)
lockToggleBg.MouseButton1Click:Connect(function() MobileButtons.Locked=not MobileButtons.Locked; TweenService:Create(lockToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(lockKnob,TweenInfo.new(0.22),{Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
lockToggleBg.MouseEnter:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
lockToggleBg.MouseLeave:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local sizeRow=Instance.new("Frame",pg("Settings")); sizeRow.Size=UDim2.new(1,0,0,38); sizeRow.BackgroundColor3=Theme.Element; sizeRow.BorderSizePixel=0; sizeRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",sizeRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",sizeRow).Color=Theme.Border
local sizeLbl=Instance.new("TextLabel",sizeRow); sizeLbl.Size=UDim2.new(0,100,1,0); sizeLbl.Position=UDim2.new(0,12,0,0); sizeLbl.BackgroundTransparency=1; sizeLbl.Text="Btn Size"; sizeLbl.TextColor3=Theme.Text; sizeLbl.Font=Theme.FontMedium; sizeLbl.TextSize=12; sizeLbl.TextXAlignment=Enum.TextXAlignment.Left
local sizeValLbl=Instance.new("TextLabel",sizeRow); sizeValLbl.Size=UDim2.new(0,40,0,16); sizeValLbl.Position=UDim2.new(0,112,0.5,-8); sizeValLbl.BackgroundTransparency=1; sizeValLbl.Text=MB_BTN_W.."px"; sizeValLbl.TextColor3=Theme.AccentGlow; sizeValLbl.Font=Theme.FontBold; sizeValLbl.TextSize=9; sizeValLbl.TextXAlignment=Enum.TextXAlignment.Center
local decBtn=Instance.new("TextButton",sizeRow); decBtn.Size=UDim2.new(0,24,0,24); decBtn.Position=UDim2.new(1,-78,0.5,-12); decBtn.BackgroundColor3=Theme.BackgroundSecondary; decBtn.BorderSizePixel=0; decBtn.Text="-"; decBtn.TextColor3=Theme.Text; decBtn.Font=Theme.FontBlack; decBtn.TextSize=14; decBtn.ZIndex=5
Instance.new("UICorner",decBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",decBtn).Color=Theme.Border
local incBtn=Instance.new("TextButton",sizeRow); incBtn.Size=UDim2.new(0,24,0,24); incBtn.Position=UDim2.new(1,-48,0.5,-12); incBtn.BackgroundColor3=Theme.BackgroundSecondary; incBtn.BorderSizePixel=0; incBtn.Text="+"; incBtn.TextColor3=Theme.Text; incBtn.Font=Theme.FontBlack; incBtn.TextSize=14; incBtn.ZIndex=5
Instance.new("UICorner",incBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",incBtn).Color=Theme.Border
decBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W-4,MB_BTN_H-3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
incBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W+4,MB_BTN_H+3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
for _,b in pairs({decBtn,incBtn}) do
    b.MouseEnter:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
    b.MouseLeave:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
end
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

local saveConfigRow=Instance.new("Frame",pg("Settings")); saveConfigRow.Size=UDim2.new(1,0,0,40); saveConfigRow.BackgroundColor3=Theme.Element; saveConfigRow.BorderSizePixel=0; saveConfigRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",saveConfigRow).CornerRadius=UDim.new(0,10)
local saveConfigBtn=Instance.new("TextButton",saveConfigRow); saveConfigBtn.Size=UDim2.new(1,-16,0,28); saveConfigBtn.Position=UDim2.new(0,8,0.5,-14)
saveConfigBtn.BackgroundColor3=Color3.fromRGB(0,40,80); saveConfigBtn.BorderSizePixel=0; saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; saveConfigBtn.Font=Theme.FontBold; saveConfigBtn.TextSize=12; saveConfigBtn.ZIndex=5; saveConfigBtn.AutoButtonColor=false
Instance.new("UICorner",saveConfigBtn).CornerRadius=UDim.new(0,9)
local saveBtnStroke=Instance.new("UIStroke",saveConfigBtn); saveBtnStroke.Color=Theme.Accent; saveBtnStroke.Thickness=1.5
saveConfigBtn.MouseEnter:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,60,120)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.AccentGlow}):Play() end)
saveConfigBtn.MouseLeave:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,40,80)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
saveConfigBtn.MouseButton1Click:Connect(function() forceSaveConfig(); saveConfigBtn.Text="â  SAVED!"; saveConfigBtn.TextColor3=Theme.Success; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Success}):Play(); task.wait(1.2); saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)

local resetMbRow=Instance.new("Frame",pg("Settings")); resetMbRow.Size=UDim2.new(1,0,0,36); resetMbRow.BackgroundColor3=Theme.Element; resetMbRow.BorderSizePixel=0; resetMbRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",resetMbRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",resetMbRow).Color=Theme.Border
local resetMbBtn=Instance.new("TextButton",resetMbRow); resetMbBtn.Size=UDim2.new(1,-16,0,24); resetMbBtn.Position=UDim2.new(0,8,0.5,-12)
resetMbBtn.BackgroundColor3=Theme.BackgroundSecondary; resetMbBtn.BorderSizePixel=0; resetMbBtn.Text="Reset Button Panel Position"; resetMbBtn.TextColor3=Theme.SubText; resetMbBtn.Font=Theme.FontMedium; resetMbBtn.TextSize=10; resetMbBtn.ZIndex=5; resetMbBtn.AutoButtonColor=false
Instance.new("UICorner",resetMbBtn).CornerRadius=UDim.new(0,7)
resetMbBtn.MouseEnter:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
resetMbBtn.MouseLeave:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
resetMbBtn.MouseButton1Click:Connect(function() if mbPanel then mbPanel.Position=UDim2.new(1,-(MB_TOTAL_W+8),0.5,-(MB_TOTAL_H/2)) end end)

makeGap("Settings",2)
local footerLbl=Instance.new("TextLabel",pg("Settings")); footerLbl.Size=UDim2.new(1,0,0,18); footerLbl.BackgroundTransparency=1; footerLbl.LayoutOrder=LO("Settings"); footerLbl.Text="â RBG â"; footerLbl.TextColor3=Theme.AccentDim; footerLbl.Font=Theme.FontBold; footerLbl.TextSize=9; footerLbl.TextXAlignment=Enum.TextXAlignment.Center

-- ========== TOGGLE GUI VISIBILITY ==========
toggleGuiVis=function()
    State.guiVisible=not State.guiVisible
    if main then main.Visible=State.guiVisible; closeBtn.Visible=State.guiVisible; if miniBtn then miniBtn.Visible=not State.guiVisible end end
end

-- ========== LOGIC ==========
local function findMedusa()
    local char=LP.Character; if not char then return nil end
    for _,tool in ipairs(char:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end
    local bp2=LP:FindFirstChild("Backpack"); if bp2 then for _,tool in ipairs(bp2:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end end
    return nil
end
local function useMedusaCounter()
    if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<25 then return end; local char=LP.Character; if not char then return end
    State.medusaDebounce=true; local med=findMedusa(); if not med then State.medusaDebounce=false; return end
    if med.Parent~=char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
end
local function onAnchorChanged(part) return part:GetPropertyChangedSignal("Anchored"):Connect(function() if part.Anchored and part.Transparency==1 then useMedusaCounter() end end) end
setupMedusaCounter=function(char) stopMedusaCounter(); if not char then return end; for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end; table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end)) end
stopMedusaCounter=function() for _,c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end; Conns.anchor={} end

startAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect() end; State.autoLeftPhase=1
    Conns.autoLeft=RunService.Heartbeat:Connect(function()
        if not State.autoLeftEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoLeftPhase==1 then
            local tgt=Vector3.new(POS.L1.X,root.Position.Y,POS.L1.Z); if (tgt-root.Position).Magnitude<1 then State.autoLeftPhase=2; return end
            local d=(POS.L1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoLeftPhase==2 then
            local tgt=Vector3.new(POS.L2.X,root.Position.Y,POS.L2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoLeftEnabled=false; if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1; if setAutoLeft then setAutoLeft(false) end; return end
            local d=(POS.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect() end; State.autoRightPhase=1
    Conns.autoRight=RunService.Heartbeat:Connect(function()
        if not State.autoRightEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoRightPhase==1 then
            local tgt=Vector3.new(POS.R1.X,root.Position.Y,POS.R1.Z); if (tgt-root.Position).Magnitude<1 then State.autoRightPhase=2; return end
            local d=(POS.R1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoRightPhase==2 then
            local tgt=Vector3.new(POS.R2.X,root.Position.Y,POS.R2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoRightEnabled=false; if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1; if setAutoRight then setAutoRight(false) end; return end
            local d=(POS.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAntiRagdoll=function()
    if Conns.antiRag then return end
    Conns.antiRag=RunService.Heartbeat:Connect(function()
        local char=LP.Character; if not char then return end; local hum2=char:FindFirstChildOfClass("Humanoid"); local root=char:FindFirstChild("HumanoidRootPart")
        if hum2 then local st=hum2:GetState(); if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then hum2:ChangeState(Enum.HumanoidStateType.Running); workspace.CurrentCamera.CameraSubject=hum2; pcall(function() local pm=LP.PlayerScripts:FindFirstChild("PlayerModule"); if pm then require(pm:FindFirstChild("ControlModule")):Enable() end end); if root then root.Velocity=Vector3.new(0,0,0); root.RotVelocity=Vector3.new(0,0,0) end end end
        for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled=true end end
    end)
end
stopAntiRagdoll=function() if Conns.antiRag then Conns.antiRag:Disconnect(); Conns.antiRag=nil end end
applyFPSBoost=function()
    pcall(function() setfpscap(999999999) end)
    local function processObj(v) pcall(function() if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0 elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1 elseif v:IsA("SpecialMesh") then v.TextureId="" elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false elseif v:IsA("SurfaceAppearance") or v:IsA("MaterialVariant") then v:Destroy() elseif v:IsA("Attachment") then v.Visible=false end end) end
    for _,v in pairs(workspace:GetDescendants()) do processObj(v) end
    pcall(function() local lighting=game:GetService("Lighting"); for _,v in pairs(lighting:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end; pcall(function() sethiddenproperty(game:GetService("Lighting"),"Technology",Enum.Technology.Legacy) end); local lighting2=game:GetService("Lighting"); lighting2.GlobalShadows=false; lighting2.FogEnd=9e9; lighting2.Brightness=0; local terrain=workspace:FindFirstChildOfClass("Terrain"); if terrain then pcall(function() sethiddenproperty(terrain,"Decoration",false) end); terrain.WaterReflectance=0; terrain.WaterTransparency=0.7; terrain.WaterWaveSize=0; terrain.WaterWaveSpeed=0 end end)
    workspace.DescendantAdded:Connect(function(v) if State.fpsBoostEnabled then task.spawn(processObj,v) end end)
end

-- ========== CHARACTER SETUP ==========
local function setupChar(char)
    task.wait(0.1); h=char:WaitForChild("Humanoid",5); hrp=char:WaitForChild("HumanoidRootPart",5); State.originalC0=nil
    if not h or not hrp then return end
    local head=char:FindFirstChild("Head")
    if head then
        local oldBB=head:FindFirstChild("SpeedBillboard"); if oldBB then oldBB:Destroy() end
        local bb=Instance.new("BillboardGui",head); bb.Name="SpeedBillboard"; bb.Size=UDim2.new(0,120,0,22); bb.StudsOffset=Vector3.new(0,3,0); bb.AlwaysOnTop=true
        speedLbl=Instance.new("TextLabel",bb); speedLbl.Size=UDim2.new(1,0,1,0); speedLbl.BackgroundTransparency=1; speedLbl.TextColor3=Theme.AccentGlow; speedLbl.Font=Theme.FontBold; speedLbl.TextScaled=true; speedLbl.TextStrokeTransparency=0.5; speedLbl.TextStrokeColor3=Color3.fromRGB(0,20,40)
    end
    if State.antiRagdollEnabled and not Conns.antiRag then task.wait(0.5); startAntiRagdoll() end
    if State.medusaCounterEnabled then setupMedusaCounter(char) end
    if State.animEnabled then task.wait(0.3); saveOriginalAnims(char); applyAnimPack(char) end
    if State.unwalkEnabled then State.unwalkEnabled=false; task.wait(0.3); startUnwalk() end
end
LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

RunService.Stepped:Connect(function()
    for _,p in ipairs(Players:GetPlayers()) do if p~=LP and p.Character then for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end end end
end)

UIS.JumpRequest:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root then root.Velocity=Vector3.new(root.Velocity.X,55,root.Velocity.Z) end
end)
RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root and root.Velocity.Y<-120 then root.Velocity=Vector3.new(root.Velocity.X,-120,root.Velocity.Z) end
end)

RunService.RenderStepped:Connect(function()
    if not (h and hrp) then return end; if State._tpInProgress then return end
    if State.autoLeftEnabled or State.autoRightEnabled then if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end; return end
    local md=h.MoveDirection; local spd=getCurrentSpeed()
    if md.Magnitude>0 then State.lastMoveDir=md; hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
    elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude>0 then local anyHeld=false; for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true; break end end; if anyHeld then hrp.Velocity=Vector3.new(State.lastMoveDir.X*spd,hrp.Velocity.Y,State.lastMoveDir.Z*spd) end end
    if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end
end)

RunService.Heartbeat:Connect(function(dt)
    if not (State.autoBatToggled and h and hrp) then resetBend(); return end
    local target,dist=getClosestPlayer(); if not (target and target.Character) then resetBend(); return end
    local tr=target.Character:FindFirstChild("HumanoidRootPart"); if not tr then resetBend(); return end
    local rj=getRootJoint(); if rj and not State.originalC0 then State.originalC0=rj.C0 end
    local yDiff=tr.Position.Y-hrp.Position.Y; local yVel=math.clamp(yDiff*20,-120,120)
    if dist>6 then
        local dir=(tr.Position-hrp.Position).Unit; hrp.AssemblyLinearVelocity=Vector3.new(dir.X*59,yVel,dir.Z*59)
        if rj then rj.C0=State.originalC0*CFrame.Angles(math.rad(40),0,0) end
    else
        State.crazyAngle=State.crazyAngle+dt*40
        local offsetX=math.cos(State.crazyAngle)*0.8; local offsetZ=math.sin(State.crazyAngle)*0.8
        local targetPos=tr.Position+Vector3.new(offsetX,0,offsetZ); local flatDir=Vector3.new(targetPos.X-hrp.Position.X,0,targetPos.Z-hrp.Position.Z)
        hrp.AssemblyLinearVelocity=Vector3.new(flatDir.Unit.X*200,yVel,flatDir.Unit.Z*200)
        if rj then local t=tick(); local fwdBack=math.sin(t*25)*math.rad(50); local sideways=math.cos(t*20)*math.rad(30); rj.C0=State.originalC0*CFrame.Angles(fwdBack,0,sideways) end
        tryHitBat()
    end
end)

UIS.InputBegan:Connect(function(inp,gp)
    if gp then return end
    if inp.UserInputType~=Enum.UserInputType.Keyboard and inp.UserInputType~=Enum.UserInputType.Gamepad1 then return end
    local kc=inp.KeyCode
    if (State.autoLeftEnabled or State.autoRightEnabled) then if MOVE_KEYS[kc] then return end end
    if kc==Keys.speed then toggleSpeedType()
    elseif kc==Keys.lagger then toggleLagger()
    elseif kc==Keys.autoBat then State.autoBatToggled=not State.autoBatToggled; if not State.autoBatToggled then resetBend() end; setAutoBat(State.autoBatToggled); autoSaveConfig()
    elseif kc==Keys.autoLeft then State.autoLeftEnabled=not State.autoLeftEnabled; if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end; if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    elseif kc==Keys.autoRight then State.autoRightEnabled=not State.autoRightEnabled; if setAutoRight then setAutoRight(State.autoRightEnabled) end; if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    elseif kc==Keys.dropBrainrot then task.spawn(runDropBrainrot)
    elseif kc==Keys.tpDown then tpToGround()
    elseif kc==Keys.guiHide then toggleGuiVis() end
end)

task.spawn(function() while task.wait(0.5) do pcall(function() if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end) end end)

local function loadConfig()
    local hasFile=false; pcall(function() hasFile=isfile("JispiHubConfig.json") end); if not hasFile then return end
    local ok,cfg=pcall(function() return HttpService:JSONDecode(readfile("JispiHubConfig.json")) end); if not ok or not cfg then return end
    if cfg.normalSpeed and type(cfg.normalSpeed)=="number" then State.normalSpeed=cfg.normalSpeed; normalBox.Text=tostring(cfg.normalSpeed) end
    if cfg.carrySpeed  and type(cfg.carrySpeed)=="number"  then State.carrySpeed=cfg.carrySpeed;   carryBox.Text=tostring(cfg.carrySpeed)   end
    if cfg.laggerSpeed and type(cfg.laggerSpeed)=="number" then State.laggerSpeed=cfg.laggerSpeed; laggerBox.Text=tostring(cfg.laggerSpeed) end
    if cfg.laggerCarrySpeed and type(cfg.laggerCarrySpeed)=="number" then State.laggerCarrySpeed=cfg.laggerCarrySpeed; carryLaggerBox.Text=tostring(cfg.laggerCarrySpeed) end
    if cfg.speedType=="normal" or cfg.speedType=="carry" then State.speedType=cfg.speedType end
    if type(cfg.laggerActive)=="boolean" then State.laggerActive=cfg.laggerActive end
    if cfg.autoBatKey  and Enum.KeyCode[cfg.autoBatKey]    then Keys.autoBat=Enum.KeyCode[cfg.autoBatKey];   if autoBatKeyBtn  then autoBatKeyBtn.Text="["..cfg.autoBatKey.."]"   end end
    if cfg.speedKey    and Enum.KeyCode[cfg.speedKey]      then Keys.speed=Enum.KeyCode[cfg.speedKey]        end
    if cfg.laggerKey   and Enum.KeyCode[cfg.laggerKey]     then Keys.lagger=Enum.KeyCode[cfg.laggerKey]      end
    if cfg.autoLeftKey  and Enum.KeyCode[cfg.autoLeftKey]  then Keys.autoLeft=Enum.KeyCode[cfg.autoLeftKey];   if autoLeftKeyBtn  then autoLeftKeyBtn.Text="["..cfg.autoLeftKey.."]"   end end
    if cfg.autoRightKey and Enum.KeyCode[cfg.autoRightKey] then Keys.autoRight=Enum.KeyCode[cfg.autoRightKey]; if autoRightKeyBtn then autoRightKeyBtn.Text="["..cfg.autoRightKey.."]" end end
    if cfg.tpDownKey    and Enum.KeyCode[cfg.tpDownKey]    then Keys.tpDown=Enum.KeyCode[cfg.tpDownKey];       if tpDownKeyBtn    then tpDownKeyBtn.Text="["..cfg.tpDownKey.."]"        end end
    if cfg.grabRadius and type(cfg.grabRadius)=="number" then AutoSteal.Radius=cfg.grabRadius; if progressRadLbl then progressRadLbl.Text="Radius: "..cfg.grabRadius end end
    if cfg.autoStealEnabled then AutoSteal.Enabled=true; setInstaGrab(true); pcall(startAutoSteal) end
    if cfg.infJump      then State.infJumpEnabled=true;      setInfJump(true)      end
    if cfg.antiRagdoll  then State.antiRagdollEnabled=true;  setAntiRag(true);     startAntiRagdoll() end
    if cfg.fpsBoost     then State.fpsBoostEnabled=true;     setFps(true);         applyFPSBoost()    end
    if cfg.medusaCounter then State.medusaCounterEnabled=true; setMedusaCounter(true); setupMedusaCounter(LP.Character) end
    if cfg.dropBrainrotKey and Enum.KeyCode[cfg.dropBrainrotKey] then Keys.dropBrainrot=Enum.KeyCode[cfg.dropBrainrotKey]; if dropBrainrotKeyBtn then dropBrainrotKeyBtn.Text="["..cfg.dropBrainrotKey.."]"                    bat:Activate() -- Ø¨ÙØ®
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local ContentProvider = game:GetService("ContentProvider")
local LP = Players.LocalPlayer

local State = {
    normalSpeed = 60, carrySpeed = 30, laggerSpeed = 13, laggerCarrySpeed = 13,
    speedType = "normal", laggerActive = false, autoBatToggled = false,
    hittingCooldown = false, infJumpEnabled = false, antiRagdollEnabled = false,
    fpsBoostEnabled = false, guiVisible = true, isStealing = false,
    stealStartTime = nil, lastStealTick = 0, medusaLastUsed = 0,
    medusaDebounce = false, medusaCounterEnabled = false, dropBrainrotActive = false,
    autoLeftEnabled = false, autoRightEnabled = false, autoLeftPhase = 1,
    autoRightPhase = 1, _tpInProgress = false, lastMoveDir = Vector3.new(0,0,0),
    animEnabled = false, unwalkEnabled = false, originalC0 = nil, crazyAngle = 0,
}

local Keys = {
    autoBat = Enum.KeyCode.E, speed = Enum.KeyCode.Q, lagger = Enum.KeyCode.C,
    guiHide = Enum.KeyCode.LeftControl, autoLeft = Enum.KeyCode.L,
    autoRight = Enum.KeyCode.R, dropBrainrot = Enum.KeyCode.H, tpDown = Enum.KeyCode.T,
}

local main, closeBtn, miniBtn
local toggleGuiVis

local MobileButtons = { Visible = true, Locked = false, BtnsObjects = {}, BtnRefs = {}, Buttons = {}, HideGuiBtn = nil }

local MB_BTN_W = 58
local MB_BTN_H = 48
local MB_COLS = 2
local MB_ROWS = 4
local MB_PAD = 3
local MB_CORNER = 14
local MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
local MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
local mbPanel = nil

local Theme = {
    Background      = Color3.fromRGB(8, 10, 20),
    BackgroundSecondary = Color3.fromRGB(12, 15, 28),
    Header          = Color3.fromRGB(5, 8, 18),
    Element         = Color3.fromRGB(14, 18, 35),
    ElementHover    = Color3.fromRGB(20, 26, 50),
    Text            = Color3.fromRGB(200, 220, 255),
    SubText         = Color3.fromRGB(80, 110, 160),
    Accent          = Color3.fromRGB(0, 180, 255),
    AccentDark      = Color3.fromRGB(0, 100, 200),
    AccentGlow      = Color3.fromRGB(0, 220, 255),
    AccentDim       = Color3.fromRGB(0, 60, 120),
    Border          = Color3.fromRGB(0, 60, 120),
    BorderLight     = Color3.fromRGB(0, 100, 180),
    Success         = Color3.fromRGB(0, 255, 160),
    Danger          = Color3.fromRGB(255, 50, 80),
    Keybind         = Color3.fromRGB(10, 20, 45),
    KeybindText     = Color3.fromRGB(0, 180, 255),
    ToggleOff       = Color3.fromRGB(18, 22, 45),
    Font            = Enum.Font.Gotham,
    FontBold        = Enum.Font.GothamBold,
    FontBlack       = Enum.Font.GothamBlack,
    FontMedium      = Enum.Font.GothamMedium,
}

local AutoSteal = {
    Enabled = false, Radius = 20, Duration = 1.3, IsStealing = false,
    Data = {}, ProgressFill = nil, ProgressText = nil,
}

local function isMyPlotByName(plotName)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return false end
    local plot = plots:FindFirstChild(plotName); if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then local yb = sign:FindFirstChild("YourBase"); if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end end
    return false
end

local function findNearestPrompt()
    local char = LP.Character; local root = char and char:FindFirstChild("HumanoidRootPart"); if not root then return nil end
    local plots = workspace:FindFirstChild("Plots"); if not plots then return nil end
    local bestPrompt, bestDist, bestName = nil, math.huge, nil
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local podiums = plot:FindFirstChild("AnimalPodiums"); if not podiums then continue end
        for _, pod in ipairs(podiums:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base"); local spawn = base and base:FindFirstChild("Spawn")
                if spawn then
                    local dist = (spawn.Position - root.Position).Magnitude
                    if dist < bestDist and dist <= AutoSteal.Radius then
                        local att = spawn:FindFirstChild("PromptAttachment")
                        if att then for _, child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") then bestPrompt, bestDist, bestName = child, dist, pod.Name; break end
                        end end
                    end
                end
            end)
        end
    end
    return bestPrompt, bestDist, bestName
end

local function executeSteal(prompt, animalName)
    if AutoSteal.IsStealing then return end
    if not AutoSteal.Data[prompt] then
        AutoSteal.Data[prompt] = {hold = {}, trigger = {}, ready = true}
        pcall(function()
            if getconnections then
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(AutoSteal.Data[prompt].hold, c.Function) end end
                for _, c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(AutoSteal.Data[prompt].trigger, c.Function) end end
            end
        end)
    end
    local data = AutoSteal.Data[prompt]; if not data.ready then return end
    data.ready = false; AutoSteal.IsStealing = true; local startTime = tick()
    local conn; conn = RunService.Heartbeat:Connect(function()
        if not AutoSteal.IsStealing then conn:Disconnect(); return end
        local prog = math.clamp((tick() - startTime) / AutoSteal.Duration, 0, 1)
        if AutoSteal.ProgressFill then AutoSteal.ProgressFill.Size = UDim2.new(prog, 0, 1, 0) end
    end)
    task.spawn(function()
        for _, f in ipairs(data.hold) do task.spawn(f) end
        task.wait(AutoSteal.Duration)
        for _, f in ipairs(data.trigger) do task.spawn(f) end
        AutoSteal.IsStealing = false; data.ready = true
        task.wait(0.6)
        if not AutoSteal.IsStealing and AutoSteal.ProgressFill then
            TweenService:Create(AutoSteal.ProgressFill, TweenInfo.new(0.4), {Size = UDim2.new(0,0,1,0)}):Play()
        end
    end)
end

local autoStealConnection = nil
local function startAutoSteal()
    if autoStealConnection then return end
    autoStealConnection = RunService.Heartbeat:Connect(function()
        if AutoSteal.Enabled and not AutoSteal.IsStealing then
            local p, _, name = findNearestPrompt(); if p then executeSteal(p, name) end
        end
    end)
end
local function stopAutoSteal()
    if autoStealConnection then autoStealConnection:Disconnect(); autoStealConnection = nil end
    AutoSteal.IsStealing = false
    for k, v in pairs(AutoSteal.Data) do if v.ready ~= nil then v.ready = true end end
end

local MOVE_KEYS = {[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local DROP_ASCEND_DURATION = 0.2; local DROP_ASCEND_SPEED = 150
local POS = {L1=Vector3.new(-476.48,-6.28,92.73),L2=Vector3.new(-483.12,-4.95,94.80),R1=Vector3.new(-476.16,-6.52,25.62),R2=Vector3.new(-483.04,-5.09,23.14)}
local Conns = {autoSteal=nil,antiRag=nil,autoLeft=nil,autoRight=nil,anchor={},progress=nil}
local h, hrp, speedLbl
local setAutoLeft, setAutoRight
local setInstaGrab, setAutoBat, setInfJump, setAntiRag, setFps, setMedusaCounter
local setAnimToggle, setUnwalkToggle
local setupMedusaCounter, stopMedusaCounter, startAntiRagdoll, stopAntiRagdoll, applyFPSBoost
local modeValLbl, normalBox, carryBox, laggerBox, carryLaggerBox
local autoBatKeyBtn, speedKeyBtn, laggerKeyBtn, autoLeftKeyBtn, autoRightKeyBtn, guiHideKeyBtn, dropBrainrotKeyBtn, tpDownKeyBtn
local setSpeedToggleUI, setLaggerToggleUI
local progressRadLbl
local startAutoLeft, stopAutoLeft, startAutoRight, stopAutoRight

local function getRootJoint()
    local char2 = LP.Character; local torso = char2 and char2:FindFirstChild("LowerTorso")
    return torso and torso:FindFirstChild("Root")
end
local function resetBend()
    local rj = getRootJoint()
    if rj and State.originalC0 then rj.C0 = State.originalC0; State.originalC0 = nil end
end
local function getBat()
    local char = LP.Character; if not char then return nil end
    local tool = char:FindFirstChild("Bat"); if tool then return tool end
    local bp2 = LP:FindFirstChild("Backpack")
    if bp2 then tool = bp2:FindFirstChild("Bat"); if tool then tool.Parent = char; return tool end end
    return nil
end
local function tryHitBat()
    if State.hittingCooldown then return end; State.hittingCooldown = true
    pcall(function()
        local bat = getBat(); if bat then bat:Activate(); local ev = bat:FindFirstChildWhichIsA("RemoteEvent"); if ev then ev:FireServer() end end
    end)
    task.delay(0.08, function() State.hittingCooldown = false end)
end
local function getClosestPlayer()
    if not hrp then return nil, math.huge end
    local cp, cd = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if tr then local d = (hrp.Position - tr.Position).Magnitude; if d < cd then cd = d; cp = p end end
        end
    end
    return cp, cd
end

local saveDebounce = false
local function autoSaveConfig()
    if saveDebounce then return end; saveDebounce = true
    task.delay(0.5, function()
        local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
        pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end); saveDebounce = false
    end)
end
local function forceSaveConfig()
    local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
    pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end)
end
localsteal(steal) local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local Conns = {autoSteal=nil,antiRag=nil,anchor={},progress=nil,aimbot=nil,batCounter=nil}
local PLOT_CACHE_DURATION = 2
local PROMPT_CACHE_REFRESH = 0.15
local STEAL_COOLDOWN = 0.1
local MEDUSA_COOLDOWN = 25
local BAT_COUNTER_COOLDOWN = 0
local batCounterDebounce = false
local batCounterLastUsed = 0
local progressRadLbl,progressFill,progressPct
local modeValLbl
local _aimTarget = nil
local _aimLastScan = 0

local VYSE_AIMBOT_SPEED = 56.5
local VYSE_HIT_DIST = 5
local SWING_COOLDOWN = 0.08
local hittingCooldown = false

local desyncEnabled = false
local setDesyncVisual = nil
local desyncActive = false
local desyncSpeedConn = nil
local G_desyncAnimate = nil

local fpsBoostEnabled = false

local function refreshUIToggles()
    if setSpeedToggleUI then setSpeedToggleUI(State.speedType == "carry") end
    if setLaggerToggleUI then setLaggerToggleUI(State.laggerActive) end
    if modeValLbl then
        if State.laggerActive then modeValLbl.Text = (State.speedType=="normal") and "â¡ Lagger Normal" or "â¡ Lagger Carry"
        else modeValLbl.Text = (State.speedType=="normal") and "Normal" or "Carry" end
    end
end
local function toggleSpeedType()
    State.speedType = (State.speedType=="normal") and "carry" or "normal"; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.carrySpeed then MobileButtons.BtnRefs.carrySpeed:SetAttribute("MB_On", State.speedType=="carry") end
end
local function toggleLagger()
    State.laggerActive = not State.laggerActive; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.lagger then MobileButtons.BtnRefs.lagger:SetAttribute("MB_On", State.laggerActive) end
end
local function getCurrentSpeed()
    if State.laggerActive then return State.speedType=="normal" and State.laggerSpeed or State.laggerCarrySpeed
    else return State.speedType=="normal" and State.normalSpeed or State.carrySpeed end
end
local function getAutoMoveSpeed()
    return State.laggerActive and State.laggerSpeed or State.normalSpeed
end
local function tpToGround()
    local char = LP.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
    local rp = RaycastParams.new(); rp.FilterType = Enum.RaycastFilterType.Exclude; rp.FilterDescendantsInstances = {char}
    local rr = workspace:Raycast(root.Position, Vector3.new(0,-500,0), rp)
    if rr then root.CFrame = CFrame.new(rr.Position + Vector3.new(0,3,0)) else root.CFrame = root.CFrame * CFrame.new(0,-20,0) end
end
local function runDropBrainrot()
    if State.dropBrainrotActive then return end
    local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if not root then return end
    State.dropBrainrotActive=true; local t0=tick(); local dc
    dc=RunService.Heartbeat:Connect(function()
        local r=char and char:FindFirstChild("HumanoidRootPart"); if not r then dc:Disconnect(); State.dropBrainrotActive=false; return end
        if tick()-t0>=DROP_ASCEND_DURATION then
            dc:Disconnect()
            local rp=RaycastParams.new(); rp.FilterDescendantsInstances={char}; rp.FilterType=Enum.RaycastFilterType.Exclude
            local rr=workspace:Raycast(r.Position,Vector3.new(0,-2000,0),rp)
            if rr then local hum2=char:FindFirstChildOfClass("Humanoid"); local off=(hum2 and hum2.HipHeight or 2)+(r.Size.Y/2); r.CFrame=CFrame.new(r.Position.X,rr.Position.Y+off,r.Position.Z); r.AssemblyLinearVelocity=Vector3.new(0,0,0) end
            State.dropBrainrotActive=false; return
        end
        r.AssemblyLinearVelocity=Vector3.new(r.AssemblyLinearVelocity.X,DROP_ASCEND_SPEED,r.AssemblyLinearVelocity.Z)
    end)
end

-- ========== MOBILE PANEL ==========
local function createMobilePanel()
    local panel = Instance.new("ScreenGui")
    panel.Name="OmniHubButtons"; panel.Parent=LP:WaitForChild("PlayerGui"); panel.ResetOnSpawn=false; panel.ZIndexBehavior=Enum.ZIndexBehavior.Sibling

    mbPanel = Instance.new("Frame", panel)
    mbPanel.Name = "MobilePanel"
    mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H)
    mbPanel.Position = UDim2.new(1, -(MB_TOTAL_W+8), 0.5, -(MB_TOTAL_H/2))
    mbPanel.BackgroundColor3 = Theme.Background
    mbPanel.BorderSizePixel = 0; mbPanel.Active = true; mbPanel.ZIndex = 50
    Instance.new("UICorner", mbPanel).CornerRadius = UDim.new(0, MB_CORNER+4)
    local mbStroke = Instance.new("UIStroke", mbPanel)
    mbStroke.Color = Theme.Accent; mbStroke.Thickness = 1.5; mbStroke.Transparency = 0.45

    local dragStart, startPos, moved = nil, nil, false
    local DRAG_THRESHOLD = 8
    mbPanel.InputBegan:Connect(function(inp)
        if MobileButtons.Locked then return end
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
            dragStart=inp.Position; startPos=mbPanel.Position; moved=false
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if MobileButtons.Locked or not dragStart then return end
        if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then
            local d=inp.Position-dragStart
            if not moved and (math.abs(d.X)+math.abs(d.Y))>DRAG_THRESHOLD then moved=true end
            if moved then mbPanel.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+d.X,startPos.Y.Scale,startPos.Y.Offset+d.Y) end
        end
    end)
    UIS.InputEnded:Connect(function(inp)
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragStart=nil; moved=false end
    end)

    local function makeMobileBtn(label, row, col, onClick)
        local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
        local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
        local btn = Instance.new("TextButton", mbPanel)
        btn.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
        btn.Position = UDim2.new(0, x, 0, y)
        btn.BackgroundColor3 = Theme.Element; btn.BorderSizePixel = 0
        btn.Text = ""; btn.AutoButtonColor = false; btn.ZIndex = 52
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, MB_CORNER)
        local bs = Instance.new("UIStroke", btn); bs.Color = Theme.Accent; bs.Thickness = 1; bs.Transparency = 0.7
        local lbl = Instance.new("TextLabel", btn)
        lbl.Size = UDim2.new(1,0,1,0); lbl.BackgroundTransparency = 1
        lbl.Text = label; lbl.TextColor3 = Theme.Accent; lbl.Font = Theme.FontBold
        lbl.TextSize = 9; lbl.TextWrapped = true; lbl.TextXAlignment = Enum.TextXAlignment.Center; lbl.TextYAlignment = Enum.TextYAlignment.Center; lbl.ZIndex = 56

        btn.MouseButton1Down:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.05), {BackgroundColor3=Theme.ElementHover}):Play()
            TweenService:Create(bs, TweenInfo.new(0.05), {Transparency=0}):Play()
        end)
        btn.MouseButton1Up:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.10), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.10), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseEnter:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.ElementHover}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.4}):Play()
            end
        end)
        btn.MouseLeave:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseButton1Click:Connect(function() if onClick then pcall(onClick) end end)

        btn:SetAttribute("MB_On", false)
        btn.AttributeChanged:Connect(function(a)
            if a == "MB_On" then
                local on = btn:GetAttribute("MB_On")
                if on then
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Accent}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(5,5,15)}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0, Color = Theme.AccentGlow}):Play()
                    bs.Thickness = 2
                else
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Element}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Theme.Accent}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0.7, Color = Theme.Accent}):Play()
                    bs.Thickness = 1
                end
            end
        end)
        table.insert(MobileButtons.BtnsObjects, btn)
        return btn
    end

    local alMbBtn = makeMobileBtn("AUTO\nLEFT", 1, 1, function()
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoLeftEnabled = not State.autoLeftEnabled
        if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end
        if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoLeft = alMbBtn

    local arMbBtn = makeMobileBtn("AUTO\nRIGHT", 1, 2, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoRightEnabled = not State.autoRightEnabled
        if setAutoRight then setAutoRight(State.autoRightEnabled) end
        if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoRight = arMbBtn

    local batMbBtn = makeMobileBtn("BAT", 2, 1, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        State.autoBatToggled = not State.autoBatToggled
        if not State.autoBatToggled then resetBend() end
        if setAutoBat then setAutoBat(State.autoBatToggled) end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoBat = batMbBtn

    makeMobileBtn("DROP", 2, 2, function() runDropBrainrot() end)

    local carryMbBtn = makeMobileBtn("CARRY\nSPD", 3, 1, function() toggleSpeedType() end)
    MobileButtons.BtnRefs.carrySpeed = carryMbBtn

    local lagMbBtn = makeMobileBtn("LAG", 3, 2, function() toggleLagger() end)
    MobileButtons.BtnRefs.lagger = lagMbBtn

    makeMobileBtn("TP\nDOWN", 4, 1, function() tpToGround() end)

    local guiMbBtn = makeMobileBtn("GUI", 4, 2, function() if toggleGuiVis then toggleGuiVis() end end)
    MobileButtons.HideGuiBtn = guiMbBtn

    task.spawn(function()
        while task.wait(0.2) do
            pcall(function()
                if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
                if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
                if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
                if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
                if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
            end)
        end
    end)

    task.spawn(function()
        task.wait(0.1)
        if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
        if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
        if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
        if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
        if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
    end)

    for _, btn in pairs(MobileButtons.BtnsObjects) do btn.Visible = MobileButtons.Visible end
    mbPanel.Visible = MobileButtons.Visible
end

local function repositionMbButtons()
    if not mbPanel then return end
    local idx = 0
    for _, child in ipairs(mbPanel:GetChildren()) do
        if child:IsA("TextButton") then
            local row = math.floor(idx / MB_COLS) + 1
            local col = (idx % MB_COLS) + 1
            local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
            local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
            child.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
            child.Position = UDim2.new(0, x, 0, y)
            idx = idx + 1
        end
    end
end

local function applyMbBtnSize(w, h2)
    w = math.clamp(w, 40, 120); h2 = math.clamp(h2, 36, 120)
    MB_BTN_W = w; MB_BTN_H = h2
    MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
    MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
    if mbPanel then mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H); repositionMbButtons() end
end

-- ========== ANIMATIONS ==========
local Anims = {idle1="rbxassetid://133806214992291",idle2="rbxassetid://94970088341563",walk="rbxassetid://707897309",run="rbxassetid://707861613",jump="rbxassetid://116936326516985",fall="rbxassetid://116936326516985",climb="rbxassetid://116936326516985",swim="rbxassetid://116936326516985",swimidle="rbxassetid://116936326516985"}
task.spawn(function() pcall(function() ContentProvider:PreloadAsync({Anims.idle1,Anims.idle2,Anims.walk,Anims.run,Anims.jump,Anims.fall,Anims.climb,Anims.swim,Anims.swimidle}) end) end)
local animHeartbeatConn=nil; local savedAnimate=nil; local originalAnims=nil
local function isPackAnim(id) if not id then return false end; for _,v in pairs(Anims) do if v==id then return true end end; return false end
local function saveOriginalAnims(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function g(obj) return obj and obj.AnimationId or nil end; local ids={idle1=g(animate.idle and animate.idle.Animation1),idle2=g(animate.idle and animate.idle.Animation2),walk=g(animate.walk and animate.walk.WalkAnim),run=g(animate.run and animate.run.RunAnim),jump=g(animate.jump and animate.jump.JumpAnim),fall=g(animate.fall and animate.fall.FallAnim),climb=g(animate.climb and animate.climb.ClimbAnim),swim=g(animate.swim and animate.swim.Swim),swimidle=g(animate.swimidle and animate.swimidle.SwimIdle)}; if not isPackAnim(ids.walk) then originalAnims=ids end end
local function applyAnimPack(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,Anims.idle1); s(animate.idle and animate.idle.Animation2,Anims.idle2); s(animate.walk and animate.walk.WalkAnim,Anims.walk); s(animate.run and animate.run.RunAnim,Anims.run); s(animate.jump and animate.jump.JumpAnim,Anims.jump); s(animate.fall and animate.fall.FallAnim,Anims.fall); s(animate.climb and animate.climb.ClimbAnim,Anims.climb); s(animate.swim and animate.swim.Swim,Anims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,Anims.swimidle) end
local function restoreOriginalAnims(char) if not originalAnims then return end; local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj and id then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,originalAnims.idle1); s(animate.idle and animate.idle.Animation2,originalAnims.idle2); s(animate.walk and animate.walk.WalkAnim,originalAnims.walk); s(animate.run and animate.run.RunAnim,originalAnims.run); s(animate.jump and animate.jump.JumpAnim,originalAnims.jump); s(animate.fall and animate.fall.FallAnim,originalAnims.fall); s(animate.climb and animate.climb.ClimbAnim,originalAnims.climb); s(animate.swim and animate.swim.Swim,originalAnims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,originalAnims.swimidle); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end
local function startAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then saveOriginalAnims(char); applyAnimPack(char); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end; animHeartbeatConn=RunService.Heartbeat:Connect(function() if not State.animEnabled then return end; local c=LP.Character; if c then applyAnimPack(c) end end) end
local function stopAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then restoreOriginalAnims(char) end end
local function startUnwalk() if State.unwalkEnabled then return end; State.unwalkEnabled=true; local c=LP.Character; if not c then return end; local hum=c:FindFirstChildOfClass("Humanoid"); if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end; local anim=c:FindFirstChild("Animate"); if anim then savedAnimate=anim:Clone(); anim:Destroy() end end
local function stopUnwalk() if not State.unwalkEnabled then return end; State.unwalkEnabled=false; local c=LP.Character; if c and savedAnimate then savedAnimate.Parent=c; savedAnimate.Disabled=false; savedAnimate=nil end; task.spawn(function() task.wait(0.15); local char=LP.Character; if not char then return end; if State.animEnabled then saveOriginalAnims(char); applyAnimPack(char) else restoreOriginalAnims(char) end end) end

-- ========== MAIN GUI ==========
local gui = Instance.new("ScreenGui")
gui.Name="OmniHubGUI"; gui.ResetOnSpawn=false; gui.DisplayOrder=10; gui.IgnoreGuiInset=true; gui.Parent=LP:WaitForChild("PlayerGui")

local stealProgressBar = Instance.new("Frame", gui)
stealProgressBar.Size=UDim2.new(0,280,0,6); stealProgressBar.Position=UDim2.new(0.5,-140,0.9,0)
stealProgressBar.BackgroundColor3=Color3.fromRGB(8,12,25); stealProgressBar.BackgroundTransparency=0; stealProgressBar.BorderSizePixel=0; stealProgressBar.ZIndex=100
Instance.new("UICorner",stealProgressBar).CornerRadius=UDim.new(1,0)
local barStroke=Instance.new("UIStroke",stealProgressBar); barStroke.Color=Theme.AccentDim; barStroke.Thickness=1
local barFill=Instance.new("Frame",stealProgressBar); barFill.Size=UDim2.new(0,0,1,0); barFill.BackgroundColor3=Theme.Accent; barFill.BorderSizePixel=0
Instance.new("UICorner",barFill).CornerRadius=UDim.new(1,0); AutoSteal.ProgressFill=barFill

local function makeStealBarDraggable(frame)
    local dragging=false; local dragInput,dragStart,startPos,activeDragConn
    local function stopDrag() dragging=false; dragInput=nil; dragStart=nil; startPos=nil; if activeDragConn then activeDragConn:Disconnect(); activeDragConn=nil end end
    frame.InputBegan:Connect(function(input) if MobileButtons.Locked then return end; if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=input.Position; startPos=frame.Position; input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End or input.UserInputState==Enum.UserInputState.Cancelled then stopDrag() end end) end end)
    frame.InputChanged:Connect(function(input) if not dragging then return end; if MobileButtons.Locked then stopDrag(); return end; if input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch then dragInput=input end end)
    UIS.InputChanged:Connect(function(input) if input==dragInput and dragging then if MobileButtons.Locked then stopDrag(); return end; local delta=input.Position-dragStart; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y) end end)
end
makeStealBarDraggable(stealProgressBar)
local function saveStealBarPosition() local pos=stealProgressBar.Position; pcall(function() writefile("JispiStealBarPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function loadStealBarPosition() local savedPos=nil; pcall(function() savedPos=readfile("JispiStealBarPos.txt") end); if savedPos and savedPos~="" then local parts={}; for part in string.gmatch(savedPos,"[^,]+") do table.insert(parts,part) end; if #parts>=4 then stealProgressBar.Position=UDim2.new(tonumber(parts[1]),tonumber(parts[2]),tonumber(parts[3]),tonumber(parts[4])) end end end
loadStealBarPosition()
local lastStealBarSave=0; stealProgressBar:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastStealBarSave>0.5 then lastStealBarSave=tick(); saveStealBarPosition() end end)

-- ========== MAIN FRAME ==========
main = Instance.new("Frame", gui)
main.Name="Main"; main.Size=UDim2.new(0,320,0,470); main.Position=UDim2.new(0,20,0,20)
main.BackgroundColor3=Theme.Background; main.BorderSizePixel=0; main.Active=true; main.ClipsDescendants=true
Instance.new("UICorner",main).CornerRadius=UDim.new(0,16)
local outerGlow=Instance.new("UIStroke",main); outerGlow.Color=Theme.Accent; outerGlow.Thickness=1.5; outerGlow.Transparency=0.3
local glowPulse=Instance.new("Frame",main); glowPulse.Size=UDim2.new(1,8,1,8); glowPulse.Position=UDim2.new(0,-4,0,-4)
glowPulse.BackgroundColor3=Theme.Accent; glowPulse.BackgroundTransparency=0.88; glowPulse.BorderSizePixel=0; glowPulse.ZIndex=-1
Instance.new("UICorner",glowPulse).CornerRadius=UDim.new(0,20)
task.spawn(function()
    while true do
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.78}):Play()
        task.wait(1.8)
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.92}):Play()
        task.wait(1.8)
    end
end)

local function saveMainPosition() local pos=main.Position; pcall(function() writefile("JispiHubGUIPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function saveMiniPosition() if miniBtn then local pos=miniBtn.Position; pcall(function() writefile("JispiMiniPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end end
local function loadMainPosition() local sp=nil; pcall(function() sp=readfile("JispiHubGUIPos.txt") end); if sp and sp~="" then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then main.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])); if closeBtn then closeBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2])+320-34,tonumber(p[3]),tonumber(p[4])+18) end end end end
local function loadMiniPosition() local sp=nil; pcall(function() sp=readfile("JispiMiniPos.txt") end); if sp and sp~="" and miniBtn then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then miniBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])) end end end

-- ========== HEADER ==========
local header=Instance.new("Frame",main)
header.Size=UDim2.new(1,0,0,52); header.BackgroundColor3=Theme.Header; header.BorderSizePixel=0; header.ZIndex=5
Instance.new("UICorner",header).CornerRadius=UDim.new(0,16)
local headerAccentLine=Instance.new("Frame",header)
headerAccentLine.Size=UDim2.new(1,0,0,2); headerAccentLine.Position=UDim2.new(0,0,1,-2)
headerAccentLine.BackgroundColor3=Theme.Accent; headerAccentLine.BorderSizePixel=0; headerAccentLine.ZIndex=6
local lineGlow=Instance.new("UIGradient",headerAccentLine)
lineGlow.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,30)),ColorSequenceKeypoint.new(0.5,Theme.AccentGlow),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,30))})
local titleLbl=Instance.new("TextLabel",header)
titleLbl.Size=UDim2.new(1,0,1,-4); titleLbl.Position=UDim2.new(0,0,0,0)
titleLbl.BackgroundTransparency=1; titleLbl.Text="â  RBG HUB  â"
titleLbl.TextColor3=Theme.AccentGlow; titleLbl.Font=Theme.FontBlack; titleLbl.TextSize=17
titleLbl.TextXAlignment=Enum.TextXAlignment.Center; titleLbl.ZIndex=6
local versionLbl=Instance.new("TextLabel",header)
versionLbl.Size=UDim2.new(0,40,0,16); versionLbl.Position=UDim2.new(1,-50,0.5,-8)
versionLbl.BackgroundTransparency=1; versionLbl.Text="v1.0"; versionLbl.TextColor3=Theme.AccentDim
versionLbl.Font=Theme.Font; versionLbl.TextSize=9; versionLbl.ZIndex=6

-- ========== CLOSE / MINI ==========
closeBtn=Instance.new("TextButton",gui)
closeBtn.Size=UDim2.new(0,28,0,28); closeBtn.BackgroundColor3=Color3.fromRGB(14,18,35)
closeBtn.BorderSizePixel=0; closeBtn.Text="â"; closeBtn.TextColor3=Theme.SubText
closeBtn.Font=Theme.FontBold; closeBtn.TextSize=14; closeBtn.ZIndex=50
Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(0,8)
local closeStroke=Instance.new("UIStroke",closeBtn); closeStroke.Color=Theme.Border; closeStroke.Thickness=1.5
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Danger,TextColor3=Color3.new(1,1,1)}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Danger}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Color3.fromRGB(14,18,35),TextColor3=Theme.SubText}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Border}):Play() end)
closeBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)

miniBtn=Instance.new("ImageButton",gui); miniBtn.Name="OmniMiniButton"; miniBtn.Size=UDim2.new(0,44,0,44)
miniBtn.Position=UDim2.new(0,20,0,100); miniBtn.BackgroundColor3=Theme.Background; miniBtn.Image=""; miniBtn.BorderSizePixel=0; miniBtn.Visible=false; miniBtn.ZIndex=50
Instance.new("UICorner",miniBtn).CornerRadius=UDim.new(0,12)
local miniStroke=Instance.new("UIStroke",miniBtn); miniStroke.Color=Theme.Accent; miniStroke.Thickness=1.8
local miniIcon=Instance.new("TextLabel",miniBtn); miniIcon.Size=UDim2.new(1,0,1,0); miniIcon.Text="â"; miniIcon.TextColor3=Theme.AccentGlow; miniIcon.Font=Theme.FontBlack; miniIcon.TextSize=20; miniIcon.BackgroundTransparency=1
miniBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)
miniBtn.MouseEnter:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Element}):Play() end)
miniBtn.MouseLeave:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Background}):Play() end)

local function makeMainDraggable(frame)
    local dragging,dragInput,dragStart,startPos,startCloseBtnPos=false,nil,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; if closeBtn then startCloseBtnPos=closeBtn.Position end; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy); if closeBtn and startCloseBtnPos then closeBtn.Position=UDim2.new(startCloseBtnPos.X.Scale,startCloseBtnPos.X.Offset+dx,startCloseBtnPos.Y.Scale,startCloseBtnPos.Y.Offset+dy) end end end)
end
local function makeMiniDraggable(frame)
    local dragging,dragInput,dragStart,startPos=false,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy) end end)
end
makeMainDraggable(main); makeMiniDraggable(miniBtn)
local lastSaveTime=0; main:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastSaveTime>0.5 then lastSaveTime=tick(); saveMainPosition() end end)
local lastMiniSaveTime=0; miniBtn:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastMiniSaveTime>0.5 then lastMiniSaveTime=tick(); saveMiniPosition() end end)
local function updateCloseButtonPosition() if main.Visible then local p=main.Position; closeBtn.Position=UDim2.new(p.X.Scale,p.X.Offset+320-34,p.Y.Scale,p.Y.Offset+18) end end
main:GetPropertyChangedSignal("Position"):Connect(updateCloseButtonPosition); main:GetPropertyChangedSignal("Visible"):Connect(updateCloseButtonPosition)

-- ========== TAB SYSTEM ==========
local tabFrame=Instance.new("Frame",main)
tabFrame.Size=UDim2.new(1,-16,0,30); tabFrame.Position=UDim2.new(0,8,0,56)
tabFrame.BackgroundTransparency=1; tabFrame.BorderSizePixel=0; tabFrame.ZIndex=3
local tabLayout=Instance.new("UIListLayout",tabFrame)
tabLayout.FillDirection=Enum.FillDirection.Horizontal; tabLayout.SortOrder=Enum.SortOrder.LayoutOrder
tabLayout.Padding=UDim.new(0,4); tabLayout.HorizontalAlignment=Enum.HorizontalAlignment.Center

local scroll=Instance.new("ScrollingFrame",main)
scroll.Size=UDim2.new(1,-16,1,-94); scroll.Position=UDim2.new(0,8,0,90)
scroll.BackgroundTransparency=1; scroll.BorderSizePixel=0; scroll.ScrollBarThickness=3
scroll.ScrollBarImageColor3=Theme.Accent; scroll.ScrollBarImageTransparency=0.4
scroll.AutomaticCanvasSize=Enum.AutomaticSize.Y; scroll.CanvasSize=UDim2.new(0,0,0,0); scroll.ZIndex=2
local listLayout=Instance.new("UIListLayout",scroll); listLayout.SortOrder=Enum.SortOrder.LayoutOrder; listLayout.Padding=UDim.new(0,5)
local pad=Instance.new("UIPadding",scroll); pad.PaddingLeft=UDim.new(0,0); pad.PaddingRight=UDim.new(0,0); pad.PaddingTop=UDim.new(0,2); pad.PaddingBottom=UDim.new(0,8)

local tabDefs = {{"Speed",1},{"Features",2},{"Settings",3}}
local tabBtns = {}; local tabPages = {}; local activeTab = "Speed"

for _,td in ipairs(tabDefs) do
    local btn=Instance.new("TextButton",tabFrame)
    btn.Size=UDim2.new(0,80,0,26); btn.BackgroundColor3=(td[1]==activeTab) and Theme.AccentDark or Theme.Element
    btn.BorderSizePixel=0; btn.Text=td[1]:upper(); btn.TextColor3=(td[1]==activeTab) and Theme.AccentGlow or Theme.SubText
    btn.Font=Theme.FontBold; btn.TextSize=9; btn.LayoutOrder=td[2]; btn.ZIndex=4; btn.AutoButtonColor=false
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,8)
    local ts=Instance.new("UIStroke",btn); ts.Color=(td[1]==activeTab) and Theme.Accent or Theme.Border; ts.Thickness=(td[1]==activeTab) and 1.5 or 1
    tabBtns[td[1]]={btn=btn,stroke=ts}
    local page=Instance.new("Frame",scroll)
    page.Size=UDim2.new(1,0,0,0); page.BackgroundTransparency=1; page.BorderSizePixel=0
    page.Visible=(td[1]==activeTab); page.ZIndex=2
    page.AutomaticSize=Enum.AutomaticSize.Y
    local pageLL=Instance.new("UIListLayout",page); pageLL.SortOrder=Enum.SortOrder.LayoutOrder; pageLL.Padding=UDim.new(0,5)
    local pagePad=Instance.new("UIPadding",page); pagePad.PaddingBottom=UDim.new(0,4)
    tabPages[td[1]]=page
    local pageLO=0
    btn.MouseButton1Click:Connect(function()
        if activeTab==td[1] then return end
        activeTab=td[1]
        for name,data in pairs(tabBtns) do
            local isA=(name==activeTab)
            TweenService:Create(data.btn,TweenInfo.new(0.15),{BackgroundColor3=isA and Theme.AccentDark or Theme.Element}):Play()
            TweenService:Create(data.btn,TweenInfo.new(0.15),{TextColor3=isA and Theme.AccentGlow or Theme.SubText}):Play()
            TweenService:Create(data.stroke,TweenInfo.new(0.15),{Color=isA and Theme.Accent or Theme.Border,Thickness=isA and 1.5 or 1}):Play()
            tabPages[name].Visible=isA
        end
    end)
    btn.MouseEnter:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end
    end)
    btn.MouseLeave:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.Element}):Play() end
    end)
end

-- ========== UI HELPERS (per-page) ==========
local pageLOs = {Speed=0, Features=0, Settings=0}
local function LO(tab) pageLOs[tab]=(pageLOs[tab] or 0)+1; return pageLOs[tab] end
local function pg(tab) return tabPages[tab] end

local function makeGap(tab,px) local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,0,0,px or 3); f.BackgroundTransparency=1; f.LayoutOrder=LO(tab) end
local function makeDivider(tab)
    local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,-8,0,1); f.BackgroundColor3=Theme.Border; f.BackgroundTransparency=0; f.BorderSizePixel=0; f.LayoutOrder=LO(tab)
    local g=Instance.new("UIGradient",f); g.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,20)),ColorSequenceKeypoint.new(0.5,Theme.Accent),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,20))})
end
local function makeSectionLabel(tab,text)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,26); row.BackgroundTransparency=1; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-8,1,0); lbl.BackgroundTransparency=1
    lbl.Text="  â "..text:upper(); lbl.TextColor3=Theme.Accent; lbl.Font=Theme.FontBold; lbl.TextSize=10; lbl.TextXAlignment=Enum.TextXAlignment.Left
end

local function makeInputRow(tab,label,default,onChange)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0,100,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.ZIndex=2
    local box=Instance.new("TextBox",row); box.Size=UDim2.new(0,60,0,26); box.Position=UDim2.new(1,-70,0.5,-13); box.BackgroundColor3=Theme.BackgroundSecondary; box.BorderSizePixel=0; box.Text=tostring(default); box.TextColor3=Theme.AccentGlow; box.Font=Theme.FontBold; box.TextSize=13; box.ClearTextOnFocus=false; box.ZIndex=3
    Instance.new("UICorner",box).CornerRadius=UDim.new(0,8)
    local boxStroke=Instance.new("UIStroke",box); boxStroke.Color=Theme.AccentDim; boxStroke.Thickness=1
    box.Focused:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.Element}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
    box.FocusLost:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.BackgroundSecondary}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); local num=tonumber(box.Text); if num~=nil then local fv=math.floor(math.clamp(num,0,500)); box.Text=tostring(fv); if onChange then onChange(tostring(fv)) end; autoSaveConfig() else box.Text=tostring(default) end end)
    return box
end

local function makeStatusRow(tab,label,valTxt)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,32); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0.5,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local val=Instance.new("TextLabel",row); val.Size=UDim2.new(0.45,-10,1,0); val.Position=UDim2.new(0.52,0,0,0); val.BackgroundTransparency=1; val.Text=valTxt; val.TextColor3=Theme.AccentGlow; val.Font=Theme.FontBold; val.TextSize=12; val.TextXAlignment=Enum.TextXAlignment.Right
    return val
end

local function makeKeybindRow(tab,label,currentKey,onChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local btn=Instance.new("TextButton",row); btn.Size=UDim2.new(0,70,0,24); btn.Position=UDim2.new(1,-80,0.5,-12); btn.BackgroundColor3=Theme.Keybind; btn.BorderSizePixel=0; btn.Text="["..currentKey.Name.."]"; btn.TextColor3=Theme.Accent; btn.Font=Theme.FontBold; btn.TextSize=10
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,7)
    local bs=Instance.new("UIStroke",btn); bs.Color=Theme.AccentDim; bs.Thickness=1
    local listening=false; local listenConn
    local function stopListen(key)
        listening=false; if listenConn then listenConn:Disconnect(); listenConn=nil end
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(btn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); btn.TextColor3=Theme.Accent
        if key then btn.Text="["..key.Name.."]"; if onChanged then onChanged(key) end; autoSaveConfig() end
    end
    btn.MouseButton1Click:Connect(function()
        if listening then stopListen(nil); return end
        listening=true; btn.Text="[Â·Â·Â·]"; btn.TextColor3=Theme.AccentGlow; btn.BackgroundColor3=Theme.Element
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
        listenConn=UIS.InputBegan:Connect(function(inp,gp)
            if not listening then return end
            if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode)
            elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode) end
        end)
    end)
    return btn
end

local function makeToggleRow(tab,label,defaultKey,defaultOn,onToggle,onKeyChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,40); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-120,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local keyBtn=nil
    if defaultKey then
        keyBtn=Instance.new("TextButton",row); keyBtn.Size=UDim2.new(0,50,0,22); keyBtn.Position=UDim2.new(1,-108,0.5,-11); keyBtn.BackgroundColor3=Theme.Keybind; keyBtn.BorderSizePixel=0; keyBtn.Text=defaultKey.Name; keyBtn.TextColor3=Theme.Accent; keyBtn.Font=Theme.FontBold; keyBtn.TextSize=9; keyBtn.ZIndex=5
        Instance.new("UICorner",keyBtn).CornerRadius=UDim.new(0,5)
        local ks=Instance.new("UIStroke",keyBtn); ks.Color=Theme.AccentDim; ks.Thickness=1
        local kListening=false; local kConn
        local function kStop(key) kListening=false; if kConn then kConn:Disconnect(); kConn=nil end; TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(keyBtn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); keyBtn.TextColor3=Theme.Accent; if key then keyBtn.Text=key.Name; if onKeyChanged then onKeyChanged(key) end; autoSaveConfig() end end
        keyBtn.MouseButton1Click:Connect(function()
            if kListening then kStop(nil); return end
            kListening=true; keyBtn.Text="Â·Â·Â·"; keyBtn.TextColor3=Theme.AccentGlow; keyBtn.BackgroundColor3=Theme.Element
            TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
            kConn=UIS.InputBegan:Connect(function(inp,gp)
                if not kListening then return end
                if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode)
                elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode) end
            end)
        end)
    end
    local toggleBg=Instance.new("TextButton",row); toggleBg.Size=UDim2.new(0,42,0,22); toggleBg.Position=UDim2.new(1,-48,0.5,-11)
    toggleBg.BackgroundColor3=defaultOn and Theme.AccentDark or Theme.ToggleOff; toggleBg.BorderSizePixel=0; toggleBg.Text=""; toggleBg.ZIndex=5; toggleBg.AutoButtonColor=false
    Instance.new("UICorner",toggleBg).CornerRadius=UDim.new(1,0)
    local tStroke=Instance.new("UIStroke",toggleBg); tStroke.Color=defaultOn and Theme.Accent or Theme.Border; tStroke.Thickness=1.5
    local knob=Instance.new("Frame",toggleBg); knob.Size=UDim2.new(0,17,0,17); knob.Position=defaultOn and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5)
    knob.BackgroundColor3=defaultOn and Theme.AccentGlow or Theme.SubText; knob.BorderSizePixel=0; knob.ZIndex=6
    Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
    local isOn=defaultOn or false
    local function setV(on)
        isOn=on
        TweenService:Create(toggleBg,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{BackgroundColor3=on and Theme.AccentDark or Theme.ToggleOff}):Play()
        TweenService:Create(tStroke,TweenInfo.new(0.22),{Color=on and Theme.Accent or Theme.Border}):Play()
        TweenService:Create(knob,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{Position=on and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=on and Theme.AccentGlow or Theme.SubText}):Play()
        TweenService:Create(rowStroke,TweenInfo.new(0.22),{Color=on and Theme.AccentDim or Theme.Border,Thickness=on and 1.5 or 1}):Play()
    end
    toggleBg.MouseButton1Click:Connect(function() isOn=not isOn; setV(isOn); if onToggle then pcall(onToggle,isOn) end; autoSaveConfig() end)
    toggleBg.MouseEnter:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
    toggleBg.MouseLeave:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)
    return setV, keyBtn
end

-- ===================== BUILD TABS =====================

-- ===== SPEED TAB =====
makeSectionLabel("Speed","Speed")
normalBox=makeInputRow("Speed","Normal",State.normalSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.normalSpeed=n end end)
carryBox=makeInputRow("Speed","Carry",State.carrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.carrySpeed=n end end)
setSpeedToggleUI,speedKeyBtn=makeToggleRow("Speed","Toggle",Keys.speed,false,function(on) toggleSpeedType() end,function(k) Keys.speed=k end)
modeValLbl=makeStatusRow("Speed","Mode","Normal")
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Lagger Speed")
laggerBox=makeInputRow("Speed","Normal",State.laggerSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerSpeed=n end end)
carryLaggerBox=makeInputRow("Speed","Carry",State.laggerCarrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerCarrySpeed=n end end)
setLaggerToggleUI,laggerKeyBtn=makeToggleRow("Speed","Toggle",Keys.lagger,false,function(on) toggleLagger() end,function(k) Keys.lagger=k end)
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Combat")
setAutoBat,autoBatKeyBtn=makeToggleRow("Speed","Bat Aimbot",Keys.autoBat,false,function(on) State.autoBatToggled=on; if not on then resetBend() end end,function(k) Keys.autoBat=k end)
makeGap("Speed",4)

-- ===== FEATURES TAB =====
makeSectionLabel("Features","Auto Steal")
local radiusBox=makeInputRow("Features","Grab Rad",AutoSteal.Radius,function(v) local n=tonumber(v); if n and n>=5 and n<=300 then AutoSteal.Radius=math.floor(n); if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end end)
setInstaGrab=makeToggleRow("Features","Auto Grab",nil,false,function(on) AutoSteal.Enabled=on; if on then startAutoSteal() else stopAutoSteal() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Active Toggles")
setInfJump=makeToggleRow("Features","Infinite Jump",nil,false,function(on) State.infJumpEnabled=on end)
setAntiRag=makeToggleRow("Features","Anti Ragdoll",nil,false,function(on) State.antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
setFps=makeToggleRow("Features","FPS Boost",nil,false,function(on) State.fpsBoostEnabled=on; if on then pcall(applyFPSBoost) end end)
setMedusaCounter=makeToggleRow("Features","Medusa Counter",nil,false,function(on) State.medusaCounterEnabled=on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
setAnimToggle=makeToggleRow("Features","Tryhard Anim",nil,false,function(on) State.animEnabled=on; if on then startAnimToggle() else stopAnimToggle() end end)
setUnwalkToggle=makeToggleRow("Features","Unwalk",nil,false,function(on) if on then startUnwalk() else stopUnwalk() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Movement")
tpDownKeyBtn=makeKeybindRow("Features","TP Down",Keys.tpDown,function(k) Keys.tpDown=k end)
dropBrainrotKeyBtn=makeKeybindRow("Features","Drop Brainrot",Keys.dropBrainrot,function(k) Keys.dropBrainrot=k end)
setAutoLeft,autoLeftKeyBtn=makeToggleRow("Features","Auto Left",Keys.autoLeft,false,function(on) State.autoLeftEnabled=on; if on then startAutoLeft() else stopAutoLeft() end end,function(k) Keys.autoLeft=k end)
setAutoRight,autoRightKeyBtn=makeToggleRow("Features","Auto Right",Keys.autoRight,false,function(on) State.autoRightEnabled=on; if on then startAutoRight() else stopAutoRight() end end,function(k) Keys.autoRight=k end)
makeGap("Features",4)

-- ===== SETTINGS TAB =====
makeSectionLabel("Settings","Interface")
guiHideKeyBtn=makeKeybindRow("Settings","Hide GUI",Keys.guiHide,function(k) Keys.guiHide=k end)
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

makeSectionLabel("Settings","Mobile Panel")
local showRow=Instance.new("Frame",pg("Settings")); showRow.Size=UDim2.new(1,0,0,38); showRow.BackgroundColor3=Theme.Element; showRow.BorderSizePixel=0; showRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",showRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",showRow).Color=Theme.Border
local showLbl=Instance.new("TextLabel",showRow); showLbl.Size=UDim2.new(1,-115,1,0); showLbl.Position=UDim2.new(0,12,0,0); showLbl.BackgroundTransparency=1; showLbl.Text="Show Buttons"; showLbl.TextColor3=Theme.Text; showLbl.Font=Theme.FontMedium; showLbl.TextSize=12; showLbl.TextXAlignment=Enum.TextXAlignment.Left
local showToggleBg=Instance.new("TextButton",showRow); showToggleBg.Size=UDim2.new(0,42,0,22); showToggleBg.Position=UDim2.new(1,-48,0.5,-11); showToggleBg.BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff; showToggleBg.BorderSizePixel=0; showToggleBg.Text=""; showToggleBg.ZIndex=5; showToggleBg.AutoButtonColor=false
Instance.new("UICorner",showToggleBg).CornerRadius=UDim.new(1,0)
local showKnob=Instance.new("Frame",showToggleBg); showKnob.Size=UDim2.new(0,17,0,17); showKnob.Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); showKnob.BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText; showKnob.BorderSizePixel=0; showKnob.ZIndex=6; Instance.new("UICorner",showKnob).CornerRadius=UDim.new(1,0)
showToggleBg.MouseButton1Click:Connect(function() MobileButtons.Visible=not MobileButtons.Visible; if mbPanel then mbPanel.Visible=MobileButtons.Visible end; TweenService:Create(showToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(showKnob,TweenInfo.new(0.22),{Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
showToggleBg.MouseEnter:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
showToggleBg.MouseLeave:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local lockRow=Instance.new("Frame",pg("Settings")); lockRow.Size=UDim2.new(1,0,0,38); lockRow.BackgroundColor3=Theme.Element; lockRow.BorderSizePixel=0; lockRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",lockRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",lockRow).Color=Theme.Border
local lockLbl=Instance.new("TextLabel",lockRow); lockLbl.Size=UDim2.new(1,-115,1,0); lockLbl.Position=UDim2.new(0,12,0,0); lockLbl.BackgroundTransparency=1; lockLbl.Text="Lock Buttons"; lockLbl.TextColor3=Theme.Text; lockLbl.Font=Theme.FontMedium; lockLbl.TextSize=12; lockLbl.TextXAlignment=Enum.TextXAlignment.Left
local lockToggleBg=Instance.new("TextButton",lockRow); lockToggleBg.Size=UDim2.new(0,42,0,22); lockToggleBg.Position=UDim2.new(1,-48,0.5,-11); lockToggleBg.BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff; lockToggleBg.BorderSizePixel=0; lockToggleBg.Text=""; lockToggleBg.ZIndex=5; lockToggleBg.AutoButtonColor=false
Instance.new("UICorner",lockToggleBg).CornerRadius=UDim.new(1,0)
local lockKnob=Instance.new("Frame",lockToggleBg); lockKnob.Size=UDim2.new(0,17,0,17); lockKnob.Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); lockKnob.BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText; lockKnob.BorderSizePixel=0; lockKnob.ZIndex=6; Instance.new("UICorner",lockKnob).CornerRadius=UDim.new(1,0)
lockToggleBg.MouseButton1Click:Connect(function() MobileButtons.Locked=not MobileButtons.Locked; TweenService:Create(lockToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(lockKnob,TweenInfo.new(0.22),{Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
lockToggleBg.MouseEnter:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
lockToggleBg.MouseLeave:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local sizeRow=Instance.new("Frame",pg("Settings")); sizeRow.Size=UDim2.new(1,0,0,38); sizeRow.BackgroundColor3=Theme.Element; sizeRow.BorderSizePixel=0; sizeRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",sizeRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",sizeRow).Color=Theme.Border
local sizeLbl=Instance.new("TextLabel",sizeRow); sizeLbl.Size=UDim2.new(0,100,1,0); sizeLbl.Position=UDim2.new(0,12,0,0); sizeLbl.BackgroundTransparency=1; sizeLbl.Text="Btn Size"; sizeLbl.TextColor3=Theme.Text; sizeLbl.Font=Theme.FontMedium; sizeLbl.TextSize=12; sizeLbl.TextXAlignment=Enum.TextXAlignment.Left
local sizeValLbl=Instance.new("TextLabel",sizeRow); sizeValLbl.Size=UDim2.new(0,40,0,16); sizeValLbl.Position=UDim2.new(0,112,0.5,-8); sizeValLbl.BackgroundTransparency=1; sizeValLbl.Text=MB_BTN_W.."px"; sizeValLbl.TextColor3=Theme.AccentGlow; sizeValLbl.Font=Theme.FontBold; sizeValLbl.TextSize=9; sizeValLbl.TextXAlignment=Enum.TextXAlignment.Center
local decBtn=Instance.new("TextButton",sizeRow); decBtn.Size=UDim2.new(0,24,0,24); decBtn.Position=UDim2.new(1,-78,0.5,-12); decBtn.BackgroundColor3=Theme.BackgroundSecondary; decBtn.BorderSizePixel=0; decBtn.Text="-"; decBtn.TextColor3=Theme.Text; decBtn.Font=Theme.FontBlack; decBtn.TextSize=14; decBtn.ZIndex=5
Instance.new("UICorner",decBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",decBtn).Color=Theme.Border
local incBtn=Instance.new("TextButton",sizeRow); incBtn.Size=UDim2.new(0,24,0,24); incBtn.Position=UDim2.new(1,-48,0.5,-12); incBtn.BackgroundColor3=Theme.BackgroundSecondary; incBtn.BorderSizePixel=0; incBtn.Text="+"; incBtn.TextColor3=Theme.Text; incBtn.Font=Theme.FontBlack; incBtn.TextSize=14; incBtn.ZIndex=5
Instance.new("UICorner",incBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",incBtn).Color=Theme.Border
decBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W-4,MB_BTN_H-3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
incBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W+4,MB_BTN_H+3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
for _,b in pairs({decBtn,incBtn}) do
    b.MouseEnter:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
    b.MouseLeave:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
end
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

local saveConfigRow=Instance.new("Frame",pg("Settings")); saveConfigRow.Size=UDim2.new(1,0,0,40); saveConfigRow.BackgroundColor3=Theme.Element; saveConfigRow.BorderSizePixel=0; saveConfigRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",saveConfigRow).CornerRadius=UDim.new(0,10)
local saveConfigBtn=Instance.new("TextButton",saveConfigRow); saveConfigBtn.Size=UDim2.new(1,-16,0,28); saveConfigBtn.Position=UDim2.new(0,8,0.5,-14)
saveConfigBtn.BackgroundColor3=Color3.fromRGB(0,40,80); saveConfigBtn.BorderSizePixel=0; saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; saveConfigBtn.Font=Theme.FontBold; saveConfigBtn.TextSize=12; saveConfigBtn.ZIndex=5; saveConfigBtn.AutoButtonColor=false
Instance.new("UICorner",saveConfigBtn).CornerRadius=UDim.new(0,9)
local saveBtnStroke=Instance.new("UIStroke",saveConfigBtn); saveBtnStroke.Color=Theme.Accent; saveBtnStroke.Thickness=1.5
saveConfigBtn.MouseEnter:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,60,120)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.AccentGlow}):Play() end)
saveConfigBtn.MouseLeave:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,40,80)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
saveConfigBtn.MouseButton1Click:Connect(function() forceSaveConfig(); saveConfigBtn.Text="â  SAVED!"; saveConfigBtn.TextColor3=Theme.Success; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Success}):Play(); task.wait(1.2); saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)

local resetMbRow=Instance.new("Frame",pg("Settings")); resetMbRow.Size=UDim2.new(1,0,0,36); resetMbRow.BackgroundColor3=Theme.Element; resetMbRow.BorderSizePixel=0; resetMbRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",resetMbRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",resetMbRow).Color=Theme.Border
local resetMbBtn=Instance.new("TextButton",resetMbRow); resetMbBtn.Size=UDim2.new(1,-16,0,24); resetMbBtn.Position=UDim2.new(0,8,0.5,-12)
resetMbBtn.BackgroundColor3=Theme.BackgroundSecondary; resetMbBtn.BorderSizePixel=0; resetMbBtn.Text="Reset Button Panel Position"; resetMbBtn.TextColor3=Theme.SubText; resetMbBtn.Font=Theme.FontMedium; resetMbBtn.TextSize=10; resetMbBtn.ZIndex=5; resetMbBtn.AutoButtonColor=false
Instance.new("UICorner",resetMbBtn).CornerRadius=UDim.new(0,7)
resetMbBtn.MouseEnter:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
resetMbBtn.MouseLeave:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
resetMbBtn.MouseButton1Click:Connect(function() if mbPanel then mbPanel.Position=UDim2.new(1,-(MB_TOTAL_W+8),0.5,-(MB_TOTAL_H/2)) end end)

makeGap("Settings",2)
local footerLbl=Instance.new("TextLabel",pg("Settings")); footerLbl.Size=UDim2.new(1,0,0,18); footerLbl.BackgroundTransparency=1; footerLbl.LayoutOrder=LO("Settings"); footerLbl.Text="â RBG â"; footerLbl.TextColor3=Theme.AccentDim; footerLbl.Font=Theme.FontBold; footerLbl.TextSize=9; footerLbl.TextXAlignment=Enum.TextXAlignment.Center

-- ========== TOGGLE GUI VISIBILITY ==========
toggleGuiVis=function()
    State.guiVisible=not State.guiVisible
    if main then main.Visible=State.guiVisible; closeBtn.Visible=State.guiVisible; if miniBtn then miniBtn.Visible=not State.guiVisible end end
end

-- ========== LOGIC ==========
local function findMedusa()
    local char=LP.Character; if not char then return nil end
    for _,tool in ipairs(char:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end
    local bp2=LP:FindFirstChild("Backpack"); if bp2 then for _,tool in ipairs(bp2:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end end
    return nil
end
local function useMedusaCounter()
    if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<25 then return end; local char=LP.Character; if not char then return end
    State.medusaDebounce=true; local med=findMedusa(); if not med then State.medusaDebounce=false; return end
    if med.Parent~=char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
end
local function onAnchorChanged(part) return part:GetPropertyChangedSignal("Anchored"):Connect(function() if part.Anchored and part.Transparency==1 then useMedusaCounter() end end) end
setupMedusaCounter=function(char) stopMedusaCounter(); if not char then return end; for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end; table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end)) end
stopMedusaCounter=function() for _,c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end; Conns.anchor={} end

startAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect() end; State.autoLeftPhase=1
    Conns.autoLeft=RunService.Heartbeat:Connect(function()
        if not State.autoLeftEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoLeftPhase==1 then
            local tgt=Vector3.new(POS.L1.X,root.Position.Y,POS.L1.Z); if (tgt-root.Position).Magnitude<1 then State.autoLeftPhase=2; return end
            local d=(POS.L1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoLeftPhase==2 then
            local tgt=Vector3.new(POS.L2.X,root.Position.Y,POS.L2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoLeftEnabled=false; if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1; if setAutoLeft then setAutoLeft(false) end; return end
            local d=(POS.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect() end; State.autoRightPhase=1
    Conns.autoRight=RunService.Heartbeat:Connect(function()
        if not State.autoRightEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoRightPhase==1 then
            local tgt=Vector3.new(POS.R1.X,root.Position.Y,POS.R1.Z); if (tgt-root.Position).Magnitude<1 then State.autoRightPhase=2; return end
            local d=(POS.R1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoRightPhase==2 then
            local tgt=Vector3.new(POS.R2.X,root.Position.Y,POS.R2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoRightEnabled=false; if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1; if setAutoRight then setAutoRight(false) end; return end
            local d=(POS.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAntiRagdoll=function()
    if Conns.antiRag then return end
    Conns.antiRag=RunService.Heartbeat:Connect(function()
        local char=LP.Character; if not char then return end; local hum2=char:FindFirstChildOfClass("Humanoid"); local root=char:FindFirstChild("HumanoidRootPart")
        if hum2 then local st=hum2:GetState(); if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then hum2:ChangeState(Enum.HumanoidStateType.Running); workspace.CurrentCamera.CameraSubject=hum2; pcall(function() local pm=LP.PlayerScripts:FindFirstChild("PlayerModule"); if pm then require(pm:FindFirstChild("ControlModule")):Enable() end end); if root then root.Velocity=Vector3.new(0,0,0); root.RotVelocity=Vector3.new(0,0,0) end end end
        for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled=true end end
    end)
end
stopAntiRagdoll=function() if Conns.antiRag then Conns.antiRag:Disconnect(); Conns.antiRag=nil end end
applyFPSBoost=function()
    pcall(function() setfpscap(999999999) end)
    local function processObj(v) pcall(function() if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0 elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1 elseif v:IsA("SpecialMesh") then v.TextureId="" elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false elseif v:IsA("SurfaceAppearance") or v:IsA("MaterialVariant") then v:Destroy() elseif v:IsA("Attachment") then v.Visible=false end end) end
    for _,v in pairs(workspace:GetDescendants()) do processObj(v) end
    pcall(function() local lighting=game:GetService("Lighting"); for _,v in pairs(lighting:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end; pcall(function() sethiddenproperty(game:GetService("Lighting"),"Technology",Enum.Technology.Legacy) end); local lighting2=game:GetService("Lighting"); lighting2.GlobalShadows=false; lighting2.FogEnd=9e9; lighting2.Brightness=0; local terrain=workspace:FindFirstChildOfClass("Terrain"); if terrain then pcall(function() sethiddenproperty(terrain,"Decoration",false) end); terrain.WaterReflectance=0; terrain.WaterTransparency=0.7; terrain.WaterWaveSize=0; terrain.WaterWaveSpeed=0 end end)
    workspace.DescendantAdded:Connect(function(v) if State.fpsBoostEnabled then task.spawn(processObj,v) end end)
end

-- ========== CHARACTER SETUP ==========
local function setupChar(char)
    task.wait(0.1); h=char:WaitForChild("Humanoid",5); hrp=char:WaitForChild("HumanoidRootPart",5); State.originalC0=nil
    if not h or not hrp then return end
    local head=char:FindFirstChild("Head")
    if head then
        local oldBB=head:FindFirstChild("SpeedBillboard"); if oldBB then oldBB:Destroy() end
        local bb=Instance.new("BillboardGui",head); bb.Name="SpeedBillboard"; bb.Size=UDim2.new(0,120,0,22); bb.StudsOffset=Vector3.new(0,3,0); bb.AlwaysOnTop=true
        speedLbl=Instance.new("TextLabel",bb); speedLbl.Size=UDim2.new(1,0,1,0); speedLbl.BackgroundTransparency=1; speedLbl.TextColor3=Theme.AccentGlow; speedLbl.Font=Theme.FontBold; speedLbl.TextScaled=true; speedLbl.TextStrokeTransparency=0.5; speedLbl.TextStrokeColor3=Color3.fromRGB(0,20,40)
    end
    if State.antiRagdollEnabled and not Conns.antiRag then task.wait(0.5); startAntiRagdoll() end
    if State.medusaCounterEnabled then setupMedusaCounter(char) end
    if State.animEnabled then task.wait(0.3); saveOriginalAnims(char); applyAnimPack(char) end
    if State.unwalkEnabled then State.unwalkEnabled=false; task.wait(0.3); startUnwalk() end
end
LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

RunService.Stepped:Connect(function()
    for _,p in ipairs(Players:GetPlayers()) do if p~=LP and p.Character then for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end end end
end)

UIS.JumpRequest:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root then root.Velocity=Vector3.new(root.Velocity.X,55,root.Velocity.Z) end
end)
RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root and root.Velocity.Y<-120 then root.Velocity=Vector3.new(root.Velocity.X,-120,root.Velocity.Z) end
end)

RunService.RenderStepped:Connect(function()
    if not (h and hrp) then return end; if State._tpInProgress then return end
    if State.autoLeftEnabled or State.autoRightEnabled then if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end; return end
    local md=h.MoveDirection; local spd=getCurrentSpeed()
    if md.Magnitude>0 then State.lastMoveDir=md; hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
    elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude>0 then local anyHeld=false; for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true; break end end; if anyHeld then hrp.Velocity=Vector3.new(State.lastMoveDir.X*spd,hrp.Velocity.Y,State.lastMoveDir.Z*spd) end end
    if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end
end)

RunService.Heartbeat:Connect(function(dt)
    if not (State.autoBatToggled and h and hrp) then resetBend(); return end
    local target,dist=getClosestPlayer(); if not (target and target.Character) then resetBend(); return end
    local tr=target.Character:FindFirstChild("HumanoidRootPart"); if not tr then resetBend(); return end
    local rj=getRootJoint(); if rj and not State.originalC0 then State.originalC0=rj.C0 end
    local yDiff=tr.Position.Y-hrp.Position.Y; local yVel=math.clamp(yDiff*20,-120,120)
    if dist>6 then
        local dir=(tr.Position-hrp.Position).Unit; hrp.AssemblyLinearVelocity=Vector3.new(dir.X*59,yVel,dir.Z*59)
        if rj then rj.C0=State.originalC0*CFrame.Angles(math.rad(40),0,0) end
    else
        State.crazyAngle=State.crazyAngle+dt*40
        local offsetX=math.cos(State.crazyAngle)*0.8; local offsetZ=math.sin(State.crazyAngle)*0.8
        local targetPos=tr.Position+Vector3.new(offsetX,0,offsetZ); local flatDir=Vector3.new(targetPos.X-hrp.Position.X,0,targetPos.Z-hrp.Position.Z)
        hrp.AssemblyLinearVelocity=Vector3.new(flatDir.Unit.X*200,yVel,flatDir.Unit.Z*200)
        if rj then local t=tick(); local fwdBack=math.sin(t*25)*math.rad(50); local sideways=math.cos(t*20)*math.rad(30); rj.C0=State.originalC0*CFrame.Angles(fwdBack,0,sideways) end
        tryHitBat()
    end
end)

UIS.InputBegan:Connect(function(inp,gp)
    if gp then return end
    if inp.UserInputType~=Enum.UserInputType.Keyboard and inp.UserInputType~=Enum.UserInputType.Gamepad1 then return end
    local kc=inp.KeyCode
    if (State.autoLeftEnabled or State.autoRightEnabled) then if MOVE_KEYS[kc] then return end end
    if kc==Keys.speed then toggleSpeedType()
    elseif kc==Keys.lagger then toggleLagger()
    elseif kc==Keys.autoBat then State.autoBatToggled=not State.autoBatToggled; if not State.autoBatToggled then resetBend() end; setAutoBat(State.autoBatToggled); autoSaveConfig()
    elseif kc==Keys.autoLeft then State.autoLeftEnabled=not State.autoLeftEnabled; if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end; if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    elseif kc==Keys.autoRight then State.autoRightEnabled=not State.autoRightEnabled; if setAutoRight then setAutoRight(State.autoRightEnabled) end; if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    elseif kc==Keys.dropBrainrot then task.spawn(runDropBrainrot)
    elseif kc==Keys.tpDown then tpToGround()
    elseif kc==Keys.guiHide then toggleGuiVis() end
end)

task.spawn(function() while task.wait(0.5) do pcall(function() if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end) end end)

local function loadConfig()
    local hasFile=false; pcall(function() hasFile=isfile("JispiHubConfig.json") end); if not hasFile then return end
    local ok,cfg=pcall(function() return HttpService:JSONDecode(readfile("JispiHubConfig.json")) end); if not ok or not cfg then return end
    if cfg.normalSpeed and type(cfg.normalSpeed)=="number" then State.normalSpeed=cfg.normalSpeed; normalBox.Text=tostring(cfg.normalSpeed) end
    if cfg.carrySpeed  and type(cfg.carrySpeed)=="number"  then State.carrySpeed=cfg.carrySpeed;   carryBox.Text=tostring(cfg.carrySpeed)   end
    if cfg.laggerSpeed and type(cfg.laggerSpeed)=="number" then State.laggerSpeed=cfg.laggerSpeed; laggerBox.Text=tostring(cfg.laggerSpeed) end
    if cfg.laggerCarrySpeed and type(cfg.laggerCarrySpeed)=="number" then State.laggerCarrySpeed=cfg.laggerCarrySpeed; carryLaggerBox.Text=tostring(cfg.laggerCarrySpeed) end
    if cfg.speedType=="normal" or cfg.speedType=="carry" then State.speedType=cfg.speedType end
    if type(cfg.laggerActive)=="boolean" then State.laggerActive=cfg.laggerActive end
    if cfg.autoBatKey  and Enum.KeyCode[cfg.autoBatKey]    then Keys.autoBat=Enum.KeyCode[cfg.autoBatKey];   if autoBatKeyBtn  then autoBatKeyBtn.Text="["..cfg.autoBatKey.."]"   end end
    if cfg.speedKey    and Enum.KeyCode[cfg.speedKey]      then Keys.speed=Enum.KeyCode[cfg.speedKey]        end
    if cfg.laggerKey   and Enum.KeyCode[cfg.laggerKey]     then Keys.lagger=Enum.KeyCode[cfg.laggerKey]      end
    if cfg.autoLeftKey  and Enum.KeyCode[cfg.autoLeftKey]  then Keys.autoLeft=Enum.KeyCode[cfg.autoLeftKey];   if autoLeftKeyBtn  then autoLeftKeyBtn.Text="["..cfg.autoLeftKey.."]"   end end
    if cfg.autoRightKey and Enum.KeyCode[cfg.autoRightKey] then Keys.autoRight=Enum.KeyCode[cfg.autoRightKey]; if autoRightKeyBtn then autoRightKeyBtn.Text="["..cfg.autoRightKey.."]" end end
    if cfg.tpDownKey    and Enum.KeyCode[cfg.tpDownKey]    then Keys.tpDown=Enum.KeyCode[cfg.tpDownKey];       if tpDownKeyBtn    then tpDownKeyBtn.Text="["..cfg.tpDownKey.."]"        end end
    if cfg.grabRadius and type(cfg.grabRadius)=="number" then AutoSteal.Radius=cfg.grabRadius; if progressRadLbl then progressRadLbl.Text="Radius: "..cfg.grabRadius end end
    if cfg.autoStealEnabled then AutoSteal.Enabled=true; setInstaGrab(true); pcall(startAutoSteal) end
    if cfg.infJump      then State.infJumpEnabled=true;      setInfJump(true)      end
    if cfg.antiRagdoll  then State.antiRagdollEnabled=true;  setAntiRag(true);     startAntiRagdoll() end
    if cfg.fpsBoost     then State.fpsBoostEnabled=true;     setFps(true);         applyFPSBoost()    end
    if cfg.medusaCounter then State.medusaCounterEnabled=true; setMedusaCounter(true); setupMedusaCounter(LP.Character) end
    if cfg.dropBrainrotKey and Enum.KeyCode[cfg.dropBrainrotKey] then Keys.dropBrainrot=Enum.KeyCode[cfg.dropBrainrotKey]; if dropBrainrotKeyBtn then dropBrainrotKeyBtn.Text="["..cfg.dropBrainrotKey.."]"                    bat:Activate() -- Ø¨ÙØ®
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local ContentProvider = game:GetService("ContentProvider")
local LP = Players.LocalPlayer

local State = {                      2 localState = { 
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local ContentProvider = game:GetService("ContentProvider")
local LP = Players.LocalPlayer

local State = {
    normalSpeed = 60, carrySpeed = 30, laggerSpeed = 13, laggerCarrySpeed = 13,
    speedType = "normal", laggerActive = false, autoBatToggled = false,
    hittingCooldown = false, infJumpEnabled = false, antiRagdollEnabled = false,
    fpsBoostEnabled = false, guiVisible = true, isStealing = false,
    stealStartTime = nil, lastStealTick = 0, medusaLastUsed = 0,
    medusaDebounce = false, medusaCounterEnabled = false, dropBrainrotActive = false,
    autoLeftEnabled = false, autoRightEnabled = false, autoLeftPhase = 1,
    autoRightPhase = 1, _tpInProgress = false, lastMoveDir = Vector3.new(0,0,0),
    animEnabled = false, unwalkEnabled = false, originalC0 = nil, crazyAngle = 0,
}

local Keys = {
    autoBat = Enum.KeyCode.E, speed = Enum.KeyCode.Q, lagger = Enum.KeyCode.C,
    guiHide = Enum.KeyCode.LeftControl, autoLeft = Enum.KeyCode.L,
    autoRight = Enum.KeyCode.R, dropBrainrot = Enum.KeyCode.H, tpDown = Enum.KeyCode.T,
}

local main, closeBtn, miniBtn
local toggleGuiVis

local MobileButtons = { Visible = true, Locked = false, BtnsObjects = {}, BtnRefs = {}, Buttons = {}, HideGuiBtn = nil }

local MB_BTN_W = 58
local MB_BTN_H = 48
local MB_COLS = 2
local MB_ROWS = 4
local MB_PAD = 3
local MB_CORNER = 14
local MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
local MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
local mbPanel = nil

local Theme = {
    Background      = Color3.fromRGB(8, 10, 20),
    BackgroundSecondary = Color3.fromRGB(12, 15, 28),
    Header          = Color3.fromRGB(5, 8, 18),
    Element         = Color3.fromRGB(14, 18, 35),
    ElementHover    = Color3.fromRGB(20, 26, 50),
    Text            = Color3.fromRGB(200, 220, 255),
    SubText         = Color3.fromRGB(80, 110, 160),
    Accent          = Color3.fromRGB(0, 180, 255),
    AccentDark      = Color3.fromRGB(0, 100, 200),
    AccentGlow      = Color3.fromRGB(0, 220, 255),
    AccentDim       = Color3.fromRGB(0, 60, 120),
    Border          = Color3.fromRGB(0, 60, 120),
    BorderLight     = Color3.fromRGB(0, 100, 180),
    Success         = Color3.fromRGB(0, 255, 160),
    Danger          = Color3.fromRGB(255, 50, 80),
    Keybind         = Color3.fromRGB(10, 20, 45),
    KeybindText     = Color3.fromRGB(0, 180, 255),
    ToggleOff       = Color3.fromRGB(18, 22, 45),
    Font            = Enum.Font.Gotham,
    FontBold        = Enum.Font.GothamBold,
    FontBlack       = Enum.Font.GothamBlack,
    FontMedium      = Enum.Font.GothamMedium,
}

local AutoSteal = {
    Enabled = false, Radius = 20, Duration = 1.3, IsStealing = false,
    Data = {}, ProgressFill = nil, ProgressText = nil,
}

local function isMyPlotByName(plotName)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return false end
    local plot = plots:FindFirstChild(plotName); if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then local yb = sign:FindFirstChild("YourBase"); if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end end
    return false
end

local function findNearestPrompt()
    local char = LP.Character; local root = char and char:FindFirstChild("HumanoidRootPart"); if not root then return nil end
    local plots = workspace:FindFirstChild("Plots"); if not plots then return nil end
    local bestPrompt, bestDist, bestName = nil, math.huge, nil
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local podiums = plot:FindFirstChild("AnimalPodiums"); if not podiums then continue end
        for _, pod in ipairs(podiums:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base"); local spawn = base and base:FindFirstChild("Spawn")
                if spawn then
                    local dist = (spawn.Position - root.Position).Magnitude
                    if dist < bestDist and dist <= AutoSteal.Radius then
                        local att = spawn:FindFirstChild("PromptAttachment")
                        if att then for _, child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") then bestPrompt, bestDist, bestName = child, dist, pod.Name; break end
                        end end
                    end
                end
            end)
        end
    end
    return bestPrompt, bestDist, bestName
end

local function executeSteal(prompt, animalName)
    if AutoSteal.IsStealing then return end
    if not AutoSteal.Data[prompt] then
        AutoSteal.Data[prompt] = {hold = {}, trigger = {}, ready = true}
        pcall(function()
            if getconnections then
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(AutoSteal.Data[prompt].hold, c.Function) end end
                for _, c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(AutoSteal.Data[prompt].trigger, c.Function) end end
            end
        end)
    end
    local data = AutoSteal.Data[prompt]; if not data.ready then return end
    data.ready = false; AutoSteal.IsStealing = true; local startTime = tick()
    local conn; conn = RunService.Heartbeat:Connect(function()
        if not AutoSteal.IsStealing then conn:Disconnect(); return end
        local prog = math.clamp((tick() - startTime) / AutoSteal.Duration, 0, 1)
        if AutoSteal.ProgressFill then AutoSteal.ProgressFill.Size = UDim2.new(prog, 0, 1, 0) end
    end)
    task.spawn(function()
        for _, f in ipairs(data.hold) do task.spawn(f) end
        task.wait(AutoSteal.Duration)
        for _, f in ipairs(data.trigger) do task.spawn(f) end
        AutoSteal.IsStealing = false; data.ready = true
        task.wait(0.6)
        if not AutoSteal.IsStealing and AutoSteal.ProgressFill then
            TweenService:Create(AutoSteal.ProgressFill, TweenInfo.new(0.4), {Size = UDim2.new(0,0,1,0)}):Play()
        end
    end)
end

local autoStealConnection = nil
local function startAutoSteal()
    if autoStealConnection then return end
    autoStealConnection = RunService.Heartbeat:Connect(function()
        if AutoSteal.Enabled and not AutoSteal.IsStealing then
            local p, _, name = findNearestPrompt(); if p then executeSteal(p, name) end
        end
    end)
end
local function stopAutoSteal()
    if autoStealConnection then autoStealConnection:Disconnect(); autoStealConnection = nil end
    AutoSteal.IsStealing = false
    for k, v in pairs(AutoSteal.Data) do if v.ready ~= nil then v.ready = true end end
end

local MOVE_KEYS = {[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local DROP_ASCEND_DURATION = 0.2; local DROP_ASCEND_SPEED = 150
local POS = {L1=Vector3.new(-476.48,-6.28,92.73),L2=Vector3.new(-483.12,-4.95,94.80),R1=Vector3.new(-476.16,-6.52,25.62),R2=Vector3.new(-483.04,-5.09,23.14)}
local Conns = {autoSteal=nil,antiRag=nil,autoLeft=nil,autoRight=nil,anchor={},progress=nil}
local h, hrp, speedLbl
local setAutoLeft, setAutoRight
local setInstaGrab, setAutoBat, setInfJump, setAntiRag, setFps, setMedusaCounter
local setAnimToggle, setUnwalkToggle
local setupMedusaCounter, stopMedusaCounter, startAntiRagdoll, stopAntiRagdoll, applyFPSBoost
local modeValLbl, normalBox, carryBox, laggerBox, carryLaggerBox
local autoBatKeyBtn, speedKeyBtn, laggerKeyBtn, autoLeftKeyBtn, autoRightKeyBtn, guiHideKeyBtn, dropBrainrotKeyBtn, tpDownKeyBtn
local setSpeedToggleUI, setLaggerToggleUI
local progressRadLbl
local startAutoLeft, stopAutoLeft, startAutoRight, stopAutoRight

local function getRootJoint()
    local char2 = LP.Character; local torso = char2 and char2:FindFirstChild("LowerTorso")
    return torso and torso:FindFirstChild("Root")
end
local function resetBend()
    local rj = getRootJoint()
    if rj and State.originalC0 then rj.C0 = State.originalC0; State.originalC0 = nil end
end
local function getBat()
    local char = LP.Character; if not char then return nil end
    local tool = char:FindFirstChild("Bat"); if tool then return tool end
    local bp2 = LP:FindFirstChild("Backpack")
    if bp2 then tool = bp2:FindFirstChild("Bat"); if tool then tool.Parent = char; return tool end end
    return nil
end
local function tryHitBat()
    if State.hittingCooldown then return end; State.hittingCooldown = true
    pcall(function()
        local bat = getBat(); if bat then bat:Activate(); local ev = bat:FindFirstChildWhichIsA("RemoteEvent"); if ev then ev:FireServer() end end
    end)
    task.delay(0.08, function() State.hittingCooldown = false end)
end
local function getClosestPlayer()
    if not hrp then return nil, math.huge end
    local cp, cd = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if tr then local d = (hrp.Position - tr.Position).Magnitude; if d < cd then cd = d; cp = p end end
        end
    end
    return cp, cd
end

local saveDebounce = false
local function autoSaveConfig()
    if saveDebounce then return end; saveDebounce = true
    task.delay(0.5, function()
        local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
        pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end); saveDebounce = false
    end)
end
local function forceSaveConfig()
    local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
    pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end)
end
localsteal(steal) local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local Conns = {autoSteal=nil,antiRag=nil,anchor={},progress=nil,aimbot=nil,batCounter=nil}
local PLOT_CACHE_DURATION = 2
local PROMPT_CACHE_REFRESH = 0.15
local STEAL_COOLDOWN = 0.1
local MEDUSA_COOLDOWN = 25
local BAT_COUNTER_COOLDOWN = 0
local batCounterDebounce = false
local batCounterLastUsed = 0
local progressRadLbl,progressFill,progressPct
local modeValLbl
local _aimTarget = nil
local _aimLastScan = 0

local VYSE_AIMBOT_SPEED = 56.5
local VYSE_HIT_DIST = 5
local SWING_COOLDOWN = 0.08
local hittingCooldown = false

local desyncEnabled = false
local setDesyncVisual = nil
local desyncActive = false
local desyncSpeedConn = nil
local G_desyncAnimate = nil

local fpsBoostEnabled = false

local function refreshUIToggles()
    if setSpeedToggleUI then setSpeedToggleUI(State.speedType == "carry") end
    if setLaggerToggleUI then setLaggerToggleUI(State.laggerActive) end
    if modeValLbl then
        if State.laggerActive then modeValLbl.Text = (State.speedType=="normal") and "â¡ Lagger Normal" or "â¡ Lagger Carry"
        else modeValLbl.Text = (State.speedType=="normal") and "Normal" or "Carry" end
    end
end
local function toggleSpeedType()
    State.speedType = (State.speedType=="normal") and "carry" or "normal"; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.carrySpeed then MobileButtons.BtnRefs.carrySpeed:SetAttribute("MB_On", State.speedType=="carry") end
end
local function toggleLagger()
    State.laggerActive = not State.laggerActive; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.lagger then MobileButtons.BtnRefs.lagger:SetAttribute("MB_On", State.laggerActive) end
end
local function getCurrentSpeed()
    if State.laggerActive then return State.speedType=="normal" and State.laggerSpeed or State.laggerCarrySpeed
    else return State.speedType=="normal" and State.normalSpeed or State.carrySpeed end
end
local function getAutoMoveSpeed()
    return State.laggerActive and State.laggerSpeed or State.normalSpeed
end
local function tpToGround()
    local char = LP.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
    local rp = RaycastParams.new(); rp.FilterType = Enum.RaycastFilterType.Exclude; rp.FilterDescendantsInstances = {char}
    local rr = workspace:Raycast(root.Position, Vector3.new(0,-500,0), rp)
    if rr then root.CFrame = CFrame.new(rr.Position + Vector3.new(0,3,0)) else root.CFrame = root.CFrame * CFrame.new(0,-20,0) end
end
local function runDropBrainrot()
    if State.dropBrainrotActive then return end
    local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if not root then return end
    State.dropBrainrotActive=true; local t0=tick(); local dc
    dc=RunService.Heartbeat:Connect(function()
        local r=char and char:FindFirstChild("HumanoidRootPart"); if not r then dc:Disconnect(); State.dropBrainrotActive=false; return end
        if tick()-t0>=DROP_ASCEND_DURATION then
            dc:Disconnect()
            local rp=RaycastParams.new(); rp.FilterDescendantsInstances={char}; rp.FilterType=Enum.RaycastFilterType.Exclude
            local rr=workspace:Raycast(r.Position,Vector3.new(0,-2000,0),rp)
            if rr then local hum2=char:FindFirstChildOfClass("Humanoid"); local off=(hum2 and hum2.HipHeight or 2)+(r.Size.Y/2); r.CFrame=CFrame.new(r.Position.X,rr.Position.Y+off,r.Position.Z); r.AssemblyLinearVelocity=Vector3.new(0,0,0) end
            State.dropBrainrotActive=false; return
        end
        r.AssemblyLinearVelocity=Vector3.new(r.AssemblyLinearVelocity.X,DROP_ASCEND_SPEED,r.AssemblyLinearVelocity.Z)
    end)
end

-- ========== MOBILE PANEL ==========
local function createMobilePanel()
    local panel = Instance.new("ScreenGui")
    panel.Name="OmniHubButtons"; panel.Parent=LP:WaitForChild("PlayerGui"); panel.ResetOnSpawn=false; panel.ZIndexBehavior=Enum.ZIndexBehavior.Sibling

    mbPanel = Instance.new("Frame", panel)
    mbPanel.Name = "MobilePanel"
    mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H)
    mbPanel.Position = UDim2.new(1, -(MB_TOTAL_W+8), 0.5, -(MB_TOTAL_H/2))
    mbPanel.BackgroundColor3 = Theme.Background
    mbPanel.BorderSizePixel = 0; mbPanel.Active = true; mbPanel.ZIndex = 50
    Instance.new("UICorner", mbPanel).CornerRadius = UDim.new(0, MB_CORNER+4)
    local mbStroke = Instance.new("UIStroke", mbPanel)
    mbStroke.Color = Theme.Accent; mbStroke.Thickness = 1.5; mbStroke.Transparency = 0.45

    local dragStart, startPos, moved = nil, nil, false
    local DRAG_THRESHOLD = 8
    mbPanel.InputBegan:Connect(function(inp)
        if MobileButtons.Locked then return end
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
            dragStart=inp.Position; startPos=mbPanel.Position; moved=false
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if MobileButtons.Locked or not dragStart then return end
        if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then
            local d=inp.Position-dragStart
            if not moved and (math.abs(d.X)+math.abs(d.Y))>DRAG_THRESHOLD then moved=true end
            if moved then mbPanel.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+d.X,startPos.Y.Scale,startPos.Y.Offset+d.Y) end
        end
    end)
    UIS.InputEnded:Connect(function(inp)
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragStart=nil; moved=false end
    end)

    local function makeMobileBtn(label, row, col, onClick)
        local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
        local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
        local btn = Instance.new("TextButton", mbPanel)
        btn.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
        btn.Position = UDim2.new(0, x, 0, y)
        btn.BackgroundColor3 = Theme.Element; btn.BorderSizePixel = 0
        btn.Text = ""; btn.AutoButtonColor = false; btn.ZIndex = 52
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, MB_CORNER)
        local bs = Instance.new("UIStroke", btn); bs.Color = Theme.Accent; bs.Thickness = 1; bs.Transparency = 0.7
        local lbl = Instance.new("TextLabel", btn)
        lbl.Size = UDim2.new(1,0,1,0); lbl.BackgroundTransparency = 1
        lbl.Text = label; lbl.TextColor3 = Theme.Accent; lbl.Font = Theme.FontBold
        lbl.TextSize = 9; lbl.TextWrapped = true; lbl.TextXAlignment = Enum.TextXAlignment.Center; lbl.TextYAlignment = Enum.TextYAlignment.Center; lbl.ZIndex = 56

        btn.MouseButton1Down:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.05), {BackgroundColor3=Theme.ElementHover}):Play()
            TweenService:Create(bs, TweenInfo.new(0.05), {Transparency=0}):Play()
        end)
        btn.MouseButton1Up:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.10), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.10), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseEnter:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.ElementHover}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.4}):Play()
            end
        end)
        btn.MouseLeave:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseButton1Click:Connect(function() if onClick then pcall(onClick) end end)

        btn:SetAttribute("MB_On", false)
        btn.AttributeChanged:Connect(function(a)
            if a == "MB_On" then
                local on = btn:GetAttribute("MB_On")
                if on then
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Accent}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(5,5,15)}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0, Color = Theme.AccentGlow}):Play()
                    bs.Thickness = 2
                else
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Element}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Theme.Accent}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0.7, Color = Theme.Accent}):Play()
                    bs.Thickness = 1
                end
            end
        end)
        table.insert(MobileButtons.BtnsObjects, btn)
        return btn
    end

    local alMbBtn = makeMobileBtn("AUTO\nLEFT", 1, 1, function()
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoLeftEnabled = not State.autoLeftEnabled
        if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end
        if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoLeft = alMbBtn

    local arMbBtn = makeMobileBtn("AUTO\nRIGHT", 1, 2, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoRightEnabled = not State.autoRightEnabled
        if setAutoRight then setAutoRight(State.autoRightEnabled) end
        if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoRight = arMbBtn

    local batMbBtn = makeMobileBtn("BAT", 2, 1, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        State.autoBatToggled = not State.autoBatToggled
        if not State.autoBatToggled then resetBend() end
        if setAutoBat then setAutoBat(State.autoBatToggled) end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoBat = batMbBtn

    makeMobileBtn("DROP", 2, 2, function() runDropBrainrot() end)

    local carryMbBtn = makeMobileBtn("CARRY\nSPD", 3, 1, function() toggleSpeedType() end)
    MobileButtons.BtnRefs.carrySpeed = carryMbBtn

    local lagMbBtn = makeMobileBtn("LAG", 3, 2, function() toggleLagger() end)
    MobileButtons.BtnRefs.lagger = lagMbBtn

    makeMobileBtn("TP\nDOWN", 4, 1, function() tpToGround() end)

    local guiMbBtn = makeMobileBtn("GUI", 4, 2, function() if toggleGuiVis then toggleGuiVis() end end)
    MobileButtons.HideGuiBtn = guiMbBtn

    task.spawn(function()
        while task.wait(0.2) do
            pcall(function()
                if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
                if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
                if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
                if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
                if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
            end)
        end
    end)

    task.spawn(function()
        task.wait(0.1)
        if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
        if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
        if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
        if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
        if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
    end)

    for _, btn in pairs(MobileButtons.BtnsObjects) do btn.Visible = MobileButtons.Visible end
    mbPanel.Visible = MobileButtons.Visible
end

local function repositionMbButtons()
    if not mbPanel then return end
    local idx = 0
    for _, child in ipairs(mbPanel:GetChildren()) do
        if child:IsA("TextButton") then
            local row = math.floor(idx / MB_COLS) + 1
            local col = (idx % MB_COLS) + 1
            local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
            local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
            child.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
            child.Position = UDim2.new(0, x, 0, y)
            idx = idx + 1
        end
    end
end

local function applyMbBtnSize(w, h2)
    w = math.clamp(w, 40, 120); h2 = math.clamp(h2, 36, 120)
    MB_BTN_W = w; MB_BTN_H = h2
    MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
    MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
    if mbPanel then mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H); repositionMbButtons() end
end

-- ========== ANIMATIONS ==========
local Anims = {idle1="rbxassetid://133806214992291",idle2="rbxassetid://94970088341563",walk="rbxassetid://707897309",run="rbxassetid://707861613",jump="rbxassetid://116936326516985",fall="rbxassetid://116936326516985",climb="rbxassetid://116936326516985",swim="rbxassetid://116936326516985",swimidle="rbxassetid://116936326516985"}
task.spawn(function() pcall(function() ContentProvider:PreloadAsync({Anims.idle1,Anims.idle2,Anims.walk,Anims.run,Anims.jump,Anims.fall,Anims.climb,Anims.swim,Anims.swimidle}) end) end)
local animHeartbeatConn=nil; local savedAnimate=nil; local originalAnims=nil
local function isPackAnim(id) if not id then return false end; for _,v in pairs(Anims) do if v==id then return true end end; return false end
local function saveOriginalAnims(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function g(obj) return obj and obj.AnimationId or nil end; local ids={idle1=g(animate.idle and animate.idle.Animation1),idle2=g(animate.idle and animate.idle.Animation2),walk=g(animate.walk and animate.walk.WalkAnim),run=g(animate.run and animate.run.RunAnim),jump=g(animate.jump and animate.jump.JumpAnim),fall=g(animate.fall and animate.fall.FallAnim),climb=g(animate.climb and animate.climb.ClimbAnim),swim=g(animate.swim and animate.swim.Swim),swimidle=g(animate.swimidle and animate.swimidle.SwimIdle)}; if not isPackAnim(ids.walk) then originalAnims=ids end end
local function applyAnimPack(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,Anims.idle1); s(animate.idle and animate.idle.Animation2,Anims.idle2); s(animate.walk and animate.walk.WalkAnim,Anims.walk); s(animate.run and animate.run.RunAnim,Anims.run); s(animate.jump and animate.jump.JumpAnim,Anims.jump); s(animate.fall and animate.fall.FallAnim,Anims.fall); s(animate.climb and animate.climb.ClimbAnim,Anims.climb); s(animate.swim and animate.swim.Swim,Anims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,Anims.swimidle) end
local function restoreOriginalAnims(char) if not originalAnims then return end; local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj and id then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,originalAnims.idle1); s(animate.idle and animate.idle.Animation2,originalAnims.idle2); s(animate.walk and animate.walk.WalkAnim,originalAnims.walk); s(animate.run and animate.run.RunAnim,originalAnims.run); s(animate.jump and animate.jump.JumpAnim,originalAnims.jump); s(animate.fall and animate.fall.FallAnim,originalAnims.fall); s(animate.climb and animate.climb.ClimbAnim,originalAnims.climb); s(animate.swim and animate.swim.Swim,originalAnims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,originalAnims.swimidle); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end
local function startAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then saveOriginalAnims(char); applyAnimPack(char); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end; animHeartbeatConn=RunService.Heartbeat:Connect(function() if not State.animEnabled then return end; local c=LP.Character; if c then applyAnimPack(c) end end) end
local function stopAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then restoreOriginalAnims(char) end end
local function startUnwalk() if State.unwalkEnabled then return end; State.unwalkEnabled=true; local c=LP.Character; if not c then return end; local hum=c:FindFirstChildOfClass("Humanoid"); if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end; local anim=c:FindFirstChild("Animate"); if anim then savedAnimate=anim:Clone(); anim:Destroy() end end
local function stopUnwalk() if not State.unwalkEnabled then return end; State.unwalkEnabled=false; local c=LP.Character; if c and savedAnimate then savedAnimate.Parent=c; savedAnimate.Disabled=false; savedAnimate=nil end; task.spawn(function() task.wait(0.15); local char=LP.Character; if not char then return end; if State.animEnabled then saveOriginalAnims(char); applyAnimPack(char) else restoreOriginalAnims(char) end end) end

-- ========== MAIN GUI ==========
local gui = Instance.new("ScreenGui")
gui.Name="OmniHubGUI"; gui.ResetOnSpawn=false; gui.DisplayOrder=10; gui.IgnoreGuiInset=true; gui.Parent=LP:WaitForChild("PlayerGui")

local stealProgressBar = Instance.new("Frame", gui)
stealProgressBar.Size=UDim2.new(0,280,0,6); stealProgressBar.Position=UDim2.new(0.5,-140,0.9,0)
stealProgressBar.BackgroundColor3=Color3.fromRGB(8,12,25); stealProgressBar.BackgroundTransparency=0; stealProgressBar.BorderSizePixel=0; stealProgressBar.ZIndex=100
Instance.new("UICorner",stealProgressBar).CornerRadius=UDim.new(1,0)
local barStroke=Instance.new("UIStroke",stealProgressBar); barStroke.Color=Theme.AccentDim; barStroke.Thickness=1
local barFill=Instance.new("Frame",stealProgressBar); barFill.Size=UDim2.new(0,0,1,0); barFill.BackgroundColor3=Theme.Accent; barFill.BorderSizePixel=0
Instance.new("UICorner",barFill).CornerRadius=UDim.new(1,0); AutoSteal.ProgressFill=barFill

local function makeStealBarDraggable(frame)
    local dragging=false; local dragInput,dragStart,startPos,activeDragConn
    local function stopDrag() dragging=false; dragInput=nil; dragStart=nil; startPos=nil; if activeDragConn then activeDragConn:Disconnect(); activeDragConn=nil end end
    frame.InputBegan:Connect(function(input) if MobileButtons.Locked then return end; if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=input.Position; startPos=frame.Position; input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End or input.UserInputState==Enum.UserInputState.Cancelled then stopDrag() end end) end end)
    frame.InputChanged:Connect(function(input) if not dragging then return end; if MobileButtons.Locked then stopDrag(); return end; if input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch then dragInput=input end end)
    UIS.InputChanged:Connect(function(input) if input==dragInput and dragging then if MobileButtons.Locked then stopDrag(); return end; local delta=input.Position-dragStart; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y) end end)
end
makeStealBarDraggable(stealProgressBar)
local function saveStealBarPosition() local pos=stealProgressBar.Position; pcall(function() writefile("JispiStealBarPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function loadStealBarPosition() local savedPos=nil; pcall(function() savedPos=readfile("JispiStealBarPos.txt") end); if savedPos and savedPos~="" then local parts={}; for part in string.gmatch(savedPos,"[^,]+") do table.insert(parts,part) end; if #parts>=4 then stealProgressBar.Position=UDim2.new(tonumber(parts[1]),tonumber(parts[2]),tonumber(parts[3]),tonumber(parts[4])) end end end
loadStealBarPosition()
local lastStealBarSave=0; stealProgressBar:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastStealBarSave>0.5 then lastStealBarSave=tick(); saveStealBarPosition() end end)

-- ========== MAIN FRAME ==========
main = Instance.new("Frame", gui)
main.Name="Main"; main.Size=UDim2.new(0,320,0,470); main.Position=UDim2.new(0,20,0,20)
main.BackgroundColor3=Theme.Background; main.BorderSizePixel=0; main.Active=true; main.ClipsDescendants=true
Instance.new("UICorner",main).CornerRadius=UDim.new(0,16)
local outerGlow=Instance.new("UIStroke",main); outerGlow.Color=Theme.Accent; outerGlow.Thickness=1.5; outerGlow.Transparency=0.3
local glowPulse=Instance.new("Frame",main); glowPulse.Size=UDim2.new(1,8,1,8); glowPulse.Position=UDim2.new(0,-4,0,-4)
glowPulse.BackgroundColor3=Theme.Accent; glowPulse.BackgroundTransparency=0.88; glowPulse.BorderSizePixel=0; glowPulse.ZIndex=-1
Instance.new("UICorner",glowPulse).CornerRadius=UDim.new(0,20)
task.spawn(function()
    while true do
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.78}):Play()
        task.wait(1.8)
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.92}):Play()
        task.wait(1.8)
    end
end)

local function saveMainPosition() local pos=main.Position; pcall(function() writefile("JispiHubGUIPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function saveMiniPosition() if miniBtn then local pos=miniBtn.Position; pcall(function() writefile("JispiMiniPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end end
local function loadMainPosition() local sp=nil; pcall(function() sp=readfile("JispiHubGUIPos.txt") end); if sp and sp~="" then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then main.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])); if closeBtn then closeBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2])+320-34,tonumber(p[3]),tonumber(p[4])+18) end end end end
local function loadMiniPosition() local sp=nil; pcall(function() sp=readfile("JispiMiniPos.txt") end); if sp and sp~="" and miniBtn then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then miniBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])) end end end

-- ========== HEADER ==========
local header=Instance.new("Frame",main)
header.Size=UDim2.new(1,0,0,52); header.BackgroundColor3=Theme.Header; header.BorderSizePixel=0; header.ZIndex=5
Instance.new("UICorner",header).CornerRadius=UDim.new(0,16)
local headerAccentLine=Instance.new("Frame",header)
headerAccentLine.Size=UDim2.new(1,0,0,2); headerAccentLine.Position=UDim2.new(0,0,1,-2)
headerAccentLine.BackgroundColor3=Theme.Accent; headerAccentLine.BorderSizePixel=0; headerAccentLine.ZIndex=6
local lineGlow=Instance.new("UIGradient",headerAccentLine)
lineGlow.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,30)),ColorSequenceKeypoint.new(0.5,Theme.AccentGlow),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,30))})
local titleLbl=Instance.new("TextLabel",header)
titleLbl.Size=UDim2.new(1,0,1,-4); titleLbl.Position=UDim2.new(0,0,0,0)
titleLbl.BackgroundTransparency=1; titleLbl.Text="â  RBG HUB  â"
titleLbl.TextColor3=Theme.AccentGlow; titleLbl.Font=Theme.FontBlack; titleLbl.TextSize=17
titleLbl.TextXAlignment=Enum.TextXAlignment.Center; titleLbl.ZIndex=6
local versionLbl=Instance.new("TextLabel",header)
versionLbl.Size=UDim2.new(0,40,0,16); versionLbl.Position=UDim2.new(1,-50,0.5,-8)
versionLbl.BackgroundTransparency=1; versionLbl.Text="v1.0"; versionLbl.TextColor3=Theme.AccentDim
versionLbl.Font=Theme.Font; versionLbl.TextSize=9; versionLbl.ZIndex=6

-- ========== CLOSE / MINI ==========
closeBtn=Instance.new("TextButton",gui)
closeBtn.Size=UDim2.new(0,28,0,28); closeBtn.BackgroundColor3=Color3.fromRGB(14,18,35)
closeBtn.BorderSizePixel=0; closeBtn.Text="â"; closeBtn.TextColor3=Theme.SubText
closeBtn.Font=Theme.FontBold; closeBtn.TextSize=14; closeBtn.ZIndex=50
Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(0,8)
local closeStroke=Instance.new("UIStroke",closeBtn); closeStroke.Color=Theme.Border; closeStroke.Thickness=1.5
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Danger,TextColor3=Color3.new(1,1,1)}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Danger}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Color3.fromRGB(14,18,35),TextColor3=Theme.SubText}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Border}):Play() end)
closeBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)

miniBtn=Instance.new("ImageButton",gui); miniBtn.Name="OmniMiniButton"; miniBtn.Size=UDim2.new(0,44,0,44)
miniBtn.Position=UDim2.new(0,20,0,100); miniBtn.BackgroundColor3=Theme.Background; miniBtn.Image=""; miniBtn.BorderSizePixel=0; miniBtn.Visible=false; miniBtn.ZIndex=50
Instance.new("UICorner",miniBtn).CornerRadius=UDim.new(0,12)
local miniStroke=Instance.new("UIStroke",miniBtn); miniStroke.Color=Theme.Accent; miniStroke.Thickness=1.8
local miniIcon=Instance.new("TextLabel",miniBtn); miniIcon.Size=UDim2.new(1,0,1,0); miniIcon.Text="â"; miniIcon.TextColor3=Theme.AccentGlow; miniIcon.Font=Theme.FontBlack; miniIcon.TextSize=20; miniIcon.BackgroundTransparency=1
miniBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)
miniBtn.MouseEnter:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Element}):Play() end)
miniBtn.MouseLeave:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Background}):Play() end)

local function makeMainDraggable(frame)
    local dragging,dragInput,dragStart,startPos,startCloseBtnPos=false,nil,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; if closeBtn then startCloseBtnPos=closeBtn.Position end; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy); if closeBtn and startCloseBtnPos then closeBtn.Position=UDim2.new(startCloseBtnPos.X.Scale,startCloseBtnPos.X.Offset+dx,startCloseBtnPos.Y.Scale,startCloseBtnPos.Y.Offset+dy) end end end)
end
local function makeMiniDraggable(frame)
    local dragging,dragInput,dragStart,startPos=false,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy) end end)
end
makeMainDraggable(main); makeMiniDraggable(miniBtn)
local lastSaveTime=0; main:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastSaveTime>0.5 then lastSaveTime=tick(); saveMainPosition() end end)
local lastMiniSaveTime=0; miniBtn:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastMiniSaveTime>0.5 then lastMiniSaveTime=tick(); saveMiniPosition() end end)
local function updateCloseButtonPosition() if main.Visible then local p=main.Position; closeBtn.Position=UDim2.new(p.X.Scale,p.X.Offset+320-34,p.Y.Scale,p.Y.Offset+18) end end
main:GetPropertyChangedSignal("Position"):Connect(updateCloseButtonPosition); main:GetPropertyChangedSignal("Visible"):Connect(updateCloseButtonPosition)

-- ========== TAB SYSTEM ==========
local tabFrame=Instance.new("Frame",main)
tabFrame.Size=UDim2.new(1,-16,0,30); tabFrame.Position=UDim2.new(0,8,0,56)
tabFrame.BackgroundTransparency=1; tabFrame.BorderSizePixel=0; tabFrame.ZIndex=3
local tabLayout=Instance.new("UIListLayout",tabFrame)
tabLayout.FillDirection=Enum.FillDirection.Horizontal; tabLayout.SortOrder=Enum.SortOrder.LayoutOrder
tabLayout.Padding=UDim.new(0,4); tabLayout.HorizontalAlignment=Enum.HorizontalAlignment.Center

local scroll=Instance.new("ScrollingFrame",main)
scroll.Size=UDim2.new(1,-16,1,-94); scroll.Position=UDim2.new(0,8,0,90)
scroll.BackgroundTransparency=1; scroll.BorderSizePixel=0; scroll.ScrollBarThickness=3
scroll.ScrollBarImageColor3=Theme.Accent; scroll.ScrollBarImageTransparency=0.4
scroll.AutomaticCanvasSize=Enum.AutomaticSize.Y; scroll.CanvasSize=UDim2.new(0,0,0,0); scroll.ZIndex=2
local listLayout=Instance.new("UIListLayout",scroll); listLayout.SortOrder=Enum.SortOrder.LayoutOrder; listLayout.Padding=UDim.new(0,5)
local pad=Instance.new("UIPadding",scroll); pad.PaddingLeft=UDim.new(0,0); pad.PaddingRight=UDim.new(0,0); pad.PaddingTop=UDim.new(0,2); pad.PaddingBottom=UDim.new(0,8)

local tabDefs = {{"Speed",1},{"Features",2},{"Settings",3}}
local tabBtns = {}; local tabPages = {}; local activeTab = "Speed"

for _,td in ipairs(tabDefs) do
    local btn=Instance.new("TextButton",tabFrame)
    btn.Size=UDim2.new(0,80,0,26); btn.BackgroundColor3=(td[1]==activeTab) and Theme.AccentDark or Theme.Element
    btn.BorderSizePixel=0; btn.Text=td[1]:upper(); btn.TextColor3=(td[1]==activeTab) and Theme.AccentGlow or Theme.SubText
    btn.Font=Theme.FontBold; btn.TextSize=9; btn.LayoutOrder=td[2]; btn.ZIndex=4; btn.AutoButtonColor=false
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,8)
    local ts=Instance.new("UIStroke",btn); ts.Color=(td[1]==activeTab) and Theme.Accent or Theme.Border; ts.Thickness=(td[1]==activeTab) and 1.5 or 1
    tabBtns[td[1]]={btn=btn,stroke=ts}
    local page=Instance.new("Frame",scroll)
    page.Size=UDim2.new(1,0,0,0); page.BackgroundTransparency=1; page.BorderSizePixel=0
    page.Visible=(td[1]==activeTab); page.ZIndex=2
    page.AutomaticSize=Enum.AutomaticSize.Y
    local pageLL=Instance.new("UIListLayout",page); pageLL.SortOrder=Enum.SortOrder.LayoutOrder; pageLL.Padding=UDim.new(0,5)
    local pagePad=Instance.new("UIPadding",page); pagePad.PaddingBottom=UDim.new(0,4)
    tabPages[td[1]]=page
    local pageLO=0
    btn.MouseButton1Click:Connect(function()
        if activeTab==td[1] then return end
        activeTab=td[1]
        for name,data in pairs(tabBtns) do
            local isA=(name==activeTab)
            TweenService:Create(data.btn,TweenInfo.new(0.15),{BackgroundColor3=isA and Theme.AccentDark or Theme.Element}):Play()
            TweenService:Create(data.btn,TweenInfo.new(0.15),{TextColor3=isA and Theme.AccentGlow or Theme.SubText}):Play()
            TweenService:Create(data.stroke,TweenInfo.new(0.15),{Color=isA and Theme.Accent or Theme.Border,Thickness=isA and 1.5 or 1}):Play()
            tabPages[name].Visible=isA
        end
    end)
    btn.MouseEnter:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end
    end)
    btn.MouseLeave:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.Element}):Play() end
    end)
end

-- ========== UI HELPERS (per-page) ==========
local pageLOs = {Speed=0, Features=0, Settings=0}
local function LO(tab) pageLOs[tab]=(pageLOs[tab] or 0)+1; return pageLOs[tab] end
local function pg(tab) return tabPages[tab] end

local function makeGap(tab,px) local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,0,0,px or 3); f.BackgroundTransparency=1; f.LayoutOrder=LO(tab) end
local function makeDivider(tab)
    local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,-8,0,1); f.BackgroundColor3=Theme.Border; f.BackgroundTransparency=0; f.BorderSizePixel=0; f.LayoutOrder=LO(tab)
    local g=Instance.new("UIGradient",f); g.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,20)),ColorSequenceKeypoint.new(0.5,Theme.Accent),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,20))})
end
local function makeSectionLabel(tab,text)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,26); row.BackgroundTransparency=1; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-8,1,0); lbl.BackgroundTransparency=1
    lbl.Text="  â "..text:upper(); lbl.TextColor3=Theme.Accent; lbl.Font=Theme.FontBold; lbl.TextSize=10; lbl.TextXAlignment=Enum.TextXAlignment.Left
end

local function makeInputRow(tab,label,default,onChange)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0,100,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.ZIndex=2
    local box=Instance.new("TextBox",row); box.Size=UDim2.new(0,60,0,26); box.Position=UDim2.new(1,-70,0.5,-13); box.BackgroundColor3=Theme.BackgroundSecondary; box.BorderSizePixel=0; box.Text=tostring(default); box.TextColor3=Theme.AccentGlow; box.Font=Theme.FontBold; box.TextSize=13; box.ClearTextOnFocus=false; box.ZIndex=3
    Instance.new("UICorner",box).CornerRadius=UDim.new(0,8)
    local boxStroke=Instance.new("UIStroke",box); boxStroke.Color=Theme.AccentDim; boxStroke.Thickness=1
    box.Focused:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.Element}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
    box.FocusLost:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.BackgroundSecondary}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); local num=tonumber(box.Text); if num~=nil then local fv=math.floor(math.clamp(num,0,500)); box.Text=tostring(fv); if onChange then onChange(tostring(fv)) end; autoSaveConfig() else box.Text=tostring(default) end end)
    return box
end

local function makeStatusRow(tab,label,valTxt)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,32); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0.5,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local val=Instance.new("TextLabel",row); val.Size=UDim2.new(0.45,-10,1,0); val.Position=UDim2.new(0.52,0,0,0); val.BackgroundTransparency=1; val.Text=valTxt; val.TextColor3=Theme.AccentGlow; val.Font=Theme.FontBold; val.TextSize=12; val.TextXAlignment=Enum.TextXAlignment.Right
    return val
end

local function makeKeybindRow(tab,label,currentKey,onChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local btn=Instance.new("TextButton",row); btn.Size=UDim2.new(0,70,0,24); btn.Position=UDim2.new(1,-80,0.5,-12); btn.BackgroundColor3=Theme.Keybind; btn.BorderSizePixel=0; btn.Text="["..currentKey.Name.."]"; btn.TextColor3=Theme.Accent; btn.Font=Theme.FontBold; btn.TextSize=10
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,7)
    local bs=Instance.new("UIStroke",btn); bs.Color=Theme.AccentDim; bs.Thickness=1
    local listening=false; local listenConn
    local function stopListen(key)
        listening=false; if listenConn then listenConn:Disconnect(); listenConn=nil end
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(btn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); btn.TextColor3=Theme.Accent
        if key then btn.Text="["..key.Name.."]"; if onChanged then onChanged(key) end; autoSaveConfig() end
    end
    btn.MouseButton1Click:Connect(function()
        if listening then stopListen(nil); return end
        listening=true; btn.Text="[Â·Â·Â·]"; btn.TextColor3=Theme.AccentGlow; btn.BackgroundColor3=Theme.Element
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
        listenConn=UIS.InputBegan:Connect(function(inp,gp)
            if not listening then return end
            if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode)
            elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode) end
        end)
    end)
    return btn
end

local function makeToggleRow(tab,label,defaultKey,defaultOn,onToggle,onKeyChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,40); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-120,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local keyBtn=nil
    if defaultKey then
        keyBtn=Instance.new("TextButton",row); keyBtn.Size=UDim2.new(0,50,0,22); keyBtn.Position=UDim2.new(1,-108,0.5,-11); keyBtn.BackgroundColor3=Theme.Keybind; keyBtn.BorderSizePixel=0; keyBtn.Text=defaultKey.Name; keyBtn.TextColor3=Theme.Accent; keyBtn.Font=Theme.FontBold; keyBtn.TextSize=9; keyBtn.ZIndex=5
        Instance.new("UICorner",keyBtn).CornerRadius=UDim.new(0,5)
        local ks=Instance.new("UIStroke",keyBtn); ks.Color=Theme.AccentDim; ks.Thickness=1
        local kListening=false; local kConn
        local function kStop(key) kListening=false; if kConn then kConn:Disconnect(); kConn=nil end; TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(keyBtn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); keyBtn.TextColor3=Theme.Accent; if key then keyBtn.Text=key.Name; if onKeyChanged then onKeyChanged(key) end; autoSaveConfig() end end
        keyBtn.MouseButton1Click:Connect(function()
            if kListening then kStop(nil); return end
            kListening=true; keyBtn.Text="Â·Â·Â·"; keyBtn.TextColor3=Theme.AccentGlow; keyBtn.BackgroundColor3=Theme.Element
            TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
            kConn=UIS.InputBegan:Connect(function(inp,gp)
                if not kListening then return end
                if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode)
                elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode) end
            end)
        end)
    end
    local toggleBg=Instance.new("TextButton",row); toggleBg.Size=UDim2.new(0,42,0,22); toggleBg.Position=UDim2.new(1,-48,0.5,-11)
    toggleBg.BackgroundColor3=defaultOn and Theme.AccentDark or Theme.ToggleOff; toggleBg.BorderSizePixel=0; toggleBg.Text=""; toggleBg.ZIndex=5; toggleBg.AutoButtonColor=false
    Instance.new("UICorner",toggleBg).CornerRadius=UDim.new(1,0)
    local tStroke=Instance.new("UIStroke",toggleBg); tStroke.Color=defaultOn and Theme.Accent or Theme.Border; tStroke.Thickness=1.5
    local knob=Instance.new("Frame",toggleBg); knob.Size=UDim2.new(0,17,0,17); knob.Position=defaultOn and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5)
    knob.BackgroundColor3=defaultOn and Theme.AccentGlow or Theme.SubText; knob.BorderSizePixel=0; knob.ZIndex=6
    Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
    local isOn=defaultOn or false
    local function setV(on)
        isOn=on
        TweenService:Create(toggleBg,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{BackgroundColor3=on and Theme.AccentDark or Theme.ToggleOff}):Play()
        TweenService:Create(tStroke,TweenInfo.new(0.22),{Color=on and Theme.Accent or Theme.Border}):Play()
        TweenService:Create(knob,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{Position=on and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=on and Theme.AccentGlow or Theme.SubText}):Play()
        TweenService:Create(rowStroke,TweenInfo.new(0.22),{Color=on and Theme.AccentDim or Theme.Border,Thickness=on and 1.5 or 1}):Play()
    end
    toggleBg.MouseButton1Click:Connect(function() isOn=not isOn; setV(isOn); if onToggle then pcall(onToggle,isOn) end; autoSaveConfig() end)
    toggleBg.MouseEnter:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
    toggleBg.MouseLeave:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)
    return setV, keyBtn
end

-- ===================== BUILD TABS =====================

-- ===== SPEED TAB =====
makeSectionLabel("Speed","Speed")
normalBox=makeInputRow("Speed","Normal",State.normalSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.normalSpeed=n end end)
carryBox=makeInputRow("Speed","Carry",State.carrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.carrySpeed=n end end)
setSpeedToggleUI,speedKeyBtn=makeToggleRow("Speed","Toggle",Keys.speed,false,function(on) toggleSpeedType() end,function(k) Keys.speed=k end)
modeValLbl=makeStatusRow("Speed","Mode","Normal")
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Lagger Speed")
laggerBox=makeInputRow("Speed","Normal",State.laggerSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerSpeed=n end end)
carryLaggerBox=makeInputRow("Speed","Carry",State.laggerCarrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerCarrySpeed=n end end)
setLaggerToggleUI,laggerKeyBtn=makeToggleRow("Speed","Toggle",Keys.lagger,false,function(on) toggleLagger() end,function(k) Keys.lagger=k end)
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Combat")
setAutoBat,autoBatKeyBtn=makeToggleRow("Speed","Bat Aimbot",Keys.autoBat,false,function(on) State.autoBatToggled=on; if not on then resetBend() end end,function(k) Keys.autoBat=k end)
makeGap("Speed",4)

-- ===== FEATURES TAB =====
makeSectionLabel("Features","Auto Steal")
local radiusBox=makeInputRow("Features","Grab Rad",AutoSteal.Radius,function(v) local n=tonumber(v); if n and n>=5 and n<=300 then AutoSteal.Radius=math.floor(n); if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end end)
setInstaGrab=makeToggleRow("Features","Auto Grab",nil,false,function(on) AutoSteal.Enabled=on; if on then startAutoSteal() else stopAutoSteal() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Active Toggles")
setInfJump=makeToggleRow("Features","Infinite Jump",nil,false,function(on) State.infJumpEnabled=on end)
setAntiRag=makeToggleRow("Features","Anti Ragdoll",nil,false,function(on) State.antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
setFps=makeToggleRow("Features","FPS Boost",nil,false,function(on) State.fpsBoostEnabled=on; if on then pcall(applyFPSBoost) end end)
setMedusaCounter=makeToggleRow("Features","Medusa Counter",nil,false,function(on) State.medusaCounterEnabled=on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
setAnimToggle=makeToggleRow("Features","Tryhard Anim",nil,false,function(on) State.animEnabled=on; if on then startAnimToggle() else stopAnimToggle() end end)
setUnwalkToggle=makeToggleRow("Features","Unwalk",nil,false,function(on) if on then startUnwalk() else stopUnwalk() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Movement")
tpDownKeyBtn=makeKeybindRow("Features","TP Down",Keys.tpDown,function(k) Keys.tpDown=k end)
dropBrainrotKeyBtn=makeKeybindRow("Features","Drop Brainrot",Keys.dropBrainrot,function(k) Keys.dropBrainrot=k end)
setAutoLeft,autoLeftKeyBtn=makeToggleRow("Features","Auto Left",Keys.autoLeft,false,function(on) State.autoLeftEnabled=on; if on then startAutoLeft() else stopAutoLeft() end end,function(k) Keys.autoLeft=k end)
setAutoRight,autoRightKeyBtn=makeToggleRow("Features","Auto Right",Keys.autoRight,false,function(on) State.autoRightEnabled=on; if on then startAutoRight() else stopAutoRight() end end,function(k) Keys.autoRight=k end)
makeGap("Features",4)

-- ===== SETTINGS TAB =====
makeSectionLabel("Settings","Interface")
guiHideKeyBtn=makeKeybindRow("Settings","Hide GUI",Keys.guiHide,function(k) Keys.guiHide=k end)
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

makeSectionLabel("Settings","Mobile Panel")
local showRow=Instance.new("Frame",pg("Settings")); showRow.Size=UDim2.new(1,0,0,38); showRow.BackgroundColor3=Theme.Element; showRow.BorderSizePixel=0; showRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",showRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",showRow).Color=Theme.Border
local showLbl=Instance.new("TextLabel",showRow); showLbl.Size=UDim2.new(1,-115,1,0); showLbl.Position=UDim2.new(0,12,0,0); showLbl.BackgroundTransparency=1; showLbl.Text="Show Buttons"; showLbl.TextColor3=Theme.Text; showLbl.Font=Theme.FontMedium; showLbl.TextSize=12; showLbl.TextXAlignment=Enum.TextXAlignment.Left
local showToggleBg=Instance.new("TextButton",showRow); showToggleBg.Size=UDim2.new(0,42,0,22); showToggleBg.Position=UDim2.new(1,-48,0.5,-11); showToggleBg.BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff; showToggleBg.BorderSizePixel=0; showToggleBg.Text=""; showToggleBg.ZIndex=5; showToggleBg.AutoButtonColor=false
Instance.new("UICorner",showToggleBg).CornerRadius=UDim.new(1,0)
local showKnob=Instance.new("Frame",showToggleBg); showKnob.Size=UDim2.new(0,17,0,17); showKnob.Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); showKnob.BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText; showKnob.BorderSizePixel=0; showKnob.ZIndex=6; Instance.new("UICorner",showKnob).CornerRadius=UDim.new(1,0)
showToggleBg.MouseButton1Click:Connect(function() MobileButtons.Visible=not MobileButtons.Visible; if mbPanel then mbPanel.Visible=MobileButtons.Visible end; TweenService:Create(showToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(showKnob,TweenInfo.new(0.22),{Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
showToggleBg.MouseEnter:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
showToggleBg.MouseLeave:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local lockRow=Instance.new("Frame",pg("Settings")); lockRow.Size=UDim2.new(1,0,0,38); lockRow.BackgroundColor3=Theme.Element; lockRow.BorderSizePixel=0; lockRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",lockRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",lockRow).Color=Theme.Border
local lockLbl=Instance.new("TextLabel",lockRow); lockLbl.Size=UDim2.new(1,-115,1,0); lockLbl.Position=UDim2.new(0,12,0,0); lockLbl.BackgroundTransparency=1; lockLbl.Text="Lock Buttons"; lockLbl.TextColor3=Theme.Text; lockLbl.Font=Theme.FontMedium; lockLbl.TextSize=12; lockLbl.TextXAlignment=Enum.TextXAlignment.Left
local lockToggleBg=Instance.new("TextButton",lockRow); lockToggleBg.Size=UDim2.new(0,42,0,22); lockToggleBg.Position=UDim2.new(1,-48,0.5,-11); lockToggleBg.BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff; lockToggleBg.BorderSizePixel=0; lockToggleBg.Text=""; lockToggleBg.ZIndex=5; lockToggleBg.AutoButtonColor=false
Instance.new("UICorner",lockToggleBg).CornerRadius=UDim.new(1,0)
local lockKnob=Instance.new("Frame",lockToggleBg); lockKnob.Size=UDim2.new(0,17,0,17); lockKnob.Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); lockKnob.BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText; lockKnob.BorderSizePixel=0; lockKnob.ZIndex=6; Instance.new("UICorner",lockKnob).CornerRadius=UDim.new(1,0)
lockToggleBg.MouseButton1Click:Connect(function() MobileButtons.Locked=not MobileButtons.Locked; TweenService:Create(lockToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(lockKnob,TweenInfo.new(0.22),{Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
lockToggleBg.MouseEnter:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
lockToggleBg.MouseLeave:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local sizeRow=Instance.new("Frame",pg("Settings")); sizeRow.Size=UDim2.new(1,0,0,38); sizeRow.BackgroundColor3=Theme.Element; sizeRow.BorderSizePixel=0; sizeRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",sizeRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",sizeRow).Color=Theme.Border
local sizeLbl=Instance.new("TextLabel",sizeRow); sizeLbl.Size=UDim2.new(0,100,1,0); sizeLbl.Position=UDim2.new(0,12,0,0); sizeLbl.BackgroundTransparency=1; sizeLbl.Text="Btn Size"; sizeLbl.TextColor3=Theme.Text; sizeLbl.Font=Theme.FontMedium; sizeLbl.TextSize=12; sizeLbl.TextXAlignment=Enum.TextXAlignment.Left
local sizeValLbl=Instance.new("TextLabel",sizeRow); sizeValLbl.Size=UDim2.new(0,40,0,16); sizeValLbl.Position=UDim2.new(0,112,0.5,-8); sizeValLbl.BackgroundTransparency=1; sizeValLbl.Text=MB_BTN_W.."px"; sizeValLbl.TextColor3=Theme.AccentGlow; sizeValLbl.Font=Theme.FontBold; sizeValLbl.TextSize=9; sizeValLbl.TextXAlignment=Enum.TextXAlignment.Center
local decBtn=Instance.new("TextButton",sizeRow); decBtn.Size=UDim2.new(0,24,0,24); decBtn.Position=UDim2.new(1,-78,0.5,-12); decBtn.BackgroundColor3=Theme.BackgroundSecondary; decBtn.BorderSizePixel=0; decBtn.Text="-"; decBtn.TextColor3=Theme.Text; decBtn.Font=Theme.FontBlack; decBtn.TextSize=14; decBtn.ZIndex=5
Instance.new("UICorner",decBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",decBtn).Color=Theme.Border
local incBtn=Instance.new("TextButton",sizeRow); incBtn.Size=UDim2.new(0,24,0,24); incBtn.Position=UDim2.new(1,-48,0.5,-12); incBtn.BackgroundColor3=Theme.BackgroundSecondary; incBtn.BorderSizePixel=0; incBtn.Text="+"; incBtn.TextColor3=Theme.Text; incBtn.Font=Theme.FontBlack; incBtn.TextSize=14; incBtn.ZIndex=5
Instance.new("UICorner",incBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",incBtn).Color=Theme.Border
decBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W-4,MB_BTN_H-3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
incBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W+4,MB_BTN_H+3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
for _,b in pairs({decBtn,incBtn}) do
    b.MouseEnter:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
    b.MouseLeave:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
end
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

local saveConfigRow=Instance.new("Frame",pg("Settings")); saveConfigRow.Size=UDim2.new(1,0,0,40); saveConfigRow.BackgroundColor3=Theme.Element; saveConfigRow.BorderSizePixel=0; saveConfigRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",saveConfigRow).CornerRadius=UDim.new(0,10)
local saveConfigBtn=Instance.new("TextButton",saveConfigRow); saveConfigBtn.Size=UDim2.new(1,-16,0,28); saveConfigBtn.Position=UDim2.new(0,8,0.5,-14)
saveConfigBtn.BackgroundColor3=Color3.fromRGB(0,40,80); saveConfigBtn.BorderSizePixel=0; saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; saveConfigBtn.Font=Theme.FontBold; saveConfigBtn.TextSize=12; saveConfigBtn.ZIndex=5; saveConfigBtn.AutoButtonColor=false
Instance.new("UICorner",saveConfigBtn).CornerRadius=UDim.new(0,9)
local saveBtnStroke=Instance.new("UIStroke",saveConfigBtn); saveBtnStroke.Color=Theme.Accent; saveBtnStroke.Thickness=1.5
saveConfigBtn.MouseEnter:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,60,120)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.AccentGlow}):Play() end)
saveConfigBtn.MouseLeave:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,40,80)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
saveConfigBtn.MouseButton1Click:Connect(function() forceSaveConfig(); saveConfigBtn.Text="â  SAVED!"; saveConfigBtn.TextColor3=Theme.Success; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Success}):Play(); task.wait(1.2); saveConfigBtn.Text="â¬¡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)

local resetMbRow=Instance.new("Frame",pg("Settings")); resetMbRow.Size=UDim2.new(1,0,0,36); resetMbRow.BackgroundColor3=Theme.Element; resetMbRow.BorderSizePixel=0; resetMbRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",resetMbRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",resetMbRow).Color=Theme.Border
local resetMbBtn=Instance.new("TextButton",resetMbRow); resetMbBtn.Size=UDim2.new(1,-16,0,24); resetMbBtn.Position=UDim2.new(0,8,0.5,-12)
resetMbBtn.BackgroundColor3=Theme.BackgroundSecondary; resetMbBtn.BorderSizePixel=0; resetMbBtn.Text="Reset Button Panel Position"; resetMbBtn.TextColor3=Theme.SubText; resetMbBtn.Font=Theme.FontMedium; resetMbBtn.TextSize=10; resetMbBtn.ZIndex=5; resetMbBtn.AutoButtonColor=false
Instance.new("UICorner",resetMbBtn).CornerRadius=UDim.new(0,7)
resetMbBtn.MouseEnter:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
resetMbBtn.MouseLeave:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
resetMbBtn.MouseButton1Click:Connect(function() if mbPanel then mbPanel.Position=UDim2.new(1,-(MB_TOTAL_W+8),0.5,-(MB_TOTAL_H/2)) end end)

makeGap("Settings",2)
local footerLbl=Instance.new("TextLabel",pg("Settings")); footerLbl.Size=UDim2.new(1,0,0,18); footerLbl.BackgroundTransparency=1; footerLbl.LayoutOrder=LO("Settings"); footerLbl.Text="â RBG â"; footerLbl.TextColor3=Theme.AccentDim; footerLbl.Font=Theme.FontBold; footerLbl.TextSize=9; footerLbl.TextXAlignment=Enum.TextXAlignment.Center

-- ========== TOGGLE GUI VISIBILITY ==========
toggleGuiVis=function()
    State.guiVisible=not State.guiVisible
    if main then main.Visible=State.guiVisible; closeBtn.Visible=State.guiVisible; if miniBtn then miniBtn.Visible=not State.guiVisible end end
end

-- ========== LOGIC ==========
local function findMedusa()
    local char=LP.Character; if not char then return nil end
    for _,tool in ipairs(char:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end
    local bp2=LP:FindFirstChild("Backpack"); if bp2 then for _,tool in ipairs(bp2:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end end
    return nil
end
local function useMedusaCounter()
    if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<25 then return end; local char=LP.Character; if not char then return end
    State.medusaDebounce=true; local med=findMedusa(); if not med then State.medusaDebounce=false; return end
    if med.Parent~=char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
end
local function onAnchorChanged(part) return part:GetPropertyChangedSignal("Anchored"):Connect(function() if part.Anchored and part.Transparency==1 then useMedusaCounter() end end) end
setupMedusaCounter=function(char) stopMedusaCounter(); if not char then return end; for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end; table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end)) end
stopMedusaCounter=function() for _,c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end; Conns.anchor={} end

startAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect() end; State.autoLeftPhase=1
    Conns.autoLeft=RunService.Heartbeat:Connect(function()
        if not State.autoLeftEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoLeftPhase==1 then
            local tgt=Vector3.new(POS.L1.X,root.Position.Y,POS.L1.Z); if (tgt-root.Position).Magnitude<1 then State.autoLeftPhase=2; return end
            local d=(POS.L1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoLeftPhase==2 then
            local tgt=Vector3.new(POS.L2.X,root.Position.Y,POS.L2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoLeftEnabled=false; if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1; if setAutoLeft then setAutoLeft(false) end; return end
            local d=(POS.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect() end; State.autoRightPhase=1
    Conns.autoRight=RunService.Heartbeat:Connect(function()
        if not State.autoRightEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoRightPhase==1 then
            local tgt=Vector3.new(POS.R1.X,root.Position.Y,POS.R1.Z); if (tgt-root.Position).Magnitude<1 then State.autoRightPhase=2; return end
            local d=(POS.R1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoRightPhase==2 then
            local tgt=Vector3.new(POS.R2.X,root.Position.Y,POS.R2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoRightEnabled=false; if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1; if setAutoRight then setAutoRight(false) end; return end
            local d=(POS.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAntiRagdoll=function()
    if Conns.antiRag then return end
    Conns.antiRag=RunService.Heartbeat:Connect(function()
        local char=LP.Character; if not char then return end; local hum2=char:FindFirstChildOfClass("Humanoid"); local root=char:FindFirstChild("HumanoidRootPart")
        if hum2 then local st=hum2:GetState(); if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then hum2:ChangeState(Enum.HumanoidStateType.Running); workspace.CurrentCamera.CameraSubject=hum2; pcall(function() local pm=LP.PlayerScripts:FindFirstChild("PlayerModule"); if pm then require(pm:FindFirstChild("ControlModule")):Enable() end end); if root then root.Velocity=Vector3.new(0,0,0); root.RotVelocity=Vector3.new(0,0,0) end end end
        for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled=true end end
    end)
end
stopAntiRagdoll=function() if Conns.antiRag then Conns.antiRag:Disconnect(); Conns.antiRag=nil end end
applyFPSBoost=function()
    pcall(function() setfpscap(999999999) end)
    local function processObj(v) pcall(function() if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0 elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1 elseif v:IsA("SpecialMesh") then v.TextureId="" elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false elseif v:IsA("SurfaceAppearance") or v:IsA("MaterialVariant") then v:Destroy() elseif v:IsA("Attachment") then v.Visible=false end end) end
    for _,v in pairs(workspace:GetDescendants()) do processObj(v) end
    pcall(function() local lighting=game:GetService("Lighting"); for _,v in pairs(lighting:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end; pcall(function() sethiddenproperty(game:GetService("Lighting"),"Technology",Enum.Technology.Legacy) end); local lighting2=game:GetService("Lighting"); lighting2.GlobalShadows=false; lighting2.FogEnd=9e9; lighting2.Brightness=0; local terrain=workspace:FindFirstChildOfClass("Terrain"); if terrain then pcall(function() sethiddenproperty(terrain,"Decoration",false) end); terrain.WaterReflectance=0; terrain.WaterTransparency=0.7; terrain.WaterWaveSize=0; terrain.WaterWaveSpeed=0 end end)
    workspace.DescendantAdded:Connect(function(v) if State.fpsBoostEnabled then task.spawn(processObj,v) end end)
end

-- ========== CHARACTER SETUP ==========
local function setupChar(char)
    task.wait(0.1); h=char:WaitForChild("Humanoid",5); hrp=char:WaitForChild("HumanoidRootPart",5); State.originalC0=nil
    if not h or not hrp then return end
    local head=char:FindFirstChild("Head")
    if head then
        local oldBB=head:FindFirstChild("SpeedBillboard"); if oldBB then oldBB:Destroy() end
        local bb=Instance.new("BillboardGui",head); bb.Name="SpeedBillboard"; bb.Size=UDim2.new(0,120,0,22); bb.StudsOffset=Vector3.new(0,3,0); bb.AlwaysOnTop=true
        speedLbl=Instance.new("TextLabel",bb); speedLbl.Size=UDim2.new(1,0,1,0); speedLbl.BackgroundTransparency=1; speedLbl.TextColor3=Theme.AccentGlow; speedLbl.Font=Theme.FontBold; speedLbl.TextScaled=true; speedLbl.TextStrokeTransparency=0.5; speedLbl.TextStrokeColor3=Color3.fromRGB(0,20,40)
    end
    if State.antiRagdollEnabled and not Conns.antiRag then task.wait(0.5); startAntiRagdoll() end
    if State.medusaCounterEnabled then setupMedusaCounter(char) end
    if State.animEnabled then task.wait(0.3); saveOriginalAnims(char); applyAnimPack(char) end
    if State.unwalkEnabled then State.unwalkEnabled=false; task.wait(0.3); startUnwalk() end
end
LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

RunService.Stepped:Connect(function()
    for _,p in ipairs(Players:GetPlayers()) do if p~=LP and p.Character then for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end end end
end)

UIS.JumpRequest:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root then root.Velocity=Vector3.new(root.Velocity.X,55,root.Velocity.Z) end
end)
RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root and root.Velocity.Y<-120 then root.Velocity=Vector3.new(root.Velocity.X,-120,root.Velocity.Z) end
end)

RunService.RenderStepped:Connect(function()
    if not (h and hrp) then return end; if State._tpInProgress then return end
    if State.autoLeftEnabled or State.autoRightEnabled then if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end; return end
    local md=h.MoveDirection; local spd=getCurrentSpeed()
    if md.Magnitude>0 then State.lastMoveDir=md; hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
    elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude>0 then local anyHeld=false; for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true; break end end; if anyHeld then hrp.Velocity=Vector3.new(State.lastMoveDir.X*spd,hrp.Velocity.Y,State.lastMoveDir.Z*spd) end end
    if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end
end)

RunService.Heartbeat:Connect(function(dt)
    if not (State.autoBatToggled and h and hrp) then resetBend(); return end
    local target,dist=getClosestPlayer(); if not (target and target.Character) then resetBend(); return end
    local tr=target.Character:FindFirstChild("HumanoidRootPart"); if not tr then resetBend(); return end
    local rj=getRootJoint(); if rj and not State.originalC0 then State.originalC0=rj.C0 end
    local yDiff=tr.Position.Y-hrp.Position.Y; local yVel=math.clamp(yDiff*20,-120,120)
    if dist>6 then
        local dir=(tr.Position-hrp.Position).Unit; hrp.AssemblyLinearVelocity=Vector3.new(dir.X*59,yVel,dir.Z*59)
        if rj then rj.C0=State.originalC0*CFrame.Angles(math.rad(40),0,0) end
    else
        State.crazyAngle=State.crazyAngle+dt*40
        local offsetX=math.cos(State.crazyAngle)*0.8; local offsetZ=math.sin(State.crazyAngle)*0.8
        local targetPos=tr.Position+Vector3.new(offsetX,0,offsetZ); local flatDir=Vector3.new(targetPos.X-hrp.Position.X,0,targetPos.Z-hrp.Position.Z)
        hrp.AssemblyLinearVelocity=Vector3.new(flatDir.Unit.X*200,yVel,flatDir.Unit.Z*200)
        if rj then local t=tick(); local fwdBack=math.sin(t*25)*math.rad(50); local sideways=math.cos(t*20)*math.rad(30); rj.C0=State.originalC0*CFrame.Angles(fwdBack,0,sideways) end
        tryHitBat()
    end
end)

UIS.InputBegan:Connect(function(inp,gp)
    if gp then return end
    if inp.UserInputType~=Enum.UserInputType.Keyboard and inp.UserInputType~=Enum.UserInputType.Gamepad1 then return end
    local kc=inp.KeyCode
    if (State.autoLeftEnabled or State.autoRightEnabled) then if MOVE_KEYS[kc] then return end end
    if kc==Keys.speed then toggleSpeedType()
    elseif kc==Keys.lagger then toggleLagger()
    elseif kc==Keys.autoBat then State.autoBatToggled=not State.autoBatToggled; if not State.autoBatToggled then resetBend() end; setAutoBat(State.autoBatToggled); autoSaveConfig()
    elseif kc==Keys.autoLeft then State.autoLeftEnabled=not State.autoLeftEnabled; if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end; if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    elseif kc==Keys.autoRight then State.autoRightEnabled=not State.autoRightEnabled; if setAutoRight then setAutoRight(State.autoRightEnabled) end; if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    elseif kc==Keys.dropBrainrot then task.spawn(runDropBrainrot)
    elseif kc==Keys.tpDown then tpToGround()
    elseif kc==Keys.guiHide then toggleGuiVis() end
end)

task.spawn(function() while task.wait(0.5) do pcall(function() if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end) end end)

local function loadConfig()
    local hasFile=false; pcall(function() hasFile=isfile("JispiHubConfig.json") end); if not hasFile then return end
    local ok,cfg=pcall(function() return HttpService:JSONDecode(readfile("JispiHubConfig.json")) end); if not ok or not cfg then return end
    if cfg.normalSpeed and type(cfg.normalSpeed)=="number" then State.normalSpeed=cfg.normalSpeed; normalBox.Text=tostring(cfg.normalSpeed) end
    if cfg.carrySpeed  and type(cfg.carrySpeed)=="number"  then State.carrySpeed=cfg.carrySpeed;   carryBox.Text=tostring(cfg.carrySpeed)   end
    if cfg.laggerSpeed and type(cfg.laggerSpeed)=="number" then State.laggerSpeed=cfg.laggerSpeed; laggerBox.Text=tostring(cfg.laggerSpeed) end
    if cfg.laggerCarrySpeed and type(cfg.laggerCarrySpeed)=="number" then State.laggerCarrySpeed=cfg.laggerCarrySpeed; carryLaggerBox.Text=tostring(cfg.laggerCarrySpeed) end
    if cfg.speedType=="normal" or cfg.speedType=="carry" then State.speedType=cfg.speedType end
    if type(cfg.laggerActive)=="boolean" then State.laggerActive=cfg.laggerActive end
    if cfg.autoBatKey  and Enum.KeyCode[cfg.autoBatKey]    then Keys.autoBat=Enum.KeyCode[cfg.autoBatKey];   if autoBatKeyBtn  then autoBatKeyBtn.Text="["..cfg.autoBatKey.."]"   end end
    if cfg.speedKey    and Enum.KeyCode[cfg.speedKey]      then Keys.speed=Enum.KeyCode[cfg.speedKey]        end
    if cfg.laggerKey   and Enum.KeyCode[cfg.laggerKey]     then Keys.lagger=Enum.KeyCode[cfg.laggerKey]      end
    if cfg.autoLeftKey  and Enum.KeyCode[cfg.autoLeftKey]  then Keys.autoLeft=Enum.KeyCode[cfg.autoLeftKey];   if autoLeftKeyBtn  then autoLeftKeyBtn.Text="["..cfg.autoLeftKey.."]"   end end
    if cfg.autoRightKey and Enum.KeyCode[cfg.autoRightKey] then Keys.autoRight=Enum.KeyCode[cfg.autoRightKey]; if autoRightKeyBtn then autoRightKeyBtn.Text="["..cfg.autoRightKey.."]" end end
    if cfg.tpDownKey    and Enum.KeyCode[cfg.tpDownKey]    then Keys.tpDown=Enum.KeyCode[cfg.tpDownKey];       if tpDownKeyBtn    then tpDownKeyBtn.Text="["..cfg.tpDownKey.."]"        end end
    if cfg.grabRadius and type(cfg.grabRadius)=="number" then AutoSteal.Radius=cfg.grabRadius; if progressRadLbl then progressRadLbl.Text="Radius: "..cfg.grabRadius end end
    if cfg.autoStealEnabled then AutoSteal.Enabled=true; setInstaGrab(true); pcall(startAutoSteal) end
    if cfg.infJump      then State.infJumpEnabled=true;      setInfJump(true)      end
    if cfg.antiRagdoll  then State.antiRagdollEnabled=true;  setAntiRag(true);     startAntiRagdoll() end
    if cfg.fpsBoost     then State.fpsBoostEnabled=true;     setFps(true);         applyFPSBoost()    end
    if cfg.medusaCounter then State.medusaCounterEnabled=true; setMedusaCounter(true); setupMedusaCounter(LP.Character) end
    if cfg.dropBrainrotKey and Enum.KeyCode[cfg.dropBrainrotKey] then Keys.dropBrainrot=Enum.KeyCode[cfg.dropBrainrotKey]; if dropBrainrotKeyBtn then dropBrainrotKeyBtn.Text="["..cfg.dropBrainrotKey.."]"                    bat:Activate() -- Ø¨ÙØ®
    normalSpeed = 60, carrySpeed = 30, laggerSpeed = 13, laggerCarrySpeed = 13,
    speedType = "normal", laggerActive = false, autoBatToggled = false,
    hittingCooldown = false, infJumpEnabled = false, antiRagdollEnabled = false,
    fpsBoostEnabled = false, guiVisible = true, isStealing = false,
    stealStartTime = nil, lastStealTick = 0, medusaLastUsed = 0,
    medusaDebounce = false, medusaCounterEnabled = false, dropBrainrotActive = false,
    autoLeftEnabled = false, autoRightEnabled = false, autoLeftPhase = 1,
    autoRightPhase = 1, _tpInProgress = false, lastMoveDir = Vector3.new(0,0,0),
    animEnabled = false, unwalkEnabled = false, originalC0 = nil, crazyAngle = 0,
}

local Keys = {
    autoBat = Enum.KeyCode.E, speed = Enum.KeyCode.Q, lagger = Enum.KeyCode.C,
    guiHide = Enum.KeyCode.LeftControl, autoLeft = Enum.KeyCode.L,
    autoRight = Enum.KeyCode.R, dropBrainrot = Enum.KeyCode.H, tpDown = Enum.KeyCode.T,
}

local main, closeBtn, miniBtn
local toggleGuiVis

local MobileButtons = { Visible = true, Locked = false, BtnsObjects = {}, BtnRefs = {}, Buttons = {}, HideGuiBtn = nil }

local MB_BTN_W = 58
local MB_BTN_H = 48
local MB_COLS = 2
local MB_ROWS = 4
local MB_PAD = 3
local MB_CORNER = 14
local MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
local MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
local mbPanel = nil

local Theme = {
    Background      = Color3.fromRGB(8, 10, 20),
    BackgroundSecondary = Color3.fromRGB(12, 15, 28),
    Header          = Color3.fromRGB(5, 8, 18),
    Element         = Color3.fromRGB(14, 18, 35),
    ElementHover    = Color3.fromRGB(20, 26, 50),
    Text            = Color3.fromRGB(200, 220, 255),
    SubText         = Color3.fromRGB(80, 110, 160),
    Accent          = Color3.fromRGB(0, 180, 255),
    AccentDark      = Color3.fromRGB(0, 100, 200),
    AccentGlow      = Color3.fromRGB(0, 220, 255),
    AccentDim       = Color3.fromRGB(0, 60, 120),
    Border          = Color3.fromRGB(0, 60, 120),
    BorderLight     = Color3.fromRGB(0, 100, 180),
    Success         = Color3.fromRGB(0, 255, 160),
    Danger          = Color3.fromRGB(255, 50, 80),
    Keybind         = Color3.fromRGB(10, 20, 45),
    KeybindText     = Color3.fromRGB(0, 180, 255),
    ToggleOff       = Color3.fromRGB(18, 22, 45),
    Font            = Enum.Font.Gotham,
    FontBold        = Enum.Font.GothamBold,
    FontBlack       = Enum.Font.GothamBlack,
    FontMedium      = Enum.Font.GothamMedium,
}

local AutoSteal = {
    Enabled = false, Radius = 20, Duration = 1.3, IsStealing = false,
    Data = {}, ProgressFill = nil, ProgressText = nil,
}

local function isMyPlotByName(plotName)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return false end
    local plot = plots:FindFirstChild(plotName); if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then local yb = sign:FindFirstChild("YourBase"); if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end end
    return false
end

local function findNearestPrompt()
    local char = LP.Character; local root = char and char:FindFirstChild("HumanoidRootPart"); if not root then return nil end
    local plots = workspace:FindFirstChild("Plots"); if not plots then return nil end
    local bestPrompt, bestDist, bestName = nil, math.huge, nil
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local podiums = plot:FindFirstChild("AnimalPodiums"); if not podiums then continue end
        for _, pod in ipairs(podiums:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base"); local spawn = base and base:FindFirstChild("Spawn")
                if spawn then
                    local dist = (spawn.Position - root.Position).Magnitude
                    if dist < bestDist and dist <= AutoSteal.Radius then
                        local att = spawn:FindFirstChild("PromptAttachment")
                        if att then for _, child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") then bestPrompt, bestDist, bestName = child, dist, pod.Name; break end
                        end end
                    end
                end
            end)
        end
    end
    return bestPrompt, bestDist, bestName
end

local function executeSteal(prompt, animalName)
    if AutoSteal.IsStealing then return end
    if not AutoSteal.Data[prompt] then
        AutoSteal.Data[prompt] = {hold = {}, trigger = {}, ready = true}
        pcall(function()
            if getconnections then
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(AutoSteal.Data[prompt].hold, c.Function) end end
                for _, c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(AutoSteal.Data[prompt].trigger, c.Function) end end
            end
        end)
    end
    local data = AutoSteal.Data[prompt]; if not data.ready then return end
    data.ready = false; AutoSteal.IsStealing = true; local startTime = tick()
    local conn; conn = RunService.Heartbeat:Connect(function()
        if not AutoSteal.IsStealing then conn:Disconnect(); return end
        local prog = math.clamp((tick() - startTime) / AutoSteal.Duration, 0, 1)
        if AutoSteal.ProgressFill then AutoSteal.ProgressFill.Size = UDim2.new(prog, 0, 1, 0) end
    end)
    task.spawn(function()
        for _, f in ipairs(data.hold) do task.spawn(f) end
        task.wait(AutoSteal.Duration)
        for _, f in ipairs(data.trigger) do task.spawn(f) end
        AutoSteal.IsStealing = false; data.ready = true
        task.wait(0.6)
        if not AutoSteal.IsStealing and AutoSteal.ProgressFill then
            TweenService:Create(AutoSteal.ProgressFill, TweenInfo.new(0.4), {Size = UDim2.new(0,0,1,0)}):Play()
        end
    end)
end

local autoStealConnection = nil
local function startAutoSteal()
    if autoStealConnection then return end
    autoStealConnection = RunService.Heartbeat:Connect(function()
        if AutoSteal.Enabled and not AutoSteal.IsStealing then
            local p, _, name = findNearestPrompt(); if p then executeSteal(p, name) end
        end
    end)
end
local function stopAutoSteal()
    if autoStealConnection then autoStealConnection:Disconnect(); autoStealConnection = nil end
    AutoSteal.IsStealing = false
    for k, v in pairs(AutoSteal.Data) do if v.ready ~= nil then v.ready = true end end
end

local MOVE_KEYS = {[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local DROP_ASCEND_DURATION = 0.2; local DROP_ASCEND_SPEED = 150
local POS = {L1=Vector3.new(-476.48,-6.28,92.73),L2=Vector3.new(-483.12,-4.95,94.80),R1=Vector3.new(-476.16,-6.52,25.62),R2=Vector3.new(-483.04,-5.09,23.14)}
local Conns = {autoSteal=nil,antiRag=nil,autoLeft=nil,autoRight=nil,anchor={},progress=nil}
local h, hrp, speedLbl
local setAutoLeft, setAutoRight
local setInstaGrab, setAutoBat, setInfJump, setAntiRag, setFps, setMedusaCounter
local setAnimToggle, setUnwalkToggle
local setupMedusaCounter, stopMedusaCounter, startAntiRagdoll, stopAntiRagdoll, applyFPSBoost
local modeValLbl, normalBox, carryBox, laggerBox, carryLaggerBox
local autoBatKeyBtn, speedKeyBtn, laggerKeyBtn, autoLeftKeyBtn, autoRightKeyBtn, guiHideKeyBtn, dropBrainrotKeyBtn, tpDownKeyBtn
local setSpeedToggleUI, setLaggerToggleUI
local progressRadLbl
local startAutoLeft, stopAutoLeft, startAutoRight, stopAutoRight

local function getRootJoint()
    local char2 = LP.Character; local torso = char2 and char2:FindFirstChild("LowerTorso")
    return torso and torso:FindFirstChild("Root")
end
local function resetBend()
    local rj = getRootJoint()
    if rj and State.originalC0 then rj.C0 = State.originalC0; State.originalC0 = nil end
end
local function getBat()
    local char = LP.Character; if not char then return nil end
    local tool = char:FindFirstChild("Bat"); if tool then return tool end
    local bp2 = LP:FindFirstChild("Backpack")
    if bp2 then tool = bp2:FindFirstChild("Bat"); if tool then tool.Parent = char; return tool end end
    return nil
end
local function tryHitBat()
    if State.hittingCooldown then return end; State.hittingCooldown = true
    pcall(function()
        local bat = getBat(); if bat then bat:Activate(); local ev = bat:FindFirstChildWhichIsA("RemoteEvent"); if ev then ev:FireServer() end end
    end)
    task.delay(0.08, function() State.hittingCooldown = false end)
end
local function getClosestPlayer()
    if not hrp then return nil, math.huge end
    local cp, cd = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if tr then local d = (hrp.Position - tr.Position).Magnitude; if d < cd then cd = d; cp = p end end
        end
    end
    return cp, cd
end

local saveDebounce = false
local function autoSaveConfig()
    if saveDebounce then return end; saveDebounce = true
    task.delay(0.5, function()
        local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
        pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end); saveDebounce = false
    end)
end
local function forceSaveConfig()
    local cfg = {normalSpeed=State.normalSpeed,carrySpeed=State.carrySpeed,laggerSpeed=State.laggerSpeed,laggerCarrySpeed=State.laggerCarrySpeed,speedType=State.speedType,laggerActive=State.laggerActive,autoBatKey=Keys.autoBat.Name,speedKey=Keys.speed.Name,laggerKey=Keys.lagger.Name,autoStealEnabled=AutoSteal.Enabled,grabRadius=AutoSteal.Radius,infJump=State.infJumpEnabled,antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,medusaCounter=State.medusaCounterEnabled,dropBrainrotKey=Keys.dropBrainrot.Name,autoLeftKey=Keys.autoLeft.Name,autoRightKey=Keys.autoRight.Name,guiHideKey=Keys.guiHide.Name,animEnabled=State.animEnabled,unwalkEnabled=State.unwalkEnabled,tpDownKey=Keys.tpDown.Name,mobileVisible=MobileButtons.Visible,mobileLocked=MobileButtons.Locked,mbBtnW=MB_BTN_W,mbBtnH=MB_BTN_H}
    pcall(function() writefile("JispiHubConfig.json", HttpService:JSONEncode(cfg)) end)
end
localsteal(steal) local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local Conns = {autoSteal=nil,antiRag=nil,anchor={},progress=nil,aimbot=nil,batCounter=nil}
local PLOT_CACHE_DURATION = 2
local PROMPT_CACHE_REFRESH = 0.15
local STEAL_COOLDOWN = 0.1
local MEDUSA_COOLDOWN = 25
local BAT_COUNTER_COOLDOWN = 0
local batCounterDebounce = false
local batCounterLastUsed = 0
local progressRadLbl,progressFill,progressPct
local modeValLbl
local _aimTarget = nil
local _aimLastScan = 0

local VYSE_AIMBOT_SPEED = 56.5
local VYSE_HIT_DIST = 5
local SWING_COOLDOWN = 0.08
local hittingCooldown = false

local desyncEnabled = false
local setDesyncVisual = nil
local desyncActive = false
local desyncSpeedConn = nil
local G_desyncAnimate = nil

local fpsBoostEnabled = false

local function refreshUIToggles()
    if setSpeedToggleUI then setSpeedToggleUI(State.speedType == "carry") end
    if setLaggerToggleUI then setLaggerToggleUI(State.laggerActive) end
    if modeValLbl then
        if State.laggerActive then modeValLbl.Text = (State.speedType=="normal") and "⚡ Lagger Normal" or "⚡ Lagger Carry"
        else modeValLbl.Text = (State.speedType=="normal") and "Normal" or "Carry" end
    end
end
local function toggleSpeedType()
    State.speedType = (State.speedType=="normal") and "carry" or "normal"; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.carrySpeed then MobileButtons.BtnRefs.carrySpeed:SetAttribute("MB_On", State.speedType=="carry") end
end
local function toggleLagger()
    State.laggerActive = not State.laggerActive; refreshUIToggles(); autoSaveConfig()
    if MobileButtons.BtnRefs.lagger then MobileButtons.BtnRefs.lagger:SetAttribute("MB_On", State.laggerActive) end
end
local function getCurrentSpeed()
    if State.laggerActive then return State.speedType=="normal" and State.laggerSpeed or State.laggerCarrySpeed
    else return State.speedType=="normal" and State.normalSpeed or State.carrySpeed end
end
local function getAutoMoveSpeed()
    return State.laggerActive and State.laggerSpeed or State.normalSpeed
end
local function tpToGround()
    local char = LP.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
    local rp = RaycastParams.new(); rp.FilterType = Enum.RaycastFilterType.Exclude; rp.FilterDescendantsInstances = {char}
    local rr = workspace:Raycast(root.Position, Vector3.new(0,-500,0), rp)
    if rr then root.CFrame = CFrame.new(rr.Position + Vector3.new(0,3,0)) else root.CFrame = root.CFrame * CFrame.new(0,-20,0) end
end
local function runDropBrainrot()
    if State.dropBrainrotActive then return end
    local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if not root then return end
    State.dropBrainrotActive=true; local t0=tick(); local dc
    dc=RunService.Heartbeat:Connect(function()
        local r=char and char:FindFirstChild("HumanoidRootPart"); if not r then dc:Disconnect(); State.dropBrainrotActive=false; return end
        if tick()-t0>=DROP_ASCEND_DURATION then
            dc:Disconnect()
            local rp=RaycastParams.new(); rp.FilterDescendantsInstances={char}; rp.FilterType=Enum.RaycastFilterType.Exclude
            local rr=workspace:Raycast(r.Position,Vector3.new(0,-2000,0),rp)
            if rr then local hum2=char:FindFirstChildOfClass("Humanoid"); local off=(hum2 and hum2.HipHeight or 2)+(r.Size.Y/2); r.CFrame=CFrame.new(r.Position.X,rr.Position.Y+off,r.Position.Z); r.AssemblyLinearVelocity=Vector3.new(0,0,0) end
            State.dropBrainrotActive=false; return
        end
        r.AssemblyLinearVelocity=Vector3.new(r.AssemblyLinearVelocity.X,DROP_ASCEND_SPEED,r.AssemblyLinearVelocity.Z)
    end)
end

-- ========== MOBILE PANEL ==========
local function createMobilePanel()
    local panel = Instance.new("ScreenGui")
    panel.Name="OmniHubButtons"; panel.Parent=LP:WaitForChild("PlayerGui"); panel.ResetOnSpawn=false; panel.ZIndexBehavior=Enum.ZIndexBehavior.Sibling

    mbPanel = Instance.new("Frame", panel)
    mbPanel.Name = "MobilePanel"
    mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H)
    mbPanel.Position = UDim2.new(1, -(MB_TOTAL_W+8), 0.5, -(MB_TOTAL_H/2))
    mbPanel.BackgroundColor3 = Theme.Background
    mbPanel.BorderSizePixel = 0; mbPanel.Active = true; mbPanel.ZIndex = 50
    Instance.new("UICorner", mbPanel).CornerRadius = UDim.new(0, MB_CORNER+4)
    local mbStroke = Instance.new("UIStroke", mbPanel)
    mbStroke.Color = Theme.Accent; mbStroke.Thickness = 1.5; mbStroke.Transparency = 0.45

    local dragStart, startPos, moved = nil, nil, false
    local DRAG_THRESHOLD = 8
    mbPanel.InputBegan:Connect(function(inp)
        if MobileButtons.Locked then return end
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
            dragStart=inp.Position; startPos=mbPanel.Position; moved=false
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if MobileButtons.Locked or not dragStart then return end
        if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then
            local d=inp.Position-dragStart
            if not moved and (math.abs(d.X)+math.abs(d.Y))>DRAG_THRESHOLD then moved=true end
            if moved then mbPanel.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+d.X,startPos.Y.Scale,startPos.Y.Offset+d.Y) end
        end
    end)
    UIS.InputEnded:Connect(function(inp)
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragStart=nil; moved=false end
    end)

    local function makeMobileBtn(label, row, col, onClick)
        local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
        local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
        local btn = Instance.new("TextButton", mbPanel)
        btn.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
        btn.Position = UDim2.new(0, x, 0, y)
        btn.BackgroundColor3 = Theme.Element; btn.BorderSizePixel = 0
        btn.Text = ""; btn.AutoButtonColor = false; btn.ZIndex = 52
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, MB_CORNER)
        local bs = Instance.new("UIStroke", btn); bs.Color = Theme.Accent; bs.Thickness = 1; bs.Transparency = 0.7
        local lbl = Instance.new("TextLabel", btn)
        lbl.Size = UDim2.new(1,0,1,0); lbl.BackgroundTransparency = 1
        lbl.Text = label; lbl.TextColor3 = Theme.Accent; lbl.Font = Theme.FontBold
        lbl.TextSize = 9; lbl.TextWrapped = true; lbl.TextXAlignment = Enum.TextXAlignment.Center; lbl.TextYAlignment = Enum.TextYAlignment.Center; lbl.ZIndex = 56

        btn.MouseButton1Down:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.05), {BackgroundColor3=Theme.ElementHover}):Play()
            TweenService:Create(bs, TweenInfo.new(0.05), {Transparency=0}):Play()
        end)
        btn.MouseButton1Up:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.10), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.10), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseEnter:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.ElementHover}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.4}):Play()
            end
        end)
        btn.MouseLeave:Connect(function()
            if not btn:GetAttribute("MB_On") then
                TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Theme.Element}):Play()
                TweenService:Create(bs, TweenInfo.new(0.1), {Transparency=0.7}):Play()
            end
        end)
        btn.MouseButton1Click:Connect(function() if onClick then pcall(onClick) end end)

        btn:SetAttribute("MB_On", false)
        btn.AttributeChanged:Connect(function(a)
            if a == "MB_On" then
                local on = btn:GetAttribute("MB_On")
                if on then
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Accent}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(5,5,15)}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0, Color = Theme.AccentGlow}):Play()
                    bs.Thickness = 2
                else
                    TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Theme.Element}):Play()
                    TweenService:Create(lbl, TweenInfo.new(0.15), {TextColor3 = Theme.Accent}):Play()
                    TweenService:Create(bs, TweenInfo.new(0.15), {Transparency = 0.7, Color = Theme.Accent}):Play()
                    bs.Thickness = 1
                end
            end
        end)
        table.insert(MobileButtons.BtnsObjects, btn)
        return btn
    end

    local alMbBtn = makeMobileBtn("AUTO\nLEFT", 1, 1, function()
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoLeftEnabled = not State.autoLeftEnabled
        if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end
        if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoLeft = alMbBtn

    local arMbBtn = makeMobileBtn("AUTO\nRIGHT", 1, 2, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoBatToggled then State.autoBatToggled=false; if setAutoBat then setAutoBat(false) end; resetBend() end
        State.autoRightEnabled = not State.autoRightEnabled
        if setAutoRight then setAutoRight(State.autoRightEnabled) end
        if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoRight = arMbBtn

    local batMbBtn = makeMobileBtn("BAT", 2, 1, function()
        if State.autoLeftEnabled then State.autoLeftEnabled=false; if setAutoLeft then setAutoLeft(false) end; stopAutoLeft() end
        if State.autoRightEnabled then State.autoRightEnabled=false; if setAutoRight then setAutoRight(false) end; stopAutoRight() end
        State.autoBatToggled = not State.autoBatToggled
        if not State.autoBatToggled then resetBend() end
        if setAutoBat then setAutoBat(State.autoBatToggled) end; autoSaveConfig()
    end)
    MobileButtons.BtnRefs.autoBat = batMbBtn

    makeMobileBtn("DROP", 2, 2, function() runDropBrainrot() end)

    local carryMbBtn = makeMobileBtn("CARRY\nSPD", 3, 1, function() toggleSpeedType() end)
    MobileButtons.BtnRefs.carrySpeed = carryMbBtn

    local lagMbBtn = makeMobileBtn("LAG", 3, 2, function() toggleLagger() end)
    MobileButtons.BtnRefs.lagger = lagMbBtn

    makeMobileBtn("TP\nDOWN", 4, 1, function() tpToGround() end)

    local guiMbBtn = makeMobileBtn("GUI", 4, 2, function() if toggleGuiVis then toggleGuiVis() end end)
    MobileButtons.HideGuiBtn = guiMbBtn

    task.spawn(function()
        while task.wait(0.2) do
            pcall(function()
                if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
                if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
                if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
                if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
                if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
            end)
        end
    end)

    task.spawn(function()
        task.wait(0.1)
        if alMbBtn then alMbBtn:SetAttribute("MB_On", State.autoLeftEnabled) end
        if arMbBtn then arMbBtn:SetAttribute("MB_On", State.autoRightEnabled) end
        if batMbBtn then batMbBtn:SetAttribute("MB_On", State.autoBatToggled) end
        if carryMbBtn then carryMbBtn:SetAttribute("MB_On", State.speedType=="carry") end
        if lagMbBtn then lagMbBtn:SetAttribute("MB_On", State.laggerActive) end
    end)

    for _, btn in pairs(MobileButtons.BtnsObjects) do btn.Visible = MobileButtons.Visible end
    mbPanel.Visible = MobileButtons.Visible
end

local function repositionMbButtons()
    if not mbPanel then return end
    local idx = 0
    for _, child in ipairs(mbPanel:GetChildren()) do
        if child:IsA("TextButton") then
            local row = math.floor(idx / MB_COLS) + 1
            local col = (idx % MB_COLS) + 1
            local x = MB_PAD + (col-1) * (MB_BTN_W + MB_PAD)
            local y = MB_PAD + (row-1) * (MB_BTN_H + MB_PAD)
            child.Size = UDim2.new(0, MB_BTN_W, 0, MB_BTN_H)
            child.Position = UDim2.new(0, x, 0, y)
            idx = idx + 1
        end
    end
end

local function applyMbBtnSize(w, h2)
    w = math.clamp(w, 40, 120); h2 = math.clamp(h2, 36, 120)
    MB_BTN_W = w; MB_BTN_H = h2
    MB_TOTAL_W = MB_COLS * MB_BTN_W + (MB_COLS + 1) * MB_PAD
    MB_TOTAL_H = MB_ROWS * MB_BTN_H + (MB_ROWS + 1) * MB_PAD
    if mbPanel then mbPanel.Size = UDim2.new(0, MB_TOTAL_W, 0, MB_TOTAL_H); repositionMbButtons() end
end

-- ========== ANIMATIONS ==========
local Anims = {idle1="rbxassetid://133806214992291",idle2="rbxassetid://94970088341563",walk="rbxassetid://707897309",run="rbxassetid://707861613",jump="rbxassetid://116936326516985",fall="rbxassetid://116936326516985",climb="rbxassetid://116936326516985",swim="rbxassetid://116936326516985",swimidle="rbxassetid://116936326516985"}
task.spawn(function() pcall(function() ContentProvider:PreloadAsync({Anims.idle1,Anims.idle2,Anims.walk,Anims.run,Anims.jump,Anims.fall,Anims.climb,Anims.swim,Anims.swimidle}) end) end)
local animHeartbeatConn=nil; local savedAnimate=nil; local originalAnims=nil
local function isPackAnim(id) if not id then return false end; for _,v in pairs(Anims) do if v==id then return true end end; return false end
local function saveOriginalAnims(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function g(obj) return obj and obj.AnimationId or nil end; local ids={idle1=g(animate.idle and animate.idle.Animation1),idle2=g(animate.idle and animate.idle.Animation2),walk=g(animate.walk and animate.walk.WalkAnim),run=g(animate.run and animate.run.RunAnim),jump=g(animate.jump and animate.jump.JumpAnim),fall=g(animate.fall and animate.fall.FallAnim),climb=g(animate.climb and animate.climb.ClimbAnim),swim=g(animate.swim and animate.swim.Swim),swimidle=g(animate.swimidle and animate.swimidle.SwimIdle)}; if not isPackAnim(ids.walk) then originalAnims=ids end end
local function applyAnimPack(char) local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,Anims.idle1); s(animate.idle and animate.idle.Animation2,Anims.idle2); s(animate.walk and animate.walk.WalkAnim,Anims.walk); s(animate.run and animate.run.RunAnim,Anims.run); s(animate.jump and animate.jump.JumpAnim,Anims.jump); s(animate.fall and animate.fall.FallAnim,Anims.fall); s(animate.climb and animate.climb.ClimbAnim,Anims.climb); s(animate.swim and animate.swim.Swim,Anims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,Anims.swimidle) end
local function restoreOriginalAnims(char) if not originalAnims then return end; local animate=char:FindFirstChild("Animate"); if not animate then return end; local function s(obj,id) if obj and id then obj.AnimationId=id end end; s(animate.idle and animate.idle.Animation1,originalAnims.idle1); s(animate.idle and animate.idle.Animation2,originalAnims.idle2); s(animate.walk and animate.walk.WalkAnim,originalAnims.walk); s(animate.run and animate.run.RunAnim,originalAnims.run); s(animate.jump and animate.jump.JumpAnim,originalAnims.jump); s(animate.fall and animate.fall.FallAnim,originalAnims.fall); s(animate.climb and animate.climb.ClimbAnim,originalAnims.climb); s(animate.swim and animate.swim.Swim,originalAnims.swim); s(animate.swimidle and animate.swimidle.SwimIdle,originalAnims.swimidle); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end
local function startAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then saveOriginalAnims(char); applyAnimPack(char); local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end; hum2:ChangeState(Enum.HumanoidStateType.Running) end end; animHeartbeatConn=RunService.Heartbeat:Connect(function() if not State.animEnabled then return end; local c=LP.Character; if c then applyAnimPack(c) end end) end
local function stopAnimToggle() if animHeartbeatConn then animHeartbeatConn:Disconnect(); animHeartbeatConn=nil end; local char=LP.Character; if char then restoreOriginalAnims(char) end end
local function startUnwalk() if State.unwalkEnabled then return end; State.unwalkEnabled=true; local c=LP.Character; if not c then return end; local hum=c:FindFirstChildOfClass("Humanoid"); if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end; local anim=c:FindFirstChild("Animate"); if anim then savedAnimate=anim:Clone(); anim:Destroy() end end
local function stopUnwalk() if not State.unwalkEnabled then return end; State.unwalkEnabled=false; local c=LP.Character; if c and savedAnimate then savedAnimate.Parent=c; savedAnimate.Disabled=false; savedAnimate=nil end; task.spawn(function() task.wait(0.15); local char=LP.Character; if not char then return end; if State.animEnabled then saveOriginalAnims(char); applyAnimPack(char) else restoreOriginalAnims(char) end end) end

-- ========== MAIN GUI ==========
local gui = Instance.new("ScreenGui")
gui.Name="OmniHubGUI"; gui.ResetOnSpawn=false; gui.DisplayOrder=10; gui.IgnoreGuiInset=true; gui.Parent=LP:WaitForChild("PlayerGui")

local stealProgressBar = Instance.new("Frame", gui)
stealProgressBar.Size=UDim2.new(0,280,0,6); stealProgressBar.Position=UDim2.new(0.5,-140,0.9,0)
stealProgressBar.BackgroundColor3=Color3.fromRGB(8,12,25); stealProgressBar.BackgroundTransparency=0; stealProgressBar.BorderSizePixel=0; stealProgressBar.ZIndex=100
Instance.new("UICorner",stealProgressBar).CornerRadius=UDim.new(1,0)
local barStroke=Instance.new("UIStroke",stealProgressBar); barStroke.Color=Theme.AccentDim; barStroke.Thickness=1
local barFill=Instance.new("Frame",stealProgressBar); barFill.Size=UDim2.new(0,0,1,0); barFill.BackgroundColor3=Theme.Accent; barFill.BorderSizePixel=0
Instance.new("UICorner",barFill).CornerRadius=UDim.new(1,0); AutoSteal.ProgressFill=barFill

local function makeStealBarDraggable(frame)
    local dragging=false; local dragInput,dragStart,startPos,activeDragConn
    local function stopDrag() dragging=false; dragInput=nil; dragStart=nil; startPos=nil; if activeDragConn then activeDragConn:Disconnect(); activeDragConn=nil end end
    frame.InputBegan:Connect(function(input) if MobileButtons.Locked then return end; if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=input.Position; startPos=frame.Position; input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End or input.UserInputState==Enum.UserInputState.Cancelled then stopDrag() end end) end end)
    frame.InputChanged:Connect(function(input) if not dragging then return end; if MobileButtons.Locked then stopDrag(); return end; if input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch then dragInput=input end end)
    UIS.InputChanged:Connect(function(input) if input==dragInput and dragging then if MobileButtons.Locked then stopDrag(); return end; local delta=input.Position-dragStart; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y) end end)
end
makeStealBarDraggable(stealProgressBar)
local function saveStealBarPosition() local pos=stealProgressBar.Position; pcall(function() writefile("JispiStealBarPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function loadStealBarPosition() local savedPos=nil; pcall(function() savedPos=readfile("JispiStealBarPos.txt") end); if savedPos and savedPos~="" then local parts={}; for part in string.gmatch(savedPos,"[^,]+") do table.insert(parts,part) end; if #parts>=4 then stealProgressBar.Position=UDim2.new(tonumber(parts[1]),tonumber(parts[2]),tonumber(parts[3]),tonumber(parts[4])) end end end
loadStealBarPosition()
local lastStealBarSave=0; stealProgressBar:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastStealBarSave>0.5 then lastStealBarSave=tick(); saveStealBarPosition() end end)

-- ========== MAIN FRAME ==========
main = Instance.new("Frame", gui)
main.Name="Main"; main.Size=UDim2.new(0,320,0,470); main.Position=UDim2.new(0,20,0,20)
main.BackgroundColor3=Theme.Background; main.BorderSizePixel=0; main.Active=true; main.ClipsDescendants=true
Instance.new("UICorner",main).CornerRadius=UDim.new(0,16)
local outerGlow=Instance.new("UIStroke",main); outerGlow.Color=Theme.Accent; outerGlow.Thickness=1.5; outerGlow.Transparency=0.3
local glowPulse=Instance.new("Frame",main); glowPulse.Size=UDim2.new(1,8,1,8); glowPulse.Position=UDim2.new(0,-4,0,-4)
glowPulse.BackgroundColor3=Theme.Accent; glowPulse.BackgroundTransparency=0.88; glowPulse.BorderSizePixel=0; glowPulse.ZIndex=-1
Instance.new("UICorner",glowPulse).CornerRadius=UDim.new(0,20)
task.spawn(function()
    while true do
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.78}):Play()
        task.wait(1.8)
        TweenService:Create(glowPulse,TweenInfo.new(1.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),{BackgroundTransparency=0.92}):Play()
        task.wait(1.8)
    end
end)

local function saveMainPosition() local pos=main.Position; pcall(function() writefile("JispiHubGUIPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end
local function saveMiniPosition() if miniBtn then local pos=miniBtn.Position; pcall(function() writefile("JispiMiniPos.txt",string.format("%.3f,%.1f,%.3f,%.1f",pos.X.Scale,pos.X.Offset,pos.Y.Scale,pos.Y.Offset)) end) end end
local function loadMainPosition() local sp=nil; pcall(function() sp=readfile("JispiHubGUIPos.txt") end); if sp and sp~="" then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then main.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])); if closeBtn then closeBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2])+320-34,tonumber(p[3]),tonumber(p[4])+18) end end end end
local function loadMiniPosition() local sp=nil; pcall(function() sp=readfile("JispiMiniPos.txt") end); if sp and sp~="" and miniBtn then local p={}; for part in string.gmatch(sp,"[^,]+") do table.insert(p,part) end; if #p>=4 then miniBtn.Position=UDim2.new(tonumber(p[1]),tonumber(p[2]),tonumber(p[3]),tonumber(p[4])) end end end

-- ========== HEADER ==========
local header=Instance.new("Frame",main)
header.Size=UDim2.new(1,0,0,52); header.BackgroundColor3=Theme.Header; header.BorderSizePixel=0; header.ZIndex=5
Instance.new("UICorner",header).CornerRadius=UDim.new(0,16)
local headerAccentLine=Instance.new("Frame",header)
headerAccentLine.Size=UDim2.new(1,0,0,2); headerAccentLine.Position=UDim2.new(0,0,1,-2)
headerAccentLine.BackgroundColor3=Theme.Accent; headerAccentLine.BorderSizePixel=0; headerAccentLine.ZIndex=6
local lineGlow=Instance.new("UIGradient",headerAccentLine)
lineGlow.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,30)),ColorSequenceKeypoint.new(0.5,Theme.AccentGlow),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,30))})
local titleLbl=Instance.new("TextLabel",header)
titleLbl.Size=UDim2.new(1,0,1,-4); titleLbl.Position=UDim2.new(0,0,0,0)
titleLbl.BackgroundTransparency=1; titleLbl.Text="◈  RBG HUB  ◈"
titleLbl.TextColor3=Theme.AccentGlow; titleLbl.Font=Theme.FontBlack; titleLbl.TextSize=17
titleLbl.TextXAlignment=Enum.TextXAlignment.Center; titleLbl.ZIndex=6
local versionLbl=Instance.new("TextLabel",header)
versionLbl.Size=UDim2.new(0,40,0,16); versionLbl.Position=UDim2.new(1,-50,0.5,-8)
versionLbl.BackgroundTransparency=1; versionLbl.Text="v1.0"; versionLbl.TextColor3=Theme.AccentDim
versionLbl.Font=Theme.Font; versionLbl.TextSize=9; versionLbl.ZIndex=6

-- ========== CLOSE / MINI ==========
closeBtn=Instance.new("TextButton",gui)
closeBtn.Size=UDim2.new(0,28,0,28); closeBtn.BackgroundColor3=Color3.fromRGB(14,18,35)
closeBtn.BorderSizePixel=0; closeBtn.Text="✕"; closeBtn.TextColor3=Theme.SubText
closeBtn.Font=Theme.FontBold; closeBtn.TextSize=14; closeBtn.ZIndex=50
Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(0,8)
local closeStroke=Instance.new("UIStroke",closeBtn); closeStroke.Color=Theme.Border; closeStroke.Thickness=1.5
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Danger,TextColor3=Color3.new(1,1,1)}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Danger}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.15),{BackgroundColor3=Color3.fromRGB(14,18,35),TextColor3=Theme.SubText}):Play(); TweenService:Create(closeStroke,TweenInfo.new(0.15),{Color=Theme.Border}):Play() end)
closeBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)

miniBtn=Instance.new("ImageButton",gui); miniBtn.Name="OmniMiniButton"; miniBtn.Size=UDim2.new(0,44,0,44)
miniBtn.Position=UDim2.new(0,20,0,100); miniBtn.BackgroundColor3=Theme.Background; miniBtn.Image=""; miniBtn.BorderSizePixel=0; miniBtn.Visible=false; miniBtn.ZIndex=50
Instance.new("UICorner",miniBtn).CornerRadius=UDim.new(0,12)
local miniStroke=Instance.new("UIStroke",miniBtn); miniStroke.Color=Theme.Accent; miniStroke.Thickness=1.8
local miniIcon=Instance.new("TextLabel",miniBtn); miniIcon.Size=UDim2.new(1,0,1,0); miniIcon.Text="◈"; miniIcon.TextColor3=Theme.AccentGlow; miniIcon.Font=Theme.FontBlack; miniIcon.TextSize=20; miniIcon.BackgroundTransparency=1
miniBtn.MouseButton1Click:Connect(function() toggleGuiVis() end)
miniBtn.MouseEnter:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Element}):Play() end)
miniBtn.MouseLeave:Connect(function() TweenService:Create(miniBtn,TweenInfo.new(0.15),{BackgroundColor3=Theme.Background}):Play() end)

local function makeMainDraggable(frame)
    local dragging,dragInput,dragStart,startPos,startCloseBtnPos=false,nil,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; if closeBtn then startCloseBtnPos=closeBtn.Position end; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy); if closeBtn and startCloseBtnPos then closeBtn.Position=UDim2.new(startCloseBtnPos.X.Scale,startCloseBtnPos.X.Offset+dx,startCloseBtnPos.Y.Scale,startCloseBtnPos.Y.Offset+dy) end end end)
end
local function makeMiniDraggable(frame)
    local dragging,dragInput,dragStart,startPos=false,nil,nil,nil
    frame.InputBegan:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then dragging=true; dragStart=inp.Position; startPos=frame.Position; inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End or inp.UserInputState==Enum.UserInputState.Cancelled then dragging=false end end) end end)
    frame.InputChanged:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end end)
    UIS.InputChanged:Connect(function(inp) if inp==dragInput and dragging then local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y; frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy) end end)
end
makeMainDraggable(main); makeMiniDraggable(miniBtn)
local lastSaveTime=0; main:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastSaveTime>0.5 then lastSaveTime=tick(); saveMainPosition() end end)
local lastMiniSaveTime=0; miniBtn:GetPropertyChangedSignal("Position"):Connect(function() if tick()-lastMiniSaveTime>0.5 then lastMiniSaveTime=tick(); saveMiniPosition() end end)
local function updateCloseButtonPosition() if main.Visible then local p=main.Position; closeBtn.Position=UDim2.new(p.X.Scale,p.X.Offset+320-34,p.Y.Scale,p.Y.Offset+18) end end
main:GetPropertyChangedSignal("Position"):Connect(updateCloseButtonPosition); main:GetPropertyChangedSignal("Visible"):Connect(updateCloseButtonPosition)

-- ========== TAB SYSTEM ==========
local tabFrame=Instance.new("Frame",main)
tabFrame.Size=UDim2.new(1,-16,0,30); tabFrame.Position=UDim2.new(0,8,0,56)
tabFrame.BackgroundTransparency=1; tabFrame.BorderSizePixel=0; tabFrame.ZIndex=3
local tabLayout=Instance.new("UIListLayout",tabFrame)
tabLayout.FillDirection=Enum.FillDirection.Horizontal; tabLayout.SortOrder=Enum.SortOrder.LayoutOrder
tabLayout.Padding=UDim.new(0,4); tabLayout.HorizontalAlignment=Enum.HorizontalAlignment.Center

local scroll=Instance.new("ScrollingFrame",main)
scroll.Size=UDim2.new(1,-16,1,-94); scroll.Position=UDim2.new(0,8,0,90)
scroll.BackgroundTransparency=1; scroll.BorderSizePixel=0; scroll.ScrollBarThickness=3
scroll.ScrollBarImageColor3=Theme.Accent; scroll.ScrollBarImageTransparency=0.4
scroll.AutomaticCanvasSize=Enum.AutomaticSize.Y; scroll.CanvasSize=UDim2.new(0,0,0,0); scroll.ZIndex=2
local listLayout=Instance.new("UIListLayout",scroll); listLayout.SortOrder=Enum.SortOrder.LayoutOrder; listLayout.Padding=UDim.new(0,5)
local pad=Instance.new("UIPadding",scroll); pad.PaddingLeft=UDim.new(0,0); pad.PaddingRight=UDim.new(0,0); pad.PaddingTop=UDim.new(0,2); pad.PaddingBottom=UDim.new(0,8)

local tabDefs = {{"Speed",1},{"Features",2},{"Settings",3}}
local tabBtns = {}; local tabPages = {}; local activeTab = "Speed"

for _,td in ipairs(tabDefs) do
    local btn=Instance.new("TextButton",tabFrame)
    btn.Size=UDim2.new(0,80,0,26); btn.BackgroundColor3=(td[1]==activeTab) and Theme.AccentDark or Theme.Element
    btn.BorderSizePixel=0; btn.Text=td[1]:upper(); btn.TextColor3=(td[1]==activeTab) and Theme.AccentGlow or Theme.SubText
    btn.Font=Theme.FontBold; btn.TextSize=9; btn.LayoutOrder=td[2]; btn.ZIndex=4; btn.AutoButtonColor=false
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,8)
    local ts=Instance.new("UIStroke",btn); ts.Color=(td[1]==activeTab) and Theme.Accent or Theme.Border; ts.Thickness=(td[1]==activeTab) and 1.5 or 1
    tabBtns[td[1]]={btn=btn,stroke=ts}
    local page=Instance.new("Frame",scroll)
    page.Size=UDim2.new(1,0,0,0); page.BackgroundTransparency=1; page.BorderSizePixel=0
    page.Visible=(td[1]==activeTab); page.ZIndex=2
    page.AutomaticSize=Enum.AutomaticSize.Y
    local pageLL=Instance.new("UIListLayout",page); pageLL.SortOrder=Enum.SortOrder.LayoutOrder; pageLL.Padding=UDim.new(0,5)
    local pagePad=Instance.new("UIPadding",page); pagePad.PaddingBottom=UDim.new(0,4)
    tabPages[td[1]]=page
    local pageLO=0
    btn.MouseButton1Click:Connect(function()
        if activeTab==td[1] then return end
        activeTab=td[1]
        for name,data in pairs(tabBtns) do
            local isA=(name==activeTab)
            TweenService:Create(data.btn,TweenInfo.new(0.15),{BackgroundColor3=isA and Theme.AccentDark or Theme.Element}):Play()
            TweenService:Create(data.btn,TweenInfo.new(0.15),{TextColor3=isA and Theme.AccentGlow or Theme.SubText}):Play()
            TweenService:Create(data.stroke,TweenInfo.new(0.15),{Color=isA and Theme.Accent or Theme.Border,Thickness=isA and 1.5 or 1}):Play()
            tabPages[name].Visible=isA
        end
    end)
    btn.MouseEnter:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end
    end)
    btn.MouseLeave:Connect(function()
        if activeTab~=td[1] then TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=Theme.Element}):Play() end
    end)
end

-- ========== UI HELPERS (per-page) ==========
local pageLOs = {Speed=0, Features=0, Settings=0}
local function LO(tab) pageLOs[tab]=(pageLOs[tab] or 0)+1; return pageLOs[tab] end
local function pg(tab) return tabPages[tab] end

local function makeGap(tab,px) local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,0,0,px or 3); f.BackgroundTransparency=1; f.LayoutOrder=LO(tab) end
local function makeDivider(tab)
    local f=Instance.new("Frame",pg(tab)); f.Size=UDim2.new(1,-8,0,1); f.BackgroundColor3=Theme.Border; f.BackgroundTransparency=0; f.BorderSizePixel=0; f.LayoutOrder=LO(tab)
    local g=Instance.new("UIGradient",f); g.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,20)),ColorSequenceKeypoint.new(0.5,Theme.Accent),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,20))})
end
local function makeSectionLabel(tab,text)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,26); row.BackgroundTransparency=1; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-8,1,0); lbl.BackgroundTransparency=1
    lbl.Text="  ◆ "..text:upper(); lbl.TextColor3=Theme.Accent; lbl.Font=Theme.FontBold; lbl.TextSize=10; lbl.TextXAlignment=Enum.TextXAlignment.Left
end

local function makeInputRow(tab,label,default,onChange)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0,100,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.ZIndex=2
    local box=Instance.new("TextBox",row); box.Size=UDim2.new(0,60,0,26); box.Position=UDim2.new(1,-70,0.5,-13); box.BackgroundColor3=Theme.BackgroundSecondary; box.BorderSizePixel=0; box.Text=tostring(default); box.TextColor3=Theme.AccentGlow; box.Font=Theme.FontBold; box.TextSize=13; box.ClearTextOnFocus=false; box.ZIndex=3
    Instance.new("UICorner",box).CornerRadius=UDim.new(0,8)
    local boxStroke=Instance.new("UIStroke",box); boxStroke.Color=Theme.AccentDim; boxStroke.Thickness=1
    box.Focused:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.Element}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
    box.FocusLost:Connect(function() TweenService:Create(box,TweenInfo.new(0.2),{BackgroundColor3=Theme.BackgroundSecondary}):Play(); TweenService:Create(boxStroke,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); local num=tonumber(box.Text); if num~=nil then local fv=math.floor(math.clamp(num,0,500)); box.Text=tostring(fv); if onChange then onChange(tostring(fv)) end; autoSaveConfig() else box.Text=tostring(default) end end)
    return box
end

local function makeStatusRow(tab,label,valTxt)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,32); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(0.5,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local val=Instance.new("TextLabel",row); val.Size=UDim2.new(0.45,-10,1,0); val.Position=UDim2.new(0.52,0,0,0); val.BackgroundTransparency=1; val.Text=valTxt; val.TextColor3=Theme.AccentGlow; val.Font=Theme.FontBold; val.TextSize=12; val.TextXAlignment=Enum.TextXAlignment.Right
    return val
end

local function makeKeybindRow(tab,label,currentKey,onChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,36); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",row).Color=Theme.Border
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,0,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local btn=Instance.new("TextButton",row); btn.Size=UDim2.new(0,70,0,24); btn.Position=UDim2.new(1,-80,0.5,-12); btn.BackgroundColor3=Theme.Keybind; btn.BorderSizePixel=0; btn.Text="["..currentKey.Name.."]"; btn.TextColor3=Theme.Accent; btn.Font=Theme.FontBold; btn.TextSize=10
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,7)
    local bs=Instance.new("UIStroke",btn); bs.Color=Theme.AccentDim; bs.Thickness=1
    local listening=false; local listenConn
    local function stopListen(key)
        listening=false; if listenConn then listenConn:Disconnect(); listenConn=nil end
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(btn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); btn.TextColor3=Theme.Accent
        if key then btn.Text="["..key.Name.."]"; if onChanged then onChanged(key) end; autoSaveConfig() end
    end
    btn.MouseButton1Click:Connect(function()
        if listening then stopListen(nil); return end
        listening=true; btn.Text="[···]"; btn.TextColor3=Theme.AccentGlow; btn.BackgroundColor3=Theme.Element
        TweenService:Create(bs,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
        listenConn=UIS.InputBegan:Connect(function(inp,gp)
            if not listening then return end
            if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode)
            elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then stopListen(nil); return end; stopListen(inp.KeyCode) end
        end)
    end)
    return btn
end

local function makeToggleRow(tab,label,defaultKey,defaultOn,onToggle,onKeyChanged)
    local row=Instance.new("Frame",pg(tab)); row.Size=UDim2.new(1,0,0,40); row.BackgroundColor3=Theme.Element; row.BorderSizePixel=0; row.LayoutOrder=LO(tab)
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
    local rowStroke=Instance.new("UIStroke",row); rowStroke.Color=Theme.Border; rowStroke.Thickness=1
    local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-120,1,0); lbl.Position=UDim2.new(0,12,0,0); lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Theme.Text; lbl.Font=Theme.FontMedium; lbl.TextSize=12; lbl.TextXAlignment=Enum.TextXAlignment.Left
    local keyBtn=nil
    if defaultKey then
        keyBtn=Instance.new("TextButton",row); keyBtn.Size=UDim2.new(0,50,0,22); keyBtn.Position=UDim2.new(1,-108,0.5,-11); keyBtn.BackgroundColor3=Theme.Keybind; keyBtn.BorderSizePixel=0; keyBtn.Text=defaultKey.Name; keyBtn.TextColor3=Theme.Accent; keyBtn.Font=Theme.FontBold; keyBtn.TextSize=9; keyBtn.ZIndex=5
        Instance.new("UICorner",keyBtn).CornerRadius=UDim.new(0,5)
        local ks=Instance.new("UIStroke",keyBtn); ks.Color=Theme.AccentDim; ks.Thickness=1
        local kListening=false; local kConn
        local function kStop(key) kListening=false; if kConn then kConn:Disconnect(); kConn=nil end; TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.AccentDim}):Play(); TweenService:Create(keyBtn,TweenInfo.new(0.2),{BackgroundColor3=Theme.Keybind}):Play(); keyBtn.TextColor3=Theme.Accent; if key then keyBtn.Text=key.Name; if onKeyChanged then onKeyChanged(key) end; autoSaveConfig() end end
        keyBtn.MouseButton1Click:Connect(function()
            if kListening then kStop(nil); return end
            kListening=true; keyBtn.Text="···"; keyBtn.TextColor3=Theme.AccentGlow; keyBtn.BackgroundColor3=Theme.Element
            TweenService:Create(ks,TweenInfo.new(0.2),{Color=Theme.Accent}):Play()
            kConn=UIS.InputBegan:Connect(function(inp,gp)
                if not kListening then return end
                if inp.UserInputType==Enum.UserInputType.Keyboard then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode)
                elseif inp.UserInputType==Enum.UserInputType.Gamepad1 then if inp.KeyCode==Enum.KeyCode.Escape then kStop(nil); return end; kStop(inp.KeyCode) end
            end)
        end)
    end
    local toggleBg=Instance.new("TextButton",row); toggleBg.Size=UDim2.new(0,42,0,22); toggleBg.Position=UDim2.new(1,-48,0.5,-11)
    toggleBg.BackgroundColor3=defaultOn and Theme.AccentDark or Theme.ToggleOff; toggleBg.BorderSizePixel=0; toggleBg.Text=""; toggleBg.ZIndex=5; toggleBg.AutoButtonColor=false
    Instance.new("UICorner",toggleBg).CornerRadius=UDim.new(1,0)
    local tStroke=Instance.new("UIStroke",toggleBg); tStroke.Color=defaultOn and Theme.Accent or Theme.Border; tStroke.Thickness=1.5
    local knob=Instance.new("Frame",toggleBg); knob.Size=UDim2.new(0,17,0,17); knob.Position=defaultOn and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5)
    knob.BackgroundColor3=defaultOn and Theme.AccentGlow or Theme.SubText; knob.BorderSizePixel=0; knob.ZIndex=6
    Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
    local isOn=defaultOn or false
    local function setV(on)
        isOn=on
        TweenService:Create(toggleBg,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{BackgroundColor3=on and Theme.AccentDark or Theme.ToggleOff}):Play()
        TweenService:Create(tStroke,TweenInfo.new(0.22),{Color=on and Theme.Accent or Theme.Border}):Play()
        TweenService:Create(knob,TweenInfo.new(0.22,Enum.EasingStyle.Quart),{Position=on and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=on and Theme.AccentGlow or Theme.SubText}):Play()
        TweenService:Create(rowStroke,TweenInfo.new(0.22),{Color=on and Theme.AccentDim or Theme.Border,Thickness=on and 1.5 or 1}):Play()
    end
    toggleBg.MouseButton1Click:Connect(function() isOn=not isOn; setV(isOn); if onToggle then pcall(onToggle,isOn) end; autoSaveConfig() end)
    toggleBg.MouseEnter:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
    toggleBg.MouseLeave:Connect(function() TweenService:Create(toggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)
    return setV, keyBtn
end

-- ===================== BUILD TABS =====================

-- ===== SPEED TAB =====
makeSectionLabel("Speed","Speed")
normalBox=makeInputRow("Speed","Normal",State.normalSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.normalSpeed=n end end)
carryBox=makeInputRow("Speed","Carry",State.carrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.carrySpeed=n end end)
setSpeedToggleUI,speedKeyBtn=makeToggleRow("Speed","Toggle",Keys.speed,false,function(on) toggleSpeedType() end,function(k) Keys.speed=k end)
modeValLbl=makeStatusRow("Speed","Mode","Normal")
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Lagger Speed")
laggerBox=makeInputRow("Speed","Normal",State.laggerSpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerSpeed=n end end)
carryLaggerBox=makeInputRow("Speed","Carry",State.laggerCarrySpeed,function(v) local n=tonumber(v); if n and n>0 and n<=500 then State.laggerCarrySpeed=n end end)
setLaggerToggleUI,laggerKeyBtn=makeToggleRow("Speed","Toggle",Keys.lagger,false,function(on) toggleLagger() end,function(k) Keys.lagger=k end)
makeGap("Speed",2); makeDivider("Speed"); makeGap("Speed",2)

makeSectionLabel("Speed","Combat")
setAutoBat,autoBatKeyBtn=makeToggleRow("Speed","Bat Aimbot",Keys.autoBat,false,function(on) State.autoBatToggled=on; if not on then resetBend() end end,function(k) Keys.autoBat=k end)
makeGap("Speed",4)

-- ===== FEATURES TAB =====
makeSectionLabel("Features","Auto Steal")
local radiusBox=makeInputRow("Features","Grab Rad",AutoSteal.Radius,function(v) local n=tonumber(v); if n and n>=5 and n<=300 then AutoSteal.Radius=math.floor(n); if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end end)
setInstaGrab=makeToggleRow("Features","Auto Grab",nil,false,function(on) AutoSteal.Enabled=on; if on then startAutoSteal() else stopAutoSteal() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Active Toggles")
setInfJump=makeToggleRow("Features","Infinite Jump",nil,false,function(on) State.infJumpEnabled=on end)
setAntiRag=makeToggleRow("Features","Anti Ragdoll",nil,false,function(on) State.antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
setFps=makeToggleRow("Features","FPS Boost",nil,false,function(on) State.fpsBoostEnabled=on; if on then pcall(applyFPSBoost) end end)
setMedusaCounter=makeToggleRow("Features","Medusa Counter",nil,false,function(on) State.medusaCounterEnabled=on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
setAnimToggle=makeToggleRow("Features","Tryhard Anim",nil,false,function(on) State.animEnabled=on; if on then startAnimToggle() else stopAnimToggle() end end)
setUnwalkToggle=makeToggleRow("Features","Unwalk",nil,false,function(on) if on then startUnwalk() else stopUnwalk() end end)
makeGap("Features",2); makeDivider("Features"); makeGap("Features",2)

makeSectionLabel("Features","Movement")
tpDownKeyBtn=makeKeybindRow("Features","TP Down",Keys.tpDown,function(k) Keys.tpDown=k end)
dropBrainrotKeyBtn=makeKeybindRow("Features","Drop Brainrot",Keys.dropBrainrot,function(k) Keys.dropBrainrot=k end)
setAutoLeft,autoLeftKeyBtn=makeToggleRow("Features","Auto Left",Keys.autoLeft,false,function(on) State.autoLeftEnabled=on; if on then startAutoLeft() else stopAutoLeft() end end,function(k) Keys.autoLeft=k end)
setAutoRight,autoRightKeyBtn=makeToggleRow("Features","Auto Right",Keys.autoRight,false,function(on) State.autoRightEnabled=on; if on then startAutoRight() else stopAutoRight() end end,function(k) Keys.autoRight=k end)
makeGap("Features",4)

-- ===== SETTINGS TAB =====
makeSectionLabel("Settings","Interface")
guiHideKeyBtn=makeKeybindRow("Settings","Hide GUI",Keys.guiHide,function(k) Keys.guiHide=k end)
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

makeSectionLabel("Settings","Mobile Panel")
local showRow=Instance.new("Frame",pg("Settings")); showRow.Size=UDim2.new(1,0,0,38); showRow.BackgroundColor3=Theme.Element; showRow.BorderSizePixel=0; showRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",showRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",showRow).Color=Theme.Border
local showLbl=Instance.new("TextLabel",showRow); showLbl.Size=UDim2.new(1,-115,1,0); showLbl.Position=UDim2.new(0,12,0,0); showLbl.BackgroundTransparency=1; showLbl.Text="Show Buttons"; showLbl.TextColor3=Theme.Text; showLbl.Font=Theme.FontMedium; showLbl.TextSize=12; showLbl.TextXAlignment=Enum.TextXAlignment.Left
local showToggleBg=Instance.new("TextButton",showRow); showToggleBg.Size=UDim2.new(0,42,0,22); showToggleBg.Position=UDim2.new(1,-48,0.5,-11); showToggleBg.BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff; showToggleBg.BorderSizePixel=0; showToggleBg.Text=""; showToggleBg.ZIndex=5; showToggleBg.AutoButtonColor=false
Instance.new("UICorner",showToggleBg).CornerRadius=UDim.new(1,0)
local showKnob=Instance.new("Frame",showToggleBg); showKnob.Size=UDim2.new(0,17,0,17); showKnob.Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); showKnob.BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText; showKnob.BorderSizePixel=0; showKnob.ZIndex=6; Instance.new("UICorner",showKnob).CornerRadius=UDim.new(1,0)
showToggleBg.MouseButton1Click:Connect(function() MobileButtons.Visible=not MobileButtons.Visible; if mbPanel then mbPanel.Visible=MobileButtons.Visible end; TweenService:Create(showToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Visible and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(showKnob,TweenInfo.new(0.22),{Position=MobileButtons.Visible and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Visible and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
showToggleBg.MouseEnter:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
showToggleBg.MouseLeave:Connect(function() TweenService:Create(showToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local lockRow=Instance.new("Frame",pg("Settings")); lockRow.Size=UDim2.new(1,0,0,38); lockRow.BackgroundColor3=Theme.Element; lockRow.BorderSizePixel=0; lockRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",lockRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",lockRow).Color=Theme.Border
local lockLbl=Instance.new("TextLabel",lockRow); lockLbl.Size=UDim2.new(1,-115,1,0); lockLbl.Position=UDim2.new(0,12,0,0); lockLbl.BackgroundTransparency=1; lockLbl.Text="Lock Buttons"; lockLbl.TextColor3=Theme.Text; lockLbl.Font=Theme.FontMedium; lockLbl.TextSize=12; lockLbl.TextXAlignment=Enum.TextXAlignment.Left
local lockToggleBg=Instance.new("TextButton",lockRow); lockToggleBg.Size=UDim2.new(0,42,0,22); lockToggleBg.Position=UDim2.new(1,-48,0.5,-11); lockToggleBg.BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff; lockToggleBg.BorderSizePixel=0; lockToggleBg.Text=""; lockToggleBg.ZIndex=5; lockToggleBg.AutoButtonColor=false
Instance.new("UICorner",lockToggleBg).CornerRadius=UDim.new(1,0)
local lockKnob=Instance.new("Frame",lockToggleBg); lockKnob.Size=UDim2.new(0,17,0,17); lockKnob.Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5); lockKnob.BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText; lockKnob.BorderSizePixel=0; lockKnob.ZIndex=6; Instance.new("UICorner",lockKnob).CornerRadius=UDim.new(1,0)
lockToggleBg.MouseButton1Click:Connect(function() MobileButtons.Locked=not MobileButtons.Locked; TweenService:Create(lockToggleBg,TweenInfo.new(0.22),{BackgroundColor3=MobileButtons.Locked and Theme.AccentDark or Theme.ToggleOff}):Play(); TweenService:Create(lockKnob,TweenInfo.new(0.22),{Position=MobileButtons.Locked and UDim2.new(1,-19,0.5,-8.5) or UDim2.new(0,2,0.5,-8.5),BackgroundColor3=MobileButtons.Locked and Theme.AccentGlow or Theme.SubText}):Play(); autoSaveConfig() end)
lockToggleBg.MouseEnter:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,46,0,26),Position=UDim2.new(1,-50,0.5,-13)}):Play() end)
lockToggleBg.MouseLeave:Connect(function() TweenService:Create(lockToggleBg,TweenInfo.new(0.15),{Size=UDim2.new(0,42,0,22),Position=UDim2.new(1,-48,0.5,-11)}):Play() end)

local sizeRow=Instance.new("Frame",pg("Settings")); sizeRow.Size=UDim2.new(1,0,0,38); sizeRow.BackgroundColor3=Theme.Element; sizeRow.BorderSizePixel=0; sizeRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",sizeRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",sizeRow).Color=Theme.Border
local sizeLbl=Instance.new("TextLabel",sizeRow); sizeLbl.Size=UDim2.new(0,100,1,0); sizeLbl.Position=UDim2.new(0,12,0,0); sizeLbl.BackgroundTransparency=1; sizeLbl.Text="Btn Size"; sizeLbl.TextColor3=Theme.Text; sizeLbl.Font=Theme.FontMedium; sizeLbl.TextSize=12; sizeLbl.TextXAlignment=Enum.TextXAlignment.Left
local sizeValLbl=Instance.new("TextLabel",sizeRow); sizeValLbl.Size=UDim2.new(0,40,0,16); sizeValLbl.Position=UDim2.new(0,112,0.5,-8); sizeValLbl.BackgroundTransparency=1; sizeValLbl.Text=MB_BTN_W.."px"; sizeValLbl.TextColor3=Theme.AccentGlow; sizeValLbl.Font=Theme.FontBold; sizeValLbl.TextSize=9; sizeValLbl.TextXAlignment=Enum.TextXAlignment.Center
local decBtn=Instance.new("TextButton",sizeRow); decBtn.Size=UDim2.new(0,24,0,24); decBtn.Position=UDim2.new(1,-78,0.5,-12); decBtn.BackgroundColor3=Theme.BackgroundSecondary; decBtn.BorderSizePixel=0; decBtn.Text="-"; decBtn.TextColor3=Theme.Text; decBtn.Font=Theme.FontBlack; decBtn.TextSize=14; decBtn.ZIndex=5
Instance.new("UICorner",decBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",decBtn).Color=Theme.Border
local incBtn=Instance.new("TextButton",sizeRow); incBtn.Size=UDim2.new(0,24,0,24); incBtn.Position=UDim2.new(1,-48,0.5,-12); incBtn.BackgroundColor3=Theme.BackgroundSecondary; incBtn.BorderSizePixel=0; incBtn.Text="+"; incBtn.TextColor3=Theme.Text; incBtn.Font=Theme.FontBlack; incBtn.TextSize=14; incBtn.ZIndex=5
Instance.new("UICorner",incBtn).CornerRadius=UDim.new(0,6); Instance.new("UIStroke",incBtn).Color=Theme.Border
decBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W-4,MB_BTN_H-3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
incBtn.MouseButton1Click:Connect(function() applyMbBtnSize(MB_BTN_W+4,MB_BTN_H+3); sizeValLbl.Text=MB_BTN_W.."px"; autoSaveConfig() end)
for _,b in pairs({decBtn,incBtn}) do
    b.MouseEnter:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
    b.MouseLeave:Connect(function() TweenService:Create(b,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
end
makeGap("Settings",2); makeDivider("Settings"); makeGap("Settings",2)

local saveConfigRow=Instance.new("Frame",pg("Settings")); saveConfigRow.Size=UDim2.new(1,0,0,40); saveConfigRow.BackgroundColor3=Theme.Element; saveConfigRow.BorderSizePixel=0; saveConfigRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",saveConfigRow).CornerRadius=UDim.new(0,10)
local saveConfigBtn=Instance.new("TextButton",saveConfigRow); saveConfigBtn.Size=UDim2.new(1,-16,0,28); saveConfigBtn.Position=UDim2.new(0,8,0.5,-14)
saveConfigBtn.BackgroundColor3=Color3.fromRGB(0,40,80); saveConfigBtn.BorderSizePixel=0; saveConfigBtn.Text="⬡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; saveConfigBtn.Font=Theme.FontBold; saveConfigBtn.TextSize=12; saveConfigBtn.ZIndex=5; saveConfigBtn.AutoButtonColor=false
Instance.new("UICorner",saveConfigBtn).CornerRadius=UDim.new(0,9)
local saveBtnStroke=Instance.new("UIStroke",saveConfigBtn); saveBtnStroke.Color=Theme.Accent; saveBtnStroke.Thickness=1.5
saveConfigBtn.MouseEnter:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,60,120)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.AccentGlow}):Play() end)
saveConfigBtn.MouseLeave:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.2),{BackgroundColor3=Color3.fromRGB(0,40,80)}):Play(); TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)
saveConfigBtn.MouseButton1Click:Connect(function() forceSaveConfig(); saveConfigBtn.Text="✓  SAVED!"; saveConfigBtn.TextColor3=Theme.Success; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Success}):Play(); task.wait(1.2); saveConfigBtn.Text="⬡  SAVE CONFIG"; saveConfigBtn.TextColor3=Theme.AccentGlow; TweenService:Create(saveBtnStroke,TweenInfo.new(0.2),{Color=Theme.Accent}):Play() end)

local resetMbRow=Instance.new("Frame",pg("Settings")); resetMbRow.Size=UDim2.new(1,0,0,36); resetMbRow.BackgroundColor3=Theme.Element; resetMbRow.BorderSizePixel=0; resetMbRow.LayoutOrder=LO("Settings")
Instance.new("UICorner",resetMbRow).CornerRadius=UDim.new(0,10); Instance.new("UIStroke",resetMbRow).Color=Theme.Border
local resetMbBtn=Instance.new("TextButton",resetMbRow); resetMbBtn.Size=UDim2.new(1,-16,0,24); resetMbBtn.Position=UDim2.new(0,8,0.5,-12)
resetMbBtn.BackgroundColor3=Theme.BackgroundSecondary; resetMbBtn.BorderSizePixel=0; resetMbBtn.Text="Reset Button Panel Position"; resetMbBtn.TextColor3=Theme.SubText; resetMbBtn.Font=Theme.FontMedium; resetMbBtn.TextSize=10; resetMbBtn.ZIndex=5; resetMbBtn.AutoButtonColor=false
Instance.new("UICorner",resetMbBtn).CornerRadius=UDim.new(0,7)
resetMbBtn.MouseEnter:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.ElementHover}):Play() end)
resetMbBtn.MouseLeave:Connect(function() TweenService:Create(resetMbBtn,TweenInfo.new(0.1),{BackgroundColor3=Theme.BackgroundSecondary}):Play() end)
resetMbBtn.MouseButton1Click:Connect(function() if mbPanel then mbPanel.Position=UDim2.new(1,-(MB_TOTAL_W+8),0.5,-(MB_TOTAL_H/2)) end end)

makeGap("Settings",2)
local footerLbl=Instance.new("TextLabel",pg("Settings")); footerLbl.Size=UDim2.new(1,0,0,18); footerLbl.BackgroundTransparency=1; footerLbl.LayoutOrder=LO("Settings"); footerLbl.Text="◈ RBG ◈"; footerLbl.TextColor3=Theme.AccentDim; footerLbl.Font=Theme.FontBold; footerLbl.TextSize=9; footerLbl.TextXAlignment=Enum.TextXAlignment.Center

-- ========== TOGGLE GUI VISIBILITY ==========
toggleGuiVis=function()
    State.guiVisible=not State.guiVisible
    if main then main.Visible=State.guiVisible; closeBtn.Visible=State.guiVisible; if miniBtn then miniBtn.Visible=not State.guiVisible end end
end

-- ========== LOGIC ==========
local function findMedusa()
    local char=LP.Character; if not char then return nil end
    for _,tool in ipairs(char:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end
    local bp2=LP:FindFirstChild("Backpack"); if bp2 then for _,tool in ipairs(bp2:GetChildren()) do if tool:IsA("Tool") then local tn=tool.Name:lower(); if tn:find("medusa") or tn:find("head") or tn:find("stone") then return tool end end end end
    return nil
end
local function useMedusaCounter()
    if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<25 then return end; local char=LP.Character; if not char then return end
    State.medusaDebounce=true; local med=findMedusa(); if not med then State.medusaDebounce=false; return end
    if med.Parent~=char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
end
local function onAnchorChanged(part) return part:GetPropertyChangedSignal("Anchored"):Connect(function() if part.Anchored and part.Transparency==1 then useMedusaCounter() end end) end
setupMedusaCounter=function(char) stopMedusaCounter(); if not char then return end; for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end; table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end)) end
stopMedusaCounter=function() for _,c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end; Conns.anchor={} end

startAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect() end; State.autoLeftPhase=1
    Conns.autoLeft=RunService.Heartbeat:Connect(function()
        if not State.autoLeftEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoLeftPhase==1 then
            local tgt=Vector3.new(POS.L1.X,root.Position.Y,POS.L1.Z); if (tgt-root.Position).Magnitude<1 then State.autoLeftPhase=2; return end
            local d=(POS.L1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoLeftPhase==2 then
            local tgt=Vector3.new(POS.L2.X,root.Position.Y,POS.L2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoLeftEnabled=false; if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1; if setAutoLeft then setAutoLeft(false) end; return end
            local d=(POS.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoLeft=function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect() end; State.autoRightPhase=1
    Conns.autoRight=RunService.Heartbeat:Connect(function()
        if not State.autoRightEnabled then return end; local char=LP.Character; if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart"); local hum2=char:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=getAutoMoveSpeed()
        if State.autoRightPhase==1 then
            local tgt=Vector3.new(POS.R1.X,root.Position.Y,POS.R1.Z); if (tgt-root.Position).Magnitude<1 then State.autoRightPhase=2; return end
            local d=(POS.R1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoRightPhase==2 then
            local tgt=Vector3.new(POS.R2.X,root.Position.Y,POS.R2.Z)
            if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0); State.autoRightEnabled=false; if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1; if setAutoRight then setAutoRight(false) end; return end
            local d=(POS.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
stopAutoRight=function()
    if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1
    local char=LP.Character; if char then local hum2=char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end; local root=char:FindFirstChild("HumanoidRootPart"); if root then root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0) end end
end
startAntiRagdoll=function()
    if Conns.antiRag then return end
    Conns.antiRag=RunService.Heartbeat:Connect(function()
        local char=LP.Character; if not char then return end; local hum2=char:FindFirstChildOfClass("Humanoid"); local root=char:FindFirstChild("HumanoidRootPart")
        if hum2 then local st=hum2:GetState(); if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then hum2:ChangeState(Enum.HumanoidStateType.Running); workspace.CurrentCamera.CameraSubject=hum2; pcall(function() local pm=LP.PlayerScripts:FindFirstChild("PlayerModule"); if pm then require(pm:FindFirstChild("ControlModule")):Enable() end end); if root then root.Velocity=Vector3.new(0,0,0); root.RotVelocity=Vector3.new(0,0,0) end end end
        for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled=true end end
    end)
end
stopAntiRagdoll=function() if Conns.antiRag then Conns.antiRag:Disconnect(); Conns.antiRag=nil end end
applyFPSBoost=function()
    pcall(function() setfpscap(999999999) end)
    local function processObj(v) pcall(function() if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0 elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1 elseif v:IsA("SpecialMesh") then v.TextureId="" elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false elseif v:IsA("SurfaceAppearance") or v:IsA("MaterialVariant") then v:Destroy() elseif v:IsA("Attachment") then v.Visible=false end end) end
    for _,v in pairs(workspace:GetDescendants()) do processObj(v) end
    pcall(function() local lighting=game:GetService("Lighting"); for _,v in pairs(lighting:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end; pcall(function() sethiddenproperty(game:GetService("Lighting"),"Technology",Enum.Technology.Legacy) end); local lighting2=game:GetService("Lighting"); lighting2.GlobalShadows=false; lighting2.FogEnd=9e9; lighting2.Brightness=0; local terrain=workspace:FindFirstChildOfClass("Terrain"); if terrain then pcall(function() sethiddenproperty(terrain,"Decoration",false) end); terrain.WaterReflectance=0; terrain.WaterTransparency=0.7; terrain.WaterWaveSize=0; terrain.WaterWaveSpeed=0 end end)
    workspace.DescendantAdded:Connect(function(v) if State.fpsBoostEnabled then task.spawn(processObj,v) end end)
end

-- ========== CHARACTER SETUP ==========
local function setupChar(char)
    task.wait(0.1); h=char:WaitForChild("Humanoid",5); hrp=char:WaitForChild("HumanoidRootPart",5); State.originalC0=nil
    if not h or not hrp then return end
    local head=char:FindFirstChild("Head")
    if head then
        local oldBB=head:FindFirstChild("SpeedBillboard"); if oldBB then oldBB:Destroy() end
        local bb=Instance.new("BillboardGui",head); bb.Name="SpeedBillboard"; bb.Size=UDim2.new(0,120,0,22); bb.StudsOffset=Vector3.new(0,3,0); bb.AlwaysOnTop=true
        speedLbl=Instance.new("TextLabel",bb); speedLbl.Size=UDim2.new(1,0,1,0); speedLbl.BackgroundTransparency=1; speedLbl.TextColor3=Theme.AccentGlow; speedLbl.Font=Theme.FontBold; speedLbl.TextScaled=true; speedLbl.TextStrokeTransparency=0.5; speedLbl.TextStrokeColor3=Color3.fromRGB(0,20,40)
    end
    if State.antiRagdollEnabled and not Conns.antiRag then task.wait(0.5); startAntiRagdoll() end
    if State.medusaCounterEnabled then setupMedusaCounter(char) end
    if State.animEnabled then task.wait(0.3); saveOriginalAnims(char); applyAnimPack(char) end
    if State.unwalkEnabled then State.unwalkEnabled=false; task.wait(0.3); startUnwalk() end
end
LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

RunService.Stepped:Connect(function()
    for _,p in ipairs(Players:GetPlayers()) do if p~=LP and p.Character then for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end end end
end)

UIS.JumpRequest:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root then root.Velocity=Vector3.new(root.Velocity.X,55,root.Velocity.Z) end
end)
RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled then return end; local char=LP.Character; if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart"); if root and root.Velocity.Y<-120 then root.Velocity=Vector3.new(root.Velocity.X,-120,root.Velocity.Z) end
end)

RunService.RenderStepped:Connect(function()
    if not (h and hrp) then return end; if State._tpInProgress then return end
    if State.autoLeftEnabled or State.autoRightEnabled then if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end; return end
    local md=h.MoveDirection; local spd=getCurrentSpeed()
    if md.Magnitude>0 then State.lastMoveDir=md; hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
    elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude>0 then local anyHeld=false; for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true; break end end; if anyHeld then hrp.Velocity=Vector3.new(State.lastMoveDir.X*spd,hrp.Velocity.Y,State.lastMoveDir.Z*spd) end end
    if speedLbl then local hs=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude; speedLbl.Text="Speed: "..string.format("%.1f",hs) end
end)

RunService.Heartbeat:Connect(function(dt)
    if not (State.autoBatToggled and h and hrp) then resetBend(); return end
    local target,dist=getClosestPlayer(); if not (target and target.Character) then resetBend(); return end
    local tr=target.Character:FindFirstChild("HumanoidRootPart"); if not tr then resetBend(); return end
    local rj=getRootJoint(); if rj and not State.originalC0 then State.originalC0=rj.C0 end
    local yDiff=tr.Position.Y-hrp.Position.Y; local yVel=math.clamp(yDiff*20,-120,120)
    if dist>6 then
        local dir=(tr.Position-hrp.Position).Unit; hrp.AssemblyLinearVelocity=Vector3.new(dir.X*59,yVel,dir.Z*59)
        if rj then rj.C0=State.originalC0*CFrame.Angles(math.rad(40),0,0) end
    else
        State.crazyAngle=State.crazyAngle+dt*40
        local offsetX=math.cos(State.crazyAngle)*0.8; local offsetZ=math.sin(State.crazyAngle)*0.8
        local targetPos=tr.Position+Vector3.new(offsetX,0,offsetZ); local flatDir=Vector3.new(targetPos.X-hrp.Position.X,0,targetPos.Z-hrp.Position.Z)
        hrp.AssemblyLinearVelocity=Vector3.new(flatDir.Unit.X*200,yVel,flatDir.Unit.Z*200)
        if rj then local t=tick(); local fwdBack=math.sin(t*25)*math.rad(50); local sideways=math.cos(t*20)*math.rad(30); rj.C0=State.originalC0*CFrame.Angles(fwdBack,0,sideways) end
        tryHitBat()
    end
end)

UIS.InputBegan:Connect(function(inp,gp)
    if gp then return end
    if inp.UserInputType~=Enum.UserInputType.Keyboard and inp.UserInputType~=Enum.UserInputType.Gamepad1 then return end
    local kc=inp.KeyCode
    if (State.autoLeftEnabled or State.autoRightEnabled) then if MOVE_KEYS[kc] then return end end
    if kc==Keys.speed then toggleSpeedType()
    elseif kc==Keys.lagger then toggleLagger()
    elseif kc==Keys.autoBat then State.autoBatToggled=not State.autoBatToggled; if not State.autoBatToggled then resetBend() end; setAutoBat(State.autoBatToggled); autoSaveConfig()
    elseif kc==Keys.autoLeft then State.autoLeftEnabled=not State.autoLeftEnabled; if setAutoLeft then setAutoLeft(State.autoLeftEnabled) end; if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end; autoSaveConfig()
    elseif kc==Keys.autoRight then State.autoRightEnabled=not State.autoRightEnabled; if setAutoRight then setAutoRight(State.autoRightEnabled) end; if State.autoRightEnabled then startAutoRight() else stopAutoRight() end; autoSaveConfig()
    elseif kc==Keys.dropBrainrot then task.spawn(runDropBrainrot)
    elseif kc==Keys.tpDown then tpToGround()
    elseif kc==Keys.guiHide then toggleGuiVis() end
end)

task.spawn(function() while task.wait(0.5) do pcall(function() if progressRadLbl then progressRadLbl.Text="Radius: "..AutoSteal.Radius end end) end end)

local function loadConfig()
    local hasFile=false; pcall(function() hasFile=isfile("JispiHubConfig.json") end); if not hasFile then return end
    local ok,cfg=pcall(function() return HttpService:JSONDecode(readfile("JispiHubConfig.json")) end); if not ok or not cfg then return end
    if cfg.normalSpeed and type(cfg.normalSpeed)=="number" then State.normalSpeed=cfg.normalSpeed; normalBox.Text=tostring(cfg.normalSpeed) end
    if cfg.carrySpeed  and type(cfg.carrySpeed)=="number"  then State.carrySpeed=cfg.carrySpeed;   carryBox.Text=tostring(cfg.carrySpeed)   end
    if cfg.laggerSpeed and type(cfg.laggerSpeed)=="number" then State.laggerSpeed=cfg.laggerSpeed; laggerBox.Text=tostring(cfg.laggerSpeed) end
    if cfg.laggerCarrySpeed and type(cfg.laggerCarrySpeed)=="number" then State.laggerCarrySpeed=cfg.laggerCarrySpeed; carryLaggerBox.Text=tostring(cfg.laggerCarrySpeed) end
    if cfg.speedType=="normal" or cfg.speedType=="carry" then State.speedType=cfg.speedType end
    if type(cfg.laggerActive)=="boolean" then State.laggerActive=cfg.laggerActive end
    if cfg.autoBatKey  and Enum.KeyCode[cfg.autoBatKey]    then Keys.autoBat=Enum.KeyCode[cfg.autoBatKey];   if autoBatKeyBtn  then autoBatKeyBtn.Text="["..cfg.autoBatKey.."]"   end end
    if cfg.speedKey    and Enum.KeyCode[cfg.speedKey]      then Keys.speed=Enum.KeyCode[cfg.speedKey]        end
    if cfg.laggerKey   and Enum.KeyCode[cfg.laggerKey]     then Keys.lagger=Enum.KeyCode[cfg.laggerKey]      end
    if cfg.autoLeftKey  and Enum.KeyCode[cfg.autoLeftKey]  then Keys.autoLeft=Enum.KeyCode[cfg.autoLeftKey];   if autoLeftKeyBtn  then autoLeftKeyBtn.Text="["..cfg.autoLeftKey.."]"   end end
    if cfg.autoRightKey and Enum.KeyCode[cfg.autoRightKey] then Keys.autoRight=Enum.KeyCode[cfg.autoRightKey]; if autoRightKeyBtn then autoRightKeyBtn.Text="["..cfg.autoRightKey.."]" end end
    if cfg.tpDownKey    and Enum.KeyCode[cfg.tpDownKey]    then Keys.tpDown=Enum.KeyCode[cfg.tpDownKey];       if tpDownKeyBtn    then tpDownKeyBtn.Text="["..cfg.tpDownKey.."]"        end end
    if cfg.grabRadius and type(cfg.grabRadius)=="number" then AutoSteal.Radius=cfg.grabRadius; if progressRadLbl then progressRadLbl.Text="Radius: "..cfg.grabRadius end end
    if cfg.autoStealEnabled then AutoSteal.Enabled=true; setInstaGrab(true); pcall(startAutoSteal) end
    if cfg.infJump      then State.infJumpEnabled=true;      setInfJump(true)      end
    if cfg.antiRagdoll  then State.antiRagdollEnabled=true;  setAntiRag(true);     startAntiRagdoll() end
    if cfg.fpsBoost     then State.fpsBoostEnabled=true;     setFps(true);         applyFPSBoost()    end
    if cfg.medusaCounter then State.medusaCounterEnabled=true; setMedusaCounter(true); setupMedusaCounter(LP.Character) end
    if cfg.dropBrainrotKey and Enum.KeyCode[cfg.dropBrainrotKey] then Keys.dropBrainrot=Enum.KeyCode[cfg.dropBrainrotKey]; if dropBrainrotKeyBtn then dropBrainrotKeyBtn.Text="["..cfg.dropBrainrotKey.."]"                    bat:Activate() --RGS HUB V2
