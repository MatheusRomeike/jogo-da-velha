# Jogo da Velha

## Pontos de Melhoria

### 1. Separação de Responsabilidades
**Justificativa:** O código JavaScript atual mistura a lógica da interface do usuário com a lógica de negócios, tornando-o difícil de manter e testar.  
**Padrão de Projeto:** MVC (Model-View-Controller)  
**Implementação:** Separe a lógica de controle e manipulação de dados em um objeto de controlador, mantendo a interface do usuário em um módulo separado.
- **Model:** Representa o estado do jogo (posição dos jogadores, vencedor, etc.)
- **View:** Responsável pela interface gráfica (HTML/CSS).
- **Controller:** Lida com a lógica do jogo (interações do usuário).
### Protótipo de Código:
```javascript
// model.js
class Game {
    constructor() {
        this.board = Array(9).fill(null);
        this.currentPlayer = 'X';
        this.winner = null;
    }

    play(position) {
        if (!this.board[position] && !this.winner) {
            this.board[position] = this.currentPlayer;
            this.winner = this.checkWinner();
            this.currentPlayer = this.currentPlayer === 'X' ? 'O' : 'X';
        }
    }

    checkWinner() {
        // lógica para verificar o vencedor
    }
}

// controller.js
const game = new Game();
function handleCellClick(position) {
    game.play(position);
    updateView();
}

// view.js
function updateView() {
    // lógica para atualizar a UI com base no estado do jogo (vitória ou velha)
}
```
---

### 2. Melhorar a Legibilidade do Código
**Justificativa:** O uso de muitos IDs de elementos e a manipulação direta do DOM tornam o código confuso.  
**Padrão de Projeto:** Observer  
**Implementação:** Utilize um padrão Observer para atualizar a interface do usuário sempre que o estado do jogo mudar.
### Protótipo de Código:
```javascript
class GameView {
    constructor(controller) {
        this.controller = controller;
        this.controller.subscribe(this);
    }

    update() {
        // Atualizar a interface do usuário com base no estado do jogo (vitória ou velha)
    }
}

const gameView = new GameView(gameController);

```
---

### 3. Validação de Vitória
**Justificativa:** A lógica de validação de vitória está muito acoplada e não é fácil de entender ou modificar.  
**Padrão de Projeto:** Strategy  
**Implementação:** Separe a lógica de validação em diferentes estratégias que podem ser trocadas conforme necessário.
### Protótipo de Código:
```javascript
class WinValidator {
    validate(board) {
        // Implementar lógica de validação
    }
}

const validator = new WinValidator();
const hasWon = validator.validate(listaValores);

```
---

### 4. Uso de Enums para Jogadores
**Justificativa:** Usar strings para identificar jogadores é propenso a erros e não é tipado.  
**Padrão de Projeto:** Enum  
**Implementação:** Use enums para representar os jogadores 'X' e 'O'.
### Protótipo de Código:
```javascript
const Player = {
    X: 'X',
    O: 'O'
};

let currentPlayer = Player.X;

```
---

### 5. Música e Efeitos Sonoros
**Justificativa:** O carregamento de arquivos de áudio deve ser feito de forma a não bloquear a inicialização da aplicação.  
**Padrão de Projeto:** Singleton  
**Implementação:** Crie uma classe Singleton para gerenciar o carregamento e a reprodução de sons.
### Protótipo de Código:
```javascript
class SoundManager {
    static instance;

    constructor() {
        if (SoundManager.instance) {
            return SoundManager.instance;
        }
        this.audio = new Audio();
        SoundManager.instance = this;
    }

    playSound(soundFile) {
        this.audio.src = soundFile;
        this.audio.play();
    }
}

const soundManager = new SoundManager();
soundManager.playSound('../audio/vitoria.mp3');
```
