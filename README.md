# Voronai Lamp

![completed_lamp](https://github.com/woodwerk/3D_print_voronoi_lamp/blob/master/media/voronai_lamp.png)

This project is the intersection of a neat lamp design (not mine) plus other 3D-printed parts and NeoPixel lighting.

## 3D Printing

### Shade
The lamp shade is from this [Thingiverse project](https://www.thingiverse.com/thing:584714). I scaled it by 50% and printed it using [ERYONE White filament](https://amzn.to/3105Kha).

### Base
The base was designed in [SketchUp](http://bit.ly/2LKs3E0) to both hold the shade and mount a [Trinket M0](https://www.adafruit.com/product/3500) underneath. The image below shows the base in **Sketchup**. The interior well is where the shade rests. The slot near the center is for the **Trinket** and wires to pass through. The hex recesses are for 2-56 nuts to secure the Trinket from the underside.The half-circle passage at the right (the rear of the base) is for a micro-USB cable to program/power the lamp. The hole at far left is access to a micro push-button switch to change lamp color patterns.

The base was printed using [TRONXY Wood filament](https://amzn.to/2LKLBYL).

![sketchup_base](http://bit.ly/2LKs3E0)

### LED pillar
The interior was lit with four [NeoPixel Sticks](https://www.adafruit.com/product/1426) (8 LEDs each). 

![neopixel_stick](http://bit.ly/312kSuy)

There is an interior 30x30 mm shaft in the shade and pillar was designed to mount two of the sticks (end-to-end) facing opposite diagonals. That is, the **Sticks** were only on two faces of the pillar. The design from **Sketchup** is shown below.

![pillar](http://bit.ly/2LRUOij)

The lower, square base fits into the interior shaft of the shade and the pillar, rotated at 45˚ relative to the base provides mounting points for the **NeoPixel Sticks**. The slots allow access to attach 2-56 nuts to the machine screws that secure the **Sticks** with slight hexagonal recesses for the nuts.

The pillar was printed from the same [ERYONE White filament](https://amzn.to/3105Kha) as the shade.

The pillar resting upon the base with **Sticks** attached and wired into the Trinket is shown below.

![pillar_and_sticks](http://bit.ly/2LLg8WB)

## Electronics and Code
An [Adafruit Trinket M0](https://www.adafruit.com/product/3500) drove the four daisy-chained **NeoPixel Sticks**. The schematic below illustrates the connection to the **NeoPixels** and the push-button switch that successively cycles the program through color schemes.

![schematic](http://bit.ly/30YV1DB)

The Arduino code (neosticks.ino) to create the evolving colored lighting is below. The Sticks are run at only 1/8 brightness `uint8_t brite = 32` to avoid overheating.

```
#include <Adafruit_NeoPixel.h>

#ifdef __AVR__
  #include <avr/power.h>
#endif

#define PIXELPIN   2
#define LED_RED   13
#define BUTTONPIN  0
#define MAXFUN     5

Adafruit_NeoPixel strip = Adafruit_NeoPixel(32, PIXELPIN, NEO_RGB + NEO_KHZ800);

int fun = 0;

uint8_t brite = 32;

uint32_t Wheel(byte WheelPos) {
  if(WheelPos < 85) {
   return strip.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
  } else if(WheelPos < 170) {
   WheelPos -= 85;
   return strip.Color(255 - WheelPos * 3, 0, WheelPos * 3);
  } else {
   WheelPos -= 170;
   return strip.Color(0, WheelPos * 3, 255 - WheelPos * 3);
  }
}

// Fill the dots one after the other with a color
void colorWipe(uint32_t c, uint8_t wait) {
  for(uint16_t i=0; i<strip.numPixels(); i++) {
      strip.setPixelColor(i, c);
      strip.show();
      delay(wait);
  }
}

void rainbow(uint8_t wait) {
  uint16_t i, j;

  for(j=0; j<256; j++) {
    for(i=0; i<strip.numPixels(); i++) {
      strip.setPixelColor(i, Wheel((i+j) & 255));
    }
    strip.show();
    delay(wait);
    if (buttonPressed()) return;
  }
}

// Slightly different, this makes the rainbow equally distributed throughout
void rainbowCycle(uint8_t wait) {
  uint16_t i, j;

  for(j=0; j<256*5; j++) { // 5 cycles of all colors on wheel
    for(i=0; i< strip.numPixels(); i++) {
      strip.setPixelColor(i, Wheel(((i * 256 / strip.numPixels()) + j) & 255));
    }
    strip.show();
    delay(wait);
    if (buttonPressed()) return;
  }
}

void movingColor(uint8_t wait) {
  uint16_t i, j;

  colorWipe(strip.Color(0, 0, 0),0);
  for(j=0; j<256; j++) {
    for(i=0; i<strip.numPixels(); i++) {
//      strip.setPixelColor(i, Wheel((i+j) & 255));
      strip.setPixelColor(i, Wheel((j*4)%256));
      strip.show();
      delay(wait);
      if (buttonPressed()) return;
      strip.setPixelColor(i, strip.Color(0, 0, 0));
    }
  }
}


void white(void) {
  colorWipe(strip.Color(255, 255, 255),0);
  strip.show();
  buttonPressed();
}

void black(void) {
  colorWipe(strip.Color(0, 0, 0),0);
  strip.show();
  while (!buttonPressed());
}

boolean buttonPressed() {
  boolean pressed = !digitalRead(BUTTONPIN);
  if (pressed) {
    
    delay(10);

    pressed = !digitalRead(BUTTONPIN);
    if (pressed) {
      fun++;
      if (fun>MAXFUN) fun = 0;
      while (!digitalRead(BUTTONPIN));
    }

    for(int j=0; j<3; j++) {
      digitalWrite(LED_RED, HIGH);
      delay(10);
      digitalWrite(LED_RED, LOW);
      delay(10);
    }
  }
  return (pressed);
}

void setup() {

  // This is for Trinket 5V 16MHz, you can remove these three lines if you are not using a Trinket
  #if defined (__AVR_ATtiny85__)
    if (F_CPU == 16000000) clock_prescale_set(clock_div_1);
  #endif
  // End of trinket special code
  
  strip.begin();
  strip.setBrightness(brite);
  strip.show(); // Initialize all pixels to 'off'
  pinMode(BUTTONPIN, INPUT);
  pinMode(LED_RED, OUTPUT);
}

void loop() {
  switch (fun) {
  case 0:
      rainbow(20);
      break;
  case 1:
      rainbow(100);
      break;
  case 2:
      rainbowCycle(20);
      break;
  case 3:
      rainbowCycle(100);
      break;
  case 4:
      white();
      break;
  case 5:
      black();
      break;
  }
}

```
