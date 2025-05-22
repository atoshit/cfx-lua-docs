# Les Itérations en Lua

## Qu'est-ce qu'une itération ?
Une **itération** en Lua est un mécanisme qui permet de parcourir une collection de données (table, string, etc.) de manière séquentielle. Il existe plusieurs types d'itérations, chacune ayant ses cas d'utilisation spécifiques.

## Types d'itérations

### 1. Boucle for numérique
```lua
-- Syntaxe de base
for i = 1, 10 do
    print(i)
end

-- Avec un pas
for i = 1, 10, 2 do
    print(i) -- Affiche: 1, 3, 5, 7, 9
end

-- En décrément
for i = 10, 1, -1 do
    print(i) -- Affiche: 10, 9, 8, ..., 1
end
```

### 2. Boucle for générique (pairs/ipairs)
```lua
local table = {
    "premier",
    "deuxième",
    "troisième",
    [5] = "cinquième",
    nom = "test"
}

-- ipairs (itération séquentielle)
for i, v in ipairs(table) do
    print(i, v) -- Affiche: 1 "premier", 2 "deuxième", 3 "troisième"
end

-- pairs (itération de toutes les clés)
for k, v in pairs(table) do
    print(k, v) -- Affiche toutes les paires clé-valeur
end
```

## Différences entre ipairs et pairs

| Caractéristique | ipairs | pairs |
|-----------------|---------|--------|
| Ordre | Séquentiel | Non garanti |
| Clés | Numériques uniquement | Tous types |
| Performance | Plus rapide | Plus lent |
| Gaps | S'arrête aux trous | Parcourt tout |
| Utilisation | Tableaux séquentiels | Tables associatives |

## Cas d'utilisation

### 1. ipairs : Pour les tableaux séquentiels
```lua
local joueurs = {
    "Alice",
    "Bob",
    "Charlie"
}

-- Bon usage de ipairs
for i, nom in ipairs(joueurs) do
    print(string.format("Joueur %d: %s", i, nom))
end
```

### 2. pairs : Pour les tables associatives
```lua
local inventaire = {
    pain = 5,
    eau = 3,
    ["clé_maison"] = 1
}

-- Bon usage de pairs
for item, quantite in pairs(inventaire) do
    print(string.format("%s: %d", item, quantite))
end
```

### 3. for numérique : Pour les itérations contrôlées
```lua
-- Création d'une grille
local grille = {}
for i = 1, 10 do
    grille[i] = {}
    for j = 1, 10 do
        grille[i][j] = 0
    end
end
```

## Optimisation

### 1. Préférer ipairs pour les tableaux
```lua
-- Plus rapide
local array = {1, 2, 3, 4, 5}
for i, v in ipairs(array) do
    -- Traitement
end

-- Plus lent
for k, v in pairs(array) do
    -- Traitement
end
```

### 2. Éviter les modifications pendant l'itération
```lua
local table = {1, 2, 3, 4, 5}

-- Mauvais
for i, v in ipairs(table) do
    table[i + 1] = v * 2 -- Peut causer des problèmes
end

-- Bon
local newTable = {}
for i, v in ipairs(table) do
    newTable[i] = v * 2
end
```

### 3. Utiliser la bonne boucle pour le bon cas
```lua
-- Pour un tableau séquentiel
local scores = {100, 200, 300, 400, 500}
for i, score in ipairs(scores) do
    -- Traitement
end

-- Pour une table associative
local config = {
    debug = true,
    maxPlayers = 32,
    serverName = "Mon Serveur"
}
for k, v in pairs(config) do
    -- Traitement
end
```

## Mauvaises pratiques à éviter

### 1. Utiliser pairs pour les tableaux séquentiels
```lua
-- Mauvais
local array = {1, 2, 3, 4, 5}
for k, v in pairs(array) do
    print(v)
end

-- Bon
for i, v in ipairs(array) do
    print(v)
end
```

### 2. Modifier la table pendant l'itération
```lua
-- Mauvais
local table = {1, 2, 3, 4, 5}
for i, v in ipairs(table) do
    if v % 2 == 0 then
        table.remove(table, i)
    end
end

-- Bon
local toRemove = {}
for i, v in ipairs(table) do
    if v % 2 == 0 then
        table.insert(toRemove, i)
    end
end
for i = #toRemove, 1, -1 do
    table.remove(table, toRemove[i])
end
```

### 3. Ignorer les performances
```lua
-- Mauvais (trop d'itérations)
for i = 1, 1000 do
    for j = 1, 1000 do
        for k = 1, 1000 do
            -- Opération coûteuse
        end
    end
end

-- Bon (optimisé)
local cache = {}
for i = 1, 1000 do
    for j = 1, 1000 do
        local key = i .. ":" .. j
        if not cache[key] then
            cache[key] = -- Opération coûteuse
        end
    end
end
```

## À retenir

1. **ipairs** :
   - Pour les tableaux séquentiels
   - Plus performant que pairs
   - S'arrête aux trous dans la séquence

2. **pairs** :
   - Pour les tables associatives
   - Parcourt toutes les clés
   - Ordre non garanti

3. **for numérique** :
   - Pour les itérations contrôlées
   - Parfait pour les boucles avec un nombre fixe d'itérations
   - Plus performant que les autres types de boucles

4. **Bonnes pratiques** :
   - Choisir la bonne boucle pour le bon cas
   - Éviter les modifications pendant l'itération
   - Optimiser les performances quand nécessaire
   - Utiliser des caches pour les opérations coûteuses
