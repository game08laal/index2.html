<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>MathMota</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.socket.io/4.5.4/socket.io.min.js"></script>

  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      text-align: center;
      background: linear-gradient(135deg, #6a11cb, #2575fc);
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      overflow: hidden;
    }
     
    h1, h2 { margin-bottom: 20px; }
    .screen {
      display: none;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      width: 100%;
      height: 100%;
      animation: fadeIn 0.5s;
    }
    .active { display: flex; }
    input, select, button {
      font-size: 1.2em;
      padding: 10px;
      margin: 10px;
      border-radius: 8px;
      border: none;
    }
    .options button {
      width: 80%;
      max-width: 400px;
      color: #fff;
      margin: 10px auto;
      transition: background 0.3s;
      cursor: pointer;
    }
    .options button:hover, .options button:active { background: #2ecc71; }
    .selected { background: #e006a2 !important; }
    #timer { font-size: 2.5em; margin: 20px; }
    canvas {
      max-width: 400px;
      max-height: 400px;
      background: #fff;
      border-radius: 10px;
      padding: 10px;
    } 
    .opcao-vermelho { background: #e74c3c; }
    .opcao-azul    { background: #3498db; }
    .opcao-amarelo { background: #f1c40f; color: #000; }
    .opcao-verde   { background: #2ecc71; }
    .correta { background: #35ba6c !important; color: #fff !important; }
    .errada  { background: #c0392b !important; color: #fff !important; }
    .options {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 15px;
      width: 90%;
      max-width: 600px;
    }
    .options button {
      padding: 40px;
      font-size: 1.5em;
      border-radius: 15px;
      cursor: pointer;
    }
    .desativado { background: #aec0c2 !important; color: #040e11 !important; }

    /* ===== Teclado Virtual ===== */
    #keyboard {
       #keyboard {
    display: none; /* oculto por padrão */
    border: 1px solid #ccc;
    padding: 10px;
    background: #f9048f;
    width: fit-content;
    position: absolute; /* para poder posicionar perto do input */
    bottom: 20px;
    left: 20px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.2);
  }

  .row {
    display: flex;
    margin-bottom: 8px;
  }

  .key {
    display: inline-block;
    padding: 10px 15px;
    margin: 2px;
    background: #1aa8cf;
    border: 1px solid #9713f5;
    border-radius: 5px;
    cursor: pointer;
    user-select: none;
  }

  .key:hover {
    background: #1310d7;
  }

  .key-wide {
    flex: 1;
    text-align: center;
  }
}
    @keyframes fadeIn { from {opacity:0} to {opacity:1} }
  </style>
</head>
<body>

<!-- Tela 1 -->
<div class="screen active" id="screen1">
  <img src="public/robo2.png" alt="carinha de robo" style="width:50%; height:auto; display:block; margin:0 auto;">
  <button onclick="mostrarTela2()">Continuar</button>
</div>
  
<!-- Tela 2: Nome e Ano -->
<div class="screen" id="screen2">
  <h1>Digite seu nome e selecione o ano escolar</h1>
  <input type="text" id="nome" placeholder="Seu nome" readonly>
  <select id="ano">
    <option value="">Escolha o ano</option>
    <option value="3º ano">3º ano</option>
    <option value="4º ano">4º ano</option>
    <option value="5º ano">5º ano</option>
    <option value="6º ano">6º ano</option>
    <option value="2º ano">2º ano técnico</option>
  </select>
  <button onclick="iniciarQuiz(); reset()">Continuar</button>

  <!-- Teclado Virtual -->
  <div id="keyboard">
    <div class="row">
      <span class="key">Q</span><span class="key">W</span><span class="key">E</span><span class="key">R</span><span class="key">T</span>
      <span class="key">Y</span><span class="key">U</span><span class="key">I</span><span class="key">O</span><span class="key">P</span>
    </div>
    <div class="row">
      <span class="key">A</span><span class="key">S</span><span class="key">D</span><span class="key">F</span><span class="key">G</span>
      <span class="key">H</span><span class="key">J</span><span class="key">K</span><span class="key">L</span>
    </div>
    <div class="row">
      <span class="key">Z</span><span class="key">X</span><span class="key">C</span><span class="key">V</span><span class="key">B</span>
      <span class="key">N</span><span class="key">M</span>
      <span class="key key-wide" data-action="backspace">⌫</span>
    </div>
    <div class="row">
      <span class="key key-wide" data-action="space">Espaço</span>
    </div>
  </div>
</div>

<!-- Outras Telas -->
<div class="screen" id="loadingScreen"><h1>Preparando para as perguntas...</h1></div>
<div class="screen" id="questionScreen">
  <h2 id="pergunta"></h2>

  <div class="options">
    <button onclick="responder(0)" id="opt0" class="opcao-vermelho"></button>
    <button onclick="responder(1)" id="opt1" class="opcao-azul"></button>
    <button onclick="responder(2)" id="opt2" class="opcao-amarelo"></button>
    <button onclick="responder(3)" id="opt3" class="opcao-verde"></button>
  </div>

  <!-- Barra de tempo rosa -->
  <div id="timerBar" style="width: 80%; height: 20px; background: #ddd; border-radius: 15px; margin: 20px auto;">
    <div id="preenchimentoBarra" style="width: 100%; height: 100%; background: #ff69b4; border-radius: 15px;"></div>
  </div>
</div>

<div class="screen" id="feedbackScreen">
  <h1 id="feedbackMsg"></h1>
  <button onclick="proximaPergunta()">Próxima</button>
</div>
<div class="screen" id="reportScreen">
  <h2 id="resumo"></h2>
  <canvas id="grafico"></canvas>
  <button onclick="mostrarRecompensa()">Ver Surpresa</button>
</div>
<div class="screen" id="rewardScreen">
  <h1>Parabéns, <span id="userName"></span>! 🎉</h1>
  <p>Você acertou tudinho! Aqui está sua surpresa: 🎁</p>
</div>

<script>
  const perguntas3 = [
    { q: "5 + 7 = ?", options: ["10", "12", "13", "14"], correct: 1, category: "Soma"},
    { q: "15 - 8 = ?", options: ["6", "7", "8", "5"], correct: 1, category: "Subtração" },
    { q: "9 + 4 = ?", options: ["12", "14", "13", "15"], correct: 2, category: "Soma" },
    { q: "18 - 6 = ?", options: ["12", "11", "10", "13"], correct: 0, category: "Subtração" },
    { q: "7 + 7 = ?", options: ["12", "15", "13", "14"], correct: 3, category: "Soma" },
    { q: "20 - 9 = ?", options: ["10", "12", "11", "9"], correct: 2, category: "Subtração" },
    { q: "6 + 8 = ?", options: ["12", "14", "13", "15"], correct: 1, category: "Soma" },
    { q: "20 - 9 = ?", options: ["10", "12", "11", "9"], correct: 2, category: "Subtração" }
  ];

  const perguntas4 = [
    { q: "6 x 7 = ?", options: ["42", "36", "40", "48"], correct: 0, category: "Multiplicação" },
    { q: "56 ÷ 8 = ?", options: ["6", "8", "7", "9"], correct: 2, category: "Divisão" },
    { q: "9 x 5 = ?", options: ["45", "40", "35", "50"], correct: 0, category: "Multiplicação" },
    { q: "72 ÷ 9 = ?", options: ["6", "9", "8", "7"], correct: 2, category: "Divisão" },
    { q: "8 x 8 = ?", options: ["64", "72", "56", "60"], correct: 0, category: "Multiplicação" },
    { q: "63 ÷ 7 = ?", options: ["8", "9", "7", "10"], correct: 1, category: "Divisão" },
    { q: "12 x 4 = ?", options: ["48", "36", "40", "44"], correct: 0, category: "Multiplicação" },
    { q: "63 ÷ 7 = ?", options: ["8", "9", "7", "10"], correct: 1, category: "Divisão" }
  ];

  const perguntas5 = [
    { q: "João comprou 3 cadernos por R$12 cada. Quanto gastou?", options: ["R$36", "R$30", "R$24", "R$40"], correct: 0, category: "Multiplicação" },
    { q: "Um ônibus transporta 45 pessoas. Quantas pessoas em 6 ônibus?", options: ["270", "300", "280", "250"], correct: 0, category: "Multiplicação" },
    { q: "Se uma pizza custa R$48 e é dividida em 8 fatias, quanto custa cada fatia?", options: ["R$6", "R$8", "R$5", "R$7"], correct: 0, category: "Divisão" },
    { q: "Um aluno leu 120 páginas em 4 dias. Quantas páginas por dia?", options: ["25", "30", "40", "35"], correct: 1, category: "Divisão" },
    { q: "Se 5 caixas têm 125 balas no total, quantas balas em cada caixa?", options: ["30", "20", "25", "15"], correct: 2, category: "Divisão" },
    { q: "Um produto de R$200 teve desconto de R$50. Quanto custa agora?", options: ["R$150", "R$160", "R$140", "R$130"], correct: 0, category: "Multiplicação" },
    { q: "Qual o resultado de 120 ÷ 5?", options: ["25", "24", "22", "20"], correct: 1, category: "Divisão"},
    { q: "João comprou 3 cadernos por R$12 cada. Quanto gastou?", options: ["R$36", "R$30", "R$24", "R$40"], correct: 0, category: "Multiplicação" }
  ];

  const perguntas6 = [
    { q: "Qual é 1/2 de 3/4?", options: ["3/8", "3/4", "2/4", "1/8"], correct: 0, category: "Fração" },
    { q: "Resolva: 2x = 10. Quanto vale x?", options: ["4", "5", "10", "2"], correct: 1, category: "Equação" },
    { q: "Quanto é 3/5 + 1/5?", options: ["4/5", "3/10", "5/10", "5/5"], correct: 0, category: "Fração" },
    { q: "Resolva: 12 ÷ (3 + 1)", options: ["3", "4", "2", "6"], correct: 0, category: "Equação" },
    { q: "Qual fração representa metade?", options: ["1/4", "1/2", "2/4", "2/3"], correct: 1, category: "Fração" },
    { q: "Resolva: x + 7 = 12", options: ["4", "5", "6", "3"], correct: 1, category: "Equação" },
    { q: "Se uma barra de chocolate tem 12 pedaços e você come 3, qual fração foi comida?", options: ["1/4", "1/2", "1/3", "1/6"], correct: 0, category: "Fração" },
    { q: "Resolva: 2x = 10. Quanto vale x?", options: ["4", "5", "10", "2"], correct: 1, category: "Equação" }
  ];

  const perguntas2 = [
    { q: "Qual é a lógica da sequência: 1, 3, 5, 7, ___?", options: ["2", "8", "9", "10"], correct: 2},
  ];

  let perguntas = []; 
  let resultados = [];
  let nome = "";
  let ano = "";
  let indice = 0;
  let acertos = 0;
  let tempo = 10;
  let timerInterval = null; 
  
  // === Teclado Virtual ===
  const input = document.getElementById("nome");
  const keyboard = document.getElementById("keyboard");

  input.addEventListener("click", (e) => {
    e.stopPropagation();
    keyboard.style.display = "block";
  });

  keyboard.addEventListener("click", (e) => {
    e.stopPropagation();
  });

  document.addEventListener("click", () => {
    keyboard.style.display = "none";
  });

  document.querySelectorAll(".key").forEach(key => {
    key.addEventListener("click", () => {
      const action = key.dataset.action;
      if (action === "backspace") {
        input.value = input.value.slice(0, -1);
      } else if (action === "space") {
        input.value += " ";
      } else {
        input.value += key.textContent;
      }
    });
  });

  function mostrarTela2() {
    document.getElementById('screen1').classList.remove('active');
    document.getElementById('screen2').classList.add('active');
  }

  function iniciarQuiz() {
    nome = document.getElementById('nome').value;
    ano = document.getElementById('ano').value;
    if (!nome || !ano) return alert("Preencha tudo!");

    if (ano === "3º ano") perguntas = perguntas3;
    if (ano === "4º ano") perguntas = perguntas4;
    if (ano === "5º ano") perguntas = perguntas5;
    if (ano === "6º ano") perguntas = perguntas6;
    if (ano === "2º ano") perguntas = perguntas2;

    trocarTela('loadingScreen');
    setTimeout(() => {
      indice = 0;
      acertos = 0;
      mostrarPergunta();
    }, 2000);
  }

  function mostrarPergunta() {
    trocarTela('questionScreen');
    const p = perguntas[indice];
    document.getElementById('pergunta').textContent = p.q;

    p.options.forEach((op, i) => {
      const botao = document.getElementById(`opt${i}`);
      botao.textContent = op;

      botao.classList.remove('correta', 'errada', 'desativado');

      if (i === 0) botao.className = 'opcao-vermelho';
      if (i === 1) botao.className = 'opcao-azul';
      if (i === 2) botao.className = 'opcao-amarelo';
      if (i === 3) botao.className = 'opcao-verde';
      botao.id = `opt${i}`;
    });

    tempo = 10;
    const barra = document.getElementById('preenchimentoBarra');
barra.style.width = '100%';
    clearInterval(timerInterval);
   timerInterval = setInterval(() => {
  tempo--;
  const barra = document.getElementById('preenchimentoBarra');
  let porcentagem = (tempo / 10) * 100; // 10 é o tempo total da pergunta
  barra.style.width = porcentagem + '%';

  if (tempo <= 0) {
    clearInterval(timerInterval);
    verificarResposta(-1);
  }
}, 1000);
  }

  function trocarTela(id) {
    document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
  }

  function responder(i) {
    clearInterval(timerInterval);
    verificarResposta(i);
  }

  function verificarResposta(i) {
    const correta = perguntas[indice].correct;
    const botoes = document.querySelectorAll('.options button');

    botoes.forEach(b => {
      b.classList.remove('correta', 'errada', 'desativado');
    });

    resultados.push({ categoria: perguntas[indice].category, acertou: i === correta });

    if (i === correta) {
      acertos++;
      socket.emit('acertou'); 
      botoes.forEach((b, index) => {
        if (index === correta) {
          b.classList.add('correta');
        } else {
          b.classList.add('desativado');
        }
      });
    } else {
      botoes.forEach((b, index) => {
        if (index === i) {
          b.classList.add('errada');
        } else if (index === correta) {
          b.classList.add('correta');
        } else {
          b.classList.add('desativado');
        }
      });
    }
    setTimeout(() => {
      if (i === correta) {
        document.getElementById('feedbackMsg').textContent = "Acertou!";
      } else {
        document.getElementById('feedbackMsg').textContent = "Essa foi uma boa tentativa! Vamos tentar de novo?";
      }
      trocarTela('feedbackScreen');
    }, 1000);
  }

  function proximaPergunta() {
    indice++;
    if (indice < perguntas.length) {
      mostrarPergunta();
    } else {
      mostrarRelatorio();
    }
  }

  function mostrarRelatorio() {
  trocarTela('reportScreen');
  const erros = perguntas.length - acertos;

  document.getElementById('resumo').textContent =
    `${nome} (${ano}) - Acertos: ${acertos}/${perguntas.length}`;

  // Centralizar o gráfico
  const graficoContainer = document.createElement('div');
  graficoContainer.style.display = 'flex';
  graficoContainer.style.justifyContent = 'center';
  graficoContainer.style.marginTop = '20px';

  const ctx = document.getElementById('grafico').getContext('2d');
  new Chart(ctx, {
    type: 'pie',
    data: {
      labels: ['Acertos', 'Erros'],
      datasets: [{
        data: [acertos, erros],
        backgroundColor: ['#3498db', '#e74c3c']
      }]
    }
  });

  graficoContainer.appendChild(document.getElementById('grafico'));
  document.getElementById('reportScreen').appendChild(graficoContainer);

  // Calcular categorias
  let categorias = {};
  resultados.forEach(r => {
    if (!categorias[r.categoria]) {
      categorias[r.categoria] = { total: 0, acertos: 0, erros: 0 };
    }
    categorias[r.categoria].total++;
    if (r.acertou) {
      categorias[r.categoria].acertos++;
    } else {
      categorias[r.categoria].erros++;
    }
  });

  // Função para criar painel de categorias
  function criarPainel(titulo, cor, dados, tipo) {
    let painel = `<div style="padding:10px;border-radius:20px;width:200px;margin:10px;">
                    <h3 style="margin:0;background-color:${cor};border-radius:15px;padding:5px;text-align:center;color:white;">${titulo}</h3>`;
    for (let cat in dados) {
      let porcento = ((tipo === 'acertos' ? dados[cat].acertos : dados[cat].erros) / dados[cat].total * 100).toFixed(1);
      painel += `<p>${cat}: ${porcento}%</p>`;
    }
    painel += `</div>`;
    return painel;
  }

  const painelAcertos = criarPainel('Acertos', '#1E90FF', categorias, 'acertos');
  const painelErros = criarPainel('Erros', '#e74c3c', categorias, 'erros');

  // Pior categoria
  let piorCategoria = null;
  let maiorErro = 0;
  for (let cat in categorias) {
    let porcentoErro = (categorias[cat].erros / categorias[cat].total) * 100;
    if (porcentoErro > maiorErro) {
      maiorErro = porcentoErro;
      piorCategoria = cat;
    }
  }

  const painelMelhoria = `<div style="padding:10px;border-radius:20px;width:300px;margin:10px;">
                            <h3 style="margin:0;background-color:#2ecc71;border-radius:15px;padding:5px;text-align:center;color:white;">Pontos de melhoria:</h3>
                            <p>O aluno <b>${nome}</b> apresentou mais dificuldades em <b>${piorCategoria}</b>, 
                            por consequência é indicado prestar apoio e mais atenção em atividades relacionadas a <b>${piorCategoria}</b>.</p>
                          </div>`;

  // Container dos painéis
  let container = document.createElement('div');
  container.style.display = 'flex';
  container.style.justifyContent = 'center';
  container.style.alignItems = 'flex-start';
  container.style.gap = '20px';
  container.style.marginTop = '20px';
  container.innerHTML = painelAcertos + painelErros + painelMelhoria;

  document.getElementById('reportScreen').appendChild(container);

  // Botão de próxima ação
  const botao = document.querySelector('#reportScreen button');
  botao.style.display = acertos >= 6 ? 'inline-block' : 'none';
}


  function mostrarRecompensa() {
    socket.emit('recompensa');
    trocarTela('rewardScreen');
    document.getElementById('userName').textContent = nome;
  }

  const socket = io('http://localhost:5001');
  console.log("O script está rodando!");

  socket.on('botao', data => {
    console.log('Evento de botão recebido! Dados:', data);
    const botao = data.botao.replace('BTN','') - 1;
    responder(botao);
  });

  function reset() {
    socket.emit("reset");
  }
</script>
</body>
</html>orAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
  }

  function responder(i) {
    clearInterval(timerInterval);
    verificarResposta(i);
  }

  function verificarResposta(i) {
    const correta = perguntas[indice].correct;
    const botoes = document.querySelectorAll('.options button');

    botoes.forEach(b => {
      b.classList.remove('correta', 'errada', 'desativado');
    });

    resultados.push({ categoria: perguntas[indice].category, acertou: i === correta });

    if (i === correta) {
      acertos++;
      socket.emit('acertou'); 
      botoes.forEach((b, index) => {
        if (index === correta) {
          b.classList.add('correta');
        } else {
          b.classList.add('desativado');
        }
      });
    } else {
      botoes.forEach((b, index) => {
        if (index === i) {
          b.classList.add('errada');
        } else if (index === correta) {
          b.classList.add('correta');
        } else {
          b.classList.add('desativado');
        }
      });
    }
    setTimeout(() => {
      if (i === correta) {
        document.getElementById('feedbackMsg').textContent = "Acertou!";
      } else {
        document.getElementById('feedbackMsg').textContent = "Essa foi uma boa tentativa! Vamos tentar de novo?";
      }
      trocarTela('feedbackScreen');
    }, 1000);
  }

  function proximaPergunta() {
    indice++;
    if (indice < perguntas.length) {
      mostrarPergunta();
    } else {
      mostrarRelatorio();
    }
  }

  function mostrarRelatorio() {
    trocarTela('reportScreen');
    const erros = perguntas.length - acertos;

    document.getElementById('resumo').textContent =
      `${nome} (${ano}) - Acertos: ${acertos}/${perguntas.length}`;

    const ctx = document.getElementById('grafico').getContext('2d');
    new Chart(ctx, {
      type: 'pie',
      data: {
        labels: ['Acertos', 'Erros'],
        datasets: [{
          data: [acertos, erros],
          backgroundColor: ['#3498db', '#e74c3c']
        }]
      }
    });

    let categorias = {};
    resultados.forEach(r => {
      if (!categorias[r.categoria]) {
        categorias[r.categoria] = { total: 0, acertos: 0, erros: 0 };
      }
      categorias[r.categoria].total++;
      if (r.acertou) {
        categorias[r.categoria].acertos++;
      } else {
        categorias[r.categoria].erros++;
      }
    });

    let painelAcertos = `<div style="background:#1E90FF;padding:10px;border-radius:20px;width:200px;margin:10px;">
                           <h3 style="margin:0;">Acertos:</h3>`;
    for (let cat in categorias) {
      let porcentoAcerto = ((categorias[cat].acertos / categorias[cat].total) * 100).toFixed(1);
      painelAcertos += `<p>${cat}: ${porcentoAcerto}%</p>`;
    }
    painelAcertos += `</div>`;

    let painelErros = `<div style="background:#e74c3c;padding:10px;border-radius:20px;width:200px;margin:10px;">
                         <h3 style="margin:0;">Erros:</h3>`;
    for (let cat in categorias) {
      let porcentoErro = ((categorias[cat].erros / categorias[cat].total) * 100).toFixed(1);
      painelErros += `<p>${cat}: ${porcentoErro}%</p>`;
    }
    painelErros += `</div>`;

    let piorCategoria = null;
    let maiorErro = 0;
    for (let cat in categorias) {
      let porcentoErro = (categorias[cat].erros / categorias[cat].total) * 100;
      if (porcentoErro > maiorErro) {
        maiorErro = porcentoErro;
        piorCategoria = cat;
      }
    }

    let painelMelhoria = `<div style="background:#2ecc71;padding:10px;border-radius:20px;width:300px;margin:10px;">
                            <h3 style="margin:0;">Pontos de melhoria:</h3>
                            <p>O aluno <b>${nome}</b> apresentou mais dificuldades em <b>${piorCategoria}</b>, 
                            por consequência é indicado prestar apoio e mais atenção em atividades relacionadas a <b>${piorCategoria}</b>.</p>
                          </div>`;

    let container = document.createElement('div');
    container.style.display = 'flex';
    container.style.justifyContent = 'center';
    container.style.alignItems = 'flex-start';
    container.style.gap = '20px';
    container.style.marginTop = '20px';
    container.innerHTML = painelAcertos + painelErros + painelMelhoria;

    document.getElementById('reportScreen').appendChild(container);

    const botao = document.querySelector('#reportScreen button');
    if (acertos >= 6) {
      botao.style.display = 'inline-block';
    } else {
      botao.style.display = 'none';
    }
  }

  function mostrarRecompensa() {
    socket.emit('recompensa');
    trocarTela('rewardScreen');
    document.getElementById('userName').textContent = nome;
  }

  const socket = io('http://localhost:5001');
  console.log("O script está rodando!");

  socket.on('botao', data => {
    console.log('Evento de botão recebido! Dados:', data);
    const botao = data.botao.replace('BTN','') - 1;
    responder(botao);
  });

  function reset() {
    socket.emit("reset");
  }
</script>
</body>
</html>
