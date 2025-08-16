## Contexte
Illustrer la capacité à vérifier la justesse des données calculées venant d'un outil externe comme SAP-IBP. Dans notre cas il s'agit du taux d'occupation des centres de maintenance des moteurs d'avion. Le choix d'Excel est lié à l'utilisation d'Excel Add-in. On utilise ChatGPT pour générer un prototype de la feuille de calcul de vérification.

## IA🤖 
**Astuces💡**
- Demander à GPT de te poser des questions pour préciser ta demande
- Demander à GPT de te proposer des pistes de simplification
- Demander à GPT de générer une proposition au préalable
- Amende la proposition jusqu'a ce qu'elle te convient avec de générer le prototype
- Quand tu es satisfait du prototype, demande à GPT de te fournir le prompt pour reprendre ailleurs le prototype

**Lien à la conversation 🔗**
```
https://chatgpt.com/c/68a05788-7224-8320-a6c3-56255e835581
```

## Prompt 
```
RÔLE
Tu es un assistant outillé (Python/Excel) qui doit produire un fichier Excel unique prêt à l’emploi.

LIVRABLE
- Nom du fichier : MAESTRO_FillRate_Simplified.xlsx
- Un seul classeur, feuilles et graphiques inclus.
- Retourne un lien de téléchargement direct une fois généré.

HYPOTHÈSES GLOBALES (figées)
- Horizon = 90 jours, calendrier 7j/7.
- Unité = slot-jour.
- Capacité constante par couple (Shop × Catégorie) : SlotsPerDay (entier).
- 100 opérations. 1 opération = 1 type = 1 catégorie = 1 shop.
- La durée d’un type est fixe (5–90 jours).
- Pas d’Output_CapacityDays : la capacité de référence est celle d’Input.

LISTES RÉFÉRENCES (noms lisibles, pas d’IDs)
- Shops : Lyon, Toulouse, Nantes, Bordeaux, Marseille.
- Catégories : Inspection, Disassembly, Repair, Assembly, TestRun.
- Types (texte libre) : “{Catégorie} Type n”.

STRUCTURE DU CLASSEUR (feuilles)

1) Parameters
- Colonnes : Parameter, Value, Notes
- Valeurs minimales :
  - START_DATE = 2025-09-01 (format ISO YYYY-MM-DD)
  - HORIZON_DAYS = 90
  - NOTES indiquant l’usage (facultatif)
- Mise en forme : entêtes en gras, gel des volets sous la ligne 1.

2) Shop_Slots
- Colonnes : Shop, Catégorie, SlotsPerDay
- Règles de génération :
  - Chaque shop propose 2 à 3 catégories (tirage aléatoire).
  - SlotsPerDay ∈ [2..6].
  - Couverture : chaque catégorie doit être présente dans au moins 2 shops.
- Mise en forme : entêtes en gras, gel des volets sous la ligne 1.

3) Input_Operations (ex-Seed_Operations)
- Colonnes (dans cet ordre) :
  - Id (ex. OP001…OP100)
  - Type (ex. “Inspection Type 1”)
  - Catégorie (parmi la liste)
  - Shop (parmi la liste)
  - Durée (jours) ∈ [5..90]
  - Start Date (date ISO dans la fenêtre des 90 jours ; la fin = start + durée - 1 doit rester ≤ fin d’horizon)
- Règles de génération :
  - 100 lignes.
  - Le Type est cohérent avec la Catégorie (“{Catégorie} Type n”).
  - Le Shop est éligible à la Catégorie (présent dans Shop_Slots).
  - Répartition aléatoire mais raisonnable (simple, pas d’optimisation).
- Mise en forme : entêtes en gras, gel des volets ; dates au format YYYY-MM-DD.

4) Output_Operations (nouveau)
- Même structure/colonnes que Input_Operations.
- Générer automatiquement ~1% d’écarts aléatoires par rapport à Input_Operations (≈ 1 opération sur 100) :
  - Modifier soit Durée (±1 à ±3 jours) soit Start Date (±1 à ±3 jours), en restant dans l’horizon.
- Mise en évidence des différences cellule par cellule, basées sur la clé Id :
  - Règle de mise en forme conditionnelle JAUNE si la valeur Output ≠ valeur Input pour la même colonne (Type, Catégorie, Shop, Durée, Start Date).
  - Exemple (idée) : pour Output_Operations!E2 (Durée), comparer à XLOOKUP($A2, Input_Operations!$A:$A, Input_Operations!$E:$E, "").

5) KPI_Check (consolidation par Shop × Catégorie)
- Colonnes (dans cet ordre) :
  - Shop
  - Catégorie
  - Input_CapacityDays
  - Input_PlanDays
  - Output_PlanDays
  - Util_Input%    (= Input_PlanDays / Input_CapacityDays)
  - Util_Output%   (= Output_PlanDays / Input_CapacityDays)
  - Écart%         (= Util_Output% - Util_Input%)
- Détails de calcul :
  - Input_CapacityDays = (SlotsPerDay du couple) × HORIZON_DAYS.
    - SlotsPerDay via SUMIFS sur Shop_Slots.
  - Input_PlanDays = somme des Durée (jours) d’Input_Operations filtrées par Shop & Catégorie (SUMIFS).
  - Output_PlanDays = idem mais sur Output_Operations.
  - Ratios formatés en pourcentage (2 décimales).
- Mise en forme conditionnelle :
  - Écart% = 0 → vert ; ≠ 0 → rouge.
  - Optionnel : surlignage si Util_*% > 100%.

6) KPI_Dashboard
- 5 graphiques “barres qui se remplissent”, un par shop.
- Pour chaque shop :
  - Table d’aide : Catégorie, Util_Input%, Util_Output% (tirées de KPI_Check).
  - Graphique en colonnes groupées (2 séries) avec les Catégories en abscisse.
  - Axe Y borné à 0–100%.
  - Titre : “{Shop} — Utilisation (%) par catégorie”.
- Important : charts compatibles Excel Desktop (ne pas cibler Google Sheets).

FORMULES EXCEL (références colonnes)
Supposons KPI_Check : A=Shop, B=Catégorie, C=Input_CapacityDays, D=Input_PlanDays, E=Output_PlanDays, F=Util_Input%, G=Util_Output%, H=Écart%, et ligne 2 = 1ʳᵉ ligne de données.

- SlotsPerDay (helper interne, ou inline)
  =SUMIFS(Shop_Slots!$C:$C, Shop_Slots!$A:$A, $A2, Shop_Slots!$B:$B, $B2)

- Input_CapacityDays (C2)
  =SlotsPerDay * 90
  (ou inline avec la formule SUMIFS × 90)

- Input_PlanDays (D2)
  =SUMIFS(Input_Operations!$E:$E, Input_Operations!$C:$C, $B2, Input_Operations!$D:$D, $A2)

- Output_PlanDays (E2)
  =SUMIFS(Output_Operations!$E:$E, Output_Operations!$C:$C, $B2, Output_Operations!$D:$D, $A2)

- Util_Input% (F2)
  =IFERROR(D2/C2,0)

- Util_Output% (G2)
  =IFERROR(E2/C2,0)

- Écart% (H2)
  =G2-F2

MISE EN FORME / UX
- Entêtes en gras, gel des volets (A2).
- Auto-fit colonnes.
- Dates en ISO (YYYY-MM-DD).
- Pour Output_Operations : règles de mise en forme conditionnelle JAUNES pour chaque colonne (Type, Catégorie, Shop, Durée, Start Date) vs Input_Operations, basées sur la clé Id.
- Pour KPI_Check : pourcentages sur F, G, H ; voyants vert/rouge sur H.

GÉNÉRATION DES DONNÉES (détails)
- Seed aléatoire fixé pour reproductibilité.
- Shop_Slots : 2–3 catégories par shop, SlotsPerDay ∈ [2..6], couverture min. 2 shops par catégorie.
- Input_Operations :
  - Id : OP001 à OP100.
  - Type : “{Catégorie} Type n” (n ∈ [1..4]).
  - Durée : uniformément entre 5 et 90 jours.
  - Start Date : tirée uniformément, en veillant à ne pas dépasser la fin d’horizon (end = start + durée - 1 ≤ START_DATE + 89 j).
  - Shop éligible à la Catégorie (via Shop_Slots).
- Output_Operations :
  - Copie d’Input_Operations + ~1% d’écarts (Durée ±1..±3 jours OU Start Date ±1..±3 jours), en restant dans l’horizon.
  - Mettre en évidence les écarts en JAUNE.

ACCEPTANCE / TDD (cases à vérifier automatiquement ou en note)
- CA1 : Si SlotsPerDay=4 et HORIZON=90 → Input_CapacityDays=360.
- CA2 : 3 opérations de 30 j → Input_PlanDays=90 → Util_Input%=25%.
- CA3 : Output_PlanDays = Input_PlanDays → Écart%=0 (voyant vert).
- CA4 : Output_PlanDays > Input_PlanDays → Écart%>0 (voyant rouge).
- CA5 : Les cellules réellement modifiées dans Output_Operations sont JAUNES (comparaison par Id).

CONTRAINTE OUTIL
- Génère réellement le fichier Excel “MAESTRO_FillRate_Simplified.xlsx” avec toutes les feuilles, formules et graphiques.
- Si tu utilises Python, privilégie openpyxl/xlsxwriter ; s’assurer que les graphiques sont visibles dans Excel Desktop.
- Fournis un lien de téléchargement direct en sortie.
```
