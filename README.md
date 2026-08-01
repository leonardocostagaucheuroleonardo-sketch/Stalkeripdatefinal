-- 👤 Stalker + Seek Music Changer

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local StarterGui = game:GetService("StarterGui")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local camera = Workspace.CurrentCamera
local humanoid = char:WaitForChild("Humanoid")

-- ================== STALKER ==================
local stalkerImages = {
    "rbxassetid://115047824955445",
    "rbxassetid://73629850429939",
    "rbxassetid://71725176156204",
    "rbxassetid://122094429760163",
    "rbxassetid://100917760105588",
    "rbxassetid://127842346062233",
    "rbxassetid://94733115025990"
}

local currentStalker = nil
local activated = false
local achievementGiven = false

local spawnSound = 136833080474934
local randomScarySounds = {136350971091939, 113917217579668}

local function caption(text)
    local MainUI = player:WaitForChild("PlayerGui"):WaitForChild("MainUI")
    local func = require(MainUI.Initiator.Main_Game)
    func.caption(text, true)
end

local function playSound(id, volume)
    local sound = Instance.new("Sound", Workspace)
    sound.SoundId = "rbxassetid://" .. id
    sound.Volume = volume or 3.5
    sound:Play()
    Debris:AddItem(sound, 6)
end

local function giveAchievement()
    if achievementGiven then return end
    achievementGiven = true

    local achievementGiver = loadstring(game:HttpGet("https://raw.githubusercontent.com/RegularVynixu/Utilities/main/Doors/Custom%20Achievements/Source.lua"))()
    achievementGiver({
        Title = "Find a Stalker",
        Desc = "Let him watch you.",
        Reason = "But don't get close...",
        Image = "rbxassetid://99486859440104"
    })
end

local function spawnStalker()
    if currentStalker then currentStalker:Destroy() end

    local stalkerPart = Instance.new("Part")
    stalkerPart.Name = "Stalker"
    stalkerPart.Size = Vector3.new(9, 12, 2.5)
    stalkerPart.Transparency = 1
    stalkerPart.Anchored = true
    stalkerPart.CanCollide = false
    stalkerPart.Parent = Workspace

    local randomImage = stalkerImages[math.random(1, #stalkerImages)]

    for _, face in pairs(Enum.NormalId:GetEnumItems()) do
        local decal = Instance.new("Decal")
        decal.Texture = randomImage
        decal.Face = face
        decal.Parent = stalkerPart
    end

    stalkerPart.CFrame = hrp.CFrame * CFrame.new(0, 3, 24)

    task.spawn(function()
        while stalkerPart.Parent do
            stalkerPart.CFrame = CFrame.new(stalkerPart.Position, camera.CFrame.Position)
            task.wait(0.03)
        end
    end)

    playSound(spawnSound, 4)

    if math.random(1, 3) == 1 then
        playSound(randomScarySounds[math.random(1, #randomScarySounds)], 3.5)
    end

    currentStalker = stalkerPart

    task.delay(math.random(45, 90), function()
        if currentStalker then currentStalker:Destroy() end
    end)
end

-- JUMPSCARE (Stalker 3)
local function jumpscare()
    local js = Instance.new("Part")
    js.Name = "StalkerJumpscare"
    js.Size = Vector3.new(7, 10, 1.5)
    js.Transparency = 1
    js.Anchored = true
    js.CanCollide = false
    js.Parent = Workspace

    local img = "rbxassetid://73629850429939"

    for _, face in pairs(Enum.NormalId:GetEnumItems()) do
        local decal = Instance.new("Decal")
        decal.Texture = img
        decal.Face = face
        decal.Parent = js
    end

    js.CFrame = hrp.CFrame * CFrame.new(0, 3, -6)  -- Mais baixo

    task.spawn(function()
        while js.Parent do
            js.CFrame = CFrame.new(js.Position, camera.CFrame.Position)
            task.wait(0.03)
        end
    end)

    task.wait(1.4)
    js:Destroy()
end

-- Sombra + Cego + Dano
task.spawn(function()
    while true do
        if currentStalker then
            local distance = (currentStalker.Position - hrp.Position).Magnitude
            if distance < 9 then
                local oldFogEnd = Lighting.FogEnd
                local oldAmbient = Lighting.Ambient
                local oldBrightness = Lighting.Brightness
                
                Lighting.FogColor = Color3.new(0,0,0)
                Lighting.FogEnd = 18
                Lighting.Ambient = Color3.new(0.01,0.01,0.01)
                Lighting.Brightness = 0.05

                caption("...")

                humanoid:TakeDamage(20)

                task.wait(2.3)

                Lighting.FogEnd = oldFogEnd
                Lighting.Ambient = oldAmbient
                Lighting.Brightness = oldBrightness
                
                giveAchievement()

                if currentStalker then
                    currentStalker:Destroy()
                    currentStalker = nil
                end
            end
        end
        task.wait(0.1)
    end
end)

-- ================== SEEK MUSIC ==================
local NEW_SEEK_ID = "rbxassetid://125959136412325"

local changedSounds = {}

local function changeSeekMusic()
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("Sound") and not changedSounds[obj] then
            local nameLower = obj.Name:lower()
            local parentName = obj.Parent and obj.Parent.Name:lower() or ""

            if nameLower:find("seek") or nameLower:find("chase") or parentName:find("seek") or parentName:find("chase") then
                changedSounds[obj] = true
                
                local wasPlaying = obj.IsPlaying
                local oldTime = obj.TimePosition
                
                obj.SoundId = NEW_SEEK_ID
                obj.Volume = 3.8
                obj.PlaybackSpeed = 1
                obj.Looped = true
                
                if wasPlaying then
                    obj:Stop()
                    task.wait(0.05)
                    obj.TimePosition = oldTime
                    obj:Play()
                end
            end
        end
    end
end

task.spawn(function()
    while true do
        changeSeekMusic()
        task.wait(2)
    end
end)

ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    task.wait(1)
    changeSeekMusic()
end)

task.wait(3)
changeSeekMusic()

-- ================== ATIVAÇÃO ==================
ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    if not activated then
        activated = true
        caption("Stalker Mod Activated!!!")
        task.wait(2)
        caption("Credit: Twixxel")
    end
end)

-- Spawn aleatório
task.spawn(function()
    while true do
        task.wait(math.random(35, 70))
        if activated and math.random(1, 4) == 1 then
            spawnStalker()
        end
    end
end)

-- Jumpscare aleatório
task.spawn(function()
    while true do
        task.wait(math.random(40, 85))
        if activated and math.random(1, 5) == 1 then
            jumpscare()
        end
    end
end)

player.Chatted:Connect(function(msg)
    local m = msg:lower()
    if m == "/stalker" then
        spawnStalker()
    elseif m == "/stalker3" then
        jumpscare()
    end
end)

print("✅ Stalker + Seek Music carregado!")
print("/stalker = Stalker normal")
print("/stalker3 = Jumpscare")


-- 👤 Stalker 2 - Chase Mode (Dano 40)

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local camera = Workspace.CurrentCamera
local humanoid = char:WaitForChild("Humanoid")

local CHASE_IMAGE = "rbxassetid://91688967181531"
local CHASE_MUSIC = "rbxassetid://129085629036594"

local chaseEntity = nil
local chaseMusic = nil

local function caption(text)
    local MainUI = player:WaitForChild("PlayerGui"):WaitForChild("MainUI")
    local func = require(MainUI.Initiator.Main_Game)
    func.caption(text, true)
end

local function startChaseMode()
    if chaseEntity then return end

    caption("It is 499 blocks away from you")
    
    chaseMusic = Instance.new("Sound", Workspace)
    chaseMusic.SoundId = CHASE_MUSIC
    chaseMusic.Volume = 3.2
    chaseMusic.Looped = true
    chaseMusic:Play()

    local chaser = Instance.new("Part")
    chaser.Size = Vector3.new(7, 13, 2)
    chaser.Transparency = 1
    chaser.Anchored = true
    chaser.CanCollide = false
    chaser.Parent = Workspace

    local decal = Instance.new("Decal")
    decal.Texture = CHASE_IMAGE
    decal.Face = Enum.NormalId.Front
    decal.Parent = chaser

    chaser.CFrame = hrp.CFrame * CFrame.new(0, 5, 70)

    chaseEntity = chaser

    local distance = 499
    local speed = 8

    task.spawn(function()
        while chaseEntity do
            distance = math.max(5, distance - 24)
            speed = speed + 2.1

            caption("It is " .. math.floor(distance) .. " blocks away from you")

            Lighting.Ambient = Color3.fromRGB(140, 0, 0)
            Lighting.Brightness = 0.35

            local dir = (hrp.Position - chaser.Position).Unit
            chaser.CFrame = CFrame.new(chaser.Position, hrp.Position)

            chaser.Position = chaser.Position + dir * speed * 0.18

            if distance <= 10 then
                caption("RUN")
                speed = speed + 10
            end

            if (chaser.Position - hrp.Position).Magnitude < 9 then
                humanoid:TakeDamage(40)   -- Dano 40 ao invés de matar
                
                task.wait(1.5)
                if chaseEntity then chaseEntity:Destroy() end
                if chaseMusic then chaseMusic:Stop() end
                
                Lighting.Ambient = Color3.new(1,1,1)
                Lighting.Brightness = 1
                break
            end
            task.wait(0.18)
        end
    end)
end

-- Spawn aleatório BEM RARO
task.spawn(function()
    while true do
        task.wait(math.random(80, 160))
        if math.random(1, 6) == 1 then
            startChaseMode()
        end
    end
end)

player.Chatted:Connect(function(msg)
    if msg:lower() == "/stalker2" then
        startChaseMode()
    end
end)

print("👤 Stalker 2 carregado!")
print("Use /stalker2 para forçar")

-- 👤 /bon - Final (Aleatório Bem Raro)

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local camera = Workspace.CurrentCamera

local IMAGE = "rbxassetid://83332839682670"
local SPAWN_SOUND = "rbxassetid://108665567548115"
local DESPAWN_SOUND = "rbxassetid://7428927488"

local current = nil
local achievementGiven = false

local function playSound(id)
    local s = Instance.new("Sound", Workspace)
    s.SoundId = id
    s.Volume = 3.5
    s:Play()
    Debris:AddItem(s, 5)
end

local function giveAchievement()
    if achievementGiven then return end
    achievementGiven = true

    local achievementGiver = loadstring(game:HttpGet(
        "https://raw.githubusercontent.com/RegularVynixu/Utilities/main/Doors/Custom%20Achievements/Source.lua"
    ))()

    achievementGiver({
        Title = "He loves watching you...",
        Desc = "He's coming back...",
        Reason = "Bon is looking at you.",
        Image = "rbxassetid://76440712166258"
    })
end

local function spawn()
    if current then current:Destroy() end

    local part = Instance.new("Part")
    part.Name = "Stalker"
    part.Size = Vector3.new(9, 12, 2.5)
    part.Transparency = 1
    part.Anchored = true
    part.CanCollide = false
    part.Parent = Workspace

    for _, face in pairs(Enum.NormalId:GetEnumItems()) do
        local decal = Instance.new("Decal")
        decal.Texture = IMAGE
        decal.Face = face
        decal.Parent = part
    end

    part.CFrame = hrp.CFrame * CFrame.new(0, 3, 20)
    current = part

    playSound(SPAWN_SOUND)
    print("Spawnou!")

    -- Sempre te olhando
    task.spawn(function()
        while part.Parent do
            part.CFrame = CFrame.new(part.Position, camera.CFrame.Position)
            task.wait(0.03)
        end
    end)

    -- Some quando olhar + som + conquista
    task.spawn(function()
        while part.Parent do
            local dir = (part.Position - camera.CFrame.Position).Unit
            if camera.CFrame.LookVector:Dot(dir) > 0.75 then
                playSound(DESPAWN_SOUND)
                giveAchievement()
                part:Destroy()
                current = nil
                print("Sumiu + conquista!")
                break
            end
            task.wait(0.1)
        end
    end)
end

-- Spawn aleatório BEM RARO
task.spawn(function()
    while true do
        task.wait(math.random(90, 180)) -- bem raro (1min30s a 3min)
        if math.random(1, 5) == 1 then
            spawn()
        end
    end
end)

player.Chatted:Connect(function(msg)
    if msg:lower() == "/bon" then
        spawn()
    end
end)

print("Pronto. Digite /bon")

-- 👤 Stalker5 - Bem Raro + Resto igual

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local humanoid = char:WaitForChild("Humanoid")
local camera = Workspace.CurrentCamera

local images = {
    "rbxassetid://117555949177010",
    "rbxassetid://117693738162384",
    "rbxassetid://127803001168102",
    "rbxassetid://72001283472257",
    "rbxassetid://106205359037024"
}

local SPAWN_SOUND = "rbxassetid://85765588267110"

local current = nil
local music = nil
local achievementGiven = false

local function giveAchievement()
    if achievementGiven then return end
    achievementGiven = true

    local achievementGiver = loadstring(game:HttpGet(
        "https://raw.githubusercontent.com/RegularVynixu/Utilities/main/Doors/Custom%20Achievements/Source.lua"
    ))()

    achievementGiver({
        Title = "YELLING'S WONT STOP...",
        Desc = "You survived yelling creature",
        Reason = "I h&t= y&ou",
        Image = "rbxassetid://99854480974574"
    })
end

local function spawn()
    if current then current:Destroy() end
    if music then music:Stop() end

    local part = Instance.new("Part")
    part.Name = "NovaEntidade"
    part.Size = Vector3.new(9, 12, 0.2)
    part.Transparency = 1
    part.Anchored = true
    part.CanCollide = false
    part.Parent = Workspace

    for _, face in pairs(Enum.NormalId:GetEnumItems()) do
        local d = Instance.new("Decal")
        d.Texture = images[1]
        d.Face = face
        d.Parent = part
    end

    part.CFrame = hrp.CFrame * CFrame.new(0, 3, 20)
    current = part

    music = Instance.new("Sound", Workspace)
    music.SoundId = SPAWN_SOUND
    music.Volume = 3.5
    music.PlaybackSpeed = 0.30
    music.Looped = false
    music:Play()

    print("Nova entidade apareceu!")

    music.Ended:Connect(function()
        if current then
            current:Destroy()
            current = nil
        end
        if music then music:Stop() end
        giveAchievement()
        print("Som acabou → sumiu + conquista!")
    end)

    -- Sempre te olhando
    task.spawn(function()
        while part.Parent do
            part.CFrame = CFrame.new(part.Position, camera.CFrame.Position)
            task.wait(0.03)
        end
    end)

    -- Troca de imagem
    task.spawn(function()
        for i = 2, #images do
            task.wait(0.6)
            if not part.Parent then return end

            for _, d in pairs(part:GetChildren()) do
                if d:IsA("Decal") then
                    d.Texture = images[i]
                end
            end
        end

        task.wait(2)
        if not part.Parent then return end

        print("Indo atrás!")

        while part.Parent do
            local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            if root then
                local dir = (root.Position - part.Position)
                if dir.Magnitude > 0.5 then
                    dir = dir.Unit
                    part.CFrame = CFrame.new(part.Position + dir * 10 * 0.05, root.Position)
                end

                if (part.Position - root.Position).Magnitude < 8 then
                    humanoid:TakeDamage(30)
                    part:Destroy()
                    current = nil
                    if music then music:Stop() end
                    giveAchievement()
                    break
                end
            end
            task.wait(0.03)
        end
    end)
end

-- Spawn aleatório BEM RARO
task.spawn(function()
    while true do
        task.wait(math.random(120, 240)) -- bem raro (2 a 4 minutos)
        if math.random(1, 6) == 1 then
            spawn()
        end
    end
end)

player.Chatted:Connect(function(msg)
    if msg:lower() == "/stalker5" then
        spawn()
    end
end)

print("✅ Stalker5 (bem raro)!")
print("Use /stalker5")

local Lighting = game:GetService("Lighting")

local cc = Lighting:FindFirstChild("DarkMode")
if not cc then
	cc = Instance.new("ColorCorrectionEffect")
	cc.Name = "DarkMode"
	cc.Parent = Lighting
end

while task.wait(0.2) do
	Lighting.ClockTime = 0
	Lighting.Brightness = 1.5
	Lighting.ExposureCompensation = -0.35
	Lighting.Ambient = Color3.fromRGB(55, 55, 55)
	Lighting.OutdoorAmbient = Color3.fromRGB(45, 45, 45)
	Lighting.EnvironmentDiffuseScale = 0.45
	Lighting.EnvironmentSpecularScale = 0.2
	Lighting.ShadowSoftness = 0.7

	cc.Brightness = -0.03
	cc.Contrast = 0.08
	cc.Saturation = -0.02
end

-- 👤 Stalker 4 - Final

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local humanoid = char:WaitForChild("Humanoid")
local camera = Workspace.CurrentCamera

local IMAGE1 = "rbxassetid://101139774680573"
local IMAGE2 = "rbxassetid://94733115025990"
local SPAWN_SOUND = "rbxassetid://73171709370707"

local current = nil
local chaseMusic = nil
local shaking = false
local achievementGiven = false
local shakeConnection = nil
local alreadyDamaged = false
local spawnCount = 0
local MAX_SPAWNS = 2

local function caption(text)
	local MainUI = player:WaitForChild("PlayerGui"):WaitForChild("MainUI")
	local func = require(MainUI.Initiator.Main_Game)
	func.caption(text, true)
end

local function giveAchievement()
	if achievementGiven then return end
	achievementGiven = true

	local achievementGiver = loadstring(game:HttpGet(
		"https://raw.githubusercontent.com/RegularVynixu/Utilities/main/Doors/Custom%20Achievements/Source.lua"
	))()

	achievementGiver({
		Title = "Next time, he'll get you.",
		Desc = "You can't hide.",
		Reason = "survive your nightmare...",
		Image = "rbxassetid://128401329091126"
	})
end

local function startShake()
	if shaking then return end
	shaking = true

	shakeConnection = RunService.RenderStepped:Connect(function()
		if not shaking then return end
		camera.CFrame = camera.CFrame * CFrame.new(
			math.random(-10, 10) / 14,
			math.random(-10, 10) / 14,
			math.random(-4, 4) / 20
		)
	end)
end

local function stopShake()
	shaking = false
	if shakeConnection then
		shakeConnection:Disconnect()
		shakeConnection = nil
	end
end

local oldFogEnd, oldFogStart, oldFogColor, oldAmbient, oldBrightness

local function startDarkFog()
	oldFogEnd = Lighting.FogEnd
	oldFogStart = Lighting.FogStart
	oldFogColor = Lighting.FogColor
	oldAmbient = Lighting.Ambient
	oldBrightness = Lighting.Brightness

	Lighting.FogColor = Color3.new(0, 0, 0)
	Lighting.Ambient = Color3.fromRGB(15, 15, 15)

	TweenService:Create(Lighting, TweenInfo.new(1.2, Enum.EasingStyle.Quad), {
		FogEnd = 35,
		FogStart = 0,
		Brightness = 0.15
	}):Play()
end

local function endDarkFog()
	TweenService:Create(Lighting, TweenInfo.new(1.8, Enum.EasingStyle.Quad), {
		FogEnd = oldFogEnd or 1000,
		FogStart = oldFogStart or 0,
		Brightness = oldBrightness or 1,
		Ambient = oldAmbient or Color3.new(1, 1, 1),
		FogColor = oldFogColor or Color3.new(0.75, 0.75, 0.75)
	}):Play()
end

local function cleanup()
	stopShake()
	endDarkFog()

	if current then
		current:Destroy()
		current = nil
	end
	if chaseMusic then
		chaseMusic:Stop()
		chaseMusic:Destroy()
		chaseMusic = nil
	end
end

local function spawn()
	if current then return end
	if spawnCount >= MAX_SPAWNS then
		print("Stalker4 já apareceu 2 vezes nessa partida.")
		return
	end

	spawnCount = spawnCount + 1
	alreadyDamaged = false

	local part = Instance.new("Part")
	part.Name = "Stalker4"
	part.Size = Vector3.new(9, 12, 0.2)
	part.Transparency = 1
	part.Anchored = true
	part.CanCollide = false
	part.Parent = Workspace

	for _, face in pairs(Enum.NormalId:GetEnumItems()) do
		local d = Instance.new("Decal")
		d.Texture = IMAGE1
		d.Face = face
		d.Parent = part
	end

	part.CFrame = hrp.CFrame * CFrame.new(0, 3, 20)
	current = part

	caption("do not hide just run")

	chaseMusic = Instance.new("Sound", Workspace)
	chaseMusic.SoundId = SPAWN_SOUND
	chaseMusic.Volume = 3.5
	chaseMusic.PlaybackSpeed = 0.20
	chaseMusic.Looped = false
	chaseMusic:Play()

	task.spawn(function()
		while chaseMusic and chaseMusic.Parent do
			if not chaseMusic.IsPlaying then
				chaseMusic.TimePosition = 1.1
				chaseMusic:Play()
			end
			task.wait(0.08)
		end
	end)

	print("Stalker4 apareceu! (" .. spawnCount .. "/2)")

	-- Sempre te olhando
	task.spawn(function()
		while part.Parent do
			local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
			if root then
				part.CFrame = CFrame.new(part.Position, root.Position)
			end
			task.wait(0.03)
		end
	end)

	-- 4s → troca imagem + névoa + treme + persegue
	task.delay(4, function()
		if not part.Parent then return end

		for _, d in pairs(part:GetChildren()) do
			if d:IsA("Decal") then
				d.Texture = IMAGE2
			end
		end

		startDarkFog()
		startShake()

		local startTime = tick()

		while part.Parent and (tick() - startTime) < 20 do
			local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
			local hum = player.Character and player.Character:FindFirstChild("Humanoid")

			if root then
				local dir = (root.Position - part.Position)
				if dir.Magnitude > 0.5 then
					dir = dir.Unit
					local shakeOffset = Vector3.new(
						math.random(-3, 3) / 10,
						math.random(-3, 3) / 10,
						0
					)
					part.CFrame = CFrame.new(part.Position + dir * 10 * 0.05 + shakeOffset, root.Position)
				end

				-- Dano 60 → some na hora
				if not alreadyDamaged and (part.Position - root.Position).Magnitude < 8 then
					alreadyDamaged = true
					if hum then
						hum:TakeDamage(60)
					end
					print("Tomou 60 de dano → sumiu!")
					cleanup()
					giveAchievement()
					return
				end
			end
			task.wait(0.03)
		end

		-- Sobreviveu os 20s
		cleanup()
		giveAchievement()
		print("Stalker4 sumiu + conquista!")
	end)
end

-- Spawn aleatório BEM BEM RARO (mas aparece na partida)
task.spawn(function()
	task.wait(math.random(40, 90)) -- primeira chance depois de 40~90s
	while spawnCount < MAX_SPAWNS do
		if math.random(1, 3) == 1 then -- chance razoável de aparecer
			spawn()
		end
		task.wait(math.random(100, 180)) -- bem espaçado
	end
end)

player.Chatted:Connect(function(msg)
	if msg:lower() == "/stalker4" then
		spawn()
	end
end)

print("✅ Stalker4 Final!")
print("Máximo 2 vezes por partida | Use /stalker4")

-- 👤 Window Entity - Conquista só uma vez

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local humanoid = char:WaitForChild("Humanoid")
local camera = Workspace.CurrentCamera

local WINDOW_IMAGE = "rbxassetid://136542774742776"
local CHASE_IMAGE = "rbxassetid://107999364222287"
local LOOK_SOUND = "rbxassetid://78494358244371"
local CHASE_MUSIC = "rbxassetid://91203761863073"
local ACHIEVEMENT_IMAGE = "rbxassetid://133276883616111"

local currentWindowEntity = nil
local cooldownActive = false
local cooldownEnd = 0
local chaseEntity = nil
local chaseMusic = nil
local achievementGiven = false

local function caption(text)
	pcall(function()
		local MainUI = player:WaitForChild("PlayerGui"):WaitForChild("MainUI")
		local func = require(MainUI.Initiator.Main_Game)
		func.caption(text, true)
	end)
end

local function giveAchievement()
	if achievementGiven then return end
	achievementGiven = true

	local achievementGiver = loadstring(game:HttpGet(
		"https://raw.githubusercontent.com/RegularVynixu/Utilities/main/Doors/Custom%20Achievements/Source.lua"
	))()

	achievementGiver({
		Title = "Don't Go Outside",
		Desc = "You waited long enough.",
		Reason = "The window was watching...",
		Image = ACHIEVEMENT_IMAGE
	})
end

local function playSound(id, volume, speed)
	local s = Instance.new("Sound", Workspace)
	s.SoundId = id
	s.Volume = volume or 3
	s.PlaybackSpeed = speed or 1
	s:Play()
	Debris:AddItem(s, 10)
	return s
end

local function findWindow()
	local rooms = Workspace:FindFirstChild("CurrentRooms")
	if not rooms then return nil end

	local candidates = {}
	for _, v in pairs(rooms:GetDescendants()) do
		local name = v.Name:lower()
		if v:IsA("BasePart") and (name:find("window") or name:find("glass") or name:find("pane")) then
			if (v.Position - hrp.Position).Magnitude < 80 then
				table.insert(candidates, v)
			end
		end
	end

	if #candidates > 0 then
		return candidates[math.random(1, #candidates)]
	end
	return nil
end

local function isLookingAt(targetPos)
	local camPos = camera.CFrame.Position
	local camLook = camera.CFrame.LookVector
	local dir = (targetPos - camPos)
	if dir.Magnitude > 45 then return false end
	dir = dir.Unit
	return camLook:Dot(dir) > 0.92
end

local function spawnChase()
	if chaseEntity then return end

	chaseMusic = Instance.new("Sound", Workspace)
	chaseMusic.SoundId = CHASE_MUSIC
	chaseMusic.Volume = 4
	chaseMusic.Looped = true
	chaseMusic:Play()

	local part = Instance.new("Part")
	part.Name = "WindowChase"
	part.Size = Vector3.new(8, 11, 0.2)
	part.Transparency = 1
	part.Anchored = true
	part.CanCollide = false
	part.Parent = Workspace

	for _, face in pairs(Enum.NormalId:GetEnumItems()) do
		local d = Instance.new("Decal")
		d.Texture = CHASE_IMAGE
		d.Face = face
		d.Parent = part
	end

	part.CFrame = hrp.CFrame * CFrame.new(0, 3, 16)
	chaseEntity = part

	print("Window Entity PERSEGUINDO!")

	task.spawn(function()
		while part.Parent do
			local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
			local hum = player.Character and player.Character:FindFirstChild("Humanoid")
			if root then
				local dir = (root.Position - part.Position)
				if dir.Magnitude > 0.5 then
					dir = dir.Unit
					part.CFrame = CFrame.new(part.Position + dir * 10 * 0.05, root.Position)
				end
				if (part.Position - root.Position).Magnitude < 8 then
					if hum then hum.Health = 0 end
					part:Destroy()
					chaseEntity = nil
					if chaseMusic then
						chaseMusic:Stop()
						chaseMusic:Destroy()
					end
					break
				end
			end
			task.wait(0.03)
		end
	end)
end

local function startCountdown()
	cooldownActive = true
	cooldownEnd = tick() + 10

	caption("dont go outside")

	task.spawn(function()
		for i = 10, 0, -1 do
			if not cooldownActive then return end
			caption(tostring(i))
			task.wait(1)
		end

		if cooldownActive and tick() >= cooldownEnd then
			cooldownActive = false
			caption("you can go")
			giveAchievement()
			print("Esperou → conquista!")
		end
	end)
end

local function spawnOnWindow()
	if currentWindowEntity then return end

	local window = findWindow()
	local spawnCF

	if window then
		spawnCF = window.CFrame * CFrame.new(0, 0, 1.2)
	else
		spawnCF = hrp.CFrame * CFrame.new(0, 3, -14)
	end

	local part = Instance.new("Part")
	part.Name = "WindowEntity"
	part.Size = Vector3.new(6, 8, 0.2)
	part.Transparency = 1
	part.Anchored = true
	part.CanCollide = false
	part.CFrame = spawnCF
	part.Parent = Workspace

	for _, face in pairs(Enum.NormalId:GetEnumItems()) do
		local d = Instance.new("Decal")
		d.Texture = WINDOW_IMAGE
		d.Face = face
		d.Parent = part
	end

	currentWindowEntity = part
	print("Window Entity na janela!")

	local looked = false
	task.spawn(function()
		while part.Parent and not looked do
			part.CFrame = CFrame.new(part.Position, camera.CFrame.Position)

			if isLookingAt(part.Position) then
				looked = true
				playSound(LOOK_SOUND, 3.5, 0.20)

				local upPos = part.Position + Vector3.new(0, 12, 0)
				local tween = TweenService:Create(part, TweenInfo.new(1.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
					Position = upPos
				})
				tween:Play()
				tween.Completed:Wait()

				if part then part:Destroy() end
				currentWindowEntity = nil

				startCountdown()
				break
			end
			task.wait(0.05)
		end
	end)
end

ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
	if cooldownActive and tick() < cooldownEnd then
		cooldownActive = false
		print("Abriu cedo demais!")
		task.wait(0.4)
		spawnChase()
	end
end)

-- Spawn aleatório bem raro (várias vezes)
task.spawn(function()
	while true do
		task.wait(math.random(140, 280))
		if math.random(1, 5) == 1 and not currentWindowEntity and not chaseEntity and not cooldownActive then
			spawnOnWindow()
		end
	end
end)

player.Chatted:Connect(function(msg)
	if msg:lower() == "/window" then
		spawnOnWindow()
	end
end)

print("✅ Window Entity carregado!")
print("Use /window")

-- 👤 Chica Entity

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local camera = Workspace.CurrentCamera

local IMAGE = "rbxassetid://95421528567921"

local current = nil

local oldFogEnd, oldFogStart, oldFogColor, oldAmbient, oldBrightness

local function startDarkFog()
	oldFogEnd = Lighting.FogEnd
	oldFogStart = Lighting.FogStart
	oldFogColor = Lighting.FogColor
	oldAmbient = Lighting.Ambient
	oldBrightness = Lighting.Brightness

	Lighting.FogColor = Color3.new(0, 0, 0)
	Lighting.Ambient = Color3.fromRGB(10, 10, 10)

	TweenService:Create(Lighting, TweenInfo.new(1.2, Enum.EasingStyle.Quad), {
		FogEnd = 18,
		FogStart = 0,
		Brightness = 0.1
	}):Play()
end

local function endDarkFog()
	TweenService:Create(Lighting, TweenInfo.new(2, Enum.EasingStyle.Quad), {
		FogEnd = oldFogEnd or 1000,
		FogStart = oldFogStart or 0,
		Brightness = oldBrightness or 1,
		Ambient = oldAmbient or Color3.new(1, 1, 1),
		FogColor = oldFogColor or Color3.new(0.75, 0.75, 0.75)
	}):Play()
end

local function findSpot()
	local rooms = Workspace:FindFirstChild("CurrentRooms")
	if not rooms then return nil end

	local candidates = {}

	for _, v in pairs(rooms:GetDescendants()) do
		if not v:IsA("BasePart") then continue end

		local name = v.Name:lower()
		local parentName = v.Parent and v.Parent.Name:lower() or ""
		local dist = (v.Position - hrp.Position).Magnitude

		local isGood =
			name:find("door") or parentName:find("door") or
			name:find("wardrobe") or name:find("locker") or name:find("closet") or
			name:find("wall") or name:find("shelf") or name:find("table") or
			name:find("cabinet") or name:find("frame")

		if isGood and dist > 8 and dist < 40 and v.Size.Magnitude > 2 then
			table.insert(candidates, v)
		end
	end

	if #candidates > 0 then
		return candidates[math.random(1, #candidates)]
	end
	return nil
end

local function spawn()
	if current then return end

	local spot = findSpot()
	local spawnCF

	if spot then
		local offset = spot.CFrame.RightVector * (math.random(0, 1) == 0 and 2.2 or -2.2)
		local back = (spot.Position - hrp.Position).Unit * -0.8
		local pos = spot.Position + offset + back + Vector3.new(0, 2.2, 0)
		spawnCF = CFrame.new(pos, hrp.Position)
		print("Chica em: " .. spot.Name)
	else
		spawnCF = hrp.CFrame * CFrame.new(4, 2.5, -6)
		print("Chica fallback")
	end

	local part = Instance.new("Part")
	part.Name = "ChicaEntity"
	part.Size = Vector3.new(5, 7, 0.2)
	part.Transparency = 1
	part.Anchored = true
	part.CanCollide = false
	part.CFrame = spawnCF
	part.Parent = Workspace
	current = part

	for _, face in pairs(Enum.NormalId:GetEnumItems()) do
		local d = Instance.new("Decal")
		d.Texture = IMAGE
		d.Face = face
		d.Parent = part
	end

	print("Chica apareceu!")

	task.spawn(function()
		while part.Parent do
			local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
			if root then
				part.CFrame = CFrame.new(part.Position, root.Position)

				-- Chegou perto → some + névoa
				if (part.Position - root.Position).Magnitude < 9 then
					part:Destroy()
					current = nil
					print("Chica sumiu → névoa!")

					startDarkFog()
					task.wait(2.5)
					endDarkFog()
					break
				end
			end
			task.wait(0.04)
		end
	end)
end

-- Spawn aleatório (várias vezes)
task.spawn(function()
	while true do
		task.wait(math.random(45, 100))
		if math.random(1, 3) == 1 and not current then
			spawn()
		end
	end
end)

player.Chatted:Connect(function(msg)
	if msg:lower() == "/chica" then
		spawn()
	end
end)

print("✅ Chica carregada!")
print("Use /chica")

-- 👤 StalkerDoor Final + Spawn bem bem raro

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local camera = Workspace.CurrentCamera

local DOOR_IMAGE   = "rbxassetid://89678769967446"
local WINDOW_IMG1  = "rbxassetid://83554491093997"
local WINDOW_IMG2  = "rbxassetid://71225962988620"
local MAIN_SOUND   = "rbxassetid://125434017517616"
local ACHIEVEMENT_IMAGE = "rbxassetid://114867313134951"

local current = nil
local active = false
local zooming = false
local zoomConn = nil
local achievementGiven = false

local function giveAchievement()
	if achievementGiven then return end
	achievementGiven = true

	local achievementGiver = loadstring(game:HttpGet(
		"https://raw.githubusercontent.com/RegularVynixu/Utilities/main/Doors/Custom%20Achievements/Source.lua"
	))()

	achievementGiver({
		Title = "Behind The Door",
		Desc = "You survived the door watcher.",
		Reason = "He was always there...",
		Image = ACHIEVEMENT_IMAGE
	})
end

local function findDoorOrSpot()
	local rooms = Workspace:FindFirstChild("CurrentRooms")
	if not rooms then return nil, "fallback" end

	local doors = {}
	local spots = {}

	for _, v in pairs(rooms:GetDescendants()) do
		if not v:IsA("BasePart") then continue end

		local name = v.Name:lower()
		local parentName = v.Parent and v.Parent.Name:lower() or ""
		local full = name .. " " .. parentName
		local dist = (v.Position - hrp.Position).Magnitude

		if full:find("wardrobe") or full:find("locker") or full:find("closet") or full:find("cabinet") then
			continue
		end

		if (name:find("door") or parentName:find("door")) and dist > 5 and dist < 45 and v.Size.Magnitude > 4 then
			table.insert(doors, v)
		end

		if dist > 8 and dist < 35 then
			if name:find("wall") or name:find("shelf") or name:find("table") or name:find("desk")
				or name:find("book") or name:find("paint") or name:find("frame")
				or parentName:find("furniture") or parentName:find("decor") then
				if v.Size.Magnitude > 3 then
					table.insert(spots, v)
				end
			end
		end
	end

	if #doors > 0 and (math.random(1, 10) <= 7 or #spots == 0) then
		return doors[math.random(1, #doors)], "door"
	elseif #spots > 0 then
		return spots[math.random(1, #spots)], "spot"
	elseif #doors > 0 then
		return doors[1], "door"
	end

	return nil, "fallback"
end

local function findWindow()
	local rooms = Workspace:FindFirstChild("CurrentRooms")
	if not rooms then return nil end

	local candidates = {}
	for _, v in pairs(rooms:GetDescendants()) do
		local name = v.Name:lower()
		if v:IsA("BasePart") and (name:find("window") or name:find("glass") or name:find("pane")) then
			if (v.Position - hrp.Position).Magnitude < 80 then
				table.insert(candidates, v)
			end
		end
	end
	if #candidates > 0 then
		return candidates[math.random(1, #candidates)]
	end
	return nil
end

local function setFace(part, texture)
	for _, d in pairs(part:GetChildren()) do
		if d:IsA("Decal") then
			d.Texture = texture
		end
	end
end

local function createPNG(size, texture, cframe)
	local part = Instance.new("Part")
	part.Size = size
	part.Transparency = 1
	part.Anchored = true
	part.CanCollide = false
	part.CFrame = cframe
	part.Parent = Workspace

	for _, face in pairs(Enum.NormalId:GetEnumItems()) do
		local d = Instance.new("Decal")
		d.Texture = texture
		d.Face = face
		d.Parent = part
	end
	return part
end

local oldFogEnd, oldFogStart, oldFogColor, oldAmbient, oldBrightness
local oldCamType

local function startDarkFog()
	oldFogEnd = Lighting.FogEnd
	oldFogStart = Lighting.FogStart
	oldFogColor = Lighting.FogColor
	oldAmbient = Lighting.Ambient
	oldBrightness = Lighting.Brightness

	Lighting.FogColor = Color3.new(0, 0, 0)
	Lighting.Ambient = Color3.fromRGB(10, 10, 10)

	TweenService:Create(Lighting, TweenInfo.new(1.2, Enum.EasingStyle.Quad), {
		FogEnd = 18,
		FogStart = 0,
		Brightness = 0.1
	}):Play()
end

local function endDarkFog()
	TweenService:Create(Lighting, TweenInfo.new(1.8, Enum.EasingStyle.Quad), {
		FogEnd = oldFogEnd or 1000,
		FogStart = oldFogStart or 0,
		Brightness = oldBrightness or 1,
		Ambient = oldAmbient or Color3.new(1, 1, 1),
		FogColor = oldFogColor or Color3.new(0.75, 0.75, 0.75)
	}):Play()
end

local function startZoom(entity)
	zooming = true
	oldCamType = camera.CameraType
	camera.CameraType = Enum.CameraType.Scriptable

	zoomConn = RunService.RenderStepped:Connect(function()
		if not zooming or not entity or not entity.Parent then return end
		local targetPos = entity.Position + (entity.Position - hrp.Position).Unit * -6 + Vector3.new(0, 1, 0)
		camera.CFrame = CFrame.new(targetPos, entity.Position)
	end)
end

local function stopZoom()
	zooming = false
	if zoomConn then
		zoomConn:Disconnect()
		zoomConn = nil
	end
	if oldCamType then
		camera.CameraType = oldCamType
	end
end

local function spawn()
	if active then return end
	active = true

	local target, kind = findDoorOrSpot()
	local spawnCF

	if target and kind == "door" then
		local mid = target.Position + Vector3.new(0, 1.2, 0)
		local side = target.CFrame.RightVector * 1.3
		spawnCF = CFrame.new(mid + side, hrp.Position)
		print("Spawn na PORTA: " .. target.Name)
	elseif target and kind == "spot" then
		local pos = target.Position + Vector3.new(0, 2, 0) + (target.Position - hrp.Position).Unit * -1.5
		spawnCF = CFrame.new(pos, hrp.Position)
		print("Spawn em lugar visível: " .. target.Name)
	else
		spawnCF = hrp.CFrame * CFrame.new(1.5, 3, -10)
		print("Fallback")
	end

	local doorEntity = createPNG(Vector3.new(6, 9, 0.2), DOOR_IMAGE, spawnCF)
	current = doorEntity

	startZoom(doorEntity)

	task.spawn(function()
		while doorEntity.Parent do
			doorEntity.CFrame = CFrame.new(doorEntity.Position, hrp.Position)
			task.wait(0.03)
		end
	end)

	task.wait(2.5)

	local sidePos = doorEntity.Position + doorEntity.CFrame.RightVector * 12
	local sideTween = TweenService:Create(doorEntity, TweenInfo.new(1.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
		Position = sidePos
	})
	sideTween:Play()
	sideTween.Completed:Wait()

	stopZoom()

	if doorEntity then doorEntity:Destroy() end
	current = nil
	task.wait(0.3)

	local window = findWindow()
	local windowCF
	if window then
		windowCF = CFrame.new(window.Position + Vector3.new(0, 0, 1.2), hrp.Position)
	else
		windowCF = hrp.CFrame * CFrame.new(0, 3, -14)
	end

	local windowEntity = createPNG(Vector3.new(6, 8, 0.2), WINDOW_IMG1, windowCF)
	current = windowEntity
	print("StalkerDoor na janela!")

	local music = Instance.new("Sound", Workspace)
	music.SoundId = MAIN_SOUND
	music.Volume = 4
	music.PlaybackSpeed = 0.20
	music.Looped = false
	music:Play()

	local usingFirst = true
	local startTime = tick()
	local duration = 6

	task.spawn(function()
		while windowEntity and windowEntity.Parent do
			local elapsed = tick() - startTime
			if elapsed >= duration then break end

			windowEntity.CFrame = CFrame.new(windowEntity.Position, camera.CFrame.Position)

			usingFirst = not usingFirst
			setFace(windowEntity, usingFirst and WINDOW_IMG1 or WINDOW_IMG2)

			local progress = elapsed / duration
			local speed = math.max(0.04, 0.22 - (progress * 0.18))
			task.wait(speed)
		end
	end)

	task.wait(duration)

	if windowEntity and windowEntity.Parent then
		windowEntity:Destroy()
	end
	current = nil

	startDarkFog()
	print("Sumiu da janela → névoa!")

	local finished = false
	music.Ended:Connect(function()
		finished = true
	end)

	task.spawn(function()
		while music and music.Parent and not finished do
			if not music.IsPlaying and music.TimePosition > 0.3 then
				finished = true
				break
			end
			task.wait(0.15)
		end
		finished = true
	end)

	repeat task.wait(0.1) until finished

	endDarkFog()

	if music then
		music:Stop()
		music:Destroy()
	end

	giveAchievement()
	print("Som acabou → névoa sumiu + conquista!")

	active = false
end

-- Spawn ALEATÓRIO BEM BEM BEM RARO
task.spawn(function()
	while true do
		task.wait(math.random(180, 360)) -- 3 a 6 minutos
		if math.random(1, 8) == 1 and not active then
			spawn()
		end
	end
end)

player.Chatted:Connect(function(msg)
	if msg:lower() == "/stalkerdoor" then
		spawn()
	end
end)

print("✅ StalkerDoor Final + bem bem raro!")
print("Use /stalkerdoor")
