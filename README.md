<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Informa - Sistema de Gestão</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        .drawer { transform: translateX(100%); transition: transform 0.3s ease-in-out; }
        .drawer.open { transform: translateX(0); }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 100; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        .modal.open { display: flex; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        body { font-family: sans-serif; }
    </style>
</head>
<body class="bg-slate-100 min-h-screen">

    <div id="login-screen" class="flex items-center justify-center h-screen px-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl w-full max-w-md border-t-8 border-blue-600">
            <h1 class="text-3xl font-black text-center text-blue-600 mb-2 italic uppercase tracking-tighter">Informa</h1>
            <p class="text-gray-400 text-center text-[10px] font-bold mb-8 tracking-widest uppercase">Gerenciamento Interno</p>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-4 mb-4 border rounded-2xl outline-blue-600 bg-gray-50 font-bold">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-4 mb-6 border rounded-2xl outline-blue-600 bg-gray-50 font-bold">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold hover:bg-blue-700 shadow-lg transition-all">ENTRAR</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-bold"></p>
        </div>
    </div>

    <div id="sistema" class="hidden">
        <nav class="bg-white border-b px-6 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-black text-blue-700 uppercase tracking-tighter italic">Informa</h2>
            <div class="flex items-center space-x-3">
                <button id="btnExcel" onclick="exportarExcelPorCategoria()" class="hidden bg-green-600 text-white text-[10px] px-4 py-2 rounded-full font-bold uppercase hover:bg-green-700 transition flex items-center gap-1">📊 Excel</button>
                <button id="adminGear" class="hidden text-2xl hover:bg-gray-100 p-2 rounded-full transition">⚙️</button>
                <button id="btnLogout" class="text-gray-500 font-bold text-xs hover:text-red-500 transition-colors uppercase">Sair</button>
            </div>
        </nav>

        <main class="p-6 max-w-6xl mx-auto">
            <div class="flex flex-col md:flex-row justify-between items-center mb-8 gap-4">
                <h1 class="text-2xl font-black text-gray-800 uppercase tracking-tighter">Equipe do Jornal</h1>
                <button onclick="abrirDrawerMembro()" class="bg-blue-600 text-white px-8 py-3 rounded-2xl font-bold shadow-lg hover:bg-blue-700 transition">+ Novo Membro</button>
            </div>

            <div class="bg-white rounded-3xl shadow-sm border overflow-hidden overflow-x-auto">
                <table class="min-w-full text-left">
                    <thead class="bg-gray-50 text-gray-400 text-[10px] font-bold uppercase tracking-widest">
                        <tr>
                            <th class="px-6 py-4">Membro</th>
                            <th class="px-6 py-4">Categoria</th>
                            <th class="px-6 py-4 text-center">Entrada</th>
                            <th class="px-6 py-4 text-center">Status</th>
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
            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Nome Completo</label>
                <input id="mNome" placeholder="Ex: João da Silva" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
            </div>
            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Categoria/Setor</label>
                <select id="mCategoria" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
                    <option value="">Selecione...</option>
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
            </div>
            <div class="grid grid-cols-2 gap-3">
                <div>
                    <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Mês Entrada</label>
                    <select id="mMesEntrada" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
                        <option value="Janeiro">Janeiro</option><option value="Fevereiro">Fevereiro</option><option value="Março">Março</option>
                        <option value="Abril">Abril</option><option value="Maio">Maio</option><option value="Junho">Junho</option>
                        <option value="Julho">Julho</option><option value="Agosto">Agosto</option><option value="Setembro">Setembro</option>
                        <option value="Outubro">Outubro</option><option value="Novembro">Novembro</option><option value="Dezembro">Dezembro</option>
                    </select>
                </div>
                <div>
                    <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Ano Entrada</label>
                    <input id="mAnoEntrada" type="number" placeholder="Ex: 2024" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
                </div>
            </div>
            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">E-mail</label>
                <input id="mEmail" type="email" placeholder="email@exemplo.com" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
            </div>
        </div>
        <button onclick="salvarMembroFirebase()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black shadow-xl mt-6 uppercase">Salvar no Sistema</button>
    </div>

    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-4xl rounded-3xl p-8 max-h-[85vh] overflow-hidden flex flex-col relative">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-2xl font-bold">&times;</button>
            <div class="flex space-x-6 border-b mb-6 text-[10px] font-black uppercase tracking-widest">
                <button onclick="switchTab('usuarios')" id="btnTabUser" class="pb-2 text-blue-600 border-b-2 border-blue-600">Acessos</button>
                <button onclick="switchTab('logs')" id="btnTabLogs" class="pb-2 text-gray-400">Logs de Atividade</button>
            </div>
            
            <div id="content-usuarios" class="tab-content active overflow-y-auto pr-2">
                <div class="bg-blue-50 p-6 rounded-3xl mb-6">
                    <p class="text-[10px] font-bold text-blue-600 uppercase mb-3">Criar Novo Acesso</p>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-3 mb-4 italic">
                        <input id="accUser" placeholder="Login" class="p-3 rounded-xl border font-bold">
                        <input id="accPass" type="password" placeholder="Senha" class="p-3 rounded-xl border font-bold">
                        <select id="accNivel" class="p-3 rounded-xl border font-bold">
                            <option value="user">User (Restrito)</option><option value="admin">Admin (Total)</option>
                        </select>
                    </div>
                    <p class="text-[10px] font-bold text-gray-400 mb-2 uppercase">Permitir visualização destas categorias:</p>
                    <div id="gridCategoriasPermitidas" class="grid grid-cols-2 md:grid-cols-5 gap-2 bg-white p-4 rounded-2xl border mb-4"></div>
                    <button onclick="criarLoginSistema()" class="w-full bg-blue-600 text-white rounded-xl font-bold py-3 shadow-lg uppercase">Confirmar Acesso</button>
                </div>
                <div id="listaAcessos" class="space-y-2"></div>
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

        const gridCats = document.getElementById('gridCategoriasPermitidas');
        listaCats.forEach(cat => {
            gridCats.innerHTML += `
                <label class="flex items-center space-x-2 text-[9px] font-bold text-gray-600 cursor-pointer">
                    <input type="checkbox" name="catPermissao" value="${cat}" class="w-3 h-3 text-blue-600">
                    <span>${cat}</span>
                </label>
            `;
        });

        async function registrarLog(acao) {
            await addDoc(collection(db, "logs"), { usuario: userLogado?.usuario || "Sistema", acao, data: serverTimestamp() });
        }

        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value;
            const p = document.getElementById('loginPass').value;
            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snap = await getDocs(q);

            if (snap.empty) { document.getElementById('erro').innerText = "Usuário ou senha inválidos"; return; }
            const userData = snap.docs[0].data();
            if (!userData.ativo) { document.getElementById('erro').innerText = "ACESSO BLOQUEADO!"; return; }

            userLogado = userData;
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            
            if (userData.nivel === 'admin') {
                document.getElementById('adminGear').classList.remove('hidden');
                document.getElementById('btnExcel').classList.remove('hidden');
            }
            
            registrarLog("Login efetuado");
            carregarMembros();
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        window.abrirDrawerMembro = (id = null) => {
            document.getElementById('editMembroId').value = id || "";
            document.getElementById('drawerMembro').classList.add('open');
            document.getElementById('drawerOverlay').classList.remove('hidden');
            if(!id) {
                document.getElementById('mNome').value = ""; document.getElementById('mEmail').value = "";
                document.getElementById('mAnoEntrada').value = ""; document.getElementById('mCategoria').value = "";
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
                mesEntrada: document.getElementById('mMesEntrada').value,
                anoEntrada: document.getElementById('mAnoEntrada').value
            };
            if(!m.nome || !m.categoria) return alert("Preencha Nome e Categoria!");

            if(id) {
                await updateDoc(doc(db, "membros", id), m);
                registrarLog(`Editou membro: ${m.nome}`);
            } else {
                m.status = "Ativo";
                await addDoc(collection(db, "membros"), m);
                registrarLog(`Criou membro: ${m.nome}`);
            }
            fecharDrawerMembro(); carregarMembros();
        };

        window.carregarMembros = async () => {
            const snap = await getDocs(query(collection(db, "membros"), orderBy("nome", "asc")));
            const lista = document.getElementById('listaMembros');
            lista.innerHTML = "";
            
            snap.forEach(d => {
                const m = d.data();
                if(userLogado.nivel === 'user' && !userLogado.categoriasPermitidas?.includes(m.categoria)) return;

                const inativo = m.status === 'Inativo';
                lista.innerHTML += `
                <tr class="hover:bg-gray-50 border-b ${inativo ? 'bg-red-50/50 opacity-60' : ''}">
                    <td class="px-6 py-4">
                        <div class="font-bold text-gray-800">${m.nome}</div>
                        <div class="text-[10px] text-gray-400">${m.email || 'Sem e-mail'}</div>
                    </td>
                    <td class="px-6 py-4"><span class="text-[9px] font-black text-blue-600 bg-blue-50 px-2 py-1 rounded border uppercase">${m.categoria}</span></td>
                    <td class="px-6 py-4 text-center text-[10px] font-bold text-gray-500">${m.mesEntrada?.substring(0,3)}/${m.anoEntrada}</td>
                    <td class="px-6 py-4 text-center">
                        <button onclick="alternarStatus('${d.id}', '${m.status}')" class="px-4 py-1.5 rounded-full text-[9px] font-black uppercase transition-all shadow-sm ${!inativo ? 'bg-green-100 text-green-700 hover:bg-green-200' : 'bg-red-100 text-red-700 hover:bg-red-200'}">
                            ${m.status || 'Ativo'}
                        </button>
                    </td>
                    <td class="px-6 py-4 text-center space-x-3 text-lg">
                        <button onclick="abrirEdicaoMembro('${d.id}')" title="Editar">✏️</button>
                        <button onclick="excluirMembro('${d.id}', '${m.nome}')" title="Excluir">🗑️</button>
                    </td>
                </tr>`;
            });
        };

        window.alternarStatus = async (id, statusAtual) => {
            const novo = statusAtual === 'Inativo' ? 'Ativo' : 'Inativo';
            await updateDoc(doc(db, "membros", id), { status: novo });
            registrarLog(`Membro ${id} agora está ${novo}`);
            carregarMembros();
        };

        window.exportarExcelPorCategoria = async () => {
            const snap = await getDocs(collection(db, "membros"));
            const categoriasFiltro = {};

            snap.forEach(d => {
                const m = d.data();
                if (!categoriasFiltro[m.categoria]) categoriasFiltro[m.categoria] = [];
                categoriasFiltro[m.categoria].push({
                    "Nome": m.nome,
                    "E-mail": m.email,
                    "Mês Entrada": m.mesEntrada,
                    "Ano Entrada": m.anoEntrada,
                    "Status": m.status || "Ativo"
                });
            });

            const wb = XLSX.utils.book_new();
            Object.keys(categoriasFiltro).forEach(cat => {
                const ws = XLSX.utils.json_to_sheet(categoriasFiltro[cat]);
                XLSX.utils.book_append_sheet(wb, ws, cat.substring(0, 31)); 
            });

            XLSX.writeFile(wb, "Informa_Relatorio_Por_Setores.xlsx");
            registrarLog("Exportou planilha Excel");
        };

        window.abrirEdicaoMembro = async (id) => {
            const snap = await getDocs(collection(db, "membros"));
            snap.forEach(d => {
                if(d.id === id) {
                    const m = d.data();
                    abrirDrawerMembro(id);
                    document.getElementById('mNome').value = m.nome;
                    document.getElementById('mEmail').value = m.email;
                    document.getElementById('mMesEntrada').value = m.mesEntrada || "Janeiro";
                    document.getElementById('mAnoEntrada').value = m.anoEntrada;
                    document.getElementById('mCategoria').value = m.categoria;
                }
            });
        };

        window.excluirMembro = async (id, nome) => {
            if(confirm(`Remover permanentemente ${nome}?`)) { 
                await deleteDoc(doc(db, "membros", id)); 
                registrarLog(`Deletou membro: ${nome}`); 
                carregarMembros(); 
            }
        };

        window.criarLoginSistema = async () => {
            const u = document.getElementById('accUser').value;
            const p = document.getElementById('accPass').value;
            const n = document.getElementById('accNivel').value;
            const selecionadas = Array.from(document.querySelectorAll('input[name="catPermissao"]:checked')).map(c => c.value);
            if(!u || !p) return alert("Preencha os campos!");
            await addDoc(collection(db, "usuarios"), { usuario: u, senha: p, nivel: n, ativo: true, categoriasPermitidas: selecionadas });
            registrarLog(`Criou acesso para ${u}`);
            carregarLogins();
            alert("Sucesso!");
        };

        window.carregarLogins = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos');
            lista.innerHTML = "";
            snap.forEach(d => {
                const u = d.data();
                lista.innerHTML += `<div class="flex justify-between p-3 border rounded-2xl bg-white shadow-sm text-[10px] items-center">
                    <span class="font-bold">${u.usuario} (${u.nivel.toUpperCase()})</span>
                    <button onclick="removerAcc('${d.id}')" class="text-red-500 font-black">REMOVER</button>
                </div>`;
            });
        };

        window.removerAcc = async (id) => { if(confirm("Apagar login?")) { await deleteDoc(doc(db, "usuarios", id)); carregarLogins(); } };

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
                lista.innerHTML += `<div class="p-1 uppercase border-b border-white/5 font-bold italic">[${l.data?.toDate().toLocaleString() || '... '}] ${l.usuario}: ${l.acao}</div>`;
            });
        };

        document.getElementById('adminGear').onclick = () => { document.getElementById('modalAdmin').classList.add('open'); carregarLogins(); };
        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');
    </script>
</body>
</html>
