---
layout: 	post
title:  	"The final countdown"
date:   	2013-04-24 15:51:31
permalink:	/countdown
---
In Flanders (Belgium) sixth-formers yearly celebrate <em>the last 100 days</em>. This tradition celebrates that high school is almost over, and that freedom is on its way. Students will usually dress up in costumes and party all day and night long. Most school also organize special activities like a breakfast or allow students to decorate the school buildings.

![100 days party](/projects/images/final6.jpg)

This year I was one of the revelers and since the event is the start of a countdown, I decided to make a big countdown timer to count down to our graduation.
# Hardware
At first I considered using large 7 segment display, but after some consideration I went with a large clock face that counts down from '100 days' to 'we survived!' (the theme of this year)

![The clock face](/projects/images/final1.jpg)

The clock face and hand are laser cuted out of 3 mm plywood, and all electronics and servo's are immediately attached to it. The board consists out of an ATmega8 running the Arduino boot loader and a RTC (ds1307).

Last year I noticed a network switch above the ceiling tiles, so I assumed there must power there. Popping my head through the ceiling confirmed that, so I just went ahead and installed the thing.

<blockquote><b>It's easier to ask forgiveness than it is to get permission.</b>  
<b>— Grace Hopper —</b></blockquote>

# Software
The software is very simple. It reads out the RTC, calculates the remaining number of days and then maps those value's to a position for the servo. The code is at the end of this post.

![the clock mounted](/projects/images/final2.jpg)
![the electronics](/projects/images/final4.jpg)
![the location of the clock](/projects/images/final9.jpg)
![more party](/projects/images/final5.jpg)
![more party](/projects/images/final7.jpg)
![more party](/projects/images/final8.jpg)



{% highlight c++ linenos%}

// Date and time functions using a DS1307 RTC connected via I2C and Wire lib

/* The following code is licensed to the public domain.
You should however probably not use it,
since it was written in a hurry and is quite crappy */ 

#include &lt;Wire.h&gt;
#include "RTClib.h"
#include  &lt;servo.h&gt;

const int ledPin =  13;      // the number of the LED pin
int ledState = 0;

int servoPin = 9;
int prevClockhandPosition = -1;
int servoMin = 2;
int servoMax = 163;

int daysInMonth[12] = {
  21, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};

int targetDay = 25;
int targetMonth = 6;
int targetYear = 2013;

int setRTC = 0;

RTC_DS1307 RTC;
Servo clockhand;

void setup () {
  Serial.begin(9600);
  Wire.begin();
  RTC.begin();
//  pinMode(16, OUTPUT); 
//  pinMode(17, OUTPUT);
  pinMode(ledPin, OUTPUT);      
//  digitalWrite(16, LOW);
//  digitalWrite(17, HIGH);

  if (! RTC.isrunning() || setRTC == 1) {
    Serial.println("RTC is NOT running!");
    //    following line sets the RTC to the date &amp; time this sketch was compiled
    RTC.adjust(DateTime(__DATE__, __TIME__));
  }
}

void loop () {
  DateTime now = RTC.now();

  int daysToTarget = 0;

  if (now.month() &lt;= targetMonth){  
    daysToTarget = targetDay - now.day();
    for (int i = now.month(); i &lt; targetMonth; i++){
      //    Serial.print("i= ");
      //    Serial.print(i);
      //    Serial.print(" DaysInMonth= ");
      //    Serial.println(daysInMonth[i-1]);
      daysToTarget += daysInMonth[i-1];
    }
  }

  Serial.print("Dagen tot proclamatie: ");
  Serial.println(daysToTarget);

  int clockhandPosition = map(daysToTarget, 0, 109, servoMin, servoMax);
  clockhandPosition = min(clockhandPosition, servoMax);
  clockhandPosition = max(clockhandPosition, servoMin);

  if (clockhandPosition != prevClockhandPosition){
    clockhand.attach(servoPin);
    clockhand.write(80);
    delay(1000);
    clockhand.write(clockhandPosition);
    delay(5000);
    clockhand.detach();
    prevClockhandPosition = clockhandPosition;
  }

  Serial.print("Servo hoek: ");
  Serial.println(clockhandPosition);
  Serial.println();

  ledState = !ledState;
  digitalWrite(ledPin, ledState);

  delay(3000);
  Serial.print(now.year(), DEC);
  Serial.print('/');
  Serial.print(now.month(), DEC);
  Serial.print('/');
  Serial.print(now.day(), DEC);
  Serial.print(' ');
  Serial.print(now.hour(), DEC);
  Serial.print(':');
  Serial.print(now.minute(), DEC);
  Serial.print(':');
  Serial.print(now.second(), DEC);
  Serial.println();

{% endhighlight %}
