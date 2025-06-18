# agrinho-2025
repositorio destinado ao projeto agrinho 2025
projeto desenvolvido com IA GEMINI, prompt: desenvolva um jogo no p5.js onde uma colheitadeira coleta trigo e após isso ele despeja no celeiro. 
// Variáveis do jogo
let trator;
let campoTrigo = [];
let celeiro;
let estadoJogo = 'jogando'; // 'jogando', 'ganhou', 'perdeu'
let pontuacao = 0;

// Imagens (substitua pelos seus próprios arquivos)
let imgTrator;
let imgTrigo;
let imgCeleiro;

function preload() {
  // Carregue suas imagens aqui
  // imgTrator = loadImage('assets/trator.png');
  // imgTrigo = loadImage('assets/trigo.png');
  // imgCeleiro = loadImage('assets/celeiro.png');
}

function setup() {
  createCanvas(800, 600);
  // Crie o trator
  trator = new Trator(50, height - 100);

  // Crie alguns trigais no campo
  for (let i = 0; i < 10; i++) {
    campoTrigo.push(new Trigo(random(width), random(height / 2)));
  }

  // Crie o celeiro
  celeiro = new Celeiro(width - 150, height - 200);
}

function draw() {
  background(135, 206, 235); // Céu azul

  if (estadoJogo === 'jogando') {
    // Desenha o campo (grama)
    fill(124, 252, 0); // Verde grama
    rect(0, height / 2, width, height / 2);

    // Atualiza e desenha o trator
    trator.mover();
    trator.desenhar();

    // Atualiza e desenha os trigais
    for (let i = campoTrigo.length - 1; i >= 0; i--) {
      campoTrigo[i].desenhar();
      // Verifica colisão do trator com o trigo
      if (trator.colideCom(campoTrigo[i])) {
        trator.colherTrigo(campoTrigo[i]);
        campoTrigo.splice(i, 1); // Remove o trigo colhido
        pontuacao++;
      }
    }

    // Desenha o celeiro
    celeiro.desenhar();

    // Se o trator tiver trigo, ele vai para o celeiro
    if (trator.temTrigo) {
      trator.irParaCeleiro(celeiro);
      if (dist(trator.x, trator.y, celeiro.x + celeiro.largura / 2, celeiro.y + celeiro.altura / 2) < 20) {
        trator.despejarTrigo();
        // Crie novos trigais para continuar o ciclo
        if (campoTrigo.length < 5) { // Mantém um mínimo de trigais no campo
            for (let i = 0; i < 5; i++) {
                campoTrigo.push(new Trigo(random(width), random(height / 2)));
            }
        }
      }
    }


    // Exibe a pontuação
    fill(0);
    textSize(24);
    text('Trigo Coletado: ' + pontuacao, 20, 30);

    // Lógica para terminar o jogo (opcional)
    // if (pontuacao >= 50) {
    //   estadoJogo = 'ganhou';
    // }
  } else if (estadoJogo === 'ganhou') {
    // Tela de vitória
    fill(0, 200, 0);
    textSize(48);
    textAlign(CENTER, CENTER);
    text('VOCÊ GANHOU!', width / 2, height / 2);
  } else if (estadoJogo === 'perdeu') {
    // Tela de derrota
    fill(200, 0, 0);
    textSize(48);
    textAlign(CENTER, CENTER);
    text('VOCÊ PERDEU!', width / 2, height / 2);
  }
}

// Classe Trator
class Trator {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.largura = 80;
    this.altura = 50;
    this.velocidade = 3;
    this.temTrigo = false;
    this.destinoX = x; // Usado para navegação automática
    this.destinoY = y; // Usado para navegação automática
  }

  desenhar() {
    fill(255, 0, 0); // Vermelho
    // if (imgTrator) {
    //   image(imgTrator, this.x, this.y, this.largura, this.altura);
    // } else {
      rect(this.x, this.y, this.largura, this.altura);
      fill(0); // Rodas
      ellipse(this.x + 15, this.y + this.altura, 20, 20);
      ellipse(this.x + this.largura - 15, this.y + this.altura, 20, 20);
    // }
  }

  mover() {
    if (keyIsDown(LEFT_ARROW)) {
      this.x -= this.velocidade;
      this.destinoX = this.x; // Cancela navegação automática
    }
    if (keyIsDown(RIGHT_ARROW)) {
      this.x += this.velocidade;
      this.destinoX = this.x; // Cancela navegação automática
    }
    if (keyIsDown(UP_ARROW)) {
      this.y -= this.velocidade;
      this.destinoY = this.y; // Cancela navegação automática
    }
    if (keyIsDown(DOWN_ARROW)) {
      this.y += this.velocidade;
      this.destinoY = this.y; // Cancela navegação automática
    }

    // Limita o trator dentro da tela
    this.x = constrain(this.x, 0, width - this.largura);
    this.y = constrain(this.y, height / 2, height - this.altura); // Mantém o trator na parte de baixo da tela
  }

  colideCom(trigo) {
    return this.x < trigo.x + trigo.tamanho &&
           this.x + this.largura > trigo.x &&
           this.y < trigo.y + trigo.tamanho &&
           this.y + this.altura > trigo.y;
  }

  colherTrigo(trigo) {
    if (!this.temTrigo) { // Só pode colher um trigo por vez
      this.temTrigo = true;
      console.log("Trigo colhido!");
    }
  }

  irParaCeleiro(celeiro) {
    // Define o destino como o centro do celeiro
    this.destinoX = celeiro.x + celeiro.largura / 2 - this.largura / 2;
    this.destinoY = celeiro.y + celeiro.altura / 2 - this.altura / 2;

    let dirX = this.destinoX - this.x;
    let dirY = this.destinoY - this.y;
    let distancia = dist(this.x, this.y, this.destinoX, this.destinoY);

    if (distancia > this.velocidade) {
      this.x += dirX / distancia * this.velocidade;
      this.y += dirY / distancia * this.velocidade;
    } else {
      this.x = this.destinoX;
      this.y = this.destinoY;
    }
  }

  despejarTrigo() {
    this.temTrigo = false;
    console.log("Trigo despejado no celeiro!");
  }
}

// Classe Trigo
class Trigo {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.tamanho = 30;
  }

  desenhar() {
    fill(255, 223, 0); // Amarelo (cor do trigo)
    // if (imgTrigo) {
    //   image(imgTrigo, this.x, this.y, this.tamanho, this.tamanho);
    // } else {
      ellipse(this.x + this.tamanho / 2, this.y + this.tamanho / 2, this.tamanho, this.tamanho);
    // }
  }
}

// Classe Celeiro
class Celeiro {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.largura = 100;
    this.altura = 80;
  }

  desenhar() {
    fill(139, 69, 19); // Marrom (cor do celeiro)
    // if (imgCeleiro) {
    //   image(imgCeleiro, this.x, this.y, this.largura, this.altura);
    // } else {
      rect(this.x, this.y, this.largura, this.altura);
      // Telhado
      triangle(this.x, this.y, this.x + this.largura / 2, this.y - 40, this.x + this.largura, this.y);
    // }
  }
}
