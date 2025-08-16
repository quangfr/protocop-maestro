```
F: MAESTRO — Maintenance moteur (micro-DSL)

B:
  G: mapping Overhaul -> [Disassembly,Repair,Assembly,TestRun]
  G: mapping QuickInspection -> [Inspection,Cleaning,Assembly,TestRun]
  G: opDur = {Inspection:2, Disassembly:3, Repair:5, Cleaning:1, Assembly:4, TestRun:2}
  G: Shop("Engine").allows=[Inspection,Disassembly,Repair,Assembly]; cap/day=8
  G: Shop("Cleaning").allows=[Cleaning]; cap/day=12
  G: Shop("Test Cell").allows=[TestRun]; cap/day=3; exceptions=[]

R:
  1 RequestType => exactement 4 ops requises
  ETA = Σ(opDur des 4) + wait(capacity,shops)

E:
  OP_SHOP_INCAPABLE -> "Atelier non compatible"
  INVALID_MAPPING -> "Opération non requise"
  CAPACITY_FULL -> "Plus de créneau"
  MISSING_REQUIRED_OPS -> "Opérations requises incomplètes"

M:
  OnTimeUrgent% = AOG.deliveredOnOrBefore(ETA) / AOG.total

S: Créer une demande Overhaul -> status=Planned & 4 ops générées
  G: Planner authentifié
  W: CreateRequest(type=Overhaul, model=CFM56, urgency=AOG, date=2025-09-01)
  T: Request.status=Planned
  T: ops == [Disassembly,Repair,Assembly,TestRun]
  T: ETA calculée via R: ETA

S: Vérifier capacité Test Cell (slots restants)
  G: Test Cell a 2 TestRun planifiés le 2025-09-03
  W: CapacityLookup(shop="Test Cell", date=2025-09-03)
  T: slotsLibres=1

S: Bloquer type d’opération incompatible avec un shop
  G: Operation(type=Repair) ciblant shop="Test Cell"
  W: SaveOperation()
  T: erreur == OP_SHOP_INCAPABLE

S: Bloquer opération non requise pour le type de demande
  G: Request.type=QuickInspection
  W: AddOperation(type=Repair)
  T: erreur == INVALID_MAPPING

S: Assurer les 4 opérations requises (ensemble exact)
  G: Request.type=Overhaul; ops=[Disassembly,Repair,TestRun] (3/4)
  W: ValidateRequiredOps()
  T: erreur == MISSING_REQUIRED_OPS
  W: AddOperation(type=Assembly); ValidateRequiredOps()
  T: ok (ensemble complet), aucun doublon

S: Suppression réouvre la capacité
  G: Test Cell cap/day=3; 3 TestRun le 2025-09-05
  W: DeleteOperation(op=TestRun@2025-09-05)
  T: CapacityLookup("Test Cell",2025-09-05).slotsLibres=1

S: KPI Urgent On-time%
  G: 10 AOG livrées; 8 livrées ≤ ETA
  W: ComputeKPI()
  T: OnTimeUrgent%=0.8

S: Navigation croisée Requests ↔ Operations
  G: Tables Requests & Operations affichées
  W: Click(RequestID="R-123" | Ops#="4")
  T: Onglet Operations ouvert & filtré sur RequestID="R-123"

```
  W: Click(OperationID="OP-9")
  T: Formulaire d’édition chargé avec OP-9
