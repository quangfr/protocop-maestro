# Contexte
**📌 Objectifs d'un prototype CSV / Excel :**

- ✅ Valider la logique de calculs sur un périmètre réduit
- 📥 Importer de la donnée fictive en masse

**🛠 Usages concrets :**
- ✍️ Spécifications / Critères d'acceptance 
- 👥 Tests utilisateurs (EXIM)
- 🖼️ Démos réalistes (EXIM)

**💡 Démarche IA**
1. 🔍 Prototyper petit avec des hypothèses de simplification
2. ✏️ Demander à GPT de te poser des questions pour préciser ton prompt
3. 🆘 Demander à GPT de te proposer des pistes d'amélioration ou de simplification
4. 🔄 Mettre le plus de demandes possibles dans ton prompt avant régénération
5. 📥 Amender le prompt jusqu'à ce qu'il te convient avec de générer le prototype
6. 📋 Quand tu es satisfait, demande à GPT de te refournir le prompt pour reprendre ailleurs

**🤖 Lien à la conversation IA**
```
https://chatgpt.com/c/68a05788-7224-8320-a6c3-56255e835581
```

# Prompt

### 1) Contexte (Hypothèses + Mise en forme) ✨
*Questions à se poser d’abord* 🤔  
- Quel est l’objectif précis et sur quelle unité/échelle on mesure l’utilisation ?  
- Quelles simplifications on fige pour isoler le calcul ?  
- Comment on présente les résultats pour qu’ils soient lisibles rapidement ?

**Objectif** 🎯  
- Mesurer le **taux de remplissage** des centres (shops) en **%**, comparer **Input (plan)** vs **Output (plan IBP)**, et **voir les écarts**.

**Hypothèses (simples & TDD-friendly)** 🧩  
- **Horizon** = 90 jours, **7/7**.  
- **Unité** = **slot-jour**.  
- **Capacité constante** par **Shop × Catégorie** : `SlotsPerDay` (entier).  
- **100 opérations**. 1 opération = 1 type = 1 catégorie = 1 shop.  
- **Durée d’un type** = **fixe** (5–90 jours).  
- Pas d’**Output_CapacityDays** : la capacité de référence est **celle d’Input** uniquement.

**Mise en forme & lisibilité** 👀  
- Afficher **Util_Input%**, **Util_Output%** et **Écart%** en **pourcentage (2 décimales)**.  
- Voyants **verts** si Écart% = 0, **rouges** sinon.  
- Dates en **YYYY-MM-DD**.  
- Entêtes en gras, volets gelés sous la 1ʳᵉ ligne.  
- Graphiques en **colonnes groupées**, axe Y **0–100%**.

### 2) Modèle (Liste de références + Formules + TDD) 🧠
*Questions à se poser d’abord* 📝  
- Quelles listes de référence minimales et sans IDs ?  
- Quelles sont les règles de génération de la donnée ?
- Comment on valide par des tests unitaires simples ?

```mermaid
classDiagram
    class Parameters {
      START_DATE: date
      HORIZON_DAYS: int = 90
    }

    class Shop {
      name: string
    }

    class Catégorie {
      name: string
    }

    class Shop_Slots {
      Shop: string
      Catégorie: string
      SlotsPerDay: int [2..6]
      CapacityDays = SlotsPerDay * HORIZON_DAYS
    }

    class Input_Operations {
      Id: string
      Type: string
      Catégorie: string
      Shop: string
      Durée_jours: int [5..90]
      StartDate: date
    }

    class Output_Operations {
      Id: string
      Type: string
      Catégorie: string
      Shop: string
      Durée_jours: int
      StartDate: date
      ~1% de différences vs Input (par Id)
    }

    class KPI_Check {
      Shop: string
      Catégorie: string
      Input_CapacityDays: int
      Input_PlanDays: int
      Output_PlanDays: int
      Util_Input_% = Input_PlanDays / Input_CapacityDays
      Util_Output_% = Output_PlanDays / Input_CapacityDays
      Écart_% = Util_Output_% - Util_Input_%
    }

    %% Relations de structure
    Shop "1" -- "0..*" Shop_Slots : offre
    Catégorie "1" -- "0..*" Shop_Slots : éligible

    Shop "1" -- "0..*" Input_Operations : planifié_dans
    Catégorie "1" -- "0..*" Input_Operations : de_cat

    Shop "1" -- "0..*" Output_Operations : issu_d'IBP
    Catégorie "1" -- "0..*" Output_Operations : de_cat

    KPI_Check "0..*" -- "1" Shop : pour
    KPI_Check "0..*" -- "1" Catégorie : pour

    %% Traçabilité Input -> Output
    Input_Operations "1" <-- "0..1" Output_Operations : même Id

    %% Dépendance au paramétrage (pour CapacityDays)
    Shop_Slots ..> Parameters : HORIZON_DAYS

```mermaid

**Listes de références (noms lisibles, pas d’IDs)** 📇  
- **Shops** : Lyon, Toulouse, Nantes, Bordeaux, Marseille.  
- **Catégories** : Inspection, Disassembly, Repair, Assembly, TestRun.  
- **Types** : texte libre au format **“{Catégorie} Type n”** (ex. “Inspection Type 1”).

**KPI (tous en %)** 📈  
- `Input_CapacityDays = SlotsPerDay(Shop×Cat) × 90`  
- `Input_PlanDays = Σ durées des opérations Input (Shop×Cat)`  
- `Output_PlanDays = Σ durées des opérations Output (Shop×Cat)`  
- `Util_Input% = Input_PlanDays / Input_CapacityDays`  
- `Util_Output% = Output_PlanDays / Input_CapacityDays`  
- `Écart% = Util_Output% – Util_Input%`

**Règles de génération (résumé)** 📦  
- **Shop_Slots** : 2–3 catégories par shop, `SlotsPerDay ∈ [2..6]`, **chaque catégorie** présente dans **≥2 shops**.  
- **Input_Operations** : `Id` = OP001…OP100 ; `Type` = “{Catégorie} Type n” (n ∈ [1..4]) ; `Durée (jours)` ∈ [5..90] ; `Start Date` uniforme, avec **fin ≤ START_DATE + 89 j** ; `Shop` éligible à la `Catégorie`.  
- **Output_Operations** : copie d’Input + **~1% d’écarts** (Durée **ou** Start Date, ±1..±3 j) **sans sortir de l’horizon** ; **mise en évidence JAUNE** via XLOOKUP sur `Id`.

**TDD / critères d’acceptation** ✅  
- **CA1** : `SlotsPerDay=4`, horizon 90 → **Input_CapacityDays=360**.  
- **CA2** : 3 opérations de 30 j → **Input_PlanDays=90**, **Util_Input%=25%**.  
- **CA3** : si **Output_PlanDays = Input_PlanDays** → **Écart%=0** (vert).  
- **CA4** : si **Output_PlanDays > Input_PlanDays** → **Écart%>0** (rouge).  
- **CA5** : toute cellule différente entre **Output_Operations** et **Input_Operations** (par `Id`) est **jaune**.

### 3) Interface 🖥️
*Questions à se poser d’abord* 💬  
- Quelles feuilles minimales pour saisir/recoller et lire les KPI ?  
- Comment signaler visuellement les différences cellule par cellule ?
- Comment garantir le calcul “input vs output” au bon grain ?

**Feuilles du classeur** 📒  
- **Parameters** : `START_DATE = 2025-09-01`, `HORIZON_DAYS = 90`.  
- **Shop_Slots** : colonnes `Shop`, `Catégorie`, `SlotsPerDay` (2–3 catégories par shop, `SlotsPerDay ∈ [2..6]`, chaque catégorie couverte par ≥2 shops).  
- **Input_Operations** *(ex-Seed_Operations)* : colonnes `Id`, `Type`, `Catégorie`, `Shop`, `Durée (jours)`, `Start Date` (100 lignes ; cohérence Catégorie/Shop ; dates dans l’horizon).  
- **Output_Operations** *(retour IBP)* : même structure ; générer ~1% d’écarts (durée ±1–3 j **ou** start ±1–3 j) ; **mise en forme conditionnelle JAUNE** si valeur ≠ Input (comparaison par `Id`, colonne à colonne).  
  - Exemple (Durée, cellule E2) : `=$E2<>XLOOKUP($A2, Input_Operations!$A:$A, Input_Operations!$E:$E, "")`  
- **KPI_Check** *(consolidation Shop×Cat)* : colonnes `Shop`, `Catégorie`, `Input_CapacityDays`, `Input_PlanDays`, `Output_PlanDays`, `Util_Input%`, `Util_Output%`, `Écart%` (formater F–H en % ; **vert** si `Écart%=0`, **rouge** sinon).  
- **KPI_Dashboard** : **5 graphiques** (un par shop) en **colonnes groupées** **Util_Input% vs Util_Output%** par **Catégorie** ; axe **0–100%** ; lecture “barres qui se remplissent” 📊.

**Formules Excel (exemples)** 🧮  
*(Supposons dans **KPI_Check** : A=Shop, B=Catégorie, C=Input_CapacityDays, D=Input_PlanDays, E=Output_PlanDays, F=Util_Input%, G=Util_Output%, H=Écart% ; ligne 2 = 1ʳᵉ donnée)*  
- **SlotsParJour (helper)** : `=SUMIFS(Shop_Slots!$C:$C, Shop_Slots!$A:$A, $A2, Shop_Slots!$B:$B, $B2)`  
- **Input_CapacityDays (C2)** : `=SlotsParJour * 90`  
- **Input_PlanDays (D2)** : `=SUMIFS(Input_Operations!$E:$E, Input_Operations!$C:$C, $B2, Input_Operations!$D:$D, $A2)`  
- **Output_PlanDays (E2)** : `=SUMIFS(Output_Operations!$E:$E, Output_Operations!$C:$C, $B2, Output_Operations!$D:$D, $A2)`  
- **Util_Input% (F2)** : `=IFERROR(D2/C2,0)`  
- **Util_Output% (G2)** : `=IFERROR(E2/C2,0)`  
- **Écart% (H2)** : `=G2 - F2`


Rappels ⚠️ : mêmes libellés des deux côtés, même horizon, pas d’Output_CapacityDays; les différences cellule passent en jaune dans Output_Operations

### 4) Technique 🛠️
*Questions à se poser d’abord* 🧪  
- Comment garantir la reproductibilité et la compatibilité Excel Desktop ?
- Comment rendre la connexion avec le flux SAP-IBP immédiate ?
- Quelles contraintes côté données aléatoires ?

**Génération & compatibilité** 🧰  
- Générer un fichier **Excel Desktop** (graphiques colonnes visibles dans Excel, non garanti pour Google Sheets).  
- Bibliothèques recommandées si script : **openpyxl** ou **xlsxwriter** (Python).  
- **Seed aléatoire fixée** pour la reproductibilité.

**Connexion au flux IBP (le plus simple)** 🔄  

**Entrée (INPUT)** : pousser/ajuster les opérations depuis **Input_Operations** vers IBP.

- Crée une feuille IBP_INPUT dans le même fichier avec l’Excel Add-in SAP IBP.
- Choisis la même période (90 j), les mêmes shops et catégories.
- Dans la vue, affiche 2 KFs : CAPACITY_SLOTS (capacité/jour) et LOAD_PLAN_SLOTS (plan/jour).
- Colle tes données (capacité + plan) depuis le classeur, puis Save Data ✅
- Côté classeur, tu continues d’utiliser Input_Operations comme référence pour les KPI (la capacité de référence reste l’input).

**Sortie (OUTPUT)** : coller le retour IBP dans **Output_Operations** (même structure).

- Ajoute l’Excel Add-in SAP IBP dans le même fichier ✅
- Crée une feuille IBP_OUTPUT avec les mêmes shops, catégories et période (90 j) que ta feuille de test
- Dans l’add-in, affiche la KF de plan/réel, puis Refresh
- Copie-colle ce résultat dans Output_Operations (même structure que Input_Operations)
- KPI_Check et KPI_Dashboard se mettent à jour tout seuls : Util_Input% vs Util_Output% et Écart%

**Livrable attendu** 📁  
- Classeur unique **MAESTRO_FillRate_Simplified.xlsx** avec toutes les feuilles, formules, formats et **5 graphiques** prêts.  
