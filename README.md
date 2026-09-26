
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador de Interceptação Interestelar: 2I e 3I</title>
    <style>
        :root {
            --bg-color: #05050a;
            --panel-bg: rgba(15, 15, 30, 0.85);
            --border-color: #1e1e3f;
            --text-color: #e0e0ff;
            --borisov-color: #00f0ff;
            --atlas-color: #39ff14;
            --ship-color: #ff00ff;
        }

        body {
            margin: 0;
            padding: 20px;
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow-x: hidden;
        }

        h1 {
            font-size: 24px;
            margin-bottom: 5px;
            text-transform: uppercase;
            letter-spacing: 2px;
            text-shadow: 0 0 10px rgba(0, 240, 255, 0.5);
            text-align: center;
        }

        p.subtitle {
            margin: 0 0 20px 0;
            color: #8a8ab0;
            font-size: 14px;
            text-align: center;
        }

        .container {
            display: flex;
            gap: 20px;
            max-width: 1200px;
            width: 100%;
            justify-content: center;
            flex-wrap: wrap;
        }

        .sim-box {
            background: var(--panel-bg);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 15px;
            display: flex;
            flex-direction: column;
            align-items: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            width: 540px;
            box-sizing: border-box;
        }

        h2 {
            font-size: 18px;
            margin: 0 0 15px 0;
            align-self: flex-start;
            border-bottom: 2px solid var(--border-color);
            width: 100%;
            padding-bottom: 5px;
        }

        .borisov-title { color: var(--borisov-color); }
        .atlas-title { color: var(--atlas-color); }

        canvas {
            background-color: #020205;
            border: 1px solid #111122;
            border-radius: 4px;
            cursor: crosshair;
        }

        .telemetry {
            margin-top: 15px;
            width: 100%;
            font-family: 'Courier New', Courier, monospace;
            font-size: 12px;
            background: rgba(0, 0, 0, 0.6);
            padding: 10px;
            border-radius: 4px;
            box-sizing: border-box;
            border-left: 3px solid var(--border-color);
        }

        .telemetry div {
            margin: 4px 0;
            display: flex;
            justify-content: space-between;
        }

        .controls {
            margin-top: 20px;
            display: flex;
            gap: 15px;
        }

        button {
            background: #111125;
            border: 1px solid #333366;
            color: var(--text-color);
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s ease;
        }

        button:hover {
            background: #1e1e3f;
            border-color: #00f0ff;
            box-shadow: 0 0 10px rgba(0, 240, 255, 0.3);
        }
    </style>
</head>
<body>

    <h1>Navegação Vetorial Interestelar</h1>
    <p class="subtitle">Simulação Matemática Analítica de Trajetórias Hiperbólicas de Escape</p>

    <div class="container">
        <!-- SIMULAÇÃO 1: BORISOV -->
        <div class="sim-box">
            <h2 class="borisov-title">Simulação 1: Interceptação 2I/Borisov</h2>
            <canvas id="canvasBorisov" width="500" height="400"></canvas>
            <div class="telemetry" id="telBorisov">
                <div><span>Status da Missão:</span><span id="stBorisov" style="color:#ff00ff">Nave em Curso</span></div>
                <div><span>Distância Relativa:</span><span id="distBorisov">0.00 UA</span></div>
                <div><span>Excentricidade (e):</span><span>1.50</span></div>
                <div><span>Periélio (q):</span><span>2.00 UA</span></div>
                <div><span>Velocidade Infinita (v_inf):</span><span>32.2 km/s</span></div>
            </div>
        </div>

        <!-- SIMULAÇÃO 2: ATLAS -->
        <div class="sim-box">
            <h2 class="atlas-title">Simulação 2: Interceptação Relativística 3I/ATLAS</h2>
            <canvas id="canvasAtlas" width="500" height="400"></canvas>
            <div class="telemetry" id="telAtlas">
                <div><span>Status da Missão:</span><span id="stAtlas" style="color:#ff00ff">Emparelhamento Vetorial</span></div>
                <div><span>Distância Relativa:</span><span id="distAtlas">0.00 UA</span></div>
                <div><span>Excentricidade (e):</span><span>6.10</span></div>
                <div><span>Periélio (q):</span><span>1.40 UA</span></div>
                <div><span>Velocidade Infinita (v_inf):</span><span>58.0 km/s</span></div>
            </div>
        </div>
    </div>

    <div class="controls">
        <button onclick="reiniciarSimulacoes()">Reiniciar Trajetórias</button>
    </div>

    <script>
        // Configurações Globais dos Modelos Matemáticos
        const CONFIG = {
            borisov: {
                canvasId: 'canvasBorisov',
                q: 2.0,       // Periélio em UA
                e: 1.5,       // Excentricidade
                rotacao: 35 * Math.PI / 180, // Ângulo galáctico de entrada
                corObjeto: '#00f0ff',
                escala: 35,   // Pixels por UA
                naveOrigemX: 1.0, // Partindo da Terra (1 UA)
                naveOrigemY: 0.0,
                idxEncontro: 0.62 // Fração do tempo da órbita para interceptar
            },
            atlas: {
                canvasId: 'canvasAtlas',
                q: 1.4,
                e: 6.1,
                rotacao: -60 * Math.PI / 180,
                corObjeto: '#39ff14',
                escala: 25,
                naveOrigemX: 0.0, // Partindo de órbita interna de alta energia
                naveOrigemY: -1.0,
                idxEncontro: 0.54
            }
        };

        // Estado das animações
        let passoGlobal = 0;
        const passosTotais = 600;
        let animacaoId = null;

        // Função Matemática Principal: Equação de Kepler para Hipérboles
        function gerarPontosHiperbole(config) {
            const pontos = [];
            const limiteAssintotico = Math.acos(-1.0 / config.e);
            const margem = 0.08; // Margem de segurança para evitar assíntota infinita
            
            const nuMin = -limiteAssintotico + margem;
            const nuMax = limiteAssintotico - margem;

            for (let i = 0; i <= passosTotais; i++) {
                const t = i / passosTotais;
                const nu = nuMin + t * (nuMax - nuMin); // Anomalia verdadeira
                
                // Raio orbital (Equação cônica geral)
                const r = config.q * (1.0 + config.e) / (1.0 + config.e * Math.cos(nu));
                
                // Conversão coordenadas polares para cartesianas com matriz de rotação
                const xBase = r * Math.cos(nu);
                const yBase = r * Math.sin(nu);
                
                const x = xBase * Math.cos(config.rotacao) - yBase * Math.sin(config.rotacao);
                const y = xBase * Math.sin(config.rotacao) + yBase * Math.cos(config.rotacao);
                
                pontos.push({x: x, y: y});
            }
            return pontos;
        }

        // Pré-computação das malhas vetoriais de órbita
        const orbitaBorisov = gerarPontosHiperbole(CONFIG.borisov);
        const orbitaAtlas = gerarPontosHiperbole(CONFIG.atlas);

        function desenharCena(canvasId, config, pontosOrbita, passoAtual) {
            const canvas = document.getElementById(canvasId);
            const ctx = canvas.getContext('2d');
            const W = canvas.width;
            const H = canvas.height;
            
            // Origem (Sol) centralizada no Canvas
            const centerX = W / 2 - 20;
            const centerY = H / 2 + 20;
            const escala = config.escala;

            ctx.clearRect(0, 0, W, H);

            // Converter coordenadas espaciais para pixels na tela
            function paraPixel(x, y) {
                return {
                    x: centerX + x * escala,
                    y: centerY - y * escala // Inverte eixo Y para padrão cartesiano
                };
            }

            // 1. Grade Espacial de Fundo (Grid de UA)
            ctx.strokeStyle = '#101025';
            ctx.lineWidth = 1;
            for (let x = 0; x < W; x += escala) {
                ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, H); ctx.stroke();
            }
            for (let y = 0; y < H; y += escala) {
                ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke();
            }

            // 2. Renderizar Órbita Terrestre (1 UA) e Sol (Origem)
            const ptSol = paraPixel(0, 0);
            
            // Órbita da Terra
            ctx.strokeStyle = 'rgba(0, 150, 255, 0.15)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.arc(ptSol.x, ptSol.y, 1.0 * escala, 0, Math.PI * 2);
            ctx.stroke();

            // Sol
            ctx.shadowBlur = 15;
            ctx.shadowColor = '#ffcc00';
            ctx.fillStyle = '#ffcc00';
            ctx.beginPath();
            ctx.arc(ptSol.x, ptSol.y, 6, 0, Math.PI * 2);
            ctx.fill();
            ctx.shadowBlur = 0;

            // 3. Renderizar Trajetória Teórica do Objeto Interestelar
            ctx.beginPath();
            ctx.strokeStyle = config.corObjeto + '55'; // Com transparência
            ctx.lineWidth = 1.5;
            ctx.setLineDash([4, 4]);
            
            for (let i = 0; i < pontosOrbita.length; i++) {
                const pt = paraPixel(pontosOrbita[i].x, pontosOrbita[i].y);
                if (i === 0) ctx.moveTo(pt.x, pt.y);
                else ctx.lineTo(pt.x, pt.y);
            }
            ctx.stroke();
            ctx.setLineDash([]);

            // 4. Computar Posições Atuais da Nave e do Objeto
            const idxAlvo = Math.floor(pontosOrbita.length * config.idxEncontro);
            const pontoAlvo = pontosOrbita[idxAlvo];
            const ptAlvoPixel = paraPixel(pontoAlvo.x, pontoAlvo.y);
            const ptNaveInicio = paraPixel(config.naveOrigemX, config.naveOrigemY);

            // Trajetória planejada de interceptação da Nave
            ctx.beginPath();
            ctx.strokeStyle = 'rgba(255, 0, 255, 0.3)';
            ctx.lineWidth = 1;
            ctx.setLineDash([2, 2]);
            ctx.moveTo(ptNaveInicio.x, ptNaveInicio.y);
            ctx.lineTo(ptAlvoPixel.x, ptAlvoPixel.y);
            ctx.stroke();
            ctx.setLineDash([]);

            // Objeto em movimento
            const idxObjeto = Math.min(passoAtual, pontosOrbita.length - 1);
            const ptObjetoAtual = pontosOrbita[idxObjeto];
            const ptObjetoPixel = paraPixel(ptObjetoAtual.x, ptObjetoAtual.y);

            // Cálculo da posição da nave
            let ptNaveAtual;
            let posNaveCoord = { x: 0, y: 0 };

            if (passoAtual <= idxAlvo) {
                const tNave = passoAtual / idxAlvo;
                posNaveCoord.x = config.naveOrigemX + (pontoAlvo.x - config.naveOrigemX) * tNave;
                posNaveCoord.y = config.naveOrigemY + (pontoAlvo.y - config.naveOrigemY) * tNave;
                ptNaveAtual = paraPixel(posNaveCoord.x, posNaveCoord.y);
            } else {
                // Após o ponto de interceptação, a nave acompanha o objeto interestelar
                posNaveCoord = ptObjetoAtual;
                ptNaveAtual = ptObjetoPixel;
            }

            // Rastro percorrido pelo Objeto Interestelar
            ctx.beginPath();
            ctx.strokeStyle = config.corObjeto;
            ctx.lineWidth = 2.5;
            const inicioRastro = Math.max(0, idxObjeto - 50);
            for (let i = inicioRastro; i <= idxObjeto; i++) {
                const pt = paraPixel(pontosOrbita[i].x, pontosOrbita[i].y);
                if (i === inicioRastro) ctx.moveTo(pt.x, pt.y);
                else ctx.lineTo(pt.x, pt.y);
            }
            ctx.stroke();

            // Rastro percorrido pela Nave
            ctx.beginPath();
            ctx.strokeStyle = '#ff00ff';
            ctx.lineWidth = 2;
            ctx.moveTo(ptNaveInicio.x, ptNaveInicio.y);
            ctx.lineTo(ptNaveAtual.x, ptNaveAtual.y);
            ctx.stroke();

            // Indicador Visual do Ponto de Interceptação Planejado
            ctx.strokeStyle = 'rgba(255, 0, 255, 0.6)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.arc(ptAlvoPixel.x, ptAlvoPixel.y, 8, 0, Math.PI * 2);
            ctx.stroke();

            // Renderizar Objeto Interestelar
            ctx.shadowBlur = 10;
            ctx.shadowColor = config.corObjeto;
            ctx.fillStyle = config.corObjeto;
            ctx.beginPath();
            ctx.arc(ptObjetoPixel.x, ptObjetoPixel.y, 5, 0, Math.PI * 2);
            ctx.fill();

            // Renderizar Nave
            ctx.shadowBlur = 10;
            ctx.shadowColor = '#ff00ff';
            ctx.fillStyle = '#ff00ff';
            ctx.beginPath();
            ctx.arc(ptNaveAtual.x, ptNaveAtual.y, 4, 0, Math.PI * 2);
            ctx.fill();
            ctx.shadowBlur = 0;

            // Pulso visual ao realizar o encontro
            if (passoAtual >= idxAlvo) {
                const raioPulso = 10 + Math.sin(passoAtual * 0.2) * 4;
                ctx.strokeStyle = '#ffffff';
                ctx.lineWidth = 1.5;
                ctx.beginPath();
                ctx.arc(ptNaveAtual.x, ptNaveAtual.y, raioPulso, 0, Math.PI * 2);
                ctx.stroke();
            }

            // Retornar a distância relativa calculada em UA
            const dx = ptObjetoAtual.x - posNaveCoord.x;
            const dy = ptObjetoAtual.y - posNaveCoord.y;
            return Math.sqrt(dx * dx + dy * dy);
        }

        function atualizarTelemetria(distBorisov, distAtlas) {
            const idxBorisovAlvo = Math.floor(passosTotais * CONFIG.borisov.idxEncontro);
            const idxAtlasAlvo = Math.floor(passosTotais * CONFIG.atlas.idxEncontro);

            // Borisov Telemetria
            document.getElementById('distBorisov').innerText = distBorisov.toFixed(2) + ' UA';
            const stB = document.getElementById('stBorisov');
            if (passoGlobal >= idxBorisovAlvo) {
                stB.innerText = "Interceptado / Acoplado";
                stB.style.color = "#00ff66";
            } else {
                stB.innerText = "Nave em Curso";
                stB.style.color = "#ff00ff";
            }

            // ATLAS Telemetria
            document.getElementById('distAtlas').innerText = distAtlas.toFixed(2) + ' UA';
            const stA = document.getElementById('stAtlas');
            if (passoGlobal >= idxAtlasAlvo) {
                stA.innerText = "Emparelhamento Concluído";
                stA.style.color = "#39ff14";
            } else {
                stA.innerText = "Aceleração Vetorial";
                stA.style.color = "#ff00ff";
            }
        }

        function loopAnimacao() {
            const distB = desenharCena('canvasBorisov', CONFIG.borisov, orbitaBorisov, passoGlobal);
            const distA = desenharCena('canvasAtlas', CONFIG.atlas, orbitaAtlas, passoGlobal);

            atualizarTelemetria(distB, distA);

            if (passoGlobal < passosTotais) {
                passoGlobal++;
            } else {
                passoGlobal = 0; // Loop contínuo da animação
            }

            animacaoId = requestAnimationFrame(loopAnimacao);
        }

        function reiniciarSimulacoes() {
            passoGlobal = 0;
        }

        // Iniciar Simulação
        loopAnimacao();
    </script>
</body>
</html>
