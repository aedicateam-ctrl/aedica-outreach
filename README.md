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
  @keyframes progress{from{width:0%}to{width:100%}}
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
  .progress-bar{height:3px;background:#f97316;border-radius:2px;transition:width 0.3s}
  .wa-btn{background:#25d366;color:#fff;border:none;border-radius:8px;padding:8px 14px;font-size:12px;font-weight:700;cursor:pointer;font-family:inherit;display:flex;align-items:center;gap:6px}
  .wa-btn:active{opacity:0.8;transform:scale(0.97)}
  .floating-bar{position:fixed;bottom:0;left:0;right:0;background:#0f0f0f;border-top:1px solid #1a1a1a;padding:10px 16px;display:flex;gap:8px;z-index:200}
</style>
</head>
<body>

<!-- SETUP -->
<div id="setupScreen" style="display:none;min-height:100vh;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:30px">
  <div style="font-family:'Syne',sans-serif;font-weight:800;font-size:28px;color:#f97316;margin-bottom:8px">AEDICA</div>
  <div style="font-size:11px;color:#444;letter-spacing:2px;text-transform:uppercase;margin-bottom:40px">Outreach Engine</div>
  <div style="width:100%;max-width:400px;background:#111;border:1px solid #1c1c1c;border-radius:16px;padding:24px">
    <div style="font-size:15px;font-weight:600;margin-bottom:6px">Gemini API Key</div>
    <div style="font-size:12px;color:#555;margin-bottom:16px;line-height:1.6">Free at <a href="https://aistudio.google.com" target="_blank" style="color:#f97316">aistudio.google.com</a> → Get API Key</div>
    <input id="apiKeyInput" type="password" placeholder="AIzaSy..."
      style="width:100%;background:#0d0d0d;border:1px solid #222;border-radius:10px;padding:12px;color:#f0ede8;font-size:14px;font-family:inherit;margin-bottom:12px"/>
    <button class="btn" onclick="saveApiKey()"
      style="width:100%;background:#f97316;border:none;color:#fff;padding:13px;border-radius:10px;font-size:14px;font-weight:700;font-family:inherit">
      Let's Go →
    </button>
    <div id="keyError" style="color:#ef4444;font-size:12px;margin-top:10px;display:none">Please enter your API key.</div>
  </div>
</div>

<!-- MAIN APP -->
<div id="mainApp" style="display:none">

  <div id="toast" style="display:none;position:fixed;top:16px;left:50%;transform:translateX(-50%);color:#fff;padding:10px 20px;border-radius:10px;z-index:9999;font-size:13px;font-weight:600;box-shadow:0 4px 24px rgba(0,0,0,0.5);white-space:nowrap;max-width:90vw;text-align:center"></div>

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
      <div style="font-size:11px;color:#383838;letter-spacing:1.5px;text-transform:uppercase;margin-bottom:14px">Tap a category → 20 leads auto-found & messaged</div>
      <div id="categoryGrid" style="display:block"></div>
    </div>
    <div id="trackerPanel" style="display:none">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px">
        <div id="filterRow" style="display:flex;gap:6px;overflow-x:auto;padding-bottom:4px;flex:1"></div>
        <button class="btn" onclick="exportCSV()" style="background:#1a1a1a;border:1px solid #222;color:#10b981;padding:6px 12px;border-radius:8px;font-size:11px;font-weight:700;font-family:inherit;flex-shrink:0;margin-left:8px">📊 Export CSV</button>
      </div>
      <div id="emptyTracker" style="text-align:center;padding:50px;color:#222;display:none">
        <div style="font-size:36px;margin-bottom:10px">📭</div>
        <div style="font-size:13px">No leads yet. Go find some.</div>
      </div>
      <div id="trackerList" style="display:flex;flex-direction:column;gap:10px"></div>
    </div>
  </div>

  <!-- CATEGORY VIEW -->
  <div id="categoryView" style="display:none;padding:0 16px">
    <div style="padding:16px 0 10px">
      <div id="catIcon" style="font-size:26px"></div>
      <div id="catLabel" style="font-family:'Syne',sans-serif;font-weight:800;font-size:22px;margin-top:6px"></div>
      <div style="font-size:11px;color:#383838;margin-top:4px;letter-spacing:1px;text-transform:uppercase">Worldwide · Full Internet Hunt</div>
    </div>

    <!-- Progress -->
    <div id="progressWrap" style="display:none;margin-bottom:16px">
      <div style="display:flex;justify-content:space-between;margin-bottom:6px">
        <div id="progressLabel" style="font-size:12px;color:#f97316;font-weight:600"></div>
        <div id="progressCount" style="font-size:12px;color:#444"></div>
      </div>
      <div style="background:#1a1a1a;border-radius:4px;height:4px">
        <div id="progressBar" class="progress-bar" style="width:0%"></div>
      </div>
    </div>

    <div id="searchingState" style="display:none;text-align:center;padding:40px 20px">
      <div class="spinning" style="font-size:32px;margin-bottom:16px">🔍</div>
      <div style="font-size:14px;color:#f97316;font-weight:600">Hunting 20 leads across the internet...</div>
      <div id="searchLog" style="font-size:12px;color:#2a2a2a;margin-top:8px"></div>
    </div>

    <!-- WA Blast Bar -->
    <div id="waBlastBar" style="display:none;background:#0d2b1a;border:1px solid #25d36633;border-radius:12px;padding:12px 14px;margin-bottom:14px;display:none">
      <div style="font-size:11px;color:#25d366;font-weight:700;letter-spacing:1px;text-transform:uppercase;margin-bottom:8px">WhatsApp Blast Ready</div>
      <div id="waBlastList" style="display:flex;flex-direction:column;gap:6px"></div>
    </div>

    <div id="resultsList" style="display:flex;flex-direction:column;gap:12px"></div>
  </div>

  <!-- FLOATING BAR (category view) -->
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
  // TRADESPEOPLE
  {id:"plumbers",label:"Plumbers",icon:"🔧",query:"plumber self employed independent",angle:"Aedica gets you more clients and pays you safely via escrow — no more chasing invoices."},
  {id:"electricians",label:"Electricians",icon:"⚡",query:"electrician self employed independent contractor",angle:"More jobs, secure payment, no middleman — that's Aedica."},
  {id:"tilers",label:"Tilers",icon:"🏠",query:"tiler flooring installer self employed",angle:"Aedica connects you directly with homeowners who need tiling done now."},
  {id:"hvac",label:"HVAC / Aircon",icon:"❄️",query:"HVAC aircon installer self employed",angle:"Homeowners need you — Aedica makes sure you get found and get paid."},
  {id:"builders",label:"Builders",icon:"🏗️",query:"builder construction contractor self employed",angle:"Aedica helps you find more building jobs and get paid securely every time."},
  {id:"painters",label:"Painters",icon:"🎨",query:"painter decorator self employed independent",angle:"Find more painting jobs through Aedica — homeowners post jobs, you get paid safely."},
  {id:"solar",label:"Solar Installers",icon:"☀️",query:"solar panel installer self employed",angle:"Solar is booming — Aedica connects you with homeowners ready to install now."},
  {id:"roofers",label:"Roofers",icon:"🏚️",query:"roofer roofing contractor self employed",angle:"Aedica gets you roofing jobs with secure payment — no more bad debt."},
  {id:"landscapers",label:"Landscapers",icon:"🌿",query:"landscaper gardener self employed",angle:"Homeowners need reliable garden services — Aedica puts you in front of them."},
  {id:"pool",label:"Pool Services",icon:"🏊",query:"pool installer cleaner self employed",angle:"Pool installs and maintenance — Aedica connects you with homeowners who need you."},
  {id:"security",label:"Security Installers",icon:"📹",query:"CCTV alarm security installer self employed",angle:"Security is in demand — Aedica helps you get more installations booked."},
  {id:"carpenters",label:"Carpenters",icon:"🪵",query:"carpenter joiner cabinet maker self employed",angle:"Find more carpentry jobs through Aedica — homeowners post jobs daily."},
  {id:"waterproofing",label:"Waterproofing",icon:"💧",query:"waterproofing contractor self employed",angle:"Aedica connects you with homeowners dealing with leaks right now."},
  {id:"pest",label:"Pest Control",icon:"🐛",query:"pest control exterminator self employed",angle:"Homeowners need pest control urgently — Aedica puts you first in line."},
  {id:"cleaning",label:"Cleaning Services",icon:"🧹",query:"cleaning service domestic worker self employed",angle:"Regular cleaning clients through Aedica — steady income, secure payments."},
  {id:"handyman",label:"Handymen",icon:"🛠️",query:"handyman odd jobs self employed",angle:"Aedica matches you with homeowners who need exactly what you can do."},
  {id:"gates",label:"Gate & Fence",icon:"🚪",query:"gate fence installer self employed",angle:"Gate and fence jobs are everywhere — Aedica connects you with homeowners."},
  {id:"movers",label:"Movers",icon:"🚚",query:"moving company removals self employed",angle:"Homeowners moving house need reliable movers — Aedica puts you on their radar."},
  {id:"locksmiths",label:"Locksmiths",icon:"🔑",query:"locksmith self employed independent",angle:"Aedica connects you with homeowners who need your services urgently."},
  {id:"glaziers",label:"Glaziers",icon:"🪟",query:"glazier window fitter self employed",angle:"Broken windows, new installs — Aedica connects you with jobs daily."},
  {id:"borehole",label:"Borehole & Water",icon:"💦",query:"borehole drilling water tank installer",angle:"Water solutions are in demand — Aedica helps you reach homeowners who need you."},
  {id:"interior",label:"Interior Decorators",icon:"🛋️",query:"interior decorator designer self employed",angle:"Homeowners looking for interior upgrades — Aedica connects you with them."},
  // BUSINESS & PARTNERS
  {id:"architects",label:"Architects",icon:"📐",query:"architect interior designer South Africa",angle:"Your clients need contractors — Aedica makes that connection seamless."},
  {id:"estate_agents",label:"Estate Agents",icon:"🏡",query:"real estate agent property South Africa",angle:"Your clients need contractors — Aedica connects them. Let's partner."},
  {id:"property_developers",label:"Property Developers",icon:"🏢",query:"property developer South Africa",angle:"Aedica streamlines contractor sourcing for your developments."},
  {id:"body_corporates",label:"Body Corporates",icon:"🏘️",query:"body corporate property manager South Africa",angle:"Managing a complex? Aedica gives you vetted contractors instantly."},
  {id:"airbnb_hosts",label:"Airbnb Hosts",icon:"🛎️",query:"Airbnb host superhost South Africa",angle:"Need reliable contractors fast? Aedica is built for SA hosts like you."},
  {id:"hardware_stores",label:"Hardware Stores",icon:"🔨",query:"hardware store owner South Africa",angle:"Your customers are contractors. Aedica helps them get more jobs."},
  {id:"stokvel",label:"Stokvel Groups",icon:"🤝",query:"stokvel community savings group South Africa",angle:"Aedica helps your members find trusted tradespeople and save on home services."},
  {id:"church",label:"Church Leaders",icon:"⛪",query:"pastor church leader community South Africa",angle:"Your congregation needs home services — Aedica is the trusted platform to recommend."},
  {id:"township",label:"Township Entrepreneurs",icon:"🏘️",query:"township entrepreneur small business South Africa",angle:"Aedica is built for SA — helping informal workers and homeowners connect safely."},
  // FINANCE & INVESTMENT
  {id:"bank_managers",label:"Bank Managers",icon:"🏦",query:"bank manager business banking South Africa FNB Absa Nedbank",angle:"Aedica's escrow wallet and loan calculator — let's explore a banking partnership."},
  {id:"fintech",label:"Fintech & Investors",icon:"💳",query:"fintech startup investor South Africa",angle:"Aedica is at MVP stage — looking for strategic partners and early backers."},
  {id:"insurance",label:"Insurance Brokers",icon:"🛡️",query:"home insurance broker South Africa",angle:"When a client claims, they need a contractor fast. Aedica solves that."},
  {id:"investors",label:"Investors & VCs",icon:"📈",query:"venture capital investor South Africa startup",angle:"Aedica is building fast — open to early investor conversations."},
  {id:"accelerators",label:"Accelerators",icon:"🚀",query:"startup accelerator incubator South Africa",angle:"Aedica is MVP-ready and looking for the right programme to scale with."},
  // MEDIA & INFLUENCE
  {id:"tech_founders",label:"Tech Founders",icon:"💻",query:"tech startup founder South Africa",angle:"Building Aedica — a home services marketplace in SA. Would love your perspective."},
  {id:"media",label:"Tech Media",icon:"📰",query:"tech journalist South Africa TechCentral Ventureburn",angle:"Building something new in SA's home services space — worth a story?"},
  {id:"influencers",label:"SA Influencers",icon:"📢",query:"South Africa business influencer entrepreneur",angle:"Aedica connects homeowners with vetted tradespeople — the SA market needs this."},
  {id:"property_lawyers",label:"Property Attorneys",icon:"⚖️",query:"property attorney conveyancer South Africa",angle:"Aedica is building the infrastructure for home services in SA — let's connect."},
  {id:"seda",label:"SEDA & Gov",icon:"🏛️",query:"SEDA small enterprise development South Africa funding",angle:"Aedica creates work for tradespeople — exploring funding alignment."},
];

const STATUS_OPTIONS = ["Sent","Replied","Not Interested","Follow Up","Partnership"];
const STATUS_COLORS = {Sent:"#f59e0b",Replied:"#10b981","Not Interested":"#ef4444","Follow Up":"#6366f1",Partnership:"#ec4899"};
const PICONS = {LinkedIn:"💼",Instagram:"📸",Facebook:"👥",Twitter:"🐦",TikTok:"🎵",YouTube:"▶️",Gumtree:"🟢",Website:"🌐",WhatsApp:"💬",Google:"🔎"};
const STORAGE_KEY = "aedica_v4";

let tracker = [];
let currentCategory = null;
let currentResults = [];
let activeTab = "search";
let filterStatus = "All";
let generatedMessages = {};
let isAutoGenerating = false;

// ── INIT ──────────────────────────────────────────────
window.addEventListener('load', () => {
  const key = localStorage.getItem("aedica_gemini_key");
  if (key) {
    document.getElementById("setupScreen").style.display = "none";
    document.getElementById("mainApp").style.display = "block";
    initApp();
  } else {
    document.getElementById("setupScreen").style.display = "flex";
    document.getElementById("mainApp").style.display = "none";
  }
});

function saveApiKey() {
  const key = document.getElementById("apiKeyInput").value.trim();
  if (!key) { document.getElementById("keyError").style.display = "block"; return; }
  localStorage.setItem("aedica_gemini_key", key);
  document.getElementById("setupScreen").style.display = "none";
  document.getElementById("mainApp").style.display = "block";
  initApp();
}

function getKey() { return localStorage.getItem("aedica_gemini_key") || ""; }

function initApp() {
  try { tracker = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]"); } catch { tracker = []; }
  buildCategoryGrid();
  renderAll();
  requestNotificationPermission();
  checkDailySummary();
}

// ── NOTIFICATIONS ─────────────────────────────────────
function requestNotificationPermission() {
  if ("Notification" in window && Notification.permission === "default") {
    Notification.requestPermission();
  }
}

function sendNotification(title, body) {
  if ("Notification" in window && Notification.permission === "granted") {
    new Notification(title, {body, icon: "https://aedica.co.za/favicon.ico"});
  }
}

function checkDailySummary() {
  const lastCheck = localStorage.getItem("aedica_last_summary");
  const today = new Date().toDateString();
  if (lastCheck === today) return;
  localStorage.setItem("aedica_last_summary", today);
  try { tracker = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]"); } catch { return; }
  const followUps = tracker.filter(t => t.status === "Follow Up").length;
  const total = tracker.length;
  const replied = tracker.filter(t => t.status === "Replied").length;
  if (total > 0) {
    setTimeout(() => {
      sendNotification("Aedica Daily Summary",
        `${total} leads tracked. ${replied} replied. ${followUps} follow-ups waiting.`);
      if (followUps > 0) showToast(`You have ${followUps} follow-up${followUps>1?"s":""} waiting!`, "error");
    }, 2000);
  }
}

function scheduleFollowUpReminder(leadName, days=3) {
  const ms = days * 24 * 60 * 60 * 1000;
  setTimeout(() => {
    sendNotification("Follow-up reminder", `Time to follow up with ${leadName} on Aedica outreach!`);
  }, ms);
}

// ── TOAST ─────────────────────────────────────────────
function showToast(msg, type="success") {
  const t = document.getElementById("toast");
  t.textContent = msg;
  t.style.background = type === "error" ? "#ef4444" : "#f97316";
  t.style.display = "block";
  setTimeout(() => t.style.display = "none", 3500);
}

// ── NAVIGATION ────────────────────────────────────────
function goHome() {
  document.getElementById("homeView").style.display = "block";
  document.getElementById("categoryView").style.display = "none";
  document.getElementById("backBtn").style.display = "none";
  document.getElementById("floatingBar").style.display = "none";
  renderAll();
}

function showCategoryView(cat) {
  currentCategory = cat;
  currentResults = [];
  generatedMessages = {};
  document.getElementById("homeView").style.display = "none";
  document.getElementById("categoryView").style.display = "block";
  document.getElementById("backBtn").style.display = "inline-block";
  document.getElementById("catIcon").textContent = cat.icon;
  document.getElementById("catLabel").textContent = cat.label;
  document.getElementById("resultsList").innerHTML = "";
  document.getElementById("floatingBar").style.display = "none";
  searchLeads(cat, false);
}

function setTab(tab) {
  activeTab = tab;
  const s = tab === "search";
  document.getElementById("searchPanel").style.display = s ? "block" : "none";
  document.getElementById("trackerPanel").style.display = s ? "none" : "block";
  document.getElementById("tabSearch").style.cssText += s ? ";background:#f97316;color:#fff;border-color:#f97316" : ";background:#161616;color:#555;border-color:#222";
  document.getElementById("tabTracker").style.cssText += s ? ";background:#161616;color:#555;border-color:#222" : ";background:#f97316;color:#fff;border-color:#f97316";
  if (tab === "tracker") renderTracker();
}

// ── BUILD GRID ────────────────────────────────────────
function buildCategoryGrid() {
  const grid = document.getElementById("categoryGrid");
  const sections = [
    {label:"🔨 Tradespeople", ids:["plumbers","electricians","tilers","hvac","builders","painters","solar","roofers","landscapers","pool","security","carpenters","waterproofing","pest","cleaning","handyman","gates","movers","locksmiths","glaziers","borehole","interior"]},
    {label:"🤝 Business & Partners", ids:["architects","estate_agents","property_developers","body_corporates","airbnb_hosts","hardware_stores","stokvel","church","township"]},
    {label:"💰 Finance & Investment", ids:["bank_managers","fintech","insurance","investors","accelerators"]},
    {label:"📣 Media & Influence", ids:["tech_founders","media","influencers","property_lawyers","seda"]},
  ];
  grid.innerHTML = sections.map(section => {
    const cats = CATEGORIES.filter(c => section.ids.includes(c.id));
    return `<div style="margin-bottom:20px">
      <div style="font-size:11px;color:#444;letter-spacing:1.5px;text-transform:uppercase;margin-bottom:10px;font-weight:600">${section.label}</div>
      <div style="display:grid;grid-template-columns:1fr 1fr;gap:8px">
        ${cats.map(cat => `
          <button class="card btn" onclick="showCategoryView(CATEGORIES.find(c=>c.id==='${cat.id}'))"
            style="background:#111;border:1px solid #1c1c1c;border-radius:12px;padding:14px 12px;text-align:left;display:flex;flex-direction:column;gap:6px;font-family:inherit">
            <span style="font-size:20px">${cat.icon}</span>
            <span style="font-size:12px;font-weight:600;color:#e0ddd8;line-height:1.4">${cat.label}</span>
          </button>`).join("")}
      </div>
    </div>`;
  }).join("");
}

// ── RENDER ALL ────────────────────────────────────────
function renderAll() {
  const followUps = tracker.filter(t => t.status === "Follow Up");
  const badge = document.getElementById("followUpBadge");
  badge.style.display = followUps.length > 0 ? "block" : "none";
  if (followUps.length > 0) badge.textContent = followUps.length + " follow-up" + (followUps.length > 1 ? "s" : "");
  document.getElementById("trackerCount").textContent = tracker.length + " tracked";
  document.getElementById("tabTracker").textContent = "📋 Tracker (" + tracker.length + ")";
  if (activeTab === "tracker") { document.getElementById("tabTracker").style.background = "#f97316"; document.getElementById("tabTracker").style.color = "#fff"; }
  const statsRow = document.getElementById("statsRow");
  if (tracker.length > 0) {
    statsRow.style.display = "flex";
    statsRow.innerHTML = ["Sent","Replied","Follow Up","Partnership"].map(s => `
      <div style="background:#111;border:1px solid ${STATUS_COLORS[s]}44;border-radius:10px;padding:10px 14px;min-width:80px;text-align:center;flex-shrink:0">
        <div style="font-size:18px;font-weight:700;color:${STATUS_COLORS[s]}">${tracker.filter(t=>t.status===s).length}</div>
        <div style="font-size:10px;color:#444;margin-top:2px">${s}</div>
      </div>`).join("");
  } else statsRow.style.display = "none";
  if (activeTab === "tracker") renderTracker();
}

// ── RENDER TRACKER ────────────────────────────────────
function renderTracker() {
  const filterRow = document.getElementById("filterRow");
  filterRow.innerHTML = ["All",...STATUS_OPTIONS].map(s => {
    const count = s==="All" ? tracker.length : tracker.filter(t=>t.status===s).length;
    const active = filterStatus===s;
    const col = STATUS_COLORS[s]||"#f97316";
    return `<button class="btn" onclick="setFilter('${s}')"
      style="padding:5px 13px;border-radius:20px;font-size:11px;font-weight:600;background:${active?col:"#161616"};color:${active?"#fff":"#555"};border:1px solid ${active?col:"#222"};white-space:nowrap;flex-shrink:0;font-family:inherit">
      ${s} (${count})</button>`;
  }).join("");
  const filtered = filterStatus==="All" ? tracker : tracker.filter(t=>t.status===filterStatus);
  const list = document.getElementById("trackerList");
  const empty = document.getElementById("emptyTracker");
  if (!filtered.length) { empty.style.display="block"; list.innerHTML=""; return; }
  empty.style.display = "none";
  list.innerHTML = filtered.map(lead => `
    <div class="fade-in" style="background:#111;border:1px solid ${STATUS_COLORS[lead.status]}33;border-radius:12px;padding:14px">
      <div style="display:flex;justify-content:space-between;margin-bottom:10px">
        <div>
          <div style="font-weight:600;font-size:14px">${lead.name}</div>
          <div style="font-size:12px;color:#666;margin-top:2px">${lead.title} · ${lead.company}</div>
          <div style="font-size:11px;color:#383838;margin-top:2px">${lead.location} · ${lead.category} · ${lead.date}</div>
          ${lead.phone ? `<div style="margin-top:4px"><a href="https://wa.me/${lead.phone.replace(/\D/g,'')}" target="_blank" class="wa-btn" style="display:inline-flex;font-size:11px;padding:5px 10px">💬 WhatsApp ${lead.phone}</a></div>` : ""}
        </div>
        <button class="btn" onclick="removeLead('${lead.id}')" style="background:none;border:none;color:#2a2a2a;font-size:18px;align-self:flex-start;font-family:inherit">✕</button>
      </div>
      <div style="display:flex;gap:6px;flex-wrap:wrap;margin-bottom:10px">
        ${STATUS_OPTIONS.map(s => `
          <button class="btn" onclick="updateStatus('${lead.id}','${s}')"
            style="padding:4px 10px;border-radius:20px;font-size:11px;font-weight:600;background:${lead.status===s?STATUS_COLORS[s]:"#1a1a1a"};color:${lead.status===s?"#fff":"#383838"};border:1px solid ${lead.status===s?STATUS_COLORS[s]:"#222"};font-family:inherit">
            ${s}</button>`).join("")}
      </div>
      ${lead.searchUrl ? `<a href="${lead.searchUrl}" target="_blank" style="display:inline-block;margin-bottom:8px;color:#f97316;font-size:12px;font-weight:600">${PICONS[lead.platform]||"🔗"} Open on ${lead.platform}</a>` : ""}
      ${lead.message ? `
        <div style="background:#0d0d0d;border:1px solid #1a1a1a;border-radius:8px;padding:10px;margin-bottom:8px">
          <div style="font-size:12px;color:#777;line-height:1.6">${lead.message}</div>
          <div style="display:flex;gap:8px;margin-top:8px;flex-wrap:wrap">
            <button class="btn" onclick="copyText('${encodeURIComponent(lead.message)}','Copied!')" style="background:none;border:1px solid #f97316;color:#f97316;padding:5px 12px;border-radius:6px;font-size:11px;font-weight:600;font-family:inherit">Copy</button>
            ${lead.phone ? `<a href="https://wa.me/${lead.phone.replace(/\D/g,'')}?text=${encodeURIComponent(lead.message)}" target="_blank" class="wa-btn" style="font-size:11px;padding:5px 10px">💬 Send via WhatsApp</a>` : ""}
          </div>
        </div>` : ""}
      ${lead.status==="Follow Up" ? `
        <button class="btn" onclick="generateFollowUp('${lead.id}')" id="fuBtn_${lead.id}"
          style="background:#6366f1;border:none;color:#fff;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600;font-family:inherit;margin-bottom:8px">
          🔁 Generate Follow-Up</button>
        <div id="fuBox_${lead.id}" style="display:none;background:#0d0d0d;border:1px solid #6366f133;border-radius:8px;padding:10px;margin-bottom:8px">
          <div id="fuText_${lead.id}" style="font-size:12px;color:#aaa;line-height:1.6"></div>
          <button class="btn" onclick="copyFu('${lead.id}')" style="margin-top:8px;background:none;border:1px solid #6366f1;color:#6366f1;padding:5px 12px;border-radius:6px;font-size:11px;font-weight:600;font-family:inherit">Copy Follow-Up</button>
        </div>` : ""}
      ${lead.status==="Replied" && !lead.replyMsg ? `
        <button class="btn" onclick="generateReply('${lead.id}')" id="repBtn_${lead.id}"
          style="background:#10b981;border:none;color:#fff;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600;font-family:inherit;margin-bottom:8px">
          💬 Generate Reply Message</button>
        <div id="repBox_${lead.id}" style="display:none;background:#0d0d0d;border:1px solid #10b98133;border-radius:8px;padding:10px;margin-bottom:8px">
          <div id="repText_${lead.id}" style="font-size:12px;color:#aaa;line-height:1.6"></div>
          <button class="btn" onclick="copyRep('${lead.id}')" style="margin-top:8px;background:none;border:1px solid #10b981;color:#10b981;padding:5px 12px;border-radius:6px;font-size:11px;font-weight:600;font-family:inherit">Copy Reply</button>
        </div>` : ""}
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
  document.getElementById("searchLog").textContent = "Scanning entire internet for 20 real leads...";

  const alreadyFound = append ? currentResults.map(r=>r.name).join(", ") : "";

  const prompt = `You are a lead generation expert for Aedica (aedica.co.za) — a home services marketplace. Founder Xolani Mthethwa needs to reach REAL people who do ${category.label} work.

CRITICAL: Search the ENTIRE internet — Facebook, Instagram, TikTok, YouTube, Gumtree, Junk Mail, OLX, local business directories, WhatsApp community pages, Google Business, personal websites, ANY platform.

Search query: "${category.query}"

Find people who are:
- Self-employed or informal tradespeople  
- Small one-person operations
- Anyone posting their trade work on social media
- People listed on classifieds or trade directories
- Anyone with a Facebook page, Instagram, or TikTok showing their work
- WORLDWIDE, but prioritise South Africa

VERY IMPORTANT: For each lead, try to find their WhatsApp number or phone number from their listings, Facebook page, or website. SA numbers start with +27, 06x, 07x, 08x.

${append ? `Already found (do NOT repeat): ${alreadyFound}` : ""}

Return EXACTLY 20 leads as a raw JSON array. ONLY valid JSON, no markdown, no backticks.

[{"name":"Name","title":"Exact trade","company":"Business or Self-employed","location":"City, Country","platform":"Facebook","searchUrl":"https://direct-url-or-google-search","phone":"+27821234567 or empty string if not found","notes":"Specific detail about them — what they post, where found, follower count etc"}]

Platform options: LinkedIn, Instagram, Facebook, TikTok, YouTube, Gumtree, WhatsApp, Website, Google
For searchUrl: direct profile URL if known, else https://www.google.com/search?q=NAME+TRADE+LOCATION`;

  try {
    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${getKey()}`,
      {
        method:"POST",
        headers:{"Content-Type":"application/json"},
        body: JSON.stringify({
          contents:[{parts:[{text:prompt}]}],
          tools:[{google_search:{}}],
          generationConfig:{temperature:0.4}
        })
      }
    );
    const data = await response.json();
    if (data.error) throw new Error(data.error.message);
    const text = data.candidates?.[0]?.content?.parts?.map(p=>p.text||"").join("")||"";
    let leads = [];
    const match = text.match(/\[[\s\S]*\]/);
    if (match) { try { leads = JSON.parse(match[0]); } catch(e) {} }

    leads = leads.map(lead => ({
      ...lead,
      phone: lead.phone || "",
      searchUrl: lead.searchUrl?.startsWith("http") ? lead.searchUrl
        : `https://www.google.com/search?q=${encodeURIComponent((lead.name||"")+" "+(lead.title||"")+" "+(lead.location||""))}`,
      platform: lead.platform || "Google"
    }));

    if (!leads.length) {
      showToast("Try again — AI had trouble parsing results", "error");
      document.getElementById("searchingState").style.display = "none";
      return;
    }

    if (append) {
      const startIndex = currentResults.length;
      currentResults = [...currentResults, ...leads];
      appendResults(leads, category, startIndex);
    } else {
      currentResults = leads;
      renderResults(leads, category);
    }

    // Auto-track all leads
    autoTrackAll(leads, category);

    // Build WA blast bar
    buildWABlast(currentResults);

    // Show floating bar
    document.getElementById("floatingBar").style.display = "flex";

    // Auto-generate messages in background
    autoGenerateMessages(leads, category, append ? currentResults.length - leads.length : 0);

  } catch(e) {
    showToast("Search failed: " + e.message, "error");
    document.getElementById("resultsList").innerHTML = `<div style="text-align:center;padding:40px;color:#444">Something went wrong. Check your API key or try again.</div>`;
  }
  document.getElementById("searchingState").style.display = "none";
}

// ── AUTO TRACK ALL ────────────────────────────────────
function autoTrackAll(leads, category) {
  let added = 0;
  leads.forEach(lead => {
    const exists = tracker.find(t => t.name===lead.name && t.company===lead.company);
    if (!exists) {
      tracker.unshift({
        id: Date.now()+Math.random(),
        name:lead.name, title:lead.title, company:lead.company,
        location:lead.location, platform:lead.platform,
        phone:lead.phone||"",
        category:category.label, status:"Sent",
        date:new Date().toLocaleDateString("en-ZA"),
        notes:"", message:"", searchUrl:lead.searchUrl||""
      });
      added++;
      if (lead.phone) scheduleFollowUpReminder(lead.name, 3);
    }
  });
  saveTracker();
  renderAll();
  if (added > 0) showToast(added + " leads auto-tracked! 🎯");
}

// ── BUILD WA BLAST ────────────────────────────────────
function buildWABlast(leads) {
  const withPhone = leads.filter(l => l.phone && l.phone.length > 6);
  const bar = document.getElementById("waBlastBar");
  const list = document.getElementById("waBlastList");
  if (!withPhone.length) { bar.style.display="none"; return; }
  bar.style.display = "block";
  list.innerHTML = `
    <div style="font-size:12px;color:#888;margin-bottom:6px">${withPhone.length} leads with WhatsApp numbers found:</div>
    ${withPhone.map(l => `
      <a href="https://wa.me/${l.phone.replace(/\D/g,'')}?text=${encodeURIComponent('Hi '+l.name+', ')}" target="_blank"
        class="wa-btn" style="font-size:12px;text-decoration:none;justify-content:flex-start">
        💬 ${l.name} (${l.phone})
      </a>`).join("")}`;
}

// ── AUTO-GENERATE MESSAGES ────────────────────────────
async function autoGenerateMessages(leads, category, startOffset=0) {
  if (isAutoGenerating) return;
  isAutoGenerating = true;
  const wrap = document.getElementById("progressWrap");
  const label = document.getElementById("progressLabel");
  const countEl = document.getElementById("progressCount");
  const bar = document.getElementById("progressBar");
  wrap.style.display = "block";

  for (let i = 0; i < leads.length; i++) {
    const globalIndex = startOffset + i;
    label.textContent = "Auto-writing messages...";
    countEl.textContent = `${i+1} / ${leads.length}`;
    bar.style.width = ((i+1)/leads.length*100)+"%";

    if (generatedMessages[globalIndex]) continue;

    const lead = leads[i];
    const platform = lead.platform || "Facebook";
    const prompt = `Write 3 short outreach messages from Xolani Mthethwa, founder of Aedica (aedica.co.za) — a South African home services marketplace where homeowners hire and pay tradespeople safely via escrow.

Target: ${lead.name}, ${lead.title} at ${lead.company}, ${lead.location}. Found on ${platform}.
Context: ${lead.notes}
Angle: ${category.angle}

3 styles:
1. CASUAL — like texting a mate, warm, short
2. DIRECT — 2 sentences, straight value proposition  
3. STORY — start with a SA pain point (not getting paid, hard to find clients), then offer Aedica

Rules: Start with their name. Mention aedica.co.za once. Sound human. Tailor tone for ${platform}. No subject line.

Return ONLY raw JSON array, no markdown:
[{"style":"Casual","message":"..."},{"style":"Direct","message":"..."},{"style":"Story","message":"..."}]`;

    try {
      const r = await fetch(
        `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${getKey()}`,
        {
          method:"POST",
          headers:{"Content-Type":"application/json"},
          body:JSON.stringify({
            contents:[{parts:[{text:prompt}]}],
            generationConfig:{temperature:0.85, maxOutputTokens:600}
          })
        }
      );
      const d = await r.json();
      if (d.error) continue;
      const raw = d.candidates?.[0]?.content?.parts?.map(p=>p.text||"").join("")||"";
      const m = raw.match(/\[[\s\S]*\]/);
      if (m) {
        try {
          const variants = JSON.parse(m[0]);
          generatedMessages[globalIndex] = variants;
          // Update the message box if rendered
          updateLeadMessageBox(globalIndex, lead, variants);
          // Save first variant as tracker message
          const trackerEntry = tracker.find(t => t.name===lead.name && t.company===lead.company);
          if (trackerEntry && !trackerEntry.message) {
            trackerEntry.message = variants[0].message;
            saveTracker();
          }
        } catch(e) {}
      }
    } catch(e) {}
    await new Promise(res => setTimeout(res, 800));
  }

  label.textContent = "✅ All messages ready!";
  countEl.textContent = "";
  setTimeout(() => { wrap.style.display="none"; }, 2000);
  isAutoGenerating = false;
  showToast("All 20 messages generated! 🔥");
}

function updateLeadMessageBox(index, lead, variants) {
  const box = document.getElementById("msgBox_"+index);
  const variantsDiv = document.getElementById("msgVariants_"+index);
  const actionsDiv = document.getElementById("msgActions_"+index);
  const btn = document.getElementById("msgBtn_"+index);
  if (!box || !variantsDiv) return;
  box.style.display = "block";
  if (btn) { btn.textContent = "✅ Message Ready — Pick a Style"; btn.style.background = "#10b981"; }
  variantsDiv.innerHTML = variants.map((v, vi) => `
    <div class="msg-variant ${vi===0?"selected":""}" id="variant_${index}_${vi}"
      onclick="selectVariant(${index},${vi},'${encodeURIComponent(JSON.stringify(variants))}')">
      <div style="font-size:10px;color:#f97316;font-weight:700;letter-spacing:1px;text-transform:uppercase;margin-bottom:6px">${v.style}</div>
      <div style="font-size:13px;color:#aaa;line-height:1.7">${v.message}</div>
    </div>`).join("");
  if (!Array.isArray(generatedMessages[index])) generatedMessages[index] = variants;
  if (actionsDiv) {
    actionsDiv.style.display = "block";
    const copyBtn = document.getElementById("copyBtn_"+index);
    if (copyBtn) copyBtn.onclick = () => {
      const sel = Array.isArray(generatedMessages[index]) ? generatedMessages[index][0].message : generatedMessages[index];
      navigator.clipboard.writeText(sel);
      showToast("Copied! Go send it 🔥");
    };
    // Add WA button if phone
    const waDiv = document.getElementById("waSendBtn_"+index);
    if (waDiv && lead.phone) {
      waDiv.style.display = "flex";
      waDiv.href = `https://wa.me/${lead.phone.replace(/\D/g,'')}?text=`;
    }
  }
}

// ── RENDER RESULTS ────────────────────────────────────
function renderResults(leads, category) {
  document.getElementById("resultsList").innerHTML = "";
  appendResults(leads, category, 0);
}

function appendResults(leads, category, startOffset) {
  const list = document.getElementById("resultsList");
  // Remove old "load more" button if exists
  const oldBtn = document.getElementById("loadMoreBtn");
  if (oldBtn) oldBtn.remove();

  const html = leads.map((lead, i) => {
    const idx = startOffset + i;
    const icon = PICONS[lead.platform]||"🔗";
    const hasPhone = lead.phone && lead.phone.length > 6;
    return `
    <div class="fade-in" id="leadCard_${idx}" style="background:#111;border:1px solid #1c1c1c;border-radius:14px;padding:16px;animation-delay:${i*0.04}s;animation-fill-mode:forwards;opacity:0">
      <div style="margin-bottom:12px">
        <div style="display:flex;align-items:center;gap:8px;margin-bottom:6px">
          <span style="background:#1a1a1a;border:1px solid #222;border-radius:6px;padding:2px 8px;font-size:11px;color:#888">${icon} ${lead.platform||"Google"}</span>
          <span style="font-size:11px;color:#383838">${lead.location}</span>
          ${hasPhone ? `<span style="background:#0d2b1a;border:1px solid #25d36633;border-radius:6px;padding:2px 8px;font-size:11px;color:#25d366">💬 WA</span>` : ""}
        </div>
        <div style="font-weight:700;font-size:15px">${lead.name}</div>
        <div style="font-size:13px;color:#666;margin-top:2px">${lead.title}</div>
        <div style="font-size:12px;color:#383838;margin-top:2px">${lead.company}</div>
        ${lead.notes ? `<div style="font-size:12px;color:#f97316;margin-top:6px;line-height:1.5;opacity:0.8;font-style:italic">"${lead.notes}"</div>` : ""}
        ${hasPhone ? `<div style="margin-top:8px"><a href="https://wa.me/${lead.phone.replace(/\D/g,'')}" target="_blank" class="wa-btn" style="font-size:11px;padding:5px 10px;display:inline-flex">💬 ${lead.phone}</a></div>` : ""}
      </div>
      <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:10px">
        <a href="${lead.searchUrl}" target="_blank"
          style="background:#1a1a1a;border:1px solid #2a2a2a;color:#f97316;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600">
          ${icon} View Profile
        </a>
      </div>

      <div id="msgBox_${idx}" style="display:none">
        <div style="font-size:10px;color:#888;font-weight:600;letter-spacing:1px;text-transform:uppercase;margin-bottom:8px">
          <span class="pulsing">✍️ Writing message...</span>
        </div>
        <div id="msgVariants_${idx}"></div>
        <div id="msgActions_${idx}" style="display:none;margin-top:10px">
          <div style="display:flex;gap:8px;flex-wrap:wrap">
            <button class="btn" id="copyBtn_${idx}"
              style="flex:1;background:#f97316;border:none;color:#fff;padding:10px;border-radius:8px;font-size:13px;font-weight:700;font-family:inherit">
              📋 Copy Message
            </button>
            ${hasPhone ? `
            <a id="waSendBtn_${idx}" href="https://wa.me/${lead.phone.replace(/\D/g,'')}?text=" target="_blank"
              class="wa-btn" style="font-size:12px;padding:10px 14px">
              💬 Send WA
            </a>` : ""}
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
  }).join("");

  list.insertAdjacentHTML("beforeend", html);
}

function selectVariant(index, vi, variantsEnc) {
  const variants = JSON.parse(decodeURIComponent(variantsEnc));
  variants.forEach((_,i) => {
    const el = document.getElementById(`variant_${index}_${i}`);
    if (el) el.classList.toggle("selected", i===vi);
  });
  const selected = variants[vi].message;
  // Update copy button
  const copyBtn = document.getElementById("copyBtn_"+index);
  if (copyBtn) copyBtn.onclick = () => { navigator.clipboard.writeText(selected); showToast("Copied! 🔥"); };
  // Update WA link
  const waBtn = document.getElementById("waSendBtn_"+index);
  if (waBtn) {
    const lead = currentResults[index];
    if (lead && lead.phone) waBtn.href = `https://wa.me/${lead.phone.replace(/\D/g,'')}?text=${encodeURIComponent(selected)}`;
  }
}

async function regenMessage(index, leadEnc, catEnc) {
  const lead = JSON.parse(decodeURIComponent(leadEnc));
  const category = JSON.parse(decodeURIComponent(catEnc));
  const variantsDiv = document.getElementById("msgVariants_"+index);
  const actionsDiv = document.getElementById("msgActions_"+index);
  if (variantsDiv) variantsDiv.innerHTML = `<div class="pulsing" style="color:#555;font-size:12px;padding:10px">Regenerating...</div>`;
  if (actionsDiv) actionsDiv.style.display = "none";
  delete generatedMessages[index];
  await autoGenerateMessages([lead], category, index);
}

// ── FOLLOW-UP GENERATOR ───────────────────────────────
async function generateFollowUp(leadId) {
  const lead = tracker.find(t=>t.id===leadId);
  if (!lead) return;
  const btn = document.getElementById("fuBtn_"+leadId);
  const box = document.getElementById("fuBox_"+leadId);
  const text = document.getElementById("fuText_"+leadId);
  btn.textContent = "Writing..."; btn.disabled = true;
  const prompt = `Write a very short follow-up message from Xolani Mthethwa (Aedica, aedica.co.za) to ${lead.name} who is a ${lead.title}.

Already sent: "${lead.message || "an intro about Aedica"}"
Platform: ${lead.platform}

Follow-up rules:
- 2-3 sentences MAX
- Reference previous message lightly ("just wanted to follow up on...")
- Add ONE new value point (e.g. "We just launched a feature that lets homeowners rate contractors — it helps you stand out")
- End with an easy yes/no question
- Sound like a real human, not pushy`;
  try {
    const r = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${getKey()}`,
      {method:"POST",headers:{"Content-Type":"application/json"},
       body:JSON.stringify({contents:[{parts:[{text:prompt}]}],generationConfig:{temperature:0.8,maxOutputTokens:200}})});
    const d = await r.json();
    if (d.error) throw new Error(d.error.message);
    const msg = d.candidates?.[0]?.content?.parts?.[0]?.text||"";
    tracker = tracker.map(t=>t.id===leadId?{...t,followUp:msg}:t);
    saveTracker();
    text.textContent = msg;
    box.style.display = "block";
    // Update copy button
    document.getElementById("fuBox_"+leadId).querySelector("button").onclick = () => {
      navigator.clipboard.writeText(msg); showToast("Follow-up copied!");
    };
  } catch(e) { showToast("Failed: "+e.message,"error"); }
  btn.textContent = "🔁 Generate Follow-Up"; btn.disabled = false;
}

function copyFu(leadId) {
  const lead = tracker.find(t=>t.id===leadId);
  if (lead && lead.followUp) { navigator.clipboard.writeText(lead.followUp); showToast("Follow-up copied!"); }
}

// ── REPLY GENERATOR ───────────────────────────────────
async function generateReply(leadId) {
  const lead = tracker.find(t=>t.id===leadId);
  if (!lead) return;
  const btn = document.getElementById("repBtn_"+leadId);
  const box = document.getElementById("repBox_"+leadId);
  const text = document.getElementById("repText_"+leadId);
  btn.textContent = "Writing..."; btn.disabled = true;
  const prompt = `Write a short reply message from Xolani Mthethwa (Aedica, aedica.co.za) to ${lead.name} who just replied to an outreach message.

They are a ${lead.title} found on ${lead.platform}.
Original message sent: "${lead.message||"intro about Aedica"}"

Since they replied positively, now write a message that:
- Thanks them warmly and briefly
- Gives them the next clear step (e.g. "Check out aedica.co.za and sign up as a contractor — takes 2 minutes")
- Offers to answer any questions
- Keeps it short (3 sentences max)
- Sounds like a real human`;
  try {
    const r = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${getKey()}`,
      {method:"POST",headers:{"Content-Type":"application/json"},
       body:JSON.stringify({contents:[{parts:[{text:prompt}]}],generationConfig:{temperature:0.8,maxOutputTokens:200}})});
    const d = await r.json();
    if (d.error) throw new Error(d.error.message);
    const msg = d.candidates?.[0]?.content?.parts?.[0]?.text||"";
    tracker = tracker.map(t=>t.id===leadId?{...t,replyMsg:msg}:t);
    saveTracker();
    text.textContent = msg;
    box.style.display = "block";
  } catch(e) { showToast("Failed: "+e.message,"error"); }
  btn.textContent = "💬 Generate Reply"; btn.disabled = false;
}

function copyRep(leadId) {
  const lead = tracker.find(t=>t.id===leadId);
  if (lead && lead.replyMsg) { navigator.clipboard.writeText(lead.replyMsg); showToast("Reply copied!"); }
}

// ── CSV EXPORT ────────────────────────────────────────
function exportCSV() {
  if (!tracker.length) { showToast("No leads to export yet!", "error"); return; }
  const headers = ["Name","Title","Company","Location","Platform","Phone","Status","Category","Date","Message","Notes","Profile URL"];
  const rows = tracker.map(l => [
    l.name, l.title, l.company, l.location, l.platform, l.phone||"",
    l.status, l.category, l.date,
    (l.message||"").replace(/"/g,'""'),
    (l.notes||"").replace(/"/g,'""'),
    l.searchUrl||""
  ].map(v => `"${v}"`).join(","));
  const csv = [headers.join(","), ...rows].join("\n");
  const blob = new Blob([csv], {type:"text/csv"});
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url; a.download = `aedica-leads-${new Date().toLocaleDateString("en-ZA").replace(/\//g,"-")}.csv`;
  a.click(); URL.revokeObjectURL(url);
  showToast("CSV downloaded! Open in Excel or Sheets 📊");
}

// ── TRACKER ACTIONS ───────────────────────────────────
function addToTracker(index) {
  const lead = currentResults[index];
  if (!lead) return;
  const exists = tracker.find(t=>t.name===lead.name&&t.company===lead.company);
  if (exists) { showToast("Already tracked."); return; }
  const msg = Array.isArray(generatedMessages[index]) ? generatedMessages[index][0].message : (generatedMessages[index]||"");
  tracker.unshift({
    id:Date.now()+""+Math.random(), name:lead.name, title:lead.title,
    company:lead.company, location:lead.location, platform:lead.platform,
    phone:lead.phone||"", category:currentCategory.label, status:"Sent",
    date:new Date().toLocaleDateString("en-ZA"), notes:"", message:msg, searchUrl:lead.searchUrl||""
  });
  saveTracker(); showToast(lead.name+" tracked ✓"); renderAll();
}

function updateStatus(id, status) {
  tracker = tracker.map(t=>t.id===id?{...t,status}:t);
  saveTracker(); renderTracker();
  if (status==="Follow Up") {
    const lead = tracker.find(t=>t.id===id);
    if (lead) scheduleFollowUpReminder(lead.name, 3);
  }
}
function updateNotes(id,notes) { tracker=tracker.map(t=>t.id===id?{...t,notes}:t); saveTracker(); }
function removeLead(id) { tracker=tracker.filter(t=>t.id!==id); saveTracker(); renderTracker(); renderAll(); showToast("Removed."); }
function saveTracker() { localStorage.setItem(STORAGE_KEY, JSON.stringify(tracker)); }
function copyText(encoded,msg) { navigator.clipboard.writeText(decodeURIComponent(encoded)); showToast(msg); }
</script>
</body>
</html>
