# ESP32-C3-BLEMouse
using ESP32 C3 super mini dev board to jiggle mouse on a PC and also page turner for pdf on tablet

## Challenges:
1. Need extra hand to turn the PDF page on a table/computer while playing a musical instrument
2. need a mouse jiggle to keep my status as "green"

why not blend in solutions for both challenges into one single ESP32 C3 super mini?
in the example code, i'm:
1. utlizing the BLEMouse.h library from https://github.com/T-vK/ESP32-BLE-Mouse?tab=readme-ov-file#features
2. using 3 Single Pole Single Throw (SPST) switch to :
     a. Scroll page up OR start mouse jiggling
     b. Scroll page down OR stop mouse jiggling
     c. Switch between these 2 modes: Mouse Jiggling and Mouse Scroll
3. I have few LEDs connected to each switches to indicates there's a HIGH when it's pressed.

## Installation
- (Make sure you can use the ESP32 with the Arduino IDE. [Instructions can be found here.](https://github.com/espressif/arduino-esp32#installation-instructions))
- [Download the latest release of this library from the release page.](https://github.com/T-vK/ESP32-BLE-Mouse/releases)
- In the Arduino IDE go to "Sketch" -> "Include Library" -> "Add .ZIP Library..." and select the file you just downloaded.
- You can now go to "File" -> "Examples" -> "ESP32 BLE Mouse" and select any of the examples to get started.

## Sample Code
``` C++
#include <BleMouse.h>

// Initialize the BLE Mouse library
BleMouse bleMouse("Logi JS3");

// Define the GPIO pin for the switch
#define STOPM_SW 2  //Stop jiggling/scroll down
#define START_SW 10  //Start jiggling/scroll up
#define MODECHG_SW 4  //change mode between Jiggler/Scroll
#define LEDMODEPIN 6
#define LEDRIGHTPIN 7
#define LEDLEFTPIN 8
bool jiggFlag = false; //false = Scroll; true = Jiggler
bool stopFlag = false;
String verStr = "100225";

void setup() {
  // Initialize serial communication for debugging
  Serial.begin(115200);

  // Configure the switch pin as input with a pull-up resistor
  pinMode(STOPM_SW, INPUT_PULLUP);
  pinMode(START_SW, INPUT_PULLUP);
  pinMode(MODECHG_SW, INPUT_PULLUP);
  pinMode(LEDMODEPIN, OUTPUT);
  pinMode(LEDLEFTPIN, OUTPUT);
  pinMode(LEDRIGHTPIN, OUTPUT);

  // Start the BLE mouse
  bleMouse.begin();
  Serial.println("BLE Mouse initialized and ready.");

  stopFlag = false; //default set to start jiggling
  jiggFlag = false; //default set to scroll (false); jiggle (true)

}

void loop() {
  // Check if BLE mouse is connected
  if(bleMouse.isConnected()){
    if(jiggFlag==false){    //jiggflag=false - pg turner
      digitalWrite(LEDMODEPIN, HIGH);
      if(digitalRead(START_SW)==LOW){
        digitalWrite(LEDLEFTPIN, HIGH);
        Serial.println("Scroll up");
        for(int i = 0; i<10; i++){
          bleMouse.move(0,0,1);
          delay(100);
        }
        digitalWrite(LEDLEFTPIN, LOW);
        delay(500);
      }

      if(digitalRead(STOPM_SW)==LOW){
        digitalWrite(LEDRIGHTPIN, HIGH);
        Serial.println("Scroll up");
        for(int i = 0; i<10; i++){
          bleMouse.move(0,0,-1);
          delay(100);
        }
        digitalWrite(LEDRIGHTPIN, LOW);
        delay(500);
      }

      if(digitalRead(MODECHG_SW) == LOW){
          digitalWrite(LEDMODEPIN, LOW);
          jiggFlag = true;
          stopFlag=false; //auto run
          delay(200);
      }
      delay(100);
    } else {                //jiggflag=true - jiggler
      digitalWrite(LEDMODEPIN, LOW);
        if(stopFlag==false){  //random jiggling
            if (digitalRead(STOPM_SW) == HIGH) {
            int x = random(-50, 51); // Random movement in X direction
            int y = random(-50, 51); // Random movement in Y direction
            int ranDly = random(100,5000);
            Serial.printf("Moving mouse: X=%d, Y=%d\n", x, y);

            // Move the mouse cursor
            bleMouse.move(x, y);

            // Delay to make the movements noticeable
            //delay(800);
            digitalWrite(LEDRIGHTPIN, HIGH);
            digitalWrite(LEDLEFTPIN, HIGH);
            delay(ranDly);
            //digitalWrite(LEDRIGHTPIN, LOW);
            //digitalWrite(LEDLEFTPIN, LOW);
          } else {
            // Switch pressed: Stop mouse movement
            Serial.println("Switch pressed: Stopping mouse movement.");
            digitalWrite(LEDRIGHTPIN, LOW);
            digitalWrite(LEDLEFTPIN, LOW);
            delay(500); // Debounce delay
            stopFlag = true;
          }
        }
        if (digitalRead(START_SW) == LOW){
          stopFlag = false;
          delay(200);
        }
        if(digitalRead(MODECHG_SW) == LOW){
          digitalWrite(LEDMODEPIN, HIGH);
          delay(300);
          jiggFlag = false;
        }
        delay(100);
    }
  
  
  } else {
    Serial.println("Waiting for Bluetooth connection...");
    delay(1000);
  }
}
```
