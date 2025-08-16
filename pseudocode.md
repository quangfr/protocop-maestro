# MAESTRO Prototype Pseudocode

## Global State
```
S = {
  mdLists: {},              // master data lists
  requestTypeToOps: {},     // request type -> required operations
  opDur: {},                // default operation durations
  shops: [],                // list of shops
  shopCapabilities: {},     // per-shop capabilities
  requests: [],             // maintenance requests
  operations: [],           // maintenance operations
  tabs: ["FORM_New_Request","FORM_Edit_Operations","TR_Maint_Requests","OP_Maint_Operations","MD_Lists","KPI_Dashboard"]
}
```

## Initialization
```
function init():
  seedMasterData()
  seedTransactions()
  initTabs()
  populateDropdowns()
  renderReqTypeHelper()
  renderShopHelper()
  renderRequestsTable()
  renderOperationsTable()
  renderMDGrid()
  renderMappingTable()
  renderDurationTable()
  renderKPI()
  renderJSONViewer()
  bindEvents()
  set default values (next IDs, initial dropdown selections)
```

## Data Seeding
```
function seedMasterData():
  fill mdLists with sample values (Urgency, Status, Shop, Location, Operation Type, Engine Model, Customer, Request Type)
  map each Request Type to exactly four Operation Types
  assign default durations for each Operation Type
  create shopCapabilities with allowed operations, default capacity, location, and capacity exceptions

function seedTransactions():
  for i in 1..50:
    build Request with random master data
    estimate end date from required operations
    append to S.requests
  while S.operations has < 200 entries:
    pick a Request
    choose allowed Operation Type
    schedule start and duration
    append Operation to S.operations
```

## Helpers
```
function renderReqTypeHelper(): display required ops for each Request Type
function renderShopHelper(shop): list allowed ops and capacity info for shop
function capacityFor(shop, date):
  determine capacity, used slots, and free slots considering exceptions and scheduled operations
function onCapLookup(): show capacityFor() result based on form inputs
```

## Request Form
```
function recalcEstEnd():
  if Request Type and Request Date set:
    sum durations of its operations + waiting time
    set Estimated End Date

function onCreateRequest():
  validate required fields
  generate new Request ID
  store request in S.requests
  refresh dropdowns, tables, and JSON viewer
```

## Operation Editor
```
function onOpShopChanged(): filter Operation Types by selected Shop
function onOpTypeChanged(): preload default duration and recompute finish date
function onOpStartDurChanged(): compute finish date from start + duration
function onSaveOperation():
  validate fields
  create or update operation in S.operations
  refresh tables and viewer
function onDeleteOperation(): remove selected operation
```

## Tables & Search
```
function renderRequestsTable(): filter by search term and display Request rows with links
function renderOperationsTable(): filter by Operation or Request ID and display rows with navigation links
```

## Master Data Editor
```
function renderMDGrid(): show editable textareas for each list in mdLists
function renderMappingTable(): editable Request Type -> four Operation Types mapping
function renderDurationTable(): editable durations per Operation Type

function applyMDChanges():
  read lists from textareas
  rebuild mapping and durations from tables
  rebuild shopCapabilities preserving existing data
  refresh dropdowns, helpers, tables, KPI, JSON viewer
```

## KPI Dashboard
```
function renderKPI():
  compute on-time percentage for urgent requests
  renderHeatmap()

function renderHeatmap():
  for each Location × Shop and next 8 weeks:
    compute utilization
    color cell green/yellow/red
```

## JSON & Persistence
```
function buildJSONSections(): gather state sections for viewer/export
function renderJSONViewer(): show collapsible JSON for each section
function copyJSON(): copy full state to clipboard
function downloadJSON(): download state as file
function doExport(): export state as JSON file
function doImport(file):
  parse JSON
  replace state S with imported values
  refresh UI elements
```

## Events
```
function bindEvents():
  attach handlers for form actions, tab navigation, searches, capacity lookup,
  master data apply, import/export, and JSON utilities
```

## Entry Point
```
document.addEventListener('DOMContentLoaded', init)
```
