<!DOCTYPE html>
<html lang="pt-pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestão de Equipa - Painel Administrativo</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; }
        
        /* Animação da Gaveta */
        .drawer {
            transform: translateX(100%);
            transition: transform 0.3s ease-in-out;
        }
        .drawer.open {
            transform: translateX(0);
        }
        
        /* Estilização da Tabela */
        .categoria-tag {
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: bold;
            text-transform: uppercase;
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <div class="max-w-6xl mx-auto flex flex-col md:flex-row justify-between items-center mb-10 gap-4">
        <div>
            <h1 class="text-4xl font-black text-gray-900 tracking-tighter uppercase">Equipa <span class="text-blue-600">Pro</span></h1>
            <p class="text-gray-500 font-medium">Gestão de departamentos e membros</p>
        </div>
        <button onclick="abrirDrawer()" class="bg-blue-600 hover:bg-blue-700 text-white px-8 py-4 rounded-2xl font-bold shadow-lg shadow-blue-200 transition-all active:scale-95 flex items-center gap-2">
            <span class="text-2xl">+</span> NOVO MEMBRO
        </button>
    </div>

    <div class="max-w-6xl mx-auto bg-white rounded-3xl shadow-sm border border-gray-100 overflow-hidden">
        <div class="overflow-x-auto">
            <table class="w-full text-left border-collapse">
                <thead>
                    <tr class="bg-gray-50 border-b border-gray-100">
                        <th class="p-6 text-[11px] font-black text-gray-400 uppercase tracking-widest">Membro</th>
                        <th class="p-6 text-[11px] font-black text-gray-400 uppercase tracking-widest">Categoria</th>
                        <th class="p-6 text-[11px] font-black text-gray-400 uppercase tracking-widest">Ano</th>
                        <th class="p-6 text-[11px] font-black text-gray-400 uppercase tracking-widest text-right">Ações</th>
                    </tr>
                </thead>
                <tbody id="tabelaCorpo">
                    </tbody>
            </table>
        </div>
    </div>

    <div id="drawerOverlay" onclick="fecharDrawer()" class="fixed inset-0 bg-black/40 hidden z-40 backdrop-blur-sm"></div>
    
    <div id="drawer" class="drawer fixed top-0 right-0 h-full w-full max-w-md bg-white shadow-2xl z-50 p-8 flex flex-col">
        <div class="flex justify-between items-center mb-8">
            <div>
                <h2 id="drawerTitulo" class="text-2xl font-black text-gray-800 uppercase tracking-tighter">Membro da Equipa</h2>
                <p class="text-sm text-gray-400">Preencha os dados do departamento</p>
            </div>
            <button onclick="fecharDrawer()" class="text-gray-400 hover:text-red-500 text-3xl transition-colors">&times;</button>
        </div>
        
        <input type="hidden" id="editId">
        
        <div class="space-y-5 overflow-y-auto pr-2 flex-1">
            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Nome Completo</label>
                <input id="mNome" placeholder="Ex: João Silva" class="w-full p-4 border border-gray-200 rounded-2xl bg-gray-50 outline-blue-600 transition-all">
            </div>

            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Categoria / Departamento</label>
                <select id="mCategoria" class="w-full p-4 border border-gray-200 rounded-2xl bg-gray-50 outline-blue-600 appearance-none cursor-pointer">
                    <option value="">Selecione uma opção...</option>
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

            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">E-mail de Contacto</label>
                <input id="mEmail" type="email" placeholder="exemplo@email.com" class="w-full p-4 border border-gray-200 rounded-2xl bg-gray-50 outline-blue-600">
            </div>

            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Ano de Entrada</label>
                <input id="mAno" type="number" placeholder="Ex: 2024" class="w-full p-4 border border-gray-200 rounded-2xl bg-gray-50 outline-blue-600">
            </div>
        </div>

        <button onclick="salvarMembro()" class="w-full bg-blue-600 text-white py-5 rounded-2xl font-black shadow-xl shadow-blue-100 hover:bg-blue-700 transition-all active:scale-95 mt-6">
            SALVAR REGISTO
        </button>
    </div>

    <script>
        // Banco de Dados Local (LocalStorage)
        let membros = JSON.parse(localStorage.getItem('equipa_db')) || [];

        function abrirDrawer(id = null) {
            document.getElementById('drawerOverlay').classList.remove('hidden');
            document.getElementById('drawer').classList.add('open');
            
            if(id) {
                const m = membros.find(item => item.id == id);
                document.getElementById('drawerTitulo').innerText = "Editar Membro";
                document.getElementById('editId').value = m.id;
                document.getElementById('mNome').value = m.nome;
                document.getElementById('mCategoria').value = m.categoria;
                document.getElementById('mEmail').value = m.email;
                document.getElementById('mAno').value = m.ano;
            } else {
                limparFormulario();
            }
        }

        function fecharDrawer() {
            document.getElementById('drawerOverlay').classList.add('hidden');
            document.getElementById('drawer').classList.remove('open');
        }

        function limparFormulario() {
            document.getElementById('drawerTitulo').innerText = "Novo Membro";
            document.getElementById('editId').value = "";
            document.getElementById('mNome').value = "";
            document.getElementById('mCategoria').value = "";
            document.getElementById('mEmail').value = "";
            document.getElementById('mAno').value = "";
        }

        function salvarMembro() {
            const id = document.getElementById('editId').value;
            const nome = document.getElementById('mNome').value;
            const categoria = document.getElementById('mCategoria').value;
            const email = document.getElementById('mEmail').value;
            const ano = document.getElementById('mAno').value;

            if(!nome || !categoria) return alert("Preencha o Nome e a Categoria!");

            if(id) {
                // Editar existente
                const index = membros.findIndex(m => m.id == id);
                membros[index] = { id: parseInt(id), nome, categoria, email, ano };
            } else {
                // Novo registo
                membros.push({ id: Date.now(), nome, categoria, email, ano });
            }

            localStorage.setItem('equipa_db', JSON.stringify(membros));
            atualizarTabela();
            fecharDrawer();
        }

        function excluirMembro(id) {
            if(confirm("Deseja apagar este membro?")) {
                membros = membros.filter(m => m.id !== id);
                localStorage.setItem('equipa_db', JSON.stringify(membros));
                atualizarTabela();
            }
        }

        function atualizarTabela() {
            const corpo = document.getElementById('tabelaCorpo');
            corpo.innerHTML = "";

            membros.forEach(m => {
                const tr = document.createElement('tr');
                tr.className = "border-b border-gray-50 hover:bg-gray-50/50 transition-colors";
                tr.innerHTML = `
                    <td class="p-6">
                        <div class="font-bold text-gray-800">${m.nome}</div>
                        <div class="text-xs text-gray-400">${m.email || 'Sem email'}</div>
                    </td>
                    <td class="p-6">
                        <span class="categoria-tag bg-blue-50 text-blue-600">${m.categoria}</span>
                    </td>
                    <td class="p-6 font-medium text-gray-500">${m.ano}</td>
                    <td class="p-6 text-right">
                        <button onclick="abrirDrawer(${m.id})" class="text-blue-500 hover:text-blue-700 font-bold mr-4">Editar</button>
                        <button onclick="excluirMembro(${m.id})" class="text-red-400 hover:text-red-600 font-bold">Apagar</button>
                    </td>
                `;
                corpo.appendChild(tr);
            });
        }

        // Inicializar tabela ao carregar página
        atualizarTabela();
    </script>
</body>
</html>
