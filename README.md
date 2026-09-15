// ==========================================
//           TA-TE-TI CON LED RGB
//              Arduino UNO
// ==========================================

// -------- BOTONES --------

const byte botones[9] = {
  A0, A1, A2,
  A3, A4, A5,
  2, 3, 4
};

// -------- FILAS DE LOS LEDs --------

const byte filas[3] = {
  5, 6, 7
};

// -------- COLUMNAS ROJAS --------

const byte rojo[3] = {
  8, 9, 10
};

// -------- COLUMNAS AZULES --------

const byte azul[3] = {
  11, 12, 13
};


// tablero:
//
// 0 = vacío
// 1 = rojo
// 2 = azul

byte tablero[9] = {
  0, 0, 0,
  0, 0, 0,
  0, 0, 0
};


// Jugador actual
byte jugador = 1;

// Indica si terminó la partida
bool partidaTerminada = false;


// ==========================================
// SETUP
// ==========================================

void setup() {

  // Configurar botones
  for (int i = 0; i < 9; i++) {
    pinMode(botones[i], INPUT_PULLUP);
  }

  // Configurar filas
  for (int i = 0; i < 3; i++) {
    pinMode(filas[i], OUTPUT);
    digitalWrite(filas[i], HIGH);
  }

  // Configurar columnas rojas
  for (int i = 0; i < 3; i++) {
    pinMode(rojo[i], OUTPUT);
    digitalWrite(rojo[i], LOW);
  }

  // Configurar columnas azules
  for (int i = 0; i < 3; i++) {
    pinMode(azul[i], OUTPUT);
    digitalWrite(azul[i], LOW);
  }
}


// ==========================================
// LOOP
// ==========================================

void loop() {

  // Mantener los LEDs encendidos
  // mediante multiplexado
  mostrarTablero();

  // Si terminó la partida, no aceptar botones
  if (partidaTerminada) {
    return;
  }

  // Revisar los 9 botones
  for (int i = 0; i < 9; i++) {

    if (digitalRead(botones[i]) == LOW) {

      // Esperar un poquito para eliminar rebotes
      delay(30);

      if (digitalRead(botones[i]) == LOW) {

        // Solo se puede jugar en un lugar vacío
        if (tablero[i] == 0) {

          // Colocar jugador
          tablero[i] = jugador;

          // Verificar ganador
          if (hayGanador(jugador)) {

            partidaTerminada = true;

            // Mostrar ganador durante 3 segundos
            mostrarGanador(jugador);

            // Reiniciar tablero
            reiniciarJuego();

            return;
          }

          // Verificar empate
          if (hayEmpate()) {

            partidaTerminada = true;

            // Mostrar empate
            mostrarEmpate();

            // Reiniciar
            reiniciarJuego();

            return;
          }

          // Cambiar jugador
          if (jugador == 1) {
            jugador = 2;
          } else {
            jugador = 1;
          }
        }

        // Esperar a que suelten el botón
        while (digitalRead(botones[i]) == LOW) {
          mostrarTablero();
        }
      }
    }
  }
}


// ==========================================
// MOSTRAR TABLERO
// ==========================================

void mostrarTablero() {

  // Apagar todo antes de cambiar de fila
  apagarColumnas();

  for (int fila = 0; fila < 3; fila++) {

    // Activar fila
    digitalWrite(filas[fila], LOW);

    for (int columna = 0; columna < 3; columna++) {

      int posicion = fila * 3 + columna;

      if (tablero[posicion] == 1) {

        // LED ROJO
        digitalWrite(rojo[columna], HIGH);

      } else if (tablero[posicion] == 2) {

        // LED AZUL
        digitalWrite(azul[columna], HIGH);
      }

      // Mantener encendido muy poquito
      delayMicroseconds(1500);

      // Apagar columnas
      apagarColumnas();
    }

    // Desactivar fila
    digitalWrite(filas[fila], HIGH);
  }
}


// ==========================================
// APAGAR COLUMNAS
// ==========================================

void apagarColumnas() {

  for (int i = 0; i < 3; i++) {
    digitalWrite(rojo[i], LOW);
    digitalWrite(azul[i], LOW);
  }
}


// ==========================================
// COMPROBAR GANADOR
// ==========================================

bool hayGanador(byte jugador) {

  if (tablero[0] == jugador && tablero[1] == jugador && tablero[2] == jugador)
    return true;

  if (tablero[3] == jugador && tablero[4] == jugador && tablero[5] == jugador)
    return true;

  if (tablero[6] == jugador && tablero[7] == jugador && tablero[8] == jugador)
    return true;

  if (tablero[0] == jugador && tablero[3] == jugador && tablero[6] == jugador)
    return true;

  if (tablero[1] == jugador && tablero[4] == jugador && tablero[7] == jugador)
    return true;

  if (tablero[2] == jugador && tablero[5] == jugador && tablero[8] == jugador)
    return true;

  if (tablero[0] == jugador && tablero[4] == jugador && tablero[8] == jugador)
    return true;

  if (tablero[2] == jugador && tablero[4] == jugador && tablero[6] == jugador)
    return true;

  return false;
}


// ==========================================
// COMPROBAR EMPATE
// ==========================================

bool hayEmpate() {

  for (int i = 0; i < 9; i++) {

    if (tablero[i] == 0) {
      return false;
    }
  }

  return true;
}


// ==========================================
// MOSTRAR GANADOR
// ==========================================

void mostrarGanador(byte ganador) {

  unsigned long tiempo = millis();

  while (millis() - tiempo < 3000) {

    // Hacer parpadear todo el tablero
    for (int i = 0; i < 4; i++) {

      // Encender todos
      for (int j = 0; j < 9; j++) {
        tablero[j] = ganador;
      }

      for (int k = 0; k < 100; k++) {
        mostrarTablero();
      }

      // Apagar todos
      for (int j = 0; j < 9; j++) {
        tablero[j] = 0;
      }

      for (int k = 0; k < 100; k++) {
        mostrarTablero();
      }
    }
  }
}


// ==========================================
// MOSTRAR EMPATE
// ==========================================

void mostrarEmpate() {

  unsigned long tiempo = millis();

  while (millis() - tiempo < 3000) {

    // Alternar rojo y azul
    for (int j = 0; j < 9; j++) {
      tablero[j] = 1;
    }

    for (int k = 0; k < 150; k++) {
      mostrarTablero();
    }

    for (int j = 0; j < 9; j++) {
      tablero[j] = 2;
    }

    for (int k = 0; k < 150; k++) {
      mostrarTablero();
    }
  }
}


// ==========================================
// REINICIAR JUEGO
// ==========================================

void reiniciarJuego() {

  // Apagar tablero
  for (int i = 0; i < 9; i++) {
    tablero[i] = 0;
  }

  jugador = 1;
  partidaTerminada = false;

  delay(500);
}
