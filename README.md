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
        /* Custom Scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
    </style>
</head>
<body class="bg-slate-100 min-h-screen font-sans flex flex-col">

    <!-- Tela de Login -->
    <div id="login-screen" class="flex flex-1 items-center justify-center px-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl w-full max-w-md border-t-8 border-blue-600">
            <h1 class="text-3xl font-black text-center text-blue-600 mb-2 italic uppercase tracking-tighter">Informa</h1>
            <p class="text-gray-400 text-center text-[10px] font-bold mb-8 tracking-widest uppercase">Gerenciamento Interno</p>
            <input type="text" id="loginUser" placeholder="Usuário" class="w-full p-4 mb-4 border rounded-2xl outline-blue-600 bg-gray-50 font-bold uppercase text-xs">
            <input type="password" id="loginPass" placeholder="Senha" class="w-full p-4 mb-6 border rounded-2xl outline-blue-600 bg-gray-50 font-bold">
            <button id="btnLogin" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold hover:bg-blue-700 shadow-lg transition-all uppercase tracking-widest">ENTRAR</button>
            <p id="erro" class="text-red-500 text-center mt-4 text-sm font-bold"></p>
        </div>
    </div>

    <!-- Sistema Principal -->
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
                <button onclick="abrirDrawerMembro()" class="bg-blue-600 text-white px-8 py-3 rounded-2xl font-bold shadow-lg hover:bg-blue-700 transition uppercase text-xs">+ Novo Membro</button>
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

    <!-- Modal de Seleção de Modelo (WhatsApp / E-mail) -->
    <div id="modalComunicacao" class="modal">
        <div class="bg-white w-full max-w-lg rounded-3xl p-8 shadow-2xl mx-4">
            <div class="flex justify-between items-center mb-6">
                <h3 id="modalComunicacaoTitulo" class="text-xl font-black text-gray-800 uppercase italic">Escolha o Modelo</h3>
                <span id="badgeTipo" class="px-3 py-1 bg-blue-100 text-blue-700 text-[10px] font-black rounded-full uppercase">Tipo</span>
            </div>
            <div id="listaModelosComunicacao" class="space-y-3 mb-6 max-h-60 overflow-y-auto pr-2"></div>
            <button onclick="fecharModalComunicacao()" class="w-full py-3 text-gray-400 font-bold uppercase text-[10px] tracking-widest hover:text-gray-600 transition">Cancelar</button>
        </div>
    </div>

    <!-- Modal Admin -->
    <div id="modalAdmin" class="modal">
        <div class="bg-white w-full max-w-4xl rounded-3xl p-8 max-h-[85vh] overflow-hidden flex flex-col relative shadow-2xl mx-4">
            <button onclick="fecharAdmin()" class="absolute top-4 right-6 text-2xl font-bold text-gray-400 hover:text-black transition">&times;</button>
            <div class="flex space-x-6 border-b mb-6 uppercase text-[10px] font-black tracking-widest overflow-x-auto">
                <button onclick="switchTab('usuarios')" id="tabUsuarios" class="pb-2 text-blue-600 border-b-2 border-blue-600 whitespace-nowrap">Acessos</button>
                <button onclick="switchTab('emails')" id="tabEmails" class="pb-2 text-gray-400 whitespace-nowrap">Modelos (Zap/Email)</button>
                <button onclick="switchTab('logs')" id="tabLogs" class="pb-2 text-gray-400 whitespace-nowrap">Logs</button>
            </div>
            
            <!-- Aba Acessos -->
            <div id="content-usuarios" class="tab-content active overflow-y-auto pr-2">
                <div class="bg-blue-50 p-6 rounded-3xl mb-6">
                    <input type="hidden" id="editUserId">
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-3 mb-4 italic">
                        <input id="accUser" placeholder="Login" class="p-3 rounded-xl border font-bold uppercase text-xs outline-blue-600">
                        <input id="accPass" type="password" placeholder="Senha" class="p-3 rounded-xl border font-bold outline-blue-600">
                        <select id="accNivel" class="p-3 rounded-xl border font-bold text-xs uppercase outline-blue-600">
                            <option value="user">User</option><option value="admin">Admin</option>
                        </select>
                    </div>
                    <p class="text-[9px] font-black text-gray-400 uppercase mb-2 ml-1">Permissões de Categoria:</p>
                    <div id="gridCategoriasPermitidas" class="grid grid-cols-2 md:grid-cols-5 gap-2 bg-white p-4 rounded-2xl border mb-4"></div>
                    <button onclick="salvarUsuarioSistema()" class="w-full bg-blue-600 text-white rounded-xl font-bold py-3 uppercase text-xs shadow-md">Salvar Novo Acesso</button>
                </div>
                <div id="listaAcessos" class="space-y-2"></div>
            </div>

            <!-- Aba Modelos de Mensagem -->
            <div id="content-emails" class="tab-content overflow-y-auto pr-2">
                <div class="bg-gray-50 p-6 rounded-3xl mb-6 border border-dashed border-gray-300">
                    <h4 class="text-[10px] font-black uppercase text-gray-400 mb-4 tracking-widest italic">Configurar Modelo Reutilizável</h4>
                    <input type="text" id="templateTitulo" placeholder="Nome do Modelo (Ex: Boas Vindas, Cobrança...)" class="w-full p-3 rounded-xl border mb-3 font-bold text-sm outline-blue-600 uppercase">
                    <input type="text" id="templateAssunto" placeholder="Assunto (Obrigatório apenas para E-mail)" class="w-full p-3 rounded-xl border mb-3 text-sm outline-blue-600">
                    <textarea id="templateTexto" placeholder="Escreva a mensagem aqui...&#10;Use [nome] para o nome do membro ser inserido automaticamente.&#10;Ex: Olá [nome], tudo bem?" class="w-full p-3 rounded-xl border mb-3 h-32 text-sm outline-blue-600"></textarea>
                    <button onclick="salvarModeloEmail()" class="w-full bg-blue-600 text-white rounded-xl font-bold py-3 uppercase text-xs">Adicionar à Lista</button>
                </div>
                <div id="listaModelosAdmin" class="grid grid-cols-1 md:grid-cols-2 gap-3"></div>
            </div>

            <div id="content-logs" class="tab-content overflow-y-auto">
                <div id="listaLogs" class="space-y-1 text-[10px] font-mono bg-slate-900 text-green-400 p-4 rounded-xl min-h-[200px]"></div>
            </div>
        </div>
    </div>

    <!-- Drawer de Membro -->
    <div id="drawerOverlay" onclick="fecharDrawerMembro()" class="fixed inset-0 bg-black/40 hidden z-40"></div>
    <div id="drawerMembro" class="drawer fixed top-0 right-0 h-full w-full max-w-md bg-white shadow-2xl z-50 p-8 flex flex-col">
        <div class="flex justify-between items-center mb-8">
            <h2 class="text-2xl font-black text-gray-800 uppercase tracking-tighter italic">Ficha Cadastral</h2>
            <button onclick="fecharDrawerMembro()" class="text-gray-300 hover:text-black transition text-2xl">&times;</button>
        </div>
        <input type="hidden" id="editMembroId">
        <div class="space-y-4 overflow-y-auto pr-2 flex-1">
            <div class="space-y-1">
                <label class="text-[9px] font-black uppercase text-gray-400 ml-1">Nome Completo</label>
                <input id="mNome" placeholder="..." class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 font-bold uppercase text-xs">
            </div>
            <div class="space-y-1">
                <label class="text-[9px] font-black uppercase text-gray-400 ml-1">Categoria de Trabalho</label>
                <select id="mCategoria" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 font-bold text-xs uppercase"></select>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                <div class="space-y-1">
                    <label class="text-[9px] font-black uppercase text-gray-400 ml-1">E-mail</label>
                    <input id="mEmail" type="email" placeholder="exemplo@gmail.com" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 text-xs">
                </div>
                <div class="space-y-1">
                    <label class="text-[9px] font-black uppercase text-gray-400 ml-1">WhatsApp</label>
                    <input id="mTelefone" type="text" placeholder="11999999999" class="w-full p-4 border rounded-2xl bg-gray-50 outline-blue-600 text-xs">
                </div>
            </div>
            <div class="space-y-1">
                <label class="text-[9px] font-black uppercase text-gray-400 ml-1">Data de Ingresso</label>
                <div class="grid grid-cols-2 gap-3">
                    <select id="mMesEntrada" class="p-4 border rounded-2xl bg-gray-50 outline-blue-600 text-xs font-bold">
                        <option value="Janeiro">Janeiro</option><option value="Fevereiro">Fevereiro</option><option value="Março">Março</option>
                        <option value="Abril">Abril</option><option value="Maio">Maio</option><option value="Junho">Junho</option>
                        <option value="Julho">Julho</option><option value="Agosto">Agosto</option><option value="Setembro">Setembro</option>
                        <option value="Outubro">Outubro</option><option value="Novembro">Novembro</option><option value="Dezembro">Dezembro</option>
                    </select>
                    <input id="mAnoEntrada" type="number" placeholder="Ano" class="p-4 border rounded-2xl bg-gray-50 outline-blue-600 font-bold text-xs">
                </div>
            </div>
        </div>
        <button onclick="salvarMembroFirebase()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black shadow-xl mt-6 hover:bg-blue-700 transition-all uppercase tracking-widest text-xs">Confirmar Dados</button>
    </div>

    <footer class="py-6 text-center text-gray-400 text-[10px] font-bold uppercase tracking-[3px] mt-auto">
        © 2025 – Gerência Informa – CLX
    </footer>

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
        let membroAlvo = null; 
        const listaCats = ["Meio Ambiente", "Linguagens", "Comunicações", "Edição de Vídeo", "Cultura", "Secretaria", "Esportes", "Presidência", "Informações", "Designer"];

        const gridCats = document.getElementById('gridCategoriasPermitidas');
        const mCatSelect = document.getElementById('mCategoria');

        mCatSelect.innerHTML = '<option value="">Selecione...</option>';
        listaCats.forEach(cat => {
            gridCats.innerHTML += `<label class="flex items-center space-x-2 text-[9px] font-black text-gray-500 cursor-pointer uppercase"><input type="checkbox" name="catPermissao" value="${cat}" class="w-3 h-3 text-blue-600 rounded"><span>${cat}</span></label>`;
            mCatSelect.innerHTML += `<option value="${cat}">${cat}</option>`;
        });

        async function registrarLog(acao) { 
            await addDoc(collection(db, "logs"), { usuario: userLogado?.usuario || "Sistema", acao, data: serverTimestamp() }); 
        }

        // Login
        document.getElementById('btnLogin').onclick = async () => {
            const u = document.getElementById('loginUser').value.trim();
            const p = document.getElementById('loginPass').value.trim();
            const q = query(collection(db, "usuarios"), where("usuario", "==", u), where("senha", "==", p));
            const snap = await getDocs(q);
            if (snap.empty) { document.getElementById('erro').innerText = "Acesso negado"; return; }
            const userData = snap.docs[0].data();
            if (userData.status === "Inativo") { document.getElementById('erro').innerText = "CONTA BLOQUEADA!"; return; }
            userLogado = userData;
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('sistema').classList.remove('hidden');
            if (userLogado.nivel === 'admin' || userLogado.categoriasPermitidas?.includes("Presidência")) document.getElementById('btnExcel').classList.remove('hidden');
            if (userLogado.nivel === 'admin') document.getElementById('adminGear').classList.remove('hidden');
            registrarLog("Iniciou sessão");
            carregarMembros();
        };

        document.getElementById('btnLogout').onclick = () => location.reload();

        // Carregar Membros
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
                    <td class="px-6 py-4 font-bold text-gray-800 uppercase text-xs">${m.nome}</td>
                    <td class="px-6 py-4"><span class="text-[9px] font-black text-blue-600 bg-blue-50 px-2 py-1 rounded border border-blue-100 uppercase">${m.categoria}</span></td>
                    <td class="px-6 py-4 text-center text-[10px] font-bold text-gray-500">${m.mesEntrada.slice(0,3)}/${m.anoEntrada}</td>
                    <td class="px-6 py-4 text-center">
                        <button onclick="toggleStatusMembro('${d.id}', '${m.status}')" class="px-3 py-1 rounded-full text-[9px] font-black uppercase tracking-tighter ${!inativo ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}">
                            ${m.status || 'Ativo'}
                        </button>
                    </td>
                    <td class="px-6 py-4 text-center space-x-4">
                        <button onclick="prepararComunicacao('whatsapp', '${m.telefone}', '${m.nome}', '${m.email}')" title="Enviar WhatsApp" class="hover:scale-125 transition-transform">🟢</button>
                        <button onclick="prepararComunicacao('email', '${m.telefone}', '${m.nome}', '${m.email}')" title="Enviar E-mail" class="hover:scale-125 transition-transform">🔵</button>
                        <button onclick="abrirEdicaoMembro('${d.id}')" title="Editar" class="text-gray-400 hover:text-blue-600 transition">✏️</button>
                        <button onclick="excluirMembro('${d.id}', '${m.nome}')" class="text-gray-400 hover:text-red-500 transition" title="Excluir">🗑️</button>
                    </td>
                </tr>`;
            });
        };

        // --- COMUNICAÇÃO (WhatsApp e E-mail) ---

        window.prepararComunicacao = async (tipo, fone, nome, email) => {
            membroAlvo = { tipo, fone, nome, email };
            const badge = document.getElementById('badgeTipo');
            const titulo = document.getElementById('modalComunicacaoTitulo');
            
            if(tipo === 'whatsapp') {
                badge.innerText = "WhatsApp";
                badge.className = "px-3 py-1 bg-green-100 text-green-700 text-[10px] font-black rounded-full uppercase";
                titulo.innerText = "Modelo de Zap";
            } else {
                badge.innerText = "E-mail";
                badge.className = "px-3 py-1 bg-blue-100 text-blue-700 text-[10px] font-black rounded-full uppercase";
                titulo.innerText = "Modelo de E-mail";
            }

            const snap = await getDocs(collection(db, "modelos_email"));
            const lista = document.getElementById('listaModelosComunicacao');
            lista.innerHTML = "";
            
            if(snap.empty) {
                lista.innerHTML = "<p class='text-center text-gray-400 text-xs italic p-4'>Nenhum modelo cadastrado. Vá em Admin > Modelos.</p>";
            } else {
                snap.forEach(d => {
                    const mod = d.data();
                    lista.innerHTML += `
                        <button onclick="enviarMensagemProgramada('${d.id}')" class="w-full text-left p-4 border-2 border-gray-100 rounded-2xl hover:border-blue-600 hover:bg-blue-50 transition-all group">
                            <div class="flex justify-between items-start">
                                <p class="font-black text-gray-700 group-hover:text-blue-600 uppercase text-[10px] tracking-widest">${mod.titulo}</p>
                                <span class="text-[8px] text-gray-300 font-bold uppercase">Selecionar</span>
                            </div>
                            <p class="text-[10px] text-gray-400 truncate mt-1">${mod.texto.substring(0, 40)}...</p>
                        </button>
                    `;
                });
            }
            document.getElementById('modalComunicacao').classList.add('open');
        };

        window.fecharModalComunicacao = () => document.getElementById('modalComunicacao').classList.remove('open');

        window.enviarMensagemProgramada = async (idModelo) => {
            const snap = await getDocs(collection(db, "modelos_email"));
            let modelo = null;
            snap.forEach(d => { if(d.id === idModelo) modelo = d.data(); });

            if(!modelo) return;

            let textoFinal = modelo.texto.replace(/\[nome\]/gi, membroAlvo.nome);
            
            if(membroAlvo.tipo === 'email') {
                if(!membroAlvo.email) return alert("Erro: Este membro não possui e-mail cadastrado.");
                const assuntoFinal = modelo.assunto || "Informa - Gestão";
                window.location.href = `mailto:${membroAlvo.email}?subject=${encodeURIComponent(assuntoFinal)}&body=${encodeURIComponent(textoFinal)}`;
            } else {
                if(!membroAlvo.fone) return alert("Erro: Este membro não possui WhatsApp cadastrado.");
                const foneLimpo = membroAlvo.fone.replace(/\D/g, '');
                // Detectar se precisa de prefixo 55 se não tiver
                const foneFinal = foneLimpo.length <= 11 ? '55' + foneLimpo : foneLimpo;
                window.open(`https://api.whatsapp.com/send?phone=${foneFinal}&text=${encodeURIComponent(textoFinal)}`, '_blank');
            }
            fecharModalComunicacao();
            registrarLog(`Disparo de ${membroAlvo.tipo.toUpperCase()}: ${membroAlvo.nome}`);
        };

        // --- ADMIN: MODELOS ---

        window.salvarModeloEmail = async () => {
            const titulo = document.getElementById('templateTitulo').value.toUpperCase();
            const assunto = document.getElementById('templateAssunto').value;
            const texto = document.getElementById('templateTexto').value;

            if(!titulo || !texto) return alert("O Nome do Modelo e o Texto são obrigatórios!");

            const m = { titulo, assunto, texto };
            await addDoc(collection(db, "modelos_email"), m);
            
            document.getElementById('templateTitulo').value = "";
            document.getElementById('templateAssunto').value = "";
            document.getElementById('templateTexto').value = "";
            
            carregarModelosAdmin();
            registrarLog(`Novo modelo criado: ${titulo}`);
        };

        window.carregarModelosAdmin = async () => {
            const snap = await getDocs(collection(db, "modelos_email"));
            const lista = document.getElementById('listaModelosAdmin');
            lista.innerHTML = "";
            snap.forEach(d => {
                const m = d.data();
                lista.innerHTML += `
                <div class="p-4 bg-white border rounded-2xl flex flex-col justify-between shadow-sm">
                    <div>
                        <div class="flex justify-between items-start mb-2">
                            <p class="text-[10px] font-black text-blue-600 uppercase tracking-tighter">${m.titulo}</p>
                            <button onclick="removerModelo('${d.id}')" class="text-red-300 hover:text-red-500 transition">✕</button>
                        </div>
                        <p class="text-[9px] text-gray-400 line-clamp-3 leading-relaxed">${m.texto}</p>
                    </div>
                </div>`;
            });
        };

        window.removerModelo = async (id) => {
            if(confirm("Deseja apagar este modelo de mensagem permanentemente?")) {
                await deleteDoc(doc(db, "modelos_email", id));
                carregarModelosAdmin();
                registrarLog("Modelo removido");
            }
        };

        // --- ADMIN: NAVEGAÇÃO ---

        window.switchTab = (t) => {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.getElementById(`content-${t}`).classList.add('active');
            
            document.getElementById('tabUsuarios').className = "pb-2 whitespace-nowrap " + (t === 'usuarios' ? 'text-blue-600 border-b-2 border-blue-600' : 'text-gray-400');
            document.getElementById('tabEmails').className = "pb-2 whitespace-nowrap " + (t === 'emails' ? 'text-blue-600 border-b-2 border-blue-600' : 'text-gray-400');
            document.getElementById('tabLogs').className = "pb-2 whitespace-nowrap " + (t === 'logs' ? 'text-blue-600 border-b-2 border-blue-600' : 'text-gray-400');
            
            if(t==='logs') carregarLogs();
            if(t==='emails') carregarModelosAdmin();
            if(t==='usuarios') carregarLogins();
        };

        // --- GESTÃO DE MEMBROS (DRAWER) ---

        window.abrirDrawerMembro = () => {
            document.getElementById('editMembroId').value = "";
            document.getElementById('mNome').value = "";
            document.getElementById('mEmail').value = "";
            document.getElementById('mTelefone').value = "";
            document.getElementById('mAnoEntrada').value = new Date().getFullYear();
            document.getElementById('drawerMembro').classList.add('open');
            document.getElementById('drawerOverlay').classList.remove('hidden');
        };

        window.abrirEdicaoMembro = async (id) => {
            const snap = await getDocs(collection(db, "membros"));
            snap.forEach(d => {
                if(d.id === id) {
                    const m = d.data();
                    document.getElementById('editMembroId').value = id;
                    document.getElementById('mNome').value = m.nome;
                    document.getElementById('mCategoria').value = m.categoria;
                    document.getElementById('mEmail').value = m.email;
                    document.getElementById('mTelefone').value = m.telefone;
                    document.getElementById('mMesEntrada').value = m.mesEntrada;
                    document.getElementById('mAnoEntrada').value = m.anoEntrada;
                }
            });
            document.getElementById('drawerMembro').classList.add('open');
            document.getElementById('drawerOverlay').classList.remove('hidden');
        };

        window.fecharDrawerMembro = () => { 
            document.getElementById('drawerMembro').classList.remove('open'); 
            document.getElementById('drawerOverlay').classList.add('hidden'); 
        };

        window.salvarMembroFirebase = async () => {
            const id = document.getElementById('editMembroId').value;
            const m = { 
                nome: document.getElementById('mNome').value.trim(), 
                categoria: document.getElementById('mCategoria').value, 
                email: document.getElementById('mEmail').value.trim(), 
                telefone: document.getElementById('mTelefone').value.trim(), 
                mesEntrada: document.getElementById('mMesEntrada').value, 
                anoEntrada: parseInt(document.getElementById('mAnoEntrada').value) || 2025 
            };
            if(!m.nome || !m.categoria) return alert("Erro: Nome e Categoria são fundamentais!");

            if(id) {
                await updateDoc(doc(db, "membros", id), m);
                registrarLog(`Editou: ${m.nome}`);
            } else {
                m.status = "Ativo";
                await addDoc(collection(db, "membros"), m);
                registrarLog(`Novo Cadastro: ${m.nome}`);
            }
            fecharDrawerMembro();
            carregarMembros();
        };

        window.toggleStatusMembro = async (id, status) => { 
            const novoStatus = status === 'Inativo' ? 'Ativo' : 'Inativo';
            await updateDoc(doc(db, "membros", id), { status: novoStatus }); 
            carregarMembros(); 
            registrarLog(`Membro ${id} agora está ${novoStatus}`);
        };

        window.excluirMembro = async (id, nome) => { 
            if(confirm(`Tem certeza que deseja apagar permanentemente o registro de ${nome}?`)) { 
                await deleteDoc(doc(db, "membros", id)); 
                registrarLog(`Eliminou registro: ${nome}`);
                carregarMembros(); 
            } 
        };

        window.exportarExcelPorCategoria = async () => {
            const snap = await getDocs(collection(db, "membros"));
            const wb = XLSX.utils.book_new();
            const dados = {};
            snap.forEach(d => { 
                const m = d.data(); 
                if(!dados[m.categoria]) dados[m.categoria] = []; 
                dados[m.categoria].push({"Nome": m.nome, "E-mail": m.email, "WhatsApp": m.telefone, "Entrada": `${m.mesEntrada}/${m.anoEntrada}`, "Status": m.status}); 
            });
            Object.keys(dados).forEach(c => XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(dados[c]), c.substring(0,30)));
            XLSX.writeFile(wb, "Equipe_Informa_2025.xlsx");
        };

        // --- ADMIN: ACESSOS ---

        window.salvarUsuarioSistema = async () => {
            const u = { 
                usuario: document.getElementById('accUser').value.trim(), 
                senha: document.getElementById('accPass').value.trim(), 
                nivel: document.getElementById('accNivel').value, 
                status: "Ativo",
                categoriasPermitidas: Array.from(document.querySelectorAll('input[name="catPermissao"]:checked')).map(c => c.value) 
            };
            if(!u.usuario || !u.senha) return alert("Usuário e Senha são necessários!");
            await addDoc(collection(db, "usuarios"), u); 
            registrarLog(`Novo acesso criado: ${u.usuario}`);
            carregarLogins();
        };

        window.carregarLogins = async () => {
            const snap = await getDocs(collection(db, "usuarios"));
            const lista = document.getElementById('listaAcessos'); 
            lista.innerHTML = "";
            snap.forEach(d => { 
                const u = d.data(); 
                lista.innerHTML += `
                    <div class="flex justify-between items-center p-3 border-b bg-white rounded-xl mb-2 shadow-sm text-[10px] font-black uppercase">
                        <span>${u.usuario} <span class="text-blue-500 ml-2">[${u.nivel}]</span></span>
                        <button onclick="excluirAcesso('${d.id}')" class="text-red-400 hover:text-red-600 transition">Remover</button>
                    </div>`; 
            });
        };

        window.excluirAcesso = async (id) => {
            if(confirm("Deseja revogar este acesso ao sistema?")) {
                await deleteDoc(doc(db, "usuarios", id));
                carregarLogins();
                registrarLog("Acesso revogado");
            }
        };

        window.carregarLogs = async () => {
            const snap = await getDocs(query(collection(db, "logs"), orderBy("data", "desc")));
            const lista = document.getElementById('listaLogs');
            lista.innerHTML = "";
            snap.forEach(d => {
                const l = d.data();
                const dataStr = l.data ? l.data.toDate().toLocaleString('pt-BR') : "...";
                lista.innerHTML += `<div>[${dataStr}] ${l.usuario}: ${l.acao}</div>`;
            });
        };

        document.getElementById('adminGear').onclick = () => { 
            document.getElementById('modalAdmin').classList.add('open'); 
            switchTab('usuarios');
        };

        window.fecharAdmin = () => document.getElementById('modalAdmin').classList.remove('open');

    </script>
</body>
</html>
