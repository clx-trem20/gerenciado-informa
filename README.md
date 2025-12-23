<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel Informa - Gestão de Equipe</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.sheetjs.com/xlsx-0.20.0/package/dist/xlsx.full.min.js"></script>
    <style>
        .drawer { transform: translateX(100%); transition: transform 0.3s ease-in-out; }
        .drawer.open { transform: translateX(0); }
        .bg-informa { background-color: #0047ab; }
        .text-informa { color: #0047ab; }
    </style>
</head>
<body class="bg-gray-100 font-sans text-gray-900">

    <div id="login-screen" class="flex items-center justify-center h-screen bg-slate-200">
        <div class="bg-white p-10 rounded-2xl shadow-2xl w-full max-w-md border-t-8 border-blue-700">
            <div class="text-center mb-8">
                <h1 class="text-3xl font-black text-blue-700 uppercase tracking-tighter">Jornal Informa</h1>
                <p class="text-gray-400 font-medium">Painel Administrativo</p>
            </div>
            <div class="space-y-4">
                <div>
                    <label class="block text-xs font-bold text-gray-500 uppercase ml-1 mb-1">Usuário</label>
                    <input type="text" id="login-user" class="w-full p-3 border-2 border-gray-100 rounded-xl focus:border-blue-500 outline-none transition">
                </div>
                <div>
                    <label class="block text-xs font-bold text-gray-500 uppercase ml-1 mb-1">Senha</label>
                    <input type="password" id="login-pass" class="w-full p-3 border-2 border-gray-100 rounded-xl focus:border-blue-500 outline-none transition">
                </div>
                <button onclick="validaLogin()" class="w-full bg-blue-700 text-white py-4 rounded-xl font-bold hover:bg-blue-800 transition shadow-lg transform hover:-translate-y-1">
                    ACESSAR PAINEL
                </button>
            </div>
        </div>
    </div>

    <div id="admin-panel" class="hidden min-h-screen">
        <nav class="bg-white shadow-sm border-b px-8 py-4 flex justify-between items-center sticky top-0 z-40">
            <h2 class="text-xl font-bold text-blue-700">INFORMA <span class="text-gray-300 font-light">| ADMIN</span></h2>
            <div class="flex space-x-3">
                <button onclick="exportarExcelSeparado()" class="bg-gray-100 text-gray-700 px-5 py-2 rounded-lg font-bold hover:bg-gray-200 transition flex items-center">
                    <span class="mr-2">📊</span> Planilha Excel
                </button>
                <button onclick="toggleDrawer()" class="bg-blue-700 text-white px-5 py-2 rounded-lg font-bold hover:bg-blue-800 transition shadow-md">
                    + Novo Participante
                </button>
            </div>
        </nav>

        <main class="p-8">
            <div class="max-w-6xl mx-auto">
                <div class="bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase">Nome Completo</th>
                                <th class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase">E-mail</th>
                                <th class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase">Ingresso</th>
                                <th class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase">Acesso</th>
                                <th class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase">Status</th>
                                <th class="px-6 py-4 text-center text-xs font-bold text-gray-500 uppercase">Ações</th>
                            </tr>
                        </thead>
                        <tbody id="membros-lista" class="divide-y divide-gray-200 bg-white">
                            </tbody>
                    </table>
                </div>
            </div>
        </main>
    </div>

    <div id="drawer-overlay" onclick="toggleDrawer()" class="fixed inset-0 bg-black/50 hidden z-40"></div>
    <div id="drawer" class="drawer fixed top-0 right-0 h-full w-full max-w-sm bg-white shadow-2xl z-50 p-8">
        <div class="flex justify-between items-center mb-10">
            <h2 class="text-2xl font-black text-gray-800">Cadastrar Membro</h2>
            <button onclick="toggleDrawer()" class="text-gray-400 hover:text-red-500 text-3xl">&times;</button>
        </div>
        
        <div class="space-y-6">
            <div>
                <label class="block text-xs font-bold text-gray-400 uppercase mb-1">Nome Completo</label>
                <input type="text" id="new-nome" class="w-full p-3 border-2 border-gray-50 rounded-lg focus:border-blue-500 outline-none bg-gray-50">
            </div>
            <div>
                <label class="block text-xs font-bold text-gray-400 uppercase mb-1">E-mail</label>
                <input type="email" id="new-email" class="w-full p-3 border-2 border-gray-50 rounded-lg focus:border-blue-500 outline-none bg-gray-50">
            </div>
            <div>
                <label class="block text-xs font-bold text-gray-400 uppercase mb-1">Data de Início</label>
                <input type="date" id="new-data-ingresso" class="w-full p-3 border-2 border-gray-50 rounded-lg focus:border-blue-500 outline-none bg-gray-50">
            </div>
            <div>
                <label class="block text-xs font-bold text-gray-400 uppercase mb-1">Tipo de Usuário</label>
                <select id="new-role" class="w-full p-3 border-2 border-gray-50 rounded-lg focus:border-blue-500 outline-none bg-gray-50">
                    <option value="Membro">Membro Comum</option>
                    <option value="Admin">Administrador</option>
                </select>
            </div>
            <button onclick="salvarMembro()" class="w-full bg-blue-700 text-white py-4 rounded-xl font-bold shadow-lg hover:bg-blue-800 transition mt-4">
                SALVAR NO JORNAL
            </button>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, doc, getDoc, collection, addDoc, getDocs, updateDoc, query, orderBy } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

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

        // LOGIN VALIDADO PELO BANCO
        window.validaLogin = async () => {
            const userIn = document.getElementById('login-user').value;
            const passIn = document.getElementById('login-pass').value;

            try {
                const docRef = doc(db, "usuarios_admin", "admin_principal");
                const docSnap = await getDoc(docRef);

                if (docSnap.exists()) {
                    const data = docSnap.data();
                    if (userIn === data.usuario && passIn === data.senha) {
                        document.getElementById('login-screen').classList.add('hidden');
                        document.getElementById('admin-panel').classList.remove('hidden');
                        carregarMembros();
                    } else { alert("Usuário ou senha incorretos."); }
                }
            } catch (e) { alert("Erro ao conectar ao banco."); }
        };

        window.toggleDrawer = () => {
            document.getElementById('drawer').classList.toggle('open');
            document.getElementById('drawer-overlay').classList.toggle('hidden');
        };

        // SALVAR MEMBRO
        window.salvarMembro = async () => {
            const nome = document.getElementById('new-nome').value;
            const email = document.getElementById('new-email').value;
            const dataIngresso = document.getElementById('new-data-ingresso').value;
            const role = document.getElementById('new-role').value;

            if(!nome || !email || !dataIngresso) return alert("Por favor, preencha todos os campos.");

            await addDoc(collection(db, "membros"), {
                nome, email, dataIngresso, role, status: "Ativo"
            });

            alert("Membro adicionado com sucesso!");
            toggleDrawer();
            carregarMembros();
            
            // Limpa campos
            document.getElementById('new-nome').value = "";
            document.getElementById('new-email').value = "";
            document.getElementById('new-data-ingresso').value = "";
        };

        // LISTAR MEMBROS
        window.carregarMembros = async () => {
            const q = query(collection(db, "membros"), orderBy("nome", "asc"));
            const querySnapshot = await getDocs(q);
            const lista = document.getElementById('membros-lista');
            lista.innerHTML = "";

            querySnapshot.forEach((doc) => {
                const m = doc.data();
                const d = m.dataIngresso.split('-').reverse().join('/');
                const inativo = m.status === 'Inativo';
                
                lista.innerHTML += `
                    <tr class="${inativo ? 'bg-red-50' : 'hover:bg-gray-50'} transition">
                        <td class="px-6 py-4 font-bold text-gray-800">${m.nome}</td>
                        <td class="px-6 py-4 text-gray-500 text-sm">${m.email}</td>
                        <td class="px-6 py-4 text-sm text-gray-600">${d}</td>
                        <td class="px-6 py-4"><span class="px-2 py-1 bg-gray-100 rounded text-xs font-bold uppercase">${m.role}</span></td>
                        <td class="px-6 py-4">
                            <span class="flex items-center text-xs font-bold ${inativo ? 'text-red-600' : 'text-green-600'}">
                                <span class="w-2 h-2 rounded-full ${inativo ? 'bg-red-600' : 'bg-green-600'} mr-2"></span>
                                ${m.status.toUpperCase()}
                            </span>
                        </td>
                        <td class="px-6 py-4 text-center space-x-3">
                            <button onclick="desativar('${doc.id}')" title="Desativar" class="text-gray-400 hover:text-red-600 transition">🚫</button>
                            <button onclick="enviarEmail('${m.email}', '${m.nome}')" title="Mandar E-mail" class="text-gray-400 hover:text-blue-600 transition">✉️</button>
                        </td>
                    </tr>
                `;
            });
        };

        window.desativar = async (id) => {
            if(confirm("Deseja marcar este participante como inativo?")) {
                await updateDoc(doc(db, "membros", id), { status: "Inativo" });
                carregarMembros();
            }
        };

        // E-MAIL DE DESPEDIDA MELHORADO
        window.enviarEmail = (email, nome) => {
            const assunto = encodeURIComponent("Foi um prazer ter você no Jornal Informa!");
            const corpo = encodeURIComponent(
                `Olá, ${nome}!\n\n` +
                `Estamos passando para agradecer formalmente por toda a sua dedicação e esforço durante o período em que colaborou conosco.\n\n` +
                `Sua contribuição foi fundamental para o Informa, e foi um verdadeiro prazer ter você na equipe!\n\n` +
                `Desejamos muito sucesso em seus próximos desafios.\n\n` +
                `Com gratidão,\nEquipe Jornal Informa.`
            );
            window.location.href = `mailto:${email}?subject=${assunto}&body=${corpo}`;
        };

        // EXCEL COM ABAS SEPARADAS
        window.exportarExcelSeparado = async () => {
            const querySnapshot = await getDocs(collection(db, "membros"));
            const ativos = [];
            const inativos = [];

            querySnapshot.forEach((doc) => {
                const m = doc.data();
                const item = {
                    "Nome Completo": m.nome,
                    "E-mail": m.email,
                    "Data Ingresso": m.dataIngresso.split('-').reverse().join('/'),
                    "Cargo": m.role
                };
                if (m.status === "Ativo") ativos.push(item);
                else inativos.push(item);
            });

            const wb = XLSX.utils.book_new();
            if (ativos.length > 0) XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(ativos), "Ativos");
            if (inativos.length > 0) XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(inativos), "Inativos");
            
            XLSX.writeFile(wb, "Relatorio_Equipe_Informa.xlsx");
        };
    </script>
</body>
</html>
