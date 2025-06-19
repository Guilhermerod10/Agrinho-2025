# Agrinho-2025
Projeto Agrinho com o tema: festejando a conexão entre o campo e a cidade
https://editor.p5js.org/guilherme.ramazottirodrigues.nascimento/sketches/7FdF1KMe4
// Jogo do Projeto Agrinho

let gameState = "start";
let fazendeiro;
let vegetais = [];
let score = 0;
let tempo = 60; // segundos
let startTime = 0;

// Quantidade de vegetais por rodada
const VEGETABLES_PER_ROUND = 8;

function setup() {
  createCanvas(800, 600);
  textFont("Arial");

  // Inicializa o fazendeiro
  fazendeiro = {
    x: width / 2 - 25,
    y: height / 2 - 25,
    size: 50,
    speed: 5,
    visible: false,
  };

  resetarVegetais();
}

function draw() {
  background(180, 230, 255);

  if (gameState === "start") {
    desenharTelaInicial();
  } else if (gameState === "playing") {
    desenhaJogo();
    atualizaTempo();
    verificaGameOver();
  } else if (gameState === "gameover") {
    desenhaTeladeGameOver();
  }
}

function desenharTelaInicial() {
  fill(0, 100, 0);
  textAlign(CENTER, CENTER);
  textSize(38);
  text("Projeto Agrinho: Coletando Vegetais", width / 2, height / 2 - 80);

  fill(60, 180, 75);
  rect(width / 2 - 75, height / 2, 150, 60, 15);

  fill(255);
  textSize(24);
  text("START", width / 2, height / 2 + 30);

  textSize(18);
  fill(60);
  text("Colete o máximo de vegetais em 1 minuto!", width / 2, height / 2 + 110);
}

// Tela de fim de jogo
function desenhaTeladeGameOver() {
  textAlign(CENTER, CENTER);
  textSize(40);
  text("Fim de Jogo!", width / 2, height / 2 - 80);

  fill(60, 180, 75);
  textSize(28);
  text("Pontuação final: " + score, width / 2, height / 2 - 30);

  fill(60);
  textSize(20);
  text("Clique para jogar novamente", width / 2, height / 2 + 50);
}

function desenhaJogo() {
  // Vegetais
  for (let veg of vegetais) {
    if (!veg.collected) drawVegetable(veg);
  }

  // Fazendeiro
  if (fazendeiro.visible) {
    desenhaFazendeiro();
    handleMovement();
    checkCollect();
  }

  // Pontuação
  fill(0, 130, 0);
  textAlign(LEFT, TOP);
  textSize(22);
  text("Vegetais: " + score, 15, 15);

  // Cronômetro
  let timeLeft = max(0, 60 - int((millis() - startTime) / 1000));
  fill(0, 130, 0);
  textAlign(RIGHT, TOP);
  textSize(22);
  text("Tempo: " + nf(timeLeft, 2) + "s", width - 15, 15);
}

function mousePressed() {
  if (gameState === "start") {
    let bx = width / 2 - 75;
    let by = height / 2;
    let bw = 150;
    let bh = 60;
    if (mouseX > bx && mouseX < bx + bw && mouseY > by && mouseY < by + bh) {
      startGame();
    }
  } else if (gameState === "gameover") {
    fazendeiro.x = width / 2 - 25;
    fazendeiro.y = height / 2 - 25;
    score = 0;
    resetarVegetais();
    startGame();
  }
}

// Iniciar jogo
function startGame() {
  gameState = "playing";
  fazendeiro.visible = true;
  startTime = millis();
  timer = 60;
  score = 0;
  resetarVegetais();
}

function desenhaFazendeiro() {
  // Corpo
  fill(222, 184, 135);
  rect(fazendeiro.x, fazendeiro.y, fazendeiro.size, fazendeiro.size, 10);

  // Chapéu
  fill(160, 82, 45);
  rect(fazendeiro.x + 10, fazendeiro.y - 15, 30, 15, 5);
  rect(fazendeiro.x + 5, fazendeiro.y - 10, 40, 10, 3);

  // Rosto
  fill(255, 228, 181);
  ellipse(fazendeiro.x + fazendeiro.size / 2, fazendeiro.y + 20, 28, 28);

  // Olhos
  fill(0);
  ellipse(fazendeiro.x + 20, fazendeiro.y + 18, 4, 4);
  ellipse(fazendeiro.x + 30, fazendeiro.y + 18, 4, 4);
}

function handleMovement() {
  if (keyIsDown(87)) {
    // W
    fazendeiro.y -= fazendeiro.speed;
  }
  if (keyIsDown(65)) {
    // A
    fazendeiro.x -= fazendeiro.speed;
  }
  if (keyIsDown(83)) {
    // S
    fazendeiro.y += fazendeiro.speed;
  }
  if (keyIsDown(68)) {
    // D
    fazendeiro.x += fazendeiro.speed;
  }
  fazendeiro.x = constrain(fazendeiro.x, 0, width - fazendeiro.size);
  fazendeiro.y = constrain(fazendeiro.y, 0, height - fazendeiro.size);
}

// Desenha vegetais diferentes usando formas e cores
function drawVegetable(veg) {
  push();
  translate(veg.x + veg.size / 2, veg.y + veg.size / 2);
  if (veg.type === "tomato") {
    fill(255, 50, 50);
    ellipse(0, 0, veg.size, veg.size);
    fill(0, 180, 0);
    rect(-5, -veg.size / 2 + 5, 10, 10, 5);
  } else if (veg.type === "carrot") {
    fill(255, 140, 0);
    triangle(
      0,
      veg.size / 2 - 3,
      -veg.size / 2 + 5,
      -veg.size / 2 + 8,
      veg.size / 2 - 5,
      -veg.size / 2 + 8
    );
    fill(0, 180, 0);
    rect(-4, -veg.size / 2 + 2, 8, 12, 4);
  } else if (veg.type === "eggplant") {
    fill(128, 0, 128);
    ellipse(0, 0, veg.size * 0.7, veg.size);
    fill(0, 180, 0);
    rect(-3, -veg.size / 2 + 8, 6, 10, 3);
  }
  pop();
}

function checkCollect() {
  let allCollected = true;
  for (let veg of vegetais) {
    if (!veg.collected) {
      allCollected = false;
      let fx = fazendeiro.x + fazendeiro.size / 2;
      let fy = fazendeiro.y + fazendeiro.size / 2;
      let vx = veg.x + veg.size / 2;
      let vy = veg.y + veg.size / 2;
      let d = dist(fx, fy, vx, vy);
      if (d < fazendeiro.size / 2 + veg.size / 2 - 7) {
        veg.collected = true;
        score += 1;
      }
    }
  }

  // Se todos coletados, gera nova rodada
  if (allCollected) {
    resetarVegetais();
  }
}

// Resetar vegetais
function resetarVegetais() {
  vegetais = [];
  for (let i = 0; i < VEGETABLES_PER_ROUND; i++) {
    vegetais.push({
      x: random(50, width - 70),
      y: random(80, height - 70),
      size: 36,
      collected: false,
      type: random(["tomato", "carrot", "eggplant"]),
    });
  }
}

//Depois de pressionado botão start inicia cronômetro
function atualizaTempo() {
  tempo = 60 - int((millis() - startTime) / 1000);
  if (tempo < 0) {
    tempo = 0;
  }
}

//Se tempo igual 0 fim de jogo
function verificaGameOver() {
  if (tempo <= 0) {
    fazendeiro.visible = false;
    gameState = "gameover";
  }
}
