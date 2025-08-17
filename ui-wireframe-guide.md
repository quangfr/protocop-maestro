# Introduction

**📌 Objectifs d'une maquette HTML :**

- 💡 Proposer et valider une vision produit
- 🖥️ Discuter et valider des idées d'interface ou un modèle de données
- 📑 Illustrer et préciser une spécification avec les développeurs comme les usagers

**🛠 Usages concrets :**

- ✍️ Spécifications / Critères d'acceptance
- 👥 Tests utilisateurs (UI/UX)

**💡 Démarche IA**

0. 🏃‍♂️ Privilégier peu de logique métier (données statiques), un style minimaliste avec le minimum de données
1. 🔍 Prototyper petit avec des hypothèses de simplification
2. ✏️ Demander à GPT de te poser des questions pour préciser ton prompt
3. 🆘 Demander à GPT de te proposer des pistes d'amélioration ou de simplification
4. 🔄 Mettre le plus de demandes possibles dans ton prompt avant régénération
5. 📥 Amender le prompt jusqu'à ce qu'il te convient avec de générer le prototype
6. 📋 Quand tu es satisfait, demande à GPT de te refournir le prompt pour reprendre ailleurs
7. 🔶 Demander à GPT de générer le diagramme UML pour vérifier ou documenter

**🤖 Lien à la conversation IA**
```
https://chatgpt.com/share/68a2691d-73cc-8006-b5bc-003d21a127c1
```

# Prompt

## 1. 🎯 Contexte  
- **Outil** : MAESTRO — gestion des demandes & opérations de maintenance moteurs Safran.  
- **Objectif** : Suivi **temps réel** des performances globales de 1000 demandes et 5000 opérations réparties sur 50 centres de maintenance.  
- **KPI clés** :  
  1. ⏱️ On-Time Delivery & Temps moyen par type de demande  
  2. ⚡ Utilisation des capacités ateliers (par site)  
  3. 🛫 Temps moyen par modèle moteur & type de demande  
  4. 🔁 Décomposition des opérations (par type de demande)  
  5. 🎯 Écart prévu vs planifié (avance/retard en jours et %)  

---

## 2. 📊 Données  

### 2.1 Objets principaux  
- **Moteurs (10 modèles)** :  
  - CFM56, LEAP-1A, LEAP-1B, GE90, GP7200, CF6-80E1, PW1100G, Trent 700, Silvercrest, M88.  

- **Types de demandes (5)** :  
  - Overhaul (Grande visite)  
  - Quick Inspection (Inspection rapide)  
  - Deep Repair (Réparation lourde)  
  - Cleaning & Test (Nettoyage + essai)  
  - Component Swap (Remplacement composants)  

- **Opérations (50 types)** regroupées en familles :  
  - Inspection  
  - Disassembly  
  - Repair  
  - Cleaning  
  - Assembly  
  - Test Run  

- **Sites (50 centres de maintenance)** :  
  - Lyon, Toulouse, Bordeaux, Hambourg, Casablanca, Dubaï… (etc., 50 noms de sites réels ou plausibles).  

### 2.2 Données quantitatives simulées (exemples réalistes)  
- **1000 demandes** actives.  
- **5000 opérations** en cours.  
- Capacité : 50 sites avec 15 visibles par page (pagination).  
- Δ prévu vs planifié exprimé en **jours** et **%** (de -50% avance à +50% retard).  

---

## 3. 🖥️ Interface  

### 3.1 Organisation  
- Style SAP IBP / Fiori :  
  - **Cartes KPI** avec icônes et jauges.  
  - **Filtres & sélecteurs** (par moteur, type de demande).  
  - **Graphiques interactifs** (barres, jauges, empilés).  
  - Pagination pour les centres de maintenance (15/page).  

### 3.2 Composants graphiques  
1. **OTD & Temps moyen (bar chart + jauge centrale)**  
   - Axe : Types de demandes.  
   - Données : Temps prévu vs réel, Δ en jours et %.  
   - Jauge centrale Δ global (-50% avance ↔ +50% retard).  

2. **Utilisation des capacités (barres triées)**  
   - Sites triés du moins capacitaire au plus capacitaire.  
   - Nb d’opérations en cours affiché.  
   - Pagination 15 sites par page.  

3. **Temps moyen par moteur (gauge chart)**  
   - Liste des 10 moteurs.  
   - Sélecteur : type de demande.  
   - Jauges comparant Prévu vs Réel.  

4. **Décomposition par type de demande (stacked bar)**  
   - Axe : Types de demandes.  
   - Barres empilées : familles d’opérations (Inspection, Disassembly, Repair, etc.).  
   - Couleurs stables par famille.  

---

## 4. ⚙️ Technique  

- **Technologies front** : HTML5, CSS3, Chart.js (ou Recharts/Highcharts selon stack), pagination en Vanilla JS.  
- **Statique** : données codées en dur pour prototype.  
- **Navigation** :  
  - Onglet “KPI Maintenance” dans l’interface Fiori-like.  
  - Filtres disponibles (type de demande, modèle moteur, site).  
- **Évolutions possibles** :  
  - Thème sombre/light toggle.  
  - Export PNG/PDF des KPI.  
  - Seuils dynamiques de couleur (vert/orange/rouge).  


