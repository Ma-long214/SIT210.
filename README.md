
// ⚠️This is NOT the code for the re-submission. It is kept to preserve the original submission for reflection purposes.⚠️



#include <Wire.h>
#include <BH1750.h>

BH1750 lightMeter;

// LED pins
const int led1 = 5;
const int led2 = 6;

// Sensor pins
const int pirPin = 2;
const int switchPin = 3;

// Flags for interrupts
volatile bool motionFlag = false;
volatile bool switchFlag = false;

// Interrupt function for PIR sensor
void motionDetected() {
  motionFlag = true;
}

// Interrupt function for switch
void switchChanged() {
  switchFlag = true;
}

void setup() {
  Serial.begin(9600);
  while (!Serial); // Wait for Serial
  delay(500);

  // LED setup
  pinMode(led1, OUTPUT);
  pinMode(led2, OUTPUT);

  // Sensor setup
  pinMode(pirPin, INPUT);
  pinMode(switchPin, INPUT_PULLUP);

  // Initialize BH1750
  Wire.begin();
  lightMeter.begin();

  // Attach interrupts
  attachInterrupt(digitalPinToInterrupt(pirPin), motionDetected, RISING);
  attachInterrupt(digitalPinToInterrupt(3), switchChanged, CHANGE);

  Serial.println("System Ready...");
}

void loop() {

  float lux = lightMeter.readLightLevel();
  int switchState = digitalRead(switchPin);

  // If the switch is ON, it takes highest priority
  if (switchState == LOW) {
    digitalWrite(led1, HIGH);
    digitalWrite(led2, HIGH);
    Serial.println("Switch ON - Lights ON (Override)");
    
    motionFlag = false; // Ignore motion detection
  }

  else {

    // Ensure LEDs turn OFF when switch is OFF
    digitalWrite(led1, LOW);
    digitalWrite(led2, LOW);

    // Motion detected AND dark condition
    if (motionFlag) {
      if (lux < 100) {
        digitalWrite(led1, HIGH);
        digitalWrite(led2, HIGH);
        Serial.println("Motion detected in dark - Lights ON");
        delay(5000);
        digitalWrite(led1, LOW);
        digitalWrite(led2, LOW);
      } 
      else {
        digitalWrite(led1, LOW);
        digitalWrite(led2, LOW);
        Serial.println("Motion detected but it is bright - Lights OFF");
      }
      motionFlag = false;
    }
  }

  // Switch interrupt message
  if (switchFlag) {
    int state = digitalRead(switchPin);

    if (state == LOW) {
      Serial.println("Switch turned ON");
    }
    else {
      Serial.println("Switch turned OFF");
    }

    switchFlag = false;
  }

  delay(200);
}

