#include <Wire.h>
#include <BH1750.h>

const int led1Pin = 2;
const int led2Pin = 3;
const int pirPin = 4;
const int switchPin = 5;

BH1750 lightMeter;
const int DARK_THRESHOLD = 50;

volatile bool pirTriggered = false;
volatile bool switchTriggered = false;

bool ledsOn = false;

void pirISR() {
  pirTriggered = true;
}

void switchISR() {
  switchTriggered = true;
}

void setup() {
  Serial.begin(9600);
  
  while (!Serial); 
  Serial.println("System Initialized.");

  // Initialize I2C bus and BH1750 sensor
  Wire.begin();
  if (lightMeter.begin()) {
    Serial.println("BH1750 Advanced begin");
  } else {
    Serial.println("Error initialising BH1750");
  }

  // Configure LED pins
  pinMode(led1Pin, OUTPUT);
  pinMode(led2Pin, OUTPUT);
  
  // Configure sensor pins
  pinMode(pirPin, INPUT); 
  pinMode(switchPin, INPUT_PULLUP);

  // --- Hardware Interrupts Setup ---
  attachInterrupt(digitalPinToInterrupt(pirPin), pirISR, RISING);
  
  attachInterrupt(digitalPinToInterrupt(switchPin), switchISR, CHANGE);
}

// --- Helper function to toggle LEDs ---
void toggleLEDs(bool turnOn) {
  if (turnOn) {
    digitalWrite(led1Pin, HIGH);
    digitalWrite(led2Pin, HIGH);
    ledsOn = true;
  }
  else {
    digitalWrite(led1Pin, LOW);
    digitalWrite(led2Pin, LOW);
    ledsOn = false;
  }
}

void loop() {
  // Slider Switch Interrupt Processing (Manual Override)
  if (switchTriggered) {
    switchTriggered = false; // Reset flag

    // Read current switch state (LOW means ON due to PULLUP)
    bool switchState = digitalRead(switchPin) == LOW;
    
    toggleLEDs(switchState);
    
    Serial.print("Event: Slider Switch Toggled. Lights ");
    Serial.println(switchState ? "ON" : "OFF");
  }

  // PIR Motion Sensor Interrupt Processing (Auto-light in the dark)
  if (pirTriggered) {
    pirTriggered = false; // Reset flag
    
    float lux = lightMeter.readLightLevel();
    Serial.print("Motion Detected. Current Light Level: ");
    Serial.print(lux);
    Serial.println(" lx");
    
    // Turn ON lights if it's dark and they are currently OFF
    if (lux < DARK_THRESHOLD && !ledsOn) {
      toggleLEDs(true);
      Serial.println("Event: Dark environment detected with motion. Lights switched ON.");
      
      // Auto-turn OFF timer
      delay(5000); 
      toggleLEDs(false);
      Serial.println("Event: Motion timer expired. Lights switched OFF automatically.");
    } 
    
    // Do nothing if it's bright
    else if (lux >= DARK_THRESHOLD) {
      Serial.println("Environment is bright enough. Lights remain OFF.");
    }
  }
}
