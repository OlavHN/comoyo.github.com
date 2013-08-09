---
layout: post
authors: [caroline, christian]
title: Measuring water and air temperature with Arduino GSM shield
category: M2M
---

If you didn't know, Comoyo's offices in Oslo are located pretty close to the water. With the warm weather we've been having lately, many Comoyans spend their lunch break down by the beach, some even bringing towels and swimming gear. That gave us an idea: What better way to demonstrate machine to machine communication than to make a combined water/air temperature sensor? Making and demonstrating a working M2M prototype is the goal of the summer intern project we have been assigned to, so we might as well make something fun while we’re at it!

![image](/assets/img/posts/m2m/matias-swimming.jpg)
*Software Engineer Matias Holte enjoying the water at Fornebu, complete with sombrero*

That decided, we went out to survey the test site to look for a good placement spot. We found a nice place under the pavilion deck, with a practical crossbar where we could place the test unit.

![image](/assets/img/posts/m2m/placement.jpg)

A problem with measuring ocean-connected water is that it is affected by the tides – at high tide, the water at the site is a lot deeper, so we need to wade to the spot. Thankfully, the weather is still warm for now, or it could be a cold affair! 
Measuring water temperature in salt water, and outdoors to boot, brings with it some challenges. Salt water and electronics don't mix well, to put it mildly. So we needed to make sure to keep things dry and safe, while still being able to measure accurately. Our solution to this was to put things in a sturdy plastic food storage container, as these are usually pretty water-tight, and then make holes for the sensors and sealing them properly around the leads.

## Materials 

For this project, we used quite a lot of stuff, ordered off Sparkfun, and bought at Clas Ohlson:

* [Arduino Uno](http://arduino.cc/en/Main/arduinoBoardUno)
* [Arduino GSM shield](http://arduino.cc/en/Main/ArduinoGSMShield)
* Dallas Semiconductor [DS18B20](https://www.sparkfun.com/products/245) one-wire temperature sensor, regular and waterproof
* [Tri-pad stripboard](http://en.wikipedia.org/wiki/Stripboard#TriPad)
* SIM card with prepaid plan
* [Battery box, battery snap, and AA batteries](http://www.clasohlson.com/no/Batteriholder/22-1545)
* LEDs, red and green
* Various resistors from a resistor kit
* Electrical wire, breakaway headers
* Cable ties and luggage straps
* Sturdy plastic box with snap lid

![image](/assets/img/posts/m2m/equipment.jpg)

For our temperature sensors, we chose a waterproof version for measuring the water, and used heat-shrink tubing to protect the regular sensor we use for air temperature measurements. We decided that air temperature measurements were also relevant, as few people like to go swimming in chilly weather even if the water is quite warm.

Arduino shields are boards that add functionality to the Arduino, plugging in to the pin headers on it. For this project, we use a [GSM shield](http://arduino.cc/en/Main/ArduinoGSMShield), to be able to send our data to our back-end service.

The GSM shield requires an activated SIM card to be able to connect to the mobile network. The GSM shield comes with a Telefónica/Bluevia SIM that can be activated, but it does not work in Norway, so we had to get a Telenor SIM. Seeing as Comoyo is part of Telenor, this was probably one of the easier parts of this project.

We decided that the easiest way to connect rest of the the components to the Arduino was to make a sort of shield out of stripboard by soldering on male pin headers. Then, we soldered on the connections and components.

![image](/assets/img/posts/m2m/shield-stripboard.jpg)
*Unfinished stripboard fitting on top of the GSM shield. In use, the stripboard will be pushed all the way down, so that the male pin headers are not visible like here.*

## Hardware design

![image](/assets/img/posts/m2m/beachSensor.png)

Having decided on the components and how to attach them, we needed to make a wiring diagram. We used Fritzing, an open-source hardware prototyping tool, to design it. Note that the pin headers are not included, instead the pins used are added as wires directly to the Arduino board. Also, the component placement is mirrored, due to the stripboard only having tracks on one side, and that side facing down. Thankfully, this problem was discovered fairly early on, and only a few components needed to be de-soldered. In the end, we had to be a little creative with the placement of the status LEDs, since they had to be visible and thus on the same side as the tracks.

![image](/assets/img/posts/m2m/finished-stripboard.jpg)

## Setting up a backend
Xively (formely Pachube, Cosm) is a cloud platform service for connecting devices to the Internet of Things. It offers an easy way to send data to the service, and an easy way to make applications based on the service and data. There are several other such services as well, but Xively seemed to be the easiest to use, providing plenty of documentation.

Once we have signed up, making a new device feed is very easy. In the developer workbench tab there is a button “+ Add Device”. When adding a device, we can select the device name, description and privacy settings. This brings us to the basic overview screen for this device. To be able to collect any data, we need to start one or more channels. Just like with the devices, there’s a big button that says “+ Add Channel”. Once this is done, keep in mind your channel and device name, and copy the API Key and feed number (locations circled in the image). If you would like to limit permissions, you can even generate a new API Key (using the “+ Add Key” button) with different permissions than the default auto-generated key. More on where to use these keys in the next section.

![image](/assets/img/posts/m2m/xively-details.png)

Once you do have data, Xively can show them to you in a handy graph, and the number of feed requests are logged, enabling you to keep an eye on what is going on with your device. You can even make Xively trigger HTTP POST requests on various conditions, if you like, by using the “+ Add Trigger” button below the API Key section. In our case, we’ve set Xively to trigger a POST request to an online web application syncing service when the device feed doesn’t receive any data for a certain period of time. The syncing service then sends us an e-mail about this.


## Programming the sensor
In this project we use two input pins for the temperature sensors, and two output pins for the status LEDs. We use pins 8 and 10 for output and 12 and 13 for input, but you may use any available pins. The temperature sensors we use transmit data over a OneWire bus, but we have chosen to use one pin per sensor (which sort of makes OneWire unnecessary). We define the input pins as integers this way:

    int air_pin = 12;
    int water_pin = 13;
    int redLed = 10;
    int greenLed = 8;

The program starts by including various libraries. [GSM.h](http://arduino.cc/en/Reference/GSM) is needed to interface with the GSM shield, [HttpClient.h](https://github.com/amcewen/HttpClient) and [Xively.h](https://github.com/xively/xively_arduino) are needed to post values to Xively, and [OneWire.h](http://www.pjrc.com/teensy/td_libs_OneWire.html) adds support for communicating with OneWire devices. The [Temperature.h](https://github.com/christiandt/BeachSensor/tree/master/Temperature) library is a library we have put together, based on [this Buildr example](http://bildr.org/2011/07/ds18b20-arduino/) to make it easier to get temperature data from the sensors without knowing too much about OneWire. If you would like to learn more about what OneWire is all about, you can check out [this page](http://www.pjrc.com/teensy/td_libs_OneWire.html) and the links on it. 

    #include <GSM.h>
    #include <HttpClient.h>
    #include <Xively.h>
    #include <OneWire.h> 
    #include <Temperature.h>

Next we define the variables used for the GSM shield. You need the PIN code for your SIM card, the APN (e.g. telenor), username and password (in our case, both are the phone number). You can find the information for your carrier [here](http://forums.pinstack.com/f24/tcp_apn_wap_gateway_port_carrier_settings-360/). This information is needed to set up a GPRS connection to transmit data over the GSM network. We then instantiate three objects of type GSMClient, GPRS and GSM.

    #define PINNUMBER "PIN"
    #define GPRS_APN "APN"
    #define GPRS_LOGIN "USERNAME"
    #define GPRS_PASSWORD "PASSWORD"

    GSMClient client;
    GPRS gprs;
    GSM gsmAccess;

Then we create two OneWire objects, one for each sensor, using the pin number as required by the constructor. Since we are not using more than one sensor per pin, these variables are different for each object.

    OneWire water(water_pin);
    OneWire air(air_pin);

We also need to provide some information to the Xively library for our device to be able to transmit our data where we want it. First we define the feed ID, which is found at the top of the feed's web page on Xively.com. Then we make three char arrays. One for the API key (again, this can be found on the feed page), one for the water sensor stream, and one for the air sensor stream. The name of the streams need to match their name on the feeds page. Next we create the datastreams, feed, and tell Xively which client to use to upload our data. Lastly, we create the Temperature object we will use to retrieve data from the sensors.

    #define FEED_ID 123456789
    char xivelyKey[] = "API_KEY";

    char myWaterTempStream[] = "water";
    char myAirTempStream[] = "air";
    unsigned long lastTime = millis();


    XivelyDatastream datastreams[] = { 
      XivelyDatastream(myWaterTempStream, strlen(myWaterTempStream), DATASTREAM_FLOAT),
      XivelyDatastream(myAirTempStream, strlen(myAirTempStream), DATASTREAM_FLOAT),
    };

    XivelyFeed feed(FEED_ID, datastreams, 2);
    XivelyClient xivelyclient(client);

    Temperature temp;

Now that we have finished defining variables and objects, there are still two functions left: Setup and loop. The setup function runs once when the Arduino is powered, or when the reset button is pressed. We use this to define our inputs and outputs, and start the GPRS connection. We use a boolean variable called notConnected to keep track of connection status. The function [gsmAccess.begin()](http://arduino.cc/en/Reference/GSMBegin) starts the modem, and returns the status of the modem. [gprs.attachGPRS()](http://arduino.cc/en/Reference/AttachGPRS) initiates the GPRS connection and returns the status of the connection. While this is not shown in the Arduino examples, these two functions should be called with a delay in between. If the modem returns GSM_READY, and the connection returns GPRS_READY, we are connected and we update notConnected. When we are connected, we light up the green LED, otherwise the red LED is lit.

    void setup(void) {
     pinMode(redLed, OUTPUT);
     pinMode(greenLed, OUTPUT);
     boolean notConnected = true;
     while (notConnected) {
        digitalWrite(redLed, HIGH);
        digitalWrite(greenLed, LOW);
        if(gsmAccess.begin(PINNUMBER)==GSM_READY){
          delay(3000);
          if(gprs.attachGPRS(GPRS_APN, GPRS_LOGIN, GPRS_PASSWORD)==GPRS_READY){
            notConnected = false;
            digitalWrite(redLed, LOW);
            digitalWrite(greenLed, HIGH);
          }
        }
        else{
          delay(1000);
        }
      }
    }

The loop function runs continuously as long as the Arduino is powered, only stopping for interrupts, and is where we will do our measurements and upload to Xively. The first thing we do is to check whether one minute (60000ms) has passed. This interval determines how often measurements are done and uploaded. The .getTemp() function in the Temperature library returns the temperature of a sensor using the given OneWire object as a float value. We use this to get the temperature of both our sensors, and put the values into the two data streams.

    if((millis()-lastTime)>=60000){
        lastTime = millis();
        temperature = temp.getTemp(water);
        datastreams[0].setFloat(temperature);
        temperature = temp.getTemp(air);
        datastreams[1].setFloat(temperature);

We now have everything we need, and are ready to post the values to Xively. This is done with the xivelyclient.put() function. It takes the feed and API key as input parameters and returns the response code. If the submission was successful (response code 200) we blink the green LED, otherwise we blink the red LED. 

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

If everything works according to plan, we should now have a working sensor that is uploading two temperature values to Xively, and displaying connection information using two LEDs. You can download the full version of the code over at GitHub: [https://github.com/comoyo/BeachSensor](https://github.com/comoyo/BeachSensor)

While this code runs well using wall power, it will probably not run longer than 24 hours if you are going to run the sensor on normal batteries. In a later post we will go through some of the things we have done to improve battery life.

## Weather-proof enclosure
We chose a sturdy 1-litre food storage box with snap lid to accommodate both the Arduino and GSM shield as well as the battery box. It also has a good seal on the lid. The difficulty lay in making a hole in it that was large enough, but without damaging the rest of the box. We also needed to be able to re-seal the box around the cable pulled through, to keep the water out. We made several attempts with screwdrivers, box cutters and wire cutters, but ended up melting through the box using the soldering iron, with the temperature turned down to avoid burning the plastic. Melting plastic is probably not very good for a soldering iron, but it’s the only thing that worked.

![image](/assets/img/posts/m2m/watertight.jpg)
*We may have gone slightly overboard with the glue gun. Notice the scratches from earlier attempts at making a hole.*

While the placement of the sensor leads works well with the components inside, they are slightly too close to the lid snaps, making opening and closing the box more difficult. A second revision would probably have them placed closer to the bottom of the box.

Once the box was finished, it was quite easy to strap the box to the beams at the test site with a luggage strap, and using cable ties to fasten the water probe to the deck pillar. The luggage strap is adjustable, making opening the box and switching the batteries relatively easy.

As this post is getting quite long, we're going to have to round it off here - stay tuned for further adventures in machine to machine communication! We'll be discussing power usage and battery issues, as well as some problems we encountered during the project.
