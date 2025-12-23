<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Sistema Informa - Enterprise v6.0</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root { --primary: #2563eb; --danger: #dc2626; --success: #10b981; --gray: #64748b; }
        body { font-family: 'Segoe UI', sans-serif; background: #f0f2f5; }
        .drawer { transform: translateX(100%); transition: transform 0.3s ease-in-out; }
        .drawer.open { transform: translateX(0); }
        .bloqueado { opacity: 0.6; background: #fee2e2 !important; }
    </style>
</head>
<body class="min-h-screen">

    <div id="login-screen" class="flex items-center justify-center h-screen">
        <div class="bg-white p-10 rounded-2xl shadow-2xl w-full max-w-md border-t-8 border-blue-600">
            <h1 class="text-3xl font-black text-center text-blue-600 mb-2">JORNAL INFORMA</h1>
            <p class="text-gray-400 text-center mb-8 uppercase text-xs font-bold tracking-widest">Acesso Restrito</p>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-3 mb-4 border rounded-xl outline-blue-500">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-3 mb-6 border rounded-xl outline-blue-500">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-3 rounded-xl font-bold hover:bg-blue-700 transition shadow-lg">ENTRAR NO SISTEMA</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-medium"></p>
        </div>
    </div>

    <div id="adminGear" class="fixed top-6 right-6 text-3xl cursor-pointer hidden z-50 bg-white p-2 rounded-full shadow-lg">⚙️</div>

    <div id="sistema" class="hidden p-6 max-w-7xl mx-auto">
        <header class="flex justify-between items-center mb-10 bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
            <div>
                <h2 class="text-2xl font-bold text-gray-800">Dashboard Equipe</h2>
                <p class="text-gray-500">Olá, <span id="userName" class="text-blue-600 font-bold"></span></p>
            </div>
            <div class="flex space-x-3">
                <button onclick="exportarExcel()" class="bg-green-600 text-white px-5 py-2 rounded-lg font-bold hover:bg-green-700 transition flex items-center shadow-md">
                    <span class="mr-2">📊</span> Excel
                </button>
                <button onclick="toggleDrawer()" class="bg-blue-600 text-white px-5 py-2 rounded-lg font-bold hover:bg-blue-700 transition shadow-md">
                    + Adicionar Membro
                </button>
                <button id="btnLogout" class="bg-gray-200 text-gray-700 px-5 py-2 rounded-lg font-bold hover:bg-gray-300 transition">Sair</button>
            </div>
        </header>

        <div class="bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden">
            <table class="min-w-full">
                <thead class="bg-gray-50 text-gray-500 text-xs font-bold uppercase">
                    <tr>
                        <th class="px-6 py-4 text-left tracking-wider">Nome Completo</th>
                        <th class="px-6 py-4 text-left tracking-wider">E-mail</th>
                        <th class="px-6 py-4 text-left tracking-wider">Ingresso</th>
                        <th class="px-6 py-4 text-left tracking-wider">Status</th>
                        <th class="px-6 py-4 text-center tracking-wider">Ações</th>
                    </tr>
                </thead>
                <tbody id="listaMembros" class="divide-y divide-gray-100 text-sm text-gray-700 font-medium"></tbody>
            </table>
        </div>
    </div>

    <div id="drawer" class="drawer fixed top-0 right-0 h-full w-full max-w-sm bg-white shadow-2xl z-50 p-8 border-l">
        <div class="flex justify-between items-center mb-8">
            <h2 class="text-2xl font-bold text-gray-800">Novo Membro</h2>
            <button onclick="toggleDrawer()" class="text-gray-400 hover:text-red-500 text-3xl">&times;</button>
        </div>
        <div class="space-y-4">
            <input type="text" id="membroNome" placeholder="Nome Completo" class="w-full p-3 border rounded-lg bg-gray-50 outline-blue-500">
            <input type="email" id="membroEmail" placeholder="E-mail" class="w-full p-3 border rounded-lg bg-gray-50 outline-blue-500">
            <input type="date" id="membroData" class="w-full p-3 border rounded-lg bg-gray-50 outline-blue-500">
            <button onclick="salvarMembro()" class="w-full bg-blue-600 text-white py-3 rounded-xl font-bold hover:bg-blue-700 shadow-lg">CADASTRAR</button>
        </div>
    </div>

    <div id="painelAdmin" class="hidden fixed inset-0 bg-black/50 backdrop-blur-sm z-[60] flex items-center justify-center p-4">
        <div class="bg-white w-full max-w-2xl rounded-2xl shadow-2xl p-8 max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold text-gray-800">⚙️ Gestão de Usuários do Sistema</h2>
                <button onclick="fecharAdmin()" class="text-gray-400 text-2xl font-bold">&times;</button>
            </div>
            
            <div class="bg-blue-50 p-6 rounded-xl mb-8">
                <h4 class="font-bold text-blue-800 mb-4">Criar Novo Acesso (Login)</h4>
                <div class="grid grid-cols-2 gap-3">
                    <input id="novoLogin" placeholder="Usuário" class="p-2 border rounded">
                    <input id="novaSenha" type="password" placeholder="Senha" class="p-2 border rounded">
                    <select id="novoNivel" class="p-2 border rounded">
                        <option value="user">Usuário Comum</option>
                        <option value="admin">Administrador</option>
                    </select>
                    <button onclick="criarAcesso()" class="bg-blue-600 text-white rounded font-bold">CRIAR</button>
                </div>
            </div>

            <h4 class="font-bold text-gray-700 mb-4 tracking-tight">Usuários com Acesso:</h4>
            <div id="listaAcessos" class="space-y-2"></div>
        </div>
    </div>

    <footer>© 2025 – Sistema Informa – Criado por <b>CLX</b></footer>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, getDocs, updateDoc, deleteDoc, doc, query, where, orderBy } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

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

        let userLogado = null;

        // --- SISTEMA DE LOGIN ---
        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value;
            const p = document.getElementById('loginPass').value;
            const erro = document.getElementById('erro');

            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snapshot = await getDocs(q);

            if (snapshot.empty) {
                erro.innerText = "Credenciais inválidas!";
                return;
            }

            const data = snapshot.docs[0].data();
            if (!data.ativo) {
                erro.innerText = "Acesso bloqueado pelo administrador.";
                return;
            }

            userLogado = data;
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            document.getElementById('userName').innerText = data.usuario;

            if (data.nivel === 'admin') {
                document.getElementById('adminGear').classList.remove('hidden');
            }
            carregarMembros();
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        // --- GESTÃO DE MEMBROS ---
        window.toggleDrawer = () => document.getElementById('drawer').classList.toggle('open');

        window.salvarMembro = async () => {
            const nome = document.getElementById('membroNome').value;
            const email = document.getElementById('membroEmail').value;
            const data = document.getElementById('membroData').value;

            if(!nome || !email || !data) return alert("Preencha tudo!");

            await addDoc(collection(db, "membros"), { nome, email, data, status: "Ativo" });
            alert("Membro cadastrado!");
            toggleDrawer();
            carregarMembros();
        };

        window.carregarMembros = async () => {
            const q = query(collection(db, "membros"), orderBy("nome", "asc"));
            const snapshot = await getDocs(q);
            const lista = document.getElementById('listaMembros');
            lista.innerHTML = "";

            snapshot.forEach(docSnap => {
                const m = docSnap.data();
                const id = docSnap.id;
                const dataF = m.data.split('-').reverse().join('/');
                const isAtivo = m.status === 'Ativo';

                lista.innerHTML += `
                    <tr class="border-b transition hover:bg-gray-50 ${!isAtivo ? 'bg-red-50 text-gray-400' : ''}">
                        <td class="px-6 py-4 font-bold">${m.nome}</td>
                        <td class="px-6 py-4 font-normal text-gray-500">${m.email}</td>
                        <td class="px-6 py-4">${dataF}</td>
                        <td class="px-6 py-4 text-xs">
                            <span class="px-2 py-1 rounded-full ${isAtivo ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}">${m.status}</span>
                        </td>
                        <td class="px-6 py-4 text-center space-x-2">
                            <button onclick="mudarStatusMembro('${id}', '${m.status}')" class="text-lg">🚫</button>
                            <button onclick="enviarEmailDespedida('${m.email}', '${m.nome}')" class="text-lg">✉️</button>
                        </td>
                    </tr>`;
            });
        };

        window.mudarStatusMembro = async (id, statusAtual) => {
            const novo = statusAtual === 'Ativo' ? 'Inativo' : 'Ativo';
            await updateDoc(doc(db, "membros", id), { status: novo });
            carregarMembros();
        };

        window.enviarEmailDespedida = (email, nome) => {
            const assunto = encodeURIComponent("Um agradecimento do Jornal Informa");
            const corpo = encodeURIComponent(`Olá, ${nome}!\n\nGostaríamos de agradecer imensamente por todo o seu esforço e dedicação durante o tempo em que colaborou com o Informa. Sua marca ficou registrada em nossa equipe!\n\nDesejamos muito sucesso em sua jornada.\n\nCom gratidão,\nEquipe Jornal Informa.`);
            window.location.href = `mailto:${email}?subject=${assunto}&body=${corpo}`;
        };

        // --- GESTÃO DE ACESSOS (ADMIN) ---
        document.getElementById('adminGear').onclick = () => {
            document.getElementById('painelAdmin').classList.remove('hidden');
            carregarAcessos();
        };
        window.fecharAdmin = () => document.getElementById('painelAdmin').classList.add('hidden');

        window.criarAcesso = async () => {
            const u = document.getElementById('novoLogin').value;
            const s = document.getElementById('novaSenha').value;
            const n = document.getElementById('novoNivel').value;
            if(!u || !s) return alert("Preencha login e senha!");

            await addDoc(collection(db, "usuarios"), { usuario: u, senha: s, nivel: n, ativo: true });
            alert("Acesso criado!");
            carregarAcessos();
        };

        window.carregarAcessos = async () => {
            const snapshot = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos');
            lista.innerHTML = "";

            snapshot.forEach(d => {
                const u = d.data();
                lista.innerHTML += `
                    <div class="flex justify-between items-center p-3 border rounded-lg ${!u.ativo ? 'bg-red-50' : ''}">
                        <span><b>${u.usuario}</b> (${u.nivel})</span>
                        <div class="space-x-2">
                            <button onclick="toggleAcesso('${d.id}', ${u.ativo})" class="text-xs bg-gray-200 p-1 rounded font-bold">${u.ativo ? 'BLOQUEAR' : 'ATIVAR'}</button>
                            ${u.usuario !== 'CLX' ? `<button onclick="removerAcesso('${d.id}')" class="text-xs bg-red-100 text-red-600 p-1 rounded font-bold">EXCLUIR</button>` : ''}
                        </div>
                    </div>`;
            });
        };

        window.toggleAcesso = async (id, status) => {
            await updateDoc(doc(db, "usuarios", id), { ativo: !status });
            carregarAcessos();
        };

        window.removerAcesso = async (id) => {
            if(confirm("Remover permanentemente?")) {
                await deleteDoc(doc(db, "usuarios", id));
                carregarAcessos();
            }
        };

        // --- EXCEL ---
        window.exportarExcel = async () => {
            const snapshot = await getDocs(collection(db, "membros"));
            const ativos = [], inativos = [];

            snapshot.forEach(d => {
                const m = d.data();
                const row = { "Nome": m.nome, "E-mail": m.email, "Ingresso": m.data.split('-').reverse().join('/') };
                if(m.status === 'Ativo') ativos.push(row);
                else inativos.push(row);
            });

            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(ativos), "Ativos");
            XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(inativos), "Inativos");
            XLSX.writeFile(wb, "Relatorio_Informa_Equipe.xlsx");
        };
    </script>
</body>
</html>
