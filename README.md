//# Arduino-Robotic-Arm---Copt-Cat
//This project controls the Cartesian robotic arm using Arduino, Motion sequence can be recorded and played back just like //Industrial robot does
/*
 COPY CAT Servo controlled Robot 5-axis Cartesian robot
 
This software logs the angular movement of joints by reading the servo angles and stores 
it in the SD card, reproduces the movement by the reading back from the SD card.
 	
 The circuit:
 * analog sensors on analog ins 0, 1, and 2
 * SD card attached to SPI bus as follows:
 ** MOSI - pin 11
 ** MISO - pin 12
 ** CLK - pin 13
 ** CS - pin 4
 
 created  24 Nov 2010
 modified 9 Apr 2012
 by Tom Igoe
 
 This example code is in the public domain.
 	 
 */

#include <SD.h>
#include <Servo.h> 
#define TRUE 1
#define FALSE 0

// On the Ethernet Shield, CS is pin 4. Note that even if it's not
// used as the CS pin, the hardware CS pin (10 on most Arduino boards
// 53 on the Mega) must be left as an output or the SD library
// functions will not work.
const int chipSelect = 4;
// constants won't change. They're used here to 
// set pin numbers:
const int buttonPin = 2;     // the number of the pushbutton pin
const int servo3 = 5, servo4 = 6,servo5 = 5;      // PWM for servo 3
// variables will change:
int buttonState = 0;         // variable for reading the pushbutton status
int recievedchar = 0,dataON=0;
int SDdata = 0;
Servo myservo1;  // create servo object to control a servo 
Servo myservo2;  // create servo object to control a servo 
Servo myservo3;  // create servo object to control a servo 
Servo myservo4;  // create servo object to control a servo 
Servo myservo5;  // create servo object to control a servo 
int servo1angle = 0,servo2angle = 0;
unsigned long current_servo_micros, prev_servo_micros;
unsigned long currentMillis,previousMillis = 0;
int scheduler = 0;
int servointerval = 20;
int S3_microlocal;
char servoON=0;
void setup()
{
     // Open serial communications and wait for port to open:
    Serial.begin(9600);
     while (!Serial) {
    ; // wait for serial port to connect. Needed for Leonardo only
  }

  pinMode(buttonPin, INPUT_PULLUP);  
    
  Serial.print("Initializing SD card...");
  // make sure that the default chip select pin is set to
  // output, even if you don't use it:
  pinMode(10, OUTPUT);
  
  // see if the card is present and can be initialized:
  if (!SD.begin(chipSelect)) {
    Serial.println("Card failed, or not present");
    // don't do anything more:
    return;
  }
  Serial.println("card initialized.");
 
}

void loop()
{

  long servo3ticks = 1000, servo4ticks = 1000, servo5ticks = 1000;           // interval for servo cyle time //

  // read the state of the pushbutton value:
  buttonState = digitalRead(buttonPin);
  recievedchar = Serial.read();
  // make a string for assembling the data to log:
  String dataString = "";
  String arraystring = "";
  String SDstring = "";
  char firstseperator=0;
  unsigned long sensorarray[10];
  int servoangle[5];
  char arrayindex = 0;

  // open the file. note that only one file can be open at a time,
  // so you have to close this one before opening another.
  if(dataON == TRUE)
  {
    
      // read three sensors and append to the string:
      for (int analogPin = 0; analogPin < 5; analogPin++) 
      {
        int sensor = analogRead(analogPin);
        sensorarray[analogPin] = sensor;
        dataString += String(sensor);
        if (analogPin < 4) 
        {
          dataString += ",";
        }
      }  
      
  
    File dataFile1 = SD.open("datalog.txt", FILE_WRITE);
    // if the file is available, write to it:
    if (dataFile1) 
    {  
      if(buttonState == HIGH)
      {
        
        // print to the serial port too:
//        Serial.println(dataString);
        
        /////////////////////////// debug code /////////////////////
          for (arrayindex= 0; arrayindex < 5; arrayindex++) 
          {
              int arrayvalue = sensorarray[arrayindex];
              //arraystring += String(arrayvalue);
          }
          
          servo1angle = map(sensorarray[0], 102, 433, 0, 170);
          arraystring += servo1angle;
          arraystring += ",";
          servo2angle = map(sensorarray[1], 84, 468, 0, 180); 
          arraystring += servo2angle;
          arraystring += ",";
          servo3ticks = map(sensorarray[2], 90, 393, 0, 150);
          arraystring += servo3ticks;
          arraystring += ",";
          servo4ticks = map(sensorarray[3], 107, 445,0, 180);
          arraystring += servo4ticks; 
          arraystring += ",";
          servo5ticks = map(sensorarray[4], 151, 406, 10, 140);
          arraystring += servo5ticks;
          arraystring += ",";
          
//          Serial.println(arraystring);
          
          
         //// WRITE TO SD CARD //
         dataFile1.println(arraystring);
         dataFile1.close();
      ///////////////////////////////////////////////////////////////// 
        delay(10);
      }
      else
      {
        dataFile1.close();
        // print to the serial port too:
        Serial.println("No signal for writing");
      }
  
    }  
    // if the file isn't open, pop up an error:
    else 
    {
      Serial.println("error opening datalog.txt");
    }
  }
  


  // the reading's most significant digit is at position 15 in the reportString:

 ////// switch case starts ///////////////////////// 
  switch (recievedchar)
  {
     case 'D':
     { 
       dataON = TRUE;
       Serial.print("Recording");
       Serial.print("\n");
       myservo1.detach();
       myservo2.detach();
       myservo3.detach();
       myservo4.detach();
       myservo5.detach();
       break;
     }
     case 'S':
     {
       dataON = FALSE;
       Serial.print("Stopped");
       Serial.print("\n");
       break;
     }
     case 'R':
       {
          myservo1.attach(5);  // attaches the servo on pin 9 to the servo object 
          myservo2.attach(3);  // attaches the servo on pin 9 to the servo object
          myservo3.attach(8);  // attaches the servo on pin 9 to the servo object 
          myservo4.attach(7);  // attaches the servo on pin 9 to the servo object
          myservo5.attach(9);  // attaches the servo on pin 9 to the servo object
          
          File dataFile1 = SD.open("datalog.txt");
          Serial.print("Playing Back Motions");
          Serial.print("\n");
        
          // if the file is available, write to it:
          if (dataFile1) 
          {
              while (dataFile1.available()) 
              {
               
                SDdata = dataFile1.read();
                SDstring += (char)SDdata; 
//                if(isDigit(SDdata))
//                {
//                 SDstring += (char)SDdata; 
//                }
//                else
//                {
//                // int Sensordata = (inString.toInt());
//                 SDstring += ("_"); 
//                }
                
                if(SDdata == '\n')
                {
                  
                  digitalWrite(servo4, LOW);
                  String reportString = SDstring;
//                  Serial.print("\n");
//                  Serial.println(reportString);
//                  Serial.print("\n");
                  
                  int seperator[5]={0};
                  int n=0,startfrom=0;
                  for(n=1;n<=5;n++)
                  {
                      seperator[n]= reportString.indexOf(',',startfrom);
                      startfrom = seperator[n]+1;
//                      Serial.print(seperator[n]);
//                      Serial.print("\n");
                  } 
                  
                  seperator[0]=-1;

               for(int count=0;count<=4;count++)
                  {
                      String sens1data = "";
                      for(int index = (seperator[count]+1); index < seperator[count+1]; index++)
                      {
                        char mostSignificantDigit =SDstring.charAt(index);
                        sens1data += String(mostSignificantDigit);
//                        Serial.print(index);
//                        Serial.print(",");
                        
                      }
                      servoangle[count+1] = sens1data.toInt();
//                      Serial.print(servoangle[count+1]);
                   //   Serial.print(sens1data);
//                      Serial.print("\n");
                  }
                                
                  delay(15);
                  SDstring = "";

                } 

//                 myservo1.write(0);
//                 myservo2.write(100);
//                 myservo3.write(100);
//                 myservo4.write(100);
//                 myservo5.write(10);
                 
               myservo1.write(servoangle[1]);
               myservo2.write(servoangle[2]);
               myservo3.write(servoangle[3]);
               myservo4.write(servoangle[4]);
               myservo5.write(servoangle[5]);
                
              }
              
              dataFile1.close();
              myservo1.detach();
              myservo2.detach();
              myservo3.detach();
              myservo4.detach();
              myservo5.detach();
          }  
          
          // if the file isn't open, pop up an error:
          else 
          {
            Serial.println("error opening datalog.txt");
          } 
          break;
       }   
              
       case 'E':
       {
         if(SD.exists("datalog.txt"))
         {
           Serial.print("datalog.txt file found, preparing to delete");
           SD.remove("datalog.txt");
           Serial.print("\n");
           Serial.print("file deleted...");
         }
         break;
       }  

       

  }
     
}
