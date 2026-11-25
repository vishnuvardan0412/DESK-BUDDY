/*
  Desk Companion - Arduino Nano + 0.96" SSD1306 OLED (I2C) + 1 push button

  Wiring:
    OLED VCC -> 5V (or 3.3V), GND -> GND, SDA -> A4, SCL -> A5
    Button   -> one leg to D2, other leg to GND (uses internal pull-up)

  Library: install "U8g2" from Library Manager.

  HOME SCREEN = the FACE. It has moods and changes on its own:
    normal, happy, sleepy, surprised, love (heart eyes), wink
  On power-up it greets you ("Hey, Vishnu!") for 3 seconds, then shows the face.
  The greeting pops up again every ~2.5 minutes while you are on the face.

  Controls (SHORT press = next screen, LONG press = action):
    Face      : long = change mood by hand
    Quote     : long = next quote (also changes every 20 s)
    Pomodoro  : long = start / pause (25 min focus, 5 min break)
    Stopwatch : long = start / pause (resets if you leave it paused)
    Decider   : long = ask, get a random answer
  Tap during the greeting to skip it.
*/

#include <U8g2lib.h>
#include <Wire.h>
#include <avr/pgmspace.h>

U8G2_SSD1306_128X64_NONAME_1_HW_I2C u8g2(U8G2_R0, U8X8_PIN_NONE);

// ====== CHANGE YOUR NAME HERE ======
const char MY_NAME[] = "Vishnu";
// ===================================

const uint8_t  BTN_PIN     = 2;
const uint16_t LONG_MS     = 600;
const uint8_t  NUM_SCREENS = 5;

enum { SCR_FACE, SCR_QUOTE, SCR_POMO, SCR_STOPWATCH, SCR_DECIDER };
uint8_t screen = SCR_FACE;

enum { M_NORMAL, M_HAPPY, M_SLEEPY, M_SURPRISED, M_LOVE, M_WINK, NUM_MOODS };

// ---------- Button: returns 0 = none, 1 = short, 2 = long ----------
uint8_t readButton() {
  static bool down = false, longFired = false;
  static uint32_t tPress = 0, tChange = 0;
  bool pressed = (digitalRead(BTN_PIN) == LOW);
  uint32_t now = millis();

  if (pressed && !down && now - tChange > 30) {
    down = true; longFired = false; tPress = now; tChange = now;
  } else if (pressed && down && !longFired && now - tPress >= LONG_MS) {
    longFired = true;
    return 2;
  } else if (!pressed && down && now - tChange > 30) {
    down = false; tChange = now;
    if (!longFired) return 1;
  }
  return 0;
}

void drawCentered(const char* s, uint8_t y) {
  uint8_t w = u8g2.getStrWidth(s);
  u8g2.drawStr((128 - w) / 2, y, s);
}

// ---------- GREETING (shown at boot and now and then) ----------
bool     greeting = false;
uint32_t greetUntil = 0, nextGreet = 0;
uint8_t  greetIdx = 0;

const char* const greetings[] = {
  "Hey,", "Hello,", "Hi there,", "Welcome back,", "Yovv bruh,"
};
const uint8_t NUM_GREETINGS = sizeof(greetings) / sizeof(greetings[0]);

void startGreeting() {
  greeting = true;
  greetUntil = millis() + 3000;
  nextGreet = millis() + 150000UL;
  greetIdx = random(0, NUM_GREETINGS);
}

void drawGreeting() {
  char nameBuf[20];
  snprintf(nameBuf, sizeof(nameBuf), "%s!", MY_NAME);
  u8g2.setFont(u8g2_font_helvB12_tr);
  drawCentered(greetings[greetIdx], 26);
  u8g2.setFont(u8g2_font_helvB18_tr);
  drawCentered(nameBuf, 54);
}

// ---------- FACE ----------
uint8_t  mood = M_NORMAL;
uint32_t moodNext = 6000;
int8_t   lookX = 0, lookTarget = 0;
uint32_t nextBlink = 2000, blinkStart = 0, nextLook = 1500;
bool     blinking = false;
uint8_t  eyeH = 30;

void pickMood() {
  uint32_t now = millis();
  if (mood != M_NORMAL) {                 // go back to normal after a while
    mood = M_NORMAL;
    moodNext = now + random(5000, 12000);
  } else {
    uint8_t r = random(1, NUM_MOODS);     // any mood except normal
    mood = r;
    moodNext = now + random(2500, 5000);
  }
}

void eyesUpdate() {
  uint32_t now = millis();

  if (now >= moodNext) pickMood();

  if (now >= nextLook) {
    lookTarget = random(-10, 11);
    nextLook = now + random(800, 3000);
  }
  if (lookX < lookTarget) lookX++;
  else if (lookX > lookTarget) lookX--;

  uint8_t baseH = (mood == M_SLEEPY) ? 12 : 30;
  eyeH = baseH;

  bool canBlink = (mood == M_NORMAL || mood == M_SLEEPY || mood == M_WINK);
  if (canBlink) {
    if (!blinking && now >= nextBlink) {
      blinking = true;
      blinkStart = now;
    }
    if (blinking) {
      uint32_t t = now - blinkStart;
      if (t >= 180) {
        blinking = false;
        nextBlink = now + random(1500, 5000);
      } else {
        float frac = (t < 90) ? 1.0 - t / 90.0 : (t - 90) / 90.0;
        eyeH = max(3, (int)(baseH * frac));
      }
    }
  }
}

void drawHeart(int cx, int cy) {
  u8g2.drawDisc(cx - 6, cy - 3, 6);
  u8g2.drawDisc(cx + 6, cy - 3, 6);
  u8g2.drawTriangle(cx - 12, cy - 1, cx + 12, cy - 1, cx, cy + 12);
}

void drawFace() {
  const uint8_t w = 30, cy = 22;
  const uint8_t xs[2] = {24, 74};
  const uint8_t SMILE = U8G2_DRAW_LOWER_LEFT | U8G2_DRAW_LOWER_RIGHT;

  // ---- eyes ----
  for (uint8_t i = 0; i < 2; i++) {
    int x  = xs[i] + lookX;
    int cx = x + w / 2;
    uint8_t r = min((uint8_t)10, (uint8_t)(eyeH / 2));

    switch (mood) {
      case M_LOVE:
        drawHeart(cx, cy);
        break;

      case M_SURPRISED:
        u8g2.drawRBox(x - 2, cy - 18, w + 4, 36, 16);   // big round eye
        u8g2.setDrawColor(0);
        u8g2.drawDisc(cx, cy, 5);                       // pupil
        u8g2.setDrawColor(1);
        break;

      case M_HAPPY:
        u8g2.drawRBox(x, cy - eyeH / 2, w, eyeH, r);
        u8g2.setDrawColor(0);
        u8g2.drawDisc(cx, cy + 17, 17);                 // cut bottom -> ^ ^ eyes
        u8g2.setDrawColor(1);
        break;

      case M_WINK:
        if (i == 1) u8g2.drawRBox(x, cy - 2, w, 5, 2);  // closed eye
        else        u8g2.drawRBox(x, cy - eyeH / 2, w, eyeH, r);
        break;

      default:  // normal, sleepy
        u8g2.drawRBox(x, cy - eyeH / 2, w, eyeH, r);
        break;
    }
  }

  // ---- mouth ----
  switch (mood) {
    case M_NORMAL:
      u8g2.drawCircle(64, 44, 7, SMILE);
      u8g2.drawCircle(64, 44, 6, SMILE);
      break;
    case M_HAPPY:
      u8g2.drawDisc(64, 40, 13, SMILE);                 // open smile
      break;
    case M_SLEEPY:
      u8g2.drawCircle(64, 52, 3, U8G2_DRAW_ALL);
      u8g2.setFont(u8g2_font_6x10_tr);
      u8g2.drawStr(100, 14, "z");
      u8g2.drawStr(108, 8, "Z");
      break;
    case M_SURPRISED:
      u8g2.drawCircle(64, 54, 5, U8G2_DRAW_ALL);
      break;
    case M_LOVE:
    case M_WINK:
      u8g2.drawCircle(64, 42, 11, SMILE);
      u8g2.drawCircle(64, 42, 10, SMILE);
      break;
  }
}

// ---------- QUOTES ('|' = line break, max ~21 chars per line) ----------
const char q0[]  PROGMEM = "Small steps every|day add up";
const char q1[]  PROGMEM = "Discipline beats|motivation";
const char q2[]  PROGMEM = "Write it, learn it,|own it";
const char q3[]  PROGMEM = "Do it now.|Future you|says thanks";
const char q4[]  PROGMEM = "Progress,|not perfection";
const char q5[]  PROGMEM = "Stay curious.|Keep building";
const char q6[]  PROGMEM = "One more problem.|One more rep";
const char q7[]  PROGMEM = "Your only rival|is yesterday's you";
const char q8[]  PROGMEM = "Start before|you feel ready";
const char q9[]  PROGMEM = "Hard now,|easy later";
const char q10[] PROGMEM = "Fix the bug,|ship the thing";
const char q11[] PROGMEM = "Consistency wins";
const char q12[] PROGMEM = "Dream big.|Start small.|Start today";
const char q13[] PROGMEM = "Every expert was|once a beginner";
const char q14[] PROGMEM = "Curiosity without|prioritization|becomes distraction";
const char q15[] PROGMEM = "Ambition without|consistency becomes|anxiety";
const char q16[] PROGMEM = "Leadership without|delegation becomes|overload";
const char q17[] PROGMEM = "Confidence without|reflection becomes|stubbornness";
const char q18[] PROGMEM = "Reflection without|action becomes|overthinking";

const char* const quotes[] PROGMEM = {
  q0, q1, q2, q3, q4, q5, q6, q7, q8, q9, q10, q11, q12, q13,q14, q15, q16, q17, q18
};
const uint8_t NUM_QUOTES = sizeof(quotes) / sizeof(quotes[0]);

uint8_t  qIdx = 0;
uint32_t qNextAuto = 0;

void quoteNext() {
  uint8_t n;
  do { n = random(0, NUM_QUOTES); } while (n == qIdx);
  qIdx = n;
  qNextAuto = millis() + 20000UL;
}

void quoteUpdate() {
  if (screen == SCR_QUOTE && millis() >= qNextAuto) quoteNext();
}

void drawQuote() {
  char buf[64];
  const char* p = (const char*)pgm_read_ptr(&quotes[qIdx]);
  strncpy_P(buf, p, sizeof(buf) - 1);
  buf[sizeof(buf) - 1] = 0;

  char* lines[4];
  uint8_t n = 0;
  lines[n++] = buf;
  for (char* c = buf; *c && n < 4; c++) {
    if (*c == '|') { *c = 0; lines[n++] = c + 1; }
  }

  u8g2.setFont(u8g2_font_6x10_tr);
  uint8_t lineH = 14;
  uint8_t y = (64 - n * lineH) / 2 + 11;
  for (uint8_t i = 0; i < n; i++) drawCentered(lines[i], y + i * lineH);
}

// ---------- POMODORO ----------
const uint16_t FOCUS_S = 25 * 60, BREAK_S = 5 * 60;
bool     pRun = false, pBreak = false;
uint16_t pRemain = FOCUS_S;
uint32_t pLast = 0;

void pomoUpdate() {
  if (!pRun) return;
  uint32_t now = millis();
  while (now - pLast >= 1000) {
    pLast += 1000;
    if (pRemain > 0) pRemain--;
    if (pRemain == 0) {
      pBreak = !pBreak;
      pRemain = pBreak ? BREAK_S : FOCUS_S;
      pRun = false;
    }
  }
}

void pomoToggle() {
  pRun = !pRun;
  if (pRun) pLast = millis();
}

void drawPomo() {
  char buf[8];
  snprintf(buf, sizeof(buf), "%02u:%02u", pRemain / 60, pRemain % 60);

  u8g2.setFont(u8g2_font_6x10_tr);
  u8g2.drawStr(0, 10, pBreak ? "BREAK" : "FOCUS");
  u8g2.drawStr(80, 10, pRun ? "running" : "hold=go");

  u8g2.setFont(u8g2_font_helvB24_tn);
  u8g2.drawStr(10, 46, buf);

  uint16_t total = pBreak ? BREAK_S : FOCUS_S;
  uint8_t fill = (uint32_t)(total - pRemain) * 124 / total;
  u8g2.drawFrame(0, 54, 128, 10);
  u8g2.drawBox(2, 56, fill, 6);
}

// ---------- STOPWATCH ----------
bool     swRun = false;
uint32_t swElapsed = 0, swStart = 0;

uint32_t swNow() { return swElapsed + (swRun ? millis() - swStart : 0); }

void swToggle() {
  if (swRun) { swElapsed += millis() - swStart; swRun = false; }
  else       { swStart = millis(); swRun = true; }
}

void drawStopwatch() {
  uint32_t ms = swNow();
  uint32_t s = ms / 1000;
  char buf[12];
  snprintf(buf, sizeof(buf), "%02lu:%02lu", s / 60, s % 60);

  u8g2.setFont(u8g2_font_6x10_tr);
  u8g2.drawStr(0, 10, "STOPWATCH");
  u8g2.drawStr(80, 10, swRun ? "running" : "hold=go");

  u8g2.setFont(u8g2_font_helvB24_tn);
  u8g2.drawStr(10, 52, buf);

  char tenths[4];
  snprintf(tenths, sizeof(tenths), ".%lu", (ms / 100) % 10);
  u8g2.setFont(u8g2_font_6x10_tr);
  u8g2.drawStr(108, 52, tenths);
}

// ---------- DECIDER ----------
bool     rolling = false;
uint32_t rollEnd = 0, rollTick = 0;
int8_t   answer = -1;

const char* answerText(int8_t i) {
  switch (i) {
    case 0: return "YES!| NOW!!!";
    case 1: return "NOPE";
    case 2: return "MAYBE";
    case 3: return "DO IT";
    case 4: return "ASK LATER";
    case 5: return "NO WAY";
    case 6: return "DEFINITELY";
    default: return "...";
  }
}

void deciderRoll() {
  rolling = true;
  rollEnd = millis() + 900;
  rollTick = 0;
}

void deciderUpdate() {
  if (!rolling) return;
  uint32_t now = millis();
  if (now - rollTick > 80) {
    answer = random(0, 7);
    rollTick = now;
  }
  if (now >= rollEnd) rolling = false;
}

void drawDecider() {
  u8g2.setFont(u8g2_font_6x10_tr);
  u8g2.drawStr(0, 10, "DECIDER");
  u8g2.drawStr(80, 10, "hold=ask");

  u8g2.setFont(u8g2_font_helvB14_tr);
  if (answer < 0) drawCentered("Ask me...", 44);
  else            drawCentered(answerText(answer), 44);
}

// ---------- MAIN ----------
void changeScreen() {
  if (screen == SCR_STOPWATCH && !swRun) swElapsed = 0;   // reset if left paused
  screen = (screen + 1) % NUM_SCREENS;
  if (screen == SCR_QUOTE) quoteNext();                   // fresh quote on entry
}

void setup() {
  pinMode(BTN_PIN, INPUT_PULLUP);
  randomSeed(analogRead(A0));
  u8g2.begin();
  u8g2.setBusClock(400000);

  qIdx = random(0, NUM_QUOTES);
  startGreeting();            // greet on power-up
}

void loop() {
  uint8_t b = readButton();
  uint32_t now = millis();

  // greeting: any press skips it, otherwise it ends by itself
  if (greeting) {
    if (b || now >= greetUntil) greeting = false;
    b = 0;
  } else if (screen == SCR_FACE && now >= nextGreet) {
    startGreeting();
  }

  if (b == 1) {
    changeScreen();
  } else if (b == 2) {
    switch (screen) {
      case SCR_FACE:      mood = (mood + 1) % NUM_MOODS;
                          moodNext = now + 8000;           break;
      case SCR_QUOTE:     quoteNext();                     break;
      case SCR_POMO:      pomoToggle();                    break;
      case SCR_STOPWATCH: swToggle();                      break;
      case SCR_DECIDER:   deciderRoll();                   break;
    }
  }

  pomoUpdate();
  eyesUpdate();
  deciderUpdate();
  quoteUpdate();

  u8g2.firstPage();
  do {
    if (greeting) {
      drawGreeting();
    } else {
      switch (screen) {
        case SCR_FACE:      drawFace();      break;
        case SCR_QUOTE:     drawQuote();     break;
        case SCR_POMO:      drawPomo();      break;
        case SCR_STOPWATCH: drawStopwatch(); break;
        case SCR_DECIDER:   drawDecider();   break;
      }
    }
  } while (u8g2.nextPage());
}