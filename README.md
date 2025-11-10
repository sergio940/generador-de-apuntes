<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gestor de Apuntes Local - Estilo Word</title>
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

    /* Barra superior */
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

    /* Contenedor principal */
    #inicio, #editorView {
        flex: 1;
        display: none;
    }
    #inicio.active, #editorView.active {
        display: flex;
    }

    /* Vista tipo Word */
    #inicio {
        justify-content: center;
        align-items: center;
        background-color: #f8fafc;
        padding: 40px;
    }

    .inicio-container {
        display: flex;
        width: 80%;
        gap: 50px;
    }

    .nuevo-panel {
        width: 50%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .nuevo-doc {
        background-color: white;
        border: 3px solid #0ea5e9;
        width: 180px;
        height: 220px;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 20px;
        color: #0ea5e9;
        border-radius: 10px;
        cursor: pointer;
        transition: 0.2s;
        font-weight: bold;
    }

    .nuevo-doc:hover {
        background-color: #e0f2fe;
        transform: scale(1.03);
    }

    .recientes-panel {
        width: 50%;
        background-color: #ffffff;
        border-radius: 10px;
        padding: 20px;
        box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }

    .recientes-panel h3 {
        color: #0ea5e9;
        border-bottom: 2px solid #0ea5e9;
        padding-bottom: 8px;
        margin-top: 0;
    }

    .archivo-item {
        padding: 10px;
        border-radius: 5px;
        margin: 5px 0;
        cursor: pointer;
        transition: 0.2s;
    }

    .archivo-item:hover {
        background-color: #e2e8f0;
    }

    /* Vista del editor */
    #editorView {
        flex-direction: column;
        height: 100%;
    }

    #toolbar {
        background-color: #0ea5e9;
        display: flex;
        gap: 10px;
        padding: 10px;
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
        border: none;
        outline: none;
        padding: 20px;
        font-size: 18px;
        resize: none;
        line-height: 1.6;
        background-color: #ffffff;
        width: 100%;
    }
</style>
</head>
<body>

<header>
    📘 Gestor de Apuntes Local
</header>

<!-- Pantalla inicial -->
<div id="inicio" class="active">
    <div class="inicio-container">
        <div class="nuevo-panel">
            <div class="nuevo-doc" onclick="nuevoDocumento()">📝 Nuevo Documento</div>
        </div>
        <div class="recientes-panel">
            <h3>Documentos guardados</h3>
            <div id="listaDocumentos"></div>
        </div>
    </div>
</div>

<!-- Vista del editor -->
<div id="editorView">
    <div id="toolbar">
        <button onclick="guardarArchivo()">💾 Guardar</button>
        <button onclick="renombrarArchivo()">✏️ Renombrar</button>
        <button onclick="volverInicio()">🏠 Volver</button>
        <button onclick="exportarArchivo()">⬇️ Exportar .txt</button>
    </div>
    <textarea id="editor" placeholder="Escribe aquí tus apuntes..."></textarea>
</div>

<script>
let documentos = JSON.parse(localStorage.getItem("apuntes_wordstyle")) || {};
let archivoActual = null;

function guardarDatos() {
    localStorage.setItem("apuntes_wordstyle", JSON.stringify(documentos));
}

function renderizarLista() {
    const contenedor = document.getElementById("listaDocumentos");
    contenedor.innerHTML = "";

    if (Object.keys(documentos).length === 0) {
        contenedor.innerHTML = "<p style='color:#94a3b8'>No hay documentos aún.</p>";
        return;
    }

    for (let nombre in documentos) {
        const div = document.createElement("div");
        div.className = "archivo-item";
        div.textContent = "📄 " + nombre;
        div.onclick = () => abrirDocumento(nombre);
        contenedor.appendChild(div);
    }
}

function nuevoDocumento() {
    const nombre = prompt("Nombre del nuevo documento:");
    if (nombre && !documentos[nombre]) {
        documentos[nombre] = "";
        archivoActual = nombre;
        cambiarVista("editor");
        document.getElementById("editor").value = "";
        guardarDatos();
        renderizarLista();
    } else if (documentos[nombre]) {
        alert("Ya existe un documento con ese nombre.");
    }
}

function abrirDocumento(nombre) {
    archivoActual = nombre;
    document.getElementById("editor").value = documentos[nombre];
    cambiarVista("editor");
}

function guardarArchivo() {
    if (!archivoActual) return;
    documentos[archivoActual] = document.getElementById("editor").value;
    guardarDatos();
    alert("✅ Documento guardado correctamente.");
}

function renombrarArchivo() {
    if (!archivoActual) return;
    const nuevo = prompt("Nuevo nombre:", archivoActual);
    if (nuevo && nuevo !== archivoActual) {
        documentos[nuevo] = documentos[archivoActual];
        delete documentos[archivoActual];
        archivoActual = nuevo;
        guardarDatos();
        renderizarLista();
        alert("✅ Documento renombrado.");
    }
}

function volverInicio() {
    guardarArchivo();
    cambiarVista("inicio");
    renderizarLista();
}

function exportarArchivo() {
    if (!archivoActual) return;
    const contenido = documentos[archivoActual];
    const blob = new Blob([contenido], { type: "text/plain" });
    const enlace = document.createElement("a");
    enlace.href = URL.createObjectURL(blob);
    enlace.download = archivoActual + ".txt";
    enlace.click();
}

function cambiarVista(vista) {
    document.getElementById("inicio").classList.remove("active");
    document.getElementById("editorView").classList.remove("active");

    if (vista === "inicio") document.getElementById("inicio").classList.add("active");
    else document.getElementById("editorView").classList.add("active");
}

renderizarLista();
</script>

</body>
</html>
