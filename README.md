<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Sistema Informa - Gestão de Acessos</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
    <style>
        :root { --primary: #2563eb; --danger: #dc2626; --success: #10b981; --gray: #64748b; }
        body{ font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f0f2f5; padding:20px; min-height:100vh; display:flex; flex-direction:column; margin:0; }
        .container{ max-width:1100px; margin:20px auto; background:#fff; padding:25px; border-radius:12px; flex:1; box-shadow:0 10px 25px rgba(0,0,0,0.05); }
        input, select, button { width:100%; padding:10px; margin-bottom:10px; border:1px solid #ddd; border-radius:6px; box-sizing: border-box; font-size: 14px; }
        button { background: var(--primary); color:#fff; border:none; cursor:pointer; font-weight:bold; transition: 0.2s; }
        button:hover { opacity: 0.9; transform: translateY(-1px); }
        button.danger { background: var(--danger); }
        button.secondary { background: var(--gray); }
        .card { border:1px solid #eee; padding:15px; border-radius:8px; margin:10px 0; background:#fff; }
        .bloqueado { background: #fee2e2 !important; border: 1px solid #ef4444; }
        #login { max-width:400px; margin:100px auto; background:#fff; padding:40px; border-radius:12px; text-align:center; box-shadow:0 15px 35px rgba(0,0,0,0.1); }
        #adminGear { position:fixed; top:20px; right:20px; font-size:28px; cursor:pointer; display:none; z-index:100; background: #fff; border-radius: 50%; padding: 5px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        footer { text-align:center; padding:20px; color:#666; font-size:13px; }
        .btn-mini { width: auto; padding: 5px 10px; font-size: 11px; margin: 2px; }
    </style>
</head>
<body>

<div id="login">
    <h2>🔐 Sistema Informa</h2>
    <input id="loginUsuario" placeholder="Usuário">
    <input id="loginSenha" type="password" placeholder="Senha">
    <button id="btnLogin">Entrar</button>
    <p id="erro" style="color:var(--danger); font-size:13px"></p>
</div>

<div id="adminGear">⚙️</div>

<div class="container" id="sistema" style="display:none">
    <div style="display:flex; justify-content:space-between; align-items:center">
        <h1>Bem-vindo, <span id="nomeDisplay"></span></h1>
        <button id="btnLogout" class="secondary" style="width:auto">Sair</button>
    </div>
    <hr>
    <p>Você está logado no painel do Jornal Informa.</p>
    </div>

<div id="painelAdmin" class="container" style="display:none; border-top: 5px solid var(--primary)">
    <h2>⚙️ Gestão de Usuários (Acesso Admin)</h2>
    <div class="card">
        <h4>Criar Novo Acesso</h4>
        <input id="novoUsuario" placeholder="Nome de Usuário (Ex: João)">
        <input id="senhaUsuario" type="password" placeholder="Senha">
        <select id="nivelUsuario">
            <option value="user">Usuário Comum</option>
            <option value="admin">Administrador</option>
        </select>
        <select id="categoriaUsuario">
            <option value="">-- Categoria (Se for comum) --</option>
            <option value="Meio Ambiente">Meio Ambiente</option>
            <option value="Linguagens">Linguagens</option>
            <option value="Comunicações">Comunicações</option>
            <option value="Edição de Vídeo">Edição de Vídeo</option>
            <option value="Cultura">Cultura</option>
            <option value="Secretaria">Secretaria</option>
            <option value="Esportes">Esportes</option>
            <option value="Presidência">Presidência</option>
            <option value="Informações">Informações</option>
            <option value="Designer">Designer</option>
        </select>
        <button id="btnAddUsuario">Adicionar Usuário ao Sistema</button>
    </div>

    <h4>👥 Usuários Cadastrados</h4>
    <div id="listaUsuarios"></div>
</div>

<footer>© 2025 – Sistema Informa – Criado por <b>CLX</b></footer>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getFirestore, collection, addDoc, getDocs, updateDoc, deleteDoc, doc, getDoc } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

// Configuração do seu Firebase
const firebaseConfig = {
    apiKey: "AIzaSyCtJytArZciWTcAaVI--bY7mSiFVE-K6Zw",
    authDomain: "informa-a8d4d.firebaseapp.com",
    projectId: "informa-a8d4d",
    storageBucket: "informa-a8d4d.firebasestorage.app",
    messagingSenderId: "201808467376",
    appId: "1:201808467376:web:bb06f0fd7e57dfa747b275"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

let usuarios = [], usuarioLogado = null;

// Atalhos para elementos
const el = (id) => document.getElementById(id);

window.addEventListener('DOMContentLoaded', async () => {
    el('btnLogin').onclick = login;
    el('btnLogout').onclick = logout;
    el('btnAddUsuario').onclick = addUsuario;
    el('adminGear').onclick = () => {
        el('painelAdmin').style.display = el('painelAdmin').style.display === 'none' ? 'block' : 'none';
    };

    // Tentar recuperar sessão salva
    const sessao = localStorage.getItem('sessao_informa');
    if(sessao) {
        const uS = JSON.parse(sessao);
        validarSessao(uS);
    }
});

async function login(){
    const userDigitado = el('loginUsuario').value;
    const senhaDigitada = el('loginSenha').value;

    const s = await getDocs(collection(db, 'usuarios'));
    usuarios = s.docs.map(d => ({id: d.id, ...d.data()}));

    const u = usuarios.find(u => u.usuario === userDigitado && u.senha === senhaDigitada);

    if(!u) return el('erro').innerText = "Usuário ou senha incorretos.";
    if(!u.ativo) return el('erro').innerText = "Este acesso foi bloqueado.";

    localStorage.setItem('sessao_informa', JSON.stringify(u));
    entrarNoSistema(u);
}

async function validarSessao(uS) {
    // Busca no banco para ver se a senha ainda é válida ou se não foi bloqueado
    const s = await getDocs(collection(db, 'usuarios'));
    usuarios = s.docs.map(d => ({id: d.id, ...d.data()}));
    const uV = usuarios.find(x => x.usuario === uS.usuario && x.senha === uS.senha);
    
    if(uV && uV.ativo) entrarNoSistema(uV);
    else logout();
}

function entrarNoSistema(u) {
    usuarioLogado = u;
    el('login').style.display = 'none';
    el('sistema').style.display = 'block';
    el('nomeDisplay').innerText = u.usuario;

    if(u.nivel === 'admin'){
        el('adminGear').style.display = 'block';
        carregarUsuarios();
    }
}

function logout() { 
    localStorage.removeItem('sessao_informa'); 
    location.reload(); 
}

// GESTÃO DE USUÁRIOS (SÓ ADMIN)
async function carregarUsuarios(){
    const s = await getDocs(collection(db, 'usuarios'));
    usuarios = s.docs.map(d => ({id: d.id, ...d.data()}));
    
    el('listaUsuarios').innerHTML = "";
    usuarios.forEach(u => {
        el('listaUsuarios').innerHTML += `
            <div class="card ${u.ativo ? '' : 'bloqueado'}">
                <b>${u.usuario}</b> - Nível: ${u.nivel} ${u.categoria ? `(${u.categoria})` : ''}
                <div style="margin-top:10px">
                    <button class="btn-mini success" onclick="resetarSenha('${u.id}', '${u.usuario}')">Mudar Senha</button>
                    <button class="btn-mini secondary" onclick="toggleUser('${u.id}', ${u.ativo})">${u.ativo ? 'Bloquear' : 'Ativar'}</button>
                    ${u.usuario !== 'CLX' ? `<button class="danger btn-mini" onclick="excluirUsuario('${u.id}')">Remover</button>` : ''}
                </div>
            </div>`;
    });
}

async function addUsuario(){
    const nome = el('novoUsuario').value;
    const senha = el('senhaUsuario').value;
    const nivel = el('nivelUsuario').value;
    const cat = el('categoriaUsuario').value;

    if(!nome || !senha) return alert("Preencha nome e senha!");

    await addDoc(collection(db, 'usuarios'), {
        usuario: nome,
        senha: senha,
        nivel: nivel,
        categoria: cat,
        ativo: true
    });

    el('novoUsuario').value = "";
    el('senhaUsuario').value = "";
    alert("Usuário criado!");
    carregarUsuarios();
}

// Funções globais para os botões da lista
window.resetarSenha = async (id, nome) => {
    const nS = prompt(`Nova senha para ${nome}:`);
    if(nS) { 
        await updateDoc(doc(db, 'usuarios', id), { senha: nS }); 
        alert("Senha alterada!");
        carregarUsuarios(); 
    }
};

window.toggleUser = async (id, stat) => { 
    await updateDoc(doc(db, 'usuarios', id), { ativo: !stat }); 
    carregarUsuarios(); 
};

window.excluirUsuario = async (id) => { 
    if(confirm("Tem certeza que deseja excluir esse acesso permanentemente?")) { 
        await deleteDoc(doc(db, 'usuarios', id)); 
        carregarUsuarios(); 
    } 
};
</script>
</body>
</html>
