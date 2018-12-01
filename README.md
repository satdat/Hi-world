# Hi-world
1st repo
I am newbe in coding world
/*
 * A library for the 4 digit display
 *
 * Copyright (c) 2012 seeed technology inc.
 * Website    : www.seeed.cc
 * Author     : Frankie.Chu
 * Create Time: 9 April,2012
 * Change Log :
 *
 * The MIT License (MIT)
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 */
#include <TimerOne.h>
#include "TM1637.h"
#define ON 1
#define OFF 0

#include <TinyGPS++.h>
#include <SoftwareSerial.h>
TinyGPSPlus gps;
 int c=0;


int Twelve = 13; // jumper for 12 or 24 hour display == ON = 24, OFF = 12
int LowPin = 10; // pin to pull jumber low for 12/24 hour display

const unsigned int clockSpeed = 1; // 10000;    // speed up clock for demo

int hours = 0;
int minutes = 0;
int  wasMinute = 99; // used for blanking leading zero

int TwelveTwentyFour = 24;
boolean blink = true;

static const int RXPin = 7, TXPin = 6;
static const uint32_t GPSBaud = 9600;
SoftwareSerial ss(RXPin, TXPin);

int8_t TimeDisp[] = {0x00,0x00,0x00,0x00};

unsigned char ClockPoint = 1;
unsigned char Update;
unsigned char halfsecond = 0;


bool ampm=1;
#define CLK 3//pins definitions for TM1637 and can be changed to other ports
#define DIO 2
TM1637 tm1637(CLK,DIO);

void setup()
{
  tm1637.set();
  tm1637.init();
  //tm1637.clearDisplay();
  Timer1.initialize(500000);//timing for 500ms
  Timer1.attachInterrupt(TimingISR);//declare the interrupt serve routine:TimingISR
  
   ss.begin(GPSBaud);
   Serial.begin(115200);

//pinMode(5, INPUT_PULLUP);
 pinMode(13, OUTPUT);

  /*digitalWrite(13,HIGH);
  delay(2000);
  digitalWrite(13,LOW);
  delay(2000);
  digitalWrite(13,HIGH);
  delay(2000);
  digitalWrite(13,LOW);
  delay(2000);
 
  
 tm1637.set();
  tm1637.init();
tm1637.clearDisplay();
*/
  
  
  
 // pinMode(Satellites, INPUT_PULLUP);
 // pinMode (LowPin, OUTPUT);
 // pinMode (LowPin2, OUTPUT);
  //digitalWrite(LowPin, LOW);
  //digitalWrite(LowPin2, LOW);
  //delay(1000);
  
}
void loop()
{  

  //delay(1000);
  while (ss.available() > 0)
 {
    if (gps.encode(ss.read()))
    {  if(digitalRead(13))
  ampm=1;
  else
  ampm=0;
  
  
      displayInfo();
      TimeUpdate();
      if (hours<=9)
      { tm1637.display(0x00,0x7f);
        tm1637.display(0x01,TimeDisp[1]);
         tm1637.display(0x02,TimeDisp[2]);
          tm1637.display(0x03,TimeDisp[3]);}
         else
      tm1637.display(TimeDisp);
    /*
      Serial.println(F(" GPS detected: ."));
      Serial.print("Satellites = ");
      int  d=gps.satellites.value();
       Serial.print(d);

       Serial.print("Time (GMT) : ");
    //if(gps.time.hour() < 10)     Serial.print("0");
        Serial.print(gps.time.hour());
     
        
      Serial.print(":");
       //if(gps.time.minute() < 10)   Serial.print("0");
       Serial.print(gps.time.minute());
      Serial.print(":");
       // if(gps.time.second() < 10)   Serial.print("0");
       Serial.println(gps.time.second());*/
    }  }
  
//    Serial.println(F("No GPS detected: check wiring."));
     if(!ss.available())
  digitalWrite(13,LOW);
  
 

}
void TimingISR()
{ 
 /* halfsecond ++;
  Update = ON;
  if(halfsecond == 2){
    second ++;
    if(second == 60)
    {
      minute ++;
      if(minute == 60)
      {
        hour ++;
        if(hour == 24)hour = 0;
        minute = 0;
      }
      second = 0;
    }
    halfsecond = 0;
  }*/
 
//  Serial.println(second);
 if(c>1)
  {ClockPoint = (~ClockPoint) & 0x01;
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
      
       
        
 
/*
    /////////////////To Clock LED Display/////////////////////////////
     hours = gps.time.hour();
   // int SW1 = digitalRead(DST);
   // int SW2 = digitalRead(EST);
    int SW3 = digitalRead(Twelve);

   // if (SW1 == LOW) DSTEST = -4;
   // if (SW2 == LOW) DSTEST = -5;
   // if (SW1 == HIGH && SW2 == HIGH) DSTEST = 0; // UTC
    if (SW3 == HIGH) {
      TwelveTwentyFour = 12;
    }
    else {
      TwelveTwentyFour = 24;
    }
   
    
 
    //hours = hours + timezonehr;
    if (hours > TwelveTwentyFour - 1) {
      hours = hours - TwelveTwentyFour;
      if (hours == 0) hours = 12; // make sure shows 12 at noon
    //  ClockPoint = (~ClockPoint) & 0x01;
      //if(ClockPoint)tm1637.point(POINT_ON);
  //else tm1637.point(POINT_OFF);
    }
    else {
      if (hours < 0) {
        hours = hours + TwelveTwentyFour;
      }
    }
    /////////////////To Clock LED Display/////////////////////////////
    minutes = gps.time.minute()+30;
    if (minutes >59){ minutes=minutes-60,hours=hours+6;}
    else {(hours = hours+5);}
    if (hours >12) {(hours=hours-12);}
//    if (minutes  != wasMinute) {
  //    Serial.print("Hours just before sent to LED ");
  //    Serial.println(hours);
     
    //  display.printTime(hours, minutes,blink=false );  // display time
    //  if (   hours <= 9) {
    //    display.print(" ");
    //  }
  //    wasMinute = minutes;
  //  }
  
 // else
 // {
  //  Serial.print(F("INVALID"));
 // }

//  Serial.println();*/
}

