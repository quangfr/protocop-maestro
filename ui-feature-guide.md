# Introduction

**📌 Objectifs d'un prototype de calcul HTML :**

- 💡 Transformer une idée en prototype concret
- 🗂️ Identifier les nouveau besoins métiers

**🛠 Usages concrets :**

- 🎨 Brainstorming / Co-construction
- ✍️ Spécifications / Critères d'acceptance
- 👥 Tests utilisateurs (UI/UX)
- 🖼️ Démos réalistes (UI/UX)

**💡 Démarche IA**

1. 🚲 Minimalisme → privilégier peu de logique métier, données statiques  
2. 🧩 Simplification → prototyper petit avec hypothèses légères  
3. 🎤 Clarification → demander à GPT de poser des questions  
4. 💡 Amélioration → demander à GPT des pistes d’optimisation  
5. 📦 Exhaustivité → regrouper un max de demandes dans un message avant régénération  
6. 📐 Vérification → générer un diagramme UML pour documenter  
7. 📝 Itération → amender le prompt jusqu’à satisfaction  
8. 📌 Transmission → demander à GPT de refournir le prompt final

**🤖 Lien à la conversation IA**
```
https://chatgpt.com/share/68a33867-818c-8006-ac8c-efbd47c3d3ec
```

# Prompt

<img width="1263" height="693" alt="image" src="https://github.com/user-attachments/assets/c9063399-6182-4e13-bbf5-190a5174e29e" />
<img width="1918" height="924" alt="image" src="https://github.com/user-attachments/assets/29f5d26d-d8f7-49c6-b3df-3367ec4b1e6e" />
<img width="1907" height="634" alt="image" src="https://github.com/user-attachments/assets/d5418cfc-539d-4a7a-a19b-80e53b9fdb76" />
<img width="1919" height="703" alt="image" src="https://github.com/user-attachments/assets/c6f3b8c2-4988-487a-9d79-27c90ed90110" />


## 1) CONTEXTE
- **But** : recommander un **centre de maintenance** pour une demande (client, moteur, type, priorité) avec 2 stratégies d’ETA :
  - ⚡ **FastETA** = acheminement aller + durée d’opération + attente (si pleine capacité) + acheminement retour
  - 🍀 **ResponsibleETA** = FastETA + **pénalité** de saturation (projection à 4 semaines)
- **Contraintes métier**
  - Urgent ≤ Standard (jamais plus long à paramètres identiques)
  - ETA **stable** si les entrées ne changent pas : aucune randomness lors du calcul (seulement à la génération initiale des master data)
  - 20 centres, chacun **2–3 moteurs** supportés (sur 5) et **2–3 types** (sur 4)
- **Simplifications**
  - **Acheminement fixe** pour chaque *client × centre* (généré une fois, aller=retour)
  - **Durée d’opération fixe** par *type × moteur* (indépendant du centre, générée une fois)
  - Pas d’aléas en exécution (seulement à la génération)

---

## 2) DONNÉES (modèle + randomization)
### 2.1 Entités
- **Centre**
  - `name`, `engines:Set`, `types:Set`
  - `capBase` (slots/jour), `trend` (−10%..+10%)
  - `capByDay[0..H-1]` (capacité quotidienne), `loadByDay[0..H-1]`
  - `loadByDayByType[type][0..H-1]`
- **Client** : `id / name`
- **Demande**
  - `id`, `client`, `centre`, `engine`, `type`, `priority ∈ {Urgent, Standard}`
  - `startDate`, `durationDays`
- **Master tables fixes**
  - `shipDays[client][centre] → jours` (fixe)
  - `opDurTE[type][engine] → jours` (fixe)

### 2.2 Paramètres (éditables)
- `seed=42`, `weeks=12` (horizon 84j), `clients=30`, `existing=250`, `%Urgent=25`
- Capacité : `capMin=4`..`capMax=10`, `trendMin=-10%`..`trendMax=+10%`
- **Durées triangulaires par type** (pour générer `opDurTE`) :
  - `TestOnly: 2,3,4` · `QuickInspection: 3,4,6` · `RepairOnly: 12,18,28` · `Overhaul: 35,45,65`
- **Acheminement** (pour générer `shipDays`) : probas pour {0..4} = {25,35,20,15,5}%
- Règles : `reserveUrgPct=25` (concept), `penaltyFactor=1.0`

### 2.3 Génération (seedée, déterministe)
1. **Centres** (20 villes FR) : moteurs & types via échantillonnage (2–3 chacun), `capBase` U[4..10], `trend` U[-10%..+10%].  
   `capByDay[d] = round(capBase × facteurJour × dérive linéaire(trend))`.
2. **Clients** : `Client-001..Client-030`.
3. **Ship matrix** `shipDays[client][centre]` : tirage discret {0..4} selon les poids.
4. **OpDurTE** : pour chaque `type×engine`, tirage triangulaire → **fixé**.
5. **Demandes existantes** (charge/heatmap) : `existing` demandes réparties dans l’horizon, affectées à un centre **compatible**, `duration = opDurTE[type][engine]`, priorité selon `%Urgent`.  
   > (Option de réalisme : biais d’arrivées “plus proche de la date d’opération”, et arrivée jusqu’à 6 mois avant, si l’horizon est étendu.)

---

## 3) INTERFACE (4 onglets, largeur 100%)
### 3.1 Onglet 1 — 📥 Demande & Reco
**Champs** : Client, Moteur, Type, Priorité, Date dispo.  
**Actions** : `Évaluer`, `Ajouter au planning`.  
**Table des centres** (compatibles seulement) :
- Colonnes : **Centre**, **⚡ FastETA (j)**, **🍀 ResponsibleETA (j)**, **Détails**  
- **Tri** par 🍀 ResponsibleETA croissant  
- **Surlignage** :  
  - 🍀 meilleur Responsible → **fond vert**  
  - ⚡ meilleur Fast → **fond jaune**  
  - `Attente>0` ou `Pénalité>0` → **valeur en rouge**  
- **Détails** (ex.) : `Aller Xj • Ops Yj • Attente Zj • Retour Xj • Pénalité Pj`

### 3.2 Onglet 2 — 🧩 Master Data
- Sliders/inputs pour tous les paramètres de 2.2
- **Regénérer** (recrée centres, matrices, demandes existantes)
- **Export/Import JSON** des `Params` + snapshot `State` (centres, demandes, matrices fixes)

### 3.3 Onglet 3 — 🏭 Centres • Heatmap
- Sélecteur de centre
- Heatmap **type × moteur** (lignes) × **S1..S12** (colonnes)
- Valeur = **max**(occupation_jour) sur la semaine ; palette **vert → jaune → rouge**

### 3.4 Onglet 4 — 📄 Demandes • Liste
- Filtres : centre, moteur, type, priorité, période
- Tableau trié par date

---

## 4) TECHNIQUE (algos & règles)
### 4.1 Calculs
- **Attente (semaine+1)** :  
  - Standard : si `max( load_j / cap_j )` sur Semaine 1 > 100%, `waitDays = ceil((occ-1)*7)`  
  - Urgent : `waitDays = 0` (priorité garantit ETA ≤ Standard)
- **FastETA** = `ship + opDurTE[type][engine] + wait + ship`  
  *(avec `ship = shipDays[client][centre]`, aller=retour, **fixe**)*  
- **Pénalité (Responsible)** :  
  - `occ4 = max_j∈S4( loadType_j / cap_j )`  (charge **par type**, au centre)  
  - `excess = max(0, occ4 - 1)`  
  - `avgDays(type) = (min + mode + max) / 3` (triangulaire)  
  - `penaltyDays = penaltyFactor * excess * avgDays(type)` (arrondi)  
- **🍀 ResponsibleETA** = `FastETA + penaltyDays`
- **Tri** : centres éligibles triés par **ResponsibleETA croissant** (onglet 1)

### 4.2 Occupation & Heatmap
- `loadByDay` et `loadByDayByType[type]` incrémentés **1 slot/jour** pendant `durationDays` à partir de `startDate`
- Heatmap : pour chaque **semaine** et **(type×moteur)** supporté par le centre,
  - On approxime la répartition “par moteur” en **partageant** `loadByDayByType[type]` sur le nombre de moteurs supportés
  - Cellule = `max( loadTypeShare_j / cap_j )` de la semaine, **bornée à 150%**

### 4.3 Déterminisme / stabilité
- Toute randomness est effectuée **une seule fois** à la **génération** (seed).
- À **paramètres de demande** inchangés, l’**ETA ne bouge pas** entre clics “Évaluer”.
- Passer **Standard → Urgent** ne peut que **réduire** (ou égaler) l’ETA.

### 4.4 États / Types (pseudo-TS)
```ts
type Priority = "Urgent" | "Standard";
type Engine = "LEAP-1A"|"LEAP-1B"|"CFM56-5B"|"CFM56-7B"|"SaM146";
type ReqType = "Overhaul"|"RepairOnly"|"QuickInspection"|"TestOnly";

interface Centre {
  id:number; name:string;
  engines:Set<Engine>; types:Set<ReqType>;
  capBase:number; trend:number;
  capByDay:number[]; loadByDay:number[];
  loadByDayByType: Record<ReqType, number[]>;
}

interface Demand {
  id:string; client:string; centre:Centre; centreName:string;
  engine:Engine; type:ReqType; priority:Priority;
  startDate:Date; duration:number;
}

interface Params {
  seed:number; weeks:number; clients:number; existing:number; pUrgent:number;
  capMin:number; capMax:number; trendMin:number; trendMax:number;
  durations: Record<ReqType, [number,number,number]>;
  shipW:number[]; reserveUrgPct:number; penaltyFactor:number;
}

interface State {
  baseDate:Date; horizonDays:number;
  centres:Centre[]; customers:string[]; demands:Demand[];
  shipDays: Record<string, Record<string, number>>; // client->centre->days
  opDurTE: Record<ReqType, Record<Engine, number>>;
}
```

