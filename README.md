#include <Dynamixel2Arduino.h>  // Bibliotheken importieren

#include <SoftwareSerial.h> // Bibliotheken importieren
SoftwareSerial soft_serial(7, 8);  // DYNAMIXELShield UART RX/TX
#define DXL_SERIAL Serial
#define DEBUG_SERIAL soft_serial
const int DXL_DIR_PIN = 2;  // DYNAMIXEL Shield DIR PIN


const uint8_t DXL_ID = 1;
const float DXL_PROTOCOL_VERSION = 2.0;

Dynamixel2Arduino dxl(DXL_SERIAL, DXL_DIR_PIN);

using namespace ControlTableItem;

float angle = 0.0;
float prevAngle = 0.0;
int16_t prevCurrent = 0;
int16_t current = 0;


//  ==  NEXTION LCD == 
#include "Nextion.h" // Bibliotheken importieren

NexButton inc = NexButton(1, 5, "b3");    // button um winkel zu erhöhen
NexButton dec = NexButton(1, 6, "b4");   // button um winkel zu verringern
NexButton ang1 = NexButton(1, 4, "b2");   // button für winkel -90°
NexButton ang2 = NexButton(1, 3, "b1");   // button für winkel 90°
NexButton ang3 = NexButton(1, 2, "b0");   // button für winkel 0°
NexButton cur1 = NexButton(1, 8, "b6"); // button kraftstufe 1
NexButton cur2 = NexButton(1, 7, "b5"); // button kraftstufe 2
NexButton cur3 = NexButton(1, 9, "b7"); // button kraftstufe 3
NexButton inc1 = NexButton (1,10, "b8"); // buttton kraft erhöhen
NexButton dec1 = NexButton (1, 11, "b9"); // button kraft niedriger stellen
NexButton pag = NexButton(0, 4, "b5"); // button seite wechseln
NexText status = NexText(0, 2, "t0");  // text1
NexText status1 = NexText (0, 5, "t1"); // text2

// Buttons registrieren
NexTouch *nex_listen_list[] = {
  &inc,
  &dec,
  &ang1,
  &ang2,
  &ang3,
  &cur1,
  &cur2,
  &cur3,
  &inc1,
  &dec1,
  &pag,

  NULL
};
// Callback Funktionen
void incPopCallback(void *ptr) {
  angle += 1;                              // Winkel um 1° erhöhen
  angle = (angle > 90.0) ? 90.0: angle;  // Winkel maximal um 90°
}
void decPopCallback(void *ptr) {
  angle -= 1;                          // Winkel um 1° verringern
  angle = (angle < -90.0) ? -90.0: angle;  // Winkel maximal um -90°
}
void ang1PopCallback(void *ptr) {
  angle = -90.0;  // Winkel einstellen für -90° Button
}
void ang2PopCallback(void *ptr) {
  angle = 90.0;  // Winkel einstellen für 90° Button
}
void ang3PopCallback(void *ptr) {
  angle = 0.0;  // Winkel einstellen für 0° Button
}
void cur1PopCallback(void *ptr) {
  current = 50.0;  // Kraftstufe 1
}
void cur2PopCallback(void *ptr) {
  current = 30.0;  // Kraftstufe 2
}
void cur3PopCallback(void *ptr) {
  current = 10.0;  // Kraftstufe 3
}
void inc1PopCallback(void *ptr) {
  current += 3;                              // Kraft erhhöhen um 3%
  current = (current > 100.0) ? 100.0: current;  // Kraft maximal 100%
}
void dec1PopCallback(void *ptr) {
  current -= 3;                              // Kraft senken um 3% erhöhen
  current = (current > -100.0) ? -100.0: current;  // Kraft minimum -100%
}




void setup() {

  DEBUG_SERIAL.begin(115200);
  nexInit();

  inc.attachPop(incPopCallback, &inc);    
  dec.attachPop(decPopCallback, &dec);    
  ang1.attachPop(ang1PopCallback, &ang1);
  ang2.attachPop(ang2PopCallback, &ang2);  
  ang3.attachPop(ang3PopCallback, &ang3);
  cur1.attachPop(cur1PopCallback, &cur1);  
  cur2.attachPop(cur2PopCallback, &cur2); 
  cur3.attachPop(cur3PopCallback, &cur3); 
  inc1.attachPop(inc1PopCallback, &inc1); 
  dec1.attachPop(dec1PopCallback, &dec1); 

  status.setText("C");                    
  delay(100);
  status.setText("C");
  delay(100);
  dxl.begin(57600);
  dxl.setPortProtocolVersion(DXL_PROTOCOL_VERSION);
  dxl.ping(DXL_ID);

  dxl.torqueOff(DXL_ID);
  dxl.setOperatingMode(DXL_ID, OP_CURRENT_BASED_POSITION);
  dxl.torqueOn(DXL_ID);

  dxl.writeControlTableItem(PROFILE_VELOCITY, DXL_ID, 65);

  dxl.setGoalPosition(DXL_ID, 0.0, UNIT_DEGREE);

  dxl.setGoalCurrent(DXL_ID, 6.0, UNIT_PERCENT);
}

void loop() {
    if (prevCurrent != current) {
    prevCurrent = current;  
    dxl.setGoalCurrent(DXL_ID, current, UNIT_PERCENT);
  }

  if (prevAngle != angle) {
    dxl.setGoalPosition(DXL_ID, angle, UNIT_DEGREE);
    prevAngle = angle;
  }
 
  nexLoop(nex_listen_list);
}
