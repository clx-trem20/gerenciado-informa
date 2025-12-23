<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Sistema Informa - Enterprise Total</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .drawer { transform: translateX(100%); transition: transform 0.3s ease-in-out; }
        .drawer.open { transform: translateX(0); }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 100; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        .modal.open { display: flex; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
    </style>
</head>
<body class="bg-slate-100 min-h-screen">

    <div id="login-screen" class="flex items-center justify-center h-screen bg-slate-200 px-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl w-full max-w-md border-t-8 border-blue-600">
            <h1 class="text-3xl font-black text-center text-blue-600 mb-2 italic">INFORMA</h1>
            <p class="text-gray-400 text-center text-[10px] font-bold mb-8 tracking-widest uppercase">Gerenciamento Interno</p>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-4 mb-4 border rounded-2xl outline-blue-600 bg-gray-50">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-4 mb-6 border rounded-2xl outline-blue-600 bg-gray-50">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold hover:bg-blue-700 shadow-lg">ENTRAR</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-bold"></p>
        </div>
    </div>

    <div id="sistema" class="hidden">
        <nav class="bg-white border-b px-6 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-black text-blue-700">INFORMA</h2>
            <div class="flex items-center space-x-4">
                <button id="adminGear" class="hidden text-2xl hover:bg-gray-100 p-2 rounded-full transition">⚙️</button>
                <button id="btnLogout" class="text-gray-500 font-bold text-sm hover:text-red-500">Sair</button>
            </div>
        </nav>

        <main class="p-8 max-w-6xl mx-auto">
            <div class="flex flex-col md:flex-row justify-between items-center mb-8 gap-4">
                <h1 class="text-2xl font-black text-gray-800 uppercase tracking-tighter">Equipe do Jornal</h1>
                <button onclick="abrirDrawer()" class="bg-blue-600 text-white px-8 py-3 rounded-2xl font-bold shadow-lg hover:bg-blue-700 transition">+ Novo Membro</button>
            </div>

            <div class="bg-white rounded-3xl shadow-sm border overflow-hidden">
                <table class="min-w-full text-left">
                    <thead class="bg-gray-50 text-gray-400 text-[10px] font-bold uppercase tracking-widest">
                        <tr>
                            <th class="px-6 py-4">Membro</th>
                            <th class="px-6 py-4">Categoria</th>
                            <th class="px-6 py-4">E-mail / Ano</th>
                            <th class="px-6 py-4">Status</th>
                            <th class="px-6 py-4 text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="listaMembros" class="divide-y divide-gray-100 text-sm"></tbody>
                </table>
            </div>
        </main>
    </div>

    <div id="drawerOverlay" onclick="fecharDrawer()" class="fixed inset-0 bg-black/40 hidden z-40"></div>
    <div id="drawer" class="drawer fixed top-0 right-0 h-full w-full max-w-md bg-white shadow-2xl z-50 p-8 flex flex-col">
        <h2 id="drawerTitulo" class="text-2xl font-black mb-8 text-gray-800 uppercase">Membro da Equipe</h2>
        <input type="hidden" id="editId">
        
        <div class="space-y-4 overflow-y-auto pr-2 flex-1">
            <input id="mNome" placeholder="Nome Completo" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
            <select id="mCategoria" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
                <option value="">Selecione a Categoria</option>
                <option value="Meio Ambiente">🌿 Meio Ambiente</option>
                <option value="Linguagens">📚 Linguagens</option>
                <option value="Comunicações">📢 Comunicações</option>
                <option value="Edição de Vídeo">🎬 Edição de Vídeo</option>
                <option value="Cultura">🎭 Cultura</option>
                <option value="Secretaria">📝 Secretaria</option>
                <option value="Esportes">⚽ Esportes</option>
                <option value="Presidência">👑 Presidência</option>
                <option value="Informações">ℹ️ Informações</option>
                <option value="Designer">🎨 Designer</option>
            </select>
            <input id="mEmail" type="email" placeholder="E-mail" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
            <input id="mAno" type="number" placeholder="Ano de Entrada" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
        </div>
        <button onclick="salvarMembroBanco()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black shadow-xl mt-6">SALVAR NO FIREBASE</button>
    </div>

    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-4xl rounded-3xl p-8 max-h-[85vh] overflow-hidden flex flex-col relative">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-2xl font-bold">&times;</button>
            <div class="flex space-x-6 border-b mb-6">
                <button onclick="switchTab('usuarios')" class="pb-2 font-bold text-blue-600 border-b-2 border-blue-600">Acessos</button>
                <button onclick="switchTab('logs')" class="pb-2 font-bold text-gray-400">Logs do Sistema</button>
            </div>
            
            <div id="content-usuarios" class="tab-content active overflow-y-auto">
                <div class="bg-blue-50 p-6 rounded-2xl mb-6 grid grid-cols-1 md:grid-cols-4 gap-2">
                    <input id="accUser" placeholder="Login" class="p-3 rounded-xl border">
                    <input id="accPass" type="password" placeholder="Senha" class="p-3 rounded-xl border">
                    <select id="accNivel" class="p-3 rounded-xl border"><option value="user">User</option><option value="admin">Admin</option></select>
                    <button onclick="criarLoginSistema()" class="bg-blue-600 text-white rounded-xl font-bold">CRIAR</button>
                </div>
                <div id="listaAcessos" class="space-y-2"></div>
            </div>

            <div id="content-logs" class="tab-content overflow-y-auto">
                <div id="listaLogs" class="space-y-2 text-[11px] font-mono"></div>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, getDocs, updateDoc, deleteDoc, doc, query, where, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

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

        // REGISTRO DE LOGS
        async function registrarLog(acao) {
            await addDoc(collection(db, "logs"), { usuario: currentUser?.usuario || "Sistema", acao, data: serverTimestamp() });
        }

        // LOGIN
        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value;
            const p = document.getElementById('loginPass').value;
            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snap = await getDocs(q);

            if (snap.empty) { document.getElementById('erro').innerText = "Credenciais incorretas"; return; }
            const data = snap.docs[0].data();
            if (!data.ativo) { document.getElementById('erro').innerText = "ACESSO BLOQUEADO!"; return; }

            currentUser = data;
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            if (data.nivel === 'admin') document.getElementById('adminGear').classList.remove('hidden');
            registrarLog("Login efetuado");
            carregarMembros();
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        // GESTÃO DE MEMBROS
        window.abrirDrawer = (id = null) => {
            document.getElementById('editId').value = id || "";
            document.getElementById('drawer').classList.add('open');
            document.getElementById('drawerOverlay').classList.remove('hidden');
            if(!id) { document.getElementById('mNome').value = ""; document.getElementById('mEmail').value = ""; }
        };

        window.fecharDrawer = () => {
            document.getElementById('drawer').classList.remove('open');
            document.getElementById('drawerOverlay').classList.add('hidden');
        };

        window.salvarMembroBanco = async () => {
            const id = document.getElementById('editId').value;
            const m = {
                nome: document.getElementById('mNome').value,
                categoria: document.getElementById('mCategoria').value,
                email: document.getElementById('mEmail').value,
                ano: document.getElementById('mAno').value,
                status: "Ativo"
            };
            if(id) {
                await updateDoc(doc(db, "membros", id), m);
                registrarLog(`Editou membro: ${m.nome}`);
            } else {
                await addDoc(collection(db, "membros"), m);
                registrarLog(`Cadastrou membro: ${m.nome}`);
            }
            fecharDrawer(); carregarMembros();
        };

        window.carregarMembros = async () => {
            const snap = await getDocs(query(collection(db, "membros"), orderBy("nome", "asc")));
            const lista = document.getElementById('listaMembros');
            lista.innerHTML = "";
            snap.forEach(d => {
                const m = d.data();
                const id = d.id;
                lista.innerHTML += `
                <tr class="hover:bg-gray-50 border-b ${m.status !== 'Ativo' ? 'bg-red-50 text-gray-400' : ''}">
                    <td class="px-6 py-4 font-bold text-gray-800">${m.nome}</td>
                    <td class="px-6 py-4 italic text-blue-600">${m.categoria}</td>
                    <td class="px-6 py-4">${m.email}<br><span class="text-[10px] text-gray-400">Ingresso: ${m.ano}</span></td>
                    <td class="px-6 py-4"><span class="px-2 py-1 rounded-full text-[10px] font-bold ${m.status === 'Ativo' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}">${m.status}</span></td>
                    <td class="px-6 py-4 text-center space-x-3 text-lg">
                        <button onclick="abrirDrawer('${id}')">✏️</button>
                        <button onclick="toggleMembro('${id}', '${m.status}', '${m.nome}')">🚫</button>
                        <button onclick="excluirMembro('${id}', '${m.nome}')">🗑️</button>
                        <button onclick="enviarEmail('${m.email}', '${m.nome}')">✉️</button>
                    </td>
                </tr>`;
            });
        };

        window.toggleMembro = async (id, status, nome) => {
            const novo = status === "Ativo" ? "Inativo" : "Ativo";
            await updateDoc(doc(db, "membros", id), { status: novo });
            registrarLog(`${novo === 'Inativo' ? 'Inativou' : 'Reativou'} membro: ${nome}`);
            carregarMembros();
        };

        window.excluirMembro = async (id, nome) => {
            if(confirm("Excluir definitivamente?")) {
                await deleteDoc(doc(db, "membros", id));
                registrarLog(`Excluiu membro: ${nome}`);
                carregarMembros();
            }
        };

        window.enviarEmail = (email, nome) => {
            window.location.href = `mailto:${email}?subject=Agradecimento&body=Olá ${nome}, obrigado por sua dedicação ao Jornal Informa!`;
        };

        // ADMIN E ACESSOS
        document.getElementById('adminGear').onclick = () => {
            document.getElementById('modalAdmin').classList.add('open');
            carregarLogins();
        };
        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');

        window.criarLoginSistema = async () => {
            const u = document.getElementById('accUser').value;
            const p = document.getElementById('accPass').value;
            const n = document.getElementById('accNivel').value;
            await addDoc(collection(db, "usuarios"), { usuario: u, senha: p, nivel: n, ativo: true });
            registrarLog(`Criou acesso para: ${u}`);
            carregarLogins();
        };

        window.carregarLogins = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos');
            lista.innerHTML = "";
            snap.forEach(d => {
                const u = d.data();
                lista.innerHTML += `
                <div class="flex justify-between p-3 border rounded-xl bg-white text-xs">
                    <span><b>${u.usuario}</b> (${u.nivel}) - Senha: ${u.senha}</span>
                    <div class="space-x-1">
                        <button onclick="resetSenha('${d.id}')" class="bg-yellow-400 p-1 rounded">SENHA</button>
                        <button onclick="bloquear('${d.id}', ${u.ativo})" class="bg-gray-100 p-1 rounded">${u.ativo ? 'BLOQUEAR' : 'ATIVAR'}</button>
                        ${u.usuario !== 'CLX' ? `<button onclick="excluirAcc('${d.id}')" class="bg-red-500 text-white p-1 rounded italic">X</button>` : ''}
                    </div>
                </div>`;
            });
        };

        window.resetSenha = async (id) => {
            const n = prompt("Nova senha:");
            if(n) await updateDoc(doc(db, "usuarios", id), { senha: n }); carregarLogins();
        };

        window.bloquear = async (id, status) => {
            await updateDoc(doc(db, "usuarios", id), { ativo: !status }); carregarLogins();
        };

        window.excluirAcc = async (id) => {
            if(confirm("Deletar login?")) await deleteDoc(doc(db, "usuarios", id)); carregarLogins();
        };

        window.switchTab = (tab) => {
            document.getElementById('content-usuarios').classList.toggle('active', tab === 'usuarios');
            document.getElementById('content-logs').classList.toggle('active', tab === 'logs');
            if(tab === 'logs') carregarLogs();
        };

        window.carregarLogs = async () => {
            const snap = await getDocs(query(collection(db, "logs"), orderBy("data", "desc")));
            const lista = document.getElementById('listaLogs');
            lista.innerHTML = "";
            snap.forEach(d => {
                const l = d.data();
                lista.innerHTML += `<div>[${l.data?.toDate().toLocaleString()}] <b>${l.usuario}:</b> ${l.acao}</div>`;
            });
        };
    </script>
</body>
</html>
