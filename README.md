<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Vitvisor Docs — Editor Local</title>
<style>
  :root{
    --blue:#0ea5e9;
    --dark:#0f172a;
    --bg:#f1f5f9;
    --card:#ffffff;
  }
  *{box-sizing:border-box}
  body{margin:0;font-family:Inter, "Segoe UI", system-ui, -apple-system, Roboto, "Helvetica Neue", Arial;background:var(--bg);color:var(--dark);height:100vh;display:flex;flex-direction:column}
  header{background:var(--dark);color:white;padding:12px 20px;font-weight:700;display:flex;align-items:center;justify-content:space-between}
  header .brand{display:flex;align-items:center;gap:12px}
  header .brand .logo{width:36px;height:36px;border-radius:6px;background:var(--blue);display:flex;align-items:center;justify-content:center;font-weight:800;color:white}
  main{flex:1;display:flex;gap:18px;padding:18px;overflow:hidden}
  /* left pane - library */
  .left{width:360px;min-width:260px;background:var(--card);border-radius:10px;padding:16px;box-shadow:0 6px 18px rgba(15,23,42,0.06);overflow:auto}
  .controls{display:flex;gap:8px;margin-bottom:12px}
  button.primary{background:var(--blue);color:white;border:0;padding:8px 12px;border-radius:8px;cursor:pointer;font-weight:600}
  button.ghost{background:transparent;border:1px solid #e6eef7;padding:8px 10px;border-radius:8px;cursor:pointer}
  h3{margin:6px 0 12px;font-size:14px;color:var(--blue)}
  .folders{display:flex;flex-direction:column;gap:8px}
  .folder-card{display:flex;justify-content:space-between;align-items:center;padding:10px;border-radius:8px;background:#f8fbff;cursor:pointer}
  .folder-card strong{font-size:14px}
  .folder-actions{display:flex;gap:6px}
  .small{font-size:12px;padding:6px 8px}
  .docs-list{margin-top:12px;display:flex;flex-direction:column;gap:10px}
  .doc-card{background:var(--card);padding:12px;border-radius:10px;box-shadow:0 4px 10px rgba(15,23,42,0.03);cursor:pointer;display:flex;flex-direction:column;gap:6px}
  .doc-title{font-weight:700}
  .doc-preview{font-size:13px;color:#475569;max-height:42px;overflow:hidden}
  /* right pane - workspace */
  .workspace{flex:1;display:flex;flex-direction:column;gap:12px;min-width:0}
  .topbar{display:flex;align-items:center;gap:10px;background:var(--card);padding:10px;border-radius:10px;box-shadow:0 6px 18px rgba(15,23,42,0.04)}
  .toolbar{display:flex;gap:6px;align-items:center;flex-wrap:wrap}
  .tool-btn{background:white;border:1px solid #e6eef7;padding:8px;border-radius:8px;cursor:pointer}
  .tool-select{padding:8px;border-radius:8px;border:1px solid #e6eef7;background:white}
  #editorHeader{display:flex;align-items:center;justify-content:space-between;gap:12px}
  #docName{font-weight:800;font-size:18px}
  #editorWrap{flex:1;display:flex;gap:12px;min-height:0}
  #editorPane{flex:1;background:var(--card);padding:18px;border-radius:10px;box-shadow:0 6px 18px rgba(15,23,42,0.04);overflow:auto}
  #editor{min-height:60vh;outline:none;padding:8px;border-radius:4px}
  .status{font-size:13px;color:#64748b}
  .rightPreview{width:320px;min-width:220px;background:linear-gradient(180deg,#ffffff,#fbfdff);padding:12px;border-radius:10px;box-shadow:0 6px 18px rgba(15,23,42,0.03);overflow:auto}
  .meta{font-size:13px;color:#64748b;margin-bottom:8px}
  /* responsive */
  @media (max-width:900px){
    main{flex-direction:column;padding:12px}
    .left{width:100%}
    .rightPreview{width:100%}
  }
</style>
</head>
<body>

<header>
  <div class="brand">
    <div class="logo">V</div>
    Vitvisor Docs — Editor local
  </div>
  <div class="status" id="saveStatus">Sin cambios</div>
</header>

<main>
  <!-- Left: library -->
  <aside class="left" aria-label="Biblioteca">
    <div class="controls">
      <button class="primary" id="btnNewDoc">Nuevo documento</button>
      <button class="ghost" id="btnNewFolder">Nueva carpeta</button>
    </div>

    <h3>Carpetas</h3>
    <div class="folders" id="foldersContainer"></div>

    <h3 style="margin-top:18px">Documentos sin carpeta</h3>
    <div class="docs-list" id="docsContainer"></div>
  </aside>

  <!-- Workspace -->
  <section class="workspace">
    <div class="topbar">
      <div style="display:flex;flex-direction:column;min-width:0">
        <div id="editorHeader">
          <div>
            <div id="docName">Selecciona o crea un documento</div>
            <div class="meta" id="docMeta">—</div>
          </div>

          <div style="display:flex;align-items:center;gap:8px">
            <button class="tool-btn" id="btnBack" title="Volver">🏠 Inicio</button>
            <button class="tool-btn" id="btnSave" title="Guardar (Ctrl+S)">💾 Guardar</button>
            <button class="tool-btn" id="btnExportDoc" title="Exportar como .doc">📤 Exportar .doc</button>
            <button class="tool-btn" id="btnExportHtml" title="Exportar HTML">🌐 Exportar HTML</button>
            <button class="tool-btn" id="btnDelete" title="Eliminar documento">🗑️ Eliminar</button>
          </div>
        </div>

        <!-- Toolbar rich text -->
        <div class="toolbar" style="margin-top:10px">
          <button class="tool-btn" onclick="exec('undo')" title="Deshacer">↶</button>
          <button class="tool-btn" onclick="exec('redo')" title="Rehacer">↷</button>
          <button class="tool-btn" onclick="exec('bold')" title="Negrita"><b>B</b></button>
          <button class="tool-btn" onclick="exec('italic')" title="Cursiva"><i>I</i></button>
          <button class="tool-btn" onclick="exec('underline')" title="Subrayado"><u>U</u></button>

          <select id="fontSize" class="tool-select" onchange="setFontSize(this.value)" title="Tamaño">
            <option value="">Tamaño</option>
            <option value="1">10px</option>
            <option value="2">12px</option>
            <option value="3">14px</option>
            <option value="4">18px</option>
            <option value="5">24px</option>
            <option value="6">32px</option>
            <option value="7">40px</option>
          </select>

          <select id="fontFamily" class="tool-select" onchange="exec('fontName', this.value)" title="Fuente">
            <option value="">Fuente</option>
            <option value="Arial">Arial</option>
            <option value="Times New Roman">Times New Roman</option>
            <option value="Georgia">Georgia</option>
            <option value="Verdana">Verdana</option>
            <option value="Segoe UI">Segoe UI</option>
          </select>

          <button class="tool-btn" onclick="exec('insertUnorderedList')" title="Lista viñetas">• Lista</button>
          <button class="tool-btn" onclick="exec('insertOrderedList')" title="Lista numerada">1. Lista</button>

          <select class="tool-select" onchange="exec('formatBlock', this.value)" title="Formato">
            <option value="">Formato</option>
            <option value="H1">Encabezado 1</option>
            <option value="H2">Encabezado 2</option>
            <option value="H3">Encabezado 3</option>
            <option value="P">Párrafo</option>
          </select>

          <input type="color" id="colorPicker" class="tool-select" style="padding:6px" title="Color texto" onchange="exec('foreColor', this.value)" />
          <label class="tool-select" style="padding:0 8px;display:inline-flex;align-items:center;gap:8px;cursor:pointer">
            <input id="imgInput" type="file" accept="image/*" style="display:none" onchange="handleImage(event)" />
            <span onclick="document.getElementById('imgInput').click()" style="cursor:pointer">🖼️ Insertar imagen</span>
          </label>
        </div>
      </div>
    </div>

    <div id="editorWrap">
      <div id="editorPane">
        <div id="editor" contenteditable="true" aria-label="Editor de documento"></div>
      </div>

      <aside class="rightPreview">
        <div style="font-weight:700;margin-bottom:8px">Previsualización rápida</div>
        <div id="quickPreview" style="font-size:14px;color:#334155"></div>
        <hr style="margin:12px 0;border:none;border-top:1px solid #eef2f7" />
        <div style="font-size:13px;color:#64748b">Opciones rápidas</div>
        <div style="display:flex;flex-direction:column;gap:8px;margin-top:8px">
          <button class="small tool-btn" onclick="renameDoc()">✏️ Renombrar</button>
          <button class="small tool-btn" onclick="duplicateDoc()">📄 Duplicar</button>
        </div>
      </aside>
    </div>
  </section>
</main>

<script>
/* ------------------------
   Modelo de datos (localStorage)
   estructura: {
     folders: { folderName: { docs: { title: html } } },
     docs: { title: html }  // docs sin carpeta
   }
   ------------------------*/
const STORAGE_KEY = 'vitvisor_docs_v3';
let model = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
if (!model.folders) model = { folders: {}, docs: {} };

let current = { folder: null, title: null };
let saveTimer = null;
let isDirty = false;
const debounceSaveMs = 700;

const dom = {
  foldersContainer: document.getElementById('foldersContainer'),
  docsContainer: document.getElementById('docsContainer'),
  btnNewDoc: document.getElementById('btnNewDoc'),
  btnNewFolder: document.getElementById('btnNewFolder'),
  editor: document.getElementById('editor'),
  docName: document.getElementById('docName'),
  docMeta: document.getElementById('docMeta'),
  quickPreview: document.getElementById('quickPreview'),
  saveStatus: document.getElementById('saveStatus'),
  btnSave: document.getElementById('btnSave'),
  btnBack: document.getElementById('btnBack'),
  btnExportDoc: document.getElementById('btnExportDoc'),
  btnExportHtml: document.getElementById('btnExportHtml'),
  btnDelete: document.getElementById('btnDelete'),
  editorPane: document.getElementById('editorPane')
};

function persist() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(model));
}

/* Renderizar lista de carpetas y documentos */
function render() {
  dom.foldersContainer.innerHTML = '';
  for (const fname of Object.keys(model.folders)) {
    const fdiv = document.createElement('div');
    fdiv.className = 'folder-card';
    fdiv.innerHTML = `<strong>${fname}</strong><div class="folder-actions">
      <button class="small" onclick="createDocInFolder(event,'${escapeJs(fname)}')">+ doc</button>
      <button class="small" onclick="renameFolder(event,'${escapeJs(fname)}')">✏️</button>
      <button class="small" onclick="deleteFolder(event,'${escapeJs(fname)}')">🗑️</button>
    </div>`;
    fdiv.onclick = (e)=>{ if(e.target.tagName!=='BUTTON') toggleFolderContents(fname); };
    dom.foldersContainer.appendChild(fdiv);

    // If folder open, show its docs inline
    if (fdiv.classList.contains('open')) {
      // not used; we'll show docs in a modal style later if desired
    }
  }

  dom.docsContainer.innerHTML = '';
  // Docs without folder
  for (const title of Object.keys(model.docs).reverse()) {
    const d = document.createElement('div');
    d.className = 'doc-card';
    d.onclick = ()=>openDoc(null, title);
    d.innerHTML = `<div class="doc-title">${title}</div>
                   <div class="doc-preview">${stripHtml(model.docs[title]).substring(0,200)}</div>`;
    dom.docsContainer.appendChild(d);
  }
}

/* Helper safe string for onclick embedding */
function escapeJs(s){ return s.replace(/\\/g,'\\\\').replace(/'/g,"\\'").replace(/"/g,'\\"'); }

/* Utilities */
function stripHtml(html){
  const tmp = document.createElement('div');
  tmp.innerHTML = html || '';
  return tmp.textContent || tmp.innerText || '';
}

/* Create folder & docs */
dom.btnNewFolder.addEventListener('click', ()=>{
  const name = prompt('Nombre de la nueva carpeta:');
  if (!name) return;
  if (model.folders[name]) return alert('Ya existe una carpeta con ese nombre');
  model.folders[name] = { docs: {} };
  persist(); render();
});

dom.btnNewDoc.addEventListener('click', ()=>{
  const title = prompt('Nombre del documento:');
  if (!title) return;
  // create in root (no folder)
  if (model.docs[title]) return alert('Ya existe un documento con ese título');
  model.docs[title] = '';
  persist(); render();
  openDoc(null, title);
});

function createDocInFolder(ev, folder){
  ev.stopPropagation();
  const title = prompt('Nombre del documento en carpeta "'+folder+'":');
  if (!title) return;
  if (model.folders[folder].docs[title]) return alert('Ya existe un documento con ese título en la carpeta');
  model.folders[folder].docs[title] = '';
  persist(); render();
  openDoc(folder, title);
}

/* rename / delete folder */
function renameFolder(ev, oldName){ ev.stopPropagation();
  const nuevo = prompt('Nuevo nombre carpeta:', oldName);
  if (!nuevo || nuevo===oldName) return;
  if (model.folders[nuevo]) return alert('Ya existe carpeta con ese nombre');
  model.folders[nuevo] = model.folders[oldName];
  delete model.folders[oldName];
  persist(); render();
}
function deleteFolder(ev, name){ ev.stopPropagation();
  if (!confirm('Eliminar carpeta "'+name+'" y todos sus documentos?')) return;
  delete model.folders[name];
  persist(); render();
}

/* open document (folder can be null) */
function openDoc(folder, title){
  current.folder = folder;
  current.title = title;
  const content = folder ? model.folders[folder].docs[title] : model.docs[title];
  dom.editor.innerHTML = content || '';
  dom.docName.textContent = title;
  dom.docMeta.textContent = folder ? `Carpeta: ${folder}` : 'Sin carpeta';
  updatePreview();
  showEditor();
  focusEditorEnd();
  setDirty(false);
}

/* Show/hide editor UI */
function showEditor(){
  document.querySelector('main').querySelector('.workspace').style.display = 'flex';
  // On mobile the left still visible; we just keep current layout.
}

/* Navigation buttons */
dom.btnBack.addEventListener('click', ()=>{
  if (isDirty) saveNow();
  // scroll back to top or just close; we keep everything visible
  dom.docName.textContent = 'Selecciona o crea un documento';
  dom.docMeta.textContent = '—';
  dom.editor.innerHTML = '';
  current = {folder:null,title:null};
  updatePreview();
});

/* Save */
function saveNow(){
  if (!current.title) return;
  const html = dom.editor.innerHTML;
  if (current.folder) model.folders[current.folder].docs[current.title] = html;
  else model.docs[current.title] = html;
  persist();
  setDirty(false);
  dom.saveStatus.textContent = 'Guardado';
  setTimeout(()=>dom.saveStatus.textContent='Sin cambios',1200);
}
dom.btnSave.addEventListener('click', saveNow);

/* Auto-save with debounce */
dom.editor.addEventListener('input', ()=>{
  setDirty(true);
  updatePreview();
  if (saveTimer) clearTimeout(saveTimer);
  saveTimer = setTimeout(()=>{ saveNow(); }, debounceSaveMs);
});

/* Dirty state */
function setDirty(v){
  isDirty = v;
  dom.saveStatus.textContent = v ? 'Cambios sin guardar' : 'Guardado';
}

/* Formatting helpers */
function exec(cmd, val){
  document.execCommand(cmd, false, val || null);
  dom.editor.focus();
}
function setFontSize(val){
  // fontSize expects 1-7; we pass the selected value directly
  if(!val) return;
  exec('fontSize', val);
}
function focusEditorEnd(){
  dom.editor.focus();
  placeCaretAtEnd(dom.editor);
}
function placeCaretAtEnd(el){
  el = el || document.body;
  const range = document.createRange();
  range.selectNodeContents(el);
  range.collapse(false);
  const sel = window.getSelection();
  sel.removeAllRanges();
  sel.addRange(range);
}

/* Image insertion from file */
function handleImage(e){
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (evt)=>{
    // insert using execCommand for caret position
    document.execCommand('insertImage', false, evt.target.result);
  };
  reader.readAsDataURL(file);
}

/* Export: .doc (Word opens HTML), and HTML */
dom.btnExportDoc.addEventListener('click', ()=>{
  if (!current.title) return alert('Selecciona un documento para exportar');
  const header = "<html><head><meta charset='utf-8'></head><body>";
  const footer = "</body></html>";
  const content = header + dom.editor.innerHTML + footer;
  const blob = new Blob([content], { type: 'application/msword' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = current.title + '.doc';
  a.click();
});
dom.btnExportHtml.addEventListener('click', ()=>{
  if (!current.title) return alert('Selecciona un documento para exportar');
  const header = "<!doctype html><html><head><meta charset='utf-8'></head><body>";
  const footer = "</body></html>";
  const content = header + dom.editor.innerHTML + footer;
  const blob = new Blob([content], { type: 'text/html' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = current.title + '.html';
  a.click();
});

/* Delete doc */
dom.btnDelete.addEventListener('click', ()=>{
  if (!current.title) return alert('Selecciona un documento');
  if (!confirm('Eliminar "'+current.title+'" ?')) return;
  if (current.folder) delete model.folders[current.folder].docs[current.title];
  else delete model.docs[current.title];
  persist();
  dom.editor.innerHTML = '';
  current = {folder:null,title:null};
  render();
  updatePreview();
});

/* Rename, Duplicate */
function renameDoc(){
  if (!current.title) return alert('Selecciona un documento');
  const nuevo = prompt('Nuevo nombre:', current.title);
  if (!nuevo || nuevo===current.title) return;
  if (current.folder){
    const folderObj = model.folders[current.folder].docs;
    if (folderObj[nuevo]) return alert('Ya existe un documento con ese nombre en la carpeta');
    folderObj[nuevo] = folderObj[current.title];
    delete folderObj[current.title];
  } else {
    if (model.docs[nuevo]) return alert('Ya existe un documento con ese nombre');
    model.docs[nuevo] = model.docs[current.title];
    delete model.docs[current.title];
  }
  current.title = nuevo;
  persist(); render();
  dom.docName.textContent = nuevo;
}
function duplicateDoc(){
  if (!current.title) return alert('Selecciona un documento');
  const copyName = current.title + ' (copia)';
  if (current.folder){
    model.folders[current.folder].docs[copyName] = model.folders[current.folder].docs[current.title];
  } else {
    model.docs[copyName] = model.docs[current.title];
  }
  persist(); render();
  alert('Documento duplicado');
}

/* Update quick preview */
function updatePreview(){
  const text = stripHtml(dom.editor.innerHTML).trim();
  dom.quickPreview.textContent = text ? text.substring(0,400) : '(vacío)';
}

/* Keyboard shortcuts */
document.addEventListener('keydown', (e)=>{
  if ((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 's'){ e.preventDefault(); saveNow(); }
  if ((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 'b'){ e.preventDefault(); exec('bold'); }
});

/* Initial render */
render();

/* Ensure clicking a folder shows its docs below in a quick modal-like expansion */
function toggleFolderContents(folderName){
  // show an overlay list of docs quickly
  const docs = model.folders[folderName].docs || {};
  const keys = Object.keys(docs);
  if (keys.length===0){ if (confirm('Sin documentos. ¿Crear uno nuevo en "'+folderName+'"?')) createDocInFolder({stopPropagation:()=>{}}, folderName); return; }
  // build a small selector
  const list = keys.map(k=>`${k}`).join('\n');
  const choice = prompt('Documentos en "'+folderName+'":\n\n'+list+'\n\nEscribe el nombre para abrir, o deja en blanco para cancelar:');
  if (!choice) return;
  if (!docs[choice]) return alert('Documento no encontrado en la carpeta');
  openDoc(folderName, choice);
}

/* small safety: focus editor when clicking inside pane */
dom.editorPane.addEventListener('click', ()=>dom.editor.focus());

/* place caret util (already above) */

// accessibility: ensure editor visible areas are scrollable on load
window.addEventListener('load', ()=>{ updatePreview(); });

</script>
</body>
</html>
