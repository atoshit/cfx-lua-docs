
# Le Cache Dynamique en Lua (FiveM)

## Qu’est-ce qu’un cache dynamique ?
Un **cache dynamique** permet de **mémoriser temporairement des données** pour éviter des appels répétés coûteux (ex : requêtes SQL, calculs).  
En Lua (et particulièrement sur FiveM), cela permet de gagner en performance, surtout quand on interagit souvent avec la base de données.

---

## ⚙Fonctionnement de base

```lua
local cache = {}

function getData(key)
    if cache[key] then
        return cache[key]
    end

    local result = MySQL.Sync.fetchAll("SELECT * FROM my_table WHERE id = @id", {
        ['@id'] = key
    })

    cache[key] = result[1]
    return result[1]
end
```

➡Le cache garde les données après la première requête, évitant des accès répétés à la base.

---

## Problème : Le cache devient obsolète ?

Solution : utiliser des **tables faibles** pour que les données soient automatiquement nettoyées si elles ne sont plus utilisées ailleurs.

---

## Tables faibles (`weak tables`) + cache

```lua
local cache = setmetatable({}, { __mode = "v" }) -- valeurs faibles

function getProperty(id)
    if cache[id] then
        return cache[id]
    end

    local result = MySQL.Sync.fetchAll("SELECT * FROM properties WHERE id = @id", {
        ['@id'] = id
    })

    cache[id] = result[1]
    return result[1]
end
```

Le **garbage collector** de Lua supprimera automatiquement les entrées dont les données ne sont plus utilisées ailleurs.

---

## Cas d’utilisation concrets sur FiveM

### 1. Caching de propriétés
```lua
local propertyCache = setmetatable({}, { __mode = "v" })

function getPropertyData(propertyId)
    if propertyCache[propertyId] then
        return propertyCache[propertyId]
    end

    local data = MySQL.Sync.fetchAll("SELECT * FROM properties WHERE id = @id", {
        ['@id'] = propertyId
    })

    propertyCache[propertyId] = data[1]
    return data[1]
end
```

---

### 2. Caching d’entités du monde temporaire (objets, véhicules)
```lua
local tempVehicles = setmetatable({}, { __mode = "v" })

function spawnTempVehicle(netId)
    local entity = NetworkGetEntityFromNetworkId(netId)
    local wrapper = { entity = entity }
    tempVehicles[netId] = wrapper
end
```

➡Le véhicule est supprimé du cache quand il n’est plus référencé nulle part.

---

### 3. Caching d’objets utilitaires comme les zones
```lua
local zoneCache = setmetatable({}, { __mode = "v" })

function getZone(id)
    if zoneCache[id] then
        return zoneCache[id]
    end

    local zone = CreateZoneForID(id)
    zoneCache[id] = zone
    return zone
end
```

---

## Mauvais usage : données persistantes (joueurs)

```lua
-- ⚠À éviter
local playerData = setmetatable({}, { __mode = "v" })

function getPlayerData(src)
    if not playerData[src] then
        playerData[src] = {
            name = GetPlayerName(src),
            inventory = {},
        }
    end
    return playerData[src]
end
```

Ici, les données joueur doivent être **persistantes** : ne pas utiliser de table faible.

---

## Bonus : Forcer le nettoyage du cache

```lua
collectgarbage("collect")
```

Cela force Lua à lancer le **ramasse-miettes** (utile pour tester le cache dynamique).

---

## À retenir

- Le **cache dynamique** est essentiel pour éviter les requêtes répétées.
- Utilise les **tables faibles** pour que les données soient **supprimées automatiquement** si inutilisées.
- ⚠N’utilise PAS les tables faibles pour des données **critiques ou persistantes**.

