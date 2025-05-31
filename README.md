--[[
    Script: Auto Colheita e Recompensa de Jardim
    Localização: Este script deve ser colocado em ServerScriptService.

    Descrição:
    Este script monitora continuamente os modelos de plantas no Workspace.
    Quando uma planta atinge a maturidade (indicada pela presença de uma 'Fruit' Part),
    o script automaticamente "colhe" a fruta (destruindo a Part da fruta)
    e recompensa o jogador proprietário da planta, adicionando um valor aos seus leaderstats.

    Recursos:
    - Auto Colheita: Detecta e remove frutas maduras automaticamente.
    - Auto TP Fruta (Recompensa): Adiciona o valor da fruta diretamente ao "Cash" do jogador.
]]

--#################################################################################################################
--                                             CONFIGURAÇÕES
--#################################################################################################################

local HARVEST_CHECK_INTERVAL = 5 -- Intervalo em segundos para verificar por plantas maduras.
                                 -- Um valor menor verifica mais frequentemente, mas pode usar mais recursos.
local FRUIT_VALUE = 10           -- O valor em dinheiro/pontos que cada fruta colhida concede ao jogador.
local PLANT_MODEL_NAME = "Plant" -- O nome dos seus modelos de planta no Workspace (ex: "Plant", "CarrotPlant").
local FRUIT_PART_NAME = "Fruit"  -- O nome da Part dentro do modelo da planta que representa a fruta madura (ex: "Carrot", "Apple").
local OWNER_VALUE_NAME = "Owner" -- O nome do StringValue (ou ObjectValue) dentro do modelo da planta que armazena o UserId do proprietário.
local CURRENCY_NAME = "Cash"     -- O nome do NumberValue nos leaderstats do jogador que você quer atualizar (ex: "Cash", "Coins").

--#################################################################################################################
--                                             SERVIÇOS
--#################################################################################################################

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService") -- Usaremos RunService para um loop mais eficiente

--#################################################################################################################
--                                             FUNÇÕES
--#################################################################################################################

--- Função para colher uma fruta de uma planta madura e recompensar o proprietário.
-- @param plantModel O modelo da planta que contém a fruta.
-- @param fruitPart A Part que representa a fruta madura.
-- @param ownerPlayer O objeto Player que é o proprietário da planta.
local function harvestFruit(plantModel, fruitPart, ownerPlayer)
    -- Verifica se o jogador ainda existe e tem leaderstats
    if not ownerPlayer or not ownerPlayer:FindFirstChild("leaderstats") then
        warn("Jogador proprietário não encontrado ou sem leaderstats para a planta: " .. plantModel.Name)
        return
    end

    local leaderstats = ownerPlayer.leaderstats
    local currency = leaderstats:FindFirstChild(CURRENCY_NAME)

    if not currency or not currency:IsA("NumberValue") then
        warn("Currency '" .. CURRENCY_NAME .. "' não encontrado ou não é um NumberValue nos leaderstats de " .. ownerPlayer.Name)
        return
    end

    -- Destrói a Part da fruta (simulando a colheita)
    fruitPart:Destroy()

    -- Adiciona o valor da fruta ao dinheiro do jogador (auto TP da fruta para o inventário/dinheiro)
    currency.Value = currency.Value + FRUIT_VALUE

    print(string.format("%s colheu %s do lote '%s' e ganhou %d de %s.",
        ownerPlayer.Name, FRUIT_PART_NAME, plantModel.Name, FRUIT_VALUE, CURRENCY_NAME))

    --[[
        IMPORTANTE: Lógica de Regrowth da Planta
        Após a colheita, você precisará de um sistema no seu jogo para:
        1. Resetar o estado da planta (ex: para "Empty" ou "Seed").
        2. Iniciar um novo ciclo de crescimento para que a planta possa produzir outra fruta.
        Isso geralmente é feito por um script separado dentro do modelo da planta ou por um sistema de gerenciamento de jardim.
        Este script de auto-colheita apenas remove a fruta e recompensa.
        Exemplo (descomente e adapte se tiver um sistema de crescimento):
        if plantModel:FindFirstChild("GrowthStage") and plantModel.GrowthStage:IsA("StringValue") then
            plantModel.GrowthStage.Value = "Empty" -- Define o estágio de volta para vazio
        end
    ]]
end

--- Função principal para verificar e colher plantas maduras.
local function checkAndHarvestPlants()
    for _, child in ipairs(Workspace:GetChildren()) do
        -- Verifica se o child é um modelo de planta configurado
        if child:IsA("Model") and child.Name == PLANT_MODEL_NAME then
            local fruitPart = child:FindFirstChild(FRUIT_PART_NAME)
            local ownerValue = child:FindFirstChild(OWNER_VALUE_NAME)

            -- Se a Part da fruta e o valor do proprietário existirem, a planta está madura e tem um dono
            if fruitPart and fruitPart:IsA("BasePart") and ownerValue and ownerValue:IsA("StringValue") then
                local ownerUserId = ownerValue.Value
                local ownerPlayer = Players:GetPlayerByUserId(tonumber(ownerUserId)) -- Converte para número

                if ownerPlayer then
                    -- Se o proprietário for encontrado, colhe a fruta
                    harvestFruit(child, fruitPart, ownerPlayer)
                else
                    warn("Proprietário com UserId " .. ownerUserId .. " não encontrado para a planta " .. child.Name)
                end
            end
        end
    end
end

--#################################################################################################################
--                                             LOOP PRINCIPAL
--#################################################################################################################

-- Usa um loop while true com wait para verificar periodicamente as plantas.
-- RunService.Heartbeat pode ser uma alternativa mais performática para jogos complexos.
while true do
    checkAndHarvestPlants()
    wait(HARVEST_CHECK_INTERVAL)
end

--#################################################################################################################
--                                             SETUP INICIAL (Exemplo)
--#################################################################################################################
-- Este é um exemplo de como você pode configurar leaderstats para novos jogadores.
-- Se você já tem um sistema de leaderstats, pode ignorar esta parte.

Players.PlayerAdded:Connect(function(player)
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player

    local cash = Instance.new("NumberValue")
    cash.Name = CURRENCY_NAME
    cash.Value = 0
    cash.Parent = leaderstats

    print(player.Name .. " entrou no jogo e seus leaderstats foram criados.")
end)
![1000115344](https://github.com/user-attachments/assets/44663fde-3f06-4894-810c-48f227fcbace)

