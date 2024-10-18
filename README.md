#include <FastLED.h>

#define PIN_STRIP 6
#define PIN_PLAYER1 7
#define PIN_PLAYER2 2
#define NUM_LEDS 101

CRGB leds[NUM_LEDS];

int ballPosition = (NUM_LEDS / 2) - 1; // Center position
bool gameActive = true;

void setup() {
  FastLED.addLeds<NEOPIXEL, PIN_STRIP>(leds, NUM_LEDS);
  pinMode(PIN_PLAYER1, INPUT_PULLUP);
  pinMode(PIN_PLAYER2, INPUT_PULLUP);
  FastLED.clear();
  FastLED.show();
}

void loop() {
  if (!gameActive) {
    twinkleEffect();
    if (digitalRead(PIN_PLAYER1) == LOW || digitalRead(PIN_PLAYER2) == LOW) {
      gameActive = true;
      resetGame();
      delay(200); // Debounce delay
    }
    return;
  }

  int player1State = digitalRead(PIN_PLAYER1);
  int player2State = digitalRead(PIN_PLAYER2);

  // Check for button presses to move the ball outwards
  if (player1State == LOW) {
    moveBall(1);
    delay(200); // Debounce delay
  }
  if (player2State == LOW) {
    moveBall(2);
    delay(200); // Debounce delay
  }

  // Check for win condition
  if (ballPosition <= 0) {
    winnerEffect(CRGB::Blue); // Blue player wins
    gameActive = false;
  } else if (ballPosition >= NUM_LEDS - 1) {
    winnerEffect(CRGB::Red); // Red player wins
    gameActive = false;
  }

  // Draw the ball and player areas
  FastLED.clear();

  // Player 1 area in red
  for (int i = 0; i <= ballPosition; i++) {
    leds[i] = CRGB::Red;
  }

  // Player 2 area in blue
  for (int i = ballPosition + 1; i < NUM_LEDS; i++) {
    leds[i] = CRGB::Blue;
  }

  // Ball marker
  leds[ballPosition] = CRGB::White;

  FastLED.show();
}

void moveBall(int player) {
  if (player == 1 && ballPosition > 0) {
    ballPosition--; // Move left
  } else if (player == 2 && ballPosition < NUM_LEDS - 1) {
    ballPosition++; // Move right
  }
}

void winnerEffect(CRGB winnerColor) {
  for (int i = 0; i < 5; i++) {
    for (int j = 0; j < NUM_LEDS; j++) {
      leds[j] = winnerColor; // Winner color
    }
    FastLED.show();
    delay(500);
    FastLED.clear();
    FastLED.show();
    delay(500);
  }
}

void twinkleEffect() {
  for (int i = 0; i < NUM_LEDS; i++) {
    leds[i] = CHSV(random(0, 255), 255, random(100, 255));
  }
  FastLED.show();
  delay(100);
}

void resetGame() {
  ballPosition = (NUM_LEDS / 2) - 1; // Center position
  FastLED.clear();
  FastLED.show();
}
