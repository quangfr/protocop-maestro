<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>MAESTRO — Maintenance moteur (prototype offline)</title>
<style>
  :root{
    --bg:#f7f7f9; --card:#ffffff; --ink:#0b1220; --muted:#5f6b7a; --accent:#0a6ed1; --accent-weak:#d1e7ff;
    --ok:#2e7d32; --warn:#ed6c02; --err:#c62828; --border:#e5e7eb; --chip:#eef2f7; --shadow:0 6px 16px rgba(10,30,60,.08);
    --radius:14px; --pad:14px; --pad-lg:18px; --gap:12px; --tabh:46px; --sticky:62px;
    --mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  }
  html,body{height:100%;}
  body{margin:0;background:var(--bg);color:var(--ink);font:14px/1.45 system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,Noto Sans,sans-serif;}
  header{position:sticky;top:0;z-index:10;background:linear-gradient(180deg,#fff,#f9fbff);border-bottom:1px solid var(--border);
         box-shadow:0 2px 10px rgba(10,30,60,.06);} 
  .bar{display:flex;align-items:center;gap:14px;min-height:56px;padding:10px 16px;}
  .brand{font-weight:700;letter-spacing:.2px}
  .tag{padding:3px 8px;border-radius:999px;background:var(--accent-weak);color:var(--accent);font-weight:600;font-size:12px}
  nav{display:flex;gap:8px;padding:8px 12px;border-top:1px solid var(--border);} 
  .tab{appearance:none;border:1px solid var(--border);background:#fff;color:#1b2a3a;padding:8px 12px;border-radius:10px;cursor:pointer;font-weight:600}
  .tab[aria-selected="true"]{background:var(--accent);color:#fff;border-color:transparent}
  main{max-width:1200px;margin:16px auto;padding:0 14px 80px;}

  .grid{display:grid;grid-template-columns:1.3fr .8fr;gap:16px}
  .card{background:var(--card);border:1px solid var(--border);border-radius:var(--radius);box-shadow:var(--shadow)}
  .card .hd{display:flex;align-items:center;justify-content:space-between;padding:12px 14px;border-bottom:1px solid var(--border);position:sticky;top:var(--sticky);background:#fff;border-top-left-radius:var(--radius);border-top-right-radius:var(--radius);z-index:1}
  .card .bd{padding:14px}
  .hd h3{margin:0;font-size:14px}
  .kpi{display:flex;gap:12px;align-items:center}
  .kpi .pill{padding:4px 10px;border-radius:10px;background:var(--chip);font-weight:700}

  label{font-weight:600;display:block;margin-top:10px;margin-bottom:4px}
  input,select{width:100%;padding:10px 12px;border:1px solid var(--border);border-radius:10px;background:#fff}
  input[type=date]{padding:9px 10px}
  .row{display:grid;grid-template-columns:repeat(2,1fr);gap:12px}
  .actions{display:flex;gap:10px;margin-top:12px;flex-wrap:wrap}
  button{background:var(--accent);color:#fff;border:none;padding:10px 14px;border-radius:10px;font-weight:700;cursor:pointer}
  button.secondary{background:#fff;color:var(--accent);border:1px solid var(--accent)}
  button.ghost{background:#fff;color:#1b2a3a;border:1px dashed var(--border)}

  table{width:100%;border-collapse:separate;border-spacing:0;}
  th,td{padding:10px 12px;border-bottom:1px solid var(--border);vertical-align:top}
  thead th{position:sticky;top:calc(var(--sticky) + 2px);background:#fff;z-index:1}
  tbody tr:hover{background:#fbfcff}
  .chip{display:inline-block;padding:2px 8px;border-radius:999px;background:var(--chip);font-weight:600;font-size:12px}
  .ok{color:var(--ok)} .warn{color:var(--warn)} .err{color:var(--err)}

  .toast{position:fixed;right:14px;bottom:14px;max-width:520px;display:none;padding:12px 14px;border-radius:12px;background:#fff;border:1px solid var(--border);box-shadow:var(--shadow)}
  .toast.show{display:block}
  .toast .title{font-weight:800;margin-bottom:6px}

  .muted{color:var(--muted)}
  .help{font-size:12px;color:var(--muted)}

  /* tiny badges */
  .badge{font:12px var(--mono);padding:2px 6px;border-radius:6px;background:#0a6ed10f;border:1px solid #0a6ed123}

  @media (max-width: 960px){
    .grid{grid-template-columns:1fr}
  }
</style>
</head>
<body>
<header>
  <div class="bar">
    <div class="brand">MAESTRO</div>
    <span class="tag">Prototype offline</span>
    <span class="muted">— Maintenance moteur (SAP‑IBP Fiori‑like)</span>
  </div>
  <nav id="tabs"></nav>
</header>
<main>
  <!-- Tab: New Request -->
  <section class="tabpane" id="tab-new">
    <div class="grid">
      <div class="card">
        <div class="hd"><h3>Nouvelle demande</h3></div>
        <div class="bd">
          <div class="row">
            <div>
              <label>Type de demande</label>
              <select id="nr-type"></select>
            </div>
            <div>
              <label>Modèle moteur</label>
              <select id="nr-model"></select>
            </div>
          </div>
          <div class="row">
            <div>
              <label>Urgence</label>
              <select id="nr-urgency"></select>
            </div>
            <div>
              <label>Date de début</label>
              <input type="date" id="nr-date" value="2025-09-01"/>
            </div>
          </div>
          <div class="actions">
            <button id="btn-create">Créer la demande</button>
            <button class="ghost" id="btn-reset">Réinitialiser</button>
          </div>
          <div id="nr-result" class="help" style="margin-top:10px"></div>
        </div>
      </div>
      <div class="card">
        <div class="hd"><h3>Aide & Capacités</h3></div>
        <div class="bd">
          <div>
            <div style="font-weight:700;margin-bottom:4px">Opérations requises par type</div>
            <div id="reqtype-helper" class="help"></div>
          </div>
          <hr style="border:none;border-top:1px solid var(--border);margin:12px 0">
          <div>
            <div style="font-weight:700;margin-bottom:4px">Capacity Lookup</div>
            <div class="row">
              <div>
                <label>Atelier (Shop)</label>
                <select id="cap-shop"></select>
              </div>
              <div>
                <label>Date</label>
                <input type="date" id="cap-date" value="2025-09-03"/>
              </div>
            </div>
            <div class="actions">
              <button class="secondary" id="btn-cap">Vérifier</button>
            </div>
            <div id="cap-out" class="kpi" style="margin-top:8px"></div>
            <div class="help">Astuce: Test Cell a 2 TestRun planifiés le 2025‑09‑03 (cap/jour 3) ⇒ 1 slot libre attendu.</div>
          </div>
        </div>
      </div>
    </div>
  </section>

  <!-- Tab: Edit Operations -->
  <section class="tabpane" id="tab-edit">
    <div class="grid">
      <div class="card">
        <div class="hd"><h3>Éditer / Ajouter une opération</h3></div>
        <div class="bd">
          <div class="row">
            <div>
              <label>ID Opération</label>
              <input id="op-id" placeholder="OP-…" />
            </div>
            <div>
              <label>ID Demande</label>
              <input id="op-req" placeholder="R-…" />
            </div>
          </div>
          <div class="row">
            <div>
              <label>Type d’opération</label>
              <select id="op-type"></select>
            </div>
            <div>
              <label>Atelier</label>
              <select id="op-shop"></select>
            </div>
          </div>
          <div class="row">
            <div>
              <label>Date planifiée</label>
              <input type="date" id="op-date" />
            </div>
            <div>
              <label>Durée (jours)</label>
              <input id="op-dur" type="number" min="1" />
            </div>
          </div>
          <div class="help" style="margin-top:6px">Note: seuls les 4 types requis par la Demande seront acceptés. Atelier doit être compatible. Capacité/jour respectée.</div>
          <div class="actions">
            <button id="btn-saveop">Enregistrer</button>
            <button class="secondary" id="btn-addop">Ajouter (nouvel ID)</button>
            <button class="ghost" id="btn-delop">Supprimer</button>
            <button class="ghost" id="btn-validate">Valider opérations requises</button>
          </div>
          <div id="op-msg" class="help" style="margin-top:10px"></div>
        </div>
      </div>
      <div class="card">
        <div class="hd"><h3>Aide Atelier</h3></div>
        <div class="bd" id="shop-helper"></div>
      </div>
    </div>
  </section>

  <!-- Tab: Requests Table -->
  <section class="tabpane" id="tab-reqs">
    <div class="card">
      <div class="hd"><h3>Demandes</h3></div>
      <div class="bd">
        <table>
          <thead><tr>
            <th>ID</th><th>Type</th><th>Modèle</th><th>Urgence</th><th>Statut</th><th>Début</th><th>ETA</th><th>Ops #</th>
          </tr></thead>
          <tbody id="req-rows"></tbody>
        </table>
      </div>
    </div>
  </section>

  <!-- Tab: Operations Table -->
  <section class="tabpane" id="tab-ops">
    <div class="card">
      <div class="hd">
        <h3>Opérations</h3>
        <div class="actions" style="margin:0">
          <input id="ops-filter" placeholder="Filtrer par RequestID (R-…)">
          <button class="secondary" id="btn-opsfilter">Filtrer</button>
          <button class="ghost" id="btn-opsreset">Réinitialiser</button>
        </div>
      </div>
      <div class="bd">
        <table>
          <thead><tr>
            <th>ID</th><th>Request</th><th>Type</th><th>Shop</th><th>Date</th><th>Durée</th>
          </tr></thead>
          <tbody id="op-rows"></tbody>
        </table>
      </div>
    </div>
  </section>

  <!-- Tab: KPI -->
  <section class="tabpane" id="tab-kpi">
    <div class="grid">
      <div class="card">
        <div class="hd"><h3>KPI Urgent & Capacités</h3></div>
        <div class="bd" id="kpi-box">
          <!-- Filled by JS -->
        </div>
      </div>
      <div class="card">
        <div class="hd"><h3>Règles & Codes erreurs</h3></div>
        <div class="bd help">
          <div><span class="badge">R1</span> 1 RequestType ⇒ exactement 4 opérations requises</div>
          <div><span class="badge">R2</span> ETA = Σ(durées des 4) + attente(capacités des shops)</div>
          <div style="margin-top:10px"><span class="badge">OP_SHOP_INCAPABLE</span> Atelier non compatible</div>
          <div><span class="badge">INVALID_MAPPING</span> Opération non requise</div>
          <div><span class="badge">CAPACITY_FULL</span> Plus de créneau</div>
          <div><span class="badge">MISSING_REQUIRED_OPS</span> Opérations requises incomplètes</div>
        </div>
      </div>
    </div>
  </section>
</main>

<div class="toast" id="toast"><div class="title">Info</div><div id="toast-body"></div></div>

<script>
const S = {
  mdLists:{}, requestTypeToOps:{}, opDur:{}, shops:[],
  requests:[], operations:[],
  counters:{r:124, op:9}, // will be bumped; OP-9 exists in seeds
  tabs:[
    {id:'tab-new', label:'New Request'},
    {id:'tab-edit', label:'Edit Operations'},
    {id:'tab-reqs', label:'Requests'},
    {id:'tab-ops', label:'Operations'},
    {id:'tab-kpi', label:'KPI'}
  ],
  filter:{opsByReq:''}
};

// ---------- Utilities ----------
const el = (sel)=>document.querySelector(sel);
const fmtDate = (d)=> d.toISOString().slice(0,10);
const addDays = (str, n)=>{ const d=new Date(str+"T00:00:00"); d.setDate(d.getDate()+n); return fmtDate(d); };
const cmpDates = (a,b)=> new Date(a) - new Date(b);
const toast=(msg,type='info')=>{ const t=el('#toast'); el('#toast .title').textContent= type==='err'? 'Erreur' : 'Info'; el('#toast-body').textContent=msg; t.classList.add('show'); setTimeout(()=>t.classList.remove('show'), 3000); };

function idReq(){ return `R-${S.counters.r++}` }
function idOp(){ return `OP-${S.counters.op++}` }

// ---------- Seeding ----------
function seedMasterData(){
  S.mdLists.Urgency = ['AOG','High','Medium','Low'];
  S.mdLists.Status  = ['Planned','In Progress','Delivered','Cancelled'];
  S.mdLists.Shop    = ['Engine','Cleaning','Test Cell'];
  S.mdLists.Location= ['Hangar A','Hangar B','Test Facility'];
  S.mdLists.OpType  = ['Inspection','Disassembly','Repair','Cleaning','Assembly','TestRun'];
  S.mdLists.EngineModel=['CFM56','LEAP‑1A','GE90'];
  S.mdLists.Customer=['AirOne','SkyJet','BlueWings'];
  S.mdLists.RequestType=['Overhaul','QuickInspection'];

  S.requestTypeToOps = {
    Overhaul:['Disassembly','Repair','Assembly','TestRun'],
    QuickInspection:['Inspection','Cleaning','Assembly','TestRun']
  };
  S.opDur = {Inspection:2, Disassembly:3, Repair:5, Cleaning:1, Assembly:4, TestRun:2};
  S.shops = [
    {name:'Engine', allows:['Inspection','Disassembly','Repair','Assembly'], capPerDay:8, exceptions:[]},
    {name:'Cleaning', allows:['Cleaning'], capPerDay:12, exceptions:[]},
    {name:'Test Cell', allows:['TestRun'], capPerDay:3, exceptions:[]}
  ];
}

function seedTransactions(){
  // Ensure OP-9 exists and capacity examples are set
  // 1) Seed a couple of planned requests to host seeded ops
  const r1 = {id:'R-120', type:'Overhaul', model:'CFM56', urgency:'High', status:'Planned', startDate:'2025-09-02', eta:'', ops:[]};
  const r2 = {id:'R-121', type:'QuickInspection', model:'LEAP‑1A', urgency:'Medium', status:'Planned', startDate:'2025-09-03', eta:'', ops:[]};
  S.requests.push(r1,r2);
  // 2) Capacity: Test Cell on 2025-09-03 has 2 TestRun (cap 3) ⇒ slots libres = 1
  const op8 = {id:'OP-8', requestId:r1.id, type:'TestRun', shop:'Test Cell', date:'2025-09-03', duration:S.opDur.TestRun}; r1.ops.push(op8.id);
  const op9 = {id:'OP-9', requestId:r2.id, type:'TestRun', shop:'Test Cell', date:'2025-09-03', duration:S.opDur.TestRun}; r2.ops.push(op9.id);
  // 3) Capacity: Test Cell on 2025-09-05 has 3 TestRun fully booked ⇒ delete to reopen 1 slot
  const r3 = {id:'R-122', type:'Overhaul', model:'GE90', urgency:'High', status:'Planned', startDate:'2025-09-04', eta:'', ops:[]};
  S.requests.push(r3);
  const op10 = {id:'OP-10', requestId:r3.id, type:'TestRun', shop:'Test Cell', date:'2025-09-05', duration:S.opDur.TestRun}; r3.ops.push(op10.id);
  const op11 = {id:'OP-11', requestId:r1.id, type:'TestRun', shop:'Test Cell', date:'2025-09-05', duration:S.opDur.TestRun}; r1.ops.push(op11.id);
  const op12 = {id:'OP-12', requestId:r2.id, type:'TestRun', shop:'Test Cell', date:'2025-09-05', duration:S.opDur.TestRun}; r2.ops.push(op12.id);

  S.operations.push(op8,op9,op10,op11,op12);
  S.counters.op = 13; // next will be OP-13

  // R‑123 for cross navigation example
  const rNav = {id:'R-123', type:'Overhaul', model:'CFM56', urgency:'AOG', status:'Planned', startDate:'2025-08-28', eta:'', ops:[]};
  S.requests.push(rNav);
  const opsNav = ['Disassembly','Repair','Assembly','TestRun'].map((t,i)=>{
    const shop = shopForOp(t);
    const date = addDays('2025-08-29', i); // simple consecutive days
    const op = {id:idOp(), requestId:rNav.id, type:t, shop, date, duration:S.opDur[t]};
    rNav.ops.push(op.id); S.operations.push(op); return op;
  });

  // KPI sample: 10 AOG delivered, 8 on time
  for(let i=1;i<=10;i++){
    const r = {id:idReq(), type: (i%2? 'Overhaul':'QuickInspection'), model:'CFM56', urgency:'AOG', status:'Delivered', startDate:'2025-08-'+String(5+i).padStart(2,'0'), eta:'', ops:[]};
    // fake ETA and deliveredDate
    const eta = addDays(r.startDate, 12); r.eta = eta; r.deliveredDate = i<=8 ? eta : addDays(eta,1);
    S.requests.push(r);
  }
}

// ---------- Helpers ----------
function getShop(name){ return S.shops.find(s=>s.name===name); }
function shopForOp(opType){
  if(['Inspection','Disassembly','Repair','Assembly'].includes(opType)) return 'Engine';
  if(opType==='Cleaning') return 'Cleaning';
  if(opType==='TestRun') return 'Test Cell';
  return 'Engine';
}
function countOpsOn(shopName, date){ return S.operations.filter(o=>o.shop===shopName && o.date===date).length }
function capacityLeft(shopName, date){ const s=getShop(shopName); return Math.max(0, s.capPerDay - countOpsOn(shopName,date)); }

function etaByScheduling(req){
  // Greedy schedule the required 4 ops in order, respecting shop capacity; cursor moves forward
  const seq = S.requestTypeToOps[req.type] || [];
  let cursor = req.startDate || fmtDate(new Date());
  let lastEnd = cursor;
  seq.forEach(opType=>{
    const shop = shopForOp(opType);
    // find earliest date >= cursor with capacity
    let d = cursor;
    while(capacityLeft(shop,d) <= 0){ d = addDays(d,1); }
    // book (virtual): push a temp count by injecting a phantom reservation in a local map? Simpler: actually count includes existing ops; to avoid polluting S, we virtually increment via a local map
  });
  // To simplify ETA without virtual booking, use Σ(durations) + waiting days approximated as days with cap=0 on cursor per op.
  // Minimal demo ETA = start + Σ(durations) + number of days where next required shop is full
  let wait = 0; let d = req.startDate;
  (S.requestTypeToOps[req.type]||[]).forEach(opType=>{
    const shop = shopForOp(opType);
    // if full on day d, slide until free
    while(capacityLeft(shop,d) <= 0){ wait++; d = addDays(d,1); }
    d = addDays(d, S.opDur[opType]); // proceed after duration
  });
  const sumDur = (S.requestTypeToOps[req.type]||[]).reduce((a,t)=>a+(S.opDur[t]||0),0);
  return addDays(req.startDate, sumDur + wait);
}

// ---------- Rendering ----------
function initTabs(){
  const n = el('#tabs'); n.innerHTML='';
  S.tabs.forEach((t,i)=>{
    const b = document.createElement('button'); b.className='tab'; b.textContent=t.label; b.dataset.target=t.id; b.setAttribute('aria-selected', i===0);
    b.onclick=()=>switchTab(t.id);
    n.appendChild(b);
  });
  switchTab(S.tabs[0].id);
}
function switchTab(id){
  document.querySelectorAll('.tabpane').forEach(p=>p.style.display = p.id===id? 'block':'none');
  document.querySelectorAll('.tab').forEach(btn=>btn.setAttribute('aria-selected', btn.dataset.target===id));
}

function populateDropdowns(){
  const fill = (sel, arr)=>{ const s=el(sel); s.innerHTML=''; arr.forEach(v=>{ const o=document.createElement('option'); o.value=o.textContent=v; s.appendChild(o); }); };
  fill('#nr-type', S.mdLists.RequestType);
  fill('#nr-model', S.mdLists.EngineModel);
  fill('#nr-urgency', S.mdLists.Urgency);
  fill('#cap-shop', S.mdLists.Shop);
  fill('#op-type', S.mdLists.OpType);
  fill('#op-shop', S.mdLists.Shop);
  // Defaults for helper
  el('#cap-shop').value = 'Test Cell';
}

function renderReqTypeHelper(){
  const box = el('#reqtype-helper');
  box.innerHTML = Object.entries(S.requestTypeToOps).map(([k,v])=>`<div><b>${k}</b> → ${v.join(', ')}</div>`).join('');
}
function renderShopHelper(){
  const box = el('#shop-helper');
  box.innerHTML = S.shops.map(s=>{
    return `<div style="margin-bottom:8px"><div><b>${s.name}</b> <span class="help">cap/jour: ${s.capPerDay}</span></div>
            <div class="help">Autorise: ${s.allows.join(', ')}</div></div>`;
  }).join('');
}

function renderRequestsTable(){
  const tb = el('#req-rows');
  tb.innerHTML='';
  S.requests.forEach(r=>{
    // compute ETA if Planned and missing
    if(!r.eta && r.status==='Planned') r.eta = etaByScheduling(r);
    const tr = document.createElement('tr');
    const opsCount = r.ops?.length||0;
    tr.innerHTML = `
      <td><a href="#" class="link" data-req="${r.id}">${r.id}</a></td>
      <td>${r.type}</td>
      <td>${r.model}</td>
      <td><span class="chip">${r.urgency}</span></td>
      <td>${r.status}</td>
      <td>${r.startDate||''}</td>
      <td>${r.eta||''}</td>
      <td><a href="#" class="opslink" data-req="${r.id}">${opsCount}</a></td>`;
    tb.appendChild(tr);
  });
  tb.querySelectorAll('a.link, a.opslink').forEach(a=>{
    a.onclick = (e)=>{ e.preventDefault(); const id=a.dataset.req; el('#ops-filter').value=id; filterOps(); switchTab('tab-ops'); };
  });
}

function renderOperationsTable(){
  const tb = el('#op-rows'); tb.innerHTML='';
  const list = S.operations.filter(o=> !S.filter.opsByReq || o.requestId===S.filter.opsByReq );
  list.sort((a,b)=> cmpDates(a.date,b.date) || a.id.localeCompare(b.id));
  list.forEach(o=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td><a href="#" data-op="${o.id}">${o.id}</a></td>
      <td><a href="#" data-req="${o.requestId}">${o.requestId}</a></td>
      <td>${o.type}</td>
      <td>${o.shop}</td>
      <td>${o.date}</td>
      <td>${o.duration}</td>`;
    tb.appendChild(tr);
  });
  // bind
  tb.querySelectorAll('a[data-op]').forEach(a=> a.onclick=(e)=>{ e.preventDefault(); loadOp(a.dataset.op); switchTab('tab-edit'); });
  tb.querySelectorAll('a[data-req]').forEach(a=> a.onclick=(e)=>{ e.preventDefault(); el('#ops-filter').value=a.dataset.req; filterOps(); });
}

function renderKPI(){
  const box = el('#kpi-box');
  const aogDelivered = S.requests.filter(r=>r.urgency==='AOG' && r.status==='Delivered');
  const ontime = aogDelivered.filter(r=> (r.deliveredDate||'') && cmpDates(r.deliveredDate, r.eta) <= 0 );
  const pct = aogDelivered.length? (ontime.length / aogDelivered.length) : 0;
  // mini capacity snapshot by shop for a chosen date (today by default)
  const today = el('#cap-date')?.value || fmtDate(new Date());
  const capHtml = S.shops.map(s=>{
    const free = capacityLeft(s.name, today);
    const tone = free>0? 'ok' : 'err';
    return `<div><b>${s.name}</b> — <span class="${tone}">${free} slots libres</span> <span class="help">(${today})</span></div>`;
  }).join('');
  box.innerHTML = `
    <div class="kpi" style="margin-bottom:12px">
      <div class="pill">On‑time AOG %</div>
      <div style="font-size:28px;font-weight:800">${(pct*100).toFixed(0)}%</div>
      <div class="help">(${ontime.length} / ${aogDelivered.length})</div>
    </div>
    <div style="font-weight:700;margin-bottom:6px">Capacité par shop (snapshot)</div>
    ${capHtml}
  `;
}

// ---------- Interactions ----------
function createRequest(){
  const type = el('#nr-type').value;
  const model = el('#nr-model').value;
  const urgency = el('#nr-urgency').value;
  const date = el('#nr-date').value;
  const req = {id:idReq(), type, model, urgency, status:'Planned', startDate:date, eta:'', ops:[]};
  // auto-generate 4 ops
  const required = S.requestTypeToOps[type]||[];
  let cursor = date;
  required.forEach(t=>{
    const shop = shopForOp(t);
    // find earliest day >= cursor with capacity
    let d = cursor;
    while(capacityLeft(shop,d) <= 0){ d = addDays(d,1); }
    const op = {id:idOp(), requestId:req.id, type:t, shop, date:d, duration:S.opDur[t]};
    S.operations.push(op); req.ops.push(op.id);
    cursor = addDays(d, S.opDur[t]); // proceed after duration
  });
  req.eta = etaByScheduling(req);
  S.requests.push(req);
  renderRequestsTable(); renderOperationsTable(); renderKPI();
  el('#nr-result').innerHTML = `<b>${req.id}</b> créé — statut <b>Planned</b>, ETA <b>${req.eta}</b>, ops: ${required.join(', ')}`;
  toast('Demande créée avec 4 opérations générées.');
}

function loadOp(opId){
  const o = S.operations.find(x=>x.id===opId); if(!o) return toast('Opération introuvable','err');
  el('#op-id').value = o.id;
  el('#op-req').value = o.requestId;
  el('#op-type').value = o.type;
  el('#op-shop').value = o.shop;
  el('#op-date').value = o.date;
  el('#op-dur').value = o.duration;
  el('#op-msg').textContent = `Chargé ${o.id}.`;
}

function saveOp(existing=true){
  const id = el('#op-id').value.trim();
  const requestId = el('#op-req').value.trim();
  const type = el('#op-type').value;
  const shop = el('#op-shop').value;
  const date = el('#op-date').value;
  const duration = Number(el('#op-dur').value||1);
  // validations
  const req = S.requests.find(r=>r.id===requestId);
  if(!req){ toast('Demande inconnue','err'); return; }
  // mapping rule
  const allowedTypes = S.requestTypeToOps[req.type]||[];
  if(!allowedTypes.includes(type)){ toast('Opération non requise (INVALID_MAPPING)','err'); return; }
  // shop capability
  const shopObj = getShop(shop);
  if(!shopObj.allows.includes(type)){ toast('Atelier non compatible (OP_SHOP_INCAPABLE)','err'); return; }
  // capacity
  const used = countOpsOn(shop,date);
  if(used >= shopObj.capPerDay){ toast('Plus de créneau (CAPACITY_FULL)','err'); return; }

  let op;
  if(existing){
    op = S.operations.find(o=>o.id===id);
    if(!op){ toast('Opération introuvable','err'); return; }
    op.requestId=requestId; op.type=type; op.shop=shop; op.date=date; op.duration=duration;
  } else {
    op = {id:id||idOp(), requestId, type, shop, date, duration};
    S.operations.push(op);
    const r = S.requests.find(r=>r.id===requestId); if(r && !r.ops.includes(op.id)) r.ops.push(op.id);
  }
  // recompute ETA for the request
  req.eta = etaByScheduling(req);
  renderOperationsTable(); renderRequestsTable(); renderKPI();
  el('#op-msg').textContent = `${existing? 'Enregistré':'Ajouté'} ${op.id}.`;
  toast(`${existing? 'Opération mise à jour':'Opération ajoutée'}.`);
}

function delOp(){
  const id = el('#op-id').value.trim();
  const idx = S.operations.findIndex(o=>o.id===id);
  if(idx<0){ toast('Opération introuvable','err'); return; }
  const op = S.operations[idx];
  S.operations.splice(idx,1);
  const r = S.requests.find(r=>r.id===op.requestId);
  if(r){ r.ops = r.ops.filter(x=>x!==id); r.eta = etaByScheduling(r); }
  renderOperationsTable(); renderRequestsTable(); renderKPI();
  el('#op-msg').textContent = `Supprimé ${id}.`;
  toast('Opération supprimée. Capacité réouverte.');
}

function validateRequiredOps(){
  const rid = el('#op-req').value.trim();
  const r = S.requests.find(x=>x.id===rid);
  if(!r){ toast('Demande inconnue','err'); return; }
  const required = new Set(S.requestTypeToOps[r.type]||[]);
  const present = new Set(S.operations.filter(o=>o.requestId===rid).map(o=>o.type));
  let ok = (required.size===present.size && [...required].every(t=>present.has(t)));
  if(!ok){ toast('Opérations requises incomplètes (MISSING_REQUIRED_OPS)','err'); }
  else { toast('OK: ensemble complet, aucun doublon.'); }
}

function capacityLookup(){
  const shop = el('#cap-shop').value; const date = el('#cap-date').value;
  const free = capacityLeft(shop,date); const used = countOpsOn(shop,date); const cap = getShop(shop).capPerDay;
  el('#cap-out').innerHTML = `<div class="pill">${shop}</div><div><b>${free}</b> slots libres</div><div class="help">${used}/${cap} utilisés</div>`;
}

function filterOps(){ S.filter.opsByReq = el('#ops-filter').value.trim(); renderOperationsTable(); }

// ---------- Bind ----------
function bind(){
  el('#btn-create').onclick = createRequest;
  el('#btn-reset').onclick = ()=>{ el('#nr-type').selectedIndex=0; el('#nr-model').selectedIndex=0; el('#nr-urgency').selectedIndex=0; el('#nr-date').value=fmtDate(new Date()); el('#nr-result').textContent=''; };
  el('#btn-cap').onclick = ()=>{ capacityLookup(); renderKPI(); };
  el('#btn-saveop').onclick = ()=>saveOp(true);
  el('#btn-addop').onclick = ()=>saveOp(false);
  el('#btn-delop').onclick = delOp;
  el('#btn-validate').onclick = validateRequiredOps;
  el('#btn-opsfilter').onclick = filterOps;
  el('#btn-opsreset').onclick = ()=>{ el('#ops-filter').value=''; filterOps(); };
}

// ---------- Init ----------
function init(){
  seedMasterData();
  seedTransactions();
  initTabs();
  populateDropdowns();
  renderReqTypeHelper();
  renderShopHelper();
  renderRequestsTable();
  renderOperationsTable();
  capacityLookup();
  renderKPI();
  bind();
}

init();
</script>
</body>
</html>
