# Les Events en FiveM

## Qu'est-ce qu'un event ?
Un **event** (événement) en FiveM est un système de communication entre les différentes parties de votre serveur. Il permet d'envoyer des messages entre :
- Le client et le serveur
- Les différentes ressources
- Les différents clients

## À quoi ça sert ?

| Utilisation courante | Description |
|---------------------|-------------|
| Communication client-serveur | Envoyer des données du client au serveur et vice-versa |
| Synchronisation | Partager des informations entre tous les joueurs |
| Déclenchement d'actions | Lancer des actions suite à un événement |
| Modularité | Séparer le code en modules communicants |

## Fonctionnement de base

### Côté serveur
```lua
-- Enregistrer un event
RegisterNetEvent('monEvent')
AddEventHandler('monEvent', function(data)
    print('Event reçu avec les données:', data)
end)

-- Déclencher un event
TriggerClientEvent('monEvent', source, {message = 'Hello!'})
```

### Côté client
```lua
-- Enregistrer un event
RegisterNetEvent('monEvent')
AddEventHandler('monEvent', function(data)
    print('Event reçu avec les données:', data)
end)

-- Déclencher un event
TriggerServerEvent('monEvent', {message = 'Hello!'})
```

## Sécurité des events

### 1. Validation des données
```lua
RegisterNetEvent('monEvent')
AddEventHandler('monEvent', function(data)
    -- Vérifier que les données sont valides
    if type(data) ~= 'table' then return end
    if not data.message then return end
    
    -- Continuer le traitement
    print(data.message)
end)
```

### 2. Rate Limiting
```lua
local cooldowns = {}

RegisterNetEvent('monEvent')
AddEventHandler('monEvent', function()
    local source = source
    local currentTime = os.time()
    
    -- Vérifier le cooldown
    if cooldowns[source] and currentTime - cooldowns[source] < 1 then
        return
    end
    
    cooldowns[source] = currentTime
    -- Continuer le traitement
end)
```

### 3. Vérification des permissions
```lua
RegisterNetEvent('adminEvent')
AddEventHandler('adminEvent', function()
    local source = source
    
    -- Vérifier si le joueur est admin
    if not IsPlayerAceAllowed(source, 'command.admin') then
        return
    end
    
    -- Continuer le traitement
end)
```

## Bonnes pratiques

### 1. Nommage des events
```lua
-- Bon
TriggerEvent('esx:playerLoaded', playerId)

-- Mauvais
TriggerEvent('event1', data)
```

### 2. Structure des données
```lua
-- Bon
TriggerEvent('inventory:addItem', {
    item = 'bread',
    count = 1,
    metadata = {
        quality = 100
    }
})

-- Mauvais
TriggerEvent('addItem', 'bread', 1, 100)
```

### 3. Gestion des erreurs
```lua
RegisterNetEvent('monEvent')
AddEventHandler('monEvent', function(data)
    local success, result = pcall(function()
        -- Code qui pourrait échouer
        return processData(data)
    end)
    
    if not success then
        print('Erreur:', result)
        return
    end
end)
```

## Cas d'usage concrets

### 1. Système d'inventaire
```lua
-- Côté serveur
RegisterNetEvent('inventory:useItem')
AddEventHandler('inventory:useItem', function(itemId)
    local source = source
    local player = ESX.GetPlayerFromId(source)
    
    if not player then return end
    
    local item = player.getInventoryItem(itemId)
    if not item or item.count < 1 then return end
    
    -- Logique d'utilisation
    player.removeInventoryItem(itemId, 1)
    TriggerClientEvent('inventory:itemUsed', source, itemId)
end)
```

### 2. Système de véhicules
```lua
-- Côté client
RegisterNetEvent('vehicles:spawnVehicle')
AddEventHandler('vehicles:spawnVehicle', function(model)
    local hash = GetHashKey(model)
    RequestModel(hash)
    
    while not HasModelLoaded(hash) do
        Wait(0)
    end
    
    local vehicle = CreateVehicle(hash, GetEntityCoords(PlayerPedId()), GetEntityHeading(PlayerPedId()), true, false)
    SetEntityAsMissionEntity(vehicle, true, true)
    
    TriggerServerEvent('vehicles:vehicleSpawned', NetworkGetNetworkIdFromEntity(vehicle))
end)
```

## Fonctionnement réseau des events

### Architecture réseau
```lua
Client A ──────┐
               │
Client B ──────┼─── Serveur FiveM ──── Base de données
               │
Client C ──────┘
```

### Types de communication

1. **Client vers Serveur (C2S)**
```lua
-- Client
TriggerServerEvent('monEvent', data)

-- Serveur
RegisterNetEvent('monEvent')
AddEventHandler('monEvent', function(data)
    -- Traitement côté serveur
end)
```
- Les données sont envoyées via le protocole UDP
- Le serveur valide l'origine (source) de l'event
- Les données sont sérialisées avant l'envoi

2. **Serveur vers Client (S2C)**
```lua
-- Serveur
TriggerClientEvent('monEvent', target, data)

-- Client
RegisterNetEvent('monEvent')
AddEventHandler('monEvent', function(data)
    -- Traitement côté client
end)
```
- Le serveur peut cibler un client spécifique ou tous les clients
- Les données sont répliquées à tous les clients ciblés

3. **Client vers Client (C2C)**
```lua
-- Client A
TriggerServerEvent('relayEvent', targetId, data)

-- Serveur
RegisterNetEvent('relayEvent')
AddEventHandler('relayEvent', function(targetId, data)
    TriggerClientEvent('monEvent', targetId, data)
end)

-- Client B
RegisterNetEvent('monEvent')
AddEventHandler('monEvent', function(data)
    -- Traitement côté client
end)
```
- Les clients ne peuvent pas communiquer directement entre eux
- La communication passe toujours par le serveur

### Sérialisation des données
```lua
-- Données simples
TriggerServerEvent('simpleEvent', "texte", 123, true)

-- Données complexes
TriggerServerEvent('complexEvent', {
    position = vector3(100, 200, 300),
    rotation = vector3(0, 0, 90),
    metadata = {
        owner = "player1",
        timestamp = os.time()
    }
})
```
- Les données sont automatiquement sérialisées en JSON
- Certains types de données ne peuvent pas être sérialisés (fonctions, userdata)
- La taille maximale d'un event est limitée (environ 32KB)

### Gestion de la latence
```lua
-- Côté client
local lastUpdate = 0
local updateInterval = 1000 -- 1 seconde

CreateThread(function()
    while true do
        if GetGameTimer() - lastUpdate > updateInterval then
            TriggerServerEvent('sync:position', GetEntityCoords(PlayerPedId()))
            lastUpdate = GetGameTimer()
        end
        Wait(0)
    end
end)
```
- Les events ne sont pas instantanés
- La latence varie selon la connexion
- Il faut prévoir des mécanismes de synchronisation

### Optimisation réseau
```lua
-- Mauvais : Trop d'events
CreateThread(function()
    while true do
        TriggerServerEvent('update:position', GetEntityCoords(PlayerPedId()))
        Wait(0)
    end
end)

-- Bon : Regroupement des données
local lastSync = 0
local syncInterval = 1000

CreateThread(function()
    while true do
        if GetGameTimer() - lastSync > syncInterval then
            local data = {
                position = GetEntityCoords(PlayerPedId()),
                health = GetEntityHealth(PlayerPedId()),
                armor = GetPedArmour(PlayerPedId())
            }
            TriggerServerEvent('sync:playerData', data)
            lastSync = GetGameTimer()
        end
        Wait(0)
    end
end)
```
- Regrouper les données pour réduire le nombre d'events
- Utiliser des intervalles de synchronisation appropriés
- Éviter les events inutiles

### Sécurité réseau
```lua
-- Côté serveur
local eventBlacklist = {
    'esx:getSharedObject',
    'esx_ambulancejob:revive'
}

AddEventHandler('onResourceStart', function(resourceName)
    if GetCurrentResourceName() ~= resourceName then return end
    
    for i = 1, #eventBlacklist do
        local eventName = eventBlacklist[i]
        RemoveEventHandler(eventName)
    end
end)
```
- Les events peuvent être interceptés
- Certains events sont sensibles et doivent être protégés
- Utiliser des systèmes de validation côté serveur

## Points importants sur le réseau

1. **Latence** : Les events ne sont pas instantanés, prévoir une latence
2. **Bande passante** : Éviter d'envoyer trop de données
3. **Synchronisation** : Utiliser les events pour synchroniser l'état du jeu
4. **Optimisation** : Regrouper les events quand possible
5. **Debug** : Utiliser les outils de debug de FiveM pour tracer les events
