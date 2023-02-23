# ESP32 BLE Keyboard & Mouse Combo library

This is a fork of the original [ESP32 BLE Keyboard & Mouse Combo library](https://github.com/Georgegipa/ESP32-BLE-Combo) -> [ESP32 BLE HID Combo library](https://github.com/peter-pakanun/ESP32-BLE-Combo)
which is based on the [BLE-Keyboard](https://github.com/T-vK/ESP32-BLE-Keyboard).

This library is a wrapper of the above fork in order to make it compatible with the [Keyboard](https://www.arduino.cc/reference/en/language/functions/usb/keyboard/) and [Mouse](https://www.arduino.cc/reference/en/language/functions/usb/mouse/).

This library fixes [the bugs found](https://github.com/T-vK/ESP32-BLE-Keyboard/issues) and adds some improvements.

## Features

 - All the features of the original library.
 - Compatibility with the [Keyboard](https://www.arduino.cc/reference/en/language/functions/usb/keyboard/) and [Mouse](https://www.arduino.cc/reference/en/language/functions/usb/mouse/) libraries.
 - Extra keyboard buttons

 - [x] Send key strokes
 - [x] Send text
 - [x] Press/release individual keys
 - [x] Media keys are supported
 - [x] Read Numlock/Capslock/Scrolllock state
 - [x] Set battery level (basically works, but doesn't show up in Android's status bar)
 - [x] Compatible with Android (tested on 10, 12 and 13)
 - [x] Compatible with Windows (tested on 11)
 - [x] Compatible with Linux (not tested)
 - [x] Compatible with MacOS X (not tested. Some people have issues, doesn't work with old devices)
 - [x] Compatible with iOS (tested on iPhone 13 Pro Max. Some people have issues, doesn't work with old devices)

## Installation
- (Make sure you can use the ESP32 with the Arduino IDE. [Instructions can be found here.](https://github.com/espressif/arduino-esp32#installation-instructions))
- [Download the latest release of this library from the release page.](https://github.com/Georgegipa/ESP32-BLE-Combo/releases)
- In the Arduino IDE go to "Sketch" -> "Include Library" -> "Add .ZIP Library..." and select the file you just downloaded.
- You can now go to "File" -> "Examples" -> "ESP32 BLE Keyboard" and select any of the examples to get started.

## Example

``` C++
/**
 * This example turns the ESP32 into a Bluetooth LE keyboard & mouse.
 * Writes the words, presses Enter, presses a media key.
 * In the end showcase the mouse functions.
 */
#include <BleKeyboard.h>
#include <BleMouse.h>

uint8_t BatteryLevel = 98;
unsigned long previousMillisBattery = 0; 

void setup() {
  Serial.begin(115200);

  bleDevice.setName("ESP32 Combo"); //call before any of the begin functions to change the device name.
  bleDevice.setManufacturer("Espressif"); //call before any of the begin functions to change the Manufacturer name.
  bleDevice.setDelay(40); //change the delay between each key event #if defined(USE_NIMBLE)

  Keyboard.begin();

  delay(3000);
  while (!bleDevice.isConnected()) {
    Serial.print(".");
    delay(250);
  }
  delay(1000);

  bleDevice.setBatteryLevel(BatteryLevel); //change the battery level

  Serial.println("...");
  Serial.println("BLE Connected!");
}

void loop() {
  Serial.println("Waiting 5 seconds...");
  delay(5000);
  if(bleDevice.isConnected()) {

    Serial.println("Sending 'Hello world'...");
    Keyboard.print("Hello world");
    delay(1000);

    Serial.println("Sending Enter key...");
    Keyboard.write(KEY_RETURN);
    delay(1000);

    Serial.println("Sending Play/Pause media key...");
    Keyboard.write(KEY_MEDIA_PLAY_PAUSE);
    delay(1000);

    //Serial.println("Sending Ctrl+Alt+Delete...");
    //Keyboard.press(KEY_LEFT_CTRL);
    //Keyboard.press(KEY_LEFT_ALT);
    //Keyboard.press(KEY_DELETE);
    //delay(100);
    //Keyboard.releaseAll();
    //delay(1000);

    Serial.println("Left click");
    Mouse.click(MOUSE_LEFT);
    delay(500);

    Serial.println("Right click");
    Mouse.click(MOUSE_RIGHT);
    delay(500);

    Serial.println("Scroll wheel click");
    Mouse.click(MOUSE_MIDDLE);
    delay(500);

    Serial.println("Back button click");
    Mouse.click(MOUSE_BACK);
    delay(500);

    Serial.println("Forward button click");
    Mouse.click(MOUSE_FORWARD);
    delay(500);

    Serial.println("Click left+right mouse button at the same time");
    Mouse.click(MOUSE_LEFT | MOUSE_RIGHT);
    delay(500);

    Serial.println("Click left+right mouse button and scroll wheel at the same time");
    Mouse.click(MOUSE_LEFT | MOUSE_RIGHT | MOUSE_MIDDLE);

    unsigned long currentMillis = millis();
    if(BatteryLevel>=4 && currentMillis - previousMillisBattery >= 3000) { // gradual discharge of the battery
      previousMillisBattery = currentMillis;
      BatteryLevel = BatteryLevel - 1;
      bleDevice.setBatteryLevel(BatteryLevel);
    }
  }
}
```

## API docs
In addition to that you can send media keys (which is not possible with the USB keyboard library). Supported are the following:
- KEY_MEDIA_NEXT_TRACK
- KEY_MEDIA_PREVIOUS_TRACK
- KEY_MEDIA_STOP
- KEY_MEDIA_PLAY_PAUSE
- KEY_MEDIA_MUTE
- KEY_MEDIA_VOLUME_UP
- KEY_MEDIA_VOLUME_DOWN
- KEY_MEDIA_WWW_HOME
- KEY_MEDIA_LOCAL_MACHINE_BROWSER // Opens "My Computer" on Windows
- KEY_MEDIA_CALCULATOR
- KEY_MEDIA_WWW_BOOKMARKS
- KEY_MEDIA_WWW_SEARCH
- KEY_MEDIA_WWW_STOP
- KEY_MEDIA_WWW_BACK
- KEY_MEDIA_CONSUMER_CONTROL_CONFIGURATION // Media Selection
- KEY_MEDIA_EMAIL_READER

### Bluetooth device settings
In order to change Bluetooth information you need to use the bleDevice object. You can ether the values of the default constructor on BleCombo.h or you can use the setter functions.
  
``` C++
bleDevice.setName("My ESP32"); //call before any of the begin functions to change the device name.
bleDevice.setBatteryLevel(80); //change the battery level to 80%
bleDevice.setDelay(100); //change the delay between each key event
bleDevice.isConnected(); //returns true if the device is connected to a host
```

### Read Numlock/Capslock/Scrolllock state
You can now query the state of the special Keys by using the variables (thanks [kilr00y](https://github.com/T-vK/ESP32-BLE-Keyboard/issues/179))

``` C++
bleDevice.NumLockOn
bleDevice.CapsLockOn
bleDevice.ScrollLockOn
```


## NimBLE-Mode
The NimBLE mode enables a significant saving of RAM and FLASH memory.

### Comparison (SendKeyStrokes.ino at compile-time)

**Standard**
```
RAM:   [=         ]   9.3% (used 30548 bytes from 327680 bytes)
Flash: [========  ]  75.8% (used 994120 bytes from 1310720 bytes)
```

**NimBLE mode**
```
RAM:   [=         ]   8.3% (used 27180 bytes from 327680 bytes)
Flash: [====      ]  44.2% (used 579158 bytes from 1310720 bytes)
```

### Comparison (SendKeyStrokes.ino at run-time)

|   | Standard | NimBLE mode | difference
|---|--:|--:|--:|
| `ESP.getHeapSize()`   | 296.804 | 321.252 | **+ 24.448**  |
| `ESP.getFreeHeap()`   | 143.572 | 260.764 | **+ 117.192** |
| `ESP.getSketchSize()` | 994.224 | 579.264 | **- 414.960** |

### How to activate NimBLE mode?

ArduinoIDE: Uncomment the first line in BleCombo.cpp
```C++
#define USE_NIMBLE
```

PlatformIO: Change your `platformio.ini` to the following settings
```ini
lib_deps = 
  NimBLE-Arduino

build_flags = 
  -D USE_NIMBLE
```

## Credits

Credits to [chegewara](https://github.com/chegewara) and [the authors of the USB keyboard library](https://github.com/arduino-libraries/Keyboard/) as this project is heavily based on their work!  
Also, credits to [duke2421](https://github.com/T-vK/ESP32-BLE-Keyboard/issues/1) who helped a lot with testing, debugging and fixing the device descriptor!
And credits to [sivar2311](https://github.com/sivar2311) for adding NimBLE support, greatly reducing the memory footprint, fixing advertising issues and for adding the `setDelay` method.
