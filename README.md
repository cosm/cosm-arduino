Cosm Arduino library
================

A library for Arduino to make it easier to talk to Cosm (the service formerly known as Pachube).

This library **requires** the HTTP Client library in https://github.com/arduino/Arduino/pull/69
(which should be included in Arduino 1.0.1.)

##Features

   1. Generic functions for:
   	- Uploading datapoints
   	- Downloading datapoints
   2. Compatible with
   	- Ethernet connections
   	- Arduino Wifi shield
   	- WiFly shield


##Setup Your Sketch

**1. Specify your API key and Feed ID**
```c
char cosmKey[] = "YOUR_COSM_API_KEY";
// Should be something like "HsNiCoe_Es2YYWltKeRFPZL2xhqSAKxIV21aV3lTL2h5OD0g"
#define FEED_ID XXXXXX 
// Should be something like 104097
```

**2. Create names for your datastreams as `char` arrays (or `String` objects for a `String` datastream)**
```c
// For datastreams of floats:
char myFloatStream[] = "humidity";
// For datastreams of ints:
char myIntStream[] = "temperature";
// For datastreams of Strings:
String myStringStream("my_thoughts_on_the_temperature");
// For datastreams of char buffers:
char myCharBufferStream[] = "more_thoughts";  // name of the array
const int bufferSize = 140;                   // size of the array
char bufferValue[bufferSize];                 // the array of chars itself
```

`String` datastreams and `char` buffer datastreams are similar: both will be able to send strings to Cosm datastreams. For beginners, using `String` datastreams will be fine much of the time. 

Using char buffers reduces the memory footprint of your sketch by not requiring the String library.  Also, using char buffers allows you to specify exactly how much memory is used for a datapoint, so you don't accidentally overflow the Arduino's mem capacity with a huge string datapoint.  It's a little bit harder to understand for beginners -- consult CosmDatastream.cpp for info.

**3. Create an array of `CosmDatastream` objects**
```c
CosmDatastream datastreams[] = {
  // Float datastreams are set up like this:
  CosmDatastream(myFloatStream, strlen(myFloatStream), DATASTREAM_FLOAT),
  // Int datastreams are set up like this:
  CosmDatastream(myIntStream, strlen(myIntStream), DATASTREAM_INT),
  // String datastreams are set up like this:
  CosmDatastream(myStringStream, DATASTREAM_STRING),
  // Char buffer datastreams are set up like this:
  CosmDatastream(myCharBufferStream, strlen(myCharBufferStream), DATASTREAM_BUFFER, bufferValue, bufferSize),
};
```
`CosmDatastream` objects can contains some or all of the following variables, depending on what type of datapoints are in the datastream (see above example for which are required):

| | Variable | Type | Description |
|---|---|:---:|---|
| 1     | aIdBuffer | char*|char array name of the char datastream
| 2     | aIdBufferLength |  int |for `int` or `float` datastreams only: the number of  `char` in the datastream name
| 3 | aType | int |**0** for String; **1** for Buffer; **2** for Int; **3** for Float
| 4 | aValueBuffer | char* | A `char` array, _aValueBufferLength_ elements long
| 5 | aValueBufferLength | int | The number of elements in the `char` array

    
**4. Last, wrap this array of `CosmDatastream` objects into a `CosmFeed`**
```c	
CosmFeed feed(FEED_ID, datastreams, 4);
```

| | Variable | Type | Description |
|---|---|:---:|---|
| 1     | aID | unsigned long | Feed number, as defined at the top of your sketch
| 2     | aDatastreams | CosmDatastream* |Your `CosmDatastream` array
| 3 | aDatastreamsCount | int | How many datastreams are in the array

**5. Instantiate the library's Cosm client**

Connecting by ethernet:

>If you're using the Ethernet library:
```c
EthernetClient client;
CosmClient cosmclient(client);
```


If you're on wireless, be sure to enter your ssid and password as the library requires, and then:
>If you're using the built-in WiFi library:
```c
WiFiClient client;
CosmClient cosmclient(client);
```

---
>If you're using the [Sparkfun WiFly] [1] library:
```c
WiFlyClient client;
CosmClient cosmclient(client);	
```
[1]: https://github.com/jmr13031/WiFly-Shield

##Sending and Retrieving Cosm Datapoints

###Read a Datapoint
```c
Serial.print("My return is: ");

float float_value = datastreams[0].getFloat();        // Retrieve a float datastream
int int_value = datastreams[1].getInt();              // Retrieve an int datastream
String string_value = datastreams[2].getString();     // Retrieve a String datastream
char[140] char_buffer = datastreams[3].getBuffer();   // Retrieve a char buffer datastream
```

###Update a Datapoint

The library makes it easy to upload data as strings or numbers.
```c
datastreams[0].setFloat(1.5);
datastreams[1].setInt(23);
datastreams[2].setString("Pretty comfy temperature");
datastreams[3].setBuffer("But quite dry");
```