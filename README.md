<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vitvisor Docs</title>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
<style>
:root {
  --primary: #1E88E5;
  --secondary: #42A5F5;
  --bg: #f4f6f9;
  --card-bg: #fff;
  --card-hover: #e3f2fd;
  --toolbar-bg: #1976D2;
  --toolbar-btn: #2196F3;
  --toolbar-btn-hover: #1565C0;
  --text-color: #222;
  --border-radius: 12px;
}

* { box-sizing: border-box; }

body {
  font-family: 'Roboto', sans-serif;
  margin: 0;
  background: var(--bg);
  color: var(--text-color);
}

header {
  background: var(--primary);
  color: white;
  padding: 18px;
  text-align: center;
  font-size: 26px;
  font-weight: 700;
  box-shadow: 0 3px 6px rgba(0,0,0,0.1);
}

#home, #editor {
  display: none;
  padding: 25px;
}
#home.active, #editor.active { display: block; }

button {
  font-family: inherit;
  background: var(--toolbar-btn);
  color: white;
  border: none;
  border-radius: var(--border-radius);
  padding: 10px 14px;
  cursor: pointer;
  font-size: 14px;
  transition: 0.3s;
}
button:hover { background: var(--toolbar-btn-hover); }

#fileList {
  display: flex;
  flex-wrap: wrap;
  gap: 18px;
  margin-top: 20px;
}

.folderCard, .docCard {
  background: var(--card-bg);
  border-radius: var(--border-radius);
  padding: 20px;
  width: 150px;
  height: 150px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  text-align: center;
  transition: 0.3s transform, 0.3s background;
}
.folderCard:hover, .docCard:hover {
  transform: translateY(-5px);
  background: var(--card-hover);
}
.folderCard span, .docCard span {
  font-size: 50px;
  margin-bottom: 10px;
}

#editorToolbar {
  background: var(--toolbar-bg);
  padding: 12px;
  border-bottom: 1px solid #ccc;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
  border-radius: var(--border-radius) var(--border-radius) 0 0;
}
#editorToolbar button, #editorToolbar select, #editorToolbar input[type=color] {
  background: var(--toolbar-btn);
  color: white;
  border: none;
  border-radius: var(--border-radius);
  padding: 6px 12px;
  cursor: pointer;
}
#editorToolbar input[type=color] {
  width: 36px;
  height: 28px;
  padding: 0;
  border: 1px solid #ccc;
  cursor: pointer;
}
#editorToolbar button:hover, #editorToolbar select:hover { background: var(--toolbar-btn-hover); }

#editorArea {
  width: 100%;
  height: 70vh;
  padding: 28px;
  background: var(--card-bg);
  border-radius: var(--border-radius);
  outline: none;
  overflow-y: auto;
  margin-top: 10px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

#docTitle {
  margin: 18px 0;
}

table {
  border-collapse: collapse;
  width: 100%;
}
table td, table th {
  border: 1px solid #000;
  padding: 8px;
  text-align: center;
}
</style>
</head>
<body>
<header>📘 Vitvisor Docs</header>

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
// Variables globales
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
    div.innerHTML=`<span>${item.type==='folder'?'📁':'📝'}</span>${item.name}`;
    div.onclick=()=>{item.type==='folder'?openFolder(i):openDoc(i,null);};
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
    const div=document.createElement('div'); div.className='docCard'; div.innerHTML=`<span>📝</span>${f.name}`;
    div.onclick=()=>openDoc(j,index); fileList.appendChild(div);
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
  table.style.border='1px solid #000';
  table.style.width='100%';
  for(let r=0;r<rows;r++){
    const tr=document.createElement('tr');
    for(let c=0;c<cols;c++){
      const td=document.createElement('td'); td.textContent='Texto';
      td.contentEditable=true;
      td.style.border='1px solid #000';
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

loadFiles();
</script>
</body>
</html>
