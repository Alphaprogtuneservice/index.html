<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ALPHA PRO | GESTÃO</title>
    
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/1162/1162456.png">

    <style>
        :root {
            --bg: #0f172a; --card: #1e293b; --accent: #3b82f6; --gold: #fbbf24;
            --text: #f8fafc; --text-dim: #94a3b8; --success: #22c55e; --danger: #ef4444;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, sans-serif; }
        body { background: var(--bg); color: var(--text); padding: 20px; padding-bottom: 100px; }
        
        .tabs { display: flex; gap: 8px; position: sticky; top: 0; background: var(--bg); padding: 10px 0; z-index: 100; }
        .tab-btn { flex: 1; padding: 12px; border-radius: 12px; border: none; background: var(--card); color: white; font-weight: bold; font-size: 0.8rem; }
        .tab-btn.active { background: var(--accent); }

        .section { display: none; animation: fadeIn 0.3s; }
        .section.active { display: block; }

        .search-box { width: 100%; padding: 15px; border-radius: 12px; border: 1px solid #334155; background: #020617; color: white; margin-bottom: 15px; font-size: 1rem; }
        
        .db-card { background: var(--card); padding: 15px; border-radius: 12px; margin-bottom: 10px; border-left: 4px solid var(--accent); }
        .db-car { font-weight: bold; color: var(--accent); }
        .db-ecu { font-family: monospace; font-size: 0.85rem; color: var(--gold); margin: 5px 0; }
        
        .btn-action { width: 100%; padding: 15px; border: none; border-radius: 12px; font-weight: bold; margin-bottom: 10px; cursor: pointer; color: white; }
        .btn-save { background: var(--success); }
        .btn-wa { background: #25D366; }
        
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body>

    <div class="tabs">
        <button class="tab-btn active" onclick="showTab(event, 'new')">REGISTO</button>
        <button class="tab-btn" onclick="showTab(event, 'db')">ECU DB</button>
        <button class="tab-btn" onclick="showTab(event, 'settings')">BACKUP</button>
    </div>

    <div id="new" class="section active">
        <div style="background:var(--card); padding:20px; border-radius:15px;">
            <input type="text" id="p-plate" class="search-box" placeholder="Matrícula">
            <input type="text" id="p-car" class="search-box" placeholder="Carro (Ex: BMW 320d)">
            <button class="btn-action btn-save" onclick="saveJob()">FINALIZAR TRABALHO</button>
        </div>
        <div id="historyList" style="margin-top:20px"></div>
    </div>

    <div id="db" class="section">
        <input type="text" id="dbSearch" class="search-box" placeholder="Pesquisar ECU..." onkeyup="searchECU()">
        <div id="dbResults"></div>
    </div>

    <div id="settings" class="section">
        <div class="db-card">
            <h3>Segurança de Dados</h3>
            <p style="color:var(--text-dim); font-size: 0.8rem; margin: 10px 0;">Como esta app é gratuita, os dados ficam no teu iPhone. Envia um backup para o teu WhatsApp para estares seguro.</p>
            <button class="btn-action btn-wa" onclick="sendBackup()">ENVIAR BACKUP P/ WHATSAPP 📲</button>
            <button class="btn-action" style="background:var(--danger)" onclick="clearAll()">LIMPAR TUDO (CUIDADO)</button>
        </div>
    </div>

<script>
    // Database Inicial
    const staticDB = [
        { m: "VW Golf 7 1.6 TDI", e: "PCR 2.1", p: "OBD/Bench" },
        { m: "BMW F30 320d", e: "EDC17C50", p: "OBD/Bench" },
        { m: "Peugeot 1.5 BHDi", e: "MD1CS003", p: "Bench" }
    ];

    function showTab(evt, id) {
        document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        evt.currentTarget.classList.add('active');
        if(id === 'new') loadHistory();
    }

    function saveJob() {
        const p = document.getElementById('p-plate').value;
        const c = document.getElementById('p-car').value;
        if(!p || !c) return alert("Preenche os dados!");

        const job = { p: p.toUpperCase(), c: c, d: new Date().toLocaleDateString() };
        const jobs = JSON.parse(localStorage.getItem('alpha_jobs') || '[]');
        jobs.unshift(job);
        localStorage.setItem('alpha_jobs', JSON.stringify(jobs));
        
        document.getElementById('p-plate').value = "";
        document.getElementById('p-car').value = "";
        loadHistory();
        alert("Serviço Guardado!");
    }

    function loadHistory() {
        const list = document.getElementById('historyList');
        const jobs = JSON.parse(localStorage.getItem('alpha_jobs') || '[]');
        list.innerHTML = "<h3>Últimos Trabalhos:</h3>" + jobs.map(j => `
            <div class="db-card"><b>${j.p}</b> - ${j.c}<br><small>${j.d}</small></div>
        `).join('');
    }

    function searchECU() {
        const q = document.getElementById('dbSearch').value.toLowerCase();
        const res = document.getElementById('dbResults');
        res.innerHTML = staticDB.filter(i => i.m.toLowerCase().includes(q) || i.e.toLowerCase().includes(q)).map(i => `
            <div class="db-card"><div class="db-car">${i.m}</div><div class="db-ecu">ECU: ${i.e}</div></div>
        `).join('');
    }

    // FUNÇÃO DE BACKUP MÁGICA
    function sendBackup() {
        const jobs = localStorage.getItem('alpha_jobs') || '[]';
        const msg = encodeURIComponent(`*BACKUP ALPHA PRO*\nData: ${new Date().toLocaleString()}\n\nDados:\n${jobs}`);
        window.open(`https://wa.me/?text=${msg}`, '_blank');
    }

    function clearAll() {
        if(confirm("Desejas apagar todos os registos?")) {
            localStorage.clear();
            location.reload();
        }
    }

    loadHistory();
    searchECU();
</script>
</body>
</html>
