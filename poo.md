# La Programmation Orientée Objet en Lua

## Qu'est-ce que la POO ?
La **Programmation Orientée Objet** (POO) est un paradigme de programmation qui organise le code autour d'objets plutôt que des actions. En Lua, la POO peut être implémentée de plusieurs façons, avec ou sans metatables.

## Concepts fondamentaux de la POO

| Concept | Description |
|---------|-------------|
| Classe | Plan de construction d'un objet |
| Objet | Instance d'une classe |
| Héritage | Capacité d'une classe à hériter des propriétés d'une autre |
| Encapsulation | Protection des données internes |
| Polymorphisme | Capacité d'un objet à prendre plusieurs formes |

## Implémentation de la POO en Lua

### 1. POO sans metatables (Approche simple)
```lua
-- Définition d'une classe simple
local Person = {
    name = "",
    age = 0
}

-- Constructeur
function Person.new(name, age)
    local self = {}
    self.name = name
    self.age = age
    return self
end

-- Méthode
function Person:sayHello()
    print("Bonjour, je m'appelle " .. self.name)
end

-- Utilisation
local person = Person.new("Alice", 25)
person:sayHello()
```

### 2. POO avec metatables (Approche avancée)
```lua
-- Définition d'une classe avec metatable
local Animal = {}
Animal.__index = Animal

-- Constructeur
function Animal.new(name)
    local self = setmetatable({}, Animal)
    self.name = name
    return self
end

-- Méthode
function Animal:makeSound()
    print("L'animal fait un son")
end

-- Héritage
local Dog = {}
Dog.__index = Dog
setmetatable(Dog, {__index = Animal})

function Dog.new(name)
    local self = Animal.new(name)
    setmetatable(self, Dog)
    return self
end

function Dog:makeSound()
    print("Woof! Je m'appelle " .. self.name)
end

-- Utilisation
local dog = Dog.new("Rex")
dog:makeSound() -- Affiche: "Woof! Je m'appelle Rex"
```

## Cas d'utilisation de la POO

### 1. Système d'inventaire
```lua
-- Classe Item
local Item = {}
Item.__index = Item

function Item.new(name, weight, value)
    local self = setmetatable({}, Item)
    self.name = name
    self.weight = weight
    self.value = value
    return self
end

function Item:getInfo()
    return string.format("%s (%.1f kg, %d$)", self.name, self.weight, self.value)
end

-- Classe Inventory
local Inventory = {}
Inventory.__index = Inventory

function Inventory.new(maxWeight)
    local self = setmetatable({}, Inventory)
    self.items = {}
    self.maxWeight = maxWeight
    return self
end

function Inventory:addItem(item)
    local totalWeight = 0
    for _, i in ipairs(self.items) do
        totalWeight = totalWeight + i.weight
    end
    
    if totalWeight + item.weight <= self.maxWeight then
        table.insert(self.items, item)
        return true
    end
    return false
end

-- Utilisation
local sword = Item.new("Épée", 2.5, 100)
local inventory = Inventory.new(10)
inventory:addItem(sword)
```

### 2. Système de véhicules
```lua
-- Classe Vehicle (sans metatable)
local Vehicle = {
    maxSpeed = 0,
    fuel = 0
}

function Vehicle.new(maxSpeed, fuel)
    local self = {
        maxSpeed = maxSpeed,
        fuel = fuel
    }
    
    function self:start()
        if self.fuel > 0 then
            print("Le véhicule démarre")
            return true
        end
        return false
    end
    
    function self:refuel(amount)
        self.fuel = self.fuel + amount
    end
    
    return self
end

-- Utilisation
local car = Vehicle.new(200, 50)
car:start()
car:refuel(20)
```

## Différences entre POO et Metatables

### POO sans Metatables
```lua
-- Approche fonctionnelle
local Player = {
    health = 100,
    armor = 0
}

function Player.new(health, armor)
    local self = {
        health = health or 100,
        armor = armor or 0
    }
    
    function self:takeDamage(amount)
        if self.armor > 0 then
            self.armor = self.armor - amount
            if self.armor < 0 then
                self.health = self.health + self.armor
                self.armor = 0
            end
        else
            self.health = self.health - amount
        end
    end
    
    return self
end
```

### Metatables sans POO
```lua
-- Utilisation de metatables pour la surcharge d'opérateurs
local Vector = {}
Vector.__index = Vector

function Vector.new(x, y)
    return setmetatable({x = x, y = y}, Vector)
end

-- Surcharge de l'opérateur +
function Vector.__add(a, b)
    return Vector.new(a.x + b.x, a.y + b.y)
end

-- Utilisation
local v1 = Vector.new(1, 2)
local v2 = Vector.new(3, 4)
local v3 = v1 + v2 -- Utilise la surcharge d'opérateur
```

## Avantages et Inconvénients

### POO avec Metatables
| Avantages | Inconvénients |
|-----------|---------------|
| Héritage plus propre | Plus complexe à comprendre |
| Meilleure encapsulation | Performance légèrement inférieure |
| Plus proche des langages OOP classiques | Plus de code à écrire |

### POO sans Metatables
| Avantages | Inconvénients |
|-----------|---------------|
| Plus simple à comprendre | Moins de fonctionnalités OOP |
| Meilleure performance | Héritage plus complexe |
| Moins de code | Moins de flexibilité |

## Bonnes pratiques

### 1. Choisir la bonne approche
```lua
-- Pour un système simple
local Config = {
    debug = false,
    maxPlayers = 32
}

-- Pour un système complexe avec héritage
local Entity = {}
Entity.__index = Entity

function Entity.new()
    return setmetatable({}, Entity)
end

local Player = {}
Player.__index = Player
setmetatable(Player, {__index = Entity})
```

### 2. Encapsulation
```lua
local BankAccount = {}
BankAccount.__index = BankAccount

function BankAccount.new(balance)
    local self = setmetatable({}, BankAccount)
    self._balance = balance -- Le underscore indique une propriété privée
    return self
end

function BankAccount:getBalance()
    return self._balance
end

function BankAccount:deposit(amount)
    if amount > 0 then
        self._balance = self._balance + amount
        return true
    end
    return false
end
```

## À retenir

1. **POO en Lua** :
   - Peut être implémentée avec ou sans metatables
   - Les metatables ne sont pas obligatoires pour la POO
   - Choisir l'approche selon la complexité du projet

2. **Metatables** :
   - Utiles pour la POO mais pas uniquement
   - Peuvent être utilisées pour d'autres fonctionnalités
   - Ajoutent de la complexité mais plus de flexibilité

3. **Bonnes pratiques** :
   - Utiliser la POO pour les systèmes complexes
   - Préférer l'approche simple pour les petits projets
   - Bien documenter la structure des classes
   - Implémenter une bonne encapsulation
