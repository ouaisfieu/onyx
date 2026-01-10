Le switch “Noms réels” ne marche plus parce que ton portail attend `entity.display.real_name`, alors que ton fichier master utilise `entity.labels.real` + `fields[]` (claims) — donc l’UI ne trouve rien à afficher. Idem pour le “niveau de détail” : il faut rendre `fields[]` (avec confidence + evidence), pas un simple objet `fields{}`. (Tu vois le mismatch dans l’import tel qu’il est codé. )

```html
<!doctype html>
<html lang="fr">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover"/>
<meta name="color-scheme" content="dark"/>
<title>NYXO • Obsidian Monitor</title>
<style>
  :root{
    --bg0:#05070b;
    --bg1:#070c12;
    --panel:#0b1220cc;
    --panel2:#0b1220f2;
    --line:#1b2944;
    --mut:#91a6c9;
    --txt:#eaf1ff;
    --good:#86fbd6;
    --lilac:#d7a8ff;
    --warn:#ffd36a;
    --bad:#ff6a8b;
    --glass: blur(14px) saturate(140%);
    --shadow: 0 18px 60px rgba(0,0,0,.55);
    --radius: 18px;
    --radius2: 14px;
    --mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono","Courier New", monospace;
    --sans: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji","Segoe UI Emoji";
  }
  *{box-sizing:border-box}
  html,body{height:100%}
  body{
    margin:0;
    font-family:var(--sans);
    color:var(--txt);
    overflow:hidden;
    background:
      radial-gradient(900px 600px at 15% 10%, rgba(134,251,214,.14), transparent 60%),
      radial-gradient(900px 600px at 85% 15%, rgba(215,168,255,.12), transparent 55%),
      radial-gradient(900px 650px at 55% 95%, rgba(134,251,214,.09), transparent 55%),
      linear-gradient(180deg,var(--bg0),var(--bg1));
  }

  canvas{position:fixed; inset:0; width:100%; height:100%}

  /* subtle overlays */
  .scanlines,.grain{position:fixed; inset:0; pointer-events:none; mix-blend-mode:overlay}
  .scanlines{opacity:.14;background:linear-gradient(to bottom, rgba(255,255,255,.05) 1px, transparent 1px);background-size:100% 3px}
  .grain{opacity:.10;background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.9' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='160' height='160' filter='url(%23n)' opacity='.45'/%3E%3C/svg%3E");background-size:220px 220px}

  /* HUD */
  .hud{
    position:fixed; left:12px; right:12px; top:12px;
    display:flex; align-items:center; gap:10px;
    z-index:20;
  }
  .logoBtn{
    width:44px; height:44px; border-radius:14px;
    background: radial-gradient(18px 18px at 30% 30%, rgba(134,251,214,.35), rgba(0,0,0,.05)),
                linear-gradient(180deg, rgba(13,20,34,.85), rgba(8,12,20,.85));
    border:1px solid rgba(255,255,255,.14);
    box-shadow: 0 12px 40px rgba(0,0,0,.35), 0 0 30px rgba(134,251,214,.15);
    display:grid; place-items:center;
    cursor:pointer;
    user-select:none;
  }
  .logoBtn span{
    font-weight:950;
    letter-spacing:.06em;
    font-family:var(--mono);
    font-size:12px;
    color:rgba(234,241,255,.92);
    text-shadow: 0 0 18px rgba(134,251,214,.22);
  }

  .toolbarWrap{flex:1; min-width:0}
  .toolbar{
    display:flex; gap:8px; align-items:center; flex-wrap:wrap;
    padding:8px;
    border-radius:16px;
    background:rgba(0,0,0,.18);
    border:1px solid rgba(255,255,255,.10);
    backdrop-filter: var(--glass);
    box-shadow: var(--shadow);
  }
  .btn{
    height:34px;
    padding:0 10px;
    border-radius:12px;
    border:1px solid rgba(255,255,255,.14);
    background:rgba(0,0,0,.18);
    color:var(--txt);
    cursor:pointer;
    font-weight:800;
    letter-spacing:.02em;
  }
  .btn:active{transform:translateY(1px)}
  .btn.primary{
    border-color: rgba(134,251,214,.35);
    background: linear-gradient(180deg, rgba(134,251,214,.16), rgba(0,0,0,.12));
    box-shadow: 0 0 26px rgba(134,251,214,.10);
  }
  .pill{
    display:flex; gap:8px; align-items:center;
    height:34px; padding:0 10px;
    border-radius:12px;
    border:1px solid rgba(255,255,255,.14);
    background:rgba(0,0,0,.12);
    color:rgba(234,241,255,.84);
    font-size:12px;
    white-space:nowrap;
  }
  .pill input{accent-color: var(--good)}
  .field{
    display:flex; gap:8px; align-items:center;
    height:34px; padding:0 10px;
    border-radius:12px;
    border:1px solid rgba(255,255,255,.14);
    background:rgba(0,0,0,.12);
    font-size:12px;
  }
  .field label{color:rgba(234,241,255,.72)}
  .field input[type="range"]{width:98px}
  .field input[type="text"]{
    width:150px;
    background:transparent;
    border:0;
    outline:none;
    color:var(--txt);
    font-family:var(--mono);
    font-size:12px;
  }

  /* Mobile-first access to monitor */
  .fab{
    position:fixed; right:12px; bottom:12px;
    z-index:22;
  }

  /* Fullscreen monitor */
  .drawerWrap{
    position:fixed; inset:0;
    pointer-events:none;
    z-index:40;
  }
  .drawerWrap.open{pointer-events:auto}
  .scrim{
    position:absolute; inset:0;
    background:rgba(0,0,0,.55);
    opacity:0;
    transition:opacity .18s ease;
  }
  .drawerWrap.open .scrim{opacity:1}
  .drawer{
    position:absolute; inset:0;
    transform: translateY(14px);
    opacity:0;
    transition: transform .20s ease, opacity .20s ease;
    display:flex;
    padding:12px;
  }
  .drawerWrap.open .drawer{transform:translateY(0); opacity:1}
  .panel{
    flex:1;
    border-radius:22px;
    background: rgba(7,10,16,.75);
    border:1px solid rgba(255,255,255,.12);
    box-shadow: var(--shadow);
    backdrop-filter: var(--glass);
    overflow:hidden;
    display:flex;
    flex-direction:column;
    min-height:0;
  }
  .hd{
    padding:12px;
    border-bottom:1px solid rgba(255,255,255,.10);
    display:flex;
    align-items:center;
    justify-content:space-between;
    gap:10px;
  }
  .hd .h{
    font-weight:950;
    letter-spacing:.08em;
    font-size:12px;
    color:rgba(234,241,255,.88);
    font-family:var(--mono);
  }
  .bd{
    padding:12px;
    overflow:auto;
    min-height:0;
  }

  .kpis{display:grid; grid-template-columns: repeat(4, 1fr); gap:10px}
  .kpi{
    border-radius:16px;
    border:1px solid rgba(255,255,255,.10);
    background:rgba(0,0,0,.16);
    padding:10px;
    min-height:72px;
  }
  .kpi .t{font-size:11px;color:rgba(234,241,255,.72)}
  .kpi .v{font-family:var(--mono);font-weight:950;font-size:20px;margin-top:4px}

  .sep{height:1px;background:rgba(255,255,255,.10);margin:12px 0}

  .grid2{display:grid; grid-template-columns: 1fr 1fr; gap:12px}
  @media(max-width:920px){.grid2{grid-template-columns:1fr}}

  .bar{height:10px;border-radius:99px;background:rgba(255,255,255,.08);overflow:hidden;border:1px solid rgba(255,255,255,.10)}
  .bar i{display:block;height:100%;width:0%;background:linear-gradient(90deg, rgba(255,106,139,.85), rgba(255,211,106,.9), rgba(134,251,214,.9));}

  .chipRow{display:flex;flex-wrap:wrap;gap:8px}
  .chip{
    font-family:var(--mono);
    font-size:11px;
    padding:6px 10px;
    border-radius:999px;
    border:1px solid rgba(255,255,255,.12);
    background:rgba(0,0,0,.12);
    color:rgba(234,241,255,.86);
  }
  .chip.good{border-color:rgba(134,251,214,.35);background:rgba(134,251,214,.10)}
  .chip.warn{border-color:rgba(255,211,106,.35);background:rgba(255,211,106,.10)}
  .chip.bad{border-color:rgba(255,106,139,.35);background:rgba(255,106,139,.10)}

  .list{
    border:1px solid rgba(255,255,255,.10);
    border-radius:16px;
    overflow:hidden;
    background:rgba(0,0,0,.14);
  }
  .item{
    padding:10px 12px;
    border-bottom:1px solid rgba(255,255,255,.08);
    display:flex;
    gap:10px;
    align-items:flex-start;
    cursor:pointer;
  }
  .item:last-child{border-bottom:0}
  .badge{width:10px;height:10px;border-radius:50%;margin-top:5px;flex:0 0 auto;opacity:.95}
  .item .meta{flex:1;min-width:0}
  .item .id{font-family:var(--mono);font-size:12px;font-weight:950;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
  .item .sub{font-size:12px;color:var(--mut);margin-top:2px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
  .item .right{font-family:var(--mono);font-size:11px;color:rgba(234,241,255,.70)}
  .hint{
    border-left:6px solid rgba(134,251,214,.50);
    background:rgba(134,251,214,.08);
    border:1px solid rgba(134,251,214,.18);
    border-radius:16px;
    padding:10px 12px;
    color:rgba(234,241,255,.82);
    font-size:12px;
  }

  textarea{
    width:100%;
    min-height:190px;
    resize:vertical;
    border-radius:14px;
    border:1px solid rgba(255,255,255,.14);
    background:rgba(0,0,0,.18);
    color:var(--txt);
    font-family:var(--mono);
    font-size:12px;
    line-height:1.45;
    padding:10px;
    outline:none;
  }
  .rowbtn{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px}

  /* hidden file input */
  #fileInput{position:absolute;left:-9999px;width:1px;height:1px;opacity:0}

  /* micro mobile */
  @media(max-width:430px){
    .field input[type="text"]{width:120px}
    .field input[type="range"]{width:86px}
    .kpis{grid-template-columns: repeat(2, 1fr)}
  }
</style>
</head>
<body>
<canvas id="stage"></canvas>
<div class="scanlines"></div>
<div class="grain"></div>

<div class="hud">
  <div class="logoBtn" id="homeBtn" title="Recentre"><span>!NYXO</span></div>

  <div class="toolbarWrap">
    <div class="toolbar" id="toolbar">
      <button class="btn primary" id="modeBtn">ZONES</button>
      <button class="btn" id="regenBtn">↻</button>

      <div class="pill"><span>Labels</span><input id="labels" type="checkbox"></div>
      <div class="pill"><span>Overlap</span><input id="overlap" type="checkbox" checked></div>
      <div class="pill"><span>Noms réels</span><input id="reveal" type="checkbox"></div>

      <div class="field"><label>Zoom</label><input id="zoom" type="range" min="40" max="180" value="100"></div>
      <div class="field"><label>Filtre</label><input id="filter" type="text" placeholder="type:place tag:atelier conf:>0.8"></div>

      <button class="btn" id="drawerToggle">Monitor ▸</button>
    </div>
  </div>
</div>

<div class="fab">
  <button class="btn primary" id="drawerFab">Monitor</button>
</div>

<div class="drawerWrap" id="drawerWrap" aria-hidden="true">
  <div class="scrim" id="scrim"></div>

  <div class="drawer" role="dialog" aria-label="NYXO Monitor">
    <div class="panel">
      <div class="hd">
        <div class="h">NYXO • MONITOR</div>
        <div style="display:flex;gap:8px;align-items:center;flex-wrap:wrap;justify-content:flex-end">
          <button class="btn" id="importFileBtn">Importer .json</button>
          <button class="btn" id="exportBtn">Exporter (UI)</button>
          <button class="btn" id="exportMasterBtn">Exporter (Master)</button>
          <button class="btn" id="sampleBtn">Exemple</button>
          <button class="btn" id="closeDrawer">✕</button>
        </div>
      </div>

      <div class="bd">
        <div class="kpis">
          <div class="kpi"><div class="t">Entités (filtre)</div><div class="v" id="kEnt">0</div></div>
          <div class="kpi"><div class="t">Liens</div><div class="v" id="kRel">0</div></div>
          <div class="kpi"><div class="t">Zones</div><div class="v" id="kZon">0</div></div>
          <div class="kpi"><div class="t">Champs (claims)</div><div class="v" id="kFld">0</div></div>
        </div>

        <div class="sep"></div>

        <div class="kpi">
          <div class="t">Fiabilité moyenne (entités filtrées)</div>
          <div class="v" id="kConf">0.00</div>
          <div class="bar"><i id="confBar"></i></div>
          <div class="t" style="margin-top:8px">Score “risque” (plus bas = plus risqué)</div>
          <div class="chipRow" style="margin-top:8px" id="riskChips"></div>
        </div>

        <div class="sep"></div>

        <div class="grid2">
          <div>
            <div style="display:flex;gap:8px;align-items:center;justify-content:space-between">
              <div style="font-weight:950;letter-spacing:.06em;font-size:12px;color:rgba(234,241,255,.88);font-family:var(--mono)">Liste</div>
              <div class="chipRow" id="typeChips"></div>
            </div>
            <div class="list" id="entityList" style="margin-top:10px"></div>
          </div>

          <div>
            <div style="font-weight:950;letter-spacing:.06em;font-size:12px;color:rgba(234,241,255,.88);font-family:var(--mono)">Détails</div>
            <div id="details" class="list" style="margin-top:10px"></div>

            <div class="sep"></div>

            <div style="font-weight:950;letter-spacing:.06em;font-size:12px;color:rgba(234,241,255,.88);font-family:var(--mono)">Import / édition rapide</div>
            <textarea id="jsonArea" spellcheck="false" style="margin-top:10px"></textarea>
            <div class="rowbtn">
              <button class="btn" id="applyBtn">Appliquer (JSON)</button>
              <button class="btn" id="clearBtn">Vider</button>
            </div>

            <div class="hint" style="margin-top:10px">
              Importe ton fichier master (qui contient <span style="font-family:var(--mono)">entities + relations + labels + fields[]</span>) : l’appli adapte automatiquement.
              Aucun nom réel n’est affiché tant que “Noms réels” est OFF.
            </div>

            <div class="sep"></div>

            <div style="font-weight:950;letter-spacing:.06em;font-size:12px;color:rgba(234,241,255,.88);font-family:var(--mono)">Journal</div>
            <div class="list" id="log" style="margin-top:10px;max-height:240px;overflow:auto"></div>

          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<input id="fileInput" type="file" accept=".json,application/json"/>

<script>
(() => {
  /* =========================
     NYXO • Obsidian Monitor
     - Mobile-first
     - Fullscreen monitor overlay
     - Imports BOTH schemas:
       A) UI schema: {zones, entities, links}
       B) Master schema: {sources, entities(labels, fields[]), relations}
     ========================= */

  const canvas = document.getElementById("stage");
  const ctx = canvas.getContext("2d", { alpha:true });

  const ui = {
    homeBtn: document.getElementById("homeBtn"),
    modeBtn: document.getElementById("modeBtn"),
    regenBtn: document.getElementById("regenBtn"),
    labels: document.getElementById("labels"),
    overlap: document.getElementById("overlap"),
    reveal: document.getElementById("reveal"),
    zoom: document.getElementById("zoom"),
    filter: document.getElementById("filter"),

    drawerWrap: document.getElementById("drawerWrap"),
    drawerToggle: document.getElementById("drawerToggle"),
    drawerFab: document.getElementById("drawerFab"),
    scrim: document.getElementById("scrim"),
    closeDrawer: document.getElementById("closeDrawer"),

    importFileBtn: document.getElementById("importFileBtn"),
    fileInput: document.getElementById("fileInput"),

    kEnt: document.getElementById("kEnt"),
    kRel: document.getElementById("kRel"),
    kZon: document.getElementById("kZon"),
    kFld: document.getElementById("kFld"),
    kConf: document.getElementById("kConf"),
    confBar: document.getElementById("confBar"),

    typeChips: document.getElementById("typeChips"),
    riskChips: document.getElementById("riskChips"),
    entityList: document.getElementById("entityList"),
    details: document.getElementById("details"),

    jsonArea: document.getElementById("jsonArea"),
    applyBtn: document.getElementById("applyBtn"),
    exportBtn: document.getElementById("exportBtn"),
    exportMasterBtn: document.getElementById("exportMasterBtn"),
    sampleBtn: document.getElementById("sampleBtn"),
    clearBtn: document.getElementById("clearBtn"),

    log: document.getElementById("log"),
  };

  const MODE = { ZONES:"ZONES", GRAPH:"GRAPH" };
  let mode = MODE.ZONES;
  let drawerOpen = false;
  let selectedId = null;

  const db = {
    meta: { dataset:"nyxo-monitor" },
    sources: [],
    zones: [],
    entities: [],
    links: [],
  };

  /* ---------- utils ---------- */
  const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
  const rnd=(a,b)=>a+Math.random()*(b-a);
  const irnd=(a,b)=>Math.floor(rnd(a,b+1));
  const safe=(x,fb)=> (x===undefined||x===null||x==="") ? fb : x;

  function hashStr(s){
    s = String(s||"");
    let h = 2166136261;
    for(let i=0;i<s.length;i++){
      h ^= s.charCodeAt(i);
      h = Math.imul(h, 16777619);
    }
    return (h>>>0);
  }
  function h2f(h){ return (h % 10000) / 10000; }

  function colorForLayer(layer){
    if(layer==="ASTRO") return "rgba(134,251,214,.95)";
    if(layer==="TAROT") return "rgba(215,168,255,.95)";
    return "rgba(234,241,255,.80)";
  }
  function colorForConfidence(c){
    if(c>=0.90) return "rgba(134,251,214,.95)";
    if(c>=0.70) return "rgba(255,211,106,.95)";
    return "rgba(255,106,139,.95)";
  }

  function log(msg){
    const el=document.createElement("div");
    el.className="item";
    el.innerHTML = `
      <div class="badge" style="background:${colorForConfidence(0.85)}"></div>
      <div class="meta">
        <div class="id">${escapeHtml(new Date().toLocaleTimeString())}</div>
        <div class="sub">${escapeHtml(msg)}</div>
      </div>`;
    ui.log.prepend(el);
    while(ui.log.children.length>60) ui.log.removeChild(ui.log.lastChild);
  }

  function escapeHtml(s){
    return String(s ?? "")
      .replaceAll("&","&amp;").replaceAll("<","&lt;")
      .replaceAll(">","&gt;").replaceAll('"',"&quot;").replaceAll("'","&#039;");
  }

  function fitCanvas(){
    const dpr = Math.max(1, Math.min(2, window.devicePixelRatio || 1));
    canvas.width = Math.floor(innerWidth * dpr);
    canvas.height = Math.floor(innerHeight * dpr);
    canvas.style.width = innerWidth+"px";
    canvas.style.height = innerHeight+"px";
    ctx.setTransform(dpr,0,0,dpr,0,0);
  }
  addEventListener("resize", fitCanvas, {passive:true});

  /* ---------- normalization: UI schema ---------- */
  function normZone(z){
    const id = safe(z.id, "Z-"+Math.random().toString(16).slice(2,8).toUpperCase());
    let x = Number.isFinite(+z.x) ? +z.x : (h2f(hashStr(id))*900-450);
    let y = Number.isFinite(+z.y) ? +z.y : (h2f(hashStr(id+"y"))*620-310);
    const layer = safe(z.layer, "NEUTRE");
    const opacity = Number.isFinite(+z.opacity) ? +z.opacity : (layer==="NEUTRE"?0.18:0.24);
    return { id, x, y, layer, opacity, sizeKm2: 1, label: z.label || id };
  }

  function normEntity(e){
    const id = safe(e.id, "E-"+Math.random().toString(16).slice(2,8).toUpperCase());
    const type = safe(e.type, "place");
    const subtype = e.subtype || null;
    const layer = safe(e.layer, "NEUTRE");
    const zone = e.zone || e.zoneId || null;
    const tags = Array.isArray(e.tags) ? e.tags : [];
    const confidence = Number.isFinite(+e.confidence) ? clamp(+e.confidence,0,1) : 0.65;

    // display schema (UI)
    const disp = e.display || {};
    const display = {
      masked_name: safe(disp.masked_name, e.name || e.title || id),
      real_name: safe(disp.real_name, null),
    };

    // keep BOTH: fieldsObj + claims[]
    const fieldsObj = (e.fields && typeof e.fields==="object" && !Array.isArray(e.fields)) ? e.fields : {};
    const claims = Array.isArray(e.claims) ? e.claims : (Array.isArray(e.fields)? e.fields : []);

    // positions (abstract)
    const hx = h2f(hashStr(id+"x"));
    const hy = h2f(hashStr(id+"y"));
    const x = Number.isFinite(+e.x) ? +e.x : (hx*900-450);
    const y = Number.isFinite(+e.y) ? +e.y : (hy*620-310);
    const vx = Number.isFinite(+e.vx) ? +e.vx : 0;
    const vy = Number.isFinite(+e.vy) ? +e.vy : 0;

    return { id,type,subtype,layer,zone,tags,confidence,display,fields:fieldsObj,claims,x,y,vx,vy };
  }

  function normLink(l){
    const id = safe(l.id, "L-"+Math.random().toString(16).slice(2,8).toUpperCase());
    const type = safe(l.type, "connect");
    const from = l.from, to = l.to;
    const confidence = Number.isFinite(+l.confidence) ? clamp(+l.confidence,0,1) : 0.6;
    const weight = Number.isFinite(+l.weight) ? +l.weight : 1.0;
    const fields = (l.fields && typeof l.fields==="object") ? l.fields : {};
    return { id,type,from,to,confidence,weight,fields };
  }

  /* ---------- adaptation: Master schema -> UI schema ---------- */
  function masterToUI(raw){
    const out = {
      meta: raw.meta || {dataset:"master-import"},
      sources: Array.isArray(raw.sources) ? raw.sources : [],
      zones: [],
      entities: [],
      links: []
    };

    const zonesMap = new Map();
    // Zones explicit
    if(Array.isArray(raw.zones)){
      for(const z of raw.zones){
        const nz = normZone({
          id: z.id,
          label: z.label || z.id,
          layer: z.layer || "NEUTRE",
          opacity: z.opacity
        });
        zonesMap.set(nz.id, nz);
      }
    }

    // helper: ensure zone exists (abstract placement, deterministic)
    function ensureZone(zoneId, layer="NEUTRE"){
      if(!zoneId) zoneId = "zone_1km2_unknown_01";
      if(zonesMap.has(zoneId)) return zoneId;
      const h = hashStr(zoneId);
      const baseX = (h2f(h) * 900 - 450);
      const baseY = (h2f(hashStr(zoneId+"y")) * 620 - 310);
      const nz = normZone({ id: zoneId, x: baseX, y: baseY, layer, opacity: (layer==="NEUTRE"?0.18:0.24), label: zoneId });
      zonesMap.set(zoneId, nz);
      return zoneId;
    }

    // entities
    const ents = Array.isArray(raw.entities) ? raw.entities : [];
    for(const e of ents){
      const labels = e.labels || {};
      const claims = Array.isArray(e.fields) ? e.fields : []; // master fields[] = claims
      // compute entity confidence: average of claim confidences if present
      let conf = 0.65;
      if(claims.length){
        const nums = claims.map(c => Number.isFinite(+c.confidence) ? clamp(+c.confidence,0,1) : null).filter(v=>v!==null);
        if(nums.length) conf = nums.reduce((a,b)=>a+b,0)/nums.length;
      }

      // map claims[] -> fieldsObj (preserve evidence)
      const fieldsObj = {};
      for(const c of claims){
        const k = safe(c.key, null);
        if(!k) continue;
        fieldsObj[k] = {
          value: c.value,
          confidence: Number.isFinite(+c.confidence) ? clamp(+c.confidence,0,1) : null,
          evidence: Array.isArray(c.evidence) ? c.evidence : []
        };
      }

      // zones: use first zoneHint if any
      const zoneHints = Array.isArray(e.zoneHints) ? e.zoneHints : [];
      const zoneId = ensureZone(zoneHints[0], "NEUTRE");

      out.entities.push(normEntity({
        id: e.id,
        type: e.type || "place",
        subtype: e.subtype || null,
        layer: "NEUTRE", // (tu peux tagger ASTRO/TAROT ensuite dans l’UI)
        zone: zoneId,
        confidence: +conf.toFixed(2),
        display: {
          masked_name: safe(labels.public, e.id),
          real_name: safe(labels.real, null)
        },
        tags: Array.isArray(e.tags) ? e.tags : [],
        claims: claims,     // keep raw claims
        fields: fieldsObj,  // normalized fields object
      }));
    }

    // relations -> links
    const rels = Array.isArray(raw.relations) ? raw.relations : (Array.isArray(raw.links) ? raw.links : []);
    for(const r of rels){
      out.links.push(normLink({
        id: r.id,
        type: r.type || "connect",
        from: r.from,
        to: r.to,
        confidence: Number.isFinite(+r.confidence) ? r.confidence : 0.6,
        weight: r.weight || 1,
        fields: {
          evidence: Array.isArray(r.evidence) ? r.evidence : (Array.isArray(r.fields?.evidence) ? r.fields.evidence : []),
          source: r.sourceId || null
        }
      }));
    }

    out.zones = [...zonesMap.values()];
    return out;
  }

  function applyRawFlexible(raw){
    // Detect schema:
    // UI schema: raw.entities + raw.links
    // Master schema: raw.entities + raw.relations + labels/fields[]
    let pack = null;

    if(raw && Array.isArray(raw.entities) && Array.isArray(raw.links)){
      pack = { meta: raw.meta, sources: raw.sources||[], zones: raw.zones||[], entities: raw.entities, links: raw.links };
      log("Import: schéma UI (entities + links)");
    } else if(raw && Array.isArray(raw.entities) && Array.isArray(raw.relations)){
      pack = masterToUI(raw);
      log("Import: schéma MASTER (entities + relations) → adapt UI");
    } else if(raw && Array.isArray(raw.entities) && !raw.links && !raw.relations){
      // fallback: try to interpret "relations" named differently
      if(Array.isArray(raw.relations)) { pack = masterToUI(raw); }
      else {
        // last resort: treat as UI and empty links
        pack = { meta: raw.meta, sources: raw.sources||[], zones: raw.zones||[], entities: raw.entities, links: [] };
      }
      log("Import: schéma partiel (fallback)");
    } else {
      pack = { meta: raw.meta||db.meta, sources: raw.sources||[], zones: raw.zones||[], entities: raw.entities||[], links: raw.links||[] };
      log("Import: schéma inconnu (fallback)");
    }

    // apply
    db.meta = pack.meta || db.meta || {dataset:"nyxo-monitor"};
    db.sources = Array.isArray(pack.sources) ? pack.sources : [];
    db.zones = (Array.isArray(pack.zones)? pack.zones: []).map(normZone);
    db.entities = (Array.isArray(pack.entities)? pack.entities: []).map(normEntity);
    db.links = (Array.isArray(pack.links)? pack.links: []).map(normLink);

    ensureZones();
    seedPositionsToZones();
    selectedId = null;

    syncTextarea();
    renderAll();
  }

  function ensureZones(){
    const map = new Map(db.zones.map(z=>[z.id,z]));
    // ensure each entity has a zone
    for(const e of db.entities){
      if(e.zone && map.has(e.zone)) continue;
      // create missing zone deterministically based on e.zone or e.id
      const zid = e.zone || "zone_1km2_unknown_01";
      if(!map.has(zid)){
        const h = hashStr(zid);
        const nz = normZone({ id:zid, x:(h2f(h)*900-450), y:(h2f(hashStr(zid+"y"))*620-310), layer:"NEUTRE", opacity:0.18, label:zid });
        map.set(zid,nz);
      }
      e.zone = zid;
    }
    db.zones = [...map.values()];
  }

  function seedPositionsToZones(){
    const zmap = new Map(db.zones.map(z=>[z.id,z]));
    for(const e of db.entities){
      const z = zmap.get(e.zone);
      if(!z) continue;
      const h = hashStr(e.id);
      const a = h2f(h) * Math.PI*2;
      const r = 18 + h2f(hashStr(e.id+"r")) * 78;
      e.x = z.x + Math.cos(a)*r + rnd(-8,8);
      e.y = z.y + Math.sin(a)*r + rnd(-8,8);
      e.vx = 0; e.vy = 0;
    }
  }

  /* ---------- selection + display ---------- */
  function displayName(e){
    const reveal = !!ui.reveal.checked;
    const masked = safe(e.display?.masked_name, e.id);
    const real = safe(e.display?.real_name, null);
    return (reveal && real) ? real : masked;
  }

  function parseFilter(txt){
    const q = (txt||"").trim();
    const out = { text:"", type:null, tag:null, zone:null, confOp:null, confVal:null };
    if(!q) return out;

    const parts = q.split(/\s+/).filter(Boolean);
    for(const p of parts){
      const m = p.match(/^([a-zA-Z_]+)\:(.+)$/);
      if(m){
        const k=m[1].toLowerCase(), v=m[2];
        if(k==="type") out.type=v.toLowerCase();
        else if(k==="tag") out.tag=v.toLowerCase();
        else if(k==="zone") out.zone=v;
        else if(k==="conf" || k==="confidence"){
          const mm=v.match(/^(>=|<=|>|<|=)?\s*([0-9]*\.?[0-9]+)$/);
          if(mm){ out.confOp=mm[1]||">="; out.confVal=parseFloat(mm[2]); }
        }
      } else {
        out.text += (out.text?" ":"") + p;
      }
    }
    out.text = out.text.toLowerCase();
    return out;
  }

  function filterEntities(){
    const f = parseFilter(ui.filter.value);
    const reveal = !!ui.reveal.checked;
    return db.entities.filter(e=>{
      if(f.type && String(e.type||"").toLowerCase()!==f.type) return false;
      if(f.zone && String(e.zone||"")!==f.zone) return false;
      if(f.tag){
        const tags = (e.tags||[]).map(t=>String(t).toLowerCase());
        if(!tags.includes(f.tag)) return false;
      }
      if(f.confVal!=null){
        const c = Number.isFinite(+e.confidence) ? +e.confidence : 0;
        const op = f.confOp || ">=";
        if(op===">=" && !(c>=f.confVal)) return false;
        if(op===">"  && !(c> f.confVal)) return false;
        if(op==="<= "&& !(c<=f.confVal)) return false;
        if(op==="<=" && !(c<=f.confVal)) return false;
        if(op==="<"  && !(c< f.confVal)) return false;
        if(op==="="  && !(Math.abs(c-f.confVal)<1e-9)) return false;
      }
      if(f.text){
        // IMPORTANT: when reveal is off, we only match masked + ids (not real name)
        const hay = (reveal ? (displayName(e)+" "+e.id) : ((e.display?.masked_name||"")+" "+e.id)).toLowerCase();
        if(!hay.includes(f.text)) return false;
      }
      return true;
    });
  }

  function renderAll(){
    renderMonitor();
    renderDetails();
    // force redraw differences obvious
    drawOnce();
  }

  function renderMonitor(){
    const ents = filterEntities();
    ui.kEnt.textContent = String(ents.length);
    ui.kRel.textContent = String(db.links.length);
    ui.kZon.textContent = String(db.zones.length);

    // count claims
    let claimCount=0;
    for(const e of ents){
      if(Array.isArray(e.claims) && e.claims.length) claimCount+=e.claims.length;
      else claimCount+=Object.keys(e.fields||{}).length;
    }
    ui.kFld.textContent = String(claimCount);

    // avg confidence
    const avg = ents.length ? ents.reduce((a,e)=>a+(Number.isFinite(+e.confidence)?+e.confidence:0.65),0)/ents.length : 0;
    ui.kConf.textContent = avg.toFixed(2);
    ui.confBar.style.width = (clamp(avg,0,1)*100).toFixed(1)+"%";

    // risk chips
    ui.riskChips.innerHTML = "";
    const bins = [
      {label:">=0.90", cls:"good", pred:(c)=>c>=0.90},
      {label:"0.70–0.89", cls:"warn", pred:(c)=>c>=0.70 && c<0.90},
      {label:"<0.70", cls:"bad", pred:(c)=>c<0.70},
    ];
    for(const b of bins){
      const n = ents.filter(e=>b.pred(+e.confidence||0)).length;
      const c = document.createElement("div");
      c.className = "chip "+b.cls;
      c.textContent = `${b.label} • ${n}`;
      ui.riskChips.appendChild(c);
    }

    // type chips
    ui.typeChips.innerHTML = "";
    const byType = {};
    for(const e of ents){
      const t = String(e.type||"unknown");
      byType[t]=(byType[t]||0)+1;
    }
    Object.keys(byType).sort((a,b)=>byType[b]-byType[a]).slice(0,6).forEach(t=>{
      const c = document.createElement("div");
      c.className="chip";
      c.textContent = `${t}:${byType[t]}`;
      ui.typeChips.appendChild(c);
    });

    // list
    ui.entityList.innerHTML = "";
    const sorted = ents.slice().sort((a,b)=> (b.confidence||0)-(a.confidence||0));
    for(const e of sorted){
      const it=document.createElement("div");
      it.className="item";
      const conf = Number.isFinite(+e.confidence)?+e.confidence:0.65;
      it.innerHTML = `
        <div class="badge" style="background:${colorForConfidence(conf)}"></div>
        <div class="meta">
          <div class="id">${escapeHtml(displayName(e))}</div>
          <div class="sub">${escapeHtml(`${e.type}${e.subtype?(" • "+e.subtype):""} • ${e.zone||"—"}`)}</div>
        </div>
        <div class="right">${escapeHtml(conf.toFixed(2))}</div>
      `;
      it.addEventListener("click",()=>{ selectedId=e.id; renderDetails(); drawOnce(); });
      ui.entityList.appendChild(it);
    }
  }

  function linkStats(entityId){
    const out = {in:[], out:[]};
    for(const l of db.links){
      if(l.from===entityId) out.out.push(l);
      if(l.to===entityId) out.in.push(l);
    }
    return out;
  }

  function renderDetails(){
    ui.details.innerHTML="";
    const e = db.entities.find(x=>x.id===selectedId);

    if(!e){
      const box=document.createElement("div");
      box.className="item";
      box.innerHTML=`
        <div class="meta">
          <div class="id">Sélection</div>
          <div class="sub">Clique une entité dans la liste (ou dans le canvas).</div>
        </div>`;
      ui.details.appendChild(box);
      return;
    }

    const conf = Number.isFinite(+e.confidence)?+e.confidence:0.65;
    const stats = linkStats(e.id);

    // header + quick edit layer
    const head=document.createElement("div");
    head.className="item";
    head.innerHTML=`
      <div class="badge" style="background:${colorForConfidence(conf)}"></div>
      <div class="meta">
        <div class="id">${escapeHtml(displayName(e))}</div>
        <div class="sub">${escapeHtml(`${e.type}${e.subtype?(" • "+e.subtype):""} • zone:${e.zone||"—"} • liens: ${stats.in.length}← ${stats.out.length}→`)}</div>
      </div>
      <div class="right">${escapeHtml(conf.toFixed(2))}</div>
    `;
    ui.details.appendChild(head);

    // layer picker (ASTRO/TAROT/NEUTRE)
    const layerBox=document.createElement("div");
    layerBox.className="item";
    layerBox.innerHTML=`
      <div class="meta">
        <div class="id">Couche</div>
        <div class="sub">Taggage rapide (sans renommer quoi que ce soit).</div>
        <div style="margin-top:8px;display:flex;gap:8px;flex-wrap:wrap">
          <button class="btn" data-layer="NEUTRE">NEUTRE</button>
          <button class="btn" data-layer="ASTRO">ASTRO</button>
          <button class="btn" data-layer="TAROT">TAROT</button>
        </div>
      </div>
    `;
    layerBox.querySelectorAll("button[data-layer]").forEach(b=>{
      b.addEventListener("click",()=>{
        e.layer = b.getAttribute("data-layer");
        // also set masked name prefix if user wants labels to help
        log(`Couche: ${e.id} -> ${e.layer}`);
        renderMonitor(); renderDetails(); drawOnce(); syncTextarea();
      });
    });
    ui.details.appendChild(layerBox);

    // tags
    const tags=document.createElement("div");
    tags.className="item";
    const tagChips = (e.tags||[]).slice(0,22).map(t=>`<span class="chip">${escapeHtml(t)}</span>`).join("");
    tags.innerHTML=`
      <div class="meta">
        <div class="id">Tags</div>
        <div class="sub">—</div>
        <div class="chipRow" style="margin-top:8px">${tagChips || `<span class="chip">—</span>`}</div>
      </div>`;
    ui.details.appendChild(tags);

    // claims / fields (détails “NSA” = key + value + confidence + evidence)
    const claims = Array.isArray(e.claims) && e.claims.length
      ? e.claims
      : Object.entries(e.fields||{}).map(([k,v])=>({key:k, value:v?.value ?? v, confidence:v?.confidence ?? null, evidence:v?.evidence ?? []}));

    const fieldsBox=document.createElement("div");
    fieldsBox.className="item";
    fieldsBox.innerHTML=`
      <div class="meta" style="width:100%">
        <div class="id">Détails (claims)</div>
        <div class="sub">${escapeHtml(`${claims.length} champ(s) • les preuves/evidence restent visibles`)}</div>
        <div style="margin-top:10px;display:grid;gap:8px">
          ${claims.slice(0,120).map(c=>{
            const cc = Number.isFinite(+c.confidence) ? clamp(+c.confidence,0,1) : null;
            const badge = cc==null ? "rgba(234,241,255,.45)" : colorForConfidence(cc);
            const v = (typeof c.value==="object") ? JSON.stringify(c.value) : String(c.value ?? "");
            const ev = Array.isArray(c.evidence) ? c.evidence : [];
            const evHtml = ev.slice(0,6).map(evv=>{
              const sid = safe(evv.sourceId, "src");
              const ref = safe(evv.ref, "");
              return `<div class="sub" style="font-family:var(--mono);opacity:.9">• ${escapeHtml(sid)} ${escapeHtml(ref)}</div>`;
            }).join("");
            return `
              <div style="border:1px solid rgba(255,255,255,.10);background:rgba(0,0,0,.12);border-radius:14px;padding:10px">
                <div style="display:flex;align-items:flex-start;gap:10px">
                  <div style="width:10px;height:10px;border-radius:50%;margin-top:4px;background:${badge}"></div>
                  <div style="flex:1;min-width:0">
                    <div style="font-family:var(--mono);font-weight:950;font-size:12px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis">${escapeHtml(c.key||"")}</div>
                    <div class="sub" style="margin-top:4px;white-space:pre-wrap;word-break:break-word">${escapeHtml(v)}</div>
                    ${evHtml ? `<div style="margin-top:8px;opacity:.9">${evHtml}</div>` : ``}
                  </div>
                  <div style="font-family:var(--mono);font-size:11px;opacity:.75">${cc==null?"—":cc.toFixed(2)}</div>
                </div>
              </div>`;
          }).join("")}
          ${claims.length>120 ? `<div class="hint">Affichage limité à 120 champs pour garder l’UI fluide (les données restent dans le JSON).</div>` : ``}
        </div>
      </div>`;
    ui.details.appendChild(fieldsBox);

    // related links preview
    const relBox=document.createElement("div");
    relBox.className="item";
    const related = [...stats.out.slice(0,12).map(l=>({dir:"→", l})), ...stats.in.slice(0,12).map(l=>({dir:"←", l}))];
    relBox.innerHTML=`
      <div class="meta" style="width:100%">
        <div class="id">Liens (aperçu)</div>
        <div class="sub">${escapeHtml(`${stats.in.length} entrants • ${stats.out.length} sortants`)}</div>
        <div style="margin-top:10px;display:grid;gap:8px">
          ${related.length ? related.map(({dir,l})=>{
            const otherId = dir==="→" ? l.to : l.from;
            const other = db.entities.find(x=>x.id===otherId);
            const name = other ? displayName(other) : otherId;
            const c = Number.isFinite(+l.confidence)?+l.confidence:0.6;
            const ev = Array.isArray(l.fields?.evidence) ? l.fields.evidence : [];
            return `
              <div style="border:1px solid rgba(255,255,255,.10);background:rgba(0,0,0,.10);border-radius:14px;padding:10px">
                <div style="display:flex;gap:10px;align-items:flex-start">
                  <div style="width:10px;height:10px;border-radius:50%;margin-top:4px;background:${colorForConfidence(c)}"></div>
                  <div style="flex:1;min-width:0">
                    <div style="font-family:var(--mono);font-weight:950;font-size:12px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis">${escapeHtml(`${dir} ${l.type} • ${name}`)}</div>
                    ${ev.length ? `<div class="sub" style="margin-top:4px;font-family:var(--mono)">• ${escapeHtml(safe(ev[0]?.sourceId,"src"))} ${escapeHtml(safe(ev[0]?.ref,""))}</div>` : `<div class="sub" style="margin-top:4px">—</div>`}
                  </div>
                  <div style="font-family:var(--mono);font-size:11px;opacity:.75">${escapeHtml(c.toFixed(2))}</div>
                </div>
              </div>`;
          }).join("") : `<div class="hint">Aucun lien dans le dataset importé (ou pas encore typé).</div>`}
        </div>
      </div>`;
    ui.details.appendChild(relBox);

    // expose selected into textarea (patch-friendly)
    // we keep full dataset there; editing is via Apply
  }

  /* ---------- JSON IO ---------- */
  function syncTextarea(){
    // Export UI schema only (internal)
    const payload = {
      meta: db.meta || {dataset:"nyxo-monitor"},
      sources: db.sources || [],
      zones: db.zones.map(z=>({id:z.id,x:+z.x.toFixed(2),y:+z.y.toFixed(2),sizeKm2:1,layer:z.layer,opacity:z.opacity,label:z.label||z.id})),
      entities: db.entities.map(e=>({
        id:e.id, type:e.type, subtype:e.subtype,
        layer:e.layer, zone:e.zone,
        confidence:+(e.confidence||0.65).toFixed(2),
        display: e.display,
        tags: e.tags||[],
        // keep both:
        fields: e.fields||{},
        claims: e.claims||[],
        x:+e.x.toFixed(2), y:+e.y.toFixed(2)
      })),
      links: db.links.map(l=>({id:l.id,type:l.type,from:l.from,to:l.to,confidence:+(l.confidence||0.6).toFixed(2),weight:l.weight||1,fields:l.fields||{}}))
    };
    ui.jsonArea.value = JSON.stringify(payload, null, 2);
  }

  function exportUI(){
    syncTextarea();
    const blob = new Blob([ui.jsonArea.value], {type:"application/json"});
    const a=document.createElement("a");
    a.href=URL.createObjectURL(blob);
    a.download="nyxo-portal-ui.json";
    a.click();
    setTimeout(()=>URL.revokeObjectURL(a.href),800);
    log("Export: UI JSON");
  }

  function exportMaster(){
    // Convert back (best effort): labels + fields[]; links -> relations
    const master = {
      meta: Object.assign({schemaVersion:"0.2.0", scope:"bruxelles"}, db.meta||{}),
      sources: db.sources||[],
      entities: db.entities.map(e=>{
        // rebuild claims[] if missing from fields
        const claims = Array.isArray(e.claims) && e.claims.length ? e.claims : Object.entries(e.fields||{}).map(([k,v])=>({
          key:k,
          value: (v && typeof v==="object" && "value" in v) ? v.value : v,
          confidence: (v && typeof v==="object" && "confidence" in v) ? v.confidence : null,
          evidence: (v && typeof v==="object" && Array.isArray(v.evidence)) ? v.evidence : []
        }));
        return {
          id: e.id,
          type: e.type,
          subtype: e.subtype || null,
          labels: { public: e.display?.masked_name || e.id, real: e.display?.real_name || null },
          privacy: { realNamePolicy: "hiddenByDefault" },
          tags: e.tags||[],
          zoneHints: e.zone ? [e.zone] : [],
          fields: claims
        };
      }),
      relations: db.links.map(l=>({
        id: l.id,
        type: l.type,
        from: l.from,
        to: l.to,
        confidence: l.confidence,
        evidence: Array.isArray(l.fields?.evidence) ? l.fields.evidence : []
      })),
      zones: db.zones.map(z=>({id:z.id,label:z.label||z.id,layer:z.layer,opacity:z.opacity}))
    };

    const blob = new Blob([JSON.stringify(master,null,2)], {type:"application/json"});
    const a=document.createElement("a");
    a.href=URL.createObjectURL(blob);
    a.download="nyxo-portal-master.json";
    a.click();
    setTimeout(()=>URL.revokeObjectURL(a.href),800);
    log("Export: MASTER JSON (best effort)");
  }

  /* ---------- monitor open/close ---------- */
  function setDrawer(open){
    drawerOpen = !!open;
    ui.drawerWrap.classList.toggle("open", drawerOpen);
    ui.drawerWrap.setAttribute("aria-hidden", drawerOpen ? "false":"true");
    ui.drawerToggle.textContent = drawerOpen ? "Monitor ◂" : "Monitor ▸";
    canvas.style.pointerEvents = drawerOpen ? "none" : "auto";
    if(drawerOpen) syncTextarea();
    drawOnce();
  }

  ui.drawerToggle.addEventListener("click",()=>setDrawer(!drawerOpen));
  ui.drawerFab.addEventListener("click",()=>setDrawer(true));
  ui.scrim.addEventListener("click",()=>setDrawer(false));
  ui.closeDrawer.addEventListener("click",()=>setDrawer(false));
  addEventListener("keydown",(e)=>{ if(e.key==="Escape") setDrawer(false); });

  /* ---------- actions ---------- */
  ui.modeBtn.addEventListener("click",()=>{
    mode = (mode===MODE.ZONES) ? MODE.GRAPH : MODE.ZONES;
    ui.modeBtn.textContent = mode;
    log(`Mode: ${mode}`);
    drawOnce();
  });

  ui.regenBtn.addEventListener("click",()=>{
    seedPositionsToZones();
    log("Positions: régénérées");
    drawOnce();
    syncTextarea();
  });

  ui.filter.addEventListener("input",()=>{ renderMonitor(); drawOnce(); });
  ui.labels.addEventListener("change",()=>drawOnce());
  ui.overlap.addEventListener("change",()=>drawOnce());
  ui.zoom.addEventListener("input",()=>drawOnce());

  ui.reveal.addEventListener("change",()=>{
    renderMonitor();
    renderDetails();
    log(ui.reveal.checked ? "Noms réels: ON" : "Noms réels: OFF");
    drawOnce();
  });

  ui.homeBtn.addEventListener("click",()=>{ view.x=0; view.y=0; view.z=1; drawOnce(); });

  ui.applyBtn.addEventListener("click",()=>{
    try{
      const raw = JSON.parse(ui.jsonArea.value||"{}");
      applyRawFlexible(raw);
      setDrawer(false);
      log(`Appliqué: entities=${db.entities.length} links=${db.links.length} zones=${db.zones.length}`);
    }catch(err){
      log("Appliquer: JSON invalide");
    }
  });

  ui.clearBtn.addEventListener("click",()=>{
    db.meta={dataset:"nyxo-monitor"};
    db.sources=[];
    db.entities=[];
    db.links=[];
    db.zones=[normZone({id:"zone_1km2_unknown_01",layer:"NEUTRE",opacity:0.18})];
    selectedId=null;
    syncTextarea();
    renderAll();
    log("Dataset vidé");
  });

  ui.exportBtn.addEventListener("click", exportUI);
  ui.exportMasterBtn.addEventListener("click", exportMaster);

  ui.sampleBtn.addEventListener("click",()=>{
    const zones=[];
    for(let i=0;i<12;i++){
      const layer=(i%3===0)?"ASTRO":(i%3===1)?"TAROT":"NEUTRE";
      zones.push({id:"Z-"+String(i+1).padStart(2,"0"),layer,opacity:layer==="NEUTRE"?0.18:0.24});
    }
    const entities=[];
    for(let i=0;i<54;i++){
      const type=(i%6===0)?"network":(i%6===1)?"org":(i%6===2)?"place":(i%6===3)?"service":(i%6===4)?"artifact":"hub";
      const layer=(i%5===0)?"ASTRO":(i%5===1)?"TAROT":"NEUTRE";
      const zone=zones[irnd(0,zones.length-1)].id;
      const conf=(Math.random()<0.65)?rnd(0.90,1.00):(Math.random()<0.6?rnd(0.70,0.89):rnd(0.45,0.69));
      entities.push({
        id:"E-"+String(i+1).padStart(3,"0"),
        type, layer, zone,
        confidence:+conf.toFixed(2),
        display:{ masked_name:`${layer}:${type.toUpperCase()}:${String(i+1).padStart(3,"0")}`, real_name:`REAL:${type.toUpperCase()}:${String(i+1).padStart(3,"0")}` },
        tags:[type,layer.toLowerCase(),"bruxelles"],
        claims:[
          {key:"note", value:"Exemple (données synthétiques).", confidence:+conf.toFixed(2), evidence:[{sourceId:"demo",ref:"seed"}]},
          {key:"contact", value:"—", confidence:0.5, evidence:[]}
        ]
      });
    }
    const links=[];
    for(let i=0;i<120;i++){
      const a=entities[irnd(0,entities.length-1)].id;
      const b=entities[irnd(0,entities.length-1)].id;
      if(a===b) continue;
      links.push({id:"L-"+String(i+1).padStart(3,"0"),type:"connect",from:a,to:b,confidence:+rnd(0.55,1.0).toFixed(2),weight:+rnd(0.6,1.3).toFixed(2),fields:{evidence:[{sourceId:"demo",ref:"synthetic"}]}})
    }
    applyRawFlexible({meta:{dataset:"nyxo-sample"},zones,entities,links});
    log("Exemple chargé");
  });

  /* file import */
  ui.importFileBtn.addEventListener("click",()=>ui.fileInput.click());
  ui.fileInput.addEventListener("change",async ()=>{
    const file=ui.fileInput.files && ui.fileInput.files[0];
    if(!file) return;
    try{
      const txt = await file.text();
      const raw = JSON.parse(txt);
      ui.jsonArea.value = JSON.stringify(raw, null, 2);
      applyRawFlexible(raw);
      log(`Fichier importé: ${file.name} • entities=${db.entities.length}`);
    }catch(err){
      log("Import fichier: JSON invalide");
    } finally {
      ui.fileInput.value="";
    }
  });

  /* ---------- canvas interaction + layout ---------- */
  const view = { x:0, y:0, z:1 };
  let dragging=false, dragStart=null, viewStart=null;

  canvas.addEventListener("pointerdown",(ev)=>{
    if(drawerOpen) return;
    canvas.setPointerCapture(ev.pointerId);
    const p = screenToWorld(ev.clientX, ev.clientY);
    const hit = hitTest(p.x,p.y);
    if(hit){
      selectedId = hit.id;
      renderDetails();
      drawOnce();
      return;
    }
    dragging=true;
    dragStart={x:ev.clientX,y:ev.clientY};
    viewStart={x:view.x,y:view.y};
  }, {passive:false});

  canvas.addEventListener("pointermove",(ev)=>{
    if(!dragging) return;
    const dx=ev.clientX-dragStart.x;
    const dy=ev.clientY-dragStart.y;
    view.x = viewStart.x + dx/view.z;
    view.y = viewStart.y + dy/view.z;
    drawOnce();
  }, {passive:true});

  canvas.addEventListener("pointerup",()=>{ dragging=false; }, {passive:true});

  canvas.addEventListener("wheel",(ev)=>{
    if(drawerOpen) return;
    ev.preventDefault();
    const dz = (ev.deltaY>0) ? 0.92 : 1.08;
    const nz = clamp(view.z*dz, 0.55, 2.2);
    view.z = nz;
    drawOnce();
  }, {passive:false});

  function screenToWorld(sx,sy){
    const cx = innerWidth/2, cy = innerHeight/2;
    const x = (sx - cx)/view.z - view.x;
    const y = (sy - cy)/view.z - view.y;
    return {x,y};
  }
  function worldToScreen(x,y){
    const cx=innerWidth/2, cy=innerHeight/2;
    return { x: cx + (x+view.x)*view.z, y: cy + (y+view.y)*view.z };
  }

  function hitTest(wx,wy){
    const ents = filterEntities();
    let best=null, bestD=1e9;
    for(const e of ents){
      const dx=wx-e.x, dy=wy-e.y;
      const d=Math.sqrt(dx*dx+dy*dy);
      const r = (e.id===selectedId)?13:10;
      if(d<r && d<bestD){ best=e; bestD=d; }
    }
    return best;
  }

  // Force-ish graph stepping (lightweight)
  function stepGraph(dt){
    const ents = filterEntities();
    if(ents.length>420) return; // keep safe
    const index = new Map(ents.map((e,i)=>[e.id,i]));
    const links = db.links.filter(l=>index.has(l.from)&&index.has(l.to));

    // mild pull to zone centers to preserve “zones” backbone
    const zmap = new Map(db.zones.map(z=>[z.id,z]));
    for(const e of ents){
      const z = zmap.get(e.zone);
      if(!z) continue;
      e.vx += (z.x - e.x)*0.02;
      e.vy += (z.y - e.y)*0.02;
    }

    // repulsion
    for(let i=0;i<ents.length;i++){
      const a=ents[i];
      for(let j=i+1;j<ents.length;j++){
        const b=ents[j];
        const dx=a.x-b.x, dy=a.y-b.y;
        let d2=dx*dx+dy*dy;
        if(d2<1) d2=1;
        const f= 2200 / d2;
        const fx=dx*Math.min(2.2,f)/Math.sqrt(d2);
        const fy=dy*Math.min(2.2,f)/Math.sqrt(d2);
        a.vx += fx; a.vy += fy;
        b.vx -= fx; b.vy -= fy;
      }
    }

    // springs
    for(const l of links){
      const a = ents[index.get(l.from)];
      const b = ents[index.get(l.to)];
      const dx=b.x-a.x, dy=b.y-a.y;
      const d=Math.sqrt(dx*dx+dy*dy)||1;
      const target = 70 + (1-(l.confidence||0.6))*60;
      const k=0.08;
      const diff = d-target;
      const fx = (dx/d)*diff*k;
      const fy = (dy/d)*diff*k;
      a.vx += fx; a.vy += fy;
      b.vx -= fx; b.vy -= fy;
    }

    // integrate
    for(const e of ents){
      e.vx *= 0.86;
      e.vy *= 0.86;
      e.x += e.vx*dt*60;
      e.y += e.vy*dt*60;
    }
  }

  /* ---------- drawing ---------- */
  function drawOnce(){
    // single frame render (we also run a few steps when in graph mode)
    const dt = 1/60;
    if(!drawerOpen && mode===MODE.GRAPH){
      for(let i=0;i<2;i++) stepGraph(dt);
    }
    draw();
  }

  function draw(){
    ctx.clearRect(0,0,innerWidth,innerHeight);

    // soft vignette
    ctx.save();
    ctx.globalAlpha=0.8;
    const g=ctx.createRadialGradient(innerWidth*0.5,innerHeight*0.55, 50, innerWidth*0.5,innerHeight*0.55, Math.max(innerWidth,innerHeight)*0.75);
    g.addColorStop(0,"rgba(0,0,0,0)");
    g.addColorStop(1,"rgba(0,0,0,.55)");
    ctx.fillStyle=g;
    ctx.fillRect(0,0,innerWidth,innerHeight);
    ctx.restore();

    if(drawerOpen) return;

    const ents = filterEntities();
    const zmap = new Map(db.zones.map(z=>[z.id,z]));

    // camera transform
    ctx.save();
    ctx.translate(innerWidth/2, innerHeight/2);
    ctx.scale(view.z, view.z);
    ctx.translate(view.x, view.y);

    // ZONES mode: show squares strongly, links faint
    // GRAPH mode: show links strongly, zones faint
    const zoneAlpha = (mode===MODE.ZONES)?0.95:0.25;
    const linkAlpha = (mode===MODE.ZONES)?0.18:0.62;

    // zones
    if(db.zones.length){
      for(const z of db.zones){
        const size = 160; // abstract “1km²” visual
        const col = colorForLayer(z.layer);
        ctx.save();
        ctx.globalAlpha = zoneAlpha * (z.opacity ?? 0.2);
        ctx.fillStyle = col.replace("0.95", "0.22");
        ctx.strokeStyle = col.replace("0.95","0.55");
        ctx.lineWidth = 2;
        const x=z.x - size/2, y=z.y - size/2;
        // overlap toggle: if off, spread zones apart visually
        if(!ui.overlap.checked){
          const h = hashStr(z.id);
          const ox = (h2f(h)*480-240);
          const oy = (h2f(hashStr(z.id+"o"))*340-170);
          ctx.translate(ox*0.25, oy*0.25);
        }
        roundRect(ctx, x, y, size, size, 18);
        ctx.fill(); ctx.stroke();

        // zone label (only if labels ON)
        if(ui.labels.checked){
          ctx.globalAlpha = zoneAlpha*0.9;
          ctx.fillStyle="rgba(234,241,255,.65)";
          ctx.font="12px "+getComputedStyle(document.documentElement).getPropertyValue("--mono");
          ctx.fillText(z.id, x+12, y+20);
        }
        ctx.restore();
      }
    }

    // links
    ctx.save();
    ctx.globalAlpha = linkAlpha;
    ctx.lineWidth = 1.6;
    const entMap = new Map(ents.map(e=>[e.id,e]));
    for(const l of db.links){
      const a = entMap.get(l.from), b = entMap.get(l.to);
      if(!a||!b) continue;
      const c = Number.isFinite(+l.confidence)?+l.confidence:0.6;
      ctx.strokeStyle = colorForConfidence(c).replace("0.95", mode===MODE.GRAPH ? "0.55":"0.28");
      ctx.beginPath();
      ctx.moveTo(a.x,a.y);
      // slight curve for readability
      const mx=(a.x+b.x)/2, my=(a.y+b.y)/2;
      ctx.quadraticCurveTo(mx+6, my-6, b.x, b.y);
      ctx.stroke();
    }
    ctx.restore();

    // nodes
    for(const e of ents){
      const r = (e.id===selectedId) ? 10.5 : 8.2;
      const conf = Number.isFinite(+e.confidence)?+e.confidence:0.65;
      const col = colorForConfidence(conf);

      // outer glow
      ctx.save();
      ctx.globalAlpha = 0.20;
      ctx.fillStyle = col.replace("0.95","0.40");
      ctx.beginPath(); ctx.arc(e.x,e.y,r+6,0,Math.PI*2); ctx.fill();
      ctx.restore();

      // core
      ctx.save();
      ctx.fillStyle = col;
      ctx.beginPath(); ctx.arc(e.x,e.y,r,0,Math.PI*2); ctx.fill();
      ctx.restore();

      // small layer accent ring
      if(e.layer && e.layer!=="NEUTRE"){
        ctx.save();
        ctx.strokeStyle = colorForLayer(e.layer);
        ctx.globalAlpha = 0.85;
        ctx.lineWidth = 2;
        ctx.beginPath(); ctx.arc(e.x,e.y,r+2.2,0,Math.PI*2); ctx.stroke();
        ctx.restore();
      }

      // labels
      if(ui.labels.checked){
        const name = displayName(e);
        ctx.save();
        ctx.fillStyle="rgba(234,241,255,.82)";
        ctx.font="12px "+getComputedStyle(document.documentElement).getPropertyValue("--mono");
        ctx.fillText(name, e.x+12, e.y-10);
        ctx.fillStyle="rgba(234,241,255,.55)";
        ctx.font="11px "+getComputedStyle(document.documentElement).getPropertyValue("--mono");
        ctx.fillText(`${e.type} • ${e.zone||"—"} • ${conf.toFixed(2)}`, e.x+12, e.y+6);
        ctx.restore();
      }
    }

    ctx.restore(); // camera

    // small mode badge (no entity names)
    ctx.save();
    ctx.globalAlpha=0.85;
    ctx.fillStyle="rgba(0,0,0,.22)";
    ctx.strokeStyle="rgba(255,255,255,.10)";
    ctx.lineWidth=1;
    roundRect(ctx, 12, innerHeight-54, 220, 40, 14);
    ctx.fill(); ctx.stroke();
    ctx.fillStyle="rgba(234,241,255,.85)";
    ctx.font="12px "+getComputedStyle(document.documentElement).getPropertyValue("--mono");
    ctx.fillText(`MODE: ${mode} • entités:${filterEntities().length}`, 24, innerHeight-30);
    ctx.restore();
  }

  function roundRect(ctx,x,y,w,h,r){
    r=Math.min(r,w/2,h/2);
    ctx.beginPath();
    ctx.moveTo(x+r,y);
    ctx.arcTo(x+w,y,x+w,y+h,r);
    ctx.arcTo(x+w,y+h,x,y+h,r);
    ctx.arcTo(x,y+h,x,y,r);
    ctx.arcTo(x,y,x+w,y,r);
    ctx.closePath();
  }

  /* ---------- boot ---------- */
  fitCanvas();

  // start with empty, but with one zone
  db.zones=[normZone({id:"zone_1km2_unknown_01",layer:"NEUTRE",opacity:0.18})];
  ensureZones();
  syncTextarea();
  renderAll();
  log("Prêt. Importe ton JSON via Monitor.");

  // draw loop (light)
  let last=performance.now();
  function tick(t){
    const dt=Math.min(0.033,(t-last)/1000); last=t;
    if(!drawerOpen && mode===MODE.GRAPH) stepGraph(dt);
    draw();
    requestAnimationFrame(tick);
  }
  requestAnimationFrame(tick);

})();
</script>
</body>
</html>
```
