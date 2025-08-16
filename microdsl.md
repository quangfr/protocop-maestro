APP: MAESTRO
STYLE: cards + tabs; sticky-headers; capacity-colors(green,yellow,red); rounded-buttons
GOAL: offline single-file HTML; create/modify requests; track ops; check shop capacity; view KPIs

DATA:
  MD Urgency = [Low,Medium,High,AOG]
  MD Status  = [New,Planned,InProgress,Done]
  MD Location= [Lyon,Paris,Toulouse,Nantes,Marseille]
  MD EngineModel = [CFM56,LEAP1A,GE90,Trent700,CF34]
  MD Customer    = [AirOne,EuroFly,BlueJet,SkyGo,Aerovia]
  MD OpType = [Inspection,Disassembly,Repair,Cleaning,Assembly,TestRun,Borescope,Calibration,Paint,ModuleSwap,Balancing,Shipping]

  SHOP Engine    (loc=Lyon,     cap/wk=8)  allows [Inspection,Disassembly,Repair,Assembly]
  SHOP Cleaning  (loc=Lyon,     cap/wk=12) allows [Cleaning]
  SHOP TestCell  (loc=Lyon,     cap/wk=3)  allows [TestRun]
  SHOP Paint     (loc=Paris,    cap/wk=6)  allows [Paint]
  SHOP Module    (loc=Toulouse, cap/wk=5)  allows [ModuleSwap,Balancing]

  MAP RequestType Overhaul        -> [Disassembly,Repair,Assembly,TestRun]
  MAP RequestType QuickInspection -> [Inspection,Cleaning,Assembly,TestRun]
  MAP RequestType DeepRepair      -> [Disassembly,Repair,Calibration,TestRun]

  DUR defaultWeeks {
    Inspection:2, Disassembly:3, Repair:5, Cleaning:1, Assembly:4, TestRun:2,
    Calibration:2, Paint:2, ModuleSwap:3, Balancing:1, Borescope:1, Shipping:1
  }

  SEED counts {Urgency:4, Status:4, Shops:10, Locations:5, OpTypes:12, EngineModels:5, Customers:5,
               RequestTypes:3, Requests:10, Operations:30}

REL:
  Request 1..* Operation
  Shop 1..* Operation
  RequestType -> exactly 4 distinct OpType

RULES:
  REQ.required = [Client,EngineModel,Location,Shop,RequestType,Urgency,Status,CreatedAt]
  REQ.endDate  = auto(calc from required ops & DUR & capacity wait)
  REQ.opsRequired = 4 distinct per RequestType

  OP.required  = [RequestId,Shop,OpType,Start,DurationWk]
  OP.durationRange = 1..5wk
  OP.endDate   = Start + DurationWk(ISO weeks)
  OP.mustBeAllowedByShop = true

  CAP.period = ISO week; compute per Shop (KPI shows per-shop capacity; heatmap by Location+Shop over next 8w)

NAV:
  Tabs = [NewRequest,EditOperations,Requests,Operations,MasterData,KPI]
  Click Request# -> open Operations(filter=RequestId)
  Click Operation# -> open EditOperations(opId)
  MasterData editable -> feeds dropdowns live
  JSON viewer/export/import = enabled

UI:
  Helper showReqTypeOps = true
  Helper showShopCapability = true
  CapacityColors = {green>=70%, yellow=40..69%, red<40%}
  Mobile breakpoint <768px -> stack sections

ERRORS:
  OP_SHOP_INCAPABLE = "Atelier non compatible"
  INVALID_MAPPING   = "Opération non requise"
  CAPACITY_FULL     = "Plus de créneau"
  MISSING_REQUIRED_OPS = "Opérations requises incomplètes"
  DURATION_OUT_OF_RANGE = "Durée hors 1..5 semaines"

TESTS:
  T1 "REQ minimal ok":
    GIVEN seed
    WHEN create Request R1 {Client:AirOne, EngineModel:CFM56, Location:Lyon, Shop:Engine,
                            RequestType:Overhaul, Urgency:High, Status:New, CreatedAt:2025-08-16}
    THEN R1.endDate = auto
     AND requiredOps(R1) = [Disassembly,Repair,Assembly,TestRun]
     AND ops.count(R1) = 4 (auto-created, planned)

  T2 "OP authorization":
    GIVEN Request R2 {type:QuickInspection, Shop:Cleaning}
    WHEN add Operation O1 {request:R2, opType:Repair, start:2025-09-01, dur:2}
    THEN ERROR OP_SHOP_INCAPABLE

  T3 "OP duration bounds":
    WHEN add Operation O2 {dur:6wk}
    THEN ERROR DURATION_OUT_OF_RANGE

  T4 "OP auto end date":
    WHEN add Operation O3 {start:2025-09-01, dur:3wk}
    THEN O3.end = 2025-09-22

  T5 "Mapping must be 4 ops":
    WHEN MAP RequestType QuickInspection -> [Inspection,Cleaning,Assembly]
    THEN ERROR MISSING_REQUIRED_OPS

  T6 "Mapping ops must be distinct":
    WHEN MAP RequestType Overhaul -> [Disassembly,Repair,Repair,TestRun]
    THEN ERROR INVALID_MAPPING

  T7 "Weekly capacity guard":
    GIVEN Shop TestCell cap/wk=1 AND week 2025-W36 has 1 TestRun booked
    WHEN schedule Operation {opType:TestRun, startWeek:2025-W36}
    THEN ERROR CAPACITY_FULL

  T8 "Nav: Requests -> Operations":
    WHEN click Request#R1
    THEN view=Operations AND filter=RequestId==R1

  T9 "Nav: Operations -> Edit":
    WHEN click Operation#O3
    THEN view=EditOperations AND form.opId==O3

  T10 "Master data live update":
    WHEN MD Customer += "AeroNova"
    THEN NewRequest.customerDropdown includes "AeroNova"

  T11 "KPI urgent on-time %":
    GIVEN urgent R3 plannedEnd=2025-09-10 actualEnd=2025-09-08
      AND urgent R4 plannedEnd=2025-09-10 actualEnd=2025-09-12
    WHEN compute KPI.onTimeUrgent%
    THEN KPI.onTimeUrgent% = 50%

  T12 "JSON roundtrip":
    WHEN export allData -> file A AND import file A into fresh app
    THEN counts match AND checksum match

  T13 "Heatmap 8 weeks":
    WHEN render heatmap
    THEN weeksShown=8 AND axes={Location,Shop}

  T14 "Sticky headers":
    WHEN scroll table Requests
    THEN header.visible = true

  T15 "Mobile stacking":
    WHEN viewport < 768px
    THEN sections.stack = true
