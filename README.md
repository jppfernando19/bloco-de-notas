[index.html 2.html](https://github.com/user-attachments/files/22015948/index.html.2.html)
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Bloco de Notas — Meu Primeiro App</title>
  <style>
    :root{
      --bg:#0f172a; /* slate-900 */
      --panel:#111827; /* gray-900 */
      --soft:#1f2937; /* gray-800 */
      --accent:#22c55e; /* emerald-500 */
      --text:#e5e7eb; /* gray-200 */
      --muted:#9ca3af; /* gray-400 */
      --danger:#ef4444; /* red-500 */
      --warning:#f59e0b; /* amber-500 */
      --shadow: 0 10px 25px rgba(0,0,0,.35);
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; font:16px/1.5 system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans, Helvetica, Arial, "Apple Color Emoji","Segoe UI Emoji";
      color:var(--text);
      background: radial-gradient(1200px 800px at 10% -10%, #1e293b, #0b1224), var(--bg);
    }
    header{
      position:sticky; top:0; z-index:10; backdrop-filter:saturate(130%) blur(8px);
      background:rgba(15,23,42,.6); border-bottom:1px solid rgba(148,163,184,.12);
    }
    .container{max-width:980px; margin:0 auto; padding:16px}
    .row{display:flex; gap:12px; align-items:center; flex-wrap:wrap}
    h1{font-size:clamp(20px,3vw,28px); margin:0; letter-spacing:.3px}
    .badge{padding:4px 10px; border-radius:999px; background:rgba(34,197,94,.12); color:#bbf7d0; border:1px solid rgba(34,197,94,.25)}
    input[type="search"]{
      flex:1; min-width:220px; padding:12px 14px; border-radius:12px; border:1px solid rgba(148,163,184,.18);
      background:var(--panel); color:var(--text); outline:none; transition:.2s border-color, .2s box-shadow;
    }
    input[type="search"]:focus{border-color:rgba(34,197,94,.55); box-shadow:0 0 0 3px rgba(34,197,94,.15)}

    main{padding:24px 0}
    .composer{background:linear-gradient(180deg, rgba(255,255,255,.04), rgba(255,255,255,.02)); border:1px solid rgba(148,163,184,.15);
      padding:16px; border-radius:16px; box-shadow:var(--shadow)}
    textarea{
      width:100%; min-height:90px; resize:vertical; padding:12px 14px; border-radius:12px;
      border:1px solid rgba(148,163,184,.18); background:var(--panel); color:var(--text); outline:none;
    }
    .controls{display:flex; gap:10px; align-items:center; justify-content:flex-end; margin-top:10px}
    .btn{cursor:pointer; border:1px solid transparent; border-radius:12px; padding:10px 14px; font-weight:600; transition:.2s transform,.2s filter,.2s background,.2s border-color}
    .btn:active{transform:translateY(1px)}
    .btn-primary{background:var(--accent); color:#04210f}
    .btn-outline{background:transparent; color:var(--text); border-color:rgba(148,163,184,.25)}
    .btn-danger{background:transparent; color:#fecaca; border-color:rgba(239,68,68,.5)}

    .meta{display:flex; justify-content:space-between; align-items:center; color:var(--muted); font-size:14px; margin-top:6px}

    .grid{ display:grid; grid-template-columns:repeat(auto-fill,minmax(240px,1fr)); gap:14px; margin-top:18px }
    .card{
      background:linear-gradient(180deg, rgba(255,255,255,.04), rgba(255,255,255,.02));
      border:1px solid rgba(148,163,184,.15); border-radius:16px; padding:14px; display:flex; flex-direction:column; gap:10px;
      box-shadow:var(--shadow)
    }
    .card .time{font-size:12px; color:var(--muted)}
    .card .content{white-space:pre-wrap}
    .card .actions{display:flex; gap:8px; justify-content:flex-end}

    .empty{opacity:.7; text-align:center; padding:24px; border:1px dashed rgba(148,163,184,.25); border-radius:16px}
    footer{opacity:.8; font-size:14px; padding:24px 0; text-align:center}
    .visually-hidden{position:absolute; width:1px; height:1px; padding:0; margin:-1px; overflow:hidden; clip:rect(0,0,0,0); white-space:nowrap; border:0}
  </style>
</head>
<body>
  <header>
    <div class="container">
      <div class="row">
        <h1>🗒️ Bloco de Notas</h1>
        <span class="badge" id="count">0 notas</span>
        <input id="search" type="search" placeholder="Buscar notas... (Ctrl+/)" aria-label="Buscar notas" />
      </div>
    </div>
  </header>

  <div class="container">
    <main>
      <section class="composer" aria-labelledby="compose-title">
        <h2 id="compose-title" class="visually-hidden">Criar nova nota</h2>
        <textarea id="note-input" placeholder="Escreva sua nota aqui..."></textarea>
        <div class="meta"><span id="char-info">0 caracteres</span></div>
        <div class="controls">
          <button id="clear" class="btn btn-outline" type="button" title="Limpar texto">Limpar</button>
          <button id="add" class="btn btn-primary" type="button" title="Salvar nota (Ctrl+Enter)">Salvar nota</button>
        </div>
      </section>

      <section style="margin-top:18px">
        <div id="list" class="grid" aria-live="polite" aria-busy="false"></div>
        <div id="empty" class="empty" hidden>Nenhuma nota ainda. Escreva algo acima e clique em <strong>Salvar nota</strong>.</div>
      </section>
    </main>
  </div>

  <footer>
    <div class="container">Feito com HTML, CSS e JavaScript. Dados salvos no seu navegador (localStorage).</div>
  </footer>

  <script>
    // ======= Utilidades =======
    const $$ = (sel, root = document) => root.querySelector(sel);
    const $$$ = (sel, root = document) => Array.from(root.querySelectorAll(sel));

    const storKey = 'notes.v1';

    function nowISO(){ return new Date().toISOString(); }
    function fmt(d){
      const date = new Date(d);
      return date.toLocaleString('pt-BR', { dateStyle:'short', timeStyle:'short' });
    }

    function load(){
      try{ return JSON.parse(localStorage.getItem(storKey)) || []; }
      catch{ return []; }
    }
    function save(notes){
      localStorage.setItem(storKey, JSON.stringify(notes));
    }

    // ======= Estado =======
    let state = { notes: load(), filter: '' };

    // ======= Renderização =======
    const list = $$('#list');
    const empty = $$('#empty');
    const count = $$('#count');

    function setCount(n){ count.textContent = `${n} ${n===1?'nota':'notas'}`; }

    function render(){
      const filtered = state.filter
        ? state.notes.filter(n => n.content.toLowerCase().includes(state.filter))
        : state.notes;

      list.innerHTML = '';
      if(filtered.length === 0){
        empty.hidden = false; setCount(0); return;
      }
      empty.hidden = true; setCount(filtered.length);

      for(const n of filtered){
        const card = document.createElement('article');
        card.className = 'card';
        card.dataset.id = n.id;
        card.innerHTML = `
          <div class="time" title="Criada: ${fmt(n.createdAt)}\nAtualizada: ${fmt(n.updatedAt)}">
            <span>Atualizada ${fmt(n.updatedAt)}</span>
          </div>
          <div class="content">${escapeHTML(n.content)}</div>
          <div class="actions">
            <button class="btn btn-outline" data-action="edit" title="Editar">Editar</button>
            <button class="btn btn-danger" data-action="delete" title="Excluir">Excluir</button>
          </div>
        `;
        list.appendChild(card);
      }
    }

    function escapeHTML(str){
      return str
        .replaceAll('&','&amp;')
        .replaceAll('<','&lt;')
        .replaceAll('>','&gt;')
        .replaceAll('"','&quot;')
        .replaceAll("'",'&#39;');
    }

    // ======= Ações =======
    function addNote(text){
      const trimmed = text.trim();
      if(!trimmed) return;
      const note = { id: crypto.randomUUID(), content: trimmed, createdAt: nowISO(), updatedAt: nowISO() };
      state.notes.unshift(note);
      save(state.notes); render();
    }
    function updateNote(id, text){
      const i = state.notes.findIndex(n => n.id===id); if(i<0) return;
      state.notes[i] = { ...state.notes[i], content: text.trim(), updatedAt: nowISO() };
      save(state.notes); render();
    }
    function deleteNote(id){
      state.notes = state.notes.filter(n => n.id!==id);
      save(state.notes); render();
    }

    // ======= Eventos UI =======
    const input = $$('#note-input');
    const addBtn = $$('#add');
    const clearBtn = $$('#clear');
    const search = $$('#search');
    const charInfo = $$('#char-info');

    input.addEventListener('input', () => {
      charInfo.textContent = `${input.value.length} caracteres`;
    });

    addBtn.addEventListener('click', () => { addNote(input.value); input.value=''; charInfo.textContent = '0 caracteres'; input.focus(); });
    clearBtn.addEventListener('click', () => { input.value=''; charInfo.textContent = '0 caracteres'; input.focus(); });

    // Salvar com Ctrl+Enter
    input.addEventListener('keydown', (e)=>{
      if(e.key==='Enter' && (e.ctrlKey||e.metaKey)){ e.preventDefault(); addBtn.click(); }
    });

    // Busca rápida (Ctrl+/)
    document.addEventListener('keydown', (e)=>{
      if((e.ctrlKey||e.metaKey) && e.key==='/'){ e.preventDefault(); search.focus(); }
    });

    search.addEventListener('input', ()=>{
      state.filter = search.value.toLowerCase(); render();
    });

    // Delegação para editar/excluir
    list.addEventListener('click', (e)=>{
      const btn = e.target.closest('button'); if(!btn) return;
      const card = e.target.closest('.card');
      const id = card?.dataset.id; if(!id) return;

      if(btn.dataset.action==='delete'){
        if(confirm('Excluir esta nota?')) deleteNote(id);
      }
      if(btn.dataset.action==='edit'){
        const note = state.notes.find(n => n.id===id);
        const novo = prompt('Editar nota:', note?.content ?? '');
        if(novo!==null){ updateNote(id, novo); }
      }
    });

    // ======= Inicialização =======
    render();
  </script>
  <script>
  if ("serviceWorker" in navigator) {
    navigator.serviceWorker.register("service-worker.js")
      .then(() => console.log("Service Worker registrado!"))
      .catch(err => console.error("Erro ao registrar Service Worker:", err));
  }
</script>

</body>
</html>
<head><link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#22c55e">
</head>
