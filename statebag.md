# Les Statebags FiveM

## Qu'est-ce qu'un Statebag ?
Un **Statebag** est un système de stockage d'état distribué dans FiveM qui permet de partager et synchroniser des données entre le client et le serveur. Il fonctionne comme un système de clé-valeur distribué avec des capacités de réplication automatique.

## Architecture technique

### Structure des Statebags
1. **Entités porteurs**
   - Entités du jeu (véhicules, peds, objets)
   - Entités globales
   - Entités joueurs
   - Entités personnalisées

2. **Types de données**
   - Valeurs primitives (string, number, boolean)
   - Tables
   - Vecteurs
   - Objets complexes

3. **Portée des données**
   - Global
   - Entité
   - Joueur
   - Ressource

## Fonctionnement technique

### Système de réplication
1. **Mécanisme de synchronisation**
   - Réplication automatique
   - Gestion des conflits
   - Optimisation réseau
   - Compression des données

2. **Gestion des états**
   - États persistants
   - États temporaires
   - États partagés
   - États locaux

3. **Système de validation**
   - Vérification des types
   - Contraintes de données
   - Permissions
   - Intégrité

### Cycle de vie

1. **Création**
   - Initialisation
   - Configuration
   - Validation
   - Réplication

2. **Modification**
   - Mise à jour
   - Synchronisation
   - Validation
   - Notification

3. **Suppression**
   - Nettoyage
   - Réplication
   - Libération
   - Finalisation

## Intégration technique

### Accès aux Statebags

1. **Côté serveur**
   - Création
   - Modification
   - Suppression
   - Validation

2. **Côté client**
   - Lecture
   - Écoute des changements
   - Réplication
   - Cache local

3. **Système de permissions**
   - Contrôle d'accès
   - Validation
   - Sécurité
   - Audit

### Mécanismes de synchronisation

1. **Réplication**
   - Diffusion
   - Ciblage
   - Optimisation
   - Gestion de la latence

2. **Gestion des conflits**
   - Résolution
   - Priorités
   - Cohérence
   - Récupération

3. **Performance**
   - Mise en cache
   - Compression
   - Batching
   - Optimisation réseau

## Fonctionnalités avancées

### Gestion de la mémoire
- Allocation optimisée
- Nettoyage automatique
- Gestion des fuites
- Compression des données

### Performance
- Optimisation de la réplication
- Gestion de la bande passante
- Réduction de la latence
- Mise en cache intelligente

### Sécurité
- Validation des données
- Protection contre les injections
- Gestion des permissions
- Audit des modifications

## Cas d'utilisation techniques

### Synchronisation d'état
1. **États de jeu**
   - Propriétés d'entités
   - États des joueurs
   - Configuration
   - Données partagées

2. **Systèmes de cache**
   - Mise en cache distribué
   - Invalidation
   - Cohérence
   - Performance

3. **Systèmes de configuration**
   - Paramètres globaux
   - Configuration par ressource
   - Paramètres dynamiques
   - Persistance

## Exemples d'utilisation pratiques

### Cas d'utilisation recommandés

1. **Propriétés de véhicules**
```lua
-- Côté serveur
local vehicle = CreateVehicle(...)
Entity(vehicle).state.fuel = 100
Entity(vehicle).state.damage = 0
Entity(vehicle).state.owner = playerId

-- Côté client
local vehicle = GetVehiclePedIsIn(PlayerPedId(), false)
local fuel = Entity(vehicle).state.fuel
```

2. **États de joueurs**
```lua
-- Côté serveur
local player = GetPlayerPed(source)
Entity(player).state.isInjured = true
Entity(player).state.currentJob = "police"

-- Côté client
local player = PlayerPedId()
if Entity(player).state.isInjured then
    -- Gérer l'état de blessure
end
```

3. **Configuration partagée**
```lua
-- Côté serveur
GlobalState.serverConfig = {
    maxPlayers = 32,
    weather = "CLEAR",
    timeScale = 1.0
}

-- Côté client
local config = GlobalState.serverConfig
```

### Cas d'utilisation à éviter

1. **Données volumineuses**
```lua
-- Mauvais : Données trop volumineuses
Entity(vehicle).state.inventory = {
    -- Liste de 1000 items avec beaucoup de détails
}

-- Bon : Données optimisées
Entity(vehicle).state.inventoryCount = 1000
Entity(vehicle).state.inventoryHash = "hash_du_contenu"
```

2. **Données fréquemment mises à jour**
```lua
-- Mauvais : Mise à jour trop fréquente
Citizen.CreateThread(function()
    while true do
        Wait(0)
        Entity(player).state.position = GetEntityCoords(player)
    end
end)

-- Bon : Utiliser les natives de synchronisation
```

3. **Données sensibles**
```lua
-- Mauvais : Données sensibles dans le statebag
Entity(player).state.password = "secret"
Entity(player).state.adminKey = "key123"

-- Bon : Stocker les données sensibles côté serveur uniquement
```

## Bonnes pratiques

### 1. Structure des données
```lua
-- Bon : Structure claire et optimisée
Entity(vehicle).state.fuel = 100
Entity(vehicle).state.damage = 0
Entity(vehicle).state.owner = playerId

-- Mauvais : Structure confuse
Entity(vehicle).state.vehicleData = {
    fuel = 100,
    damage = 0,
    owner = playerId
}
```

### 2. Gestion des permissions
```lua
-- Bon : Vérification des permissions
if IsPlayerAceAllowed(source, 'command.setstate') then
    Entity(vehicle).state.owner = playerId
end

-- Mauvais : Pas de vérification
Entity(vehicle).state.owner = playerId
```

### 3. Optimisation des mises à jour
```lua
-- Bon : Mise à jour optimisée
local lastUpdate = 0
Citizen.CreateThread(function()
    while true do
        Wait(1000)
        if GetGameTimer() - lastUpdate > 5000 then
            Entity(vehicle).state.fuel = currentFuel
            lastUpdate = GetGameTimer()
        end
    end
end)

-- Mauvais : Mise à jour constante
Citizen.CreateThread(function()
    while true do
        Wait(0)
        Entity(vehicle).state.fuel = currentFuel
    end
end)
```

## Mauvaises pratiques à éviter

### 1. Surcharge des statebags
```lua
-- Mauvais : Trop de données
Entity(player).state.playerData = {
    inventory = {...},
    stats = {...},
    skills = {...},
    quests = {...},
    -- etc...
}

-- Bon : Données essentielles uniquement
Entity(player).state.health = 100
Entity(player).state.armor = 50
Entity(player).state.currentJob = "police"
```

### 2. Utilisation incorrecte des types
```lua
-- Mauvais : Types inappropriés
Entity(vehicle).state.position = "x:100,y:200,z:300"
Entity(vehicle).state.isActive = "true"

-- Bon : Types appropriés
Entity(vehicle).state.position = vector3(100, 200, 300)
Entity(vehicle).state.isActive = true
```

### 3. Manque de validation
```lua
-- Mauvais : Pas de validation
Entity(player).state.health = newHealth

-- Bon : Validation des données
if type(newHealth) == "number" and newHealth >= 0 and newHealth <= 100 then
    Entity(player).state.health = newHealth
end
```

## Cas d'utilisation spécifiques

### 1. Système de véhicules
```lua
-- État du véhicule
Entity(vehicle).state.fuel = 100
Entity(vehicle).state.damage = 0
Entity(vehicle).state.owner = playerId
Entity(vehicle).state.locked = true
Entity(vehicle).state.engine = false
```

### 2. Système de jobs
```lua
-- État du joueur
Entity(player).state.job = "police"
Entity(player).state.jobGrade = 3
Entity(player).state.onDuty = true
Entity(player).state.lastPaycheck = GetGameTimer()
```

### 3. Système de météo
```lua
-- État global
GlobalState.weather = "RAIN"
GlobalState.weatherIntensity = 0.8
GlobalState.timeScale = 1.0
GlobalState.freezeTime = true
```

## À retenir

1. **Utilisation appropriée**
   - Données de synchronisation
   - États partagés
   - Configuration
   - Propriétés d'entités

2. **Optimisation**
   - Structure des données
   - Fréquence des mises à jour
   - Taille des données
   - Types appropriés

3. **Sécurité**
   - Validation des données
   - Gestion des permissions
   - Protection des données sensibles
   - Audit des modifications

4. **Performance**
   - Mise en cache
   - Réduction des mises à jour
   - Optimisation réseau
   - Gestion de la mémoire

## Fonctionnement réseau des Statebags

### Architecture réseau
1. **Protocole de réplication**
   - Utilisation du protocole de réplication de FiveM
   - Compression des données en temps réel
   - Gestion de la bande passante
   - Optimisation des paquets

2. **Mécanisme de synchronisation**
   - Réplication différentielle
   - Mise à jour incrémentale
   - Gestion des conflits
   - Cohérence des données

3. **Gestion de la latence**
   - Prédiction de mouvement
   - Interpolation des données
   - Gestion des timeouts
   - Récupération des états

### Flux de données
1. **Côté serveur**
   - Validation des données
   - Compression
   - Diffusion aux clients
   - Gestion des priorités

2. **Côté client**
   - Réception des mises à jour
   - Décompression
   - Application des changements
   - Mise en cache local

3. **Optimisation réseau**
   - Batching des mises à jour
   - Compression adaptative
   - Filtrage des données
   - Gestion de la bande passante

## Comparaison des approches

### Avantages et Inconvénients

| Aspect | Avantages | Inconvénients |
|--------|-----------|---------------|
| **Performance** | • Réplication automatique<br>• Optimisation réseau intégrée<br>• Mise en cache intelligente<br>• Compression des données | • Overhead réseau pour petites données<br>• Latence de synchronisation<br>• Coût en bande passante<br>• Impact sur les performances client |
| **Sécurité** | • Validation côté serveur<br>• Protection contre les injections<br>• Gestion des permissions<br>• Audit des modifications | • Exposition des données aux clients<br>• Risque de manipulation<br>• Complexité de sécurisation<br>• Coût en ressources serveur |
| **Facilité d'utilisation** | • API simple<br>• Synchronisation automatique<br>• Intégration native<br>• Documentation complète | • Courbe d'apprentissage<br>• Complexité de débogage<br>• Limitations techniques<br>• Contraintes d'utilisation |
| **Maintenance** | • Gestion automatique<br>• Nettoyage intégré<br>• Monitoring facile<br>• Débogage simplifié | • Complexité de maintenance<br>• Coût en ressources<br>• Dépendance au système<br>• Limitations de personnalisation |
| **Scalabilité** | • Distribution automatique<br>• Gestion de la charge<br>• Optimisation automatique<br>• Adaptation dynamique | • Limitations de taille<br>• Coût en ressources<br>• Contraintes de performance<br>• Complexité de scaling |

### Comparaison avec d'autres méthodes

| Méthode | Statebags | Events | Variables locales |
|---------|-----------|---------|------------------|
| **Synchronisation** | Automatique | Manuel | Aucune |
| **Performance** | Moyenne | Élevée | Très élevée |
| **Sécurité** | Élevée | Moyenne | Faible |
| **Facilité** | Élevée | Moyenne | Très élevée |
| **Maintenance** | Moyenne | Élevée | Faible |
| **Scalabilité** | Élevée | Moyenne | Faible |

## Cas d'utilisation recommandés

### Scénarios optimaux
1. **Données partagées**
   - États de jeu
   - Configuration
   - Propriétés d'entités
   - Données synchronisées

2. **Systèmes distribués**
   - Multi-joueurs
   - Réplication d'état
   - Synchronisation
   - Partage de données

3. **Configuration dynamique**
   - Paramètres serveur
   - Options de jeu
   - États globaux
   - Variables partagées

### Scénarios à éviter
1. **Données sensibles**
   - Informations critiques
   - Données privées
   - Clés de sécurité
   - Données confidentielles

2. **Données volumineuses**
   - Grands ensembles
   - Données binaires
   - Fichiers
   - Ressources lourdes

3. **Données fréquentes**
   - Positions en temps réel
   - États changeants
   - Données de rendu
   - Informations temporaires

## À retenir

1. **Fonctionnement réseau**
   - Protocole de réplication
   - Optimisation des données
   - Gestion de la latence
   - Synchronisation

2. **Choix d'utilisation**
   - Évaluer les besoins
   - Considérer les limitations
   - Optimiser l'utilisation
   - Sécuriser les données

3. **Performance**
   - Gérer la bande passante
   - Optimiser les mises à jour
   - Utiliser le cache
   - Surveiller les ressources

4. **Maintenance**
   - Documenter l'utilisation
   - Monitorer les performances
   - Gérer les erreurs
   - Maintenir la sécurité
