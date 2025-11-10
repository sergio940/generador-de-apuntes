<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Word Local - Vitvisor Notes</title>
<style>
    * { box-sizing: border-box; }
    body {
        margin: 0;
        font-family: "Segoe UI", sans-serif;
        background-color: #f1f5f9;
        height: 100vh;
        display: flex;
        flex-direction: column;
    }

    header {
        background-color: #1e293b;
        color: white;
        padding: 15px 30px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        font-size: 20px;
        font-weight: bold;
    }

    /* Pantalla inicial */
    #inicio, #editorView {
        flex: 1;
        display: none;
    }
    #inicio.active, #editorView.active {
        display: flex;
    }

    #inicio {
        justify-content: center;
        align-items: flex-start;
        background-color: #f8fafc;
        padding: 50px;
        gap: 40px;
        flex-wrap: wrap;
        overflow-y: auto;
    }

    .doc-card {
        width: 220px;
        height: 260px;
        background-color: white;
        border-radius: 10px;
        box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        padding: 15px;
        cursor: pointer;
        transition: 0.2s;
    }

    .doc-card:hover {
        transform: scale(1.03);
        border: 3px solid #0ea5e9;
    }

    .doc-card h3 {
        margin: 0;
        font-size: 16px;
        color: #0f172a;
        text-align: center;
    }

    .preview {
        background-color: #f1f5f9;
        flex-grow: 1;
        padding: 10px;
        font-size: 13px;
        overflow: hidden;
        color: #475569;
        border-radius: 6px;
    }

    /* Botón de nuevo documento */
    .nuevo {
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 40px;
        color: #0ea5e9;
        background-color: white;
        border: 3px dashed #0ea5e9;
        transition: 0.2s;
    }
    .nuevo:hover {
        background-color: #e0f2fe;
    }

    /* Editor */
    #toolbar {
        background-color: #0ea5e9;
        display: flex;
        gap: 10px;
        padding: 10px;
        flex-wrap: wrap;
    }

    #toolbar button {
        background: white;
        border: none;
        padding: 8px 14px;
        border-radius: 6px;
        color: #0ea5e9;
        font-weight: bold;
        cursor: pointer;
        transition: 0.2s;
    }

    #toolbar button:hover {
        background-color: #e0f2fe;
    }

    #editor {
        flex: 1;
        padding: 20px;
        background-color: white;
        overflow-y: auto;
        line-height: 1.6;
        font-size: 17px;
        border: none;
        outline: none;
    }
</style>
</head>
<body>

<header>📝 Vitvisor Word Local</header>

<!-- Vista inicial -->
<div id="inicio" class="active" id="homeView"></div>

<!-- Vista del editor -->
<div id="editorView">
    <div id="toolbar">
        <button onclick="guardarArchivo()">💾 Guardar</button>
        <button onclick="volverInicio()">🏠 Inicio</button>
        <button onclick="document.execCommand('bold',false,null)">𝐍</button>
        <button onclick="document.execCommand('italic',false,null)">𝑰</button>
        <button onclick="document.execCommand('underline',false,null)">U̲</button>
        <button onclick="insertarImagen()">🖼️ Imagen</button>
        <button onclick="exportarDOC()">⬇️ Exportar Word</button>
    </div>
    <div id="editor" contenteditable="true"></div>
</div>

<script>
let documentos = JSON.parse(localStorage.getItem("word_vitvisor_docs")) || {};
let archivoActual = null;

function guardarDatos() {
    localStorage.setItem("word_vitvisor_docs", JSON.stringify(documentos));
}

function renderInicio() {
    const contenedor = document.getElementById("inicio");
    contenedor.innerHTML = "";

    // Botón nuevo documento
    const nuevo = document.createElement("div");
    nuevo.className = "doc-card nuevo";
    nuevo.innerHTML = "+";
    nuevo.onclick = nuevoDocumento;
    contenedor.appendChild(nuevo);

    // Mostrar documentos guardados
    for (let nombre in documentos) {
        const doc = document.createElement("div");
        doc.className = "doc-card";
        doc.onclick = () => abrirDocumento(nombre);

        const titulo = document.createElement("h3");
        titulo.textContent = nombre;

        const prev = document.createElement("div");
        prev.className = "preview";
        prev.innerHTML = documentos[nombre].substring(0, 200) || "<i>Vacío...</i>";

        doc.appendChild(titulo);
        doc.appendChild(prev);
        contenedor.appendChild(doc);
    }
}

function nuevoDocumento() {
    const nombre = prompt("Nombre del nuevo documento:");
    if (nombre && !documentos[nombre]) {
        documentos[nombre] = "";
        archivoActual = nombre;
        cambiarVista("editor");
        document.getElementById("editor").innerHTML = "";
        guardarDatos();
        renderInicio();
    } else if (documentos[nombre]) {
        alert("Ya existe un documento con ese nombre.");
    }
}

function abrirDocumento(nombre) {
    archivoActual = nombre;
    cambiarVista("editor");
    document.getElementById("editor").innerHTML = documentos[nombre];
}

function guardarArchivo() {
    if (!archivoActual) return;
    documentos[archivoActual] = document.getElementById("editor").innerHTML;
    guardarDatos();
    alert("✅ Documento guardado correctamente.");
}

function volverInicio() {
    guardarArchivo();
    cambiarVista("inicio");
    renderInicio();
}

function insertarImagen() {
    const input = document.createElement("input");
    input.type = "file";
    input.accept = "image/*";
    input.onchange = e => {
        const file = e.target.files[0];
        if (!file) return;
        const reader = new FileReader();
        reader.onload = function(evt) {
            const img = document.createElement("img");
            img.src = evt.target.result;
            img.style.maxWidth = "400px";
            img.style.display = "block";
            img.style.margin = "10px 0";
            document.getElementById("editor").appendChild(img);
        };
        reader.readAsDataURL(file);
    };
    input.click();
}

function exportarDOC() {
    if (!archivoActual) return;
    const contenido = document.getElementById("editor").innerHTML;
    const blob = new Blob(['<html><body>' + contenido + '</body></html>'], { type: 'application/msword' });
    const enlace = document.createElement("a");
    enlace.href = URL.createObjectURL(blob);
    enlace.download = archivoActual + ".doc";
    enlace.click();
}

function cambiarVista(vista) {
    document.getElementById("inicio").classList.remove("active");
    document.getElementById("editorView").classList.remove("active");
    if (vista === "inicio") document.getElementById("inicio").classList.add("active");
    else document.getElementById("editorView").classList.add("active");
}

renderInicio();
</script>

</body>
</html>
