<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vitvisor Docs</title>
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
.folder, .document {
  background: white;
  border-radius: 10px;
  padding: 15px;
  margin: 10px 0;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  cursor: pointer;
}
.folder:hover, .document:hover {
  background: #eef5ff;
}
#editorToolbar {
  background: #fff;
  padding: 10px;
  border-bottom: 1px solid #ccc;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}
#editorToolbar button, #editorToolbar select {
  background: #0078d7;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 6px 10px;
  cursor: pointer;
}
#editorArea {
  width: 100%;
  height: 70vh;
  padding: 20px;
  background: white;
  border: none;
  outline: none;
  resize: none;
  overflow-y: auto;
}
#docTitle {
  margin: 15px 0;
}
</style>
</head>
<body>
<header>📘 Vitvisor Docs</header>

<!-- PANTALLA PRINCIPAL -->
<section id="home" class="active">
  <button id="newFolderBtn">📁 Nueva carpeta</button>
  <button id="newDocBtn">📝 Nuevo documento</button>
  <h2>Tus archivos</h2>
  <div id="fileList"></div>
</section>

<!-- EDITOR -->
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
    <select onchange="format('fontSize', this.value)">
      <option value="">Tamaño</option>
      <option value="2">Pequeño</option>
      <option value="3">Normal</option>
      <option value="5">Grande</option>
      <option value="7">Muy grande</option>
    </select>
    <button id="addImgBtn">🖼️ Imagen</button>
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
let currentDoc = null;
let currentFolder = null;

// Función: cargar archivos y carpetas
function loadFiles() {
  fileList.innerHTML = '';
  const data = JSON.parse(localStorage.getItem('vitvisor_files') || '[]');

  data.forEach((item, i) => {
    const div = document.createElement('div');
    div.className = item.type === 'folder' ? 'folder' : 'document';
    div.textContent = item.name;
    div.onclick = () => {
      if (item.type === 'folder') {
        openFolder(i);
      } else {
        openDoc(i, null);
      }
    };
    fileList.appendChild(div);
  });
}

// Crear nueva carpeta
document.getElementById('newFolderBtn').onclick = () => {
  const name = prompt('Nombre de la carpeta:');
  if (!name) return;
  const data = JSON.parse(localStorage.getItem('vitvisor_files') || '[]');
  data.push({ type: 'folder', name, files: [] });
  localStorage.setItem('vitvisor_files', JSON.stringify(data));
  loadFiles();
};

// Crear nuevo documento
document.getElementById('newDocBtn').onclick = () => {
  const name = prompt('Nombre del documento:');
  if (!name) return;
  const data = JSON.parse(localStorage.getItem('vitvisor_files') || '[]');
  if (currentFolder !== null) {
    data[currentFolder].files.push({ type: 'doc', name, content: '' });
  } else {
    data.push({ type: 'doc', name, content: '' });
  }
  localStorage.setItem('vitvisor_files', JSON.stringify(data));
  loadFiles();
};

// Abrir carpeta
function openFolder(index) {
  currentFolder = index;
  fileList.innerHTML = '';
  const data = JSON.parse(localStorage.getItem('vitvisor_files') || '[]');
  const folder = data[index];

  const back = document.createElement('button');
  back.textContent = '⬅️ Volver';
  back.onclick = () => { currentFolder = null; loadFiles(); };
  fileList.appendChild(back);

  folder.files.forEach((f, j) => {
    const div = document.createElement('div');
    div.className = 'document';
    div.textContent = f.name;
    div.onclick = () => openDoc(j, index);
    fileList.appendChild(div);
  });

  const newDocBtn = document.createElement('button');
  newDocBtn.textContent = '📝 Nuevo documento';
  newDocBtn.onclick = () => {
    const name = prompt('Nombre del documento:');
    if (!name) return;
    folder.files.push({ type: 'doc', name, content: '' });
    data[index] = folder;
    localStorage.setItem('vitvisor_files', JSON.stringify(data));
    openFolder(index);
  };
  fileList.appendChild(newDocBtn);
}

// Abrir documento
function openDoc(docIndex, folderIndex) {
  const data = JSON.parse(localStorage.getItem('vitvisor_files') || '[]');
  let doc;
  if (folderIndex !== null) {
    doc = data[folderIndex].files[docIndex];
    currentFolder = folderIndex;
  } else {
    doc = data[docIndex];
    currentFolder = null;
  }
  currentDoc = docIndex;
  docTitle.textContent = doc.name;
  editorArea.innerHTML = doc.content;
  home.classList.remove('active');
  editor.classList.add('active');
}

// Guardar documento
document.getElementById('saveBtn').onclick = () => {
  if (currentDoc === null) return;
  const data = JSON.parse(localStorage.getItem('vitvisor_files') || '[]');
  if (currentFolder !== null) {
    data[currentFolder].files[currentDoc].content = editorArea.innerHTML;
  } else {
    data[currentDoc].content = editorArea.innerHTML;
  }
  localStorage.setItem('vitvisor_files', JSON.stringify(data));
  alert('Documento guardado 💾');
};

// Insertar imagen
document.getElementById('addImgBtn').onclick = () => {
  const url = prompt('Introduce la URL de la imagen:');
  if (url) document.execCommand('insertImage', false, url);
};

// Volver al inicio
document.getElementById('backBtn').onclick = () => {
  editor.classList.remove('active');
  home.classList.add('active');
  loadFiles();
};

// Funciones de formato
function format(cmd, value = null) {
  document.execCommand(cmd, false, value);
}

// Inicializar
loadFiles();
</script>
</body>
</html>
