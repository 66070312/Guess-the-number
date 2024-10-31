# Physical Computing Project 2024 - IT KMITL
  เกม "Guess The Number" เป็นเกมที่ให้ผู้เล่นเดาหมายเลขที่สุ่มขึ้นมา โดยไมโครคอนโทรลเลอร์จะช่วยในการสุ่มหมายเลขและตรวจสอบคำตอบของผู้เล่น เกมนี้มีวัตถุประสงค์เพื่อให้ความบันเทิงและพัฒนาทักษะการคิดวิเคราะห์ของผู้เล่น
## Main Function
*  สุ่มหมายเลข: ฟังก์ชันนี้จะใช้ในการสุ่มหมายเลขที่ผู้เล่นจะต้องเดา โดยกำหนดช่วงของหมายเลขเป็นเลข 0-9 ทั้งหมด 3 หลัก
*  รับข้อมูลจากผู้เล่น: ฟังก์ชันนี้จะรอรับการป้อนข้อมูลจากผู้เล่นผ่านปุ่มกด
*  ตรวจสอบคำตอบ: ฟังก์ชันนี้จะเปรียบเทียบคำตอบของผู้เล่นกับหมายเลขที่สุ่มขึ้นมาจะมีการแสดงผลว่า "เลขที่รับเข้ามาตรงตามตำแหน่งกับเลขที่สุ่มมา" และ "เลขที่รับเข้ามามีอยู่ในเลขที่สุ่มมาแต่ไม่ตรงตามตำแหน่ง"
*  แจ้งผลและรีเซ็ตเกม: ฟังก์ชันนี้เริ่มเมื่อผู้เล่นเดาตัวเลขที่สุ่มขึ้นมาถูกต้อง และมีให้เริ่มเกมใหม่
## Website
https://66070312.github.io/Guess-the-number/
## Clip VDO
https://www.youtube.com/watch?v=0Fopl630zts&t=11s

เนื่องจากตัว hardware มีปัญหาที่ไม่สามารถหาทางแก้ไขได้(รวมถึงการซื้อใหม่)จึงจำเป็นต้องเปลี่ยน demo จากตัว microcontroller แบบตัวจริงเป็นแบบจำลองซึ่งจะใช้ source code เดียวกัน
## Poster
C:\Users\User\Documents\GitHub\Guess-the-number\poster\poster.jpg
## Source Code
```
#include <Keypad.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

const byte ROWS = 4; 
const byte COLS = 4; 
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPins[ROWS] = {2, 3, 4, 5}; 
byte colPins[COLS] = {6, 7,  8,  9};

Keypad keypad = Keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS );

LiquidCrystal_I2C lcd(0x27, 16, 2);

String text = "";
int correct = 0;
int wrongPlace = 0;
int randomNum[3];
String old = "";

void setup() {
  Serial.begin(9600);
  lcd.begin(16, 2);
  lcd.init();
  lcd.clear();
  lcd.backlight();

  randomSeed(analogRead(10));
  for (int i = 0; i < 3; i++) {
    randomNum[i] = random(0, 10);
  }
  
  lcd.print("Guessing Number!");
  delay(1000);
  lcd.clear();
  lcd.print("Enter:");
  lcd.blink();
  lcd.setCursor(0,1);
  lcd.print("Old:");
  lcd.setCursor(13,0);
  lcd.print("C:");
  lcd.print(correct);
  lcd.setCursor(12,1);
  lcd.print("WP:");
  lcd.print(wrongPlace);
  lcd.setCursor(6,0);
}

void loop() {
  char key = keypad.getKey();
  if (key != NO_KEY) {
    if (text.length() < 3) {
      if (isDigit(key)) {
        text = text + key;
        lcd.clear();
        lcdInit();
        lcd.print(text);
        Serial.println(key);
        delay(200);
      }
    }
    if (text.length() > 0) {
      if (key == '*') {
        text.remove(text.length() - 1);
        lcd.clear();
        lcdInit();
        lcd.print(text);
        Serial.println(key);
      }
    }
    if (text.length() == 3) {
      if (key == '#') {
        correct = 0;
        wrongPlace = 0;
        Serial.println(key);

        bool usedRandomNum[3] = {false, false, false};
        bool usedText[3] = {false, false, false};
        for (int i = 0; i < 3; i++) {
          if (i < 3 && randomNum[i] == (text[i] - '0')) {
            correct++;
            usedRandomNum[i] = true;
            usedText[i] = true;
          }
        }
        for (int i = 0; i < 3; i++) {
          for (int j = 0; j < 3; j++) {
            if (text[i] - '0' == randomNum[j] && !usedRandomNum[j] && !usedText[i]) {
              wrongPlace++;
              usedRandomNum[j] = true;
              usedText[i] = true;
              break;
            }
          }
        }
        if (correct != 3) {
          lcd.clear();
          old = text;
          text = "";
          lcdInit();
        } else {
          lcd.clear();
          lcd.setCursor(3,0);
          lcd.print("YOU WINNN!");
          delay(1600);
          lcd.clear();
          lcd.setCursor(0,0);
          lcd.print("New Challenge..");
          delay(1100);
          text = "";
          correct = 0;
          wrongPlace = 0;
          randomNum[3];
          old = "";
          setup();
        }
      }
    }
  }
}

void lcdInit() {
  lcd.print("Enter:");
  lcd.setCursor(0,1);
  lcd.print("Old:");
  lcd.print(old);
  lcd.setCursor(13,0);
  lcd.print("C:");
  lcd.print(correct);
  lcd.setCursor(12,1);
  lcd.print("WP:");
  lcd.print(wrongPlace);
  lcd.setCursor(6,0);
}
```
## About Us 
66070293 พงศ์ณวิชญ์ งามสวนพูล

66070312 ศุภกฤต ศรีชู
