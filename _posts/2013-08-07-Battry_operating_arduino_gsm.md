---
layout: post
authors: christian
title: Battery operating the Arduino GSM shield
category: M2M
---

**This is a follow up post on how to build a water temperature sensor using the arduino GSM shield and Xively. This post should make sense on its own, but if you want to read the first part you can find it [here](/blog/2013/08/01/m2m_adventures/).**

The biggest problem with the sensor we made in the first post, is that it cannot operate on battery power for any considerable amount of time. In our testing, using 8 AA-batteries, the sensor lasted for about 24hrs, which is not acceptable for most sensors. So, how can we make it run for longer periods of time?

![image](/assets/img/posts/m2m/battery.jpg)

## Power down the modem and LEDs
The first thing to do is to disconnect the GPRS connection and turn off the modem when it is not in use. The modem is the biggest energy consumer, reaching 2W in some configurations. So we need a way to switch it off and on again.

We start by making a function to turn on the modem, and set up the connection. This should seem familiar if you have read the previous post, as the code is more or less identical to its setup function. Whereas in the previous version of the program you might get away with not having a delay between connecting to GSM and setting up the GPRS connection, it becomes quite important here since we are disconnecting and connecting more than once. In our testing we usually had problems connecting to the network during rush-hour when using too small of a delay, so even if your code works right now, it might not tomorrow morning. I still had problems using 1s delay, so I bumped it up to 3s, and has not had any problems since. You might get away with a lower delay, but it should at least be 1s.

	void startConnection(){
	  while (notConnected) {
	    digitalWrite(3,HIGH);
	    if(gsmAccess.begin(PINNUMBER)==GSM_READY){
	      delay(3000);
	      if(gprs.attachGPRS(GPRS_APN, GPRS_LOGIN, GPRS_PASSWORD)==GPRS_READY){
	        notConnected = false;
	        digitalWrite(redLed, LOW);
	        digitalWrite(greenLed, HIGH);
	      }
	    }
	    else{
	      digitalWrite(redLed, HIGH);
	      delay(1000);
	    }
	  }
	}


Then lets look at the disconnect function. The heart of this function is the call to the shutdown() function. We use the notConnected boolean to check that we are connected, then try to shut down the modem. If it managed to shut down, we turn off all LEDs to save power while not connected.


	void closeConnection(){
	  while(notConnected==false){
	    if(gsmAccess.shutdown()){
	        delay(1000);
      	    digitalWrite(3,LOW);
	        notConnected = true;
	        digitalWrite(redLed, LOW);
	        digitalWrite(greenLed, LOW);
	    }
	    else{
	      delay(1000);
	    }
	  }
	}

We now have a way of turning the modem off and on in between measurements. So, in the loop function, we start by calling startConnection(), then measure temperature, upload the data, and end with calling closeConnection().

## Set serial pins to low
Setting the serial RX pin on the GSM shield low after shutdown saved us a lot of energy. This seems to be a problem that [is well known](http://forum.arduino.cc/index.php?topic=158811.0), and will be added in a future version of the library. In the meantime, we need to do this ourselves.

## Change GSM operating frequency
The Arduino GSM shield uses the Quectel M10 modem. This is a quad-band modem, meaning it can run on all four most common GSM frequencies. The modems [specifications](http://www.quectel.com/UploadFile/Product/Quectel_M10_GSM_Specification_V3.0.pdf) tells us that the modem uses 1W of power on 850/900 MHz, and 2W on 1800/1900 MHz. Our measurements showed the same results, even if the connection was considerably weaker on 1800 MHz than on 900 MHz. So changing the operating frequency of the modem will likely save us some energy. A simple way to change the modems operating frequency, is using the [Arduino Band Management](http://arduino.cc/en/Tutorial/GSMToolsBandManagement) program. When you change the frequency it is changed internally in the modem so you don't have to specify it every time you upload new code

When you run this program you chan either choose single frequencies (top three choises) or a combination of several frequencies. If you choose one of the combinations, the Arduino is going to choose to connect to the frequency with the strongest signal. This won't always be the best in terms of power, so in our case we use the DCS frequency specifically. In Norway (at least) this frequency is not used as much outside of the cities, so we will lose some portability. Make sure the frequency you choose is available in the area you are going to use your Arduino.

## Let the Arduino sleep
There are several pages on the internet explaining how to make your Arduino sleep, but the by far easiest way to do this is to use Rocket Scream's [LowPower library](https://github.com/rocketscream/Low-Power). From the code below, you can see that we call LowPower with the powerDown function. This puts the Arduino to sleep and sets the watchdog timer to wake it up. The ATmega328 supports sleep up to 8 seconds, so we set the timer to that in the parameters of the function. We also turn off the Analog to Digital Converter and the Brownout Detector to save even more energy.

The biggest problem with making the Arduino sleep, is that it stops counting running milliseconds while in sleep. Therefore the previous way of posting values every five minutes, won't work anymore. Instead we start counting sleep-cycles. Counting 25 sleep cycles plus the time it takes to get and upload data, we get data spaced somewhere between 4 and 5 minutes apart. You can choose how many sleep cycles you want to wait between each measurement, and of course the battery life will improve some with a longer interval (more cycles). If you want to read more about sleep and the watchdog timer, you can take a look at the [ATmega328 documentation](http://www.atmel.com/Images/doc8161.pdf) and [Rocket Scream's own description](http://www.rocketscream.com/blog/2011/07/04/lightweight-low-power-arduino-library/) of their library.

	void loop(void) {
	  if(wait>=25){
	    startConnection();
	    temperature = temp.getTemp(water);
	    datastreams[0].setFloat(temperature);
	    temperature = temp.getTemp(air);
	    datastreams[1].setFloat(temperature);
	    int ret = xivelyclient.put(feed, xivelyKey);
	    if(ret == 200){
	      digitalWrite(greenLed, LOW);
	      delay(100);
	      digitalWrite(greenLed, HIGH);
	    }
	    else{
	      digitalWrite(redLed, HIGH);
	      delay(100);
	      digitalWrite(redLed, LOW);
	    }
	    closeConnection();
	    wait=0;
	  }	  	  
	  LowPower.powerDown(SLEEP_8S, ADC_OFF, BOD_OFF);	  
	  wait++;
	}
	
## Other things to consider
The Arduino was made to be versatile, not to be used for low power projects. Using the voltage regulator and other standard features of the Arduino, we loose a lot more energy than what we would if we made a specific board for this specific use case. This guide was made to be simple for others to follow, so we decided to only do changes to the program, not the hardware. If you want to lower the power consumption further than we have done here, you should take a look at [gammon.com.au/power](http://gammon.com.au/power) which has a lot of good tips.

All the code used in this, and the previous post is available at [https://github.com/christiandt/BeachSensor](https://github.com/christiandt/BeachSensor)