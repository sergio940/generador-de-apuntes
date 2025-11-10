<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vitvisor Docs Futurista Claro</title>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap" rel="stylesheet">
<style>
:root {
  --primary: #00cfff;
  --secondary: #ff00d4;
  --bg: #f0f4f8;
  --card-bg: #ffffff;
  --card-hover: #e0f0ff;
  --toolbar-bg: #00bcd4;
  --toolbar-btn: #0097a7;
  --toolbar-btn-hover: #00796b;
  --text-color: #0a0a0a;
  --border-radius: 16px;
  --shadow-glow: 0 0 15px rgba(0,192,255,0.5), 0 0 25px rgba(255,0,212,0.4);
}

* { box-sizing: border-box; }

body {
  font-family: 'Orbitron', sans-serif;
  margin: 0;
  background: var(--bg);
  color: var(--text-color);
}

header {
  background: linear-gradient(90deg, var(--primary), var(--secondary));
  color: white;
  padding: 20px;
  text-align: center;
  font-size: 28px;
  font-weight: 700;
  box-shadow: 0 0 20px rgba(0,192,255,0.5);
  text-shadow: 0 0 8px rgba(255,0,212,0.4);
}

#home, #editor { display: none; padding: 25px; }
#home.active, #editor.active { display: block; }

button {
  font-family: 'Orbitron', sans-serif;
  background: var(--toolbar-btn);
  color: var(--text-color);
  border: none;
  border-radius: var(--border-radius);
  padding: 10px 16px;
  cursor: pointer;
  font-size: 14px;
  transition: 0.3s all;
  box-shadow: var(--shadow-glow);
}
button:hover { background: var(--toolbar-btn-hover); transform: translateY(-2px) scale(1.05); }

#fileList {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-top: 25px;
}

.folderCard, .docCard {
  background: var(--card-bg);
  border-radius: var(--border-radius);
  padding: 20px;
  width: 160px;
  height: 160px;
  box-shadow: var(--shadow-glow);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  text-align: center;
  transition: 0.3s transform, 0.3s background;
  position: relative;
  color: var(--text-color);
}
.folderCard:hover, .docCard:hover { transform: translateY(-5px) scale(1.05); background: var(--card-hover); }
.folderCard span, .docCard span { font-size: 55px; margin-bottom: 10px; }

.deleteBtn {
  position: absolute;
  top: 6px;
  right: 6px;
  background: #ff0055;
  font-size: 14px;
  padding: 4px 6px;
  border-radius: var(--border-radius);
  box-shadow: 0 0 12px rgba(255,0,85,0.7);
}

#editorToolbar {
  background: var(--toolbar-bg);
  padding: 14px;
  border-bottom: 1px solid #ccc;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
  border-radius: var(--border-radius) var(--border-radius) 0 0;
}
#editorToolbar button, #editorToolbar select, #editorToolbar input[type=color] {
  background: var(--toolbar-btn);
  color: var(--text-color);
  border: none;
  border-radius: var(--border-radius);
  padding: 6px 12px;
  cursor: pointer;
  box-shadow: var(--shadow-glow);
}
#editorToolbar input[type=color] { width: 36px; height: 28px; padding: 0; cursor: pointer; }

#editorArea {
  width: 100%;
  height: 70vh;
  padding: 30px;
  background: #e6f2ff;
  border-radius: var(--border-radius);
  outline: none;
  overflow-y: auto;
  margin-top: 10px;
  box-shadow: var(--shadow-glow);
  color: var(--text-color);
}

#docTitle { margin: 18px 0; }

table { border-collapse: collapse; width: 100%; }
table td, table th { border: 1px solid #00cfff; padding: 8px; text-align: center; color: var(--text-color); }
</style>
</head>
<body>
<header>🚀 Vitvisor Docs Futurista Claro</header>

<section id="home" class="active">
  <button id="newFolderBtn">📁 Nueva carpeta</button>
  <button id="newDocBtn">📝 Nuevo documento</button>
  <h2>Tus archivos</h2>
  <div id="fileList"></div>
</section>

<section id="editor">
  <div id="editorToolbar">
    <button id="backBtn">🏠 Inicio</button>
    <button onclick="format('bold')"><b>N</b></button>
    <button onclick="format('italic')"><i>K</i></button>
    <button onclick="format('underline')"><u>S</u></button>
    <button onclick="format('insertUnorderedList')">• Lista</button>
    <button onclick="format('insertOrderedList')">1. Lista</button>
    <button onclick="format('justifyLeft')">↞</button>
    <button onclick="format('justifyCenter')">↔</button>
    <button onclick="format('justifyRight')">↠</button>
    <select id="fontSizeSelect">
      <option value="">Tamaño letra</option>
      <option value="1">Muy pequeño</option>
      <option value="2">Pequeño</option>
      <option value="3">Normal</option>
      <option value="4">Mediano</option>
      <option value="5">Grande</option>
      <option value="6">Muy grande</option>
      <option value="7">Enorme</option>
    </select>
    <input type="color" id="colorPicker" title="Color de texto">
    <button id="addImgBtn">🖼️ Imagen</button>
    <button id="addTableBtn">🧾 Tabla</button>
    <button id="saveBtn">💾 Guardar</button>
  </div>
  <h2 id="docTitle"></h2>
  <div id="editorArea" contenteditable="true"></div>
</section>

<script>
// Variables
const home=document.getElementById('home');
const editor=document.getElementById('editor');
const fileList=document.getElementById('fileList');
const editorArea=document.getElementById('editorArea');
const docTitle=document.getElementById('docTitle');
const colorPicker=document.getElementById('colorPicker');
const fontSizeSelect=document.getElementById('fontSizeSelect');
let currentDoc=null;
let currentFolder=null;

// Evitar caracteres extraños
document.addEventListener('keydown', e => { if(e.key==='Dead') e.preventDefault(); });

// Cargar archivos
function loadFiles(){
  fileList.innerHTML='';
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  data.forEach((item,i)=>{
    const div=document.createElement('div');
    div.className=item.type==='folder'?'folderCard':'docCard';
    div.innerHTML=`
      <span>${item.type==='folder'?'📁':'📝'}</span>${item.name}
      <button class="deleteBtn" onclick="deleteItem(${i}); event.stopPropagation();">🗑️</button>
    `;
    div.onclick=()=>{ item.type==='folder'?openFolder(i):openDoc(i,null); };
    fileList.appendChild(div);
  });
}

// Crear carpeta
document.getElementById('newFolderBtn').onclick=()=>{
  const name=prompt('Nombre de la carpeta:'); if(!name) return;
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  data.push({type:'folder',name,files:[]});
  localStorage.setItem('vitvisor_files',JSON.stringify(data));
  loadFiles();
}

// Crear documento
document.getElementById('newDocBtn').onclick=()=>{
  const name=prompt('Nombre del documento:'); if(!name) return;
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  if(currentFolder!==null) data[currentFolder].files.push({type:'doc',name,content:''});
  else data.push({type:'doc',name,content:''});
  localStorage.setItem('vitvisor_files',JSON.stringify(data));
  loadFiles();
}

// Abrir carpeta
function openFolder(index){
  currentFolder=index; fileList.innerHTML='';
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]'); const folder=data[index];
  const back=document.createElement('button'); back.textContent='⬅️ Volver'; back.onclick=()=>{currentFolder=null;loadFiles();};
  fileList.appendChild(back);

  folder.files.forEach((f,j)=>{
    const div=document.createElement('div');
    div.className='docCard';
    div.innerHTML=`
      <span>📝</span>${f.name}
      <button class="deleteBtn" onclick="deleteDoc(${j}); event.stopPropagation();">🗑️</button>
    `;
    div.onclick=()=>openDoc(j,index);
    fileList.appendChild(div);
  });

  const newDocBtn=document.createElement('button'); newDocBtn.textContent='📝 Nuevo documento'; 
  newDocBtn.onclick=()=>{
    const name=prompt('Nombre del documento:'); if(!name) return; folder.files.push({type:'doc',name,content:''});
    data[index]=folder; localStorage.setItem('vitvisor_files',JSON.stringify(data)); openFolder(index);
  };
  fileList.appendChild(newDocBtn);
}

// Abrir documento
function openDoc(docIndex, folderIndex){
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]'); let doc;
  if(folderIndex!==null){ doc=data[folderIndex].files[docIndex]; currentFolder=folderIndex; }
  else { doc=data[docIndex]; currentFolder=null; }
  currentDoc=docIndex; docTitle.textContent=doc.name; editorArea.innerHTML=doc.content||"";
  home.classList.remove('active'); editor.classList.add('active');
}

// Guardar
document.getElementById('saveBtn').onclick=()=>{
  if(currentDoc===null) return;
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  if(currentFolder!==null) data[currentFolder].files[currentDoc].content=editorArea.innerHTML;
  else data[currentDoc].content=editorArea.innerHTML;
  localStorage.setItem('vitvisor_files',JSON.stringify(data));
  alert('Documento guardado 💾');
}

// Imagen
document.getElementById('addImgBtn').onclick=()=>{
  const url=prompt('Introduce la URL de la imagen:'); if(url) document.execCommand('insertImage',false,url);
}

// Tabla
document.getElementById('addTableBtn').onclick=()=>{
  const rows=parseInt(prompt('Número de filas:',2)); if(!rows) return;
  const cols=parseInt(prompt('Número de columnas:',2)); if(!cols) return;
  const table=document.createElement('table');
  table.style.border='1px solid #00cfff';
  table.style.width='100%';
  for(let r=0;r<rows;r++){
    const tr=document.createElement('tr');
    for(let c=0;c<cols;c++){
      const td=document.createElement('td'); td.textContent='Texto';
      td.contentEditable=true;
      td.style.border='1px solid #00cfff';
      td.onclick=()=>{
        const bg=prompt('Color de fondo (hex o nombre):',td.style.backgroundColor); if(bg) td.style.backgroundColor=bg;
      };
      tr.appendChild(td);
    }
    table.appendChild(tr);
  }
  editorArea.appendChild(table);
}

// Color de texto
colorPicker.onchange=()=>document.execCommand('foreColor',false,colorPicker.value);

// Tamaño de letra
fontSizeSelect.onchange=()=>{ const size=fontSizeSelect.value; if(size) document.execCommand('fontSize',false,size); };

// Volver al inicio
document.getElementById('backBtn').onclick=()=>{ editor.classList.remove('active'); home.classList.add('active'); loadFiles(); }

// Formato
function format(cmd,value=null){ document.execCommand(cmd,false,value); }

// Eliminar carpeta o documento desde home
function deleteItem(index){
  if(!confirm('¿Seguro que quieres eliminar este elemento?')) return;
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  data.splice(index,1);
  localStorage.setItem('vitvisor_files',JSON.stringify(data));
  loadFiles();
}

// Eliminar documento dentro de carpeta
function deleteDoc(docIndex){
  if(!confirm('¿Seguro que quieres eliminar este documento?')) return;
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  if(currentFolder!==null){
    data[currentFolder].files.splice(docIndex,1);
    localStorage.setItem('vitvisor_files',JSON.stringify(data));
    openFolder(currentFolder);
  }
}

loadFiles();
</script>
</body>
</html>
