
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel Mestre - Centros, 3 Dias e Médias</title>
    <style>
        body {
            background-color: #0b0e11;
            color: #eaecef;
            font-family: monospace;
            padding: 10px;
            margin: 0;
        }
        h2 { text-align: center; color: #f0b90b; font-size: 16px; margin-bottom: 10px; }
        
        .botoes-container {
            display: flex;
            flex-wrap: wrap;
            gap: 5px;
            justify-content: center;
            margin-bottom: 15px;
        }
        .btn-tempo {
            background-color: #1e2329;
            color: #eaecef;
            border: 1px solid #474d57;
            padding: 8px 10px;
            font-family: monospace;
            font-weight: bold;
            cursor: pointer;
            border-radius: 4px;
            font-size: 12px;
        }
        .btn-tempo.ativo {
            background-color: #f0b90b;
            color: #000;
            border-color: #f0b90b;
        }

        .alerta-box {
            background-color: #1e2329;
            border: 2px solid #f0b90b;
            padding: 10px;
            text-align: center;
            font-size: 14px;
            font-weight: bold;
            margin-bottom: 10px;
            border-radius: 5px;
        }
        .painel-info {
            background-color: #181a20;
            border: 1px solid #2b313a;
            padding: 12px;
            border-radius: 6px;
            font-size: 13px;
            margin-bottom: 15px;
        }
        .linha-info {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
            border-bottom: 1px solid #2b313a;
            padding-bottom: 6px;
            align-items: center;
        }
        .secao-titulo {
            color: #f0b90b;
            margin-top: 15px;
            margin-bottom: 6px;
            font-weight: bold;
        }
        .medias-grid {
            color: #848e9c;
            font-size: 11px;
            line-height: 1.8;
        }
        .badge-compra {
            color: #0ecb81;
            font-weight: bold;
        }
        .badge-venda {
            color: #f6465d;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <h2>BTUSD - CENTROS, 3 DIAS E MÉDIAS</h2>

    <div class="botoes-container">
        <button class="btn-tempo ativo" onclick="mudarTempo('1m', this)">1m</button>
        <button class="btn-tempo" onclick="mudarTempo('2m', this)">2m</button>
        <button class="btn-tempo" onclick="mudarTempo('3m', this)">3m</button>
        <button class="btn-tempo" onclick="mudarTempo('4m', this)">4m</button>
        <button class="btn-tempo" onclick="mudarTempo('5m', this)">5m</button>
        <button class="btn-tempo" onclick="mudarTempo('6m', this)">6m</button>
        <button class="btn-tempo" onclick="mudarTempo('10m', this)">10m</button>
        <button class="btn-tempo" onclick="mudarTempo('12m', this)">12m</button>
        <button class="btn-tempo" onclick="mudarTempo('20m', this)">20m</button>
        <button class="btn-tempo" onclick="mudarTempo('30m', this)">30m</button>
        <button class="btn-tempo" onclick="mudarTempo('1h', this)">1h</button>
        <button class="btn-tempo" onclick="mudarTempo('2h', this)">2h</button>
        <button class="btn-tempo" onclick="mudarTempo('3h', this)">3h</button>
    </div>

    <div id="status-sinal" class="alerta-box" style="color: #f0b90b;">
        🔍 CARREGANDO...
    </div>

    <div id="painel-detalhes" class="painel-info">
        Selecione um tempo acima...
    </div>

<script>
let tempoAtualBinance = '1m';
const periodosMa = [19, 38, 97, 191, 383, 575, 979];
let dadosGlobais = [];
let dadosDiarios = [];

function mudarTempo(intervalo, elemento) {
    tempoAtualBinance = intervalo;
    document.querySelectorAll('.btn-tempo').forEach(b => b.classList.remove('ativo'));
    elemento.classList.add('ativo');
    carregarDados();
}

// Configuração para suportar nativos e customizados (como 2m, 4m, 6m, 12m, 2h, 3h)
function obterConfiguracaoBinance(tempo) {
    switch(tempo) {
        case '1m':  return { baseApi: '1m', fator: 1 };
        case '2m':  return { baseApi: '1m', fator: 2 };
        case '3m':  return { baseApi: '3m', fator: 1 };
        case '4m':  return { baseApi: '1m', fator: 4 };
        case '5m':  return { baseApi: '5m', fator: 1 };
        case '6m':  return { baseApi: '1m', fator: 6 };
        case '10m': return { baseApi: '5m', fator: 2 };
        case '12m': return { baseApi: '3m', fator: 4 };
        case '20m': return { baseApi: '10m', fator: 2 };
        case '30m': return { baseApi: '30m', fator: 1 };
        case '1h':  return { baseApi: '1h', fator: 1 };
        case '2h':  return { baseApi: '1h', fator: 2 };
        case '3h':  return { baseApi: '1h', fator: 3 };
        default:    return { baseApi: '1m', fator: 1 };
    }
}

function agruparVelas(velasOriginal, fator) {
    if (fator <= 1) return velasOriginal;
    let velasAgrupadas = [];
    for (let i = 0; i < velasOriginal.length; i += fator) {
        let bloco = velasOriginal.slice(i, i + fator);
        if (bloco.length === 0) continue;
        
        let openTime = bloco[0][0];
        let open = bloco[0][1];
        let high = -Infinity;
        let low = Infinity;
        let close = bloco[bloco.length - 1][4];
        let volume = 0;
        
        bloco.forEach(v => {
            let h = parseFloat(v[2]);
            let l = parseFloat(v[3]);
            if (h > high) high = h;
            if (l < low) low = l;
            volume += parseFloat(v[5]);
        });
        
        velasAgrupadas.push([openTime, open, high.toString(), low.toString(), close, volume.toString()]);
    }
    return velasAgrupadas;
}

function calcularCentro(velas) {
    if (!velas || velas.length === 0) return 0;
    let ultimaVela = velas[velas.length - 1];
    let maxima = parseFloat(ultimaVela[2]);
    let minima = parseFloat(ultimaVela[3]);
    return (maxima + minima) / 2;
}

function calcularMedia(velas, periodo) {
    if (velas.length < periodo) return null;
    let soma = 0;
    for (let i = velas.length - periodo; i < velas.length; i++) {
        soma += parseFloat(velas[i][4]);
    }
    return soma / periodo;
}

function valorParaHora(valor) {
    let numStr = valor.toFixed(2);
    let partes = numStr.split('.');
    let parteInteira = partes[0];
    let centavos = partes[1] || '00';

    if (parteInteira.length >= 4) {
        let horasBrutas = parseInt(parteInteira.substring(parteInteira.length - 4, parteInteira.length - 2));
        let minutos = parteInteira.substring(parteInteira.length - 2);
        
        let horasAjustadas = horasBrutas % 24;
        let hFormatado = String(horasAjustadas).padStart(2, '0');
        let mFormatado = parseInt(minutos) > 59 ? "59" : minutos;
        let sFormatado = parseInt(centavos) > 59 ? "59" : centavos;

        return `${hFormatado}:${mFormatado}:${sFormatado}`;
    }
    return "00:00:00";
}

async function carregarDados() {
    try {
        let config = obterConfiguracaoBinance(tempoAtualBinance);
        
        // Pega dados do tempo selecionado
        let res = await fetch(`https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=${config.baseApi}&limit=1000`);
        let dadosBrutos = await res.json();
        dadosGlobais = agruparVelas(dadosBrutos, config.fator);

        // Pega dados diários para as aberturas dos últimos 3 dias
        let resDiario = await fetch(`https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=1d&limit=4`);
        dadosDiarios = await resDiario.json();

        atualizarTela();
    } catch (e) {
        document.getElementById("status-sinal").innerText = "❌ ERRO AO CONECTAR NA BINANCE";
    }
}

function atualizarTela() {
    if (!dadosGlobais.length) return;

    let centro = calcularCentro(dadosGlobais);
    let precoAtual = parseFloat(dadosGlobais[dadosGlobais.length - 1][4]);

    let sinalTexto = "";
    let corSinal = "";

    if (precoAtual >= centro) {
        sinalTexto = `🚀 [${tempoAtualBinance.toUpperCase()}] PREÇO ACIMA DO CENTRO -> COMPRA FORTE!`;
        corSinal = "#0ecb81";
    } else {
        sinalTexto = `📉 [${tempoAtualBinance.toUpperCase()}] PREÇO ABAIXO DO CENTRO -> VENDA FORTE!`;
        corSinal = "#f6465d";
    }

    let caixaSinal = document.getElementById("status-sinal");
    caixaSinal.innerText = sinalTexto;
    caixaSinal.style.color = corSinal;
    caixaSinal.style.borderColor = corSinal;

    let precoHora = valorParaHora(precoAtual);
    let centroHora = valorParaHora(centro);

    // Extração das aberturas dos 3 dias (se houver dados suficientes)
    let aberturasHtml = "";
    if (dadosDiarios.length >= 3) {
        // Índice atual e anteriores no array diário
        let diaAtualVal = parseFloat(dadosDiarios[dadosDiarios.length - 1][1]);
        let diaAnteriorVal = parseFloat(dadosDiarios[dadosDiarios.length - 2][1]);
        let segundoDiaVal = parseFloat(dadosDiarios[dadosDiarios.length - 3][1]);

        aberturasHtml = `
            <div class="linha-info"><span>Abertura Hoje (Atual):</span> <b>$ ${diaAtualVal.toFixed(2)} &nbsp;|&nbsp; <span style="color: #f0b90b;">🕒 ${valorParaHora(diaAtualVal)}</span></b></div>
            <div class="linha-info"><span>Abertura Ontem (1º Dia):</span> <b>$ ${diaAnteriorVal.toFixed(2)} &nbsp;|&nbsp; <span style="color: #f0b90b;">🕒 ${valorParaHora(diaAnteriorVal)}</span></b></div>
            <div class="linha-info"><span>Abertura Anteontem (2º Dia):</span> <b>$ ${segundoDiaVal.toFixed(2)} &nbsp;|&nbsp; <span style="color: #f0b90b;">🕒 ${valorParaHora(segundoDiaVal)}</span></b></div>
        `;
    }

    // Cálculo de todas as Médias Móveis
    let mediasHtml = "";
    periodosMa.forEach(p => {
        let maVal = calcularMedia(dadosGlobais, p);
        if (maVal) {
            let maHora = valorParaHora(maVal);
            let statusAlerta = (precoAtual >= maVal) ? 
                `<span class="badge-compra">🟢 [COMPRA]</span>` : 
                `<span class="badge-venda">🔴 [VENDA]</span>`;

            mediasHtml += `MA ${p}: <b>$ ${maVal.toFixed(2)}</b> &nbsp;|&nbsp; <span style="color: #f0b90b;">🕒 ${maHora}</span> &nbsp;→&nbsp; ${statusAlerta}<br>`;
        } else {
            mediasHtml += `MA ${p}: <i>Dados insuficientes</i><br>`;
        }
    });

    document.getElementById("painel-detalhes").innerHTML = `
        <div class="linha-info">
            <span>Preço do BTC:</span> <b>$ ${precoAtual.toFixed(2)} &nbsp;|&nbsp; <span style="color: #f0b90b;">🕒 ${precoHora}</span></b>
        </div>
        <div class="linha-info">
            <span>Centro da Vela (${tempoAtualBinance.toUpperCase()}):</span> <b>$ ${centro.toFixed(2)} &nbsp;|&nbsp; <span style="color: #0ecb81;">🕒 ${centroHora}</span></b>
        </div>
        <div class="linha-info">
            <span>Contagem Regressiva:</span> <b id="timer-txt" style="color: #f0b90b;">--:--</b>
        </div>
        
        <div class="secao-titulo">📅 ABERTURAS DOS 3 DIAS:</div>
        ${aberturasHtml}

        <div class="secao-titulo">⏱️ MÉDIAS MESTRES (VALOR E HORA LADO A LADO):</div>
        <div class="medias-grid">${mediasHtml}</div>
    `;
    
    atualizarTimer();
}

function atualizarTimer() {
    const agora = new Date();
    let segundos = agora.getSeconds();
    let minutos = agora.getMinutes();
    let horas = agora.getHours();
    
    let restoSegundos = 59 - segundos;
    let sFormatado = String(restoSegundos).padStart(2, '0');

    let tempoTexto = "";
    if (tempoAtualBinance.endsWith('h')) {
        let horasAlvo = parseInt(tempoAtualBinance);
        let restoH = horasAlvo - 1 - (horas % horasAlvo);
        let restoMin = 59 - minutos;
        tempoTexto = `0${restoH}:${String(restoMin).padStart(2, '0')}:${sFormatado}`;
    } else {
        let minutosAlvo = parseInt(tempoAtualBinance);
        let restoMin = minutosAlvo - 1 - (minutos % minutosAlvo);
        if (restoMin < 0) restoMin = 0;
        let mFormatado = String(restoMin).padStart(2, '0');
        tempoTexto = `${mFormatado}:${sFormatado}`;
    }

    let elemTimer = document.getElementById("timer-txt");
    if (elemTimer) {
        elemTimer.innerText = tempoTexto;
    }
}

carregarDados();
setInterval(carregarDados, 10000);
setInterval(atualizarTimer, 1000);
</script>

</body>
</html>
