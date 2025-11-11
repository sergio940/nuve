<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MiniCloud</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background: linear-gradient(135deg, #0a192f, #1e3c72);
        color: #fff;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100vh;
        margin: 0;
    }
    .container {
        background: rgba(255,255,255,0.1);
        padding: 20px;
        border-radius: 12px;
        box-shadow: 0 0 15px rgba(0,0,0,0.3);
        width: 320px;
        text-align: center;
    }
    input, button, textarea {
        width: 90%;
        margin: 8px 0;
        padding: 8px;
        border: none;
        border-radius: 6px;
    }
    input, textarea {
        background: #eee;
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
    .hidden { display: none; }
    textarea {
        height: 100px;
        resize: none;
    }
</style>
</head>
<body>

<div class="container" id="authBox">
    <h2>☁️ MiniCloud</h2>
    <input type="text" id="username" placeholder="Usuario">
    <input type="password" id="password" placeholder="Contraseña">
    <button onclick="register()">Registrarse</button>
    <button onclick="login()">Iniciar Sesión</button>
</div>

<div class="container hidden" id="cloudBox">
    <h2>☁️ Bienvenido, <span id="userDisplay"></span></h2>
    <textarea id="fileContent" placeholder="Escribe aquí tu texto o datos..."></textarea>
    <button onclick="saveFile()">💾 Guardar en la nube</button>
    <button onclick="downloadFile()">⬇️ Descargar</button>
    <button onclick="logout()">🔒 Cerrar sesión</button>
</div>

<script>
    const authBox = document.getElementById("authBox");
    const cloudBox = document.getElementById("cloudBox");
    const fileContent = document.getElementById("fileContent");
    const userDisplay = document.getElementById("userDisplay");

    function register() {
        const user = username.value.trim();
        const pass = password.value.trim();
        if(!user || !pass) return alert("Completa usuario y contraseña.");
        if(localStorage.getItem("user_" + user)) return alert("Usuario ya existe.");
        localStorage.setItem("user_" + user, JSON.stringify({ password: pass, data: "" }));
        alert("Usuario registrado correctamente.");
    }

    function login() {
        const user = username.value.trim();
        const pass = password.value.trim();
        const data = localStorage.getItem("user_" + user);
        if(!data) return alert("Usuario no encontrado.");
        const account = JSON.parse(data);
        if(account.password !== pass) return alert("Contraseña incorrecta.");
        userDisplay.textContent = user;
        fileContent.value = account.data || "";
        authBox.classList.add("hidden");
        cloudBox.classList.remove("hidden");
        localStorage.setItem("activeUser", user);
    }

    function saveFile() {
        const user = localStorage.getItem("activeUser");
        if(!user) return;
        const data = JSON.parse(localStorage.getItem("user_" + user));
        data.data = fileContent.value;
        localStorage.setItem("user_" + user, JSON.stringify(data));
        alert("Datos guardados correctamente ☁️");
    }

    function downloadFile() {
        const blob = new Blob([fileContent.value], { type: "text/plain" });
        const link = document.createElement("a");
        link.href = URL.createObjectURL(blob);
        link.download = "minicloud_" + userDisplay.textContent + ".txt";
        link.click();
    }

    function logout() {
        cloudBox.classList.add("hidden");
        authBox.classList.remove("hidden");
        localStorage.removeItem("activeUser");
        username.value = "";
        password.value = "";
    }

    // Autologin si hay sesión
    window.onload = () => {
        const active = localStorage.getItem("activeUser");
        if(active) {
            const acc = JSON.parse(localStorage.getItem("user_" + active));
            userDisplay.textContent = active;
            fileContent.value = acc.data || "";
            authBox.classList.add("hidden");
            cloudBox.classList.remove("hidden");
        }
    };
</script>

</body>
</html>
