<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MiniCloud Álbum Funcional</title>
<style>
body {
    font-family: Arial, sans-serif;
    background: #121212;
    color: #fff;
    margin: 0;
    padding: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.container {
    width: 90%;
    max-width: 900px;
    background: rgba(255,255,255,0.05);
    padding: 20px;
    border-radius: 12px;
    text-align: center;
}

input, button {
    padding: 8px;
    margin: 5px 0;
    border-radius: 6px;
    border: none;
}

button {
    background: #1db954;
    color: white;
    font-weight: bold;
    cursor: pointer;
}

button:hover {
    background: #1ed760;
}

#fileGrid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 15px;
    margin-top: 15px;
}

.fileCard {
    background: rgba(255,255,255,0.1);
    border-radius: 10px;
    padding: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
    position: relative;
}

.fileCard img {
    max-width: 100px;
    max-height: 100px;
    border-radius: 6px;
    margin-bottom: 8px;
}

.fileName {
    font-size: 0.85rem;
    word-break: break-word;
    text-align: center;
}

.menuBtn {
    position: absolute;
    top: 5px;
    right: 5px;
    background: transparent;
    color: #fff;
    border: none;
    font-size: 1.2rem;
    cursor: pointer;
}

.menuOptions {
    position: absolute;
    top: 25px;
    right: 5px;
    background: rgba(0,0,0,0.9);
    border-radius: 6px;
    display: none;
    flex-direction: column;
    width: 100px;
}

.menuOptions button {
    width: 100%;
    margin: 0;
    padding: 5px;
    border-radius: 0;
    background: none;
    color: #fff;
    text-align: left;
    font-size: 0.85rem;
}

.menuOptions button:hover {
    background: #1db954;
}

.hidden { display: none; }
</style>
</head>
<body>

<div class="container" id="authBox">
    <h2>☁️ MiniCloud Álbum</h2>
    <input type="text" id="username" placeholder="Usuario"><br>
    <input type="password" id="password" placeholder="Contraseña"><br>
    <button onclick="register()">Registrarse</button>
    <button onclick="login()">Iniciar sesión</button>
</div>

<div class="container hidden" id="cloudBox">
    <h3>Bienvenido, <span id="userDisplay"></span></h3>
    <button onclick="document.getElementById('fileInput').click()">➕ Nuevo</button>
    <input type="file" id="fileInput" multiple class="hidden"><br>
    <div id="fileGrid"></div>
    <button onclick="exportData()">Exportar archivos</button>
    <input type="file" id="importFile" class="hidden">
    <button onclick="document.getElementById('importFile').click()">Importar archivos</button>
    <button onclick="logout()">Cerrar sesión</button>
</div>

<script>
let currentUser = null;

// Registro
function register() {
    const user = document.getElementById("username").value.trim();
    const pass = document.getElementById("password").value.trim();
    if(!user || !pass) return alert("Completa usuario y contraseña.");
    if(localStorage.getItem("user_" + user)) return alert("Usuario ya existe.");
    localStorage.setItem("user_" + user, JSON.stringify({password: pass, files: []}));
    alert("Usuario registrado.");
}

// Login
function login() {
    const user = document.getElementById("username").value.trim();
    const pass = document.getElementById("password").value.trim();
    const data = localStorage.getItem("user_" + user);
    if(!data) return alert("Usuario no encontrado.");
    const account = JSON.parse(data);
    if(account.password !== pass) return alert("Contraseña incorrecta.");
    currentUser = user;
    document.getElementById("userDisplay").textContent = user;
    document.getElementById("authBox").classList.add("hidden");
    document.getElementById("cloudBox").classList.remove("hidden");
    loadFiles();
}

// Subida de archivos
document.getElementById("fileInput").onchange = async (e) => {
    const files = e.target.files;
    if(!files.length) return;
    const account = JSON.parse(localStorage.getItem("user_" + currentUser));
    for(const f of files) {
        const reader = new FileReader();
        reader.onload = () => {
            account.files.push({name: f.name, type: f.type, data: reader.result});
            localStorage.setItem("user_" + currentUser, JSON.stringify(account));
            renderFile(f); // Mostrar directamente en el álbum
        };
        reader.readAsDataURL(f);
    }
};

// Renderizar un archivo en el grid
function renderFile(f, index) {
    const grid = document.getElementById("fileGrid");

    const card = document.createElement("div");
    card.classList.add("fileCard");

    if(f.type.startsWith("image/")){
        const img = document.createElement("img");
        img.src = f.data;
        card.appendChild(img);
    } else {
        const placeholder = document.createElement("div");
        placeholder.style.height = "80px";
        placeholder.style.display = "flex";
        placeholder.style.alignItems = "center";
        placeholder.style.justifyContent = "center";
        placeholder.textContent = "📄";
        card.appendChild(placeholder);
    }

    const name = document.createElement("div");
    name.classList.add("fileName");
    name.textContent = f.name;
    card.appendChild(name);

    // Botón de menú
    const menuBtn = document.createElement("button");
    menuBtn.classList.add("menuBtn");
    menuBtn.textContent = "⋮";
    card.appendChild(menuBtn);

    const menu = document.createElement("div");
    menu.classList.add("menuOptions");

    const downloadBtn = document.createElement("button");
    downloadBtn.textContent = "Descargar";
    downloadBtn.onclick = () => {
        const link = document.createElement("a");
        link.href = f.data;
        link.download = f.name;
        link.click();
        menu.style.display = "none";
    };

    const deleteBtn = document.createElement("button");
    deleteBtn.textContent = "Eliminar";
    deleteBtn.onclick = () => {
        const account = JSON.parse(localStorage.getItem("user_" + currentUser));
        const idx = account.files.findIndex(file => file.name === f.name && file.data === f.data);
        if(idx > -1) account.files.splice(idx,1);
        localStorage.setItem("user_" + currentUser, JSON.stringify(account));
        grid.removeChild(card);
    };

    menu.appendChild(downloadBtn);
    menu.appendChild(deleteBtn);
    card.appendChild(menu);

    menuBtn.onclick = () => {
        menu.style.display = menu.style.display === "flex" ? "none" : "flex";
    };

    grid.appendChild(card);
}

// Cargar todos los archivos al iniciar sesión
function loadFiles() {
    const account = JSON.parse(localStorage.getItem("user_" + currentUser));
    const grid = document.getElementById("fileGrid");
    grid.innerHTML = "";
    account.files.forEach(f => renderFile(f));
}

// Exportar archivos
function exportData() {
    const account = JSON.parse(localStorage.getItem("user_" + currentUser));
    const blob = new Blob([JSON.stringify(account)], {type: "application/json"});
    const link = document.createElement("a");
    link.href = URL.createObjectURL(blob);
    link.download = currentUser + "_cloud.json";
    link.click();
}

// Importar archivos
document.getElementById("importFile").onchange = async (e) => {
    const file = e.target.files[0];
    if(!file) return;
    const text = await file.text();
    localStorage.setItem("user_" + currentUser, text);
    loadFiles();
    alert("Archivos importados correctamente");
};

// Logout
function logout() {
    currentUser = null;
    document.getElementById("cloudBox").classList.add("hidden");
    document.getElementById("authBox").classList.remove("hidden");
    document.getElementById("username").value = "";
    document.getElementById("password").value = "";
}
</script>

</body>
</html>
