# LinkItONE
PubNub for the Mediatek Linkit ONE SDK
# PubNub LinkIt ONE Library 
This library enables your sketches to communicate with the PubNub Data Stream Network, using the Wi-Fi feature of the LinkIt ONE development board. Your application can publish messages and subscribe to channels using the PubNub API. A “Hello World” example client application is provided.
## Supported Hardware
The LinkIt ONE development board is based on the MediaTek MT2502 (Aster) chipset, in combination with high performance Wi-Fi (MT5931) and GNSS (MT3332) chipsets. In addition to enabling a wide range of Wearables and IoT proof-of-concepts to be developed, its built-in Wi-Fi feature makes it ideal for creating devices that use the features of the PubNub Data Stream Network. For more information on the LinkIt ONE development board and its hardware specifications, refer to the LinkIt ONE HDK web page. For information on setup and configuration of the LinkIt ONE development board, see the LinkIt ONE Get Started guide. 
## Library Installation
Sketches for the LinkIt ONE development board are created with the MediaTek LinkIt ONE SDK (for Arduino), a plug-in for the Arduino IDE that includes libraries for the LinkIt ONE API and a board firmware update tool. Before you can make use of the PubNub LinkIt ONE Library, you’ll need to install the LinkIt ONE SDK into your copy of the Arduino IDE. Instructions for doing this for Microsoft Windows or Apple Mac OS X are provided in the LinkIt ONE Get Started guide. 
Next, download the PubNub LinkIt ONE Library from its GitHub repository. Create a new folder named PubNub in your Arduino libraries folder. Move the contents of the PubNub LinkIt ONE Library to the newly created folder in Arduino\libraries\PubNub.
Congratulations, your PubNub LinkIt ONE Library is ready to use. 
## Wi-Fi Support Example
The following example project demonstrates the use of the PubNub LinkIt ONE library with the LinkIt ONE development board’s Wi-Fi. The example contains three files: ``PubNub.h `` (header file), ``PubNub.cpp`` (source file) and ``PubNubWifi.ino`` (Arduino sketch).
## Creating the Hello World Wi-Fi sketch (``PubNubWifi.ino``)
The sketch is created as follows:

1)	Include the libraries.

``#include "SPI.h"``
``#include "PubNub.h"``

2)	Include the Wi-Fi library path:

a)	For Arduino version 1.6.4. 

``#include <LWiFi.h>``
``#include "LWiFiClient.h"``

b)	For Arduino versions 1.5.6 and 1.5.7.

``#include "LWiFi\LWiFi.h"``
``#include "LWiFi\LWiFiClient.h"``

3)	Define a Wi-Fi access point to establish a connection.
###

``#define WIFI_AP "RP95" // provide your WIFI_AP name`` 

``#define WIFI_PASSWORD "355972054522595" //provide your WIFI password``

``#define WIFI_AUTH LWIFI_WPA``

4)	Define the PubNub publish and subscribe keys.

``char pubkey[] = "pub-c-94177ec3-f576-41d9-b062-e9f934be8228";``

``char subkey[] = "sub-c-b4b4edac-0934-11e5-9ffb-0619f8945a4f";``

5)	Initiate the ``setup()`` function:

a)	Assign on board LED pins to initial LOW voltage for publish and subscribe events.
###

    void setup()
    {
        pinMode(subLedPin, OUTPUT);
        pinMode(pubLedPin, OUTPUT);
        digitalWrite(subLedPin, LOW);
        digitalWrite(pubLedPin, LOW);

    
b)	Start serial communication at a baud rate of 9600 and print a message “Serial Setup” to the serial output. 
    
    Serial.begin(9600);
    Serial.println("Serial set up");
       

c)	Establish communication over Wi-Fi in a while loop, by calling the connect method of the LWiFi class. Once it’s connected, send the "Connected to AP" message to the serial output.

    Serial.println("Connecting to AP");
    while (0 == LWiFi.connect(WIFI_AP, LWiFiLoginInfo(WIFI_AUTH, WIFI_PASSWORD)))
    {
        Serial.println(" . ");
        delay(1000);
    }
    Serial.println("Connected to AP");     

d)	Begin communication with PubNub using the publish (pubkey) and subscribe (subkey) keys defined earlier. 

    PubNub.begin(pubkey, subkey);
    Serial.println("PubNub set up");
}

6)	Provide continuous communication with PubNub in the ``loop()`` function.

a)	Publish a message over the Wi-Fi connection.

  i)	Define a client object of LwiFiClient class and printout a message "publishing a message" to the serial output. The client object is then assigned to the PubNub published message. 

    void loop()
    {
        LWiFiClient *client;
        Serial.println("publishing a message");
        client = PubNub.publish(channel, "\"\\\"Hello world!\\\" she said.\""); 
    
ii)	If the Wi-Fi client is not available printout the "publishing error" message to the serial output, then wait for a second (1000 ms) by calling the delay(1000) function before trying to connect again. 
    
    if (!client) {
        Serial.println("publishing error");
        delay(1000);
        return;
    }   

iii)	Call the connected() method of the client object in a while loop, read the message and print it out to the serial output. 
    
    while (client->connected()) {
        while (client->connected() && !client->available()); // wait
        char c = client->read();
        Serial.print(c);
    }
    
iv)	Stop the publication of messages on PubNub by calling the stop() method of the client object, then blink the LED as confirmation with the flash(pubLEDPin) function. 

    client->stop();
    Serial.println();
    flash(pubLedPin);
    
b)	Subscribe to a channel and read a message using a Wi-Fi connection.

i)	Print out a message on the serial output indicating PubNub subscription status. Define a pclient object of the PubSubClient class and if it’s not available send an error message to the serial output.

    Serial.println("waiting for a message (subscribe)");
    PubSubClient *pclient = PubNub.subscribe(channel);
    if (!pclient) {
        Serial.println("subscription error");
        delay(1000);
        return;
    }    
    
ii)	In a while loop read the subscribed message and print it out on the serial output. Stop the communication and flash the LED as a confirmation by calling the flash(subLedPin) function. 

    while (pclient->wait_for_data()) {
        char c = pclient->read();
        Serial.print(c);
    }
    pclient->stop();
    Serial.println();
    flash(subLedPin);

c)	Retrieving message history. 

i)	The PubNub LinkIt ONE library also enables retrieval of message history. The functionality is similar to message publication (a) and subscribe and read messages (b). 

    Serial.println("retrieving message history");
    client = PubNub.history(channel);
    if (!client) {
        Serial.println("history error");
        delay(1000);
        return;
    }
    while (client->connected()) {
        while (client->connected() && !client->available()); // wait
        char c = client->read();
        Serial.print(c);
    }
    client->stop();
    Serial.println();

    delay(10000);
}

## Defining the Wi-Fi library in the PubNub LinkIt ONE library
If you are running Arduino IDE 1.6.4, the content of the PubNub.h header file should include the following library support for Wi-Fi communication. 

``#define PubNub_WiFi``
``#elif defined(PubNub_WiFi)``
``#include "LWiFiClient.h"``
``#define PubNub_BASE_CLIENT LWiFiClient``
``#else``
``#error PubNub_BASE_CLIENT set to an invalid value!``
``#endif``

If your Arduino IDE version is 1.5.6 or 1.5.7, use ``#include "LWiFi\LWiFiClient.h"`` instead of `` #include "LWiFiClient.h".``

Congratulations, you now have an example application to publish, subscribe and retrieve messages from a channel using LinkIt ONE Wi-Fi support with the PubNub LinkIt ONE Library. 

