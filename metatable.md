
# 📘 Cours Avancé : Les Metatables en Lua

## 🔍 Qu’est-ce qu’une metatable ?
En Lua, **toutes les tables sont dynamiques**. Une **metatable** est une **table spéciale** qu’on peut associer à une autre table pour **modifier ou étendre son comportement** : surcharge d’opérateurs, gestion d’accès à des clés manquantes, définition de comportements personnalisés, etc.

## 🧠 À quoi ça sert ?

| Utilisation courante                | Description |
|------------------------------------|-------------|
| Surcharge d’opérateurs             | Redéfinir le comportement de `+`, `-`, `==`, etc. |
| Comportement par défaut / héritage | Simuler des classes ou prototypes |
| Accès dynamique à des clés         | Utiliser `__index` ou `__newindex` |
| Contrôle de lecture/écriture       | Bloquer ou logger les modifications |

## ⚙️ Fonctionnement de base

```lua
local t = {}
local mt = {}

setmetatable(t, mt)
```

> `t` est la table normale, `mt` est la metatable qu’on lui associe.

## 🔑 Principaux champs de metatables

| Clé        | Description |
|------------|-------------|
| `__index`  | Appelé quand une clé **n'existe pas** dans la table |
| `__newindex` | Appelé quand une **nouvelle clé** est affectée |
| `__add`, `__sub`, `__mul`, etc. | Surcharge des opérateurs |
| `__eq`, `__lt`, `__le` | Comparaison personnalisée |
| `__tostring` | Affichage personnalisé avec `print()` |
| `__call` | Rendre une table **appelable comme une fonction** |

## 🧪 Exemple 1 : Simuler un objet avec `__index`

```lua
local Person = {}
Person.__index = Person

function Person:new(name)
    local obj = setmetatable({}, self)
    obj.name = name
    return obj
end

function Person:greet()
    print("Salut, je suis " .. self.name)
end

local p = Person:new("Luc")
p:greet()  -- Affiche : Salut, je suis Luc
```

## 🔐 Exemple 2 : Protection en écriture avec `__newindex`

```lua
local secureTable = {}
local proxy = {}

setmetatable(proxy, {
    __index = secureTable,
    __newindex = function(t, key, value)
        error("Écriture interdite sur cette table")
    end
})

proxy.test = 123 -- ❌ Provoque une erreur
```

## ➕ Exemple 3 : Surcharge d’opérateur `+`

```lua
local Vector = {}
Vector.__index = Vector

function Vector:new(x, y)
    return setmetatable({x = x, y = y}, self)
end

function Vector.__add(a, b)
    return Vector:new(a.x + b.x, a.y + b.y)
end

function Vector:__tostring()
    return "(" .. self.x .. ", " .. self.y .. ")"
end

local v1 = Vector:new(1, 2)
local v2 = Vector:new(3, 4)
local v3 = v1 + v2

print(v3) -- Affiche : (4, 6)
```

## ☎️ Exemple 4 : Table appelable avec `__call`

```lua
local counter = setmetatable({value = 0}, {
    __call = function(self)
        self.value = self.value + 1
        return self.value
    end
})

print(counter()) -- 1
print(counter()) -- 2
```

## 📊 Différences par rapport aux tables normales

| Fonction | Table normale | Table avec metatable |
|----------|----------------|-----------------------|
| Accès clé inconnue | Retourne `nil` | Peut déclencher `__index` |
| Affectation clé inconnue | Ajoute la clé | Peut déclencher `__newindex` |
| Comparaison (`==`) | Compare les références | Peut être personnalisée |
| Addition (`+`) | Impossible | Possible avec `__add` |
| Appel (`()`) | Erreur | Possible avec `__call` |

## 🔧 Cas d’usage concrets en développement Lua / FiveM

| Cas d’usage                        | Exemple |
|------------------------------------|---------|
| Système d'inventaire objet         | Objets avec surcharge de `+`, `==`, `__index` |
| Wrapper réseau (`NetEntity`)       | Table avec surcharge `__tostring` + `__call` |
| Système de permissions dynamiques  | Lecture sécurisée via `__index` |
| Simulation de classes / objets     | Utilisation de `:new()` + `__index` |
| Debugging / logging                | Espionner les écritures via `__newindex` |

## 📌 Bonnes pratiques

- Utilise `__index` pour **implémenter l’héritage ou le fallback**.
- Utilise `__tostring` pour rendre tes objets **plus lisibles** dans les logs.
- Ne surcharge pas tout **sans raison** : chaque metatable ajoute une complexité.

## 🧠 À retenir

- Les metatables permettent d’ajouter **des comportements avancés aux tables**.
- Tu peux créer des **objets orientés objet**, des **proxies sécurisés**, ou même des **DSLs en Lua**.
- C’est un outil **puissant** pour structurer un gros projet (ex: framework RP dans FiveM).
