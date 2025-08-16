# MAESTRO Prototype Pseudocode (Aircraft Engine Maintenance)

## Global State
```
S = {
  mdLists: {},              // listes maîtres aéronautiques: Urgency (incl. AOG), Status, Shop (Engine Shop, Test Cell...), Location (Hangar/Test Facility),
                            // Operation Type (Inspection, Disassembly, Repair, Cleaning, Assembly, Test Run),
                            // Engine Model (CFM56, LEAP-1A, GE90...), Customer (compagnies aériennes, lessors),
                            // Request Type (Overhaul, Quick Inspection, Deep Repair...)

  requestTypeToOps: {},     // règle métier: chaque type de demande moteur -> 4 opérations obligatoires (ex: Overhaul = Disassembly, Repair, Assembly, Test Run)
  opDur: {},                // durées standard par type d'opération (jours/heures) selon pratiques MRO/atelier moteur
  shops: [],                // ateliers impliqués: Engine Shop (mécanique), Cleaning Shop, Test Cell (banc d'essai), etc.
  shopCapabilities: {},     // capacités par atelier: opérations autorisées, capacité quotidienne/hebdo, localisation, exceptions (pannes banc, congés)
  requests: [],             // demandes de maintenance moteur (par n° de série moteur, modèle, client, urgence AOG/Routine)
  operations: [],           // opérations planifiées (avec shop, durée, dates)
  tabs: ["FORM_New_Request","FORM_Edit_Operations","TR_Maint_Requests","OP_Maint_Operations","MD_Lists","KPI_Dashboard"]
}
```

## Initialization
```
function init():
  seedMasterData()          // injecte un référentiel moteur (modèles, clients, ateliers, opérations, urgences)
  seedTransactions()        // génère des demandes/opérations de démonstration (flotte moteur d'une compagnie)
  initTabs()
  populateDropdowns()
  renderReqTypeHelper()     // rappelle, pour chaque type de demande, les 4 opérations moteur requises
  renderShopHelper()        // affiche les spécialités/capacités d'un atelier (ex: Test Cell = Test Run uniquement)
  renderRequestsTable()
  renderOperationsTable()
  renderMDGrid()
  renderMappingTable()
  renderDurationTable()
  renderKPI()               // KPIs focalisés aviation: ponctualité AOG, charge des bancs d'essai, etc.
  renderJSONViewer()
  bindEvents()
  set default values (next IDs, initial dropdown selections)
```

## Data Seeding
```
function seedMasterData():
  // Remplit les listes maîtres orientées moteurs d'avion
  fill mdLists with sample values
    Urgency: [AOG (Aircraft On Ground), Routine]
    Status: [Planned, In Progress, Completed]
    Shop: [Engine Shop, Cleaning Shop, Test Cell]
    Location: [Hangar 1, Hangar 2, Test Facility]
    Operation Type: [Inspection, Disassembly, Repair, Cleaning, Assembly, Test Run]
    Engine Model: [CFM56, LEAP-1A, GE90]
    Customer: [Airline A, Airline B, Leasing Company]
    Request Type: [Overhaul, Quick Inspection, Deep Repair]

  // Chaque type de demande moteur impose exactement 4 opérations
  map each Request Type to exactly four Operation Types
    Overhaul -> [Disassembly, Repair, Assembly, Test Run]
    Quick Inspection -> [Inspection, Cleaning, Assembly, Test Run]
    Deep Repair -> [Disassembly, Repair, Assembly, Test Run]

  // Durées par défaut réalistes (exprimées en jours)
  assign default durations for each Operation Type
    Inspection=2, Disassembly=3, Repair=5, Cleaning=1, Assembly=4, Test Run=2

  // Capacités par atelier (ex: Test Cell limité par créneaux/bancs)
  create shopCapabilities with allowed operations, default capacity, location, and capacity exceptions
    Engine Shop: ops=[Inspection, Disassembly, Repair, Assembly], cap/day=8
    Cleaning Shop: ops=[Cleaning], cap/day=12
    Test Cell: ops=[Test Run], cap/day=3, exceptions=[maintenance banc, indispo]

function seedTransactions():
  // Génère un carnet de demandes moteur (flotte client)
  for i in 1..10:
    build Request with random master data (Engine Model, Customer, Urgency, Request Type, Request Date)
    estimate end date from required operations (somme durées + attentes inter-ateliers)
    append to S.requests

  // Planifie des opérations moteur jusqu'à 30 entrées en respectant les capacités d'ateliers
  while S.operations has < 30 entries:
    pick a Request
    choose allowed Operation Type (compatibles shopCapabilities)
    schedule start and duration (respect des créneaux Test Cell pour Test Run)
    append Operation to S.operations
```

## Helpers
```
function renderReqTypeHelper(): display required ops for each Request Type (pédagogie maintenance moteur)
function renderShopHelper(shop): list allowed ops and capacity info for shop (ex: Test Cell -> Test Run only, 3 slots/day)
function capacityFor(shop, date):
  determine capacity, used slots, and free slots considering exceptions and scheduled operations
  // utile pour AOG: vérifier si un créneau Test Run tôt est possible
function onCapLookup(): show capacityFor() result based on form inputs (shop/date)
```

## Request Form
```
function recalcEstEnd():
  if Request Type and Request Date set:
    sum durations of its operations + waiting time (ex: file d'attente Test Cell)
    set Estimated End Date (ETA remise moteur au client)

function onCreateRequest():
  validate required fields (Engine Model, Engine Serial, Customer, Urgency, Request Type)
  generate new Request ID
  store request in S.requests
  refresh dropdowns, tables, and JSON viewer
```

## Operation Editor
```
function onOpShopChanged(): filter Operation Types by selected Shop (évite de planifier Repair sur Test Cell)
function onOpTypeChanged(): preload default duration and recompute finish date (durées standard MRO)
function onOpStartDurChanged(): compute finish date from start + duration (gère chevauchements/attentes inter-shops)

function onSaveOperation():
  validate fields (cohérence modèle moteur/opération, shop compatible)
  create or update operation in S.operations
  refresh tables and viewer

function onDeleteOperation(): remove selected operation (réouvre la capacité du shop)
```

##
