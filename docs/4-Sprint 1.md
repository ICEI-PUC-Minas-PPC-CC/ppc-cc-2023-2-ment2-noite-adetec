# Sprint 1

Para o sprint 1 foram feitos:

**1°)** Lista de materiais que serão necessários para a construção do projeto.

**2°)** Um código do jogo dos gênios para a mesa:

#include <LiquidCrystal.h>
#include <Adafruit_NeoPixel.h>
#include <Keypad.h>

LiquidCrystal lcd(12, 11, 5, 4, 3, 2); // Pinos do LCD
Adafruit_NeoPixel strip = Adafruit_NeoPixel(4, 10, NEO_GRB + NEO_KHZ800); // Pinos do LED RGB

int redLED = 0;
int greenLED = 1;
int blueLED = 2;
int yellowLED = 3;

int sequence[100]; // Armazena a sequência gerada aleatoriamente
int playerSequence[100]; // Armazena a sequência do jogador
int sequenceLength = 0; // Comprimento da sequência atual
int playerInputIndex = 0; // Índice para rastrear a entrada do jogador

const byte ROWS = 4; // Quatro linhas do teclado
const byte COLS = 4; // Quatro colunas do teclado
char keys[ROWS][COLS] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};
byte rowPins[ROWS] = {9, 8, 7, 6}; // Pinos das linhas conectados ao Arduino
byte colPins[COLS] = {5, 4, 3, 2}; // Pinos das colunas conectados ao Arduino
Keypad keypad = Keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS );

void setup() {
  lcd.begin(16, 2); // Inicializa o LCD com 16 colunas e 2 linhas
  strip.begin();    // Inicializa o NeoPixel
  strip.show();      // Inicializa todos os LEDs em OFF

  lcd.print("Jogo dos Genios");
  delay(2000);
  lcd.clear();
}

void loop() {
  lcd.setCursor(0, 0);
  lcd.print("Escolher opcao:");
  lcd.setCursor(0, 1);
  lcd.print("1: Aleatorio 2: Manual");

  int option = 0;
  while (option != 1 && option != 2) {
    option = readKeypad(); // Função para ler a seleção do usuário
  }

  if (option == 1) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Iniciando aleatorio...");
    playGame(true); // Inicia o jogo com cores e velocidades aleatórias
  } else if (option == 2) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Selecionar cor:");
    lcd.setCursor(0, 1);
    lcd.print("0: Vermelho 1: Verde");
    lcd.setCursor(0, 2);
    lcd.print("2: Azul 3: Amarelo");

    int selectedColor = 0;
    while (selectedColor < 0 || selectedColor > 3) {
      selectedColor = readKeypad(); // Função para ler a seleção do usuário
    }

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Iniciando manual...");
    playGame(false, selectedColor); // Inicia o jogo com cor e velocidade selecionadas manualmente
  }
}

char readKeypad() {
  char key = keypad.getKey();
  if (key) {
    return key;
  }
  return 0; // Retorna 0 se nenhum botão for pressionado
}

void playGame(bool randomSequence, int selectedColor = 0) {
  if (randomSequence) {
    // Gera uma sequência aleatória de cores e velocidades
    generateRandomSequence();
  } else {
    // Adiciona a cor selecionada manualmente à sequência
    sequence[0] = selectedColor;
    sequenceLength = 1;
  }

  // Mostra a sequência ao jogador
  showSequence();

  playerInputIndex = 0; // Reinicia o índice de entrada do jogador
  while (playerInputIndex < sequenceLength) {
    char playerChoice = readKeypad(); // Leitura da entrada do jogador

    // Verifica se a escolha do jogador está correta
    if (playerChoice == keys[0][sequence[playerInputIndex]]) {
      playerInputIndex++;
    } else {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Game Over!");
      lcd.setCursor(0, 1);
      lcd.print("Pontuacao: ");
      lcd.print(playerInputIndex);
      delay(2000);
      lcd.clear();
      return; // Fim do jogo
    }
  }

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Voce Venceu!");
  delay(2000);
  lcd.clear();
}

void generateRandomSequence() {
  sequenceLength = 0;
  for (int i = 0; i < 100; i++) {
    int color = random(4); // Gera um número aleatório de 0 a 3 para representar as cores
    int speed = random(1, 4); // Gera um número aleatório de 1 a 3 para representar as velocidades
    sequence[i] = color * 10 + speed; // Combina a cor e a velocidade em um único número
    sequenceLength++;
  }
}

void showSequence() {
  for (int i = 0; i < sequenceLength; i++) {
    int color = sequence[i] / 10; // Extrai a cor da sequência
    int speed = sequence[i] % 10; // Extrai a velocidade da sequência

    // Mostra a cor no NeoPixel com base na escolha da sequência
    if (color == redLED) {
      setNeoPixelColor(255, 0, 0); // Vermelho
    } else if (color == greenLED) {
      setNeoPixelColor(0, 255, 0); // Verde
    } else if (color == blueLED) {
      setNeoPixelColor(0, 0, 255); // Azul
    } else if (color == yellowLED) {
      setNeoPixelColor(255, 255, 0); // Amarelo
    }

    delay(speed * 1000); // Aguarda com base na velocidade
    clearNeoPixel(); // Desliga o NeoPixel
    delay(500); // Pausa de 500ms entre as cores
  }
}

void setNeoPixelColor(int red, int green, int blue) {
  for (int i = 0; i < strip.numPixels(); i++) {
    strip.setPixelColor(i, strip.Color(red, green, blue));
  }
  strip.show();
}

void clearNeoPixel() {
  for (int i = 0; i < strip.numPixels(); i++) {
    strip.setPixelColor(i, strip.Color(0, 0, 0)); // Desliga todos os LEDs
  }
  strip.show();
}

