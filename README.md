<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bloc de Apuntes Local</title>
<style>
    body {
        font-family: "Segoe UI", sans-serif;
        display: flex;
        height: 100vh;
        margin: 0;
        background-color: #f3f4f6;
    }
    #sidebar {
        width: 280px;
        background: #1e293b;
        color: white;
        padding: 15px;
        overflow-y: auto;
    }
    #sidebar h2 {
        margin-top: 0;
        text-align: center;
        color: #38bdf8;
    }
    #content {
        flex-grow: 1;
        display: flex;
        flex-direction: column;
    }
    #editor {
        flex-grow: 1;
        padding: 15px;
        outline: none;
        border: none;
        font-size: 16px;
        resize: none;
        background: #ffffff;
    }
    button {
        background-color: #38bdf8;
        color: white;
        border: none;
        padding: 8px 12px;
        border-radius: 6px;
        margin: 4px 0;
        cursor: pointer;
        font-size: 14px;
    }
    button:hover {
        background-color: #0284c7;
    }
    .folder, .file {
        margin-left: 10px;
        cursor: pointer;
        padding: 4px 6px;
        border-radius: 5px;
    }
    .folder:hover, .file:hover {
        background-color: #334155;
    }
    .active {
        background-color: #38bdf8 !important;
        color: black;
    }
</style>
</head>
<body>

<div id="sidebar">
    <h2>📁 Mis Apuntes</h2>
    <button onclick="crearCarpeta()">➕ Nueva Carpeta</button>
    <div id="estructura"></div>
</div>

<div id="content">
    <textarea id="editor" placeholder="Selecciona o crea un archivo para escribir..."></textarea>
</div>

<script>
let datos = JSON.parse(localStorage.getItem("apuntes_locales")) || {};
let carpetaActual = null;
let archivoActual = null;

function guardarDatos() {
    localStorage.setItem("apuntes_locales", JSON.stringify(datos));
}

function renderizarEstructura() {
    const estructura = document.getElementById("estructura");
    estructura.innerHTML = "";
    for (let carpeta in datos) {
        const divCarpeta = document.createElement("div");
        divCarpeta.className = "folder";
        divCarpeta.textContent = "📂 " + carpeta;
        divCarpeta.onclick = () => seleccionarCarpeta(carpeta);

        // Botón nuevo archivo
        const btnArchivo = document.createElement("button");
        btnArchivo.textContent = "📄 +";
        btnArchivo.style.marginLeft = "10px";
        btnArchivo.onclick = (e) => {
            e.stopPropagation();
            crearArchivo(carpeta);
        };
        divCarpeta.appendChild(btnArchivo);
        estructura.appendChild(divCarpeta);

        // Archivos dentro
        for (let archivo in datos[carpeta]) {
            const divArchivo = document.createElement("div");
            divArchivo.className = "file";
            divArchivo.textContent = "📝 " + archivo;
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

function seleccionarCarpeta(nombre) {
    carpetaActual = nombre;
    archivoActual = null;
    document.getElementById("editor").value = "";
}

function seleccionarArchivo(carpeta, archivo) {
    carpetaActual = carpeta;
    archivoActual = archivo;
    document.getElementById("editor").value = datos[carpeta][archivo];
}

document.getElementById("editor").addEventListener("input", () => {
    if (carpetaActual && archivoActual) {
        datos[carpetaActual][archivoActual] = document.getElementById("editor").value;
        guardarDatos();
    }
});

renderizarEstructura();
</script>

</body>
</html>
