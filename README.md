# Robots-mouth-with-LED-Matrix

# Description 
In this project, a robot’s mouth using an LED matrix is programmed using Arduino MEGA. This is supposed to mimic the movement of a talking mouth. The simulation is done in Wokwi. 

# Components
-	MAX7219 LED matrix
-	Jumper wires 
-	Arduino MEGA

LED matrix is a system of interconnected leds. The leds will light up as programmed by the driver, which in this project is the MAX7219 driver. It is a simple board of 5 pins/connectors that allows the conncetion of several modules in series. "It contains a BCD decoder, a multiplexer, and an 8×8 static RAM that stores each digit for the LED matrix display. Driver control is possible via SPI communication. One of the advantages of this driver is that MAX7219 can activate each column and row for a very short time. This allows fast switching through the columns and rows that the human eye perceives as a continuous light."

# Steps 
## 1. Building the Circuit (connections) 
The LED matrix has 5 pins each for a specific purpose. Start by connecting the first top pin to the 5V pin in the Arduino, the second pin to the GND pin in the Arduino, the third pin (DIN, second (CS), and third (CLK) to Arduino pins 11, 10, and 13 respectively. 
See the circuit diagram attached in the next section for a clearer observation. 

### Circuit Diagram 
The circuit diagram is shown below. 
 
![image](https://github.com/user-attachments/assets/001ca607-a6f4-4250-81c9-7c9b21842f38)

We are using one LED matrix; this can be adjusted based on your dimensions. Two or more LEDs can be connected and programmed together for a bigger display. 

## Programming the Matrix by Arduino MEGA 
To control the LED matrix and make it mimic the mouth’s movement, the controller is programmed using the code below. 

```
// defining the pin connections
#define CLK 13
#define DIN 11
#define CS  10

// defining the segments in x and y
#define X_SEGMENTS 4
#define Y_SEGMENTS 1
#define NUM_SEGMENTS (X_SEGMENTS * Y_SEGMENTS)

byte fb[8 * NUM_SEGMENTS];  // Framebuffer

// define the mouth's patterns // 

// smiley mouth 
byte smiley_mouth[8][4] = {
 { B00000000, B00000000, B00000000, B00000000 },
 { B00000000, B00000000, B00000000, B00000000 },
 { B00000001, B10000000, B00000001, B10000000 },
 { B00000000, B11000000, B00000011, B00000000 },
 { B00000000, B01111111, B11111110, B00000000 },
 { B00000000, B00111111, B11111100, B00000000 },
 { B00000000, B00000000, B00000000, B00000000 },
 { B00000000, B00000000, B00000000, B00000000 }
};

// closed mouth
byte closed_mouth[8][4] = {
  { B00000000, B00000000, B00000000, B00000000 },
  { B00000000, B00000000, B00000000, B00000000 },
  { B00000001, B11111111, B11111111, B00000000 },
  { B00000001, B11111111, B11111111, B00000000 },
  { B00000000, B00111111, B11111000, B00000000 },
  { B00000000, B00111111, B11111000, B00000000 },
  { B00000000, B00000000, B00000000, B00000000 },
  { B00000000, B00000000, B00000000, B00000000 }
};

// open mouth 
byte opened_mouth[8][4] = {
  { B00000000, B00000000, B00000000, B00000000 },
  { B00000001, B11111111, B11111111, B00000000 },
  { B00000001, B11111111, B11111111, B00000000 },
  { B00000000, B00000000, B00000000, B00000000 },
  { B00000000, B00000000, B00000000, B00000000 },
  { B00000000, B00111111, B11111000, B00000000 },
  { B00000000, B00111111, B11111000, B00000000 },
  { B00000000, B00000000, B00000000, B00000000 }
};

// sending commands to segments and selecting them
void shiftAll(byte send_to_address, byte send_this_data) {
  digitalWrite(CS, LOW);
  for (int i = 0; i < NUM_SEGMENTS; i++) {
    shiftOut(DIN, CLK, MSBFIRST, send_to_address);
    shiftOut(DIN, CLK, MSBFIRST, send_this_data);
  }
  digitalWrite(CS, HIGH);
}

void setup() {
  Serial.begin(115200); // serial communication
  pinMode(CLK, OUTPUT); // outputs 
  pinMode(DIN, OUTPUT);
  pinMode(CS, OUTPUT);

  // Setup each MAX7219
  shiftAll(0x0f, 0x00); 
  shiftAll(0x0b, 0x07);
  shiftAll(0x0c, 0x01); 
  shiftAll(0x0a, 0x0f); 
  shiftAll(0x09, 0x00); 

  // Sstart with the smiley mouth
  drawPattern(smiley_mouth);
  show();
  delay(2000); // display the smile for 2 seconds
}

// alternate between closed and open mouth to mimic the motion of a talking mouth
void loop() {
  for (int i = 0; i < 3; i++) {
    drawPattern(closed_mouth);
    show();
    delay(300);
    drawPattern(opened_mouth);
    show();
    delay(300);
  }
}

void drawPattern(byte pattern[8][4]) {
  clear();
  for (int y = 0; y < 8; y++) {
    for (int x = 0; x < 32; x++) {
      byte byteIndex = x / 8;
      byte bitIndex = 7 - (x % 8);
      set_pixel(x, y, (pattern[y][byteIndex] >> bitIndex) & 1);
    }
  }
}

void set_pixel(uint8_t x, uint8_t y, uint8_t mode) {
  if (x >= X_SEGMENTS * 8 || y >= 8) return; // Boundary check
  byte *addr = &fb[x / 8 + y * X_SEGMENTS];
  byte mask = 128 >> (x % 8);
  switch (mode) {
    case 0: // clear pixel
      *addr &= ~mask;
      break;
    case 1: // plot pixel
      *addr |= mask;
      break;
    case 2: // XOR pixel
      *addr ^= mask;
      break;
  }
}

// clear buffer
void clear() {
  byte *addr = fb;
  for (byte i = 0; i < 8 * NUM_SEGMENTS; i++) {
    *addr++ = 0;
  }
}

// sends data to specific row in matrix 
void show() {
  for (byte row = 0; row < 8; row++) {
    digitalWrite(CS, LOW);
    byte segment = NUM_SEGMENTS;
    while (segment--) {
      byte x = segment % X_SEGMENTS;
      byte y = segment / X_SEGMENTS * 8;
      byte addr = (row + y) * X_SEGMENTS;

      if (segment & X_SEGMENTS) { // odd rows of segments
        shiftOut(DIN, CLK, MSBFIRST, 8 - row);
        shiftOut(DIN, CLK, LSBFIRST, fb[addr + x]);
      } else { // even rows of segments
        shiftOut(DIN, CLK, MSBFIRST, 1 + row);
        shiftOut(DIN, CLK, MSBFIRST, fb[addr - x + X_SEGMENTS - 1]);
      }
    }
    digitalWrite(CS, HIGH);
  }
}
```
The mouth patterns were done using an online led matrix font generator. (*https://www.riyas.org/2013/12/online-led-matrix-font-generator-with.html#google_vignette*) 

## Results 
#### 1. Smiley mouth at the beginning: 

 ![image](https://github.com/user-attachments/assets/aca8c632-0169-420a-9e9b-d3611f309813)


#### 2. Closed Mouth: 
 
![image](https://github.com/user-attachments/assets/74ca09e4-1b95-4355-9c6b-b77c227e67ff)


#### 3. Open Mouth: 

![image](https://github.com/user-attachments/assets/30634f8d-f327-47be-9b0d-1583f870bf2a)

  
The mouth will start with a smile for 2 seconds, then will alternate between open and closed as in talking movement. 

