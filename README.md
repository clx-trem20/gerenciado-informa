<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Sistema Informa - Enterprise</title>
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

    <div id="login-screen" class="flex items-center justify-center h-screen px-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl w-full max-w-md border-t-8 border-blue-600">
            <h1 class="text-3xl font-black text-center text-blue-600 mb-2 italic uppercase">Informa</h1>
            <p class="text-gray-400 text-center text-[10px] font-bold mb-8 tracking-widest uppercase">Gerenciamento Interno</p>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-4 mb-4 border rounded-2xl outline-blue-600 bg-gray-50">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-4 mb-6 border rounded-2xl outline-blue-600 bg-gray-50">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold hover:bg-blue-700 shadow-lg">ENTRAR</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-bold"></p>
        </div>
    </div>

    <div id="sistema" class="hidden">
        <nav class="bg-white border-b px-6 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-black text-blue-700 uppercase">Informa</h2>
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
        <button onclick="salvarMembroBanco()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black shadow-xl mt-6">SALVAR NO SISTEMA</button>
    </div>

    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-4xl rounded-3xl p-8 max-h-[85vh] overflow-hidden flex flex-col relative">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-2xl font-bold">&times;</button>
            <div class="flex space-x-6 border-b mb-6">
                <button onclick="switchTab('usuarios')" id="btnTabUser" class="pb-2 font-bold text-blue-600 border-b-2 border-blue-600">Acessos</button>
                <button onclick="switchTab('logs')" id="btnTabLogs" class="pb-2 font-bold text-gray-400">Logs</button>
            </div>
            
            <div id="content-usuarios" class="tab-content active overflow-y-auto pr-2">
                <div class="bg-blue-50 p-6 rounded-2xl mb-6 grid grid-cols-1 md:grid-cols-4 gap-2 italic">
                    <input id="accUser" placeholder="Login" class="p-3 rounded-xl border">
                    <input id="accPass" type="password" placeholder="Senha" class="p-3 rounded-xl border">
                    <select id="accNivel" class="p-3 rounded-xl border"><option value="user">User</option><option value="admin">Admin</option></select>
                    <button onclick="criarLoginSistema()" class="bg-blue-600 text-white rounded-xl font-bold">CRIAR</button>
                </div>
                <div id="listaAcessos" class="space-y-3"></div>
            </div>

            <div id="content-logs" class="tab-content overflow-y-auto pr-2">
                <div id="listaLogs" class="space-y-2 text-[11px] font-mono bg-gray-50 p-4 rounded-xl"></div>
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

        async function registrarLog(acao) {
            await addDoc(collection(db, "logs"), { usuario: currentUser?.usuario || "Sistema", acao, data: serverTimestamp() });
        }

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

        window.abrirDrawer = (id = null) => {
            document.getElementById('editId').value = id || "";
            document.getElementById('drawer').classList.add('open');
            document.getElementById('drawerOverlay').classList.remove('hidden');
            if(!id) {
                document.getElementById('mNome').value = "";
                document.getElementById('mEmail').value = "";
                document.getElementById('mAno').value = "";
                document.getElementById('mCategoria').value = "";
                document.getElementById('drawerTitulo').innerText = "Cadastrar Membro";
            } else {
                document.getElementById('drawerTitulo').innerText = "Editar Membro";
            }
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
            if(!m.nome || !m.categoria) return alert("Nome e Categoria são obrigatórios!");

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
                    <td class="px-6 py-4"><span class="text-blue-600 font-semibold text-xs bg-blue-50 px-2 py-1 rounded-lg">${m.categoria}</span></td>
                    <td class="px-6 py-4 text-xs font-medium text-gray-500">${m.email}<br>Ingresso: ${m.ano}</td>
                    <td class="px-6 py-4 text-center space-x-4">
                        <button onclick="abrirDrawerComDados('${id}')" title="Editar">✏️</button>
                        <button onclick="toggleMembro('${id}', '${m.status}', '${m.nome}')" title="Bloquear/Ativar">🚫</button>
                        <button onclick="excluirMembro('${id}', '${m.nome}')" title="Excluir">🗑️</button>
                        <button onclick="enviarEmail('${m.email}', '${m.nome}')" title="Email">✉️</button>
                    </td>
                </tr>`;
            });
        };

        window.abrirDrawerComDados = async (id) => {
            const snap = await getDocs(collection(db, "membros"));
            snap.forEach(d => {
                if(d.id === id) {
                    const m = d.data();
                    abrirDrawer(id);
                    document.getElementById('mNome').value = m.nome;
                    document.getElementById('mEmail').value = m.email;
                    document.getElementById('mAno').value = m.ano;
                    document.getElementById('mCategoria').value = m.categoria;
                }
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

        // ACESSOS SISTEMA
        window.criarLoginSistema = async () => {
            const u = document.getElementById('accUser').value;
            const p = document.getElementById('accPass').value;
            const n = document.getElementById('accNivel').value;
            if(!u || !p) return;
            await addDoc(collection(db, "usuarios"), { usuario: u, senha: p, nivel: n, ativo: true });
            registrarLog(`Criou acesso: ${u}`);
            carregarLogins();
        };

        window.carregarLogins = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos');
            lista.innerHTML = "";
            snap.forEach(d => {
                const u = d.data();
                lista.innerHTML += `
                <div class="flex justify-between items-center p-4 border rounded-2xl bg-white">
                    <div class="flex flex-col">
                        <span class="font-bold text-gray-800 text-sm">${u.usuario}</span>
                        <span class="text-[10px] text-blue-500 font-bold uppercase tracking-widest">${u.nivel}</span>
                    </div>
                    <div class="space-x-1">
                        <button onclick="resetSenha('${d.id}')" class="bg-blue-600 text-white px-3 py-1.5 rounded-lg text-[10px] font-bold">NOVA SENHA</button>
                        <button onclick="bloquearAcc('${d.id}', ${u.ativo})" class="bg-gray-100 px-3 py-1.5 rounded-lg text-[10px] font-bold">${u.ativo ? 'BLOQUEAR' : 'ATIVAR'}</button>
                        ${u.usuario !== 'CLX' ? `<button onclick="excluirAcc('${d.id}')" class="bg-red-50 text-red-600 px-3 py-1.5 rounded-lg text-[10px] font-bold italic">EXCLUIR</button>` : ''}
                    </div>
                </div>`;
            });
        };

        window.resetSenha = async (id) => {
            const n = prompt("Digite a nova senha:");
            if(n) {
                await updateDoc(doc(db, "usuarios", id), { senha: n });
                alert("Senha alterada!");
            }
        };

        window.bloquearAcc = async (id, status) => {
            await updateDoc(doc(db, "usuarios", id), { ativo: !status });
            carregarLogins();
        };

        window.excluirAcc = async (id) => {
            if(confirm("Deletar este acesso?")) {
                await deleteDoc(doc(db, "usuarios", id));
                carregarLogins();
            }
        };

        window.switchTab = (tab) => {
            document.getElementById('content-usuarios').classList.toggle('active', tab === 'usuarios');
            document.getElementById('content-logs').classList.toggle('active', tab === 'logs');
            document.getElementById('btnTabUser').classList.toggle('text-blue-600', tab === 'usuarios');
            document.getElementById('btnTabUser').classList.toggle('border-blue-600', tab === 'usuarios');
            document.getElementById('btnTabLogs').classList.toggle('text-blue-600', tab === 'logs');
            document.getElementById('btnTabLogs').classList.toggle('border-blue-600', tab === 'logs');
            if(tab === 'logs') carregarLogs();
        };

        window.carregarLogs = async () => {
            const snap = await getDocs(query(collection(db, "logs"), orderBy("data", "desc")));
            const lista = document.getElementById('listaLogs');
            lista.innerHTML = "";
            snap.forEach(d => {
                const l = d.data();
                const data = l.data?.toDate().toLocaleString() || "...";
                lista.innerHTML += `<div class="p-1 border-b"><span class="text-blue-600 font-bold">[${data}]</span> <b>${l.usuario}:</b> ${l.acao}</div>`;
            });
        };

        document.getElementById('adminGear').onclick = () => {
            document.getElementById('modalAdmin').classList.add('open');
            carregarLogins();
        };
        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');
    </script>
</body>
</html>
