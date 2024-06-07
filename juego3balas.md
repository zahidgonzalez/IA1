## aqui se declaran las variables a utilizar las dimensiones de la pantalla, las 3 balas y las 3 naves, asi como los movimientos tanto como del mono como los de la bala
```
var w = 800;
var h = 400;
var jugador;
var fondo;

var bala, balaD = false, nave;
var bala2, balaD2 = false, nave2;
var bala3, balaD3 = false, nave3;

var salto;
var avanza;
var retrocede;
var menu;

var velocidadBala;
var despBala, despBala2, despBala3;
var velocidadBala2;
var velocidadBala3;
var despBalaHorizontal3;
var despBalaVertical3;
var estatusAire;
var estatuSuelo;
var estatusDerecha;
var estatusIzquierda;

var nnNetwork, nnEntrenamiento, nnSalida, datosEntrenamiento = [];
var modoAuto = false, eCompleto = false;

var maxDerecha = 118; // Límite de desplazamiento hacia la derecha (50 inicial + 68)
var maxIzquierda = 50; // Límite de desplazamiento hacia la izquierda (50 inicial)

var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render: render });
```
## La función preload() es una función utilizada  para cargar todos los recursos necesarios para el juego antes de que comience.
```
function preload() {
    juego.load.image('fondo', 'assets/game/fondonieve.jpg');
    juego.load.spritesheet('mono', 'assets/sprites/altair.png', 32, 48);
    juego.load.image('nave', 'assets/game/ufo.png');
    juego.load.image('bala', 'assets/sprites/bala.png');
    juego.load.image('menu', 'assets/game/menu.png');
}
```
## en esta parte se crean o se le ingresan las caracteristas a las variables ya declaradas
### esta primer parte tiene que ver mas con el escenario ya que se le asigna la gravedad y los fotogramas
```
function create() {
    juego.physics.startSystem(Phaser.Physics.ARCADE);
    juego.physics.arcade.gravity.y = 800;
    juego.time.desiredFps = 30;
     fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');
```
### en esta segunda parte es como que los valores de donde se inicialaran los objetos
```
   
    nave = juego.add.sprite(w - 100, h - 70, 'nave');
    bala = juego.add.sprite(w - 100, h, 'bala');
    nave2 = juego.add.sprite(w - 800, h - 400, 'nave');
    bala2 = juego.add.sprite(w - 760, h - 380, 'bala');
    nave3 = juego.add.sprite(w - 100, h - 400, 'nave');
    bala3 = juego.add.sprite(w - 100, h - 370, 'bala');
    jugador = juego.add.sprite(50, h, 'mono');
```
### en esta tercera parte se le agregan las funciones de movimiento al jugador
```
    juego.physics.enable(jugador);
    jugador.body.collideWorldBounds = true;
    var corre = jugador.animations.add('corre', [8, 9, 10, 11]);
    jugador.animations.play('corre', 10, true);
```
### e esta cuarta parte se le asignas como tipo la fisica a la nave como a la bala en especifico a esta ultima se le indica que colisione con los limites del fondo
```
    juego.physics.enable(bala);
    bala.body.collideWorldBounds = true;
    juego.physics.enable(nave);
    nave.body.collideWorldBounds = true;
    juego.physics.enable(bala2);
    bala2.body.collideWorldBounds = true;
    juego.physics.enable(bala3);
    bala3.body.collideWorldBounds = true;
```
### en esta quinta parte basicamente es el menu de pausa y sus funciones
```
    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' });
    pausaL.inputEnabled = true;
    pausaL.events.onInputUp.add(pausa, self);
    juego.input.onDown.add(mPausa, self);
```
### en esta sexta parte se especifican la funcionalidad de las teclas de la pc para el juego
```
    salto = juego.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR);
    avanza = juego.input.keyboard.addKey(Phaser.Keyboard.RIGHT);
    retrocede = juego.input.keyboard.addKey(Phaser.Keyboard.LEFT); // Nueva tecla para retroceder
```
### aqui se crea la red neuronal junto con su model de entrenamiento
```
    nnNetwork = new synaptic.Architect.Perceptron(6, 5, 5, 5, 4);
    nnEntrenamiento = new synaptic.Trainer(nnNetwork);
}
```
### rate: Es la tasa de aprendizaje, que controla qué tan rápido se ajustan los pesos de la red durante el entrenamiento.
### iterations: Es el número de iteraciones de entrenamiento que se realizarán.
### shuffle: Indica si se deben mezclar los datos de entrenamiento antes de cada iteración.
```
function enRedNeural() {
    nnEntrenamiento.train(datosEntrenamiento, { rate: 0.0003, iterations: 10000, shuffle: true });
}
```
### esta parte imprime en consola los valores de entrada que recibe la función. Estos valores son estados del entorno del juego que se están utilizando como entrada para la red neuronal.
```
function datosDeEntrenamiento(param_entrada) {
    console.log("Entrada", param_entrada[0] + " " + param_entrada[1] + " " + param_entrada[2] + " " + param_entrada[3] + " " + param_entrada[4] + " " + param_entrada[5]);
    ```
### primeramente se utiliza la red neuronal despues aas salidas de la red neuronal (nnSalida) se redondean multiplicando por 100 para expresarlas como porcentajes
      ```
    nnSalida = nnNetwork.activate(param_entrada);
    var aire = Math.round(nnSalida[0] * 100);
    var piso = Math.round(nnSalida[1] * 100);
    var derecha = Math.round(nnSalida[2] * 100);
    var izquierda = Math.round(nnSalida[3] * 100);
      ```
### Imprime por consola los resultados del procesamiento de la red neuronal, mostrando el porcentaje de probabilidad calculado para cada posible estado (aire, suelo, derecha, izquierda).
      ```
    console.log(" En el Aire %: " + aire
        + "\n En el suelo %: " + piso
        + "\n En la derecha %: " + derecha
        + "\n En la izquierda %: " + izquierda
    );
     ```
### Evalúa si la salida para "derecha" es mayor o igual que la salida para "izquierda" y lo imprime por consola. Esto sirve para tomar decisiones en el juego basadas en la salida de la red neuronal.
       ``` 
    console.log("OUTPUTS: " + nnSalida[2] >= nnSalida[3])
    return nnSalida[2] >= nnSalida[3];
}
```
### Esta función parece estar diseñada para evaluar la salida de una red neuronal después de activarla con ciertos datos de entrada (param_entrada), y tomar una decisión basada en esa salida.
```
function EntrenamientoSalto(param_entrada) {
    console.log("Entrada", param_entrada[0] + " " + param_entrada[1]);
    nnSalida = nnNetwork.activate(param_entrada);
    var aire = Math.round(nnSalida[0] * 100);
    var piso = Math.round(nnSalida[1] * 100);
    console.log(" En el Aire %: " + aire
        + "\n En el suelo %: " + piso
    );
    return nnSalida[0] >= nnSalida[1];
}
```
### las siguientes dos funciones están diseñadas para manejar la pausa del juego y la interacción con un menú de pausa como la funcionalidad de resetear los valores cada que se inicia un nuevo juego.
```
function pausa() {
    juego.paused = true;
    menu = juego.add.sprite(w / 2, h / 2, 'menu');
    menu.anchor.setTo(0.5, 0.5);
}

function mPausa(event) {
    if (juego.paused) {
        var menu_x1 = w / 2 - 270 / 2, menu_x2 = w / 2 + 270 / 2,
            menu_y1 = h / 2 - 180 / 2, menu_y2 = h / 2 + 180 / 2;

        var mouse_x = event.x,
            mouse_y = event.y;

        if (mouse_x > menu_x1 && mouse_x < menu_x2 && mouse_y > menu_y1 && mouse_y < menu_y2) {
            if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 && mouse_y <= menu_y1 + 90) {
                eCompleto = false;
                datosEntrenamiento = [];
                modoAuto = false;
            } else if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 + 90 && mouse_y <= menu_y2) {
                if (!eCompleto) {
                    console.log("", "Entrenamiento " + datosEntrenamiento.length + " valores");
                    enRedNeural();
                    eCompleto = true;
                }
                modoAuto = true;
            }

            resetVariables();
            resetVariables2();
            resetVariables3();
            resetPlayer();
            juego.paused = false;
            menu.destroy();
        }
    }
}
```
### Las siguientes funciones son las que se encargargan de restablecer los valores de todo los objetos asi como sus posiciones estas se mandan a llamar en la parte de arriba
```
function resetVariables() {
    bala.body.velocity.x = 0;
    bala.position.x = w - 100;
    balaD = false;
}

function resetVariables2() {
    bala2.body.velocity.y = -270;
    bala2.position.y = h - 400;
    balaD2 = false;
}

function resetVariables3() {
    bala3.body.velocity.y = -270;
    bala3.body.velocity.x = 0;
    bala3.position.x = w - 100;
    bala3.position.y = h - 500;
    balaD3 = false;
}

function resetPlayer() {
    jugador.position.x = 50;
}
```
### Estas funciones controlan los movimientos del jugador
```
function saltar() {
    jugador.body.velocity.y = -270;
}

function correr() {
    jugador.body.velocity.x = 200;
}

function correrAtras() {
    jugador.body.velocity.x = -75;
}

function Detenerse() {
    jugador.body.velocity.x = 0;
}
```
### Esta función se encarga de actualizar la lógica del juego, verificar colisiones, manejar entradas de usuario y realizar otras operaciones necesarias para mantener el estado del juego y la interacción del jugador. se maneja si el juego esta en modo automatico o no si es falso entonces procesede a leer los datos de entrada si es true tomara los datos de entrenamiento correspondientes en cada funcion. ademas si es falso los datos ingresados desde las entradas seran tomadas en cuenta para el entrenamiento
```
function update() {
    fondo.tilePosition.x -= 1;

    juego.physics.arcade.collide(nave, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala2, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala3, jugador, colisionH, null, this);

    estatuSuelo = 1;
    estatusAire = 0;

    estatusDerecha = 0;
    estatusIzquierda = 1;

    if (!jugador.body.onFloor()) {
        estatusAire = 1;
        estatuSuelo = 0;
    }
    if (jugador.body.velocity.x > 0) {
        estatusDerecha = 1;
        estatusIzquierda = 0;
        console.log("movimiento a la derecha");
    } else if (jugador.body.velocity.x < 0) {
        estatusDerecha = 0;
        estatusIzquierda = 1;
        console.log("movimiento a la izquierda");
    }

    despBala = Math.floor(jugador.position.x - bala.position.x);
    despBala2 = Math.floor(jugador.position.x - bala2.position.x);
    despBalaHorizontal3 = Math.floor(jugador.position.x - bala3.position.x);
    despBalaVertical3 = Math.floor(jugador.position.y - bala3.position.y);
    despBala3 = Math.floor(despBalaHorizontal3 + despBalaVertical3);

    if (modoAuto == false && salto.isDown && !estatusAire) {
        if (jugador.body.velocity.x <= 0) {
            jugador.body.velocity.x = 150;
            saltar();
            correr();
        } else {
            saltar();
            Detenerse();
        }
    }
    if (modoAuto == false && avanza.isDown && jugador.body.onFloor()) {
        correr();
    }

    if (modoAuto == false && retrocede.isDown && jugador.body.onFloor()) { // Movimiento hacia la izquierda
        correrAtras();
    }

    if (modoAuto == false && jugador.body.onFloor() && jugador.position.x >= maxDerecha) {
        correrAtras();
    }

    if (modoAuto == false && !avanza.isDown && !retrocede.isDown && jugador.body.onFloor()) {
        Detenerse();
    }

    // Limitar el movimiento del jugador a la derecha y regresarlo si alcanza el límite
    if (jugador.position.x > maxDerecha) {
        jugador.position.x = maxDerecha;
        // Solo retrocede si el jugador está en el suelo
        if (jugador.body.onFloor()) {
            correrAtras();
        }
    }

    // Limitar el movimiento del jugador hacia atrás para que no pase su posición inicial
    if (jugador.position.x < maxIzquierda) {
        jugador.position.x = maxIzquierda;
        Detenerse();
    }

    if (modoAuto == true && bala.position.x > 0 && jugador.body.onFloor()) {
        console.log("Saltar: " + EntrenamientoSalto([despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3]));
        if (EntrenamientoSalto([despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3])) {
            if (jugador.body.velocity.x <= 0) {
                jugador.body.velocity.x = 150;
                console.log("Salta");
                saltar();
                correr();
            } else {
                saltar();
                Detenerse();
            }
        }
        console.log("Corre: " + datosDeEntrenamiento([despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3]));
        if (datosDeEntrenamiento([despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3])) {
            console.log("corre");
            correr();
        } else if (jugador.body.onFloor() && jugador.position.x >= 250) {
            Detenerse();
            correrAtras();
        }
    }

    if (balaD == false) {
        disparo();
    }

    if (balaD2 == false) {
        disparo2();
    }

    if (balaD3 == false) {
        disparo3();
    }

    if (bala.position.x <= 0) {
        resetVariables();
    }

    if (bala2.position.y >= 355) {
        resetVariables2();
    }

    if (bala3.position.x <= 0 || bala3.position.y >= 355) {
        resetVariables3();
    }

    if (modoAuto == false && bala.position.x > 0) {
        datosEntrenamiento.push({
            'input': [despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3],
            'output': [estatusAire, estatuSuelo, estatusDerecha, estatusIzquierda]
        });
    }
}
```
### en esta ultima parte se utiliza la logica para establecer el direccionamiento de la bala como se es deseado, dependiendo de que punto del eje hacia que punto es si los valores son negativos o positivos
```
function disparo() {
    velocidadBala = -1 * velocidadRandom(200, 540);
    bala.body.velocity.y = 0;
    bala.body.velocity.x = velocidadBala;
    balaD = true;
}

function disparo2() {
    velocidadBala2 = -1 * velocidadRandom(200, 400);
    bala2.body.velocity.y = 0;
    balaD2 = true;
}

function disparo3() {
    velocidadBala3 = -1 * velocidadRandom(200, 370);
    bala3.body.velocity.y = 0;
    bala3.body.velocity.x = 1.60 * velocidadBala3;
    balaD3 = true;
}

function colisionH() {
    pausa();
}

function velocidadRandom(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

function render() {

}
```