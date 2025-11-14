<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Atividade — Formas de Armazenagem e WMS</title>
 <img src="https://www.senairs.org.br/sites/default/files/styles/scale_sm/public/logos/avatares_sistema_fiergs_senai_cor.png?itok=fuuHGGm3"
       alt="Logo SENAI"
       style="height:60px; margin-right:16px; border-radius:8px;">
<style>
  :root{
    --bg:#0f1724; --card:#0b1220; --accent:#06b6d4; --muted:#94a3b8; --ok:#16a34a; --err:#ef4444;
    --glass: rgba(255,255,255,0.03);
    font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  body{background:linear-gradient(180deg,#071023 0%, #071829 100%); color:#e6eef6; margin:0; padding:24px;}
  .container{max-width:980px; margin:0 auto;}
  header{display:flex; gap:16px; align-items:center; margin-bottom:18px;}
  header h1{font-size:20px; margin:0;}
  .card{background:var(--card); padding:18px; border-radius:12px; box-shadow: 0 6px 20px rgba(2,6,23,0.6);}
  .meta{display:flex; gap:12px; align-items:center; flex-wrap:wrap;}
  label{font-size:14px; color:var(--muted);}
  input[type="text"]{padding:8px 10px; border-radius:8px; border:1px solid rgba(255,255,255,0.05); background:var(--glass); color:inherit; outline:none;}
  .controls{display:flex; gap:10px; margin-left:auto;}
  button{background:var(--accent); color:#042026; border:none; padding:8px 12px; border-radius:10px; cursor:pointer; font-weight:600;}
  button.ghost{background:transparent; color:var(--muted); border:1px solid rgba(255,255,255,0.04);}
  .question{margin-top:12px; padding:14px; border-radius:10px; background: linear-gradient(180deg, rgba(255,255,255,0.02), transparent); border:1px solid rgba(255,255,255,0.02);}
  .qtext{font-weight:600; margin-bottom:8px;}
  .opts{display:grid; grid-template-columns:1fr; gap:8px;}
  .opt{display:flex; gap:10px; align-items:center; padding:8px; border-radius:8px; cursor:pointer; border:1px dashed rgba(255,255,255,0.03);}
  .opt:hover{background:rgba(255,255,255,0.02);}
  .opt input{accent-color:var(--accent);}
  .feedback{margin-top:8px; font-weight:600;}
  .scorebar{display:flex; gap:10px; align-items:center; margin-top:12px;}
  .progress{height:14px; background:#071722; border-radius:12px; flex:1; overflow:hidden; border:1px solid rgba(255,255,255,0.03);}
  .progress > i{display:block; height:100%; background:linear-gradient(90deg,var(--accent),#7c3aed); width:0%;}
  .result{margin-top:14px; display:none; padding:14px; border-radius:10px;}
  .result.good{background:linear-gradient(90deg, rgba(22,163,74,0.1), rgba(124,58,237,0.04)); border:1px solid rgba(22,163,74,0.12);}
  .medal{font-size:28px; margin-top:8px;}
  .refs{font-size:13px; color:var(--muted); margin-top:14px;}
  @media print{
    body{background:white;color:black;}
    header, .controls, button, .refs{display:none;}
    .card{box-shadow:none; border-radius:0; background:transparent;}
  }
</style>
</head>
<body>
  <div class="container">
    <header>
      <div>
        <h1>Atividade: Formas de armazenagem & WMS</h1>
        <div class="meta">
          <div class="card" style="display:inline-block; padding:8px 12px; border-radius:8px;">
            Professor: Luiz Eduardo Peixoto
          </div>
        </div>
      </div>
      <div class="controls">
        <button id="exportPdf">Exportar para PDF</button>
        <button class="ghost" id="resetBtn">Reiniciar</button>
      </div>
    </header>

    <div class="card">
      <div style="display:flex; gap:12px; align-items:center;">
        <label for="aluno">Nome do aluno:</label>
        <input id="aluno" type="text" placeholder="Digite seu nome aqui" />
      </div>

      <div id="quizArea">
        <!-- Perguntas serão injetadas via JS -->
      </div>

      <div class="scorebar">
        <div style="min-width:160px">
          <div style="font-size:13px; color:var(--muted)">Pontuação: <span id="score">0</span>/<span id="maxScore">15</span></div>
          <div style="font-size:12px; color:var(--muted)">Acertos: <span id="correctCount">0</span></div>
        </div>

        <div class="progress" title="Progresso">
          <i id="progressFill"></i>
        </div>

        <div style="min-width:160px; text-align:right;">
          <div style="font-size:13px; color:var(--muted)">Tentativas totais: <span id="attempts">0</span></div>
        </div>
      </div>

      <div id="finalResult" class="result"></div>

      <div class="refs">
        Referências: material do curso — <em>3º Logística de armazenagem</em>.
      </div>
    </div>
  </div>

<script>
/*
  Regras:
  - 15 perguntas, 5 opções cada.
  - A resposta correta só é revelada ao aluno quando ele acerta.
  - Pontuação: cada acerto vale 1 ponto.
  - Medalhas: Ouro >= 90%, Prata >=70%, Bronze >=50%.
  - Exportar para PDF: usa window.print (gera versão para impressão).
*/

const questions = [
  {
    q: "1) Qual a diferença principal entre armazenagem tradicional e automatizada?",
    opts: [
      "A armazenagem tradicional usa apenas códigos de barras; a automatizada trata só de empilhamento.",
      "A armazenagem tradicional é baseada em processos manuais; a automatizada integra equipamentos e sistemas que reduzem tarefas manuais.",
      "A armazenagem automatizada não necessita de software WMS em hipótese alguma.",
      "Na armazenagem tradicional não existem estantes nem paletes.",
      "A armazenagem automatizada é sempre mais barata que a tradicional."
    ],
    a: 1,
    explain: "Armazenagem tradicional depende de operações manuais; a automatizada incorpora equipamentos (transelevadores, robôs) e sistemas que reduzem o trabalho manual."
  },
  {
    q: "2) O que é um WMS (Warehouse Management System)?",
    opts: [
      "Um tipo de empilhadeira elétrica.",
      "Um sistema de gestão que otimiza operações do armazém (recebimento, estocagem, picking, expedição).",
      "Uma norma de segurança para elevar paletes.",
      "Um padrão de palete utilizado no Brasil.",
      "Um controlador lógico de portas do galpão."
    ],
    a: 1,
    explain: "WMS é um software que gerencia e otimiza as operações do armazém, do recebimento à expedição."
  },
  {
    q: "3) Qual das alternativas é uma vantagem típica do WMS?",
    opts: [
      "Aumento da dependência de trabalho manual sem ganhos de controle.",
      "Redução de controles e inventários.",
      "Aumento da acuracidade de estoque e redução do tempo de inventário.",
      "Eliminação total da necessidade de funcionários.",
      "Transformar pallets em caixas automaticamente."
    ],
    a: 2,
    explain: "Um WMS aumenta a acuracidade dos estoques e reduz o tempo de inventário, entre outras vantagens."
  },
  {
    q: "4) 'Flow rack' é uma opção de armazenagem adequada para:",
    opts: [
      "Produtos extremamente pesados que não rolam.",
      "Produtos em caixas pequenas que precisam rotatividade (FIFO/PVPS).",
      "Armazenar líquidos sem embalagem.",
      "Substituir docas de recebimento.",
      "Armazenar somente equipamentos de grande dimensão (ex. laminas)."
    ],
    a: 1,
    explain: "Flow rack usa gravidade e roletes, ideal para rotatividade e entradas/saídas constantes (FIFO/PVPS)."
  },
  {
    q: "5) Em estruturas porta-pallet, o que significa 'vão'?",
    opts: [
      "A largura do corredor.",
      "O número do documento fiscal.",
      "A posição individual para um palete em um nível (como um 'apartamento').",
      "O tipo de palete utilizado.",
      "A altura total do armazém."
    ],
    a: 2,
    explain: "Vão é a posição porta-palete onde um palete fica armazenado — análogo a um 'apartamento' em um prédio."
  },
  {
    q: "6) Por que em endereçamento de armazém recomenda-se usar somente números (evitar letras)?",
    opts: [
      "Porque letras são ilegíveis para empilhadeiras.",
      "Porque códigos numéricos facilitam cálculos, leitura de códigos de barras e escalabilidade do sistema.",
      "Letras são proibidas por normas internacionais.",
      "Porque letras ocupam mais espaço nas etiquetas.",
      "Porque números são mais bonitos."
    ],
    a: 1,
    explain: "Endereços numéricos facilitam a leitura, cálculos e compactação em códigos de barras."
  },
  {
    q: "7) O que é 'unitização' em armazenagem?",
    opts: [
      "Separar produtos sem empilhar.",
      "Unir mercadorias em cargas unitárias (paletes) para otimizar transporte e manuseio.",
      "Retirar embalagens primárias de produtos.",
      "A técnica de empilhamento para empilhadeiras pequenas.",
      "A criação de etiquetas para endereçamento."
    ],
    a: 1,
    explain: "Unitização é agrupar mercadorias em cargas unitárias, como paletes, para facilitar transporte e conferência."
  },
  {
    q: "8) Qual tecnologia é frequentemente usada com WMS para coleta de dados em tempo real?",
    opts: [
      "Caneta e papel.",
      "Coletores de dados com RFID e leitores de código de barras.",
      "Rodas de empilhadeira manuais.",
      "Máquinas de envelope.",
      "Balanças mecânicas antigas."
    ],
    a: 1,
    explain: "WMS costuma integrar coletores de dados com RFID ou leitores de código de barras para operação em tempo real."
  },
  {
    q: "9) 'Drive-in' e 'Drive-thru' são sistemas de estantes indicados para:",
    opts: [
      "Alta rotatividade de muitos SKU diferentes.",
      "Armazenagem compacta de grandes quantidades do mesmo SKU, reduzindo corredores.",
      "Somente produtos frágeis.",
      "Substituir WMS em centros de distribuição.",
      "Armazenar somente líquidos."
    ],
    a: 1,
    explain: "Drive-in/drive-thru são usados para armazenar muitas unidades do mesmo SKU com espaço otimizado; corredores são reduzidos."
  },
  {
    q: "10) Uma vantagem clara da automatização do armazém é:",
    opts: [
      "Reduzir erros humanos e otimizar movimentações quando bem implementada com software.",
      "Eliminar totalmente qualquer custo.",
      "Tornar o espaço mais barato por metro quadrado.",
      "Tornar os processos menos visíveis.",
      "Impedir auditorias."
    ],
    a: 0,
    explain: "Automatização, integrada a software, reduz erros e otimiza a movimentação — mas exige investimento inicial."
  },
  {
    q: "11) Qual é uma das principais responsabilidades operacionais de um WMS?",
    opts: [
      "Gerar receitas financeiras diretamente.",
      "Alocar mercadorias no armazém de acordo com características físicas e regras operacionais.",
      "Escrever manuais de RH.",
      "Substituir toda a sinalização física do armazém.",
      "Determinar o salário dos operadores."
    ],
    a: 1,
    explain: "WMS aloca mercadorias conforme características (tamanho, peso, giro) e regras, integrando o fluxo do armazém."
  },
  {
    q: "12) 'Mini-load' é indicado para:",
    opts: [
      "Peças de pequeno porte armazenadas em caixas/bandejas e movimentadas por transelevadores.",
      "Transporte rodoviário de pallets.",
      "Substituir empilhadeiras padrão em todos os armazéns.",
      "Armazenar grandes bobinas metálicas.",
      "Única solução para armazéns frigoríficos."
    ],
    a: 0,
    explain: "Mini-load é usado para pequenos itens em caixas/bandejas, com transelevadores para otimizar espaços verticais."
  },
  {
    q: "13) Em termos de layout, para onde devem ficar os itens de maior rotatividade (popularidade)?",
    opts: [
      "Próximo às áreas de saída/expedição para reduzir deslocamentos.",
      "No ponto mais distante do armazém.",
      "No meio de um corredor aleatório.",
      "Sempre no nível mais alto do porta-pallet.",
      "Em áreas externas sem proteção."
    ],
    a: 0,
    explain: "Itens de alta rotatividade devem ficar próximos à saída para reduzir tempo de picking e deslocamentos."
  },
  {
    q: "14) Quais desafios permanecem mesmo com um WMS bem implementado?",
    opts: [
      "Erro humano, necessidade de treinamento e manutenção de dados atualizados.",
      "WMS resolve todos os problemas sem necessidade de pessoas.",
      "Aumento infinito de espaço sem custo.",
      "Eliminar inspeções físicas totalmente.",
      "Garantir que nenhum equipamento quebre."
    ],
    a: 0,
    explain: "Mesmo com WMS, erro humano, treinamento e qualidade dos dados continuam sendo desafios centrais."
  },
  {
    q: "15) PVPS (Primeiro a Vencer, Primeiro a Sair) é uma estratégia voltada para:",
    opts: [
      "Produtos com maior valor monetário apenas.",
      "Produtos com data de validade que devem sair primeiro (priorizar vencimento próximo).",
      "Organizar equipamentos de movimentação.",
      "Eliminar inventários.",
      "Rastrear empilhadeiras."
    ],
    a: 1,
    explain: "PVPS prioriza a saída de produtos com validade próxima, evitando perdas por vencimento."
  }
];

// Estado
let score = 0;
let correctCount = 0;
let attempts = 0;
const maxScore = questions.length;

document.getElementById('maxScore').textContent = maxScore;

function createQuiz(){
  const area = document.getElementById('quizArea');
  area.innerHTML = '';
  questions.forEach((item, idx) => {
    const div = document.createElement('div');
    div.className = 'question';
    div.id = 'q'+idx;
    div.innerHTML = `
      <div class="qtext">${item.q}</div>
      <div class="opts" id="opts-${idx}"></div>
      <div class="feedback" id="fb-${idx}"></div>
    `;
    area.appendChild(div);

    const optsDiv = div.querySelector('#opts-'+idx);
    item.opts.forEach((text, j) => {
      const opt = document.createElement('label');
      opt.className = 'opt';
      opt.innerHTML = `
        <input type="radio" name="q${idx}" value="${j}" />
        <div style="flex:1">${text}</div>
      `;
      // click handler
      opt.addEventListener('click', (e) => {
        // find radio
        const input = opt.querySelector('input');
        input.checked = true;
        handleAnswer(idx, j);
      });
      optsDiv.appendChild(opt);
    });
  });
}

function handleAnswer(qIdx, chosenIdx){
  attempts++;
  document.getElementById('attempts').textContent = attempts;
  const q = questions[qIdx];
  const fb = document.getElementById('fb-'+qIdx);
  // If already answered correctly, do nothing
  if (fb.dataset.correct === "true") return;

  if (chosenIdx === q.a){
    // Correct!
    score++;
    correctCount++;
    fb.dataset.correct = "true";
    fb.style.color = "var(--ok)";
    fb.innerHTML = `✔ Correto! ${q.explain}`;
    // Mark option visually
    const container = document.getElementById('q'+qIdx);
    Array.from(container.querySelectorAll('.opt')).forEach((el, i) => {
      if (i === chosenIdx){
        el.style.background = "linear-gradient(90deg, rgba(16,185,129,0.06), rgba(124,58,237,0.02))";
        el.style.border = "1px solid rgba(16,185,129,0.18)";
      } else {
        el.style.opacity = "0.7";
      }
    });
    updateProgress();
    checkFinal();
  } else {
    // Wrong: do not reveal correct alternative. Prompt to try again.
    fb.style.color = "var(--err)";
    fb.innerHTML = `✖ Errado. Tente novamente (dica: reveja conceitos de armazenamento e WMS).`;
    // small shake or style to indicate try again
    const cont = document.getElementById('q'+qIdx);
    cont.animate([{transform:'translateX(0)'},{transform:'translateX(-6px)'},{transform:'translateX(6px)'},{transform:'translateX(0)'}], {duration:260});
  }
}

function updateProgress(){
  document.getElementById('score').textContent = score;
  document.getElementById('correctCount').textContent = correctCount;
  const pct = Math.round((correctCount / maxScore) * 100);
  const fill = document.getElementById('progressFill');
  fill.style.width = pct + '%';
}

function checkFinal(){
  // if all correct, show final result
  if (correctCount === maxScore){
    const name = document.getElementById('aluno').value || 'Aluno(a)';
    const pct = Math.round((score / maxScore) * 100);
    const final = document.getElementById('finalResult');
    final.style.display = 'block';
    final.className = 'result good';
    const medal = (pct >= 90) ? '🏅 OURO' : (pct >= 70) ? '🥈 PRATA' : (pct >= 50) ? '🥉 BRONZE' : '🎖️ PARTICIPAÇÃO';
    final.innerHTML = `
      <div style="font-size:16px; font-weight:700;">Parabéns, ${name}!</div>
      <div style="margin-top:6px;">Você acertou ${score}/${maxScore} (${pct}%).</div>
      <div class="medal">${medal}</div>
      <div style="margin-top:8px; font-size:13px; color:var(--muted)">Medalhas: Ouro ≥ 90%, Prata ≥ 70%, Bronze ≥ 50%.</div>
    `;
  } else {
    // partial progress message
    const final = document.getElementById('finalResult');
    final.style.display = 'block';
    final.className = 'result';
    const pct = Math.round((score / maxScore) * 100);
    final.innerHTML = `<div style="font-weight:700">Progresso: ${score}/${maxScore} (${pct}%). Continue respondendo até completar todas as questões.</div>`;
  }
}

document.getElementById('exportPdf').addEventListener('click', () => {
  // prepare for print: hide feedback that is not relevant? We'll just call print.
  window.print();
});

document.getElementById('resetBtn').addEventListener('click', () => {
  if (!confirm('Deseja reiniciar a atividade (as respostas serão perdidas)?')) return;
  score = 0; correctCount = 0; attempts = 0;
  document.getElementById('attempts').textContent = attempts;
  document.getElementById('score').textContent = score;
  document.getElementById('correctCount').textContent = correctCount;
  document.getElementById('progressFill').style.width = '0%';
  createQuiz();
  document.getElementById('finalResult').style.display = 'none';
});

createQuiz();
updateProgress();
</script>
</body>
</html>

