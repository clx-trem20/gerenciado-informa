<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Sistema Informa - Admin Pro</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 100; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        .modal.open { display: flex; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .btn-tab.active { border-bottom: 3px solid #2563eb; color: #2563eb; }
    </style>
</head>
<body class="bg-slate-50 min-h-screen">

    <div id="login-screen" class="flex items-center justify-center h-screen px-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl w-full max-w-md border border-gray-100">
            <h1 class="text-3xl font-black text-center text-blue-700 mb-2 italic">INFORMA</h1>
            <p class="text-gray-400 text-center text-xs font-bold mb-8 uppercase tracking-widest">Painel Administrativo</p>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-4 mb-4 border rounded-2xl outline-blue-600 bg-gray-50">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-4 mb-6 border rounded-2xl outline-blue-600 bg-gray-50">
            <button id="btnLogin" class="w-full bg-blue-700 text-white py-4 rounded-2xl font-bold hover:bg-blue-800 transition-all shadow-lg active:scale-95">ACESSAR</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-bold"></p>
        </div>
    </div>

    <div id="sistema" class="hidden">
        <nav class="bg-white border-b px-6 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-black text-blue-700">INFORMA</h2>
            <div class="flex items-center space-x-4">
                <button id="adminGear" class="hidden text-2xl hover:bg-gray-100 p-2 rounded-full transition">⚙️</button>
                <button id="btnLogout" class="text-gray-500 font-bold text-sm hover:text-red-500 transition">Sair</button>
            </div>
        </nav>
        <main class="p-8 max-w-5xl mx-auto">
            <div class="bg-white p-12 rounded-3xl shadow-sm border border-gray-100 text-center">
                <h1 class="text-3xl font-bold text-gray-800">Olá, <span id="userName" class="text-blue-700"></span>!</h1>
                <p class="text-gray-500 mt-2">O sistema está pronto para uso.</p>
            </div>
        </main>
    </div>

    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-4xl rounded-3xl shadow-2xl p-8 max-h-[85vh] overflow-hidden flex flex-col relative">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-gray-400 text-3xl hover:text-red-500">&times;</button>
            
            <div class="flex space-x-6 border-b mb-6">
                <button onclick="switchTab('usuarios')" id="tabUser" class="btn-tab pb-2 font-bold text-gray-500 active">Usuários</button>
                <button onclick="switchTab('logs')" id="tabLogs" class="btn-tab pb-2 font-bold text-gray-500">Log de Atividades</button>
            </div>

            <div id="content-usuarios" class="tab-content active overflow-y-auto pr-2">
                <div class="bg-blue-50 p-6 rounded-2xl mb-6">
                    <h4 class="text-blue-800 font-bold text-xs uppercase mb-4">Novo Acesso</h4>
                    <div class="grid grid-cols-1 md:grid-cols-4 gap-2">
                        <input id="novoLogin" placeholder="Login" class="p-3 rounded-xl border outline-blue-500">
                        <input id="novaSenha" type="password" placeholder="Senha" class="p-3 rounded-xl border outline-blue-500">
                        <select id="novoNivel" class="p-3 rounded-xl border outline-blue-500 bg-white">
                            <option value="user">Membro</option>
                            <option value="admin">Admin</option>
                        </select>
                        <button onclick="criarAcesso()" class="bg-blue-700 text-white rounded-xl font-bold">CRIAR</button>
                    </div>
                </div>
                <div id="listaAcessos" class="space-y-3"></div>
            </div>

            <div id="content-logs" class="tab-content overflow-y-auto pr-2">
                <div class="flex justify-between items-center mb-4">
                    <h4 class="text-gray-700 font-bold">Histórico Recente</h4>
                    <button onclick="limparLogs()" class="text-xs text-red-500 font-bold hover:underline">Limpar Histórico</button>
                </div>
                <div id="listaLogs" class="space-y-2 text-xs font-mono"></div>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, getDocs, updateDoc, deleteDoc, doc, query, where, orderBy, limit, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAZsk5iPq2plw37JcgNFEhbKXkOGjtMYZU",
            authDomain: "informa-gerenciamento.firebaseapp.com",
            projectId: "informa-gerenciamento",
            storageBucket: "informa-gerenciamento.firebasestorage.app",
            messagingSenderId: "916183491768",
            appId: "1:916183491768:web:1ee681afba0189fc16c04d"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        let currentUser = null;

        // --- FUNÇÃO DE LOG ---
        async function registrarLog(acao) {
            await addDoc(collection(db, "logs"), {
                usuario: currentUser ? currentUser.usuario : "Sistema",
                acao: acao,
                data: serverTimestamp()
            });
        }

        // --- LOGIN ---
        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value;
            const p = document.getElementById('loginPass').value;
            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snap = await getDocs(q);

            if (snap.empty) { document.getElementById('erro').innerText = "Erro no Login!"; return; }
            const data = snap.docs[0].data();
            if (!data.ativo) { document.getElementById('erro').innerText = "CONTA BLOQUEADA!"; return; }

            currentUser = data;
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            document.getElementById('userName').innerText = data.usuario;
            if (data.nivel === 'admin') document.getElementById('adminGear').classList.remove('hidden');
            
            registrarLog("Login efetuado");
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        // --- GESTÃO ADMIN ---
        window.switchTab = (tab) => {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.btn-tab').forEach(b => b.classList.remove('active'));
            document.getElementById(`content-${tab}`).classList.add('active');
            document.getElementById(`tab${tab.charAt(0).toUpperCase() + tab.slice(1)}`).classList.add('active');
            if(tab === 'logs') carregarLogs();
        };

        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');
        document.getElementById('adminGear').onclick = () => {
            document.getElementById('modalAdmin').classList.add('open');
            carregarAcessos();
        };

        window.criarAcesso = async () => {
            const u = document.getElementById('novoLogin').value;
            const s = document.getElementById('novaSenha').value;
            const n = document.getElementById('novoNivel').value;
            if(!u || !s) return;
            await addDoc(collection(db, "usuarios"), { usuario: u, senha: s, nivel: n, ativo: true });
            registrarLog(`Criou novo usuário: ${u}`);
            carregarAcessos();
            document.getElementById('novoLogin').value = ""; document.getElementById('novaSenha').value = "";
        };

        window.carregarAcessos = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos');
            lista.innerHTML = "";
            snap.forEach(d => {
                const u = d.data();
                lista.innerHTML += `
                <div class="flex items-center justify-between p-4 border rounded-2xl bg-white shadow-sm ${!u.ativo ? 'border-red-200 bg-red-50' : ''}">
                    <div><b class="text-gray-700">${u.usuario}</b> <span class="text-[10px] bg-gray-200 px-2 rounded">${u.nivel}</span></div>
                    <div class="flex space-x-2">
                        <button onclick="redefinirSenha('${d.id}', '${u.usuario}')" class="text-[10px] font-bold p-2 bg-yellow-400 rounded-lg">SENHA</button>
                        <button onclick="toggleBloqueio('${d.id}', ${u.ativo}, '${u.usuario}')" class="text-[10px] font-bold p-2 ${u.ativo ? 'bg-orange-400' : 'bg-green-500 text-white'} rounded-lg">${u.ativo ? 'BLOQUEAR' : 'ATIVAR'}</button>
                        ${u.usuario !== 'CLX' ? `<button onclick="excluirUser('${d.id}', '${u.usuario}')" class="text-[10px] font-bold p-2 bg-red-600 text-white rounded-lg">X</button>` : ''}
                    </div>
                </div>`;
            });
        };

        window.redefinirSenha = async (id, nome) => {
            const ns = prompt(`Nova senha para ${nome}:`);
            if(ns) { 
                await updateDoc(doc(db, "usuarios", id), { senha: ns }); 
                registrarLog(`Alterou senha de: ${nome}`);
                carregarAcessos();
            }
        };

        window.toggleBloqueio = async (id, status, nome) => {
            await updateDoc(doc(db, "usuarios", id), { ativo: !status });
            registrarLog(`${status ? 'Bloqueou' : 'Ativou'} usuário: ${nome}`);
            carregarAcessos();
        };

        window.excluirUser = async (id, nome) => {
            if(confirm("Excluir?")) { 
                await deleteDoc(doc(db, "usuarios", id)); 
                registrarLog(`Excluiu usuário: ${nome}`);
                carregarAcessos();
            }
        };

        window.carregarLogs = async () => {
            const q = query(collection(db, "logs"), orderBy("data", "desc"), limit(50));
            const snap = await getDocs(q);
            const lista = document.getElementById('listaLogs');
            lista.innerHTML = "";
            snap.forEach(d => {
                const log = d.data();
                const data = log.data?.toDate().toLocaleString('pt-BR') || "Agora";
                lista.innerHTML += `<div class="p-2 border-b text-gray-600"><span class="text-blue-600 font-bold">[${data}]</span> <b>${log.usuario}:</b> ${log.acao}</div>`;
            });
        };

        window.limparLogs = async () => {
            if(confirm("Limpar todo o histórico?")) {
                const snap = await getDocs(collection(db, "logs"));
                snap.forEach(async d => await deleteDoc(doc(db, "logs", d.id)));
                carregarLogs();
            }
        };
    </script>
</body>
</html>
