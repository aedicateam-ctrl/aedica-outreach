---
---
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Aedica Outreach Engine</title>
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=Syne:wght@700;800&display=swap" rel="stylesheet"/>
<style>
  *{box-sizing:border-box;margin:0;padding:0}
  body{background:#0a0a0a;color:#f0ede8;font-family:'DM Sans','Segoe UI',sans-serif;min-height:100vh;padding-bottom:100px}
  ::-webkit-scrollbar{width:4px}
  ::-webkit-scrollbar-thumb{background:#f97316;border-radius:2px}
  @keyframes fadeIn{from{opacity:0;transform:translateY(10px)}to{opacity:1;transform:translateY(0)}}
  @keyframes spin{to{transform:rotate(360deg)}}
  @keyframes pulse{0%,100%{opacity:1}50%{opacity:0.3}}
  .fade-in{animation:fadeIn 0.35s ease forwards}
  .spinning{display:inline-block;animation:spin 1s linear infinite}
  .pulsing{animation:pulse 1.4s infinite}
  .btn{transition:all 0.15s;cursor:pointer}
  .btn:active{opacity:0.75;transform:scale(0.97)}
  .card:active{transform:scale(0.97);transition:transform 0.1s}
  textarea{resize:none;outline:none}
  input{outline:none}
  a{text-decoration:none}
  .msg-variant{background:#0d0d0d;border:1px solid #2a2a2a;border-radius:10px;padding:12px;margin-bottom:8px;cursor:pointer;transition:border-color 0.15s}
  .msg-variant.selected{border-color:#f97316;background:#1a0a00}
  .wa-btn{background:#25d366;color:#fff;border:none;border-radius:8px;padding:8px 14px;font-size:12px;font-weight:700;cursor:pointer;font-family:inherit;display:inline-flex;align-items:center;gap:6px;text-decoration:none}
  .floating-bar{position:fixed;bottom:0;left:0;right:0;background:#0f0f0f;border-top:1px solid #1a1a1a;padding:10px 16px;display:flex;gap:8px;z-index:200}
</style>
</head>
<body>

<!-- SETUP -->
<div id="setupScreen" style="display:none;min-height:100vh;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:30px">
  <div style="font-family:'Syne',sans-serif;font-weight:800;font-size:28px;color:#f97316;margin-bottom:8px">AEDICA</div>
  <div style="font-size:11px;color:#444;letter-spacing:2px;text-transform:uppercase;margin-bottom:40px">Outreach Engine v2</div>
  <div style="width:100%;max-width:420px;background:#111;border:1px solid #1c1c1c;border-radius:16px;padding:24px;display:flex;flex-direction:column;gap:20px">

    <div>
      <div style="font-size:15px;font-weight:600;margin-bottom:4px">🔥 Groq API Key <span style="color:#10b981;font-size:11px;font-weight:400">(AI — free, super fast)</span></div>
      <div style="font-size:12px;color:#555;margin-bottom:10px;line-height:1.6">
        Get free at <a href="https://console.groq.com" target="_blank" style="color:#f97316">console.groq.com</a> → Sign up → API Keys → Create Key
      </div>
      <input id="groqKeyInput" type="password" placeholder="gsk_..."
        style="width:100%;background:#0d0d0d;border:1px solid #222;border-radius:10px;padding:12px;color:#f0ede8;font-size:14px;font-family:inherit"/>
    </div>

    <div>
      <div style="font-size:15px;font-weight:600;margin-bottom:4px">🔍 Serper API Key <span style="color:#10b981;font-size:11px;font-weight:400">(Google search — 2500 free/month)</span></div>
      <div style="font-size:12px;color:#555;margin-bottom:10px;line-height:1.6">
        Get free at <a href="https://serper.dev" target="_blank" style="color:#f97316">serper.dev</a> → Sign up → Dashboard → API Key
      </div>
      <input id="serperKeyInput" type="password" placeholder="your-serper-key..."
        style="width:100%;background:#0d0d0d;border:1px solid #222;border-radius:10px;padding:12px;color:#f0ede8;font-size:14px;font-family:inherit"/>
    </div>

    <button class="btn" onclick="saveKeys()"
      style="width:100%;background:#f97316;border:none;color:#fff;padding:13px;border-radius:10px;font-size:14px;font-weight:700;font-family:inherit">
      Let's Go →
    </button>
    <div id="keyError" style="color:#ef4444;font-size:12px;display:none;text-align:center">Please enter both keys.</div>
  </div>
</div>

<!-- MAIN APP -->
<div id="mainApp" style="display:none">

  <div id="toast" style="display:none;position:fixed;top:16px;left:50%;transform:translateX(-50%);color:#fff;padding:10px 20px;border-radius:10px;z-index:9999;font-size:13px;font-weight:600;box-shadow:0 4px 24px rgba(0,0,0,0.5);max-width:90vw;text-align:center"></div>

  <!-- HEADER -->
  <div style="background:#0f0f0f;border-bottom:1px solid #1a1a1a;padding:14px 18px;position:sticky;top:0;z-index:100;display:flex;align-items:center;justify-content:space-between">
    <div style="display:flex;align-items:center;gap:10px">
      <button id="backBtn" class="btn" onclick="goHome()" style="display:none;background:none;border:none;color:#f97316;font-size:22px;line-height:1;font-family:inherit">←</button>
      <div>
        <div style="font-family:'Syne',sans-serif;font-weight:800;font-size:17px;color:#f97316">AEDICA</div>
        <div style="font-size:10px;color:#444;letter-spacing:2px;text-transform:uppercase">Outreach Engine</div>
      </div>
    </div>
    <div style="display:flex;gap:8px;align-items:center">
      <div id="followUpBadge" style="display:none;background:#6366f1;color:#fff;border-radius:20px;padding:3px 10px;font-size:11px;font-weight:700"></div>
      <div id="trackerCount" style="background:#1a1a1a;border:1px solid #222;border-radius:8px;padding:5px 12px;font-size:12px;color:#555">0 tracked</div>
    </div>
  </div>

  <!-- HOME VIEW -->
  <div id="homeView" class="fade-in" style="padding:0 16px">
    <div id="statsRow" style="display:none;gap:8px;padding:14px 0 6px;overflow-x:auto"></div>
    <div style="display:flex;gap:8px;margin:14px 0">
      <button id="tabSearch" class="btn" onclick="setTab('search')"
        style="flex:1;padding:11px;border-radius:10px;background:#f97316;color:#fff;border:1px solid #f97316;font-size:13px;font-weight:600;font-family:inherit">🔍 Find Leads</button>
      <button id="tabTracker" class="btn" onclick="setTab('tracker')"
        style="flex:1;padding:11px;border-radius:10px;background:#161616;color:#555;border:1px solid #222;font-size:13px;font-weight:600;font-family:inherit">📋 Tracker (0)</button>
    </div>
    <div id="searchPanel">
      <div style="font-size:11px;color:#383838;letter-spacing:1.5px;text-transform:uppercase;margin-bottom:14px">Tap a category → 20 real leads + messages auto-written</div>
      <div id="categoryGrid"></div>
    </div>
    <div id="trackerPanel" style="display:none">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px">
        <div id="filterRow" style="display:flex;gap:6px;overflow-x:auto;padding-bottom:4px;flex:1"></div>
        <button class="btn" onclick="exportCSV()" style="background:#1a1a1a;border:1px solid #222;color:#10b981;padding:6px 12px;border-radius:8px;font-size:11px;font-weight:700;font-family:inherit;flex-shrink:0;margin-left:8px">📊 CSV</button>
      </div>
      <div id="emptyTracker" style="text-align:center;padding:50px;color:#222;display:none">
        <div style="font-size:36px;margin-bottom:10px">📭</div>
        <div style="font-size:13px">No leads yet.</div>
      </div>
      <div id="trackerList" style="display:flex;flex-direction:column;gap:10px"></div>
    </div>
  </div>

  <!-- CATEGORY VIEW -->
  <div id="categoryView" style="display:none;padding:0 16px">
    <div style="padding:16px 0 10px">
      <div id="catIcon" style="font-size:26px"></div>
      <div id="catLabel" style="font-family:'Syne',sans-serif;font-weight:800;font-size:22px;margin-top:6px"></div>
      <div style="font-size:11px;color:#383838;margin-top:4px;letter-spacing:1px;text-transform:uppercase">Worldwide · Real Google Search · LLaMA AI</div>
    </div>

    <div id="progressWrap" style="display:none;margin-bottom:16px">
      <div style="display:flex;justify-content:space-between;margin-bottom:6px">
        <div id="progressLabel" style="font-size:12px;color:#f97316;font-weight:600"></div>
        <div id="progressCount" style="font-size:12px;color:#444"></div>
      </div>
      <div style="background:#1a1a1a;border-radius:4px;height:4px">
        <div id="progressBar" style="height:4px;background:#f97316;border-radius:2px;transition:width 0.3s;width:0%"></div>
      </div>
    </div>

    <div id="searchingState" style="display:none;text-align:center;padding:40px 20px">
      <div class="spinning" style="font-size:32px;margin-bottom:16px">🔍</div>
      <div style="font-size:14px;color:#f97316;font-weight:600">Searching Google for real leads...</div>
      <div id="searchLog" style="font-size:12px;color:#2a2a2a;margin-top:8px"></div>
    </div>

    <div id="waBlastBar" style="display:none;background:#0d2b1a;border:1px solid #25d36633;border-radius:12px;padding:12px 14px;margin-bottom:14px">
      <div style="font-size:11px;color:#25d366;font-weight:700;letter-spacing:1px;text-transform:uppercase;margin-bottom:8px">💬 WhatsApp Blast Ready</div>
      <div id="waBlastList" style="display:flex;flex-direction:column;gap:6px"></div>
    </div>

    <div id="resultsList" style="display:flex;flex-direction:column;gap:12px"></div>
  </div>

  <div id="floatingBar" class="floating-bar" style="display:none">
    <button class="btn" onclick="searchLeads(currentCategory, true)"
      style="flex:1;background:#161616;border:1px solid #222;color:#888;padding:12px;border-radius:10px;font-size:13px;font-weight:600;font-family:inherit">
      🔄 Load 20 More
    </button>
    <button class="btn" onclick="exportCSV()"
      style="background:#1a1a1a;border:1px solid #222;color:#10b981;padding:12px 16px;border-radius:10px;font-size:13px;font-weight:600;font-family:inherit">
      📊 CSV
    </button>
  </div>
</div>

<script>
const CATEGORIES = [
  {id:"plumbers",label:"Plumbers",icon:"🔧",query:"plumber self employed independent South Africa",angle:"Aedica gets you more clients and pays you safely via escrow — no more chasing invoices."},
  {id:"electricians",label:"Electricians",icon:"⚡",query:"electrician self employed contractor South Africa",angle:"More jobs, secure payment, no middleman — that's Aedica."},
  {id:"tilers",label:"Tilers",icon:"🏠",query:"tiler flooring installer self employed",angle:"Aedica connects you with homeowners who need tiling done now."},
  {id:"hvac",label:"HVAC / Aircon",icon:"❄️",query:"HVAC aircon installer self employed South Africa",angle:"Homeowners need you — Aedica makes sure you get found and get paid."},
  {id:"builders",label:"Builders",icon:"🏗️",query:"builder construction contractor self employed South Africa",angle:"Aedica helps you find more building jobs and get paid securely."},
  {id:"painters",label:"Painters",icon:"🎨",query:"painter decorator self employed independent South Africa",angle:"Find more painting jobs through Aedica — homeowners post jobs, you get paid safely."},
  {id:"solar",label:"Solar Installers",icon:"☀️",query:"solar panel installer self employed South Africa",angle:"Solar is booming — Aedica connects you with homeowners ready to install."},
  {id:"roofers",label:"Roofers",icon:"🏚️",query:"roofer roofing contractor self employed South Africa",angle:"Aedica gets you roofing jobs with secure payment — no more bad debt."},
  {id:"landscapers",label:"Landscapers",icon:"🌿",query:"landscaper gardener self employed South Africa",angle:"Homeowners need reliable garden services — Aedica puts you in front of them."},
  {id:"pool",label:"Pool Services",icon:"🏊",query:"pool installer cleaner self employed South Africa",angle:"Pool installs and maintenance — Aedica connects you with homeowners."},
  {id:"security",label:"Security Installers",icon:"📹",query:"CCTV alarm security installer self employed South Africa",angle:"Security is in demand — Aedica helps you get more installations booked."},
  {id:"carpenters",label:"Carpenters",icon:"🪵",query:"carpenter joiner cabinet maker self employed South Africa",angle:"Find more carpentry jobs through Aedica."},
  {id:"waterproofing",label:"Waterproofing",icon:"💧",query:"waterproofing contractor self employed South Africa",angle:"Aedica connects you with homeowners dealing with leaks right now."},
  {id:"pest",label:"Pest Control",icon:"🐛",query:"pest control exterminator self employed South Africa",angle:"Homeowners need pest control urgently — Aedica puts you first."},
  {id:"cleaning",label:"Cleaning Services",icon:"🧹",query:"cleaning service self employed South Africa",angle:"Regular cleaning clients through Aedica — steady income, secure payments."},
  {id:"handyman",label:"Handymen",icon:"🛠️",query:"handyman odd jobs self employed South Africa",angle:"Aedica matches you with homeowners who need exactly what you can do."},
  {id:"gates",label:"Gate & Fence",icon:"🚪",query:"gate fence installer self employed South Africa",angle:"Gate and fence jobs everywhere — Aedica connects you with homeowners."},
  {id:"movers",label:"Movers",icon:"🚚",query:"moving company removals self employed South Africa",angle:"Homeowners moving house need reliable movers — Aedica puts you on their radar."},
  {id:"locksmiths",label:"Locksmiths",icon:"🔑",query:"locksmith self employed South Africa",angle:"Aedica connects you with homeowners who need you urgently."},
  {id:"glaziers",label:"Glaziers",icon:"🪟",query:"glazier window fitter self employed South Africa",angle:"Broken windows, new installs — Aedica connects you with jobs daily."},
  {id:"borehole",label:"Borehole & Water",icon:"💦",query:"borehole drilling water tank installer South Africa",angle:"Water solutions in demand — Aedica helps you reach homeowners."},
  {id:"interior",label:"Interior Decorators",icon:"🛋️",query:"interior decorator designer self employed South Africa",angle:"Homeowners looking for interior upgrades — Aedica connects you."},
  {id:"architects",label:"Architects",icon:"📐",query:"architect designer South Africa",angle:"Your clients need contractors — Aedica makes that connection seamless."},
  {id:"estate_agents",label:"Estate Agents",icon:"🏡",query:"real estate agent property South Africa",angle:"Your clients need contractors — Aedica connects them. Let's partner."},
  {id:"property_developers",label:"Property Developers",icon:"🏢",query:"property developer South Africa",angle:"Aedica streamlines contractor sourcing for your developments."},
  {id:"body_corporates",label:"Body Corporates",icon:"🏘️",query:"body corporate property manager South Africa",angle:"Managing a complex? Aedica gives you vetted contractors instantly."},
  {id:"airbnb_hosts",label:"Airbnb Hosts",icon:"🛎️",query:"Airbnb host superhost South Africa",angle:"Need reliable contractors fast? Aedica is built for SA hosts."},
  {id:"hardware_stores",label:"Hardware Stores",icon:"🔨",query:"hardware store owner South Africa",angle:"Your customers are contractors. Aedica helps them get more jobs."},
  {id:"stokvel",label:"Stokvel Groups",icon:"🤝",query:"stokvel community savings group South Africa",angle:"Aedica helps your members find trusted tradespeople."},
  {id:"church",label:"Church Leaders",icon:"⛪",query:"pastor church leader community South Africa",angle:"Your congregation needs home services — Aedica is the platform to recommend."},
  {id:"township",label:"Township Entrepreneurs",icon:"🏘️",query:"township entrepreneur small business South Africa",angle:"Aedica is built for SA — helping informal workers and homeowners connect safely."},
  {id:"bank_managers",label:"Bank Managers",icon:"🏦",query:"bank manager business banking South Africa",angle:"Aedica's escrow wallet and loan calculator — let's explore a banking partnership."},
  {id:"fintech",label:"Fintech & Investors",icon:"💳",query:"fintech startup investor South Africa",angle:"Aedica is at MVP stage — looking for strategic partners and early backers."},
  {id:"insurance",label:"Insurance Brokers",icon:"🛡️",query:"home insurance broker South Africa",angle:"When a client claims, they need a contractor fast. Aedica solves that."},
  {id:"investors",label:"Investors & VCs",icon:"📈",query:"venture capital investor South Africa startup",angle:"Aedica is building fast — open to early investor conversations."},
  {id:"accelerators",label:"Accelerators",icon:"🚀",query:"startup accelerator incubator South Africa",angle:"Aedica is MVP-ready and looking for the right programme."},
  {id:"tech_founders",label:"Tech Founders",icon:"💻",query:"tech startup founder South Africa",angle:"Building Aedica — a home services marketplace in SA. Would love your perspective."},
  {id:"media",label:"Tech Media",icon:"📰",query:"tech journalist South Africa TechCentral",angle:"Building something new in SA's home services space — worth a story?"},
  {id:"influencers",label:"SA Influencers",icon:"📢",query:"South Africa business influencer entrepreneur",angle:"Aedica connects homeowners with vetted tradespeople — the SA market needs this."},
  {id:"property_lawyers",label:"Property Attorneys",icon:"⚖️",query:"property attorney conveyancer South Africa",angle:"Aedica is building the infrastructure for home services in SA — let's connect."},
  {id:"seda",label:"SEDA & Gov",icon:"🏛️",query:"SEDA small enterprise development South Africa",angle:"Aedica creates work for tradespeople — exploring funding alignment."},
];

const STATUS_OPTIONS = ["Sent","Replied","Not Interested","Follow Up","Partnership"];
const STATUS_COLORS = {Sent:"#f59e0b",Replied:"#10b981","Not Interested":"#ef4444","Follow Up":"#6366f1",Partnership:"#ec4899"};
const PICONS = {LinkedIn:"💼",Instagram:"📸",Facebook:"👥",Twitter:"🐦",TikTok:"🎵",YouTube:"▶️",Gumtree:"🟢",Website:"🌐",WhatsApp:"💬",Google:"🔎"};
const STORAGE_KEY = "aedica_v5";

let tracker = [], currentCategory = null, currentResults = [];
let activeTab = "search", filterStatus = "All";
let generatedMessages = {}, isAutoGenerating = false;

// ── KEYS ──────────────────────────────────────────────
function getGroqKey() { return localStorage.getItem("aedica_groq_key")||""; }
function getSerperKey() { return localStorage.getItem("aedica_serper_key")||""; }

function saveKeys() {
  const g = document.getElementById("groqKeyInput").value.trim();
  const s = document.getElementById("serperKeyInput").value.trim();
  if (!g || !s) { document.getElementById("keyError").style.display="block"; return; }
  localStorage.setItem("aedica_groq_key", g);
  localStorage.setItem("aedica_serper_key", s);
  document.getElementById("setupScreen").style.display = "none";
  document.getElementById("mainApp").style.display = "block";
  initApp();
}

window.addEventListener('load', () => {
  if (getGroqKey() && getSerperKey()) {
    document.getElementById("setupScreen").style.display = "none";
    document.getElementById("mainApp").style.display = "block";
    initApp();
  } else {
    document.getElementById("setupScreen").style.display = "flex";
    document.getElementById("mainApp").style.display = "none";
  }
});

function initApp() {
  try { tracker = JSON.parse(localStorage.getItem(STORAGE_KEY)||"[]"); } catch { tracker=[]; }
  buildCategoryGrid();
  renderAll();
  if ("Notification" in window && Notification.permission==="default") Notification.requestPermission();
  checkDailySummary();
}

// ── GROQ AI CALL ──────────────────────────────────────
async function groq(systemPrompt, userPrompt, maxTokens=1000, temp=0.7) {
  const res = await fetch("https://api.groq.com/openai/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": "Bearer " + getGroqKey()
    },
    body: JSON.stringify({
      model: "llama-3.3-70b-versatile",
      messages: [
        {role:"system", content: systemPrompt},
        {role:"user", content: userPrompt}
      ],
      temperature: temp,
      max_tokens: maxTokens
    })
  });
  const data = await res.json();
  if (data.error) throw new Error(data.error.message);
  return data.choices?.[0]?.message?.content || "";
}

// ── SERPER SEARCH ─────────────────────────────────────
async function serperSearch(query, num=10) {
  const res = await fetch("https://google.serper.dev/search", {
    method: "POST",
    headers: {"Content-Type":"application/json", "X-API-KEY": getSerperKey()},
    body: JSON.stringify({q: query, num, gl:"za"})
  });
  const data = await res.json();
  if (data.error) throw new Error("Serper: " + (data.message||data.error));
  return data;
}

// ── NOTIFICATIONS ─────────────────────────────────────
function sendNotification(title, body) {
  if ("Notification" in window && Notification.permission==="granted") {
    new Notification(title, {body});
  }
}
function checkDailySummary() {
  const today = new Date().toDateString();
  if (localStorage.getItem("aedica_summary_date")===today) return;
  localStorage.setItem("aedica_summary_date", today);
  setTimeout(() => {
    const fu = tracker.filter(t=>t.status==="Follow Up").length;
    const replied = tracker.filter(t=>t.status==="Replied").length;
    if (tracker.length > 0) {
      sendNotification("Aedica Daily", `${tracker.length} leads tracked · ${replied} replied · ${fu} follow-ups due`);
      if (fu > 0) showToast(`You have ${fu} follow-up${fu>1?"s":""} waiting!`, "error");
    }
  }, 2000);
}
function scheduleFollowUpReminder(name) {
  setTimeout(() => sendNotification("Follow-up reminder", `Time to follow up with ${name}!`), 3*24*60*60*1000);
}

// ── TOAST ─────────────────────────────────────────────
function showToast(msg, type="success") {
  const t = document.getElementById("toast");
  t.textContent = msg; t.style.background = type==="error"?"#ef4444":"#f97316";
  t.style.display = "block"; setTimeout(()=>t.style.display="none", 3500);
}

// ── NAV ───────────────────────────────────────────────
function goHome() {
  document.getElementById("homeView").style.display = "block";
  document.getElementById("categoryView").style.display = "none";
  document.getElementById("backBtn").style.display = "none";
  document.getElementById("floatingBar").style.display = "none";
  renderAll();
}

function showCategoryView(cat) {
  currentCategory = cat; currentResults = []; generatedMessages = {};
  document.getElementById("homeView").style.display = "none";
  document.getElementById("categoryView").style.display = "block";
  document.getElementById("backBtn").style.display = "inline-block";
  document.getElementById("catIcon").textContent = cat.icon;
  document.getElementById("catLabel").textContent = cat.label;
  document.getElementById("resultsList").innerHTML = "";
  document.getElementById("floatingBar").style.display = "none";
  document.getElementById("waBlastBar").style.display = "none";
  searchLeads(cat, false);
}

function setTab(tab) {
  activeTab = tab;
  const s = tab==="search";
  document.getElementById("searchPanel").style.display = s?"block":"none";
  document.getElementById("trackerPanel").style.display = s?"none":"block";
  document.getElementById("tabSearch").style.background = s?"#f97316":"#161616";
  document.getElementById("tabSearch").style.color = s?"#fff":"#555";
  document.getElementById("tabSearch").style.borderColor = s?"#f97316":"#222";
  document.getElementById("tabTracker").style.background = s?"#161616":"#f97316";
  document.getElementById("tabTracker").style.color = s?"#555":"#fff";
  document.getElementById("tabTracker").style.borderColor = s?"#222":"#f97316";
  if (tab==="tracker") renderTracker();
}

// ── GRID ──────────────────────────────────────────────
function buildCategoryGrid() {
  const grid = document.getElementById("categoryGrid");
  const sections = [
    {label:"🔨 Tradespeople", ids:["plumbers","electricians","tilers","hvac","builders","painters","solar","roofers","landscapers","pool","security","carpenters","waterproofing","pest","cleaning","handyman","gates","movers","locksmiths","glaziers","borehole","interior"]},
    {label:"🤝 Business & Partners", ids:["architects","estate_agents","property_developers","body_corporates","airbnb_hosts","hardware_stores","stokvel","church","township"]},
    {label:"💰 Finance & Investment", ids:["bank_managers","fintech","insurance","investors","accelerators"]},
    {label:"📣 Media & Influence", ids:["tech_founders","media","influencers","property_lawyers","seda"]},
  ];
  grid.innerHTML = sections.map(sec => {
    const cats = CATEGORIES.filter(c=>sec.ids.includes(c.id));
    return `<div style="margin-bottom:20px">
      <div style="font-size:11px;color:#444;letter-spacing:1.5px;text-transform:uppercase;margin-bottom:10px;font-weight:600">${sec.label}</div>
      <div style="display:grid;grid-template-columns:1fr 1fr;gap:8px">
        ${cats.map(cat=>`
          <button class="card btn" onclick="showCategoryView(CATEGORIES.find(c=>c.id==='${cat.id}'))"
            style="background:#111;border:1px solid #1c1c1c;border-radius:12px;padding:14px 12px;text-align:left;display:flex;flex-direction:column;gap:6px;font-family:inherit">
            <span style="font-size:20px">${cat.icon}</span>
            <span style="font-size:12px;font-weight:600;color:#e0ddd8;line-height:1.4">${cat.label}</span>
          </button>`).join("")}
      </div></div>`;
  }).join("");
}

// ── RENDER ALL ────────────────────────────────────────
function renderAll() {
  const fu = tracker.filter(t=>t.status==="Follow Up");
  const badge = document.getElementById("followUpBadge");
  badge.style.display = fu.length?"block":"none";
  if (fu.length) badge.textContent = fu.length+" follow-up"+(fu.length>1?"s":"");
  document.getElementById("trackerCount").textContent = tracker.length+" tracked";
  document.getElementById("tabTracker").textContent = "📋 Tracker ("+tracker.length+")";
  if (activeTab==="tracker") { document.getElementById("tabTracker").style.background="#f97316"; document.getElementById("tabTracker").style.color="#fff"; }
  const sr = document.getElementById("statsRow");
  if (tracker.length>0) {
    sr.style.display="flex";
    sr.innerHTML = ["Sent","Replied","Follow Up","Partnership"].map(s=>`
      <div style="background:#111;border:1px solid ${STATUS_COLORS[s]}44;border-radius:10px;padding:10px 14px;min-width:80px;text-align:center;flex-shrink:0">
        <div style="font-size:18px;font-weight:700;color:${STATUS_COLORS[s]}">${tracker.filter(t=>t.status===s).length}</div>
        <div style="font-size:10px;color:#444;margin-top:2px">${s}</div>
      </div>`).join("");
  } else sr.style.display="none";
  if (activeTab==="tracker") renderTracker();
}

// ── RENDER TRACKER ────────────────────────────────────
function renderTracker() {
  const fr = document.getElementById("filterRow");
  fr.innerHTML = ["All",...STATUS_OPTIONS].map(s=>{
    const c=s==="All"?tracker.length:tracker.filter(t=>t.status===s).length;
    const a=filterStatus===s, col=STATUS_COLORS[s]||"#f97316";
    return `<button class="btn" onclick="setFilter('${s}')"
      style="padding:5px 13px;border-radius:20px;font-size:11px;font-weight:600;background:${a?col:"#161616"};color:${a?"#fff":"#555"};border:1px solid ${a?col:"#222"};white-space:nowrap;flex-shrink:0;font-family:inherit">${s} (${c})</button>`;
  }).join("");
  const filtered = filterStatus==="All"?tracker:tracker.filter(t=>t.status===filterStatus);
  const list = document.getElementById("trackerList");
  if (!filtered.length) { document.getElementById("emptyTracker").style.display="block"; list.innerHTML=""; return; }
  document.getElementById("emptyTracker").style.display="none";
  list.innerHTML = filtered.map(lead=>`
    <div class="fade-in" style="background:#111;border:1px solid ${STATUS_COLORS[lead.status]}33;border-radius:12px;padding:14px">
      <div style="display:flex;justify-content:space-between;margin-bottom:10px">
        <div>
          <div style="font-weight:600;font-size:14px">${lead.name}</div>
          <div style="font-size:12px;color:#666;margin-top:2px">${lead.title} · ${lead.company}</div>
          <div style="font-size:11px;color:#383838;margin-top:2px">${lead.location} · ${lead.category} · ${lead.date}</div>
          ${lead.phone?`<div style="margin-top:6px"><a href="https://wa.me/${lead.phone.replace(/\D/g,'')}" target="_blank" class="wa-btn" style="font-size:11px;padding:5px 10px">💬 ${lead.phone}</a></div>`:""}
        </div>
        <button class="btn" onclick="removeLead('${lead.id}')" style="background:none;border:none;color:#2a2a2a;font-size:18px;align-self:flex-start;font-family:inherit">✕</button>
      </div>
      <div style="display:flex;gap:6px;flex-wrap:wrap;margin-bottom:10px">
        ${STATUS_OPTIONS.map(s=>`
          <button class="btn" onclick="updateStatus('${lead.id}','${s}')"
            style="padding:4px 10px;border-radius:20px;font-size:11px;font-weight:600;background:${lead.status===s?STATUS_COLORS[s]:"#1a1a1a"};color:${lead.status===s?"#fff":"#383838"};border:1px solid ${lead.status===s?STATUS_COLORS[s]:"#222"};font-family:inherit">${s}</button>`).join("")}
      </div>
      ${lead.searchUrl?`<a href="${lead.searchUrl}" target="_blank" style="display:inline-block;margin-bottom:8px;color:#f97316;font-size:12px;font-weight:600">${PICONS[lead.platform]||"🔗"} Open on ${lead.platform||"web"}</a>`:""}
      ${lead.message?`
        <div style="background:#0d0d0d;border:1px solid #1a1a1a;border-radius:8px;padding:10px;margin-bottom:8px">
          <div style="font-size:12px;color:#777;line-height:1.6">${lead.message}</div>
          <div style="display:flex;gap:8px;margin-top:8px;flex-wrap:wrap">
            <button class="btn" onclick="copyText('${encodeURIComponent(lead.message)}','Copied!')" style="background:none;border:1px solid #f97316;color:#f97316;padding:5px 12px;border-radius:6px;font-size:11px;font-weight:600;font-family:inherit">Copy</button>
            ${lead.phone?`<a href="https://wa.me/${lead.phone.replace(/\D/g,'')}?text=${encodeURIComponent(lead.message)}" target="_blank" class="wa-btn" style="font-size:11px;padding:5px 10px">💬 Send WA</a>`:""}
          </div>
        </div>`:""}
      ${lead.status==="Follow Up"?`
        <button class="btn" onclick="generateFollowUp('${lead.id}')" id="fuBtn_${lead.id}"
          style="background:#6366f1;border:none;color:#fff;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600;font-family:inherit;margin-bottom:8px">🔁 Generate Follow-Up</button>
        <div id="fuBox_${lead.id}" style="display:none;background:#0d0d0d;border:1px solid #6366f133;border-radius:8px;padding:10px;margin-bottom:8px">
          <div id="fuText_${lead.id}" style="font-size:12px;color:#aaa;line-height:1.6"></div>
          <button class="btn" onclick="copyFu('${lead.id}')" style="margin-top:8px;background:none;border:1px solid #6366f1;color:#6366f1;padding:5px 12px;border-radius:6px;font-size:11px;font-weight:600;font-family:inherit">Copy</button>
        </div>`:""}
      ${lead.status==="Replied"?`
        <button class="btn" onclick="generateReply('${lead.id}')" id="repBtn_${lead.id}"
          style="background:#10b981;border:none;color:#fff;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600;font-family:inherit;margin-bottom:8px">💬 Generate Reply</button>
        <div id="repBox_${lead.id}" style="display:none;background:#0d0d0d;border:1px solid #10b98133;border-radius:8px;padding:10px;margin-bottom:8px">
          <div id="repText_${lead.id}" style="font-size:12px;color:#aaa;line-height:1.6"></div>
          <button class="btn" onclick="copyRep('${lead.id}')" style="margin-top:8px;background:none;border:1px solid #10b981;color:#10b981;padding:5px 12px;border-radius:6px;font-size:11px;font-weight:600;font-family:inherit">Copy</button>
        </div>`:""}
      <textarea onchange="updateNotes('${lead.id}',this.value)" placeholder="Notes..." rows="2"
        style="width:100%;background:#0d0d0d;border:1px solid #1c1c1c;border-radius:8px;padding:8px;color:#666;font-size:12px;font-family:inherit">${lead.notes||""}</textarea>
    </div>`).join("");
}

function setFilter(s) { filterStatus=s; renderTracker(); }

// ── SEARCH LEADS ──────────────────────────────────────
async function searchLeads(category, append=false) {
  document.getElementById("searchingState").style.display = "block";
  document.getElementById("floatingBar").style.display = "none";
  if (!append) document.getElementById("resultsList").innerHTML = "";

  try {
    // Step 1: Real Google search via Serper
    document.getElementById("searchLog").textContent = "Searching Google...";
    const alreadyNames = append ? currentResults.map(r=>r.name) : [];

    // Run 2 searches to get more diverse results
    const query1 = category.query;
    const query2 = category.query + " Facebook Instagram phone number contact";
    document.getElementById("searchLog").textContent = "Running Google searches...";

    const [s1, s2] = await Promise.all([
      serperSearch(query1, 10),
      serperSearch(query2, 10)
    ]);

    const allResults = [...(s1.organic||[]), ...(s2.organic||[])];
    const snippets = allResults.map((r,i) =>
      `${i+1}. Title: ${r.title}\n   Snippet: ${r.snippet||""}\n   URL: ${r.link}`
    ).join("\n\n");

    // Step 2: Groq extracts 20 structured leads from real search results
    document.getElementById("searchLog").textContent = "AI extracting leads from results...";

    const systemPrompt = `You are a lead extraction expert for Aedica (aedica.co.za), a South African home services marketplace. Extract real people and businesses from search results. Always return valid JSON only — no markdown, no backticks, no explanation.`;

    const userPrompt = `Extract 20 real leads for "${category.label}" from these Google search results.

SEARCH RESULTS:
${snippets}

For each lead find:
- Their name or business name
- What they do exactly
- Their location
- Which platform they were found on (Facebook, Instagram, TikTok, Gumtree, Website, LinkedIn, YouTube, WhatsApp, Google)
- Their direct profile URL or best search URL
- Any phone number found (SA numbers: +27 / 06x / 07x / 08x format)
- A specific note about them (what makes them a good lead, what they post about, follower count, etc)

${alreadyNames.length ? `Do NOT include these already found: ${alreadyNames.join(", ")}` : ""}

If there aren't 20 in the results, create plausible leads based on the search context but mark them with notes saying "suggested lead".

Return ONLY this JSON array:
[{"name":"string","title":"string","company":"string","location":"string","platform":"string","searchUrl":"string","phone":"string or empty","notes":"string"}]`;

    const raw = await groq(systemPrompt, userPrompt, 2000, 0.3);
    let leads = [];
    const match = raw.match(/\[[\s\S]*\]/);
    if (match) { try { leads = JSON.parse(match[0]); } catch(e) {} }

    leads = leads.slice(0, 20).map(l => ({
      ...l,
      phone: l.phone||"",
      searchUrl: l.searchUrl?.startsWith("http") ? l.searchUrl
        : `https://www.google.com/search?q=${encodeURIComponent((l.name||"")+" "+(l.title||"")+" "+(l.location||""))}`,
      platform: l.platform||"Google"
    }));

    if (!leads.length) { showToast("No leads found, try again","error"); document.getElementById("searchingState").style.display="none"; return; }

    const offset = append ? currentResults.length : 0;
    if (append) currentResults = [...currentResults, ...leads];
    else currentResults = leads;

    if (append) appendResults(leads, category, offset);
    else renderResults(leads, category);

    autoTrackAll(leads, category);
    buildWABlast(currentResults);
    document.getElementById("floatingBar").style.display = "flex";
    autoGenerateMessages(leads, category, offset);

  } catch(e) {
    showToast("Search failed: " + e.message, "error");
    document.getElementById("resultsList").innerHTML = `<div style="text-align:center;padding:40px;color:#444;font-size:13px">Search failed: ${e.message}<br><br>Check your API keys.</div>`;
  }
  document.getElementById("searchingState").style.display = "none";
}

// ── AUTO TRACK ────────────────────────────────────────
function autoTrackAll(leads, category) {
  let added = 0;
  leads.forEach(lead => {
    if (!tracker.find(t=>t.name===lead.name&&t.company===lead.company)) {
      tracker.unshift({
        id:Date.now()+Math.random(), name:lead.name, title:lead.title,
        company:lead.company, location:lead.location, platform:lead.platform,
        phone:lead.phone||"", category:category.label, status:"Sent",
        date:new Date().toLocaleDateString("en-ZA"),
        notes:"", message:"", searchUrl:lead.searchUrl||""
      });
      added++;
      if (lead.phone) scheduleFollowUpReminder(lead.name);
    }
  });
  saveTracker(); renderAll();
  if (added > 0) showToast(added + " leads auto-tracked! 🎯");
}

// ── WA BLAST ──────────────────────────────────────────
function buildWABlast(leads) {
  const wp = leads.filter(l=>l.phone&&l.phone.length>6);
  const bar = document.getElementById("waBlastBar");
  if (!wp.length) { bar.style.display="none"; return; }
  bar.style.display = "block";
  document.getElementById("waBlastList").innerHTML = `
    <div style="font-size:12px;color:#888;margin-bottom:6px">${wp.length} leads with WhatsApp numbers:</div>
    ${wp.map(l=>`<a href="https://wa.me/${l.phone.replace(/\D/g,'')}?text=${encodeURIComponent('Hi '+l.name+', my name is Xolani from Aedica (aedica.co.za). ')}" target="_blank" class="wa-btn" style="font-size:12px">💬 ${l.name} · ${l.phone}</a>`).join("")}`;
}

// ── AUTO-GENERATE MESSAGES ────────────────────────────
async function autoGenerateMessages(leads, category, offset=0) {
  if (isAutoGenerating) return;
  isAutoGenerating = true;
  const wrap = document.getElementById("progressWrap");
  const label = document.getElementById("progressLabel");
  const countEl = document.getElementById("progressCount");
  const bar = document.getElementById("progressBar");
  wrap.style.display = "block";

  for (let i=0; i<leads.length; i++) {
    const idx = offset + i;
    label.textContent = "Auto-writing messages...";
    countEl.textContent = `${i+1} / ${leads.length}`;
    bar.style.width = ((i+1)/leads.length*100)+"%";
    if (generatedMessages[idx]) continue;

    try {
      const lead = leads[i];
      const platform = lead.platform||"Facebook";
      const variants = await writeMessageVariants(lead, category, platform);
      generatedMessages[idx] = variants;
      updateLeadMessageBox(idx, lead, variants);
      const t = tracker.find(t=>t.name===lead.name&&t.company===lead.company);
      if (t && !t.message) { t.message=variants[0].message; saveTracker(); }
    } catch(e) {}
    await new Promise(r=>setTimeout(r,600));
  }

  label.textContent = "✅ All messages ready!";
  countEl.textContent = "";
  setTimeout(()=>wrap.style.display="none", 2000);
  isAutoGenerating = false;
  showToast("All messages written by LLaMA AI! 🔥");
}

async function writeMessageVariants(lead, category, platform) {
  const toneMap = {LinkedIn:"professional but warm",Instagram:"casual and friendly",Facebook:"casual like texting a friend",TikTok:"very short and punchy",YouTube:"friendly and genuine",Gumtree:"direct and practical",Website:"brief and professional",WhatsApp:"very casual, like WhatsApp",Google:"friendly and direct"};
  const tone = toneMap[platform]||"casual and genuine";

  const system = `You are Xolani Mthethwa, founder of Aedica (aedica.co.za) — a South African home services marketplace where homeowners hire and pay tradespeople safely via escrow wallet. You write genuine, human outreach messages. Never sound like AI or a salesperson. Always return valid JSON only.`;

  const prompt = `Write 3 outreach message variants to ${lead.name}, a ${lead.title} at ${lead.company} in ${lead.location}. Found on ${platform}.

About them: ${lead.notes||"self-employed tradesperson"}
Platform tone: ${tone}
Outreach angle: ${category.angle}

Style 1 - CASUAL: Like texting a mate. Warm, short, human.
Style 2 - DIRECT: 2 sentences max. Lead with the benefit they get.
Style 3 - STORY: Start with a real SA pain point (not getting paid, hard to find work), then show how Aedica fixes it.

ALL messages must:
- Start with their name
- Mention aedica.co.za once
- Be under 5 sentences
- Sound like a real person wrote it
- Be tailored for ${platform}

Return ONLY this JSON (no markdown):
[{"style":"Casual","message":"..."},{"style":"Direct","message":"..."},{"style":"Story","message":"..."}]`;

  const raw = await groq(system, prompt, 700, 0.85);
  const match = raw.match(/\[[\s\S]*\]/);
  if (match) { try { return JSON.parse(match[0]); } catch(e) {} }
  return [{style:"Message", message:raw.trim()}];
}

function updateLeadMessageBox(idx, lead, variants) {
  const box = document.getElementById("msgBox_"+idx);
  const vDiv = document.getElementById("msgVariants_"+idx);
  const aDiv = document.getElementById("msgActions_"+idx);
  const btn = document.getElementById("msgStatus_"+idx);
  if (!box||!vDiv) return;
  box.style.display = "block";
  if (btn) { btn.textContent="✅ Ready — pick a style"; btn.style.color="#10b981"; }
  vDiv.innerHTML = variants.map((v,vi)=>`
    <div class="msg-variant ${vi===0?"selected":""}" id="variant_${idx}_${vi}"
      onclick="selectVariant(${idx},${vi},'${encodeURIComponent(JSON.stringify(variants))}')">
      <div style="font-size:10px;color:#f97316;font-weight:700;letter-spacing:1px;text-transform:uppercase;margin-bottom:5px">${v.style}</div>
      <div style="font-size:13px;color:#ccc;line-height:1.7">${v.message}</div>
    </div>`).join("");
  if (aDiv) {
    aDiv.style.display="block";
    const cb = document.getElementById("copyBtn_"+idx);
    if (cb) cb.onclick = ()=>{ navigator.clipboard.writeText(variants[0].message); showToast("Copied! 🔥"); };
    const wb = document.getElementById("waSendBtn_"+idx);
    if (wb&&lead.phone) wb.href=`https://wa.me/${lead.phone.replace(/\D/g,'')}?text=${encodeURIComponent(variants[0].message)}`;
  }
}

// ── RENDER ────────────────────────────────────────────
function renderResults(leads, category) {
  document.getElementById("resultsList").innerHTML = "";
  appendResults(leads, category, 0);
}

function appendResults(leads, category, offset) {
  const list = document.getElementById("resultsList");
  list.insertAdjacentHTML("beforeend", leads.map((lead,i)=>{
    const idx = offset+i;
    const icon = PICONS[lead.platform]||"🔗";
    const hasPhone = lead.phone&&lead.phone.length>6;
    return `
    <div class="fade-in" id="leadCard_${idx}" style="background:#111;border:1px solid #1c1c1c;border-radius:14px;padding:16px;animation-delay:${i*0.04}s;animation-fill-mode:forwards;opacity:0">
      <div style="margin-bottom:12px">
        <div style="display:flex;align-items:center;gap:8px;flex-wrap:wrap;margin-bottom:6px">
          <span style="background:#1a1a1a;border:1px solid #222;border-radius:6px;padding:2px 8px;font-size:11px;color:#888">${icon} ${lead.platform}</span>
          <span style="font-size:11px;color:#383838">${lead.location}</span>
          ${hasPhone?`<span style="background:#0d2b1a;border:1px solid #25d36633;border-radius:6px;padding:2px 8px;font-size:11px;color:#25d366">💬 WA</span>`:""}
        </div>
        <div style="font-weight:700;font-size:15px">${lead.name}</div>
        <div style="font-size:13px;color:#666;margin-top:2px">${lead.title}</div>
        <div style="font-size:12px;color:#383838;margin-top:2px">${lead.company}</div>
        ${lead.notes?`<div style="font-size:12px;color:#f97316;margin-top:6px;line-height:1.5;opacity:0.8;font-style:italic">"${lead.notes}"</div>`:""}
        ${hasPhone?`<div style="margin-top:8px"><a href="https://wa.me/${lead.phone.replace(/\D/g,'')}" target="_blank" class="wa-btn" style="font-size:11px;padding:5px 10px">💬 ${lead.phone}</a></div>`:""}
      </div>
      <a href="${lead.searchUrl}" target="_blank"
        style="display:inline-block;background:#1a1a1a;border:1px solid #2a2a2a;color:#f97316;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600;margin-bottom:12px">
        ${icon} View Profile
      </a>

      <div id="msgStatus_${idx}" style="font-size:11px;color:#555;margin-bottom:6px" class="pulsing">✍️ Writing message...</div>
      <div id="msgBox_${idx}" style="display:none">
        <div style="font-size:10px;color:#888;font-weight:600;letter-spacing:1px;text-transform:uppercase;margin-bottom:8px">Pick a style:</div>
        <div id="msgVariants_${idx}"></div>
        <div id="msgActions_${idx}" style="display:none;margin-top:10px">
          <div style="display:flex;gap:8px;flex-wrap:wrap">
            <button class="btn" id="copyBtn_${idx}"
              style="flex:1;background:#f97316;border:none;color:#fff;padding:10px;border-radius:8px;font-size:13px;font-weight:700;font-family:inherit">
              📋 Copy
            </button>
            ${hasPhone?`<a id="waSendBtn_${idx}" href="https://wa.me/${lead.phone.replace(/\D/g,'')}?text=" target="_blank" class="wa-btn" style="padding:10px 14px;font-size:12px">💬 Send WA</a>`:""}
            <a href="${lead.searchUrl}" target="_blank"
              style="background:#1a1a1a;border:1px solid #333;color:#fff;padding:10px 14px;border-radius:8px;font-size:12px;font-weight:700;display:flex;align-items:center">
              ${icon} Open
            </a>
          </div>
          <button class="btn" onclick="regenMessage(${idx},'${encodeURIComponent(JSON.stringify(lead))}','${encodeURIComponent(JSON.stringify(category))}')"
            style="width:100%;background:none;border:1px solid #222;color:#555;padding:8px;border-radius:8px;font-size:12px;font-family:inherit;margin-top:8px">
            🔁 Regenerate
          </button>
        </div>
      </div>
    </div>`;
  }).join(""));
}

function selectVariant(idx, vi, variantsEnc) {
  const v = JSON.parse(decodeURIComponent(variantsEnc));
  v.forEach((_,i)=>{ const el=document.getElementById(`variant_${idx}_${i}`); if(el)el.classList.toggle("selected",i===vi); });
  const msg = v[vi].message;
  const cb = document.getElementById("copyBtn_"+idx);
  if (cb) cb.onclick = ()=>{ navigator.clipboard.writeText(msg); showToast("Copied! 🔥"); };
  const wb = document.getElementById("waSendBtn_"+idx);
  const lead = currentResults[idx];
  if (wb&&lead?.phone) wb.href=`https://wa.me/${lead.phone.replace(/\D/g,'')}?text=${encodeURIComponent(msg)}`;
}

async function regenMessage(idx, leadEnc, catEnc) {
  const lead = JSON.parse(decodeURIComponent(leadEnc));
  const cat = JSON.parse(decodeURIComponent(catEnc));
  const vDiv = document.getElementById("msgVariants_"+idx);
  const aDiv = document.getElementById("msgActions_"+idx);
  if (vDiv) vDiv.innerHTML=`<div class="pulsing" style="color:#555;font-size:12px;padding:10px">Regenerating with LLaMA...</div>`;
  if (aDiv) aDiv.style.display="none";
  delete generatedMessages[idx];
  try {
    const variants = await writeMessageVariants(lead, cat, lead.platform||"Facebook");
    generatedMessages[idx] = variants;
    updateLeadMessageBox(idx, lead, variants);
  } catch(e) { showToast("Failed: "+e.message,"error"); }
}

// ── FOLLOW-UP ─────────────────────────────────────────
async function generateFollowUp(leadId) {
  const lead = tracker.find(t=>t.id===leadId);
  if (!lead) return;
  const btn = document.getElementById("fuBtn_"+leadId);
  btn.textContent="Writing..."; btn.disabled=true;
  try {
    const msg = await groq(
      `You are Xolani Mthethwa, founder of Aedica (aedica.co.za). Write genuine, human follow-up messages. Return only the message text, nothing else.`,
      `Write a 2-3 sentence follow-up message to ${lead.name}, a ${lead.title} on ${lead.platform||"social media"}.
Previous message: "${lead.message||"intro about Aedica"}"
Rules: Reference previous message lightly. Add one new value point about Aedica. End with easy yes/no question. Sound human not pushy.`,
      200, 0.8
    );
    tracker = tracker.map(t=>t.id===leadId?{...t,followUp:msg}:t);
    saveTracker();
    document.getElementById("fuText_"+leadId).textContent = msg;
    document.getElementById("fuBox_"+leadId).style.display = "block";
  } catch(e) { showToast("Failed: "+e.message,"error"); }
  btn.textContent="🔁 Generate Follow-Up"; btn.disabled=false;
}
function copyFu(id) { const l=tracker.find(t=>t.id===id); if(l?.followUp){navigator.clipboard.writeText(l.followUp);showToast("Copied!");} }

// ── REPLY ─────────────────────────────────────────────
async function generateReply(leadId) {
  const lead = tracker.find(t=>t.id===leadId);
  if (!lead) return;
  const btn = document.getElementById("repBtn_"+leadId);
  btn.textContent="Writing..."; btn.disabled=true;
  try {
    const msg = await groq(
      `You are Xolani Mthethwa, founder of Aedica (aedica.co.za). Write genuine, human reply messages. Return only the message text.`,
      `Write a reply to ${lead.name} (${lead.title}) who just responded positively to your Aedica outreach.
Give them the next clear step: sign up at aedica.co.za as a contractor. Keep it under 3 sentences. Sound warm and human.`,
      200, 0.8
    );
    tracker = tracker.map(t=>t.id===leadId?{...t,replyMsg:msg}:t);
    saveTracker();
    document.getElementById("repText_"+leadId).textContent = msg;
    document.getElementById("repBox_"+leadId).style.display = "block";
  } catch(e) { showToast("Failed: "+e.message,"error"); }
  btn.textContent="💬 Generate Reply"; btn.disabled=false;
}
function copyRep(id) { const l=tracker.find(t=>t.id===id); if(l?.replyMsg){navigator.clipboard.writeText(l.replyMsg);showToast("Copied!");} }

// ── CSV EXPORT ────────────────────────────────────────
function exportCSV() {
  if (!tracker.length) { showToast("No leads to export!","error"); return; }
  const h = ["Name","Title","Company","Location","Platform","Phone","Status","Category","Date","Message","Notes","URL"];
  const rows = tracker.map(l=>[l.name,l.title,l.company,l.location,l.platform,l.phone||"",l.status,l.category,l.date,(l.message||"").replace(/"/g,'""'),(l.notes||"").replace(/"/g,'""'),l.searchUrl||""].map(v=>`"${v}"`).join(","));
  const csv = [h.join(","),...rows].join("\n");
  const a = document.createElement("a");
  a.href = URL.createObjectURL(new Blob([csv],{type:"text/csv"}));
  a.download = `aedica-leads-${new Date().toLocaleDateString("en-ZA").replace(/\//g,"-")}.csv`;
  a.click();
  showToast("CSV downloaded! 📊");
}

// ── TRACKER HELPERS ───────────────────────────────────
function updateStatus(id,status) {
  tracker=tracker.map(t=>t.id===id?{...t,status}:t);
  saveTracker(); renderTracker();
  if (status==="Follow Up") { const l=tracker.find(t=>t.id===id); if(l)scheduleFollowUpReminder(l.name); }
}
function updateNotes(id,notes) { tracker=tracker.map(t=>t.id===id?{...t,notes}:t); saveTracker(); }
function removeLead(id) { tracker=tracker.filter(t=>t.id!==id); saveTracker(); renderTracker(); renderAll(); showToast("Removed."); }
function saveTracker() { localStorage.setItem(STORAGE_KEY,JSON.stringify(tracker)); }
function copyText(enc,msg) { navigator.clipboard.writeText(decodeURIComponent(enc)); showToast(msg); }
</script>
</body>
</html>
