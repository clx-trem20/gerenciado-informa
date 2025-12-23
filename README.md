<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Informa - Gestão de Permissões</title>
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
            <p class="text-gray-400 text-center text-[10px] font-bold mb-8 tracking-widest uppercase">Controle de Acessos</p>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-4 mb-4 border rounded-2xl outline-blue-600 bg-gray-50 font-bold">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-4 mb-6 border rounded-2xl outline-blue-600 bg-gray-50 font-bold">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold hover:bg-blue-700 shadow-lg transition-all">ENTRAR NO SISTEMA</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-bold"></p>
        </div>
    </div>

    <div id="sistema" class="hidden">
        <nav class="bg-white border-b px-6 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-black text-blue-700 uppercase tracking-tighter italic">Informa</h2>
            <div class="flex items-center space-x-4">
                <span id="labelPermissao" class="text-[10px] bg-slate-100 text-slate-600 px-3 py-1 rounded-full font-bold uppercase border"></span>
                <button id="adminGear" class="hidden text-2xl hover:bg-gray-100 p-2 rounded-full transition">⚙️</button>
                <button id="btnLogout" class="text-gray-500 font-bold text-sm hover:text-red-500 transition-colors">Sair</button>
            </div>
        </nav>

        <main class="p-8 max-w-6xl mx-auto">
            <div class="flex flex-col md:flex-row justify-between items-center mb-8 gap-4">
                <h1 class="text-2xl font-black text-gray-800 uppercase tracking-tighter">Equipe do Jornal</h1>
                <button onclick="abrirDrawerMembro()" class="bg-blue-600 text-white px-8 py-3 rounded-2xl font-bold shadow-lg hover:bg-blue-700 transition">+ Novo Membro</button>
            </div>

            <div class="bg-white rounded-3xl shadow-sm border overflow-hidden">
                <table class="min-w-full text-left">
                    <thead class="bg-gray-50 text-gray-400 text-[10px] font-bold uppercase tracking-widest">
                        <tr>
                            <th class="px-6 py-4">Membro</th>
                            <th class="px-6 py-4">Categoria</th>
                            <th class="px-6 py-4">Status</th>
                            <th class="px-6 py-4 text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="listaMembros" class="divide-y divide-gray-100 text-sm"></tbody>
                </table>
            </div>
        </main>
    </div>

    <div id="drawerOverlay" onclick="fecharDrawerMembro()" class="fixed inset-0 bg-black/40 hidden z-40"></div>
    <div id="drawerMembro" class="drawer fixed top-0 right-0 h-full w-full max-w-md bg-white shadow-2xl z-50 p-8 flex flex-col">
        <h2 class="text-2xl font-black mb-8 text-gray-800 uppercase tracking-tighter">Cadastro de Membro</h2>
        <input type="hidden" id="editMembroId">
        <div class="space-y-4 overflow-y-auto pr-2 flex-1">
            <input id="mNome" placeholder="Nome Completo" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 font-medium">
            <select id="mCategoria" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 font-medium">
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
            <select id="mStatus" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 font-bold">
                <option value="Ativo">✅ Ativo</option>
                <option value="Inativo">❌ Inativo</option>
            </select>
        </div>
        <button onclick="salvarMembroFirebase()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black shadow-xl mt-6 hover:bg-blue-700 transition-all">SALVAR MEMBRO</button>
    </div>

    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-4xl rounded-3xl p-8 max-h-[85vh] overflow-hidden flex flex-col relative shadow-2xl">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-2xl font-bold">&times;</button>
            <div class="flex space-x-6 border-b mb-6">
                <button onclick="switchTab('usuarios')" id="btnTabUser" class="pb-2 font-bold text-blue-600 border-b-2 border-blue-600 uppercase text-xs">Acessos</button>
                <button onclick="switchTab('logs')" id="btnTabLogs" class="pb-2 font-bold text-gray-400 uppercase text-xs">Registro de Logs</button>
            </div>
            
            <div id="content-usuarios" class="tab-content active overflow-y-auto pr-2">
                <div class="bg-blue-50 p-6 rounded-3xl mb-6">
                    <h3 class="text-xs font-black text-blue-800 mb-4 uppercase">Novo Usuário e Permissões</h3>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-3 mb-4">
                        <input id="accUser" placeholder="Login" class="p-3 rounded-xl border font-bold">
                        <input id="accPass" type="password" placeholder="Senha" class="p-3 rounded-xl border font-bold">
                        <select id="accNivel" class="p-3 rounded-xl border font-bold">
                            <option value="user">User (Restrito)</option>
                            <option value="admin">Admin (Total)</option>
                        </select>
                    </div>
                    <p class="text-[10px] font-bold text-gray-400 mb-2 uppercase">Categorias permitidas para este usuário:</p>
                    <div id="gridCategoriasPermitidas" class="grid grid-cols-2 md:grid-cols-5 gap-2 bg-white p-4 rounded-2xl border">
                        </div>
                    <button onclick="criarLoginSistema()" class="w-full bg-blue-600 text-white rounded-xl font-bold py-3 mt-4 shadow-lg">CRIAR ACESSO</button>
                </div>
                <div id="listaAcessos" class="space-y-3"></div>
            </div>

            <div id="content-logs" class="tab-content overflow-y-auto">
                <div id="listaLogs" class="space-y-1 text-[10px] font-mono bg-slate-900 text-green-400 p-4 rounded-xl"></div>
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
        
        let userLogado = null;
        const listaCats = ["Meio Ambiente", "Linguagens", "Comunicações", "Edição de Vídeo", "Cultura", "Secretaria", "Esportes", "Presidência", "Informações", "Designer"];

        // Injetar Categorias no Painel Admin
        const gridCats = document.getElementById('gridCategoriasPermitidas');
        listaCats.forEach(cat => {
            gridCats.innerHTML += `
                <label class="flex items-center space-x-2 text-[10px] font-bold text-gray-600 cursor-pointer">
                    <input type="checkbox" name="catPermissao" value="${cat}" class="w-4 h-4 rounded border-gray-300 text-blue-600">
                    <span>${cat}</span>
                </label>
            `;
        });

        async function registrarLog(acao) {
            await addDoc(collection(db, "logs"), { usuario: userLogado?.usuario || "Sistema", acao, data: serverTimestamp() });
        }

        // LOGIN COM VERIFICAÇÃO DE PERMISSÕES
        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value;
            const p = document.getElementById('loginPass').value;
            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snap = await getDocs(q);

            if (snap.empty) { document.getElementById('erro').innerText = "Credenciais inválidas"; return; }
            const userData = snap.docs[0].data();
            if (!userData.ativo) { document.getElementById('erro').innerText = "ACESSO DESATIVADO!"; return; }

            userLogado = userData;
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            
            if (userData.nivel === 'admin') {
                document.getElementById('adminGear').classList.remove('hidden');
                document.getElementById('labelPermissao').innerText = "Modo: Administrador Geral";
            } else {
                document.getElementById('labelPermissao').innerText = "Setores permitidos: " + (userData.categoriasPermitidas?.length || 0);
            }

            registrarLog("Iniciou sessão");
            carregarMembros();
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        // GESTÃO DE MEMBROS
        window.abrirDrawerMembro = (id = null) => {
            document.getElementById('editMembroId').value = id || "";
            document.getElementById('drawerMembro').classList.add('open');
            document.getElementById('drawerOverlay').classList.remove('hidden');
            if(!id) {
                document.getElementById('mNome').value = ""; document.getElementById('mEmail').value = "";
                document.getElementById('mAno').value = ""; document.getElementById('mCategoria').value = "";
            }
        };

        window.fecharDrawerMembro = () => {
            document.getElementById('drawerMembro').classList.remove('open');
            document.getElementById('drawerOverlay').classList.add('hidden');
        };

        window.salvarMembroFirebase = async () => {
            const id = document.getElementById('editMembroId').value;
            const m = {
                nome: document.getElementById('mNome').value,
                categoria: document.getElementById('mCategoria').value,
                email: document.getElementById('mEmail').value,
                ano: document.getElementById('mAno').value,
                status: document.getElementById('mStatus').value
            };
            if(id) {
                await updateDoc(doc(db, "membros", id), m);
                registrarLog(`Editou membro: ${m.nome}`);
            } else {
                await addDoc(collection(db, "membros"), m);
                registrarLog(`Adicionou membro: ${m.nome}`);
            }
            fecharDrawerMembro(); carregarMembros();
        };

        window.carregarMembros = async () => {
            const snap = await getDocs(query(collection(db, "membros"), orderBy("nome", "asc")));
            const lista = document.getElementById('listaMembros');
            lista.innerHTML = "";
            
            snap.forEach(d => {
                const m = d.data();
                
                // FILTRO DE PERMISSÃO:
                if(userLogado.nivel === 'user') {
                    const permitidas = userLogado.categoriasPermitidas || [];
                    if(!permitidas.includes(m.categoria)) return; // Se não tem permissão para essa cat, pula.
                }

                const inativo = m.status === 'Inativo';
                lista.innerHTML += `
                <tr class="hover:bg-gray-50 border-b ${inativo ? 'bg-red-50/50 opacity-60 italic' : ''}">
                    <td class="px-6 py-4 font-bold text-gray-800">${m.nome}</td>
                    <td class="px-6 py-4 text-[10px] font-bold text-blue-600 uppercase bg-blue-50/50 inline-block mt-3 rounded-lg px-2 border border-blue-100">${m.categoria}</td>
                    <td class="px-6 py-4">
                        <span class="text-[10px] font-black ${!inativo ? 'text-green-600' : 'text-red-500'}">● ${m.status.toUpperCase()}</span>
                    </td>
                    <td class="px-6 py-4 text-center space-x-3 text-lg">
                        <button onclick="abrirEdicaoMembro('${d.id}')">✏️</button>
                        <button onclick="excluirMembro('${d.id}', '${m.nome}')">🗑️</button>
                        <button onclick="enviarDespedida('${m.email}', '${m.nome}')">✉️</button>
                    </td>
                </tr>`;
            });
        };

        window.abrirEdicaoMembro = async (id) => {
            const snap = await getDocs(collection(db, "membros"));
            snap.forEach(d => {
                if(d.id === id) {
                    const m = d.data();
                    abrirDrawerMembro(id);
                    document.getElementById('mNome').value = m.nome;
                    document.getElementById('mEmail').value = m.email;
                    document.getElementById('mAno').value = m.ano;
                    document.getElementById('mCategoria').value = m.categoria;
                    document.getElementById('mStatus').value = m.status || "Ativo";
                }
            });
        };

        window.excluirMembro = async (id, nome) => {
            if(confirm(`Excluir ${nome}?`)) { await deleteDoc(doc(db, "membros", id)); registrarLog(`Excluiu membro: ${nome}`); carregarMembros(); }
        };

        window.enviarDespedida = (email, nome) => {
            window.location.href = `mailto:${email}?subject=Jornal Informa&body=Olá ${nome}, muito obrigado pela sua colaboração conosco!`;
        };

        // ADMINISTRAÇÃO DE ACESSOS
        window.criarLoginSistema = async () => {
            const u = document.getElementById('accUser').value;
            const p = document.getElementById('accPass').value;
            const n = document.getElementById('accNivel').value;
            const checks = document.querySelectorAll('input[name="catPermissao"]:checked');
            const selecionadas = Array.from(checks).map(c => c.value);

            if(!u || !p) return alert("Login e Senha obrigatórios!");

            await addDoc(collection(db, "usuarios"), { 
                usuario: u, senha: p, nivel: n, ativo: true, 
                categoriasPermitidas: selecionadas 
            });
            
            registrarLog(`Criou acesso para: ${u} com ${selecionadas.length} permissões.`);
            carregarLogins();
            alert("Usuário criado com sucesso!");
        };

        window.carregarLogins = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos');
            lista.innerHTML = "";
            snap.forEach(d => {
                const u = d.data();
                lista.innerHTML += `
                <div class="flex flex-col p-4 border rounded-2xl bg-white shadow-sm gap-2">
                    <div class="flex justify-between items-center">
                        <span class="font-black text-gray-800 uppercase tracking-tighter text-sm italic">${u.usuario} <small class="text-blue-500 font-bold ml-2">(${u.nivel})</small></span>
                        <div class="space-x-1">
                            <button onclick="bloquearAcesso('${d.id}', ${u.ativo})" class="bg-slate-100 px-3 py-1 rounded-lg text-[10px] font-bold uppercase">${u.ativo ? 'Bloquear' : 'Ativar'}</button>
                            ${u.usuario !== 'CLX' ? `<button onclick="removerAcesso('${d.id}')" class="bg-red-50 text-red-600 px-3 py-1 rounded-lg text-[10px] font-bold uppercase italic">Remover</button>` : ''}
                        </div>
                    </div>
                    <div class="flex flex-wrap gap-1">
                        ${u.nivel === 'admin' ? '<span class="text-[9px] font-bold text-green-600 uppercase">Tudo Liberado</span>' : (u.categoriasPermitidas?.map(c => `<span class="text-[8px] bg-gray-100 p-1 rounded font-bold">${c}</span>`).join('') || '<span class="text-red-400 text-[8px] uppercase">Nenhuma permissão</span>')}
                    </div>
                </div>`;
            });
        };

        window.bloquearAcesso = async (id, status) => {
            await updateDoc(doc(db, "usuarios", id), { ativo: !status }); carregarLogins();
        };

        window.removerAcesso = async (id) => {
            if(confirm("Remover este login?")) { await deleteDoc(doc(db, "usuarios", id)); carregarLogins(); }
        };

        window.switchTab = (tab) => {
            document.getElementById('content-usuarios').classList.toggle('active', tab === 'usuarios');
            document.getElementById('content-logs').classList.toggle('active', tab === 'logs');
            document.getElementById('btnTabUser').className = tab === 'usuarios' ? 'pb-2 font-bold text-blue-600 border-b-2 border-blue-600 uppercase text-xs' : 'pb-2 font-bold text-gray-400 uppercase text-xs';
            document.getElementById('btnTabLogs').className = tab === 'logs' ? 'pb-2 font-bold text-blue-600 border-b-2 border-blue-600 uppercase text-xs' : 'pb-2 font-bold text-gray-400 uppercase text-xs';
            if(tab === 'logs') carregarLogs();
        };

        window.carregarLogs = async () => {
            const snap = await getDocs(query(collection(db, "logs"), orderBy("data", "desc")));
            const lista = document.getElementById('listaLogs');
            lista.innerHTML = "";
            snap.forEach(d => {
                const l = d.data();
                const data = l.data?.toDate().toLocaleString() || "...";
                lista.innerHTML += `<div class="p-1 border-b border-white/10 uppercase"><span class="text-gray-500 font-bold">[${data}]</span> <b>${l.usuario}:</b> ${l.acao}</div>`;
            });
        };

        document.getElementById('adminGear').onclick = () => { document.getElementById('modalAdmin').classList.add('open'); carregarLogins(); };
        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');
    </script>
</body>
</html>
