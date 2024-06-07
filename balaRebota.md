### en esta primera parte se declaran las variables a utilizar en este caso se inicializan algunas
```
var w = 450;
var h = 400;
var jugador, fondo, bala;
var cursors, menu;
var nnNetwork, nnTrainer, nnOutput, trainingData = [];
var autoMode = false;
var trainingComplete = false;
var trainingGames = 3;
var currentGame = 0;
var PX = 200, PY = 200;
var REVERTIRV = false, REVERTIRH = false;
var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render: render,});
```
### en la siguiente funcion se especifica las rutas de lo que le dara forma o imagen a los objetos dentro de juego
```
function preload() {
  juego.load.image('fondo', 'assets/game/fondonieve.jpg');
  juego.load.spritesheet('mono', 'assets/sprites/altair.png', 32, 48);
  juego.load.image('menu', 'assets/game/menu.png');
  juego.load.image('bala', 'assets/sprites/bala.png');
}
```
### aqui se inicia el sistema de físicas la cual permite aplicar colisiones, gravedad y etc. se establece la gravedad en el eje Y a cero, y se estableceel control de la velocidad a la que se actualiza la lógica del juego y se renderiza cada fotograma.
```
function create() {
  juego.physics.startSystem(Phaser.Physics.ARCADE);
  juego.physics.arcade.gravity.y = 0;
  juego.time.desiredFps = 30;

  fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');
```
### aqui se establecen los movimientos de mono
```
  jugador = juego.add.sprite(w / 2, h / 2, 'mono');
  juego.physics.enable(jugador);
  jugador.body.collideWorldBounds = true;
  jugador.animations.add('corre', [8, 9, 10, 11]);
  jugador.animations.play('corre', 10, true);
```
### aqui se establece los  movimientos de la bala asi como la  funcion para que esta rebote
```
  bala = juego.add.sprite(0, 0, 'bala');
  juego.physics.enable(bala);
  bala.body.collideWorldBounds = true;
  bala.body.bounce.set(1);
  setRandomBalaVelocity();

  pausaL = juego.add.text(w - 100, 20, 'Pausa', {
    font: '20px Arial',
    fill: '#fff',
  });
  pausaL.inputEnabled = true;
  pausaL.events.onInputUp.add(pausar, self);
  juego.input.onDown.add(manejarPausa, self);
```
```
  cursors = juego.input.keyboard.createCursorKeys();
```
### se crea la red neuronal con 9 neuronas de entrada, 10 neuronas en una capa oculta y 4 neuronas de salida
```
  nnNetwork = new synaptic.Architect.Perceptron(9, 10, 4); 
  nnTrainer = new synaptic.Trainer(nnNetwork);
}
```
### aqui se crea la logica de la bala para darle una trayectoria incial aleatoria
```
function setRandomBalaVelocity() {
  var baseSpeed = 550;
  var angle = juego.rnd.angle();
  bala.body.velocity.set(
    Math.cos(angle) * baseSpeed,
    Math.sin(angle) * baseSpeed
  );
}
```
### La función update() sirve para manejar el desplazamiento del fondo, controlar el movimiento del jugador ya sea de manera manual o automática, y detectar colisiones entre la bala y el jugador.
```
function update() {
  fondo.tilePosition.x -= 1;

  if (!autoMode) {
    manejarMovimientoManual();
  } else {
    if (trainingData.length > 0) {
      manejarMovimientoAutomatico();
    } else {
      jugador.body.velocity.x = 0;
      jugador.body.velocity.y = 0;
    }
  }

  juego.physics.arcade.collide(bala, jugador, colisionar, null, this);
}
```
### en esta parte se tiene un control y registro de los movimiento y velocidad del jugador en conjunto con las teclas presionadas
```
function manejarMovimientoManual() {
  var prevX = jugador.body.velocity.x;
  var prevY = jugador.body.velocity.y;

  jugador.body.velocity.x = 0;
  jugador.body.velocity.y = 0;

  var moveLeft = cursors.left.isDown ? 1 : 0;
  var moveRight = cursors.right.isDown ? 1 : 0;
  var moveUp = cursors.up.isDown ? 1 : 0;
  var moveDown = cursors.down.isDown ? 1 : 0;

  if (moveLeft) {
    jugador.body.velocity.x = -300;
  } else if (moveRight) {
    jugador.body.velocity.x = 300;
  }

  if (moveUp) {
    jugador.body.velocity.y = -300;
  } else if (moveDown) {
    jugador.body.velocity.y = 300;
  }

  if (jugador.body.velocity.x !== prevX || jugador.body.velocity.y !== prevY) {
    registrarDatosEntrenamiento();
  }

  console.log('-------------------------------------------');
  console.log('Manual - Movimiento:');
  console.log(`Izquierda: ${moveLeft}`);
  console.log(`Derecha: ${moveRight}`);
  console.log(`Arriba: ${moveUp}`);
  console.log(`Abajo: ${moveDown}`);
}
```

##calcula la diferencia de posición entre la bala y el jugador
```
function manejarMovimientoAutomatico() {
  var dx = bala.x - jugador.x;
  var dy = bala.y - jugador.y;
```
### Calcula la distancia euclidiana entre la bala y el jugador
 ` var distancia = Math.sqrt(dx * dx + dy * dy);`

### Crea un array con los datos de entrada para la red neuronal
```
  var input = [
    dx,                    
    dy,                    
    distancia,             
    jugador.x,            
    jugador.y,            
    cursors.left.isDown,   
    cursors.right.isDown,  
    cursors.up.isDown,    
    cursors.down.isDown    
  ];
```
### se Activa la red neuronal con los datos de entrada y se determinan los movimientos basado en las salidas de la red
```
  nnOutput = nnNetwork.activate(input);
  var moveLeft = nnOutput[0] > 0.5 ? 1 : 0;
  var moveRight = nnOutput[1] > 0.5 ? 1 : 0;
  var moveUp = nnOutput[2] > 0.5 ? 1 : 0;
  var moveDown = nnOutput[3] > 0.5 ? 1 : 0;
```
### Aplica la velocidad al jugador en el eje x y en el eje y según las decisiones de la red neuronal
```
  jugador.body.velocity.x = (moveRight - moveLeft) * 300; // Movimiento horizontal
  jugador.body.velocity.y = (moveDown - moveUp) * 300;   // Movimiento vertical
```
### Mensajes de depuración en la consola para mostrar el estado del movimiento automático
```
  console.log('-------------------------------------------');
  console.log(`Auto - Movimiento:`);
  console.log(`Izquierda: ${moveLeft}`);
  console.log(`Derecha: ${moveRight}`);
  console.log(`Arriba: ${moveUp}`);
  console.log(`Abajo: ${moveDown}`);
}
```
### primero se asegura que el juego no esté en modo automático. para que solo se registrarán datos cuando el jugador esté controlando manualmente el movimiento una vez hecho el entrenaminetos estos datos se almecenan.
```
function registrarDatosEntrenamiento() {
  if (!autoMode && bala.position.x > 0) {
    var dx = bala.x - jugador.x;
    var dy = bala.y - jugador.y;
    var distancia = Math.sqrt(dx * dx + dy * dy);
    var datosIzquierda = cursors.left.isDown ? 1 : 0;
    var datosDerecha = cursors.right.isDown ? 1 : 0;
    var datosArriba = cursors.up.isDown ? 1 : 0;
    var datosAbajo = cursors.down.isDown ? 1 : 0;

    trainingData.push({
      'input': [dx, dy, distancia, jugador.x, jugador.y, datosIzquierda, datosDerecha, datosArriba, datosAbajo],
      'output': [datosIzquierda, datosDerecha, datosArriba, datosAbajo]
    });

    console.log('Datos de Entrenamiento Registrados');
  }
}
```
### en la ultima parte son caracteristicas y funciones del menu de pausa asi como el reset del juego y sus objetos
```
function colisionar() {
  autoMode = true;
  pausar();
}

function pausar() {
  juego.paused = true;
  menu = juego.add.sprite(w / 2, h / 2, 'menu');
  menu.anchor.setTo(0.5, 0.5);
}

function manejarPausa(event) {
  if (juego.paused) {
    var menu_x1 = w / 2 - 270 / 2, menu_x2 = w / 2 + 270 / 2,
        menu_y1 = h / 2 - 180 / 2, menu_y2 = h / 2 + 180 / 2;
    var mouse_x = event.x, mouse_y = event.y;

    if (mouse_x > menu_x1 && mouse_x < menu_x2 && mouse_y > menu_y1 && mouse_y < menu_y2) {
      if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 && mouse_y <= menu_y1 + 90) {
        entrenamientoCompleto = false;
        trainingData = [];
        autoMode = false;
      } else if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 + 90 && mouse_y <= menu_y2) {
        if (!trainingComplete && trainingData.length > 0) {
          nnTrainer.train(trainingData, { rate: 0.0003, iterations: 10000, shuffle: true });
          trainingComplete = true;
        }
        autoMode = true;
      }
      menu.destroy();
      resetGame();
      juego.paused = false;
    }
  }
}

function resetGame() {
  jugador.x = w / 2;
  jugador.y = h / 2;
  jugador.body.velocity.x = 0;
  jugador.body.velocity.y = 0;
  bala.x = 0;
  bala.y = 0;
  setRandomBalaVelocity();
}

function render() {}
```