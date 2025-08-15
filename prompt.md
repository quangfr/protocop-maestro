# Créer un prototype HTML

## 1️⃣ Identité
**Nom** : MAESTRO  
**Style** : clair, type SAP IBP  
**Objectif** : suivre les demandes et opérations de maintenance, avec vue sur les capacités et indicateurs

## 2️⃣ Les données de base
**Urgency** : Low, Medium, High, Urgent  
**Status** : Open, Ready, In Progress, Done  
**Shops** : Shop 1 à Shop 10  
**Locations** : CDG, LYS, NCE, TLS, ORY  
**Operation Types** : Inspection, Disassembly, Cleaning, NDT, Repair, Assembly, Balancing, Test Bench, Painting, Documentation, Shipping, QA  
**Mapping Request → Operations** :
- Light Check → Inspection, Cleaning, Documentation, QA
- Standard Overhaul → Disassembly, NDT, Repair, Assembly
- Major Overhaul → Disassembly, Repair, Assembly, Test Bench

**Durées par défaut** :
- Inspection = 1 semaine
- Disassembly = 3 semaines
- Cleaning = 1 semaine
- NDT = 2 semaines
- Repair = 4 semaines
- Assembly = 3 semaines
- Balancing = 1 semaine
- Test Bench = 2 semaines
- Painting = 1 semaine
- Documentation = 1 semaine
- Shipping = 1 semaine
- QA = 1 semaine

**Capacités par défaut** :
- Chaque shop a 4–10 opérations autorisées
- Capacité : 1–3 opérations/semaine
- Exceptions possibles (dates spécifiques + capacité modifiée)

**Données initiales** : 10 demandes, 30 opérations

## 3️⃣ Par onglet
### 📝 FORM_New_Request
- Formulaire pour créer une demande
- Champs obligatoires en jaune
- Calcul automatique de la date estimée de fin (durées totales + marge)
- Panneau d’aide : liste des opérations requises selon le type choisi + test de capacité

### 🛠 FORM_Edit_Operations
- Formulaire pour éditer une opération
- Panneau d’aide : opérations possibles dans le shop + capacité par défaut

### 📋 TR_Maint_Requests
- Tableau des demandes avec recherche et entêtes fixes
- Clic sur Request ID ou Ops # → ouvre OP_Maint_Operations filtré
- Boutons Import / Export JSON

### 📋 OP_Maint_Operations
- Tableau des opérations avec recherche et entêtes fixes
- Filtrage par Request ID
- Clic sur Operation ID → ouvre FORM_Edit_Operations pré-rempli
- Clic sur Request ID → retourne sur TR_Maint_Requests filtré

### 📚 MD_Lists
- Édition des listes de valeurs, mapping, durées et capacités
- Boutons : “Appliquer” (rafraîchit) / “Réinitialiser” (revient aux données par défaut)
- JSON Viewer : voir toutes les données, copier, télécharger

### 📊 KPI_Dashboard
- KPI : % de demandes urgentes terminées à temps
- Heatmap : taux d’utilisation des capacités par lieu × atelier (vert < 50 %, jaune 50–99 %, rouge ≥ 100 %)

## 4️⃣ Règles communes
**Navigation** :
- Clic sur Request ID ou Ops # → onglet opérations filtré
- Clic sur Operation ID → formulaire d’édition pré-rempli
- Clic sur Request ID dans opérations → retour aux demandes filtrées

**Contraintes techniques** :
- Un seul fichier HTML
- Vanilla JS uniquement
- Pas de dépendances externes
- 100 % offline
