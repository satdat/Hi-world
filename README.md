# Hi-world
1st repo
I am newbe in coding world
#include <TM1637.h>

#include <TimerOne.h>

#include <TinyGPS++.h>
#include <SoftwareSerial.h>
/*
   This sample sketch demonstrates the normal use of a TinyGPS++ (TinyGPSPlus) object.
   It requires the use of SoftwareSerial, and assumes that you have a
   4800-baud serial GPS device hooked up on pins 4(rx) and 3(tx).
*/
#define ON 1
#define OFF 0
static const int RXPin = 7, TXPin = 8;
static const uint32_t GPSBaud = 9600;

// The TinyGPS++ object
TinyGPSPlus gps;
int  c=0;
 int Twelve = 13; // jumper for 12 or 24 hour display == ON = 24, OFF = 12
int LowPin = 10; // pin to pull jumber low for 12/24 hour display

//const unsigned int clockSpeed = 1; // 10000;    // speed up clock for demo

int hours = 0;
int minutes = 0;
//int  wasMinute = 99; // used for blanking leading zero

//int TwelveTwentyFour = 24;
//boolean blink = true;

//
// The serial connection to the GPS device
SoftwareSerial ss(RXPin, TXPin);
int8_t TimeDisp[] = {0x00,0x00,0x00,0x00};
unsigned char ClockPoint = 1;
//unsigned char Update;
//unsigned char halfsecond = 0;
//unsigned char second;
//unsigned char minute =0;
//unsigned char hour = 12;

//bool ampm=1;
#define CLK 3//pins definitions for TM1637 and can be changed to other ports    
#define DIO 2
TM1637 tm1637(CLK,DIO);
/*  #define SW_HOUR A0                               // Set HOURS button
#define SW_MIN  A1                               // Set MINUTES button


//RTC_DS1307 RTC;                                  // RTC DS1307 use SDA(A4) and SCL(A5) pins

//boolean dp = true;                               // digital points
uint8_t m;                                       // minutes
uint8_t h;                                       // hours
uint8_t tmp;
unsigned long vr = 0;                            // time variable
*/
void setup()
{
  tm1637.set();
  tm1637.init();
  tm1637.clearDisplay();
  Timer1.initialize(500000);//timing for 500ms
  Timer1.attachInterrupt(TimingISR);//declare the interrupt serve routine:TimingISR
  Serial.begin(115200);
  ss.begin(GPSBaud);

  Serial.println(F("DeviceExample.ino"));
  Serial.println(F("A simple demonstration of TinyGPS++ with an attached GPS module"));
  Serial.print(F("Testing TinyGPS++ library v. ")); Serial.println(TinyGPSPlus::libraryVersion());
  Serial.println(F("by Mikal Hart"));
  Serial.println();
}

void loop()
{
  
  
  // This sketch displays information every time a new sentence is correctly encoded.
  while (ss.available() > 0)
  { 
    if (gps.encode(ss.read()))
  //{ if(digitalRead(13))
  //ampm=1;}
 //else
  //ampm=0;  
      displayInfo();
      TimeUpdate();
      if (hours<=9)
    {   tm1637.display(0x00,0x7f);}
        else
        tm1637.display(0x00,TimeDisp[0]);
        tm1637.display(0x01,TimeDisp[1]);
         tm1637.display(0x02,TimeDisp[2]);
          tm1637.display(0x03,TimeDisp[3]);
      //    else
    //  tm1637.display(TimeDisp);
  if (millis() > 5000 && gps.charsProcessed() < 10)
  {
    Serial.println(F("No GPS detected: check wiring."));
    while(true);
     Serial.println(F("No GPS detected: check wiring."));
     if(!ss.available())
  digitalWrite(13,HIGH);}
}
}
  void TimingISR()
{
  
 if (c>1)
 { ClockPoint = (~ClockPoint) & 0x01;
 c=0;}
 c=c+1;
}
void TimeUpdate(void)
{
  if(ClockPoint)tm1637.point(POINT_ON);
  else tm1637.point(POINT_OFF);
  TimeDisp[0] = hours / 10;
  TimeDisp[1] = hours % 10;
  TimeDisp[2] = minutes / 10;
  TimeDisp[3] = minutes % 10;
  
}
void displayInfo()
{
  hours=gps.time.hour();
   // hours=13;     
         hours=hours+5;
        if(hours>=24)
        {hours=hours-24;}
        minutes=gps.time.minute();
     // minutes=15;
      minutes=minutes+30;
      if(minutes>=60)
     { 
    
      if(hours>=23)
      hours=0;
        hours=hours+1;
    minutes=minutes-60;}
    
    //if(ampm)
      {   if(hours>12)
        hours=(hours-12);
        //Serial.println("am");
      }
      
  Serial.print(F("Location: ")); 
  if (gps.location.isValid())
  {
    Serial.print(gps.location.lat(), 6);
    Serial.print(F(","));
    Serial.print(gps.location.lng(), 6);
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.print(F("  Date/Time: "));
  if (gps.date.isValid())
  {
    Serial.print(gps.date.month());
    Serial.print(F("/"));
    Serial.print(gps.date.day());
    Serial.print(F("/"));
    Serial.print(gps.date.year());
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.print(F(" "));
  if (gps.time.isValid())
  {
    if (gps.time.hour() < 10) Serial.print(F("0"));
    Serial.print(gps.time.hour());
    Serial.print(F(":"));
    if (gps.time.minute() < 10) Serial.print(F("0"));
    Serial.print(gps.time.minute());
    Serial.print(F(":"));
    if (gps.time.second() < 10) Serial.print(F("0"));
    Serial.print(gps.time.second());
    Serial.print(F("."));
    if (gps.time.centisecond() < 10) Serial.print(F("0"));
    Serial.print(gps.time.centisecond());
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.println();
}

