-- =====================================================
-- TESTE DE CARGA NO PETVAULT (ARMAZÉM DE PETS)
// Use APENAS na cópia do jogo
-- =====================================================

-- \\ Seu código original aqui até a parte da UI...
-- [mantenha todo o código original até a linha dos botões]

-- ========== CONFIGURAÇÃO ESPECÍFICA PARA PETVAULT ==========
local TEST_CONFIG = {
    -- Intensidade do teste
    intensidade = {
        payloadKB = 500,          -- Comece com 500KB (metade do original)
        chamadasPorSegundo = 3,    -- 3 chamadas por segundo
        duracaoSegundos = 20,      -- 20 segundos de teste
    },
    
    -- IMPORTANTE: agora temos 3 modos de teste
    modoTeste = {
        nome = "original",         -- "original", "leve", "pesado"
        descricao = "Cópia fiel do exploit antigo (4.2M caracteres)"
    }
}

-- ========== FUNÇÕES DE TESTE OTIMIZADAS ==========

-- Pega o menor pet (igual ao original)
function getLowestPet(Size)
    local LevelCap = Size or math.huge
    for _, Data in pairs(require(game.ReplicatedStorage.ModuleScripts.LocalDairebStore).GetStoreProxy("GameData"):GetData("Pets")) do
        if not Data.Locked and Data.Level < LevelCap then
            return Data.UID
        end
    end
    return "UID_TESTE_123"  -- fallback
end

-- Gera payload igual ao exploit (4.2M de '9's) ou versões menores
local function gerarPayload(tamanhoEmKB)
    -- Se for 4200KB, é exatamente o original (4.2M caracteres)
    return string.rep("9", tamanhoEmKB * 1024)
end

-- Teste específico para PetVault
local function testarPetVault()
    print("\n🔬 TESTANDO PETVAULT - ARMAZÉM DE PETS")
    print("Modo:", TEST_CONFIG.modoTeste.nome)
    print("Payload:", TEST_CONFIG.intensidade.payloadKB, "KB")
    print("Chamadas/s:", TEST_CONFIG.intensidade.chamadasPorSegundo)
    print("Duração:", TEST_CONFIG.intensidade.duracaoSegundos, "s\n")
    
    local remote = ReplicatedStorage:FindFirstChild("Remote") and 
                   ReplicatedStorage.Remote:FindFirstChild("PetVault")
    
    if not remote then
        Status.Text = 'Status: <font color="#ff5252">PetVault não encontrado!</font>'
        return
    end
    
    local stats = {
        chamadas = 0,
        sucessos = 0,
        falhas = 0,
        inicio = os.clock()
    }
    
    local totalChamadas = TEST_CONFIG.intensidade.duracaoSegundos * TEST_CONFIG.intensidade.chamadasPorSegundo
    local payload = gerarPayload(TEST_CONFIG.intensidade.payloadKB)
    
    for i = 1, totalChamadas do
        stats.chamadas = stats.chamadas + 1
        
        -- Tenta reproduzir EXATAMENTE o exploit
        local sucesso, erro = pcall(function()
            local petUID = getLowestPet()
            -- OS 3 ARGUMENTOS EXATOS DO EXPLOIT:
            -- 1. UID do pet
            -- 2. true (provavelmente "guardar no vault")
            -- 3. string GIGANTE (o payload)
            remote:FireServer(petUID, true, payload)
        end)
        
        if sucesso then
            stats.sucessos = stats.sucessos + 1
        else
            stats.falhas = stats.falhas + 1
        end
        
        -- Atualiza interface
        if i % math.floor(totalChamadas/5) == 0 then
            local percentual = math.floor((i/totalChamadas)*100)
            Status.Text = string.format('Status: <font color="#ffc249">Testando PetVault... %d%%</font>', percentual)
        end
        
        wait(1 / TEST_CONFIG.intensidade.chamadasPorSegundo)
    end
    
    -- Relatório específico
    local elapsed = os.clock() - stats.inicio
    print("\n📊 RELATÓRIO - TESTE DO PETVAULT")
    print("━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    print("Duração real: %.1fs"):format(elapsed)
    print("Chamadas: %d"):format(stats.chamadas)
    print("Sucessos: %d"):format(stats.sucessos)
    print("Falhas: %d"):format(stats.falhas)
    print("Taxa: %.1f chamadas/s"):format(stats.chamadas/elapsed)
    
    -- Análise específica
    if stats.falhas > 0 then
        print("\n🛡️ SERVIDOR REJEITOU", stats.falhas, "chamadas - PROTEÇÃO ATIVA!")
        if stats.falhas == stats.chamadas then
            print("✅ BLOQUEIO TOTAL - servidor rejeitou TODAS as tentativas")
        else
            print("⚠️ BLOQUEIO PARCIAL - algumas chamadas passaram")
        end
    else
        print("\n❌ VULNERABILIDADE DETECTADA!")
        print("   Todas as", stats.chamadas, "chamadas foram aceitas")
        print("   O servidor NÃO está rejeitando dados grandes!")
        print("   Implemente URGENTE:")
        print("   - Limite de tamanho de payload")
        print("   - Rate limiting")
        print("   - Validação de dados no servidor")
    end
    
    Status.Text = 'Status: <font color="#85b651">Teste concluído</font>'
end

-- ========== BOTÃO FREEZE DATA MODIFICADO ==========
Buttons.Start.Interact.MouseButton1Click:Connect(function()
    if Enabled then 
        Status.Text = 'Status: <font color="#ffc249">Teste em andamento...</font>'
        return 
    end
    Enabled = true
    ChangeColor(Buttons.Start, Color3.fromRGB(175,175,175), Color3.fromRGB(100,100,100))
    
    -- Executa teste específico do PetVault
    coroutine.wrap(testarPetVault)()
end)

-- ========== BOTÕES PARA MUDAR INTENSIDADE (use os extras) ==========

-- Botão Dungeon = Modo Leve (teste suave)
Buttons2.Dungeon.Interact.MouseButton1Click:Connect(function()
    TEST_CONFIG.intensidade.payloadKB = 100
    TEST_CONFIG.intensidade.chamadasPorSegundo = 2
    TEST_CONFIG.modoTeste.nome = "leve"
    TEST_CONFIG.modoTeste.descricao = "100KB - teste suave"
    Status.Text = 'Status: <font color="#85b651">Modo LEVE ativado</font>'
    ChangeColor(Buttons2.Dungeon, Color3.fromRGB(116,209,255), Color3.fromRGB(41, 134, 255))
    print("🟢 Modo LEVE: 100KB, 2 chamadas/s")
end)

-- Botão Dupe Machine = Modo Original (igual ao exploit)
Buttons2.DupeUI.Interact.MouseButton1Click:Connect(function()
    TEST_CONFIG.intensidade.payloadKB = 4200  -- 4.2M caracteres
    TEST_CONFIG.intensidade.chamadasPorSegundo = 1
    TEST_CONFIG.modoTeste.nome = "original"
    TEST_CONFIG.modoTeste.descricao = "4.2M chars - cópia exata do exploit"
    Status.Text = 'Status: <font color="#ff5252">MODO ORIGINAL ATIVADO</font>'
    ChangeColor(Buttons2.DupeUI, Color3.fromRGB(255, 96, 234), Color3.fromRGB(255, 33, 244))
    print("🔴 MODO ORIGINAL: 4.2M caracteres, 1 chamada/s")
end)

-- ========== RESTO DO CÓDIGO ORIGINAL (mantido) ==========
-- [cole aqui o resto do código original, incluindo as funções de UI, drag, etc.]
