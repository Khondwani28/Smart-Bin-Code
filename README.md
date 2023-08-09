# IOT BIN NODE CODE
/*

  PROJECT: IOT DUSTBIN

*/



// include the library code:

#include <LiquidCrystal.h>

#include <Servo.h>

Servo myservo;  // create servo object to control a servo

// twelve servo objects can be created on most boards



#include <SPI.h>

#include <RH_RF95.h>



// Singleton instance of the radio driver

RH_RF95 rf95;



// initialize the library by associating any needed LCD interface pin

// with the arduino pin number it is connected to

const int rs = A0, en = A1, d4 = A2, d5 = A3, d6 = A4, d7 = A5;

LiquidCrystal lcd(rs, en, d4, d5, d6, d7);



const int trigPin = 4;

const int echoPin2 = 8;

const int echoPin1 = 5;



long duration2;

int distance2;

long duration1;

int distance1;

int percent = 0;



int pos = 0;    // variable to store the servo position



void setup() {

  // set up the LCD's number of columns and rows:

  lcd.begin(16, 4);



  pinMode (trigPin, OUTPUT);

  pinMode (echoPin2, INPUT);

  pinMode (echoPin1, INPUT);



  myservo.attach(3);  // attaches the servo on pin 9 to the servo object

 myservo.write(pos);

  Serial.begin(9600);



    if (!rf95.init())

    Serial.println("init failed");

  // Defaults after init are 434.0MHz, 13dBm, Bw = 125 kHz, Cr = 4/5, Sf = 128chips/symbol, CRC on



  // The default transmitter power is 13dBm, using PA_BOOST.

  // If you are using RFM95/96/97/98 modules which uses the PA_BOOST transmitter pin, then 

  // you can set transmitter powers from 5 to 23 dBm:

//  driver.setTxPower(23, false);

  // If you are using Modtronix inAir4 or inAir9,or any other module which uses the

  // transmitter RFO pins and not the PA_BOOST pins

  // then you can configure the power transmitter power for -1 to 14 dBm and with useRFO true. 

  // Failure to do that will result in extremely low transmit powers.

//  driver.setTxPower(14, true);



}



void loop() {



  check_bin_level();

  lcd.clear();

  lcd.setCursor(0, 0);

  lcd.print("LoRa IoT DUSTBIN");

  lcd.setCursor(0, 2);

  lcd.print("BIN LEVEL: ");

  lcd.setCursor(5, 3);

  lcd.print(percent);

  lcd.print("%");



  delay(100); 

  

  if ( percent > 95) {

    //    digitalWrite(red_LED, HIGH);

    //    digitalWrite(green_LED, LOW);

  }

  else {

    //    digitalWrite(red_LED, LOW);

    //    digitalWrite(green_LED, HIGH);



    check_proximity();

    if (distance1 < 10) {

      //open bin

      for (pos = 0; pos <= 180; pos += 1) { // goes from 0 degrees to 180 degrees

        // in steps of 1 degree

        myservo.write(pos);              // tell servo to go to position in variable 'pos'

        delay(15);                       // waits 15ms for the servo to reach the position

      }

      //keep bin open until person moves away

      while (distance1 < 10) {

        check_proximity();

        delay(10);

      }

      //close bin

      for (pos = 180; pos >= 0; pos -= 1) { // goes from 180 degrees to 0 degrees

       myservo.write(pos);              // tell servo to go to position in variable 'pos'

        delay(15);                       // waits 15ms for the servo to reach the position

      }

    }



  }



  Serial.println("Sending to rf95_server");

  // Send a message to rf95_server

  char data[3];

  itoa(percent, data, 10);

//uint8_t data[] = {percent};

  rf95.send(data, sizeof(data));

  

  rf95.waitPacketSent();

  // Now wait for a reply

  uint8_t buf[RH_RF95_MAX_MESSAGE_LEN];

  uint8_t len = sizeof(buf);



  if (rf95.waitAvailableTimeout(3000))

  { 

    // Should be a reply message for us now   

    if (rf95.recv(buf, &len))

   {

      Serial.print("got reply: ");

      Serial.println((char*)buf);

//      Serial.print("RSSI: ");

//      Serial.println(rf95.lastRssi(), DEC);    

    }

    else

    {

      Serial.println("recv failed");

    }

  }

  else

  {

    Serial.println("No reply, is rf95_server running?");

  }

  delay(400);



  delay(500);

}



void check_bin_level() {

  digitalWrite (trigPin, LOW);

  delayMicroseconds (2);

  //set the trigPin on HIGH state for 10 micro seconds

  digitalWrite(trigPin, HIGH);

  delayMicroseconds(10);

  digitalWrite(trigPin, LOW);



  duration2 = pulseIn(echoPin2, HIGH);



  distance2 = duration2 * 0.034 / 2;



  percent = map( distance2, 29, 4, 0, 100);



}



void check_proximity() {

  digitalWrite (trigPin, LOW);

  delayMicroseconds (2);

  //set the trigPin on HIGH state for 10 micro seconds

  digitalWrite(trigPin, HIGH);

  delayMicroseconds(10);

  digitalWrite(trigPin, LOW);



  duration1 = pulseIn(echoPin1, HIGH);



  distance1 = duration1 * 0.034 / 2;

  Serial.print("Distance: ");

  Serial.print(distance1);

  Serial.println(" cm");



}





