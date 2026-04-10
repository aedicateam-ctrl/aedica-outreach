<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Aedica Outreach Engine</title>
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=Syne:wght@700;800&display=swap" rel="stylesheet"/>
<style>
  *{box-sizing:border-box;margin:0;padding:0}
  body{background:#0a0a0a;color:#f0ede8;font-family:'DM Sans','Segoe UI',sans-serif;min-height:100vh;padding-bottom:80px}
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
</style>
</head>
<body>

<!-- SETUP SCREEN -->
<div id="setupScreen" style="display:none;min-height:100vh;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:30px">
  <div style="font-family:'Syne',sans-serif;font-weight:800;font-size:28px;color:#f97316;margin-bottom:8px">AEDICA</div>
  <div style="font-size:11px;color:#444;letter-spacing:2px;text-transform:uppercase;margin-bottom:40px">Outreach Engine</div>
  <div style="width:100%;max-width:400px;background:#111;border:1px solid #1c1c1c;border-radius:16px;padding:24px">
    <div style="font-size:15px;font-weight:600;margin-bottom:6px">Enter your Gemini API Key</div>
    <div style="font-size:12px;color:#555;margin-bottom:16px;line-height:1.6">Get a free key at <a href="https://aistudio.google.com" target="_blank" style="color:#f97316">aistudio.google.com</a> → Get API Key. Free, 2000 searches/day.</div>
    <input id="apiKeyInput" type="password" placeholder="AIzaSy..."
      style="width:100%;background:#0d0d0d;border:1px solid #222;border-radius:10px;padding:12px;color:#f0ede8;font-size:14px;font-family:inherit;margin-bottom:12px"/>
    <button class="btn" onclick="saveApiKey()"
      style="width:100%;background:#f97316;border:none;color:#fff;padding:13px;border-radius:10px;font-size:14px;font-weight:700;font-family:inherit">
      Let's Go →
    </button>
    <div id="keyError" style="color:#ef4444;font-size:12px;margin-top:10px;display:none">Please enter your API key first.</div>
  </div>
</div>

<!-- MAIN APP -->
<div id="mainApp" style="display:none">

  <!-- TOAST -->
  <div id="toast" style="display:none;position:fixed;top:16px;left:50%;transform:translateX(-50%);color:#fff;padding:10px 20px;border-radius:10px;z-index:9999;font-size:13px;font-weight:600;box-shadow:0 4px 24px rgba(0,0,0,0.5);white-space:nowrap"></div>

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

    <!-- Stats row -->
    <div id="statsRow" style="display:none;gap:8px;padding:14px 0 6px;overflow-x:auto"></div>

    <!-- Tabs -->
    <div style="display:flex;gap:8px;margin:14px 0">
      <button id="tabSearch" class="btn" onclick="setTab('search')"
        style="flex:1;padding:11px;border-radius:10px;background:#f97316;color:#fff;border:1px solid #f97316;font-size:13px;font-weight:600;font-family:inherit">
        🔍 Find Leads
      </button>
      <button id="tabTracker" class="btn" onclick="setTab('tracker')"
        style="flex:1;padding:11px;border-radius:10px;background:#161616;color:#555;border:1px solid #222;font-size:13px;font-weight:600;font-family:inherit">
        📋 Tracker (0)
      </button>
    </div>

    <!-- Search panel -->
    <div id="searchPanel">
      <div style="font-size:11px;color:#383838;letter-spacing:1.5px;text-transform:uppercase;margin-bottom:14px">Tap a category → AI searches the real web</div>
      <div id="categoryGrid" style="display:grid;grid-template-columns:1fr 1fr;gap:10px"></div>
    </div>

    <!-- Tracker panel -->
    <div id="trackerPanel" style="display:none">
      <div id="filterRow" style="display:flex;gap:6px;overflow-x:auto;margin-bottom:14px;padding-bottom:4px"></div>
      <div id="emptyTracker" style="text-align:center;padding:50px;color:#222;display:none">
        <div style="font-size:36px;margin-bottom:10px">📭</div>
        <div style="font-size:13px">No leads yet. Go find some.</div>
      </div>
      <div id="trackerList" style="display:flex;flex-direction:column;gap:10px"></div>
    </div>
  </div>

  <!-- CATEGORY VIEW -->
  <div id="categoryView" style="display:none;padding:0 16px">
    <div style="padding:16px 0 12px">
      <div id="catIcon" style="font-size:26px"></div>
      <div id="catLabel" style="font-family:'Syne',sans-serif;font-weight:800;font-size:22px;margin-top:6px"></div>
      <div style="font-size:11px;color:#383838;margin-top:4px;letter-spacing:1px;text-transform:uppercase">South Africa · Live Web Search</div>
    </div>

    <div id="searchingState" style="display:none;text-align:center;padding:50px 20px">
      <div class="spinning" style="font-size:32px;margin-bottom:16px">🔍</div>
      <div style="font-size:14px;color:#f97316;font-weight:600">Searching the web...</div>
      <div id="searchLog" style="font-size:12px;color:#2a2a2a;margin-top:8px"></div>
    </div>

    <div id="resultsList" style="display:flex;flex-direction:column;gap:12px"></div>
  </div>

</div>

<script>
const CATEGORIES = [
  {id:"plumbers",label:"Plumbers",icon:"🔧",query:"plumber South Africa",angle:"Get more clients through Aedica — homeowners find you, you get paid securely."},
  {id:"electricians",label:"Electricians",icon:"⚡",query:"electrician contractor South Africa",angle:"Get more clients through Aedica — homeowners find you, you get paid securely."},
  {id:"tilers",label:"Tilers",icon:"🏠",query:"tiler flooring contractor South Africa",angle:"Get more clients through Aedica — homeowners find you, you get paid securely."},
  {id:"hvac",label:"HVAC Installers",icon:"❄️",query:"HVAC air conditioning installer South Africa",angle:"Get more clients through Aedica — homeowners find you, you get paid securely."},
  {id:"builders",label:"Building Contractors",icon:"🏗️",query:"building contractor construction South Africa",angle:"Get more clients through Aedica — homeowners find you, you get paid securely."},
  {id:"architects",label:"Architects & Designers",icon:"📐",query:"architect interior designer South Africa",angle:"Connect with homeowners actively looking for your services on Aedica."},
  {id:"estate_agents",label:"Estate Agents",icon:"🏡",query:"real estate agent property South Africa",angle:"Your clients need contractors — Aedica connects them. Let's partner."},
  {id:"property_developers",label:"Property Developers",icon:"🏢",query:"property developer South Africa",angle:"Aedica can streamline contractor sourcing for your developments."},
  {id:"body_corporates",label:"Body Corporate Managers",icon:"🏘️",query:"body corporate property manager South Africa",angle:"Managing a complex? Aedica gives you vetted contractors instantly."},
  {id:"bank_managers",label:"Bank Managers",icon:"🏦",query:"bank manager business banking South Africa FNB Absa Nedbank",angle:"Aedica is building an escrow wallet and loan calculator — let's explore a partnership."},
  {id:"fintech",label:"Fintech & Investors",icon:"💳",query:"fintech startup investor South Africa",angle:"Aedica is a marketplace at MVP stage — looking for strategic partners and early backers."},
  {id:"insurance",label:"Insurance Brokers",icon:"🛡️",query:"home insurance broker South Africa",angle:"When a client claims, they need a contractor fast. Aedica solves that."},
  {id:"property_lawyers",label:"Property Attorneys",icon:"⚖️",query:"property attorney conveyancer South Africa",angle:"Aedica is building the infrastructure for home services in SA — let's connect."},
  {id:"tech_founders",label:"Tech Founders (SA)",icon:"💻",query:"tech startup founder South Africa",angle:"Building Aedica — a home services marketplace in SA. Would love your perspective."},
  {id:"investors",label:"SA Investors & VCs",icon:"📈",query:"venture capital investor South Africa startup",angle:"Aedica is at MVP stage and building fast — open to early conversations."},
  {id:"media",label:"Tech Journalists & Media",icon:"📰",query:"tech journalist South Africa TechCentral Ventureburn",angle:"Building something new in SA's home services space — worth a story?"},
  {id:"influencers",label:"SA Business Influencers",icon:"📢",query:"South Africa business influencer LinkedIn entrepreneur",angle:"Aedica connects homeowners with vetted contractors — the SA market needs this."},
  {id:"hardware_stores",label:"Hardware Stores",icon:"🔨",query:"hardware store owner South Africa",angle:"Your customers are contractors. Aedica helps them get more jobs."},
  {id:"airbnb_hosts",label:"Airbnb Superhosts",icon:"🛋️",query:"Airbnb superhost South Africa",angle:"Need reliable contractors fast? Aedica is building exactly that for SA hosts."},
  {id:"seda",label:"SEDA & Gov Programmes",icon:"🏛️",query:"SEDA small enterprise development South Africa funding",angle:"Aedica is an SA startup creating work for tradespeople — exploring funding alignment."},
  {id:"nhbrc",label:"NHBRC Contacts",icon:"📋",query:"NHBRC home builders South Africa registration",angle:"Aedica vets contractors — alignment with NHBRC standards is key. Let's talk."},
  {id:"car_wash",label:"Car Wash Owners",icon:"🚗",query:"car wash business owner South Africa",angle:"Aedica is expanding home services — car wash is on our roadmap. Early partnership?"},
  {id:"accelerators",label:"Accelerators & Incubators",icon:"🚀",query:"startup accelerator incubator South Africa",angle:"Aedica is at MVP stage and ready to scale — exploring accelerator programmes."},
];

const STATUS_OPTIONS = ["Sent","Replied","Not Interested","Follow Up","Partnership"];
const STATUS_COLORS = {Sent:"#f59e0b",Replied:"#10b981","Not Interested":"#ef4444","Follow Up":"#6366f1",Partnership:"#ec4899"};
const STORAGE_KEY = "aedica_gemini_v1";

let tracker = [];
let currentCategory = null;
let currentResults = [];
let activeTab = "search";
let filterStatus = "All";
let generatedMessages = {};

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
}

// ── TOAST ─────────────────────────────────────────────
function showToast(msg, type="success") {
  const t = document.getElementById("toast");
  t.textContent = msg;
  t.style.background = type === "error" ? "#ef4444" : "#f97316";
  t.style.display = "block";
  setTimeout(() => t.style.display = "none", 3000);
}

// ── NAVIGATION ────────────────────────────────────────
function goHome() {
  document.getElementById("homeView").style.display = "block";
  document.getElementById("categoryView").style.display = "none";
  document.getElementById("backBtn").style.display = "none";
  currentResults = [];
  generatedMessages = {};
  renderAll();
}

function showCategoryView(cat) {
  currentCategory = cat;
  document.getElementById("homeView").style.display = "none";
  document.getElementById("categoryView").style.display = "block";
  document.getElementById("backBtn").style.display = "inline-block";
  document.getElementById("catIcon").textContent = cat.icon;
  document.getElementById("catLabel").textContent = cat.label;
  document.getElementById("resultsList").innerHTML = "";
  searchLeads(cat);
}

// ── TABS ──────────────────────────────────────────────
function setTab(tab) {
  activeTab = tab;
  const isSearch = tab === "search";
  document.getElementById("searchPanel").style.display = isSearch ? "block" : "none";
  document.getElementById("trackerPanel").style.display = isSearch ? "none" : "block";
  document.getElementById("tabSearch").style.background = isSearch ? "#f97316" : "#161616";
  document.getElementById("tabSearch").style.color = isSearch ? "#fff" : "#555";
  document.getElementById("tabSearch").style.borderColor = isSearch ? "#f97316" : "#222";
  document.getElementById("tabTracker").style.background = isSearch ? "#161616" : "#f97316";
  document.getElementById("tabTracker").style.color = isSearch ? "#555" : "#fff";
  document.getElementById("tabTracker").style.borderColor = isSearch ? "#222" : "#f97316";
  if (tab === "tracker") renderTracker();
}

// ── BUILD GRID ────────────────────────────────────────
function buildCategoryGrid() {
  const grid = document.getElementById("categoryGrid");
  grid.innerHTML = CATEGORIES.map(cat => `
    <button class="card btn" onclick="showCategoryView(CATEGORIES.find(c=>c.id==='${cat.id}'))"
      style="background:#111;border:1px solid #1c1c1c;border-radius:12px;padding:14px 12px;text-align:left;display:flex;flex-direction:column;gap:6px;font-family:inherit">
      <span style="font-size:20px">${cat.icon}</span>
      <span style="font-size:12px;font-weight:600;color:#e0ddd8;line-height:1.4">${cat.label}</span>
    </button>
  `).join("");
}

// ── RENDER ALL ────────────────────────────────────────
function renderAll() {
  const followUps = tracker.filter(t => t.status === "Follow Up");
  const badge = document.getElementById("followUpBadge");
  if (followUps.length > 0) {
    badge.style.display = "block";
    badge.textContent = followUps.length + " follow up" + (followUps.length > 1 ? "s" : "");
  } else {
    badge.style.display = "none";
  }

  document.getElementById("trackerCount").textContent = tracker.length + " tracked";
  document.getElementById("tabTracker").textContent = "📋 Tracker (" + tracker.length + ")";
  if (activeTab === "tracker") { document.getElementById("tabTracker").style.background = "#f97316"; document.getElementById("tabTracker").style.color = "#fff"; }

  // Stats row
  const statsRow = document.getElementById("statsRow");
  if (tracker.length > 0) {
    statsRow.style.display = "flex";
    statsRow.innerHTML = ["Sent","Replied","Follow Up","Partnership"].map(s => `
      <div style="background:#111;border:1px solid ${STATUS_COLORS[s]}44;border-radius:10px;padding:10px 14px;min-width:80px;text-align:center;flex-shrink:0">
        <div style="font-size:18px;font-weight:700;color:${STATUS_COLORS[s]}">${tracker.filter(t=>t.status===s).length}</div>
        <div style="font-size:10px;color:#444;margin-top:2px">${s}</div>
      </div>
    `).join("");
  } else {
    statsRow.style.display = "none";
  }

  if (activeTab === "tracker") renderTracker();
}

// ── RENDER TRACKER ────────────────────────────────────
function renderTracker() {
  const filterRow = document.getElementById("filterRow");
  filterRow.innerHTML = ["All", ...STATUS_OPTIONS].map(s => {
    const count = s === "All" ? tracker.length : tracker.filter(t => t.status === s).length;
    const active = filterStatus === s;
    const col = STATUS_COLORS[s] || "#f97316";
    return `<button class="btn" onclick="setFilter('${s}')"
      style="padding:5px 13px;border-radius:20px;font-size:11px;font-weight:600;background:${active ? col : "#161616"};color:${active ? "#fff" : "#555"};border:1px solid ${active ? col : "#222"};white-space:nowrap;flex-shrink:0;font-family:inherit">
      ${s} (${count})
    </button>`;
  }).join("");

  const filtered = filterStatus === "All" ? tracker : tracker.filter(t => t.status === filterStatus);
  const list = document.getElementById("trackerList");
  const empty = document.getElementById("emptyTracker");

  if (filtered.length === 0) {
    empty.style.display = "block";
    list.innerHTML = "";
    return;
  }
  empty.style.display = "none";

  list.innerHTML = filtered.map(lead => `
    <div class="fade-in" style="background:#111;border:1px solid ${STATUS_COLORS[lead.status]}33;border-radius:12px;padding:14px">
      <div style="display:flex;justify-content:space-between;margin-bottom:10px">
        <div>
          <div style="font-weight:600;font-size:14px">${lead.name}</div>
          <div style="font-size:12px;color:#666;margin-top:2px">${lead.title} · ${lead.company}</div>
          <div style="font-size:11px;color:#383838;margin-top:2px">${lead.location} · ${lead.category} · ${lead.date}</div>
        </div>
        <button class="btn" onclick="removeLead('${lead.id}')" style="background:none;border:none;color:#2a2a2a;font-size:18px;align-self:flex-start;font-family:inherit">✕</button>
      </div>
      <div style="display:flex;gap:6px;flex-wrap:wrap;margin-bottom:10px">
        ${STATUS_OPTIONS.map(s => `
          <button class="btn" onclick="updateStatus('${lead.id}','${s}')"
            style="padding:4px 10px;border-radius:20px;font-size:11px;font-weight:600;background:${lead.status===s ? STATUS_COLORS[s] : "#1a1a1a"};color:${lead.status===s ? "#fff" : "#383838"};border:1px solid ${lead.status===s ? STATUS_COLORS[s] : "#222"};font-family:inherit">
            ${s}
          </button>`).join("")}
      </div>
      ${lead.searchUrl ? `<a href="${lead.searchUrl}" target="_blank" style="display:inline-block;margin-bottom:8px;color:#f97316;font-size:12px;font-weight:600">🔗 Open on ${lead.platform}</a>` : ""}
      ${lead.message ? `
        <div style="background:#0d0d0d;border:1px solid #1a1a1a;border-radius:8px;padding:10px;margin-bottom:8px">
          <div style="font-size:12px;color:#777;line-height:1.6">${lead.message}</div>
          <button class="btn" onclick="copyText('${encodeURIComponent(lead.message)}','Copied!')" style="margin-top:8px;background:none;border:1px solid #f97316;color:#f97316;padding:5px 12px;border-radius:6px;font-size:11px;font-weight:600;font-family:inherit">Copy Message</button>
        </div>` : ""}
      <textarea onchange="updateNotes('${lead.id}',this.value)" placeholder="Notes..." rows="2"
        style="width:100%;background:#0d0d0d;border:1px solid #1c1c1c;border-radius:8px;padding:8px;color:#666;font-size:12px;font-family:inherit">${lead.notes}</textarea>
    </div>
  `).join("");
}

function setFilter(s) { filterStatus = s; renderTracker(); }

// ── SEARCH LEADS ──────────────────────────────────────
async function searchLeads(category) {
  document.getElementById("searchingState").style.display = "block";
  document.getElementById("resultsList").innerHTML = "";
  document.getElementById("searchLog").textContent = "Asking Gemini to search the web...";

  const prompt = `You are a lead research assistant for Aedica (aedica.co.za), a South African home services marketplace.

Search for REAL South African ${category.label} that Xolani Mthethwa (founder of Aedica) can contact for business outreach.

Search query: "${category.query} South Africa social media"

Find real people at real companies in Johannesburg, Cape Town, Pretoria, Durban, or Mpumalanga.
For each person find where they are ACTUALLY active online: LinkedIn, Instagram, Facebook, Twitter/X, TikTok, or their own website.

Return EXACTLY 6 leads as a raw JSON array. No markdown, no backticks, no explanation — ONLY the JSON array.

Format:
[{"name":"Full Name","title":"Job Title","company":"Company Name","location":"City, SA","platform":"Instagram","searchUrl":"https://www.instagram.com/theirhandle","notes":"Why they are relevant to Aedica"}]

platform must be one of: LinkedIn, Instagram, Facebook, Twitter, TikTok, Website. Use the platform they are most active on.`;

  try {
    document.getElementById("searchLog").textContent = "Searching the web for real leads...";

    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${getKey()}`,
      {
        method: "POST",
        headers: {"Content-Type":"application/json"},
        body: JSON.stringify({
          contents: [{parts:[{text: prompt}]}],
          tools: [{google_search:{}}],
          generationConfig: {temperature: 0.3}
        })
      }
    );

    const data = await response.json();
    document.getElementById("searchLog").textContent = "Processing results...";

    if (data.error) throw new Error(data.error.message);

    const text = data.candidates?.[0]?.content?.parts?.map(p => p.text || "").join("") || "";

    let leads = [];
    const match = text.match(/\[[\s\S]*?\]/);
    if (match) {
      try { leads = JSON.parse(match[0]); } catch {}
    }

    // Fix URLs
    leads = leads.map(lead => ({
      ...lead,
      searchUrl: lead.searchUrl?.startsWith("http")
        ? lead.searchUrl
        : `https://www.linkedin.com/search/results/people/?keywords=${encodeURIComponent((lead.name||"")+" "+(lead.company||"")+" South Africa")}`,
      platform: lead.platform || "LinkedIn"
    }));

    if (!leads.length) {
      leads = [{
        name: category.label + " in South Africa",
        title: category.label,
        company: "Search manually",
        location: "South Africa",
        platform: "LinkedIn",
        searchUrl: `https://www.linkedin.com/search/results/people/?keywords=${encodeURIComponent(category.query)}&geoUrn=%5B%22105149290%22%5D`,
        notes: "Tap the link to search LinkedIn directly."
      }];
      showToast("Limited results — use the LinkedIn link.", "error");
    }

    currentResults = leads;
    renderResults(leads, category);

  } catch(e) {
    showToast("Search failed: " + e.message, "error");
    document.getElementById("resultsList").innerHTML = `<div style="text-align:center;padding:40px;color:#444">Something went wrong. Check your API key or try again.</div>`;
  }

  document.getElementById("searchingState").style.display = "none";
}

// ── RENDER RESULTS ────────────────────────────────────
function renderResults(leads, category) {
  const list = document.getElementById("resultsList");
  list.innerHTML = leads.map((lead, i) => `
    <div class="fade-in" style="background:#111;border:1px solid #1c1c1c;border-radius:14px;padding:16px;animation-delay:${i*0.06}s;animation-fill-mode:forwards;opacity:0">
      <div style="margin-bottom:12px">
        <div style="font-weight:700;font-size:15px">${lead.name}</div>
        <div style="font-size:13px;color:#666;margin-top:3px">${lead.title}</div>
        <div style="font-size:12px;color:#383838;margin-top:2px">${lead.company} · ${lead.location}</div>
        ${lead.notes ? `<div style="font-size:12px;color:#f97316;margin-top:6px;line-height:1.5;opacity:0.75">${lead.notes}</div>` : ""}
      </div>
      <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:10px">
        <a href="${lead.searchUrl}" target="_blank"
          style="background:#1a1a1a;border:1px solid #2a2a2a;color:#f97316;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600">
          ${{LinkedIn:"💼",Instagram:"📸",Facebook:"👥",Twitter:"🐦",TikTok:"🎵",Website:"🌐"}[lead.platform]||"🔗"} Find on ${lead.platform||"LinkedIn"}
        </a>
        <button class="btn" id="msgBtn_${i}" onclick="generateMessage(${i},'${encodeURIComponent(JSON.stringify(lead))}','${encodeURIComponent(JSON.stringify(category))}')"
          style="background:#f97316;border:none;color:#fff;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600;font-family:inherit">
          ✍️ Write Message
        </button>
        <button class="btn" onclick="addToTracker(${i})"
          style="background:#1a1a1a;border:1px solid #222;color:#10b981;padding:7px 14px;border-radius:8px;font-size:12px;font-weight:600;font-family:inherit">
          + Track
        </button>
      </div>
      <div id="msgBox_${i}" style="display:none;background:#0d0d0d;border:1px solid #f9731622;border-radius:10px;padding:14px">
        <div style="font-size:10px;color:#f97316;font-weight:700;letter-spacing:1.5px;text-transform:uppercase;margin-bottom:8px">Your Message</div>
        <div id="msgText_${i}" style="font-size:13px;color:#aaa;line-height:1.75"></div>
        <button class="btn" id="copyBtn_${i}" style="margin-top:12px;background:#f97316;border:none;color:#fff;padding:10px 16px;border-radius:8px;font-size:13px;font-weight:700;width:100%;font-family:inherit">
          📋 Copy & Go Send It
        </button>
      </div>
    </div>
  `).join("") + `
    <button class="btn" onclick="searchLeads(currentCategory)"
      style="background:#161616;border:1px solid #222;color:#444;padding:14px;border-radius:12px;font-size:13px;font-weight:600;width:100%;margin-top:4px;margin-bottom:16px;font-family:inherit">
      🔄 Search Again for More
    </button>`;
}

// ── GENERATE MESSAGE ──────────────────────────────────
async function generateMessage(index, leadEnc, catEnc) {
  const lead = JSON.parse(decodeURIComponent(leadEnc));
  const category = JSON.parse(decodeURIComponent(catEnc));
  const btn = document.getElementById("msgBtn_" + index);
  btn.textContent = "Writing...";
  btn.style.background = "#1a1a1a";
  btn.disabled = true;

  const platform = lead.platform || "LinkedIn";
  const platformStyle = {
    LinkedIn: "professional but warm LinkedIn DM",
    Instagram: "casual friendly Instagram DM",
    Facebook: "friendly Facebook message",
    Twitter: "short punchy Twitter/X DM (under 280 chars)",
    TikTok: "casual TikTok DM",
    Website: "short professional email or contact form message"
  }[platform] || "short casual DM";

  const prompt = `Write a short, genuine ${platformStyle} from Xolani Mthethwa, founder of Aedica (aedica.co.za) — a South African home services marketplace connecting homeowners with vetted contractors, with an escrow wallet for secure payments.

Write to: ${lead.name}, ${lead.title} at ${lead.company} in ${lead.location}.
Platform: ${platform}
Angle: ${category.angle}

Rules:
- Max 4 sentences
- Sound like a real person, not AI
- No subject line
- Start with their name
- Mention aedica.co.za naturally
- Be warm, direct and genuine
- Tailor the tone for ${platform}`;

  try {
    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${getKey()}`,
      {
        method: "POST",
        headers: {"Content-Type":"application/json"},
        body: JSON.stringify({
          contents: [{parts:[{text: prompt}]}],
          generationConfig: {temperature: 0.8, maxOutputTokens: 300}
        })
      }
    );
    const data = await response.json();
    if (data.error) throw new Error(data.error.message);
    const msg = data.candidates?.[0]?.content?.parts?.[0]?.text || "";

    generatedMessages[index] = msg;

    const box = document.getElementById("msgBox_" + index);
    document.getElementById("msgText_" + index).textContent = msg;
    box.style.display = "block";

    document.getElementById("copyBtn_" + index).onclick = () => {
      navigator.clipboard.writeText(msg);
      showToast("Copied! Go paste and send it.");
    };

  } catch(e) {
    showToast("Message generation failed.", "error");
  }

  btn.textContent = "✍️ Write Message";
  btn.style.background = "#f97316";
  btn.disabled = false;
}

// ── TRACKER ACTIONS ───────────────────────────────────
function addToTracker(index) {
  const lead = currentResults[index];
  if (!lead) return;
  const exists = tracker.find(t => t.name === lead.name && t.company === lead.company);
  if (exists) { showToast("Already tracked."); return; }
  const entry = {
    id: Date.now() + "" + Math.random(),
    name: lead.name,
    title: lead.title,
    company: lead.company,
    location: lead.location,
    platform: lead.platform,
    category: currentCategory.label,
    status: "Sent",
    date: new Date().toLocaleDateString("en-ZA"),
    notes: "",
    message: generatedMessages[index] || "",
    searchUrl: lead.searchUrl || ""
  };
  tracker.unshift(entry);
  saveTracker();
  showToast(lead.name + " added ✓");
  renderAll();
}

function updateStatus(id, status) {
  tracker = tracker.map(t => t.id === id ? {...t, status} : t);
  saveTracker();
  renderTracker();
}

function updateNotes(id, notes) {
  tracker = tracker.map(t => t.id === id ? {...t, notes} : t);
  saveTracker();
}

function removeLead(id) {
  tracker = tracker.filter(t => t.id !== id);
  saveTracker();
  renderTracker();
  renderAll();
  showToast("Removed.");
}

function saveTracker() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tracker));
}

function copyText(encoded, msg) {
  navigator.clipboard.writeText(decodeURIComponent(encoded));
  showToast(msg);
}
</script>
</body>
</html>


      
