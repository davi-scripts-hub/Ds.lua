-- DS HUB | Davi Scripts — v1.2 Final
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local SoundService = game:GetService("SoundService")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Debris = game:GetService("Debris")
local VU = game:GetService("VirtualUser")

local player = Players.LocalPlayer
local pg = player:WaitForChild("PlayerGui")

local old = pg:FindFirstChild("DSHub")
if old then old:Destroy() end

local B = Color3.fromRGB(24, 117, 255)
local BG = Color3.fromRGB(5, 12, 28)
local PANEL = Color3.fromRGB(9, 23, 48)
local PANEL2 = Color3.fromRGB(13, 34, 68)
local TXT = Color3.fromRGB(237, 245, 255)
local MUTED = Color3.fromRGB(151, 179, 220)
local GRID = Color3.fromRGB(28, 83, 157)
local OK = Color3.fromRGB(50, 200, 120)
local ERR = Color3.fromRGB(220, 60, 70)
local WARN = Color3.fromRGB(240, 180, 40)

local original = {
    FogStart = Lighting.FogStart, FogEnd = Lighting.FogEnd,
    FogColor = Lighting.FogColor, Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient, Brightness = Lighting.Brightness,
}
local originalSkies = {}
for _, item in ipairs(Lighting:GetChildren()) do
    if item:IsA("Sky") then table.insert(originalSkies, item:Clone()) end
end

local DS = { Sons = true, Animacoes = true, Logs = {}, Toggles = {}, BrilhoAtivo = true, IY = { carregado = false } }

local function make(className, props, parent)
    local object = Instance.new(className)
    for key, value in pairs(props or {}) do object[key] = value end
    object.Parent = parent
    return object
end

local function round(object, radius)
    make("UICorner", { CornerRadius = UDim.new(0, radius or 12) }, object)
end

local function outline(object, color, transparency)
    make("UIStroke", { Color = color or GRID, Transparency = transparency or 0.25, Thickness = 1 }, object)
end

local gui = make("ScreenGui", { Name = "DSHub", ResetOnSpawn = false, ZIndexBehavior = Enum.ZIndexBehavior.Sibling }, pg)

local function tocar()
    if not DS.Sons then return end
    local s = Instance.new("Sound")
    s.SoundId = "rbxassetid://6042053626"
    s.Volume = 0.25
    s.PlaybackSpeed = 1.4
    s.Parent = SoundService
    s:Play()
    Debris:AddItem(s, 1)
end

local function logar(tipo, msg)
    table.insert(DS.Logs, { hora = os.date("%H:%M:%S"), tipo = tipo, msg = msg })
    if #DS.Logs > 100 then table.remove(DS.Logs, 1) end
end

local main

local function notify(msg, tipo)
    logar(tipo or "info", msg)
    if not main or not main.Parent then return end
    local cor = B
    if tipo == "sucesso" then cor = OK
    elseif tipo == "erro" then cor = ERR
    elseif tipo == "aviso" then cor = WARN end
    local toast = make("TextLabel", {
        AnchorPoint = Vector2.new(0.5, 1),
        Position = UDim2.new(0.5, 0, 1, -12),
        Size = UDim2.new(1, -24, 0, 36),
        BackgroundColor3 = PANEL2,
        Text = msg,
        TextWrapped = true,
        TextColor3 = TXT,
        Font = Enum.Font.GothamMedium,
        TextSize = 12,
        BorderSizePixel = 0,
        ZIndex = 100,
    }, main)
    round(toast, 14)
    outline(toast, cor, 0.1)
    task.delay(2.4, function()
        if toast and toast.Parent then
            TweenService:Create(toast, TweenInfo.new(0.18), { BackgroundTransparency = 1, TextTransparency = 1 }):Play()
            task.delay(0.2, function()
                if toast and toast.Parent then toast:Destroy() end
            end)
        end
    end)
end

local bubble = make("ImageButton", {
    Name = "DSHubFloatingButton",
    AnchorPoint = Vector2.new(1, 0.5),
    Position = UDim2.new(1, -18, 0.55, 0),
    Size = UDim2.new(0, 58, 0, 58),
    BackgroundColor3 = B,
    Image = "rbxassetid://125168523230863",
    ScaleType = Enum.ScaleType.Fit,
    BorderSizePixel = 0,
    AutoButtonColor = false,
    ZIndex = 50,
}, gui)
round(bubble, 100)
outline(bubble, Color3.fromRGB(135, 197, 255), 0.08)
make("UIGradient", {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(44, 148, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(8, 57, 165)),
    }),
    Rotation = 45,
}, bubble)

main = make("Frame", {
    Name = "Main",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromOffset(1, 1),
    BackgroundColor3 = BG,
    BackgroundTransparency = 0.04,
    BorderSizePixel = 0,
    ClipsDescendants = true,
    Visible = false,
    ZIndex = 10,
}, gui)
round(main, 20)
local bordaMain = outline(main, B, 0.12)
make("UISizeConstraint", { MinSize = Vector2.new(620, 400), MaxSize = Vector2.new(1400, 900) }, main)

task.spawn(function()
    while gui.Parent do
        if DS.BrilhoAtivo and main.Visible then
            TweenService:Create(bordaMain, TweenInfo.new(1.2, Enum.EasingStyle.Sine), { Transparency = 0.5 }):Play()
            task.wait(1.2)
            TweenService:Create(bordaMain, TweenInfo.new(1.2, Enum.EasingStyle.Sine), { Transparency = 0.05 }):Play()
            task.wait(1.2)
        else
            task.wait(0.4)
        end
    end
end)local gridLayer = make("Frame", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, ZIndex = 10 }, main)
for i = 0, 14 do
    make("Frame", { Position = UDim2.new(i/14, 0, 0, 0), Size = UDim2.new(0, 1, 1, 0), BackgroundColor3 = GRID, BackgroundTransparency = 0.84, ZIndex = 10 }, gridLayer)
end
for i = 0, 9 do
    make("Frame", { Position = UDim2.new(0, 0, i/9, 0), Size = UDim2.new(1, 0, 0, 1), BackgroundColor3 = GRID, BackgroundTransparency = 0.84, ZIndex = 10 }, gridLayer)
end

local header = make("Frame", { Size = UDim2.new(1, 0, 0, 62), BackgroundColor3 = PANEL, BackgroundTransparency = 0.05, ZIndex = 12 }, main)
round(header, 20)
make("Frame", { Position = UDim2.new(0, 0, 1, -15), Size = UDim2.new(1, 0, 0, 15), BackgroundColor3 = PANEL, ZIndex = 12 }, header)
make("ImageLabel", { Position = UDim2.new(0, 14, 0, 9), Size = UDim2.fromOffset(42, 42), BackgroundTransparency = 1, Image = "rbxassetid://125168523230863", ScaleType = Enum.ScaleType.Fit, ZIndex = 14 }, header)
make("TextLabel", { Position = UDim2.new(0, 64, 0, 6), Size = UDim2.new(0.55, 0, 0, 30), BackgroundTransparency = 1, Text = "DS HUB", Font = Enum.Font.GothamBlack, TextSize = 24, TextColor3 = TXT, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 13 }, header)
make("TextLabel", { Position = UDim2.new(0, 66, 0, 35), Size = UDim2.new(0.65, 0, 0, 15), BackgroundTransparency = 1, Text = "DAVI SCRIPTS  •  DARK BLUE  •  v1.2", Font = Enum.Font.GothamMedium, TextSize = 9, TextColor3 = Color3.fromRGB(89, 181, 255), TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 13 }, header)

local close = make("TextButton", { AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -12, 0.5, 0), Size = UDim2.fromOffset(32, 32), Text = "×", Font = Enum.Font.GothamBold, TextSize = 23, TextColor3 = TXT, BackgroundColor3 = PANEL2, BorderSizePixel = 0, ZIndex = 14 }, header)
round(close, 100)

local side = make("Frame", { Position = UDim2.new(0, 10, 0, 72), Size = UDim2.new(0, 126, 1, -84), BackgroundColor3 = PANEL, BackgroundTransparency = 0.05, BorderSizePixel = 0, ZIndex = 12 }, main)
round(side, 15)
outline(side, GRID, 0.5)

local body = make("ScrollingFrame", { Position = UDim2.new(0, 146, 0, 72), Size = UDim2.new(1, -156, 1, -84), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 3, ScrollBarImageColor3 = B, AutomaticCanvasSize = Enum.AutomaticSize.Y, CanvasSize = UDim2.new(), ZIndex = 12 }, main)
make("UIPadding", { PaddingTop = UDim.new(0, 2), PaddingRight = UDim.new(0, 5), PaddingBottom = UDim.new(0, 8) }, body)

local pages, navButtons = {}, {}

local function page(name)
    local p = make("Frame", { Name = "Page" .. name, Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, Visible = false, ZIndex = 12 }, body)
    make("UIListLayout", { Padding = UDim.new(0, 9), SortOrder = Enum.SortOrder.LayoutOrder }, p)
    pages[name] = p
    return p
end

local function show(name)
    for pageName, pageObject in pairs(pages) do
        pageObject.Visible = pageName == name
    end
    for buttonName, buttonObject in pairs(navButtons) do
        if DS.Animacoes then
            TweenService:Create(buttonObject, TweenInfo.new(0.15), { BackgroundColor3 = buttonName == name and B or PANEL2 }):Play()
        else
            buttonObject.BackgroundColor3 = buttonName == name and B or PANEL2
        end
    end
end

local function card(parent, title, subtitle)
    local frame = make("Frame", { Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundColor3 = PANEL, BackgroundTransparency = 0.04, BorderSizePixel = 0, ZIndex = 13 }, parent)
    round(frame, 15)
    outline(frame, GRID, 0.5)
    make("UIPadding", { PaddingTop = UDim.new(0, 12), PaddingBottom = UDim.new(0, 12), PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12) }, frame)
    make("UIListLayout", { Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder }, frame)
    make("TextLabel", { Size = UDim2.new(1, 0, 0, 23), BackgroundTransparency = 1, Text = title, Font = Enum.Font.GothamBold, TextSize = 14, TextColor3 = TXT, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 14 }, frame)
    if subtitle then
        make("TextLabel", { Size = UDim2.new(1, 0, 0, 18), BackgroundTransparency = 1, Text = subtitle, Font = Enum.Font.Gotham, TextSize = 10, TextColor3 = MUTED, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 14 }, frame)
    end
    return frame
end

local function label(parent, text)
    return make("TextLabel", { Size = UDim2.new(1, 0, 0, 20), BackgroundTransparency = 1, Text = text, Font = Enum.Font.Gotham, TextSize = 11, TextColor3 = MUTED, TextXAlignment = Enum.TextXAlignment.Left, TextWrapped = true, ZIndex = 14 }, parent)
end

local function button(parent, text, callback)
    local b = make("TextButton", { Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = PANEL2, Text = text, TextColor3 = TXT, Font = Enum.Font.GothamMedium, TextSize = 11, BorderSizePixel = 0, AutoButtonColor = false, ZIndex = 14 }, parent)
    round(b, 12)
    outline(b, GRID, 0.55)
    b.MouseEnter:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.12), { BackgroundColor3 = Color3.fromRGB(18, 52, 102) }):Play()
    end)
    b.MouseLeave:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.12), { BackgroundColor3 = PANEL2 }):Play()
    end)
    b.MouseButton1Click:Connect(function()
        tocar()
        if callback then
            local ok, err = pcall(callback)
            if not ok then
                warn("[DS HUB]", err)
                notify("Erro: " .. tostring(err), "erro")
            end
        end
    end)
    return b
end

local function textbox(parent, placeholder, value)
    local t = make("TextBox", { Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = Color3.fromRGB(5, 16, 36), Text = value or "", PlaceholderText = placeholder, PlaceholderColor3 = MUTED, TextColor3 = TXT, Font = Enum.Font.Gotham, TextSize = 12, ClearTextOnFocus = false, BorderSizePixel = 0, ZIndex = 14 }, parent)
    round(t, 12)
    outline(t, GRID, 0.45)
    return t
end

local function toggle(parent, text, initial, callback)
    local row = make("Frame", { Size = UDim2.new(1, 0, 0, 32), BackgroundTransparency = 1, ZIndex = 14 }, parent)
    make("TextLabel", { Size = UDim2.new(1, -58, 1, 0), BackgroundTransparency = 1, Text = text, TextColor3 = TXT, Font = Enum.Font.Gotham, TextSize = 11, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 14 }, row)
    local salvo = DS.Toggles[text]
    local on = salvo ~= nil and salvo or (initial == true)
    local sw = make("TextButton", { AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, 0, 0.5, 0), Size = UDim2.new(0, 43, 0, 23), Text = "", BackgroundColor3 = on and B or Color3.fromRGB(43, 55, 77), BorderSizePixel = 0, ZIndex = 15 }, row)
    round(sw, 100)
    local knob = make("Frame", { AnchorPoint = Vector2.new(0, 0.5), Position = on and UDim2.new(1, -20, 0.5, 0) or UDim2.new(0, 3, 0.5, 0), Size = UDim2.new(0, 16, 0, 16), BackgroundColor3 = Color3.new(1, 1, 1), BorderSizePixel = 0, ZIndex = 16 }, sw)
    round(knob, 100)
    sw.MouseButton1Click:Connect(function()
        tocar()
        on = not on
        DS.Toggles[text] = on
        TweenService:Create(sw, TweenInfo.new(0.14), { BackgroundColor3 = on and B or Color3.fromRGB(43, 55, 77) }):Play()
        TweenService:Create(knob, TweenInfo.new(0.14), { Position = on and UDim2.new(1, -19, 0.5, 0) or UDim2.new(0, 3, 0.5, 0) }):Play()
        if callback then pcall(callback, on) end
    end)
    if salvo == true and callback then pcall(callback, true) end
end

local function restoreLighting()
    for key, value in pairs(original) do Lighting[key] = value end
    for _, item in ipairs(Lighting:GetChildren()) do
        if item:IsA("Sky") then item:Destroy() end
    end
    for _, sky in ipairs(originalSkies) do sky:Clone().Parent = Lighting end
    notify("Iluminação restaurada.", "sucesso")
end

local searchTargets = {}
local function registerSearchable(frame) table.insert(searchTargets, frame) end
local function filterAll(query)
    query = (query or ""):lower()
    for _, frame in ipairs(searchTargets) do
        if not frame.Parent then continue end
        if query == "" then frame.Visible = true; continue end
        local found = false
        for _, d in ipairs(frame:GetDescendants()) do
            if (d:IsA("TextLabel") or d:IsA("TextButton") or d:IsA("TextBox")) and d.Text and d.Text:lower():find(query, 1, true) then
                found = true
                break
            end
        end
        frame.Visible = found
    end
endlocal menu = { "Principal", "Skybox", "Neblina", "Personagens", "Players", "Teleport", "Visual", "Admin", "Credits", "Configurações" }
local icons = { "⌂", "☁", "≋", "♟", "➤", "◎", "◉", "🛠", "★", "⚙" }
for index, name in ipairs(menu) do
    local nav = make("TextButton", {
        Position = UDim2.new(0, 7, 0, 8 + (index - 1) * 38),
        Size = UDim2.new(1, -14, 0, 32),
        BackgroundColor3 = name == "Principal" and B or PANEL2,
        Text = icons[index] .. "  " .. name,
        TextColor3 = TXT,
        Font = Enum.Font.GothamMedium,
        TextSize = 10,
        TextXAlignment = Enum.TextXAlignment.Left,
        BorderSizePixel = 0,
        AutoButtonColor = false,
        ZIndex = 14,
    }, side)
    round(nav, 11)
    navButtons[name] = nav
    nav.MouseButton1Click:Connect(function() tocar() show(name) end)
end

local p = page("Principal")
local c = card(p, "Bem-vindo ao DS HUB", "Davi Scripts • Interface azul neon")
label(c, "Painel horizontal inspirado na sua referência.")
label(c, "Atalho: pressione RightShift para abrir/fechar.")
button(c, "Restaurar iluminação original", function() restoreLighting() end)
button(c, "Abrir Skybox", function() show("Skybox") end)
button(c, "Abrir Players", function() show("Players") end)
button(c, "Abrir Admin", function() show("Admin") end)

p = page("Skybox")
c = card(p, "Skybox", "Efeito visual local no seu ambiente de teste")
label(c, "ID numérico de uma imagem compatível com Skybox")
local skyBox = textbox(c, "Digite o ID da imagem")
button(c, "Aplicar Skybox", function()
    local id = skyBox.Text:match("%d+")
    if not id then notify("Digite um ID numérico válido.", "aviso") return end
    for _, item in ipairs(Lighting:GetChildren()) do
        if item:IsA("Sky") then item:Destroy() end
    end
    local sky = Instance.new("Sky")
    local asset = "rbxassetid://" .. id
    sky.SkyboxBk, sky.SkyboxDn, sky.SkyboxFt = asset, asset, asset
    sky.SkyboxLf, sky.SkyboxRt, sky.SkyboxUp = asset, asset, asset
    sky.Parent = Lighting
    notify("Skybox aplicada localmente.", "sucesso")
end)
button(c, "Restaurar Skybox e iluminação", function() restoreLighting() end)

p = page("Neblina")
c = card(p, "Neblina", "Efeitos locais")
toggle(c, "Neblina escura", false, function(on)
    if on then
        Lighting.FogColor = Color3.fromRGB(6, 10, 22)
        Lighting.FogStart, Lighting.FogEnd = 0, 95
        Lighting.Ambient = Color3.fromRGB(22, 26, 43)
        Lighting.OutdoorAmbient = Color3.fromRGB(18, 22, 39)
        notify("Neblina escura ativada.", "sucesso")
    else
        restoreLighting()
    end
end)
button(c, "Neblina azul", function()
    Lighting.FogColor = Color3.fromRGB(13, 44, 100)
    Lighting.FogStart, Lighting.FogEnd = 0, 125
    notify("Neblina azul aplicada.", "sucesso")
end)
button(c, "Restaurar iluminação", function() restoreLighting() end)

p = page("Personagens")
c = card(p, "Personagens", "Presets demonstrativos")
label(c, "Esses botões não trocam o avatar automaticamente.")
for _, characterName in ipairs({ "R6", "R15", "Gojo", "Luffy", "Zoro" }) do
    button(c, "Selecionar " .. characterName, function()
        notify("Preset selecionado: " .. characterName, "info")
    end)
end

p = page("Players")
c = card(p, "Players", "Movimento e personagem no seu próprio jogo")
local infiniteJumpEnabled = false
local jumpConnection
toggle(c, "Infinite Jump", false, function(on)
    infiniteJumpEnabled = on
    if on then
        if jumpConnection then jumpConnection:Disconnect() end
        jumpConnection = UIS.JumpRequest:Connect(function()
            if not infiniteJumpEnabled then return end
            local character = player.Character
            local humanoid = character and character:FindFirstChildOfClass("Humanoid")
            if humanoid then humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
        end)
        notify("Infinite Jump ativado.", "sucesso")
    else
        if jumpConnection then jumpConnection:Disconnect(); jumpConnection = nil end
        notify("Infinite Jump desativado.", "info")
    end
end)

local savedTransparency = {}
toggle(c, "Invisibility", false, function(on)
    local character = player.Character
    if not character then notify("Personagem não encontrado.", "erro") return end
    if on then
        savedTransparency = {}
        for _, obj in ipairs(character:GetDescendants()) do
            if obj:IsA("BasePart") or obj:IsA("Decal") then
                savedTransparency[obj] = obj.Transparency
                obj.Transparency = 1
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") then
                obj.Enabled = false
            end
        end
        notify("Invisibilidade visual local ativada.", "sucesso")
    else
        for obj, value in pairs(savedTransparency) do
            if obj and obj.Parent then obj.Transparency = value end
        end
        savedTransparency = {}
        for _, obj in ipairs(character:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") then obj.Enabled = true end
        end
        notify("Visibilidade restaurada.", "info")
    end
end)

label(c, "Velocidade de caminhada")
local speed = textbox(c, "Velocidade (padrão 16)", "16")
button(c, "Aplicar velocidade", function()
    local number = tonumber(speed.Text)
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if not number or not humanoid then notify("Valor inválido ou personagem indisponível.", "erro") return end
    number = math.clamp(number, 0, 100)
    humanoid.WalkSpeed = number
    notify("Velocidade aplicada: " .. number, "sucesso")
end)
button(c, "Restaurar velocidade padrão", function()
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then humanoid.WalkSpeed = 16 end
    speed.Text = "16"
    notify("Velocidade restaurada.", "info")
end)

p = page("Teleport")
c = card(p, "Teleport", "Coordenadas do seu próprio ambiente de teste")
local x = textbox(c, "Coordenada X", "0")
local y = textbox(c, "Coordenada Y", "10")
local z = textbox(c, "Coordenada Z", "0")
button(c, "Teleportar para coordenadas", function()
    local a, b, d = tonumber(x.Text), tonumber(y.Text), tonumber(z.Text)
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not (a and b and d and root) then notify("Confira as coordenadas.", "erro") return end
    root.CFrame = CFrame.new(a, b, d)
    notify("Teleport realizado.", "sucesso")
end)

p = page("Visual")
c = card(p, "Visual", "Iluminação local")
toggle(c, "Iluminação azul", false, function(on)
    if on then
        Lighting.Ambient = Color3.fromRGB(34, 63, 116)
        Lighting.OutdoorAmbient = Color3.fromRGB(20, 39, 82)
        notify("Iluminação azul ativada.", "sucesso")
    else
        Lighting.Ambient = original.Ambient
        Lighting.OutdoorAmbient = original.OutdoorAmbient
        notify("Iluminação restaurada.", "info")
    end
end)
toggle(c, "Brilho alto", false, function(on)
    Lighting.Brightness = on and 3 or original.Brightness
    notify(on and "Brilho aumentado." or "Brilho restaurado.", "info")
end)p = page("Admin")

local cSearch = card(p, "🔍 Pesquisa Global", "Filtra botões em TODAS as abas em tempo real")
local searchBox = textbox(cSearch, "Digite para pesquisar (ex: fly, speed, skybox)...")
searchBox:GetPropertyChangedSignal("Text"):Connect(function()
    filterAll(searchBox.Text)
end)
button(cSearch, "Limpar pesquisa", function()
    searchBox.Text = ""
    filterAll("")
end)

local cIY = card(p, "Infinite Yield", "Carrega o IY completo + botões de atalho")
label(cIY, "Clique em Carregar primeiro, depois use os atalhos abaixo.")
button(cIY, "🚀 Carregar Infinite Yield", function()
    if DS.IY.carregado then notify("Infinite Yield já foi carregado.", "aviso") return end
    notify("Carregando Infinite Yield...", "info")
    local ok, err = pcall(function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
    end)
    if ok then
        DS.IY.carregado = true
        notify("Infinite Yield carregado com sucesso!", "sucesso")
    else
        warn("[DS HUB] Erro IY:", err)
        notify("Falha ao carregar IY: " .. tostring(err), "erro")
    end
end)

local function IYcmd(cmd)
    if not DS.IY.carregado then notify("Carregue o Infinite Yield primeiro.", "aviso") return end
    local exec = rawget(_G, "IY_ExecuteCommand") or (getgenv and getgenv().IY_ExecuteCommand)
    if type(exec) == "function" then
        local ok, err = pcall(exec, cmd)
        if ok then notify("IY: " .. cmd, "sucesso") return end
    end
    pcall(function()
        local chat = player.PlayerGui:FindFirstChild("Chat")
        if chat then
            local frame = chat:FindFirstChild("Frame", true)
            if frame then
                local tb = frame:FindFirstChildOfClass("TextBox")
                if tb then
                    game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents"):FindFirstChild("SayMessageRequest"):FireServer(cmd, "All")
                end
            end
        end
    end)
    notify("Comando enviado: " .. cmd, "info")
end

local iyCmds = {
    { "✈ Fly", "fly" }, { "🚫 Noclip", "noclip" }, { "🛑 Unfly", "unfly" }, { "✅ Unnoclip", "unnoclip" },
    { "🛡 God", "god" }, { "💀 Ungod", "ungod" }, { "❤ Heal", "heal" }, { "🔄 Refresh", "refresh" },
    { "🔁 Rejoin", "rejoin" }, { "🌐 Serverhop", "serverhop" }, { "👁 ESP", "esp" }, { "🙈 NoESP", "noesp" },
    { "💡 Fullbright", "fullbright" }, { "⏰ Anti-AFK", "antiafk" }, { "☠ Kill All", "killall" },
    { "👻 Invisible", "invisible" }, { "👤 Visible", "visible" }, { "⚡ Speed 16", "speed 16" },
    { "💨 Speed 50", "speed 50" }, { "🏃 Speed 100", "speed 100" },
}
for _, par in ipairs(iyCmds) do
    button(cIY, par[1], function() IYcmd(par[2]) end)
end
label(cIY, "⚠ Use apenas em ambiente próprio/autorizado.")

local cTroll = card(p, "Trolls (C00LCLAN)", "Funções originais preservadas")

button(cTroll, "Sound GUI", function()
    local guiExist = pg:FindFirstChild("SoundGui")
    if guiExist then guiExist:Destroy() return end
    local sg = make("ScreenGui", { Name = "SoundGui", ResetOnSpawn = false }, pg)
    local fr = make("Frame", { Size = UDim2.new(0, 398, 0, 206), Position = UDim2.new(0, 700, 0, 300), BackgroundColor3 = Color3.fromRGB(0, 0, 0), BorderSizePixel = 2, BorderColor3 = Color3.fromRGB(255, 0, 0) }, sg)
    make("TextLabel", { Size = UDim2.new(0, 231, 0, 48), Position = UDim2.new(0.2, 0, 0.03, 0), BackgroundTransparency = 1, Text = "c00lSound GUI!", TextColor3 = Color3.new(1, 1, 1), Font = Enum.Font.Arial, TextSize = 23 }, fr)
    local ids = make("TextBox", { Size = UDim2.new(0, 307, 0, 33), Position = UDim2.new(0.11, 0, 0.3, 0), BackgroundColor3 = Color3.new(0, 0, 0), TextColor3 = Color3.new(1, 1, 1), PlaceholderText = "Sound ID (sem rbxassetid://)", Font = Enum.Font.Arial, TextSize = 12, BorderColor3 = Color3.fromRGB(255, 0, 0), BorderSizePixel = 2 }, fr)
    local add = make("TextButton", { Size = UDim2.new(0, 307, 0, 36), Position = UDim2.new(0.11, 0, 0.71, 0), BackgroundColor3 = Color3.new(0, 0, 0), Text = "Add Sound!", TextColor3 = Color3.new(1, 1, 1), Font = Enum.Font.Arial, TextSize = 22, BorderColor3 = Color3.fromRGB(255, 0, 0), BorderSizePixel = 2 }, fr)
    local ex = make("TextButton", { Size = UDim2.new(0, 25, 0, 23), Position = UDim2.new(0.92, 0, 0.03, 0), BackgroundColor3 = Color3.new(0, 0, 0), Text = "X", TextColor3 = Color3.new(1, 1, 1), Font = Enum.Font.Arial, TextSize = 14, BorderColor3 = Color3.fromRGB(255, 0, 0), BorderSizePixel = 2 }, fr)
    ex.MouseButton1Click:Connect(function() sg:Destroy() end)
    add.MouseButton1Click:Connect(function()
        local sid = ids.Text:match("%d+")
        if not sid then add.Text = "No ID!" task.wait(2) add.Text = "Add Sound!" return end
        local snd = Instance.new("Sound")
        snd.Name = "c00lsound23sas"
        snd.SoundId = "rbxassetid://" .. sid
        snd.Parent = workspace
        snd:Play()
        Debris:AddItem(snd, 10)
        add.Text = "OK!"
        task.wait(1.5)
        add.Text = "Add Sound!"
    end)
    notify("Sound GUI aberto.", "sucesso")
end)

button(cTroll, "DANCE", function()
    local anim = Instance.new("Animation")
    anim.AnimationId = "rbxassetid://27789359"
    local h = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if h then
        local t = h:LoadAnimation(anim)
        t:Play()
    end
    notify("Dançando!", "sucesso")
end)

button(cTroll, "HatHub", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/inkdupe/hat-scripts/refs/heads/main/updatedhathub.lua"))()
    notify("HatHub carregado.", "sucesso")
end)

button(cTroll, "Hitbox Extender", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/AAPVdev/scripts/refs/heads/main/UI_LimbExtender.lua"))()
    notify("Hitbox carregado.", "sucesso")
end)

button(cTroll, "Part Grab", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/v0c0n1337/scripts/refs/heads/main/Unachored_parts_controller_v2.lua.txt"))()
    notify("Part Grab carregado.", "sucesso")
end)

button(cTroll, "Thomas Train", function()
    loadstring(game:HttpGet("https://pastebin.com/raw/dcywCSCs"))()
    notify("Thomas carregado.", "sucesso")
end)

button(cTroll, "I... Am Steve", function()
    loadstring(game:HttpGet("https://pastebin.com/raw/miABMATi"))()
    notify("Steve carregado.", "sucesso")
end)

button(cTroll, "Fake VR", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/randomstring0/Qwerty/refs/heads/main/qwerty46.lua"))()
    notify("Fake VR carregado.", "sucesso")
end)

button(cTroll, "Telekinesis", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/randomstring0/Qwerty/refs/heads/main/qwerty1.lua"))()
    notify("Telekinesis carregado.", "sucesso")
end)

button(cTroll, "Control NPC", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/randomstring0/fe-source/refs/heads/main/NPC/source/main.Luau"))()
    notify("Control NPC carregado.", "sucesso")
end)

button(cTroll, "Invis GUI V2", function()
    loadstring(game:HttpGet("https://pastebin.com/raw/hracJzLa"))()
    notify("Invis GUI V2 carregado.", "sucesso")
end)

button(cTroll, "JOHN DOE", function()
    local redSkyboxAssetId = "rbxassetid://1012887"
    local sky = Lighting:FindFirstChildOfClass("Sky")
    if not sky then sky = Instance.new("Sky", Lighting) end
    sky.SkyboxBk = redSkyboxAssetId
    sky.SkyboxDn = redSkyboxAssetId
    sky.SkyboxFt = redSkyboxAssetId
    sky.SkyboxLf = redSkyboxAssetId
    sky.SkyboxRt = redSkyboxAssetId
    sky.SkyboxUp = redSkyboxAssetId
    local function setupCharacter(character)
        local humanoid = character:WaitForChild("Humanoid")
        local hrp = character:WaitForChild("HumanoidRootPart")
        local naeeym = Instance.new("BillboardGui", character)
        naeeym.Size = UDim2.new(0, 100, 0, 40)
        naeeym.StudsOffset = Vector3.new(0, 2, 0)
        naeeym.Adornee = character:WaitForChild("Head")
        local tecks = Instance.new("TextLabel", naeeym)
        tecks.BackgroundTransparency = 1
        tecks.Font = Enum.Font.Fantasy
        tecks.TextSize = 24
        tecks.TextColor3 = Color3.new(0, 0, 0)
        tecks.Size = UDim2.new(1, 0, 0.5, 0)
        tecks.Text = "JOHN DOE"
        task.spawn(function()
            while character:IsDescendantOf(workspace) do
                tecks.Text = "BURN IN HELL"
                task.wait(0.3)
                tecks.Text = "JUST GIVE UP"
                task.wait(0.3)
                tecks.Text = "HOPELESS"
                task.wait(0.3)
                tecks.Text = "JOHN DOE"
                task.wait(0.3)
            end
        end)
    end
    player.CharacterAdded:Connect(setupCharacter)
    if player.Character then setupCharacter(player.Character) end
    notify("JOHN DOE ativado.", "sucesso")
end)

button(cTroll, "Decal Spam", function()
    local decalID = 8408806737
    local function exPro(root)
        for _, v in pairs(root:GetChildren()) do
            if v:IsA("Decal") and v.Texture ~= "http://www.roblox.com/asset/?id=" .. decalID then
                v.Parent = nil
            elseif v:IsA("BasePart") then
                v.Material = "Plastic"
                v.Transparency = 0
                for _, face in ipairs({ "Front", "Back", "Right", "Left", "Top", "Bottom" }) do
                    local d = Instance.new("Decal", v)
                    d.Texture = "http://www.roblox.com/asset/?id=" .. decalID
                    d.Face = face
                end
            end
            exPro(v)
        end
    end
    exPro(workspace)
    local s = Instance.new("Sky")
    s.Name = "Sky"
    s.Parent = Lighting
    local sid = "http://www.roblox.com/asset/?id=" .. decalID
    s.SkyboxBk, s.SkyboxDn, s.SkyboxFt = sid, sid, sid
    s.SkyboxLf, s.SkyboxRt, s.SkyboxUp = sid, sid, sid
    Lighting.TimeOfDay = 12
    notify("Decal Spam aplicado.", "sucesso")
end)local cUtil = card(p, "Utilidades (C00LCLAN)", "Funções originais preservadas")

button(cUtil, "Nameless Admin", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/ltseverydayyou/Nameless-Admin/main/Source.lua"))()
    notify("Nameless carregado.", "sucesso")
end)

button(cUtil, "TP Tool", function()
    loadstring(game:HttpGet("https://pastebin.com/raw/WUwLqAzr"))()
    notify("TP Tool carregado.", "sucesso")
end)

button(cUtil, "GOD Mode Toggle", function()
    loadstring(game:HttpGet("https://pastebin.com/raw/eD8HBXGS"))()
    notify("God Mode carregado.", "sucesso")
end)

button(cUtil, "Server Hop", function()
    local ok, res = pcall(function()
        return game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100&excludeFullGames=true")
    end)
    if not ok then notify("Falha HTTP.", "erro") return end
    local body = HttpService:JSONDecode(res)
    local servers = {}
    for _, v in ipairs(body.data) do
        if v.playing < v.maxPlayers and v.id ~= game.JobId then
            table.insert(servers, v.id)
        end
    end
    if #servers == 0 then notify("Sem servidores.", "erro") return end
    TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], player)
    notify("Teleportando...", "sucesso")
end)

button(cUtil, "Save Position", function()
    local h = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if h then DS.SavedCF = h.CFrame notify("Posição salva.", "sucesso") end
end)

button(cUtil, "Load Position", function()
    local h = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not DS.SavedCF then notify("Nada salvo.", "aviso") return end
    if h then h.CFrame = DS.SavedCF notify("Posição carregada.", "sucesso") end
end)

button(cUtil, "Drift Car", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/AstraOutlight/my-scripts/refs/heads/main/fe%20car%20v3"))()
    notify("Drift Car carregado.", "sucesso")
end)

button(cUtil, "PART ORBIT", function()
    loadstring(game:HttpGet("https://pastebin.com/raw/aZjaAr6F", true))()
    notify("PART ORBIT carregado.", "sucesso")
end)

button(cUtil, "Fling GUI", function()
    loadstring(game:HttpGet("https://pastebin.com/raw/VynRhxJV", true))()
    notify("Fling carregado.", "sucesso")
end)

button(cUtil, "Fly (movement)", function()
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
    local hrp = player.Character.HumanoidRootPart
    if DS.FlyBG then DS.FlyBG:Destroy() DS.FlyBG = nil end
    if DS.FlyBV then DS.FlyBV:Destroy() DS.FlyBV = nil end
    if DS.FlyConn then DS.FlyConn:Disconnect() DS.FlyConn = nil end
    DS.FlyBG = Instance.new("BodyGyro", hrp)
    DS.FlyBG.P = 9e4
    DS.FlyBG.MaxTorque = Vector3.new(9e4, 9e4, 9e4)
    DS.FlyBG.CFrame = hrp.CFrame
    DS.FlyBV = Instance.new("BodyVelocity", hrp)
    DS.FlyBV.MaxForce = Vector3.new(9e4, 9e4, 9e4)
    DS.FlyConn = RunService.Heartbeat:Connect(function()
        local cam = workspace.CurrentCamera.CFrame
        local dir = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then dir = dir + cam.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then dir = dir - cam.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then dir = dir - cam.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then dir = dir + cam.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.Space) then dir = dir + Vector3.new(0, 1, 0) end
        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir = dir - Vector3.new(0, 1, 0) end
        if dir.Magnitude > 0 then dir = dir.Unit * 50 end
        DS.FlyBV.Velocity = dir
        DS.FlyBG.CFrame = CFrame.new(hrp.Position, hrp.Position + cam.LookVector)
    end)
    notify("Fly ativado. Aperte W/A/S/D.", "sucesso")
end)

p = page("Credits")
c = card(p, "Credits", "Créditos do projeto")
label(c, "Script feito por: Davi Scripts")
label(c, "Interface: DS HUB - Dark Blue Edition")
label(c, "C00LCLAN buttons: preservados do original")
label(c, "Infinite Yield: EdgeIY (script externo)")
label(c, "Versão: 1.2")
button(c, "Voltar para Principal", function() show("Principal") end)p = page("Configurações")
c = card(p, "⚙ Configurações Gerais", "Personalização")
toggle(c, "Sons do hub", true, function(on)
    DS.Sons = on
    notify(on and "Sons on." or "Sons off.", "info")
end)
toggle(c, "Animações do hub", true, function(on)
    DS.Animacoes = on
    notify(on and "Animações on." or "Animações off.", "info")
end)
toggle(c, "Brilho pulsante", true, function(on)
    DS.BrilhoAtivo = on
    notify(on and "Brilho on." or "Brilho off.", "info")
end)

local cDiag = card(p, "📊 Diagnóstico", "Informações em tempo real")
local lblFPS = make("TextLabel", { Size = UDim2.new(1, 0, 0, 22), BackgroundTransparency = 1, Text = "FPS: 60  |  Ping: 0ms", Font = Enum.Font.GothamMedium, TextSize = 12, TextColor3 = TXT, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 14 }, cDiag)
local fps = 60
local ping = 0
local frames = 0
local lastTick = tick()
task.spawn(function()
    RunService.RenderStepped:Connect(function() frames = frames + 1 end)
    while lblFPS.Parent do
        if tick() - lastTick >= 1 then
            fps = frames
            frames = 0
            lastTick = tick()
            pcall(function() ping = math.floor(player:GetNetworkPing() * 1000) end)
            lblFPS.Text = string.format("FPS: %d  |  Ping: %dms", fps, ping)
        end
        task.wait(0.5)
    end
end)

local cLog = card(p, "📝 Log de Ações", "Últimas 10 ações")
local logLabel = make("TextLabel", { Size = UDim2.new(1, 0, 0, 150), BackgroundColor3 = Color3.fromRGB(5, 16, 36), Text = "Nenhuma ação registrada.", TextWrapped = true, TextColor3 = MUTED, Font = Enum.Font.Code, TextSize = 10, TextXAlignment = Enum.TextXAlignment.Left, TextYAlignment = Enum.TextYAlignment.Top, BorderSizePixel = 0, ZIndex = 14 }, cLog)
round(logLabel, 10)
outline(logLabel, GRID, 0.45)
make("UIPadding", { PaddingTop = UDim.new(0, 6), PaddingLeft = UDim.new(0, 6), PaddingRight = UDim.new(0, 6) }, logLabel)
local function atualizarLog()
    local txt = ""
    local ini = math.max(1, #DS.Logs - 9)
    for i = ini, #DS.Logs do
        local l = DS.Logs[i]
        txt = txt .. string.format("[%s] %s: %s\n", l.hora, string.upper(l.tipo), l.msg)
    end
    if txt == "" then txt = "Nenhuma ação registrada." end
    logLabel.Text = txt
end
button(cLog, "🗑 Limpar log", function()
    DS.Logs = {}
    atualizarLog()
    notify("Log limpo.", "sucesso")
end)

local cExtra = card(p, "⚡ Extras", "Funções adicionais")
toggle(c, "Anti-AFK", false, function(on)
    if DS.AntiAFK then DS.AntiAFK:Disconnect() DS.AntiAFK = nil end
    if on then
        DS.AntiAFK = player.Idled:Connect(function()
            VU:CaptureController()
            VU:ClickButton2(Vector2.new())
        end)
        notify("Anti-AFK on.", "sucesso")
    else
        notify("Anti-AFK off.", "info")
    end
end)
button(cExtra, "🚪 Rejoin", function()
    notify("Reconectando...", "aviso")
    task.wait(0.5)
    TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, player)
end)
toggle(cExtra, "Modo Compacto", false, function(on)
    local sz = on and UDim2.new(0, 480, 0, 320) or UDim2.new(0.82, 0, 0.84, 0)
    TweenService:Create(main, TweenInfo.new(0.3), { Size = sz }):Play()
    notify(on and "Compacto on." or "Normal.", "info")
end)
button(cExtra, "⚠ Restaurar TUDO", function()
    local conf = Instance.new("TextButton")
    conf.Size = UDim2.new(1, 0, 0, 36)
    conf.BackgroundColor3 = WARN
    conf.Text = "⚠ CLIQUE DE NOVO PARA CONFIRMAR"
    conf.TextColor3 = Color3.new(0, 0, 0)
    conf.Font = Enum.Font.GothamBold
    conf.TextSize = 11
    conf.BorderSizePixel = 0
    conf.ZIndex = 20
    conf.Parent = cExtra
    round(conf, 12)
    local cancelou = false
    conf.MouseButton1Click:Connect(function()
        cancelou = true
        conf:Destroy()
        for k, v in pairs(original) do Lighting[k] = v end
        for _, s in ipairs(originalSkies) do s:Clone().Parent = Lighting end
        DS.Toggles = {}
        notify("TUDO restaurado.", "sucesso")
    end)
    task.delay(4, function()
        if not cancelou and conf.Parent then
            conf:Destroy()
            notify("Restauração cancelada.", "info")
        end
    end)
end)

for _, pgObj in pairs(pages) do registerSearchable(pgObj) end

local isOpen = false
local animating = false
local openSize = UDim2.new(0.82, 0, 0.84, 0)
local closedSize = UDim2.fromOffset(12, 12)

local function openPanel()
    if isOpen or animating or not gui.Parent then return end
