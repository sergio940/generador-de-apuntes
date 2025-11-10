<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vitvisor Docs - Álbum</title>
<style>
body {
  font-family: 'Segoe UI', sans-serif;
  margin: 0;
  background: #f4f6f9;
  color: #222;
}
header {
  background: #0078d7;
  color: white;
  padding: 15px;
  text-align: center;
  font-size: 22px;
  font-weight: bold;
}
#home, #editor {
  display: none;
  padding: 30px;
}
#home.active, #editor.active {
  display: block;
}
button {
  background: #0078d7;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 15px;
  transition: 0.3s;
}
button:hover {
  background: #005a9e;
}
#fileList {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
}
.folderCard, .docCard {
  background: white;
  border-radius: 12px;
  padding: 20px;
  width: 140px;
  height: 140px;
  box-shadow: 0 3px 8px rgba(0,0,0,0.2);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  text-align: center;
  transition: transform 0.2s;
}
.folderCard:hover, .docCard:hover {
  transform: scale(1.05);
  background: #eef5ff;
}
.folderCard span, .docCard span {
  font-size: 40px;
  margin-bottom: 10px;
}
#editorToolbar {
  background: #fff;
  padding: 10px;
  border-bottom: 1px solid #ccc;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-items: center;
}
#editorToolbar button, #editorToolbar select, #editorToolbar input[type=color] {
  background: #0078d7;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 6px 10px;
  cursor: pointer;
}
#editorToolbar input[type=color] {
  width: 40px;
  height: 30px;
  padding: 0;
  border: 1px solid #ccc;
  cursor: pointer;
}
#editorArea {
  width: 100%;
  height: 70vh;
  padding: 30px;
  background: white;
  border: none;
  outline: none;
  overflow-y: auto;
}
#docTitle {
  margin: 15px 0;
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
const home = document.getElementById('home');
const editor = document.getElementById('editor');
const fileList = document.getElementById('fileList');
const editorArea = document.getElementById('editorArea');
const docTitle = document.getElementById('docTitle');
const colorPicker = document.getElementById('colorPicker');
const fontSizeSelect = document.getElementById('fontSizeSelect');
let currentDoc = null;
let currentFolder = null;

// Evitar caracteres extraños
document.addEventListener('keydown', e => { if (e.key==='Dead') e.preventDefault(); });

// Cargar archivos
function loadFiles() {
  fileList.innerHTML = '';
  const data = JSON.parse(localStorage.getItem('vitvisor_files') || '[]');

  data.forEach((item,i)=>{
    const div = document.createElement('div');
    div.className = item.type==='folder'?'folderCard':'docCard';
    div.innerHTML = `<span>${item.type==='folder'?'📁':'📝'}</span>${item.name}`;
    div.onclick = ()=>{ item.type==='folder'?openFolder(i):openDoc(i,null); };
    fileList.appendChild(div);
  });
}

// Crear carpeta
document.getElementById('newFolderBtn').onclick = ()=>{
  const name=prompt('Nombre de la carpeta:'); if(!name) return;
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  data.push({type:'folder',name,files:[]});
  localStorage.setItem('vitvisor_files',JSON.stringify(data));
  loadFiles();
};

// Crear documento
document.getElementById('newDocBtn').onclick = ()=>{
  const name=prompt('Nombre del documento:'); if(!name) return;
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  if(currentFolder!==null) data[currentFolder].files.push({type:'doc',name,content:''});
  else data.push({type:'doc',name,content:''});
  localStorage.setItem('vitvisor_files',JSON.stringify(data));
  loadFiles();
};

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
document.getElementById('saveBtn').onclick = ()=>{
  if(currentDoc===null) return;
  const data=JSON.parse(localStorage.getItem('vitvisor_files')||'[]');
  if(currentFolder!==null) data[currentFolder].files[currentDoc].content=editorArea.innerHTML;
  else data[currentDoc].content=editorArea.innerHTML;
  localStorage.setItem('vitvisor_files',JSON.stringify(data));
  alert('Documento guardado 💾');
};

// Imagen
document.getElementById('addImgBtn').onclick = ()=>{
  const url=prompt('Introduce la URL de la imagen:'); if(url) document.execCommand('insertImage',false,url);
};

// Tabla
document.getElementById('addTableBtn').onclick = ()=>{
  const rows=parseInt(prompt('Número de filas:',2)); if(!rows) return;
  const cols=parseInt(prompt('Número de columnas:',2)); if(!cols) return;
  const table=document.createElement('table');
  for(let r=0;r<rows;r++){
    const tr=document.createElement('tr');
    for(let c=0;c<cols;c++){
      const td=document.createElement('td'); td.textContent='Texto';
      td.contentEditable=true; td.onclick=()=>{
        const bg=prompt('Color de fondo (hex o nombre):',td.style.backgroundColor);
        if(bg) td.style.backgroundColor=bg;
      };
      tr.appendChild(td);
    }
    table.appendChild(tr);
  }
  editorArea.appendChild(table);
};

// Color de texto
colorPicker.onchange = ()=>document.execCommand('foreColor', false, colorPicker.value);

// Tamaño de letra
fontSizeSelect.onchange = ()=>{ const size=fontSizeSelect.value; if(size) document.execCommand('fontSize', false, size); };

// Volver al inicio
document.getElementById('backBtn').onclick = ()=>{
  editor.classList.remove('active'); home.classList.add('active'); loadFiles();
};

// Formato
function format(cmd,value=null){ document.execCommand(cmd,false,value); }

loadFiles();
</script>
</body>
</html>
