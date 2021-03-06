# WARNING! I'm hack and slashing parts of this codebase, Expect it to have breaking changes from the forked codebase...

Wifi connection and configuration manager for ESP8266 and ESP32.

Forked from [https://github.com/nrwiersma/ConfigManager](https://github.com/nrwiersma/ConfigManager). The lib was close but still had some issues for my use case. So time to fork it and mod it.

This library was made to ease the complication of configuring Wifi and other
settings on an ESP8266 or ESP32. It is roughly split into two parts, Wifi configuration
and REST variable configuration.

# Requires

* [ArduinoJson](https://github.com/bblanchon/ArduinoJson) version 6

# Quick Start

## Installing
This forked lib isn't "released" yet, so it may break at any time. :-P (I plan on attack the begin function next... You have been warned) Once I have made the major changes I need to the lib. I will start locking in versions and publishing it to lib managers. But if you really want to add this forked version before I have made my breaking changes I guess you can, just don't moan to me when I break your things :-P

I use [PlatformIO](https://platformio.org/) over Arduino IDE cause I use VSCode for a ton of things. You should be able to use this lib by adding the following to your platformio.ini

```
lib_deps =
  https://github.com/CrosseyeJack/ConfigManager.git
```

## Usage

Include the library in your sketch

```cpp
#include <ConfigManager.h>
```

Create your `config` and `meta` structs. These are the definitions for your settings.

```cpp
struct Config {
    char name[20];
    bool enabled;
    int8_t hour;
    char password[20];
} config;

struct Metadata {
    int8_t version;
} meta;

```

> Note: These are examples of possible settings. The `config` is required, these values are arbitrary.

Initialize a global instance of the library

```cpp
ConfigManager configManager;
```

In your setup function start the manager

```cpp
configManager.setAPName("Demo");
configManager.setAPFilename("/index.html");
configManager.addParameter("name", config.name, 20);
configManager.addParameter("enabled", &config.enabled);
configManager.addParameter("hour", &config.hour);
configManager.addParameter("password", config.password, 20, set);
configManager.addParameter("version", &meta.version, get);
configManager.begin(config);
```

In your loop function, run the manager loop

```cpp
configManager.loop();
```

To access your config data through device code

```cpp
char* name = config.name;
int version = meta.version;
```

To change a value through device code

```cpp
strncpy(config.name, "New Name", 8);
meta.version = 8;
configManager.save();
```

### Additional files saved

Upload the ```index.html``` file found in the ```data``` directory into the SPIFFS.
Instructions on how to do this vary based on your IDE. Below are links instructions
on the most common IDEs:

#### ESP8266

* [Arduino IDE](http://arduino-esp8266.readthedocs.io/en/latest/filesystem.html#uploading-files-to-file-system)

* [Platform IO](http://docs.platformio.org/en/stable/platforms/espressif.html#uploading-files-to-file-system-spiffs)

#### ESP32

* [Arduino IDE](https://github.com/me-no-dev/arduino-esp32fs-plugin)

* [Platform IO](http://docs.platformio.org/en/stable/platforms/espressif32.html#uploading-files-to-file-system-spiffs)

# Documentation

## Debugging

### Enabling

By default, ConfigManager runs in `DEBUG_MODE` off. This is to allow the serial iterface to communicate as needed.
To turn on debugging, add the following line inside your `setup` routine:

```
DEBUG_MODE = true
```

### Using

To use the debugger, change your `Serial.print` calls to `DebugPrint`. Output will then be toggled via the debugger.

## Methods

### getMode
```
Mode getMode()
```
> Gets the current mode, **ap** or **api**.

### setAPName
```
void setAPName(const char *name)
```
> Sets the name used for the access point.

### setAPPassword
```
void setAPPassword(const char *password)
```
> Sets the password used for the access point. For WPA2-PSK network it should be at least 8 character long.
> If not specified, the access point will be open for anybody to connect to.

### setAPFilename
```
void setAPFilename(const char *filename)
```
> Sets the path in SPIFFS to the webpage to be served by the access point.

### setAPTimeout
```
void setAPTimeout(const int timeout)
```
> Sets the access point timeout, in seconds (default 0, no timeout).
>
> **Note:** *The timeout starts when the access point is started, but is evaluated in the loop function.*

### setAPCallback
```
void setAPCallback(std::function<void(WebServer*)> callback)
```
> Sets a function that will be called when the WebServer is started in AP mode allowing custom HTTP endpoints to be created.

### setAPICallback
```
void setAPICallback(std::function<void(WebServer*)> callback)
```
> Sets a function that will be called when the WebServer is started in API/Settings mode allowing custom HTTP endpoints to be created.

### setWifiConnectRetries
```
void setWifiConnectRetries(const int retries)
```
> Sets the number of Wifi connection retires. Defaults to 20.

### setWifiConnectInterval
```
void setWifiConnectInterval(const int interval)
```
> Sets the interval (in milliseconds) between Wifi connection retries. Defaults to 500ms.

### setWebPort
```
void setWebPort(const int port)
```
> Sets the port that the web server listens on. Defaults to 80.

### addParameter
```
template<typename T>
void addParameter(const char *name, T *variable)
```
or
```
template<typename T>
void addParameter(const char *name, T *variable, ParameterMode mode)
```
> Adds a parameter to the REST interface. The optional mode can be set to ```set```
> or ```get``` to make the parameter read or write only (defaults to ```both```).

### addParameter (string)
```
void addParameter(const char *name, char *variable, size_t size)
```
or
```
void addParameter(const char *name, char *variable, size_t size, ParameterMode mode)
```
> Adds a character array parameter to the REST interface.The optional mode can be set to ```set```
> or ```get``` to make the parameter read or write only (defaults to ```both```).

### clearWifiSettings(bool reboot)

> Sets SSID/Password to `NULL`
> The `bool reboot` indicates if the device should restart after clearing the values.

### clearSettings(bool reboot)

> Sets all settings to `NULL`. This is useful to initialize the memory of the device.
> The `bool reboot` indicates if the device should restart after clearing the values.

### begin
```
template<typename T>
void begin(T &config)
```
> Starts the configuration manager. The config parameter will be saved into
> and retrieved from the EEPROM.

### save
```
void save()
```
> Saves the config passed to the begin function to the EEPROM.

### loop
```
void loop()
```
> Handles any waiting REST requests.

### streamFile(const char &ast;file, const char mime[])

```
server->on("/settings.html", HTTPMethod::HTTP_GET, [server](){
    configManager.streamFile(settingsHTMLFile, mimeHTML);
});
```

> Stream a file to the server when using custom routing endpoints. See `example/save_config_demo.ino`


# Endpoints


## GET /

###### Modes: *AP and API*

> Gets the HTML page that is used to set the Wifi SSID and password.

+ Response 200 *(text/html)*

## POST /

###### Modes: *AP and API*

> Sets the Wifi SSID and password. The form example can be found in the ```data``` directory.

+ Request *(application/x-www-form-urlencoded)*

```
ssid=access point&password=some password
```

+ Request *(application/json)*

```json
{
  "ssid": "access point",
  "password": "some password"
}
```

## What the fuck happened to GET /scan ?

Well I felt the built in endpoint was flawed. But it can be re-added in setAPCallback

add a call back where you set up configmanager

```cpp
configManager.setAPCallback(configManagerAPStarted);
```

Then inside configManagerAPStarted add a scan endpoint. Something like this will do. Note: I just wrote this off the top of my head and not even compiled it :-P so YMMV, Worse case just rip the old /scan code out of the lib and C+P it here.

```cpp
void configManagerAPStarted(WebServer *server)
{
  server->on("/scan", HTTPMethod::HTTP_GET, [server]() {
    int n = WiFi.scanNetworks(); // Scan for networks
    if (n == WIFI_SCAN_FAILED || n == WIFI_SCAN_RUNNING) // Failed to complete scan, just return an empty array - you will prob want to send back an error and handle that error in your ajax call.
    {
      server->send(200, "application/json", "[]");
      return;
    }

    // Calculate the capacity needed to store the scanned AP's- If you tweek this then use https://arduinojson.org/v6/assistant/ to work out the capacity you will need
    const size_t capacity = JSON_ARRAY_SIZE(n) + n*JSON_OBJECT_SIZE(3));
    // n = the number of ssid's WiFi.scanNetworks() found, 3 = the number of params in each object
    // And boom (headshot) a DynamicJsonDocument the size needed to store the AP's without hard coding a max size
    // This is just a simple example, in your real code you might want to double that that you have the mem avaliable to create this doc befor actually creating it.
    
    // Create the json document
    DynamicJsonDocument networkJsonDoc = DynamicJsonDocument(capacity);
    // In my IRL implentations of this code, I normally keep networkJsonDoc as a global (So I can scan for networks in the background and not lock up the endpoint while the ESP scans for networks)

    // for loop though the scanned networks and slap them into networkJsonDoc
    // You might want to filter out any dupe SSID's here. Personally I left the dupes in and handle them in the JS on the config html page.
    for (size_t i = 0; i < n; i++)
    {
      JsonObject network = networkJsonDoc.createNestedObject();
      network["ssid"] = WiFi.SSID(i); // Add the SSID

      network["strength"] = WiFi.RSSI(i); // Add the RSSI

      network["security"] = (WiFi.encryptionType(i) == WIFI_AUTH_OPEN) ? false : true; // Add the protection status of this AP
      // In this example we are presuming that no one is using enterprise WPA - so just saying "yeah we gonna need a password from you to connect to this network"
    }
    WiFi.scanDelete(); // Delete the scan as its not needed any more. - not really needed but might as well.

    String json_string; // Create a Json String
    serializeJson(networkJsonDoc, json_string); // convert networkJsonDoc into a json string to send it over the wire. 

    server->send(200, "application/json", json_string); // Send the json string back to the client.
  });
}
```