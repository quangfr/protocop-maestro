# Introduction

**📌 Objectifs d'une maquette graphique :**

- 🖥️ Discuter et valider des idées d'interface
- 📑 Illustrer et préciser une spécification

**🛠 Usages concrets :**

- 🎨 Brainstorming / Co-construction
- ✍️ Spécifications / Critères d'acceptance
- 👥 Tests utilisateurs (UI/UX)
- 🖼️ Démos réalistes (UI/UX)

**💡 Démarche IA**

1. 👁️ **Conversion** → générer un **wireframe ou une maquette HTML** à partir d'une copie d'écran  
2. 📝 **Protodescription** → demander un **prompt descriptif** de la maquette ou de la copie d'écran
3. ✅ **Validation** → valider la structure graphique **en texte**   
4. 📑 **Décomposition** → procéder graphiquement onglet par onglet, **composant par composant**  
5. 🎲 **Choix** → demander **plusieurs propositions visuels UX/UI** pour choisir
6. 📦 **Exhaustivité** → regrouper un max de demandes dans un message avant régénération  
7. 🏗️ **Assemblage** → une fois satisfait, assembler toute l’interface  
8. 📜 **Clôture** → demander la synthèse du **prompt final**  


**🤖 Lien à la conversation IA**
```
https://chatgpt.com/share/68a2f878-e484-8006-998c-d25c5c591a19
```

# Prompt

<img width="1189" height="878" alt="image" src="https://github.com/user-attachments/assets/11df08f2-6cf8-4e47-a2cd-2b1383cd89ca" />

## 1. Contexte
- MAESTRO est un outil de gestion des demandes et opérations de maintenance moteur (MRO).
- Objectif : donner une vision synthétique et opérationnelle à un instant T sur un grand volume de données.  
- Public cible : planificateurs, responsables de maintenance, managers de centres.
- Besoin : une interface claire, proche de SAP IBP / Fiori, qui permette de visualiser rapidement les écarts de délais, la qualité du respect des délais et l’utilisation des capacités.

---

## 2. Données
- **Périmètre :**
  - 1000 demandes de maintenance  
  - 5000 opérations associées  
  - 50 centres de maintenance dans le monde  
  - 10 modèles de moteurs (ex : CFM56, LEAP, GE90…)  
  - 50 types d’opérations (inspection, réparation, assemblage, tests…)  
  - 5 types de demandes (Overhaul complet, AOG, Quick Inspection, Deep Repair, Scheduled Check)

- **Indicateurs suivis :**
  1. ⏱️ Délai moyen de traitement : prévu vs réel
  2. 📊 Taux de respect des délais (On-Time Delivery) : répartition à l’heure, léger retard, retard critique
  3. ⚙️ Taux d’utilisation des capacités ateliers : % d’occupation par centre et par semaine

---

## 3. Interface
- **Barre de filtres généraux (statiques)** : période, centre, type d’opération, type de demande, moteur.  
- **Disposition en 2 lignes :**
  - Ligne 1 → 2 cartes côte à côte :  
    - **Graphique 1 :** Barres horizontales, une par type de demande. La barre représente le délai réel, et un trait vertical indique le délai prévu. Tooltip : valeurs prévues, réelles et écart (%).  
    - **Graphique 2 :** Barres horizontales empilées, une par type de moteur. Répartition verte (à l’heure), orange (léger retard), rouge (retard critique). Tooltip : nb exact + % par segment.
  - Ligne 2 → 1 carte pleine largeur :  
    - **Graphique 3 :** Heatmap 13 semaines × 20 centres. Chaque cellule = % d’utilisation (vert <85%, orange 85–100%, rouge >100%), avec le % affiché et un tooltip détaillé (opérations planifiées / capacité).

- **Style attendu :**
  - Esthétique SAP Fiori / IBP : cartes blanches, ombrées, titres clairs.  
  - Couleurs vives pour les états (vert, orange, rouge).  
  - Cellules de heatmap collées avec séparateurs hebdo plus marqués.  
  - Simplicité, lisibilité, centrage sur les indicateurs.

---

## 4. Technique
- **Volume simulé** : données générées automatiquement (pseudo-aléatoires crédibles).  
- **Génération** : délais prévus et réels, répartition OTD, utilisation des capacités.  
- **Tooltips** : toujours présents pour afficher la valeur détaillée.  
- **Interactions** : filtres statiques en haut (non connectés), mais possibilité de les rendre dynamiques plus tard.  
- **Responsive** : 2 colonnes sur desktop, 1 colonne sur mobile.  
