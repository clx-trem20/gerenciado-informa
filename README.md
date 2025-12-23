<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Informa - Gestão de Membros</title>
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
    </style>
</head>
<body class="bg-slate-100 min-h-screen font-sans flex flex-col">

    <div id="login-screen" class="flex flex-1 items-center justify-center px-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl w-full max-w-md border-t-8 border-blue-600">
            <h1 class="text-3xl font-black text-center text-blue-600 mb-2 italic uppercase tracking-tighter">Informa</h1>
            <p class="text-gray-400 text-center text-[10px] font-bold mb-8 tracking-widest uppercase">Gerenciamento Interno</p>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-4 mb-4 border rounded-2xl outline-blue-600 bg-gray-50 font-bold">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-4 mb-6 border rounded-2xl outline-blue-600 bg-gray-50 font-bold">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold hover:bg-blue-700 shadow-lg transition-all">ENTRAR</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-bold"></p>
        </div>
    </div>

    <div id="sistema" class="hidden flex-1 flex flex-col">
        <nav class="bg-white border-b px-6 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-black text-blue-700 uppercase tracking-tighter italic">Informa</h2>
            <div class="flex items-center space-x-3">
                <button id="btnExcel" onclick="exportarExcelPorCategoria()" class="hidden bg-green-600 text-white text-[10px] px-4 py-2 rounded-full font-bold uppercase hover:bg-green-700 transition">📊 Excel</button>
                <button id="adminGear" class="hidden text-2xl hover:bg-gray-100 p-2 rounded-full transition">⚙️</button>
                <button id="btnLogout" class="text-gray-500 font-bold text-xs hover:text-red-500 transition-colors uppercase">Sair</button>
            </div>
        </nav>

        <main class="p-6 max-w-6xl mx-auto w-full flex-1">
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

    <footer class="py-6 text-center text-gray-400 text-[10px] font-bold uppercase tracking-[3px] mt-auto">
        © 2025 – Criado por CLX
    </footer>

    <div id="drawerOverlay" onclick="fecharDrawerMembro()" class="fixed inset-0 bg-black/40 hidden z-40"></div>
    <div id="drawerMembro" class="drawer fixed top-0 right-0 h-full w-full max-w-md bg-white shadow-2xl z-50 p-8 flex flex-col">
        <h2 class="text-2xl font-black mb-8 text-gray-800 uppercase tracking-tighter italic">Dados do Membro</h2>
        <input type="hidden" id="editMembroId">
        <div class="space-y-4 overflow-y-auto pr-2 flex-1">
            <input id="mNome" placeholder="Nome Completo" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 font-medium">
            <select id="mCategoria" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 font-medium"></select>
            <input id="mEmail" type="email" placeholder="E-mail" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
            <input id="mTelefone" type="text" placeholder="WhatsApp (DDD + Número)" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600">
            <div class="grid grid-cols-2 gap-3">
                <select id="mMesEntrada" class="p-4 border rounded-2xl bg-gray-50 outline-blue-600">
                    <option value="Janeiro">Janeiro</option><option value="Fevereiro">Fevereiro</option><option value="Março">Março</option>
                    <option value="Abril">Abril</option><option value="Maio">Maio</option><option value="Junho">Junho</option>
                    <option value="Julho">Julho</option><option value="Agosto">Agosto</option><option value="Setembro">Setembro</option>
                    <option value="Outubro">Outubro</option><option value="Novembro">Novembro</option><option value="Dezembro">Dezembro</option>
                </select>
                <input id="mAnoEntrada" type="number" placeholder="Ano" class="p-4 border rounded-2xl bg-gray-50 outline-blue-600">
            </div>
        </div>
        <button onclick="salvarMembroFirebase()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black shadow-xl mt-6 hover:bg-blue-700 transition-all uppercase">Salvar Membro</button>
    </div>

    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-4xl rounded-3xl p-8 max-h-[85vh] overflow-hidden flex flex-col relative shadow-2xl">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-2xl font-bold">&times;</button>
            <div class="flex space-x-6 border-b mb-6 uppercase text-[10px] font-black tracking-widest">
                <button onclick="switchTab('usuarios')" class="pb-2 text-blue-600 border-b-2 border-blue-600">Acessos</button>
                <button onclick="switchTab('logs')" class="pb-2 text-gray-400">Logs</button>
            </div>
            <div id="content-usuarios" class="tab-content active overflow-y-auto pr-2">
                <div class="bg-blue-50 p-6 rounded-3xl mb-6">
                    <input type="hidden" id="editUserId">
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-3 mb-4 italic">
                        <input id="accUser" placeholder="Login" class="p-3 rounded-xl border font-bold uppercase text-xs">
                        <input id="accPass" type="password" placeholder="Senha" class="p-3 rounded-xl border font-bold">
                        <select id="accNivel" class="p-3 rounded-xl border font-bold text-xs uppercase">
                            <option value="user">User</option><option value="admin">Admin</option>
                        </select>
                    </div>
                    <div id="gridCategoriasPermitidas" class="grid grid-cols-2 md:grid-cols-5 gap-2 bg-white p-4 rounded-2xl border mb-4"></div>
                    <button onclick="salvarUsuarioSistema()" class="w-full bg-blue-600 text-white rounded-xl font-bold py-3 uppercase">Salvar Acesso</button>
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
        const mCatSelect = document.getElementById('mCategoria');
        mCatSelect.innerHTML = '<option value="">Selecione a Categoria</option>';
        listaCats.forEach(cat => {
            gridCats.innerHTML += `<label class="flex items-center space-x-2 text-[9px] font-bold text-gray-600 cursor-pointer"><input type="checkbox" name="catPermissao" value="${cat}" class="w-3 h-3 text-blue-600"><span>${cat}</span></label>`;
            mCatSelect.innerHTML += `<option value="${cat}">${cat}</option>`;
        });

        async function registrarLog(acao) { await addDoc(collection(db, "logs"), { usuario: userLogado?.usuario || "Sistema", acao, data: serverTimestamp() }); }

        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value;
            const p = document.getElementById('loginPass').value;
            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snap = await getDocs(q);
            if (snap.empty) { document.getElementById('erro').innerText = "Credenciais inválidas"; return; }
            const userData = snap.docs[0].data();
            if (userData.status === "Inativo") { document.getElementById('erro').innerText = "BLOQUEADO!"; return; }
            userLogado = userData;
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            if (userLogado.nivel === 'admin' || userLogado.categoriasPermitidas?.includes("Presidência")) document.getElementById('btnExcel').classList.remove('hidden');
            if (userLogado.nivel === 'admin') document.getElementById('adminGear').classList.remove('hidden');
            carregarMembros();
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        window.abrirDrawerMembro = (id = null) => {
            document.getElementById('editMembroId').value = id || "";
            document.getElementById('drawerMembro').classList.add('open');
            document.getElementById('drawerOverlay').classList.remove('hidden');
            if(!id) {
                document.getElementById('mNome').value = ""; 
                document.getElementById('mEmail').value = "";
                document.getElementById('mTelefone').value = "";
                document.getElementById('mAnoEntrada').value = new Date().getFullYear();
            }
        };

        window.fecharDrawerMembro = () => { document.getElementById('drawerMembro').classList.remove('open'); document.getElementById('drawerOverlay').classList.add('hidden'); };

        window.salvarMembroFirebase = async () => {
            const id = document.getElementById('editMembroId').value;
            const m = { 
                nome: document.getElementById('mNome').value, 
                categoria: document.getElementById('mCategoria').value, 
                email: document.getElementById('mEmail').value, 
                telefone: document.getElementById('mTelefone').value,
                mesEntrada: document.getElementById('mMesEntrada').value, 
                anoEntrada: document.getElementById('mAnoEntrada').value 
            };
            if(!m.nome || !m.categoria) return alert("Nome e Categoria são obrigatórios!");
            if(id) await updateDoc(doc(db, "membros", id), m);
            else { m.status = "Ativo"; await addDoc(collection(db, "membros"), m); }
            fecharDrawerMembro(); carregarMembros();
        };

        window.carregarMembros = async () => {
            const snap = await getDocs(query(collection(db, "membros"), orderBy("nome", "asc")));
            const lista = document.getElementById('listaMembros');
            lista.innerHTML = "";
            const isPres = userLogado.categoriasPermitidas?.includes("Presidência");
            const isAdmin = userLogado.nivel === 'admin';
            snap.forEach(d => {
                const m = d.data();
                if(!isAdmin && !isPres && !userLogado.categoriasPermitidas?.includes(m.categoria)) return;
                const inativo = m.status === 'Inativo';
                lista.innerHTML += `
                <tr class="hover:bg-gray-50 border-b ${inativo ? 'bg-red-50/50 opacity-60' : ''}">
                    <td class="px-6 py-4 font-bold text-gray-800">${m.nome}</td>
                    <td class="px-6 py-4"><span class="text-[9px] font-black text-blue-600 bg-blue-50 px-2 py-1 rounded border uppercase">${m.categoria}</span></td>
                    <td class="px-6 py-4 text-center text-[10px] font-bold text-gray-500">${m.mesEntrada.slice(0,3)}/${m.anoEntrada}</td>
                    <td class="px-6 py-4 text-center"><button onclick="toggleStatusMembro('${d.id}', '${m.status}')" class="px-3 py-1 rounded-full text-[9px] font-black uppercase ${!inativo ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}">${m.status || 'Ativo'}</button></td>
                    <td class="px-6 py-4 text-center space-x-2 text-lg">
                        <button onclick="enviarComunicacao('whatsapp', '${m.telefone}', '${m.nome}', '${m.status || 'Ativo'}', '${m.email}')" title="WhatsApp">💬</button>
                        <button onclick="enviarComunicacao('email', '${m.telefone}', '${m.nome}', '${m.status || 'Ativo'}', '${m.email}')" title="E-mail">✉️</button>
                        <button onclick="abrirEdicaoMembro('${d.id}')" title="Editar">✏️</button>
                        <button onclick="excluirMembro('${d.id}', '${m.nome}')" class="text-red-500" title="Excluir">🗑️</button>
                    </td>
                </tr>`;
            });
        };

        window.enviarComunicacao = (tipo, fone, nome, status, email) => {
            let msg = "";
            if (status === "Ativo") {
                msg = `Olá, ${nome}!\n\nInformamos que, após 3 meses de estágio, você foi efetivada no Informa.\n\nDurante esse período, apresentou desempenho consistente, comprometimento e responsabilidade, atendendo às expectativas da função.\n\nParabenizamos pela efetivação e desejamos sucesso e crescimento contínuo nesta nova etapa.\n\nAtenciosamente,\nEquipe de Gestão – Jornal Informa\n© 2025 – Criado por CLX`;
            } else {
                msg = `Olá, ${nome}.\n\nInformamos que, após o período de 3 meses de estágio, a efetivação não foi possível neste momento.\n\nAgradecemos pelo empenho e dedicação demonstrados durante esse período.\n\nDesejamos sucesso em seus próximos passos e ficamos à disposição para futuras oportunidades.\n\nAtenciosamente,\nEquipe de Gestão – Jornal Informa\n© 2025 – Criado por CLX`;
            }

            if (tipo === 'email') {
                if(!email) return alert("E-mail não cadastrado.");
                const assunto = status === "Ativo" ? "Parabéns pela Efetivação" : "Comunicado de Estágio";
                window.location.href = `mailto:${email}?subject=${encodeURIComponent(assunto)}&body=${encodeURIComponent(msg)}`;
            } else {
                if(!fone) {
                    fone = prompt("Telefone não encontrado. Digite com DDD (apenas números):", "55");
                }
                if(fone) {
                    const limpo = fone.replace(/\D/g, '');
                    window.open(`https://api.whatsapp.com/send?phone=${limpo}&text=${encodeURIComponent(msg)}`, '_blank');
                }
            }
        };

        window.toggleStatusMembro = async (id, status) => { await updateDoc(doc(db, "membros", id), { status: status === 'Inativo' ? 'Ativo' : 'Inativo' }); carregarMembros(); };
        window.excluirMembro = async (id, nome) => { if(confirm(`Excluir permanentemente ${nome}?`)) { await deleteDoc(doc(db, "membros", id)); carregarMembros(); } };
        
        window.exportarExcelPorCategoria = async () => {
            const snap = await getDocs(collection(db, "membros"));
            const wb = XLSX.utils.book_new();
            const dados = {};
            snap.forEach(d => { const m = d.data(); if(!dados[m.categoria]) dados[m.categoria] = []; dados[m.categoria].push({"Nome": m.nome, "Telefone": m.telefone, "E-mail": m.email, "Status": m.status || "Ativo"}); });
            Object.keys(dados).forEach(c => XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(dados[c]), c.substring(0,30)));
            XLSX.writeFile(wb, "Equipe_Informa.xlsx");
        };

        window.abrirEdicaoMembro = async (id) => {
            const snap = await getDocs(collection(db, "membros"));
            snap.forEach(d => {
                if(d.id === id) {
                    const m = d.data(); abrirDrawerMembro(id);
                    document.getElementById('mNome').value = m.nome; 
                    document.getElementById('mEmail').value = m.email;
                    document.getElementById('mTelefone').value = m.telefone || "";
                    document.getElementById('mMesEntrada').value = m.mesEntrada; 
                    document.getElementById('mAnoEntrada').value = m.anoEntrada;
                    document.getElementById('mCategoria').value = m.categoria;
                }
            });
        };

        window.salvarUsuarioSistema = async () => {
            const id = document.getElementById('editUserId').value;
            const u = { usuario: document.getElementById('accUser').value, senha: document.getElementById('accPass').value, nivel: document.getElementById('accNivel').value, categoriasPermitidas: Array.from(document.querySelectorAll('input[name="catPermissao"]:checked')).map(c => c.value) };
            if(id) await updateDoc(doc(db, "usuarios", id), u); else { u.status = "Ativo"; await addDoc(collection(db, "usuarios"), u); }
            carregarLogins();
        };

        window.carregarLogins = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos'); lista.innerHTML = "";
            snap.forEach(d => { const u = d.data(); lista.innerHTML += `<div class="flex justify-between p-3 border rounded-2xl bg-white text-[10px] font-bold items-center"><span>${u.usuario.toUpperCase()}</span><div class="space-x-2"><button onclick="toggleStatusUser('${d.id}', '${u.status}')" class="px-2 py-1 rounded ${u.status === 'Inativo' ? 'bg-red-500 text-white' : 'bg-green-500 text-white'}">${u.status || 'Ativo'}</button><button onclick="removerAcc('${d.id}')" class="text-gray-400">🗑️</button></div></div>`; });
        };

        window.toggleStatusUser = async (id, status) => { await updateDoc(doc(db, "usuarios", id), { status: status === "Inativo" ? "Ativo" : "Inativo" }); carregarLogins(); };
        window.removerAcc = async (id) => { if(confirm("Excluir?")) { await deleteDoc(doc(db, "usuarios", id)); carregarLogins(); } };
        window.switchTab = (t) => { document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active')); document.getElementById(`content-${t}`).classList.add('active'); if(t==='logs') carregarLogs(); };
        window.carregarLogs = async () => {
            const snap = await getDocs(query(collection(db, "logs"), orderBy("data", "desc")));
            const lista = document.getElementById('listaLogs'); lista.innerHTML = "";
            snap.forEach(d => { const l = d.data(); lista.innerHTML += `<div class="p-1 border-b border-white/10">[${l.data?.toDate().toLocaleString()}] ${l.usuario}: ${l.acao}</div>`; });
        };
        document.getElementById('adminGear').onclick = () => { document.getElementById('modalAdmin').classList.add('open'); carregarLogins(); };
        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');
    </script>
</body>
</html>
