<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Sanatan Dharm — My Notes</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--accent:#f3c677;--muted:#9aa8b2}
    *{box-sizing:border-box}
    body{margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,'Noto Sans',sans-serif;background:linear-gradient(180deg,#071326 0%,#081424 50%, #08131a 100%);color:#e6eef6;min-height:100vh;display:flex;flex-direction:column}
    header{padding:18px 16px;display:flex;align-items:center;gap:12px}
    .logo{width:56px;height:56px;border-radius:12px;background:radial-gradient(circle at 30% 20%, #ffdca0 0%, #f3c677 18%, #c48e3a 45%, transparent 46%), rgba(255,255,255,0.03);display:flex;align-items:center;justify-content:center;box-shadow:0 6px 20px rgba(0,0,0,0.6)}
    .logo .om{font-weight:700;color:#3b2a05;font-size:26px}
    h1{font-size:18px;margin:0}
    p.subtitle{margin:0;color:var(--muted);font-size:13px}main{padding:16px;flex:1;display:grid;grid-template-columns:1fr;gap:12px;max-width:980px;margin:0 auto;width:100%}

.hero{background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);padding:14px;border-radius:14px;display:flex;flex-direction:column;gap:8px;align-items:flex-start}
.hero h2{margin:0;font-size:20px}
.hero p{margin:0;color:var(--muted);font-size:14px}

.controls{display:flex;gap:8px;flex-wrap:wrap}
button{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 12px;border-radius:10px;color:inherit;font-weight:600;backdrop-filter: blur(4px);cursor:pointer}
button.primary{background:linear-gradient(90deg,rgba(243,198,119,0.12),rgba(243,198,119,0.06));border:1px solid rgba(243,198,119,0.15)}

.cards{display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:12px}
.card{background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);padding:12px;border-radius:12px;position:relative;overflow:hidden;min-height:120px}
.card h3{margin:0 0 6px 0}
.card p{margin:0;color:var(--muted)}

.edit-bar{position:fixed;right:16px;bottom:16px;display:flex;flex-direction:column;gap:8px}
.edit-btn{width:56px;height:56px;border-radius:999px;display:flex;align-items:center;justify-content:center;font-weight:700;font-size:18px}

/* simple glowing Om animation background */
.bg-om{position:fixed;right:8%;top:10%;opacity:0.09;font-size:220px;pointer-events:none;transform:rotate(-15deg)}

/* admin-only badge */
.badge{padding:6px 8px;border-radius:999px;background:rgba(255,255,255,0.03);font-size:12px}

/* modal */
.modal{position:fixed;left:0;top:0;width:100%;height:100%;display:flex;align-items:center;justify-content:center;background:linear-gradient(0deg,rgba(2,6,23,0.6),rgba(2,6,23,0.6));backdrop-filter:blur(6px)}
.modal .sheet{background:var(--card);padding:16px;border-radius:12px;min-width:300px}
input,textarea,select{width:100%;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.05);background:transparent;color:inherit;margin-top:8px}
textarea{min-height:120px}

footer{padding:12px;text-align:center;color:var(--muted);font-size:13px}

/* small responsive tweaks */
@media (max-width:520px){.logo{width:48px;height:48px}.bg-om{font-size:140px;right:-10%;top:4%}}

/* gentle fade-in for content */
.fade-in{animation:fade 700ms ease both}
@keyframes fade{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:none}}

/* subtle card hover effect (viewers only) */
.card:hover{transform:translateY(-6px);transition:transform .25s}

  </style>
</head>
<body>
  <div class="bg-om">ॐ</div>
  <header>
    <div class="logo"><div class="om">ॐ</div></div>
    <div>
      <h1>Sanatan Dharm — My Notes</h1>
      <p class="subtitle">Share teachings, quotes, and articles. Edit locally on your phone — protected by password.</p>
    </div>
  </header>  <main>
    <section class="hero fade-in" id="hero">
      <h2 id="siteTitle">Sanatan Dharm — Truth & Wisdom</h2>
      <p id="siteDesc">A small personal collection of teachings, translations and reflections. Visitors can view. Only you can edit (via Admin).</p>
      <div class="controls">
        <button id="addSectionBtn" class="primary">+ Add Section</button>
        <button id="importBtn">Import JSON</button>
        <button id="exportBtn">Export JSON</button>
        <button id="resetBtn">Reset Data</button>
      </div>
    </section><section class="cards" id="sectionsContainer" aria-live="polite"></section>

  </main>  <div class="edit-bar">
    <button id="adminToggle" class="edit-btn badge">Admin</button>
    <button id="loginBtn" class="edit-btn">🔐</button>
  </div>  <footer>Made for you • Data saved locally on your device (no server). Use Export to backup.</footer>  <!-- modals -->  <div id="modalRoot" style="display:none"></div>  <script>
    // Simple local editable site. Data saved in localStorage under 'sd_site_data'. Admin password hashed and saved in 'sd_admin_hash'.

    // util: sha-256 (returns hex)
    async function sha256Hex(text){
      const enc = new TextEncoder();
      const data = enc.encode(text);
      const hash = await crypto.subtle.digest('SHA-256', data);
      return Array.from(new Uint8Array(hash)).map(b=>b.toString(16).padStart(2,'0')).join('');
    }

    // default data
    const defaultData = {
      title: 'Sanatan Dharm — Truth & Wisdom',
      description: 'A small personal collection of teachings, translations and reflections. Visitors can view. Only you can edit (via Admin).',
      sections: [
        {id: Date.now()+1, title:'About', content:'This site contains notes, passages and explanations related to Sanatan Dharm. Use the + Add Section button to add more.'},
        {id: Date.now()+2, title:'Short Quotes', content:'\u201cSatyam vada, dharmam chara. Speak truth, follow dharma.\u201d'}
      ]
    };

    // load/save
    function loadData(){
      try{
        const raw = localStorage.getItem('sd_site_data');
        return raw ? JSON.parse(raw) : structuredClone(defaultData);
      }catch(e){console.error(e); return structuredClone(defaultData)}
    }
    function saveData(d){ localStorage.setItem('sd_site_data', JSON.stringify(d)); render(); }
    function resetData(){ if(confirm('Reset site data to default?')){ localStorage.removeItem('sd_site_data'); render(); }}

    // admin helpers
    let isAdmin = false;
    async function setAdminPassword(pw){ const h = await sha256Hex(pw); localStorage.setItem('sd_admin_hash', h); alert('Admin password set. Remember it.'); }
    async function checkAdminPassword(pw){ const h = await sha256Hex(pw); return h === localStorage.getItem('sd_admin_hash'); }

    // rendering
    function el(tag, attrs={}, children=''){ const e = document.createElement(tag); for(const k in attrs){ if(k.startsWith('on')) e.addEventListener(k.slice(2), attrs[k]); else e.setAttribute(k, attrs[k]); } if(typeof children === 'string') e.innerHTML = children; else if(Array.isArray(children)) children.forEach(c=>e.appendChild(c)); return e }

    function render(){
      const data = loadData();
      document.getElementById('siteTitle').textContent = data.title;
      document.getElementById('siteDesc').textContent = data.description;
      const container = document.getElementById('sectionsContainer'); container.innerHTML='';
      data.sections.forEach(s=>{
        const card = el('article',{class:'card fade-in','data-id':s.id});
        const h = el('h3',{}, s.title);
        const p = el('p',{}, s.content.replace(/\n/g,'<br>'));
        card.appendChild(h); card.appendChild(p);
        if(isAdmin){
          const edit = el('button',{}, 'Edit'); edit.style.position='absolute'; edit.style.right='10px'; edit.style.top='10px'; edit.addEventListener('click',()=>openEditSectionModal(s.id));
          const del = el('button',{}, 'Delete'); del.style.position='absolute'; del.style.right='72px'; del.style.top='10px'; del.addEventListener('click',()=>{ if(confirm('Delete this section?')){ deleteSection(s.id); }});
          card.appendChild(edit); card.appendChild(del);
        }
        container.appendChild(card);
      })
    }

    function addSection(title='New Section', content='Write here...'){
      const data = loadData(); data.sections.unshift({id:Date.now(), title, content}); saveData(data);
    }
    function deleteSection(id){ const data = loadData(); data.sections = data.sections.filter(s=>s.id!=id); saveData(data); }

    // edit modal
    function openModal(html){ const root = document.getElementById('modalRoot'); root.style.display='block'; root.innerHTML = `<div class="modal" id="modalOverlay"> <div class="sheet">${html}</div></div>`;
      document.getElementById('modalOverlay').addEventListener('click',(e)=>{ if(e.target.id==='modalOverlay'){ closeModal(); } });
    }
    function closeModal(){ const root = document.getElementById('modalRoot'); root.style.display='none'; root.innerHTML=''; }

    function openEditSectionModal(id){ const data = loadData(); const s = data.sections.find(x=>x.id==id); openModal(`<h3>Edit Section</h3>
      <label>Title</label><input id="m_title" value="${escapeHtml(s.title)}" />
      <label>Content (HTML allowed)</label><textarea id="m_content">${escapeHtml(s.content)}</textarea><div style="display:flex;gap:8px;margin-top:8px"><button id="saveSec">Save</button><button id="cancelSec">Cancel</button></div>`);
  document.getElementById('saveSec').addEventListener('click',()=>{
    s.title = document.getElementById('m_title').value;
    s.content = document.getElementById('m_content').value;
    saveData(data); closeModal();
  });
  document.getElementById('cancelSec').addEventListener('click',closeModal);
}

function escapeHtml(s){ return (s+'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;'); }

// init buttons
document.getElementById('addSectionBtn').addEventListener('click',()=>{
  if(!isAdmin){ alert('Only admin can add sections. Tap Admin → 🔐 to login.'); return }
  openModal(`<h3>Add Section</h3>
    <label>Title</label><input id="m_title" placeholder="Section title" />
    <label>Content</label><textarea id="m_content" placeholder="Write content..."></textarea>
    <div style="display:flex;gap:8px;margin-top:8px"><button id="saveSec">Add</button><button id="cancelSec">Cancel</button></div>`);
  document.getElementById('saveSec').addEventListener('click',()=>{
    const t = document.getElementById('m_title').value || 'Untitled';
    const c = document.getElementById('m_content').value || '';
    addSection(t,c); closeModal();
  });
  document.getElementById('cancelSec').addEventListener('click',closeModal);
});

document.getElementById('exportBtn').addEventListener('click',()=>{
  const data = localStorage.getItem('sd_site_data') || JSON.stringify(defaultData);
  const blob = new Blob([data], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download = 'sanatan_site_backup.json'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
});

document.getElementById('importBtn').addEventListener('click',()=>{
  if(!isAdmin){ alert('Only admin can import.'); return }
  openModal(`<h3>Import JSON</h3><p style="color:var(--muted);font-size:13px">Paste JSON exported from this site.</p><textarea id="m_json"></textarea><div style="display:flex;gap:8px;margin-top:8px"><button id="doImport">Import</button><button id="cancelImport">Cancel</button></div>`);
  document.getElementById('doImport').addEventListener('click',()=>{
    try{ const raw = document.getElementById('m_json').value; const parsed = JSON.parse(raw); localStorage.setItem('sd_site_data', JSON.stringify(parsed)); closeModal(); render(); alert('Imported'); }
    catch(e){ alert('Invalid JSON') }
  });
  document.getElementById('cancelImport').addEventListener('click',closeModal);
});

document.getElementById('resetBtn').addEventListener('click',resetData);

// admin/login flow
document.getElementById('loginBtn').addEventListener('click',async ()=>{
  const has = localStorage.getItem('sd_admin_hash');
  if(!has){ // first time - set password
    openModal(`<h3>Set Admin Password</h3><p style="color:var(--muted);font-size:13px">Choose a password that only you know. It will be stored hashed locally on this device.</p><input id="m_pw" type="password" placeholder="New password" /><div style="display:flex;gap:8px;margin-top:8px"><button id="setPw">Set Password</button><button id="cancelPw">Cancel</button></div>`);
    document.getElementById('setPw').addEventListener('click',async ()=>{ const pw = document.getElementById('m_pw').value; if(!pw){ alert('Enter a password'); return } await setAdminPassword(pw); closeModal(); });
    document.getElementById('cancelPw').addEventListener('click',closeModal);
  } else {
    openModal(`<h3>Admin Login</h3><input id="m_pw" type="password" placeholder="Password" /><div style="display:flex;gap:8px;margin-top:8px"><button id="doLogin">Login</button><button id="cancelPw">Cancel</button></div>`);
    document.getElementById('doLogin').addEventListener('click',async ()=>{ const pw = document.getElementById('m_pw').value; if(!pw) return; const ok = await checkAdminPassword(pw); if(ok){ isAdmin = true; closeModal(); document.getElementById('adminToggle').textContent='Admin ✓'; render(); } else alert('Wrong password'); });
    document.getElementById('cancelPw').addEventListener('click',closeModal);
  }
});

document.getElementById('adminToggle').addEventListener('click',()=>{
  if(!isAdmin){ alert('Tap the 🔐 button to login as admin.'); return }
  // quick admin tools: edit site title/desc
  const data = loadData(); openModal(`<h3>Edit Site Info</h3><label>Site Title</label><input id="m_title" value="${escapeHtml(data.title)}" />
    <label>Site Description</label><textarea id="m_desc">${escapeHtml(data.description)}</textarea>
    <div style="display:flex;gap:8px;margin-top:8px"><button id="saveInfo">Save</button><button id="logout">Logout</button></div>`);
  document.getElementById('saveInfo').addEventListener('click',()=>{ data.title = document.getElementById('m_title').value; data.description = document.getElementById('m_desc').value; saveData(data); closeModal(); });
  document.getElementById('logout').addEventListener('click',()=>{ isAdmin = false; closeModal(); document.getElementById('adminToggle').textContent='Admin'; render(); });
});

// initial render
render();

// small friendly hint shown once
if(!localStorage.getItem('sd_seen_hint')){
  localStorage.setItem('sd_seen_hint','1');
  setTimeout(()=>alert('Tip: This site saves changes only on your device. Use Export to back up and Import to restore on the same device.'),400);
}

  </script>
</body>
</html>