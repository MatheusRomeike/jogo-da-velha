# Jogo da Velha

## Pontos de Melhoria

### 1. Separação de Responsabilidades
**Justificativa:** O código JavaScript atual mistura a lógica da interface do usuário com a lógica de negócios, tornando-o difícil de manter e testar.  
**Padrão de Projeto:** MVC (Model-View-Controller)  
**Implementação:** Separe a lógica de controle e manipulação de dados em um objeto de controlador, mantendo a interface do usuário em um módulo separado.
- **Model:** Responsável por armazenar o estado do jogo e a lógica central, como alternância de jogadores e checagem de vitória.
- **View:** Manipula a exibição e atualizações na interface do usuário, separando a interação com o DOM da lógica de jogo.
- **Controller:** Interpreta ações do usuário, manipulando a lógica do jogo e atualizando a interface.
### Protótipo de Código:
```javascript
// model.js
class Game {
    constructor() {
        this.board = Array(9).fill(null);
        this.currentPlayer = Player.X;
        this.winner = null;
    }

    play(position) {
        if (!this.board[position] && !this.winner) {
            this.board[position] = this.currentPlayer;
            this.winner = this.checkWinner();
            if (!this.winner) this.currentPlayer = this.currentPlayer === Player.X ? Player.O : Player.X;
        }
    }

    checkWinner() {
        const winPatterns = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],
            [0, 3, 6], [1, 4, 7], [2, 5, 8],
            [0, 4, 8], [2, 4, 6]
        ];
        for (const [a, b, c] of winPatterns) {
            if (this.board[a] && this.board[a] === this.board[b] && this.board[a] === this.board[c]) {
                return this.board[a];
            }
        }
        return this.board.includes(null) ? null : 'Draw';
    }

    reset() {
        this.board = Array(9).fill(null);
        this.currentPlayer = Player.X;
        this.winner = null;
    }
}
```

```javascript
// view.js
class GameView {
    constructor(controller) {
        this.controller = controller;
        this.cells = document.querySelectorAll('.cell');
        this.message = document.querySelector('.message');
        this.restartButton = document.querySelector('.restart');

        this.cells.forEach((cell, index) => {
            cell.addEventListener('click', () => this.controller.handleCellClick(index));
        });
        this.restartButton.addEventListener('click', () => this.controller.handleRestart());
    }

    update(board, currentPlayer, winner) {
        this.cells.forEach((cell, index) => {
            cell.textContent = board[index];
        });
        this.message.textContent = winner 
            ? (winner === 'Draw' ? 'It\'s a draw!' : `Player ${winner} wins!`)
            : `Player ${currentPlayer}'s turn`;
    }
}
```

```javascript
// controller.js
class GameController {
    constructor(model, view) {
        this.model = model;
        this.view = view;
    }

    handleCellClick(position) {
        this.model.play(position);
        this.view.update(this.model.board, this.model.currentPlayer, this.model.winner);
    }

    handleRestart() {
        this.model.reset();
        this.view.update(this.model.board, this.model.currentPlayer, this.model.winner);
    }
}

const game = new Game();
const controller = new GameController(game, new GameView(controller));
controller.view.update(game.board, game.currentPlayer, game.winner);

```
---

### 2. Melhorar a Legibilidade do Código com Observer
**Justificativa:** O uso de muitos IDs de elementos e a manipulação direta do DOM tornam o código confuso.  
**Padrão de Projeto:** Observer  
**Implementação:** Utilize um padrão Observer para atualizar a interface do usuário sempre que o estado do jogo mudar, automaticamente.
### Protótipo de Código:
```javascript
// game.js (Model)
class Game {
    constructor() {
        this.board = Array(9).fill(null);
        this.currentPlayer = Player.X;
        this.winner = null;
        this.subscribers = [];
    }

    subscribe(observer) {
        this.subscribers.push(observer);
    }

    notify() {
        this.subscribers.forEach(observer => observer.update(this));
    }

    play(position) {
        if (!this.board[position] && !this.winner) {
            this.board[position] = this.currentPlayer;
            this.winner = this.checkWinner();
            if (!this.winner) this.currentPlayer = this.currentPlayer === Player.X ? Player.O : Player.X;
            this.notify();  // Notifica as views sobre a mudança de estado
        }
    }

    checkWinner() {
        const winPatterns = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],
            [0, 3, 6], [1, 4, 7], [2, 5, 8],
            [0, 4, 8], [2, 4, 6]
        ];
        for (const [a, b, c] of winPatterns) {
            if (this.board[a] && this.board[a] === this.board[b] && this.board[a] === this.board[c]) {
                return this.board[a];
            }
        }
        return this.board.includes(null) ? null : 'Draw';
    }

    reset() {
        this.board = Array(9).fill(null);
        this.currentPlayer = Player.X;
        this.winner = null;
        this.notify();
    }
}

// gameView.js (View)
class GameView {
    constructor(game) {
        this.game = game;
        this.cells = document.querySelectorAll('.cell');
        this.message = document.querySelector('.message');
        this.restartButton = document.querySelector('.restart');
        
        this.game.subscribe(this);  // Inscreve a view para receber atualizações do game

        this.cells.forEach((cell, index) => {
            cell.addEventListener('click', () => this.game.play(index));
        });
        this.restartButton.addEventListener('click', () => this.game.reset());
    }

    update(game) {
        // Atualiza as células com o estado atual do tabuleiro
        this.cells.forEach((cell, index) => {
            cell.textContent = game.board[index];
        });

        // Exibe uma mensagem sobre o estado atual (turno do jogador ou resultado)
        this.message.textContent = game.winner 
            ? (game.winner === 'Draw' ? 'It\'s a draw!' : `Player ${game.winner} wins!`)
            : `Player ${game.currentPlayer}'s turn`;
    }
}

// main.js
const game = new Game();
const gameView = new GameView(game);


```
---

### 3. Validação de Vitória
**Justificativa:** A lógica de validação de vitória está muito acoplada e não é fácil de entender ou modificar.  
**Padrão de Projeto:** Strategy  
**Implementação:** Separar a lógica de vitória em uma classe WinValidator permite testar e modificar a lógica de vitória de forma independente.
### Protótipo de Código:
```javascript
class WinValidator {
    constructor() {
        this.winPatterns = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],
            [0, 3, 6], [1, 4, 7], [2, 5, 8],
            [0, 4, 8], [2, 4, 6]
        ];
    }

    validate(board) {
        for (const [a, b, c] of this.winPatterns) {
            if (board[a] && board[a] === board[b] && board[a] === board[c]) {
                return board[a];
            }
        }
        return board.includes(null) ? null : 'Draw';
    }
}

const validator = new WinValidator();
const result = validator.validate(board);


```
---

### 4. Uso de Enums para Jogadores
**Justificativa:** Strings “mágicas” para representar jogadores são propensas a erros. Usar um Object.freeze cria uma enumeração clara e imutável em JavaScript. 
**Padrão de Projeto:** Enum  
**Implementação:** Em JavaScript, podemos usar Object.freeze() para criar uma enumeração segura.
### Protótipo de Código:
```javascript
const Player = Object.freeze({
    X: 'X',
    O: 'O'
});

let currentPlayer = Player.X;

```
---

### 5. Música e Efeitos Sonoros
**Justificativa:** Evita múltiplas instâncias de gerenciadores de som e facilita o controle de efeitos sonoros.
**Padrão de Projeto:** Singleton  
**Implementação:** Object.freeze garante que uma instância única seja usada, controlando o acesso ao som.
### Protótipo de Código:
```javascript
class SoundManager {
    constructor() {
        if (!SoundManager.instance) {
            this.audio = new Audio();
            SoundManager.instance = this;
        }
        return SoundManager.instance;
    }

    playSound(soundFile) {
        this.audio.src = soundFile;
        this.audio.play();
    }
}

const soundManager = Object.freeze(new SoundManager());
soundManager.playSound('path/to/sound.mp3');

```
