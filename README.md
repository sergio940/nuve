<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MiniCloud Archivos</title>
<style>
body {
    font-family: Arial, sans-serif;
    background: #1e1e2f;
    color: #fff;
    display: flex;
    justify-content: center;
    padding: 2rem;
}
.container {
    background: rgba(255,255,255,0.1);
    padding: 20px;
    border-radius: 12px;
    width: 360px;
    text-align: center;
}
input, button { width: 90%; margin: 0.5rem 0; padding: 8px; border-radius: 6px; border: none; }
button { background: #1db954; color: #fff; font-weight: bold; cursor: pointer; }
button:hover { background: #1ed760; }
ul { list-style: none; padding: 0; max-height: 200px; overflow-y: auto; }
li { background: #2e2e4f; margin: 0.3rem 0; padding: 0.5rem; border-radius: 6px; display: flex; justify-content: space-between; align-items: center; }
.hidden { display: none; }
a { color: #1db954; text-decoration: none; margin-right: 5px; }
</style>
</head>
<body>

<div class="container" id="authBox">
    <h2>☁️ MiniCloud Archivos</h2>
    <input type="text" id="username" placeholder="Usuario">
    <input type="password" id="password" placeholder="Contraseña">
    <button onclick="register()">Registrarse</button>
    <button onclick="login()">Iniciar Sesión</button>
</div>

<div class="container hidden" id="cloudBox">
    <h3>Bienvenido, <span id="userDisplay"></span></h3>
    <input type="file" id="fileInput" multiple><br>
    <ul id="fileList"></ul>
    <button onclick="exportData()">Exportar mis archivos</button>
    <input type="file" id="importFile" style="display:none">
    <button onclick="document.getElementById('importFile').click()">Importar archivos</button>
    <button onclick="logout()">Cerrar sesión</button>
</div>

<script>
let currentUser = null;

// Autenticación
function register() {
    const user = document.getElementById("username").value.trim();
    const pass = document.getElementById("password").value.trim();
    if(!user || !pass) return alert("Rellena usuario y contraseña.");
    if(localStorage.getItem("user_" + user)) return alert("Usuario ya existe.");
    localStorage.setItem("user_" + user, JSON.stringify({password: pass, files: []}));
    alert("Usuario registrado.");
}

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
            account.files.push({name: f.name, data: reader.result});
            localStorage.setItem("user_" + currentUser, JSON.stringify(account));
            loadFiles();
        };
        reader.readAsDataURL(f);
    }
};

// Listar archivos
function loadFiles() {
    const account = JSON.parse(localStorage.getItem("user_" + currentUser));
    const list = document.getElementById("fileList");
    list.innerHTML = "";
    if(account.files.length === 0) list.innerHTML = "<li>No hay archivos</li>";
    account.files.forEach((f, i) => {
        const li = document.createElement("li");
        const link = document.createElement("a");
        link.href = f.data;
        link.download = f.name;
        link.textContent = f.name;
        const del = document.createElement("button");
        del.textContent = "Eliminar";
        del.onclick = () => {
            account.files.splice(i,1);
            localStorage.setItem("user_" + currentUser, JSON.stringify(account));
            loadFiles();
        };
        li.appendChild(link);
        li.appendChild(del);
        list.appendChild(li);
    });
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
