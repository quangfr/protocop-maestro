# 1️⃣ Identité

**Nom affiché** : MAESTRO  
**Style** : Thème clair, interface SAP IBP-like (cartes, onglets, entêtes figées, couleurs codes capacité, boutons arrondis)  
**Objectif** : Gérer en local les demandes et opérations de maintenance sur moteurs d’avion : création, édition, suivi des capacités et visualisation d’indicateurs clés, le tout dans un fichier HTML unique sans dépendances externes.

---

# 2️⃣ Modèle de données

## Objets métiers

### Demande (Request)
- **Propriétés** : ID, Client, Modèle moteur, Localisation, Atelier (Shop), Type de demande, Urgence, Statut, Date demande, Date de fin estimée, Notes
- **Validations** : champs obligatoires (Client, Moteur, Localisation, Shop, Type, Urgence, Statut), ID auto, Date fin calculée automatiquement
- **Contraintes** : chaque type de demande impose exactement 4 opérations requises ; relations vers un Shop et un Type de demande existants

### Opération (Operation)
- **Propriétés** : ID, Request ID, Shop, Type d’opération, Statut, Date début, Durée (1–5 semaines), Date fin
- **Validations** : champs obligatoires (Request ID, Shop, Type op, Statut), ID auto, Date fin calculée depuis début+durée
- **Contraintes** : Type op autorisé par Shop, lié à une demande existante

### Listes maîtres (`mdLists`)
- Urgency, Status, Shop, Location, Operation Type, Engine Model, Customer, Request Type

### Mapping Request Type → Ops requises
- **Règle** : chaque Request Type doit avoir exactement 4 types d’opération distincts

### Durées par type d’opération (`opDur`)
- **Valeur** : entre 1 et 5 semaines

### Capacités atelier (`shopCapabilities`)
- **Propriétés** : Ops autorisées, capacité par défaut, localisation, exceptions de capacité par période

## Relations
- 1 Demande → N Opérations
- 1 Shop → N Operation Types autorisés
- 1 Request Type → exactement 4 Operation Types
- ShopCapabilities liés à Shop et Location

## Données d’exemple (seed)
- Urgency : 4 valeurs
- Status : 4 valeurs
- Shops : 10 (S01 à S10)
- Locations : 5
- Operation Types : 12
- Engine Models : 5
- Customers : 5
- Request Types : 3 (4 ops chacune)
- Durées : 1 à 5 semaines
- Transactions : 10 demandes, 30 opérations aléatoires

---

# 3️⃣ Navigation et écrans

## Onglets / écrans

### Nouvelle demande
- Formulaire création demande (champs obligatoires en jaune)
- Helper panel : liste des 4 ops requises par type, Capacity Lookup (date+shop+op type → capacity/used/free)
- **Actions** : créer demande, recalculer date fin estimée

### Éditer opérations
- Formulaire création/édition opération
- Helper panel : liste ops autorisées par shop, capacité par défaut et exceptions
- **Actions** : sauvegarder ou supprimer opération

### Demandes
- Tableau filtrable par Request ID
- Liens sur Request ID / nombre d’ops → ouvrent tableau opérations filtré

### Opérations
- Tableau filtrable par Operation ID ou Request ID
- Liens sur Request ID → ouvrent tableau demandes filtré
- Liens sur Operation ID → ouvrent formulaire édition opération pré-rempli

### Listes maîtres
- Colonnes éditables pour chaque liste
- Table mapping (Request Type → 4 ops) avec contrôle unicité
- Table durées par type op
- Bouton **Apply MD Changes**
- JSON Viewer (read-only, collapsible, copy, download)

### KPI Dashboard
- On-time % urgent (calcul sur demandes Urgent livrées à temps)
- Heatmap capacité hebdo (Location × Shop, 8 semaines, codes couleur vert/jaune/rouge)

## Navigation
- Barre d’onglets persistante en haut
- Clics sur IDs dans les tableaux pour naviguer avec filtre appliqué
- Passage auto à l’écran concerné après clic

---

# 4️⃣ Techniques

## Fonctionnalités avancées
- Fichier HTML autonome (offline, Vanilla JS, pas de dépendance externe)
- Données seed aléatoires au démarrage
- Dropdowns dynamiques (filtrage ops par shop)
- Calculs auto (dates fin, capacity lookup, KPI)
- Recherche texte avec rafraîchissement instantané
- Import / Export JSON complet (tous objets métiers)
- JSON Viewer avec sections repliables, copy/download
- Tableaux avec entêtes sticky et survol ligne
- KPI & Heatmap calculés à la volée
- Responsive simplifié (stack en mobile)

## Contraintes techniques
- Pas de stockage persistant natif (hors export/import JSON)
- Pas de backend, toute logique côté client
- Capacités calculées à la semaine ISO entière
- Mapping et durées doivent respecter les validations sinon avertissement
