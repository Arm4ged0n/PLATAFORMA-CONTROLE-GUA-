<!DOCTYPE html>
<html lang="pt-PT">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Auditoria Hídrica Sandro</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: black; }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useMemo, useEffect } = React;

        // Ícones simples em SVG para não depender de bibliotecas externas na Vercel
        const IconActivity = () => <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"></polyline></svg>;
        const IconTrash = () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>;
        const IconPlus = () => <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>;

        const App = () => {
            const LEITURA_FECHAMENTO_OFICIAL = 28.282; 
            const TAXA_FIXA_SERVICO = 39.32;
            const VALOR_AGUA_22M3 = 193.78; 
            const PRECO_M3_REAL = VALOR_AGUA_22M3 / 22;

            const [logs, setLogs] = useState(() => {
                const salvo = localStorage.getItem('historico_sandro_v9');
                return salvo ? JSON.parse(salvo) : [
                    { id: 12, valor: 28.488, desc: 'Sábado Noite', timestamp: 1739037600000 },
                    { id: 1, valor: 28.139, desc: 'Domingo Inicial', timestamp: 1738454400000 }
                ];
            });

            const [novoValor, setNovoValor] = useState('');

            useEffect(() => {
                localStorage.setItem('historico_sandro_v9', JSON.stringify(logs));
            }, [logs]);

            const stats = useMemo(() => {
                const atual = logs.length > 0 ? logs[0].valor : LEITURA_FECHAMENTO_OFICIAL;
                const litrosConsumidos = Math.round(Math.max(atual - LEITURA_FECHAMENTO_OFICIAL, 0) * 1000);
                
                let mediaDiaria = 733; // Padrão da sua fatura
                if (logs.length > 1) {
                    const m3NoPeriodo = logs[0].valor - logs[logs.length - 1].valor;
                    const diffMs = logs[0].timestamp - logs[logs.length - 1].timestamp;
                    const dias = Math.max(diffMs / (1000 * 60 * 60 * 24), 0.5);
                    mediaDiaria = Math.round((m3NoPeriodo * 1000) / dias);
                }
                if (mediaDiaria < 100) mediaDiaria = 733;

                const faturaTotal = TAXA_FIXA_SERVICO + ((mediaDiaria * 30 / 1000) * PRECO_M3_REAL);
                return { atual, litrosConsumidos, mediaDiaria, faturaTotal, totalCiclos: Math.floor(litrosConsumidos/100), progressoCiclo: litrosConsumidos % 100 };
            }, [logs]);

            const handleAdd = () => {
                if (!novoValor) return;
                const novoLog = {
                    id: Date.now(),
                    valor: parseFloat(novoValor),
                    desc: `Registado em ${new Date().toLocaleDateString('pt-PT')} às ${new Date().toLocaleTimeString('pt-PT', {hour:'2d', minute:'2d'})}`,
                    timestamp: Date.now()
                };
                setLogs([novoLog, ...logs]);
                setNovoValor('');
            };

            return (
                <div className="min-h-screen p-4 pb-20 max-w-[450px] mx-auto space-y-4">
                    <header className="flex justify-between items-center bg-slate-900/40 p-5 rounded-[2rem] border border-slate-800">
                        <div className="flex items-center gap-3">
                            <div className="p-2 bg-blue-600 rounded-xl"><IconActivity /></div>
                            <div className="text-left">
                                <h1 className="text-lg font-black italic text-white uppercase leading-none">HIDRO SANDRO</h1>
                                <p className="text-[7px] font-bold text-slate-500 uppercase mt-1">Vercel Deployment</p>
                            </div>
                        </div>
                    </header>

                    <div className="bg-gradient-to-br from-red-900/40 to-slate-900/40 border border-red-500/30 p-6 rounded-[2.5rem] text-left">
                        <p className="text-[9px] font-black uppercase text-red-400 mb-2 italic tracking-widest">Estimativa Próxima Conta</p>
                        <p className="text-5xl font-black italic text-white tracking-tighter">R$ {stats.faturaTotal.toFixed(2)}</p>
                        <p className="text-[9px] text-slate-400 mt-2 italic">* Média de {stats.mediaDiaria}L/dia (Baseado na Fatura de 22m³)</p>
                    </div>

                    <div className="bg-slate-900/40 border border-slate-800 p-6 rounded-[2.5rem] text-left">
                        <h2 className="text-[8px] font-black uppercase text-slate-500 mb-4 tracking-widest">Caixas Gastas (100L)</h2>
                        <p className="text-4xl font-black italic text-white mb-4">{stats.totalCiclos} <span className="text-sm opacity-30 uppercase">Vezes</span></p>
                        <div className="w-full h-3 bg-slate-950 rounded-full overflow-hidden border border-white/5">
                            <div className="h-full bg-blue-500 transition-all duration-1000" style={{ width: `${stats.progressoCiclo}%` }}></div>
                        </div>
                    </div>

                    <div className="grid grid-cols-2 gap-3">
                        <div className="bg-slate-900/30 border border-slate-800 p-5 rounded-[2rem] text-left">
                            <p className="text-[8px] font-black text-slate-500 uppercase mb-1">Gasto Real</p>
                            <p className="text-2xl font-black italic text-white">{stats.litrosConsumidos}L</p>
                        </div>
                        <div className="bg-slate-900/30 border border-slate-800 p-5 rounded-[2rem] text-left border-l-4 border-l-green-500">
                            <p className="text-[8px] font-black text-slate-500 uppercase mb-1 text-green-500">Hidrómetro</p>
                            <p className="text-2xl font-black italic text-green-500">{stats.atual.toFixed(3)}</p>
                        </div>
                    </div>

                    <div className="bg-blue-600 p-1 rounded-[2.5rem]">
                        <div className="bg-black p-6 rounded-[2.4rem] space-y-4">
                            <input type="number" step="0.001" value={novoValor} onChange={e => setNovoValor(e.target.value)} 
                                className="w-full bg-slate-900 border border-slate-800 rounded-2xl p-4 text-4xl font-black text-center text-white outline-none focus:border-blue-500" placeholder="28.XXX" />
                            <button onClick={handleAdd} className="w-full bg-blue-600 text-black font-black py-4 rounded-2xl text-[10px] uppercase flex items-center justify-center gap-2">
                                <IconPlus /> Gravar Leitura
                            </button>
                        </div>
                    </div>

                    <div className="space-y-3 pb-10">
                        {logs.slice(0, 10).map(log => (
                            <div key={log.id} className="bg-slate-900/20 border border-slate-800 p-4 rounded-2xl flex justify-between items-center text-left">
                                <div>
                                    <p className="text-[7px] font-black text-slate-600 uppercase mb-1">{log.desc}</p>
                                    <p className="text-xl font-black text-slate-100 italic leading-none">{log.valor.toFixed(3)}</p>
                                </div>
                                <button onClick={() => setLogs(logs.filter(l => l.id !== log.id))} className="text-slate-800 p-2"><IconTrash /></button>
                            </div>
                        ))}
                    </div>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>

