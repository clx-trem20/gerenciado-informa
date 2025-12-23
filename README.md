<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Sistema Informa - Administração</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.6); z-index: 100; align-items: center; justify-content: center; backdrop-filter: blur(4px); }
        .modal.open { display: flex; }
    </style>
</head>
<body class="bg-slate-100 min-h-screen">

    <div id="login-screen" class="flex items-center justify-center h-screen">
        <div class="bg-white p-10 rounded-2xl shadow-2xl w-full max-w-md border-t-8 border-blue-600">
            <h1 class="text-3xl font-black text-center text-blue-600 mb-6 uppercase tracking-tighter">Jornal Informa</h1>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-3 mb-4 border rounded-xl outline-blue-500 bg-gray-50">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-3 mb-6 border rounded-xl outline-blue-500 bg-gray-50">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-3 rounded-xl font-bold hover:bg-blue-700 transition shadow-lg">ENTRAR NO SISTEMA</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-semibold"></p>
        </div>
    </div>

    <div id="sistema" class="hidden">
        <nav class="bg-white shadow-sm border-b px-8 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-bold text-blue-700 uppercase">Jornal Informa</h2>
            <div class="flex items-center space-x-4">
                <button id="adminGear" class="hidden text-2xl hover:rotate-90 transition duration-500">⚙️</button>
                <button id="btnLogout" class="bg-gray-200 text-gray-700 px-4 py-1.5 rounded-lg font-bold hover:bg-gray-300 transition">Sair</button>
            </div>
        </nav>

        <main class="p-8 max-w-6xl mx-auto">
            <div class="bg-white p-8 rounded-2xl shadow-sm border text-center">
                <h1 class="text-2xl font-bold text-gray-800">Bem-vindo, <span id="userName" class="text-blue-600"></span>!</h1>
                <p class="text-gray-500 mt-2">Você está autenticado no painel de controle.</p>
            </div>
        </main>
    </div>

    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-3xl rounded-2xl shadow-2xl p-8 max-h-[90vh] overflow-y-auto relative">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-gray-400 text-3xl hover:text-red-500">&times;</button>
            
            <h2 class="text-2xl font-black text-gray-800 mb-6 border-b pb-4">⚙️ Controle de Acessos</h2>
            
            <div class="bg-blue-50 p-6 rounded-xl mb-8">
                <h4 class="font-bold text-blue-800 mb-4 uppercase text-xs tracking-widest">Criar Novo Login</h4>
                <div class="grid grid-cols-1 md:grid-cols-4 gap-3">
                    <input id="novoLogin" placeholder="Nome de Usuário" class="p-2.5 border rounded-lg outline-blue-400">
                    <input id="novaSenha" type="password" placeholder="Senha" class="p-2.5 border rounded-lg outline-blue-400">
                    <select id="novoNivel" class="p-2.5 border rounded-lg outline-blue-400 bg-white">
                        <option value="user">Membro (Normal)</option>
                        <option value="admin">Administrador</option>
                    </select>
                    <button onclick="criarAcesso()" class="bg-blue-600 text-white rounded-lg font-bold hover:bg-blue-700 transition">CRIAR</button>
                </div>
            </div>

            <h4 class="font-bold text-gray-700 mb-4 uppercase text-xs tracking-widest">Gerenciar Contas</h4>
            <div id="listaAcessos" class="space-y-3">
                </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, getDocs, updateDoc, deleteDoc, doc, query, where } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

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

        // --- LOGIN ---
        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value;
            const p = document.getElementById('loginPass').value;
            const erro = document.getElementById('erro');

            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snap = await getDocs(q);

            if (snap.empty) {
                erro.innerText = "Usuário ou senha incorretos!";
                return;
            }

            const user = snap.docs[0].data();
            if (!user.ativo) {
                erro.innerText = "ACESSO BLOQUEADO!";
                return;
            }

            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            document.getElementById('userName').innerText = user.usuario;

            if (user.nivel === 'admin') {
                document.getElementById('adminGear').classList.remove('hidden');
            }
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        // --- PAINEL ADMIN (ENGRENAGEM) ---
        document.getElementById('adminGear').onclick = () => {
            document.getElementById('modalAdmin').classList.add('open');
            carregarAcessos();
        };
        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');

        window.criarAcesso = async () => {
            const u = document.getElementById('novoLogin').value;
            const s = document.getElementById('novaSenha').value;
            const n = document.getElementById('novoNivel').value;
            if(!u || !s) return alert("Preencha todos os campos!");

            await addDoc(collection(db, "usuarios"), { usuario: u, senha: s, nivel: n, ativo: true });
            alert("Acesso criado com sucesso!");
            carregarAcessos();
            document.getElementById('novoLogin').value = ""; document.getElementById('novaSenha').value = "";
        };

        window.carregarAcessos = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos');
            lista.innerHTML = "";

            snap.forEach(d => {
                const u = d.data();
                const id = d.id;
                lista.innerHTML += `
                    <div class="flex flex-col md:flex-row md:items-center justify-between p-4 border rounded-xl bg-white shadow-sm ${!u.ativo ? 'border-red-300 bg-red-50' : ''}">
                        <div class="mb-3 md:mb-0">
                            <span class="font-bold text-gray-800 text-lg">${u.usuario}</span>
                            <span class="ml-2 px-2 py-0.5 bg-gray-100 text-[10px] font-bold uppercase rounded text-gray-500 border">${u.nivel}</span>
                            <div class="text-[11px] text-gray-400">Senha: ${u.senha}</div>
                        </div>
                        <div class="flex flex-wrap gap-2">
                            <button onclick="mudarSenha('${id}', '${u.usuario}')" class="px-3 py-1 bg-yellow-500 text-white text-xs font-bold rounded-lg hover:bg-yellow-600 transition">REDEFINIR SENHA</button>
                            <button onclick="toggleBloqueio('${id}', ${u.ativo})" class="px-3 py-1 ${u.ativo ? 'bg-orange-500' : 'bg-green-600'} text-white text-xs font-bold rounded-lg transition">
                                ${u.ativo ? 'BLOQUEAR' : 'DESBLOQUEAR'}
                            </button>
                            ${u.usuario !== 'CLX' ? `<button onclick="excluirUser('${id}')" class="px-3 py-1 bg-red-600 text-white text-xs font-bold rounded-lg hover:bg-red-700 transition">EXCLUIR</button>` : ''}
                        </div>
                    </div>`;
            });
        };

        window.mudarSenha = async (id, nome) => {
            const nova = prompt(`Digite a nova senha para ${nome}:`);
            if(nova) {
                await updateDoc(doc(db, "usuarios", id), { senha: nova });
                carregarAcessos();
            }
        };

        window.toggleBloqueio = async (id, status) => {
            await updateDoc(doc(db, "usuarios", id), { ativo: !status });
            carregarAcessos();
        };

        window.excluirUser = async (id) => {
            if(confirm("Deseja deletar permanentemente este acesso?")) {
                await deleteDoc(doc(db, "usuarios", id));
                carregarAcessos();
            }
        };
    </script>
</body>
</html>
