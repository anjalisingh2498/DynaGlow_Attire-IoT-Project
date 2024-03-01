#include <Adafruit_CircuitPlayground.h>
#include <FastLED.h>

#define Left_strip_pin 12  //led strip is soldered to pin 12 and 9
#define Right_strip_pin 9  
#define LED_PIN     6    //neo pixels led is soldered to pin 6
#define CP_PIN      17   //circuit playground's neopixels live on pin 17
#define NUM_LEDS    4   // number of LEDs in my strand
#define NUM_LEFT_STRIP  8
#define NUM_RIGHT_STRIP  8
#define NUM_CP      10   // number of neopixels on the circuit playground
#define COLOR_ORDER GRB

uint8_t brightness = 255;  //led strand brightness control
uint8_t cpbrightness = 40;  //circuit playground brightness control

int STEPS = 25;  //makes the rainbow colors more or less spread out
int NUM_MODES = 5;  // change this number if you add or subtract modes

CRGB led_left[NUM_LEFT_STRIP];     //Set up different arrays for the neopixel strand and the circuit playground
CRGB led_right[NUM_RIGHT_STRIP];      // so that we can control the brightness separately
CRGB leds[NUM_LEDS];  
CRGB cp[NUM_CP];      

CRGBPalette16 currentPalette;
TBlendType currentBlending;

int ledMode = 0;       //Initial mode 
bool leftButtonPressed;
bool rightButtonPressed;

// SOUND REACTIVE SETUP --------------

#define MIC_PIN         A4                                    // Analog port for microphone
#define DC_OFFSET  0                                          // DC offset in mic signal - if unusure, leave 0
                                                              
#define NOISE     100                                         // Noise/hum/interference in mic signal and increased value until it went quiet
#define SAMPLES   60                                          // Length of buffer for dynamic level adjustment
#define TOP (NUM_CP + 2)                                      // Allow dot to go slightly off scale
#define PEAK_FALL 10                                          // Rate of peak falling dot
 
byte
  peak      = 0,                                              // Used for falling dot
  dotCount  = 0,                                              // Frame counter for delaying dot-falling speed
  volCount  = 0;                                              // Frame counter for storing past volume data
int
  vol[SAMPLES],                                               // Collection of prior volume samples
  lvl       = 10,                                             // Current audio level, change this number to adjust sensitivity
  minLvlAvg = 0,                                              // For dynamic adjustment of graph low & high
  maxLvlAvg = 512;

// MOTION CONTROL SETUP----------

#define MOVE_THRESHOLD 10 // movement sensitivity.  lower number = less twinklitude

float X, Y, Z;
//                                 R      G      B
uint8_t myFavoriteColors[][3] = {{200,   100,   200},   
                                 {200,   200,   100},   // Change colors by inputting diferent R, G, B values on these lines
                                 {100,   200,   200},   
                               };

#define FAVCOLORS sizeof(myFavoriteColors) / 3

//                                 R      G      B
uint8_t myFavoriteColors_L[][3] = {{150,   255,   200},   
                                 {200,   150,   255},   // Change colors by inputting diferent R, G, B values on these lines
                                 {255,   200,   150},   
                               };

#define FAVCOLORS_L sizeof(myFavoriteColors_L) / 3

//                                 R      G      B
uint8_t myFavoriteColors_R[][3] = {{230,   90,   100},   
                                 {145,   67,   230},   // Change colors by inputting diferent R, G, B values on these lines
                                 {56,   240,   255},   
                               };

#define FAVCOLORS_R sizeof(myFavoriteColors_R) / 3


void setup() {
  Serial.begin(9600);
  CircuitPlayground.begin();
  FastLED.addLeds<WS2812B, Left_strip_pin, COLOR_ORDER>(leds, NUM_LEFT_STRIP).setCorrection( TypicalLEDStrip );
  FastLED.addLeds<WS2812B, Right_strip_pin, COLOR_ORDER>(leds, NUM_RIGHT_STRIP).setCorrection( TypicalLEDStrip );
  FastLED.addLeds<WS2812B, LED_PIN, COLOR_ORDER>(leds, NUM_LEDS).setCorrection( TypicalLEDStrip );
  FastLED.addLeds<WS2812B, CP_PIN, COLOR_ORDER>(cp, 10).setCorrection( TypicalLEDStrip );
  currentBlending = LINEARBLEND;
  set_max_power_in_volts_and_milliamps(5, 500);               // FastLED 2.1 Power management set at 5V, 500mA
}

void loop() {
  leftButtonPressed = CircuitPlayground.leftButton();
  rightButtonPressed = CircuitPlayground.rightButton();

  if (leftButtonPressed) {  //left button cycles through modes
    clearpixels(); 
    ledMode=ledMode+1;
    delay(300);
    if (ledMode > NUM_MODES){
    ledMode=0;
     }
  }
  if (rightButtonPressed) {   // right button turns all leds off
    ledMode=99;
  }
  
  switch (ledMode) {
       case 0: currentPalette = RainbowColors_p; rainbow(); break; 
       case 1: motion(); break;
       case 2: soundreactive(); break; 
       case 3: currentPalette = OceanColors_p; rainbow(); break;                    
       case 4: currentPalette = LavaColors_p; rainbow(); break; 
       case 5: currentPalette = RainbowStripeColors_p; rainbow(); break;    
       case 99: clearpixels(); break;
  }
}

void clearpixels()
{
  CircuitPlayground.clearPixels();
   for( int i = 0; i < NUM_CP; i++) {
   leds[i]= CRGB::Black; 
   cp[i]= CRGB::Black;
   led_left[i]= CRGB::Black;
   led_right[i]= CRGB::Black;
    }
   FastLED.show();
}

void rainbow()
{
  
  static uint8_t startIndex = 0;
  startIndex = startIndex + 1; /* motion speed */

  FillLEDsFromPaletteColors( startIndex);

  FastLED.show();
  FastLED.delay(10);
}
  
//this bit is in every palette mode, needs to be in there just once
void FillLEDsFromPaletteColors( uint8_t colorIndex)
{ 
  for( int i = 0; i < NUM_CP; i++) {
    leds[i] = ColorFromPalette( currentPalette, colorIndex, brightness, currentBlending);
    cp[i] = ColorFromPalette( currentPalette, colorIndex, cpbrightness, currentBlending);
    led_left[i] = ColorFromPalette( currentPalette, colorIndex, brightness, currentBlending);
    led_right[i] = ColorFromPalette( currentPalette, colorIndex, brightness, currentBlending);
    colorIndex += STEPS;
  }
}

void soundreactive() {
  uint8_t  i;
  uint16_t minLvl, maxLvl;
  int      n, height;
   
  n = analogRead(MIC_PIN);                                    // Raw reading from mic
  n = abs(n - 512 - DC_OFFSET);                               // Center on zero
  
  n = (n <= NOISE) ? 0 : (n - NOISE);                         // Remove noise/hum
  lvl = ((lvl * 7) + n) >> 3;                                 // "Dampened" reading (else looks twitchy)
 
  // Calculate bar height based on dynamic min/max levels (fixed point):
  height = TOP * (lvl - minLvlAvg) / (long)(maxLvlAvg - minLvlAvg);
 
  if (height < 0L)       height = 0;                          // Clip output
  else if (height > TOP) height = TOP;
  if (height > peak)     peak   = height;                     // Keep 'peak' dot at top
 
 
  // Color pixels based on rainbow gradient -- neopixel led strand
  for (i=0; i<NUM_LEDS; i++) {
    if (i >= height)   leds[i].setRGB( 0, 0,0);
    else leds[i] = CHSV(map(i,0,NUM_LEDS-1,0,255), 255, brightness);  //constrain colors here by changing HSV values
  }

  // Color pixels based on rainbow gradient -- led strip
  for (i=0; i<NUM_LEFT_STRIP; i++) {
    if (i >= height) {
      led_left[i].setRGB( 0, 0,0);
      led_right[i].setRGB( 0, 0,0);
    }
    else{
      led_left[i] = CHSV(map(i,0,NUM_LEFT_STRIP-1,0,255), 255, brightness);  //constrain colors here by changing HSV values
      led_right[i] = CHSV(map(i,0,NUM_RIGHT_STRIP-1,0,255), 255, brightness);
    }
  }
 
  // Draw peak dot  -- neopixel led strand
  if (peak > 0 && peak <= NUM_LEDS-1) leds[peak] = CHSV(map(peak,0,NUM_LEDS-1,0,255), 255, brightness);

  // Draw peak dot  -- led strips
  if (peak > 0 && peak <= NUM_LEFT_STRIP-1) {
    led_left[peak] = CHSV(map(peak,0,NUM_LEFT_STRIP-1,0,255), 255, brightness);
    led_right[peak] = CHSV(map(peak,0,NUM_RIGHT_STRIP-1,0,255), 255, brightness);
  }

  // Color pixels based on rainbow gradient  -- circuit playground
  for (i=0; i<NUM_CP; i++) {
    if (i >= height)   cp[i].setRGB( 0, 0,0);
    else cp[i] = CHSV(map(i,0,NUM_CP-1,0,255), 255, cpbrightness);  //constrain colors here by changing HSV values
  }
 
  // Draw peak dot  -- circuit playground
  if (peak > 0 && peak <= NUM_CP-1) cp[peak] = CHSV(map(peak,0,NUM_LEDS-1,0,255), 255, cpbrightness);

  // Every few frames, make the peak pixel drop by 1:
  if (++dotCount >= PEAK_FALL) {                            // fall rate 
    if(peak > 0) peak--;
      dotCount = 0;
  }
  
  vol[volCount] = n;                                          // Save sample for dynamic leveling
  if (++volCount >= SAMPLES) volCount = 0;                    // Advance/rollover sample counter
 
  // Get volume range of prior frames
  minLvl = maxLvl = vol[0];
  for (i=1; i<SAMPLES; i++) {
    if (vol[i] < minLvl)      minLvl = vol[i];
    else if (vol[i] > maxLvl) maxLvl = vol[i];
  }
  // minLvl and maxLvl indicate the volume range over prior frames, used
  // for vertically scaling the output graph (so it looks interesting
  // regardless of volume level).  If they're too close together though
  // (e.g. at very low volume levels) the graph becomes super coarse
  // and 'jumpy'...so keep some minimum distance between them (this
  // also lets the graph go to zero when no sound is playing):
  if((maxLvl - minLvl) < TOP) maxLvl = minLvl + TOP;
  minLvlAvg = (minLvlAvg * 63 + minLvl) >> 6;                 // Dampen min/max levels
  maxLvlAvg = (maxLvlAvg * 63 + maxLvl) >> 6;                 // (fake rolling average)


  show_at_max_brightness_for_power();                         // Power managed FastLED display
  Serial.print("Sound: ");Serial.println(LEDS.getFPS()); 
}

void motion() {
  X = CircuitPlayground.motionX();
  Y = CircuitPlayground.motionY();
  Z = CircuitPlayground.motionZ();
 
  double storedVector = X*X;
  storedVector += Y*Y;
  storedVector += Z*Z;
  storedVector = sqrt(storedVector);
//  Serial.print("Len: "); Serial.println(storedVector);
  
  // wait a bit
  delay(100);
  
  // get new data!
  X = CircuitPlayground.motionX();
  Y = CircuitPlayground.motionY();
  Z = CircuitPlayground.motionZ();
  double newVector = X*X;
  newVector += Y*Y;
  newVector += Z*Z;
  newVector = sqrt(newVector);
  Serial.print("Len: "); Serial.print(storedVector);Serial.print(",New Len: "); Serial.println(newVector);
  
  // are we moving 
  if (abs(10*newVector - 10*storedVector) > MOVE_THRESHOLD) {
    Serial.println("Twinkle!");
    flashRandom(5, 1);  // first number is 'wait' delay, shorter num == shorter twinkle
    flashRandom(5, 3);  // second number is how many neopixels to simultaneously light up
    flashRandom(5, 2);
  }
}

void flashRandom(int wait, uint8_t howmany) {

  for(uint16_t i=0; i<howmany; i++) {
    // pick a random favorite color!
    int c = random(FAVCOLORS);
    int red = myFavoriteColors[c][0];
    int green = myFavoriteColors[c][1];
    int blue = myFavoriteColors[c][2]; 

    // get a random pixel from the list
    int j = random(NUM_LEDS);
    //Serial.print("Lighting up "); Serial.println(j); 
    
    // now we will 'fade' it in 5 steps
    for (int x=0; x < 5; x++) {
      int r = red * (x+1); r /= 5;
      int g = green * (x+1); g /= 5;
      int b = blue * (x+1); b /= 5;
      
      leds[j].r = r;
      leds[j].g = g;
      leds[j].b = b;
      FastLED.show();
      CircuitPlayground.setPixelColor(j, r, g, b);
      delay(wait);
    }
    // & fade out in 5 steps
    for (int x=5; x >= 0; x--) {
      int r = red * x; r /= 5;
      int g = green * x; g /= 5;
      int b = blue * x; b /= 5;

      leds[j].r = r;
      leds[j].g = g;
      leds[j].b = b;
      FastLED.show(); 
      CircuitPlayground.setPixelColor(j, r, g, b); 
      delay(wait);
    }

    int cl = random(FAVCOLORS_L);
    int red_l = myFavoriteColors_L[cl][0];
    int green_l = myFavoriteColors_L[cl][1];
    int blue_l = myFavoriteColors_L[cl][2]; 

    // get a random pixel from the list
    int jl = random(NUM_LEFT_STRIP);
    //Serial.print("Lighting up "); Serial.println(j); 
    
    // now we will 'fade' it in 5 steps
    for (int x=0; x < 5; x++) {
      int rl = red_l * (x+1); rl /= 5;
      int gl = green_l * (x+1); gl /= 5;
      int bl = blue_l * (x+1); bl /= 5;
      
      led_left[jl].r = rl;
      led_left[jl].g = gl;
      led_left[jl].b = bl;
      FastLED.show();
      CircuitPlayground.setPixelColor(jl, rl, gl, bl);
      delay(wait);
    }
    // & fade out in 5 steps
    for (int x=5; x >= 0; x--) {
      int rl = red_l * x; rl /= 5;
      int gl = green_l * x; gl /= 5;
      int bl = blue_l * x; bl /= 5;

      led_left[jl].r = rl;
      led_left[jl].g = gl;
      led_left[jl].b = bl;
      FastLED.show(); 
      CircuitPlayground.setPixelColor(jl, rl, gl, bl); 
      delay(wait);
    }

    int cr = random(FAVCOLORS_R);
    int red_r = myFavoriteColors_R[cr][0];
    int green_r = myFavoriteColors_R[cr][1];
    int blue_r = myFavoriteColors_R[cr][2]; 

    // get a random pixel from the list
    int jr = random(NUM_RIGHT_STRIP);
    //Serial.print("Lighting up "); Serial.println(j); 
    
    // now we will 'fade' it in 5 steps
    for (int x=0; x < 5; x++) {
      int rr = red_r * (x+1); rr /= 5;
      int gr = green_r * (x+1); gr /= 5;
      int br = blue_r * (x+1); br /= 5;
      
      led_right[jr].r = rr;
      led_right[jr].g = gr;
      led_right[jr].b = br;
      FastLED.show();
      CircuitPlayground.setPixelColor(jr, rr, gr, br);
      delay(wait);
    }
    // & fade out in 5 steps
    for (int x=5; x >= 0; x--) {
      int rr = red_r * x; rr /= 5;
      int gr = green_r * x; gr /= 5;
      int br = blue_r * x; br /= 5;

      led_right[jr].r = rr;
      led_right[jr].g = gr;
      led_right[jr].b = br;
      FastLED.show(); 
      CircuitPlayground.setPixelColor(jr, rr, gr, br); 
      delay(wait);
    }
  }
  // LEDs will be off when done (they are faded to 0)
}
