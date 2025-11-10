<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vitvisor Docs</title>
<style>
body {
  font-family: 'Segoe UI', sans-serif;
  margin: 0;
  background: #f2f4f8;
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
#newDocBtn, #newFolderBtn {
  background: #0078d7;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  margin-right: 10px;
}
.folder, .document {
  background: white;
  border-radius: 10px;
  padding: 15px;
  margin: 10px 0;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  cursor: pointer;
}
#editorToolbar {
  background: #fff;
  padding: 10px;
  border-bottom: 1px solid #ccc;
  display: flex;
  gap: 10px;
}
#editorToolbar button {
  background: #0078d7;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 6px 12px;
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
</style>
</head>
<body>
<header>📘 Vitvisor Docs</header>

<!-- PANTALLA PRINCIPAL -->
<section id="home" class="active">
  <button id="newFolderBtn">Nueva carpeta 📁</button>
  <button id="newDocBtn">Nuevo documento 📝</button>
  <h2>Tus archivos</h2>
  <div id="fileList"></div>
</section>

<!-- EDITOR -->
<section id="editor">
  <div id="editorToolbar">
    <button id="backBtn">🏠 Inicio</button>
    <button id="saveBtn">💾 Guardar</button>
    <button id="addImgBtn">🖼️ Insertar imagen</button>
    <button id="exportBtn">📤 Exportar Word</button>
  </div>
  <h2 id="docTitle"></h2>
  <div id="editorArea" contenteditable="true"></div>
</section>

<script src="https://cdn.jsdelivr.net/npm/docx@8.1.0/build/index.umd.js"></script>
<script>
const home = document.getElementById('home');
const editor = document.getElementById('editor');
const newDocBtn = document.getElementById('newDocBtn');
const fileList = document.getElementById('fileList');
const editorArea = document.getElementById('editorArea');
const docTitle = document.getElementById('docTitle');
let currentDoc = null;

// Cargar archivos
function loadFiles() {
  fileList.innerHTML = '';
  const docs = JSON.parse(localStorage.getItem('vitvisor_docs') || '[]');
  docs.forEach((d, i) => {
    const div = document.createElement('div');
    div.className = 'document';
    div.textContent = d.title;
    div.onclick = () => openDoc(i);
    fileList.appendChild(div);
  });
}

// Nuevo documento
newDocBtn.onclick = () => {
  const title = prompt('Nombre del documento:');
  if (!title) return;
  const docs = JSON.parse(localStorage.getItem('vitvisor_docs') || '[]');
  docs.push({ title, content: '' });
  localStorage.setItem('vitvisor_docs', JSON.stringify(docs));
  loadFiles();
};

// Abrir documento
function openDoc(index) {
  const docs = JSON.parse(localStorage.getItem('vitvisor_docs') || '[]');
  currentDoc = index;
  docTitle.textContent = docs[index].title;
  editorArea.innerHTML = docs[index].content;
  home.classList.remove('active');
  editor.classList.add('active');
}

// Guardar
document.getElementById('saveBtn').onclick = () => {
  if (currentDoc === null) return;
  const docs = JSON.parse(localStorage.getItem('vitvisor_docs') || '[]');
  docs[currentDoc].content = editorArea.innerHTML;
  localStorage.setItem('vitvisor_docs', JSON.stringify(docs));
  alert('Documento guardado correctamente 💾');
};

// Volver al inicio
document.getElementById('backBtn').onclick = () => {
  editor.classList.remove('active');
  home.classList.add('active');
  loadFiles();
};

// Insertar imagen
document.getElementById('addImgBtn').onclick = () => {
  const url = prompt('URL de la imagen:');
  if (url) {
    const img = document.createElement('img');
    img.src = url;
    img.style.maxWidth = '100%';
    editorArea.appendChild(img);
  }
};

// Exportar a Word
document.getElementById('exportBtn').onclick = async () => {
  if (!window.docx) return alert('No se puede exportar, la librería no cargó.');
  const { Document, Packer, Paragraph } = window.docx;
  const text = editorArea.innerText;
  const doc = new Document({
    sections: [{ properties: {}, children: [new Paragraph(text)] }]
  });
  const blob = await Packer.toBlob(doc);
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `${docTitle.textContent}.docx`;
  a.click();
};

// Inicializar
loadFiles();
</script>
</body>
</html>
