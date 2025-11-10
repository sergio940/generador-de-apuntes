<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gestor de Apuntes Local</title>
<style>
    * {
        box-sizing: border-box;
    }
    body {
        margin: 0;
        font-family: "Segoe UI", sans-serif;
        display: flex;
        height: 100vh;
        background-color: #f1f5f9;
    }

    /* Sidebar */
    #sidebar {
        width: 320px;
        background-color: #1e293b;
        color: white;
        display: flex;
        flex-direction: column;
        padding: 20px;
        overflow-y: auto;
        border-right: 3px solid #0ea5e9;
    }

    #sidebar h2 {
        text-align: center;
        color: #38bdf8;
        margin-bottom: 20px;
    }

    button {
        background-color: #38bdf8;
        color: white;
        border: none;
        padding: 8px 12px;
        border-radius: 6px;
        margin: 6px 0;
        cursor: pointer;
        font-size: 14px;
        width: 100%;
        font-weight: bold;
        transition: background 0.2s ease;
    }

    button:hover {
        background-color: #0284c7;
    }

    .folder, .file {
        margin: 4px 0 4px 10px;
        padding: 6px;
        cursor: pointer;
        border-radius: 5px;
        transition: 0.2s;
    }

    .folder:hover, .file:hover {
        background-color: #334155;
    }

    .file.active {
        background-color: #38bdf8;
        color: black;
        font-weight: bold;
    }

    /* Editor */
    #main {
        flex: 1;
        display: flex;
        flex-direction: column;
    }

    #toolbar {
        background-color: #0ea5e9;
        padding: 10px;
        display: flex;
        gap: 10px;
        justify-content: flex-start;
    }

    #toolbar button {
        width: auto;
        padding: 8px 15px;
        border-radius: 8px;
        background-color: white;
        color: #0ea5e9;
        font-weight: bold;
    }

    #toolbar button:hover {
        background-color: #f0f9ff;
    }

    #editor {
        flex: 1;
        padding: 20px;
        font-size: 18px;
        border: none;
        outline: none;
        resize: none;
        background-color: white;
        width: 100%;
        height: 100%;
        line-height: 1.6;
        color: #0f172a;
    }

</style>
</head>
<body>

<div id="sidebar">
    <h2>📁 Mis Apuntes</h2>
    <button onclick="crearCarpeta()">➕ Nueva Carpeta</button>
    <div id="estructura"></div>
</div>

<div id="main">
    <div id="toolbar">
        <button onclick="guardarArchivo()">💾 Guardar</button>
        <button onclick="renombrarArchivo()">✏️ Renombrar</button>
        <button onclick="eliminarArchivo()">🗑️ Eliminar</button>
        <button onclick="exportarArchivo()">⬇️ Exportar .txt</button>
    </div>
    <textarea id="editor" placeholder="Crea o selecciona un archivo para escribir tus apuntes..."></textarea>
</div>

<script>
let datos = JSON.parse(localStorage.getItem("apuntes_locales_v2")) || {};
let carpetaActual = null;
let archivoActual = null;

function guardarDatos() {
    localStorage.setItem("apuntes_locales_v2", JSON.stringify(datos));
}

function renderizarEstructura() {
    const estructura = document.getElementById("estructura");
    estructura.innerHTML = "";
    for (let carpeta in datos) {
        const divCarpeta = document.createElement("div");
        divCarpeta.className = "folder";
        divCarpeta.innerHTML = `📂 <strong>${carpeta}</strong>`;

        const btnArchivo = document.createElement("button");
        btnArchivo.textContent = "📄 Nuevo archivo";
        btnArchivo.style.marginTop = "5px";
        btnArchivo.style.fontSize = "12px";
        btnArchivo.onclick = (e) => {
            e.stopPropagation();
            crearArchivo(carpeta);
        };

        estructura.appendChild(divCarpeta);
        estructura.appendChild(btnArchivo);

        for (let archivo in datos[carpeta]) {
            const divArchivo = document.createElement("div");
            divArchivo.className = "file";
            divArchivo.textContent = "📝 " + archivo;
            if (carpeta === carpetaActual && archivo === archivoActual) {
                divArchivo.classList.add("active");
            }
            divArchivo.onclick = () => seleccionarArchivo(carpeta, archivo);
            estructura.appendChild(divArchivo);
        }
    }
}

function crearCarpeta() {
    const nombre = prompt("Nombre de la nueva carpeta:");
    if (nombre && !datos[nombre]) {
        datos[nombre] = {};
        guardarDatos();
        renderizarEstructura();
    }
}

function crearArchivo(carpeta) {
    const nombre = prompt("Nombre del nuevo archivo:");
    if (nombre && !datos[carpeta][nombre]) {
        datos[carpeta][nombre] = "";
        guardarDatos();
        renderizarEstructura();
    }
}

function seleccionarArchivo(carpeta, archivo) {
    carpetaActual = carpeta;
    archivoActual = archivo;
    document.getElementById("editor").value = datos[carpeta][archivo];
    renderizarEstructura();
}

function guardarArchivo() {
    if (!carpetaActual || !archivoActual) {
        alert("Selecciona un archivo antes de guardar.");
        return;
    }
    datos[carpetaActual][archivoActual] = document.getElementById("editor").value;
    guardarDatos();
    alert("✅ Archivo guardado correctamente.");
}

function renombrarArchivo() {
    if (!carpetaActual || !archivoActual) {
        alert("Selecciona un archivo para renombrar.");
        return;
    }
    const nuevoNombre = prompt("Nuevo nombre del archivo:", archivoActual);
    if (nuevoNombre && nuevoNombre !== archivoActual) {
        datos[carpetaActual][nuevoNombre] = datos[carpetaActual][archivoActual];
        delete datos[carpetaActual][archivoActual];
        archivoActual = nuevoNombre;
        guardarDatos();
        renderizarEstructura();
    }
}

function eliminarArchivo() {
    if (!carpetaActual || !archivoActual) {
        alert("Selecciona un archivo para eliminar.");
        return;
    }
    if (confirm(`¿Seguro que deseas eliminar "${archivoActual}"?`)) {
        delete datos[carpetaActual][archivoActual];
        document.getElementById("editor").value = "";
        archivoActual = null;
        guardarDatos();
        renderizarEstructura();
    }
}

function exportarArchivo() {
    if (!carpetaActual || !archivoActual) {
        alert("Selecciona un archivo para exportar.");
        return;
    }
    const contenido = datos[carpetaActual][archivoActual];
    const blob = new Blob([contenido], { type: "text/plain" });
    const enlace = document.createElement("a");
    enlace.href = URL.createObjectURL(blob);
    enlace.download = archivoActual + ".txt";
    enlace.click();
}

renderizarEstructura();
</script>

</body>
</html>
