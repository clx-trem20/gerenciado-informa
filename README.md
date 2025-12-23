<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Sistema Informa - Gestão Total</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .drawer { transform: translateX(100%); transition: transform 0.3s ease-in-out; }
        .drawer.open { transform: translateX(0); }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 100; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        .modal.open { display: flex; }
    </style>
</head>
<body class="bg-gray-100 min-h-screen">

    <div id="login-screen" class="flex items-center justify-center h-screen bg-slate-200">
        <div class="bg-white p-10 rounded-3xl shadow-2xl w-full max-w-md border-t-8 border-blue-600">
            <h1 class="text-3xl font-black text-center text-blue-600 mb-8 italic">INFORMA ADMIN</h1>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-4 mb-4 border rounded-2xl outline-blue-600">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-4 mb-6 border rounded-2xl outline-blue-600">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold hover:bg-blue-700 transition shadow-lg">ENTRAR</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-bold"></p>
        </div>
    </div>

    <div id="sistema" class="hidden">
        <nav class="bg-white shadow-sm border-b px-8 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-bold text-blue-700 uppercase">Jornal Informa</h2>
            <div class="flex items-center space-x-4">
                <button id="adminGear" class="hidden text-2xl p-2 hover:bg-gray-100 rounded-full transition">⚙️</button>
                <button id="btnLogout" class="text-gray-500 font-bold hover:text-red-500">Sair</button>
            </div>
        </nav>

        <main class="p-8 max-w-7xl mx-auto">
            <div class="flex justify-between items-center mb-8">
                <h1 class="text-2xl font-black text-gray-800">EQUIPE DO JORNAL</h1>
                <button onclick="abrirDrawer()" class="bg-blue-600 text-white px-6 py-3 rounded-xl font-bold shadow-lg hover:bg-blue-700">+ Novo Membro</button>
            </div>

            <div class="bg-white rounded-2xl shadow-sm border overflow-hidden">
                <table class="min-w-full text-left">
                    <thead class="bg-gray-50 text-gray-400 text-xs font-bold uppercase">
                        <tr>
                            <th class="px-6 py-4">Membro</th>
                            <th class="px-6 py-4">Categoria</th>
                            <th class="px-6 py-4">E-mail</th>
                            <th class="px-6 py-4">Ano</th>
                            <th class="px-6 py-4">Status</th>
                            <th class="px-6 py-4 text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="listaMembros" class="divide-y divide-gray-100 text-sm font-medium"></tbody>
                </table>
            </div>
        </main>
    </div>

    <div id="drawerOverlay" onclick="fecharDrawer()" class="fixed inset-0 bg-black/40 hidden z-40"></div>
    <div id="drawer" class="drawer fixed top-0 right-0 h-full w-full max-w-md bg-white shadow-2xl z-50 p-8">
        <h2 id="drawerTitulo" class="text-2xl font-black mb-8 text-gray-800 uppercase">Cadastrar Membro</h2>
        <input type="hidden" id="editId">
        <div class="space-y-4">
            <input id="mNome" placeholder="Nome Completo" class="w-full p-4 border rounded-xl bg-gray-50 outline-blue-600">
            <select id="mCategoria" class="w-full p-4 border rounded-xl bg-gray-50 outline-blue-600">
                <option value="">Selecione a Categoria</option>
                <option value="Meio Ambiente">Meio Ambiente</option>
                <option value="Linguagens">Linguagens</option>
                <option value="Comunicações">Comunicações</option>
                <option value="Secretaria">Secretaria</option>
                <option value="Edição de Vídeo">Edição de Vídeo</option>
                <option value="Designer">Designer</option>
            </select>
            <input id="mEmail" type="email" placeholder="E-mail" class="w-full p-4 border rounded-xl bg-gray-50 outline-blue-600">
            <input id="mAno" type="number" placeholder="Ano de Entrada" class="w-full p-4 border rounded-xl bg-gray-50 outline-blue-600">
            <button onclick="salvarMembroBanco()" class="w-full bg-blue-600 text-white py-4 rounded-xl font-bold shadow-xl">SALVAR NO SISTEMA</button>
        </div>
    </div>

    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-4xl rounded-3xl p-8 max-h-[85vh] overflow-y-auto relative">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-2xl font-bold">&times;</button>
            <h2 class="text-2xl font-black mb-6">⚙️ Painel de Acessos</h2>
            
            <div class="bg-blue-50 p-6 rounded-2xl mb-8">
                <h4 class="text-xs font-bold text-blue-700 uppercase mb-4">Novo Usuário do Sistema</h4>
                <div class="grid grid-cols-2 md:grid-cols-4 gap-2">
                    <input id="accUser" placeholder="Login" class="p-2 border rounded-lg">
                    <input id="accPass" type="password" placeholder="Senha" class="p-2 border rounded-lg">
                    <select id="accNivel" class="p-2 border rounded-lg">
                        <option value="user">Comum</option>
                        <option value="admin">Admin</option>
                    </select>
                    <button onclick="criarLoginSistema()" class="bg-blue-600 text-white rounded-lg font-bold">CRIAR</button>
                </div>
            </div>
            <div id="listaAcessos" class="space-y-2"></div>
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
        let userLogado = null;

        // --- LOGIN ---
        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value;
            const p = document.getElementById('loginPass').value;
            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snap = await getDocs(q);

            if (snap.empty) { document.getElementById('erro').innerText = "Usuário ou senha incorretos!"; return; }
            const data = snap.docs[0].data();
            if (!data.ativo) { document.getElementById('erro').innerText = "ACESSO BLOQUEADO!"; return; }

            userLogado = data;
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            if (data.nivel === 'admin') document.getElementById('adminGear').classList.remove('hidden');
            
            carregarMembros();
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        // --- GESTÃO DE MEMBROS ---
        window.abrirDrawer = (id = null) => {
            document.getElementById('drawerTitulo').innerText = id ? "Editar Membro" : "Cadastrar Membro";
            document.getElementById('editId').value = id || "";
            if(!id) {
                document.getElementById('mNome').value = "";
                document.getElementById('mEmail').value = "";
                document.getElementById('mAno').value = "";
            }
            document.getElementById('drawer').classList.add('open');
            document.getElementById('drawerOverlay').classList.remove('hidden');
        };

        window.fecharDrawer = () => {
            document.getElementById('drawer').classList.remove('open');
            document.getElementById('drawerOverlay').classList.add('hidden');
        };

        window.salvarMembroBanco = async () => {
            const id = document.getElementById('editId').value;
            const dados = {
                nome: document.getElementById('mNome').value,
                categoria: document.getElementById('mCategoria').value,
                email: document.getElementById('mEmail').value,
                ano: document.getElementById('mAno').value,
                status: "Ativo"
            };

            if (id) {
                await updateDoc(doc(db, "membros", id), dados);
            } else {
                await addDoc(collection(db, "membros"), dados);
            }
            fecharDrawer();
            carregarMembros();
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
                    <td class="px-6 py-4">${m.categoria}</td>
                    <td class="px-6 py-4 font-normal">${m.email}</td>
                    <td class="px-6 py-4">${m.ano}</td>
                    <td class="px-6 py-4"><span class="px-2 py-1 rounded-full text-[10px] font-bold ${m.status === 'Ativo' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}">${m.status.toUpperCase()}</span></td>
                    <td class="px-6 py-4 text-center space-x-3">
                        <button onclick="abrirDrawer('${id}')" class="text-blue-500 hover:scale-125 transition">✏️</button>
                        <button onclick="alternarStatusMembro('${id}', '${m.status}')" title="Ativar/Inativar">🚫</button>
                        <button onclick="excluirMembro('${id}')" class="text-red-500">🗑️</button>
                        <button onclick="enviarEmail('${m.email}', '${m.nome}')">✉️</button>
                    </td>
                </tr>`;
                // Para carregar os dados na edição
                if(document.getElementById('editId').value === id) {
                    document.getElementById('mNome').value = m.nome;
                    document.getElementById('mEmail').value = m.email;
                    document.getElementById('mAno').value = m.ano;
                    document.getElementById('mCategoria').value = m.categoria;
                }
            });
        };

        window.alternarStatusMembro = async (id, status) => {
            const novo = status === "Ativo" ? "Inativo" : "Ativo";
            await updateDoc(doc(db, "membros", id), { status: novo });
            carregarMembros();
        };

        window.excluirMembro = async (id) => {
            if(confirm("Deseja excluir este membro permanentemente?")) {
                await deleteDoc(doc(db, "membros", id));
                carregarMembros();
            }
        };

        window.enviarEmail = (email, nome) => {
            const assunto = encodeURIComponent("Agradecimento - Jornal Informa");
            const corpo = encodeURIComponent(`Olá, ${nome}!\n\nGostaríamos de agradecer por todo o seu empenho e dedicação ao Jornal Informa.\n\nDesejamos muito sucesso em seus próximos passos!\n\nAtenciosamente,\nEquipe Jornal Informa.`);
            window.location.href = `mailto:${email}?subject=${assunto}&body=${corpo}`;
        };

        // --- ADMIN (ENGRENAGEM) ---
        document.getElementById('adminGear').onclick = () => {
            document.getElementById('modalAdmin').classList.add('open');
            carregarLoginsSistema();
        };
        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');

        window.criarLoginSistema = async () => {
            const u = document.getElementById('accUser').value;
            const p = document.getElementById('accPass').value;
            const n = document.getElementById('accNivel').value;
            await addDoc(collection(db, "usuarios"), { usuario: u, senha: p, nivel: n, ativo: true });
            carregarLoginsSistema();
        };

        window.carregarLoginsSistema = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos');
            lista.innerHTML = "";
            snap.forEach(d => {
                const u = d.data();
                lista.innerHTML += `
                <div class="flex justify-between items-center p-3 border rounded-xl bg-white">
                    <span><b>${u.usuario}</b> (${u.nivel})</span>
                    <div class="flex space-x-2">
                        <button onclick="resetarSenhaSistema('${d.id}')" class="bg-yellow-400 px-2 py-1 rounded text-[10px] font-bold">SENHA</button>
                        <button onclick="toggleBloqueioSistema('${d.id}', ${u.ativo})" class="bg-gray-100 px-2 py-1 rounded text-[10px] font-bold">${u.ativo ? 'BLOQUEAR' : 'ATIVAR'}</button>
                        ${u.usuario !== 'CLX' ? `<button onclick="excluirAcessoSistema('${d.id}')" class="bg-red-500 text-white px-2 py-1 rounded text-[10px] font-bold">X</button>` : ''}
                    </div>
                </div>`;
            });
        };

        window.resetarSenhaSistema = async (id) => {
            const n = prompt("Nova senha:");
            if(n) await updateDoc(doc(db, "usuarios", id), { senha: n });
            carregarLoginsSistema();
        };

        window.toggleBloqueioSistema = async (id, status) => {
            await updateDoc(doc(db, "usuarios", id), { ativo: !status });
            carregarLoginsSistema();
        };

        window.excluirAcessoSistema = async (id) => {
            if(confirm("Excluir login?")) await deleteDoc(doc(db, "usuarios", id));
            carregarLoginsSistema();
        };
    </script>
</body>
</html>
