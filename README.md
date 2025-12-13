#include <LiquidCrystal_I2C.h>
#include <Servo.h>

// -------- LCD Configuration --------
LiquidCrystal_I2C lcd(0x27, 16, 4);

// -------- Servo Configuration -------- 
Servo trackerServo;
int angle = 90;

// -------- LDR Pins -------- 
#define LDR_EAST  A1
#define LDR_WEST  A0
int sensitivity = 40;

// -------- Voltage Sensor -------- 
#define PANEL_VOLT A2
float panelVoltage = 0.0;
float sensedVoltage = 0.0;
const float Rtop = 30000.0;
const float Rbottom = 7500.0;

void setup() {
  Serial.begin(9600);

  // --- LCD Init ----
  lcd.begin();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print(" SMART SOLAR ");
  lcd.setCursor(0, 1);
  lcd.print(" TRACKING SYS ");
  delay(2000);
  lcd.clear();

  // ---- Servo Init ----
  trackerServo.attach(3);
  trackerServo.write(angle);

  pinMode(LDR_EAST, INPUT);
  pinMode(LDR_WEST, INPUT);
}

void loop() {

  // -------- Read Panel Voltage -------- 
  int sensorValue = analogRead(PANEL_VOLT);
  sensedVoltage = (sensorValue * 5.0) / 1023.0;
  panelVoltage = sensedVoltage * ((Rtop + Rbottom) / Rbottom);

  // -------- Read LDR Values -------- 
  int eastLight = analogRead(LDR_EAST);
  int westLight = analogRead(LDR_WEST);

  int diff = eastLight - westLight;

  // -------- Tracking Algorithm -------- 
  if (abs(diff) > sensitivity) {
    if (diff > 0) {
      angle = angle + 2;   // Move East
    } else {
      angle = angle - 2;   // Move West
    }
  }

  angle = constrain(angle, 0, 180);
  trackerServo.write(angle);

  // -------- LCD Output -------- 
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("PV Volt: ");
  lcd.print(panelVoltage, 2);
  lcd.print("V");

  lcd.setCursor(0, 1);
  if (abs(diff) <= sensitivity) {
    lcd.print("Status: Aligned");
    Serial.println("Panel Aligned");
  } 
  else if (diff > 0) {
    lcd.print("Moving: EAST ");
    Serial.println("Moving EAST");
  } 
  else {
    lcd.print("Moving: WEST ");
    Serial.println("Moving WEST");
  }

  // -------- Serial Debug -------- 
  Serial.print("Voltage: ");
  Serial.print(panelVoltage, 2);
  Serial.print("V | East: ");
  Serial.print(eastLight);
  Serial.print(" | West: ");
  Serial.print(westLight);
  Serial.print(" | Angle: ");
  Serial.println(angle);

  delay(600);
}
