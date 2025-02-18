# ESP32-C3-BLEMouse
using ESP32 C3 super mini dev board to jiggle mouse on a PC and also page turner for pdf on tablet

Challenges:
1. Need extra hand to turn the PDF page on a table/computer while playing a musical instrument
2. need a mouse jiggle to keep my status as "green"

why not blend in solutions for both challenges into one single ESP32 C3 super mini?
in the example code, i'm:
1. utlizing the BLEMouse.h library from https://github.com/T-vK/ESP32-BLE-Mouse?tab=readme-ov-file#features
2. using 3 Single Pole Single Throw (SPST) switch to :
     a. Scroll page up OR start mouse jiggling
     b. Scroll page down OR stop mouse jiggling
     c. Switch between these 2 modes: Mouse Jiggling and Mouse Scroll

# Sample Code
#include <BleMouse.h>

/***
ver
150125  - Initial version
        - able to connect via BT on ESP32 C3 supermini
        - Swtich 2 as stop; Switch 10 (C3 supermini) as start
        - random coordinate for the mouse to move across the screen
280125  - add in Random delay for each coordinate
100225  - Copied from sk_BTMouse_Jiggler280125
        - adding in mouse scroll. instead of using keyboard to turn page
130225  - final version for ESP32 where change mode pin will toggle between jigler and page turn
        - next version will have web server embedded
***/


// Initialize the BLE Mouse library
BleMouse bleMouse("Logi JS3");

// Define the GPIO pin for the switch
#define STOPM_SW 2
//#define START_SW 16 //for WROOM
#define START_SW 10
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

  stopFlag = false;
  jiggFlag = false; //default set to scroll

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
  } else {
    Serial.println("Waiting for Bluetooth connection...");
    delay(1000);
  }
}
