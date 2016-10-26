# Hello, World!

Every new thingsSDK project starts with a _Hello, World!_ example. The "Hello, World!" of electronics is a blinking LED (light).

Each ESP8266 has a blue LED that's connected to a the `D2` pin. Pins can be programatically triggered. In this example the `digitalWrite` function gets called which will either switch the LED on or off.

The following code will switch the LED off and on every 0.5 seconds.

```javascript
var isOn = false;
var interval = 500; // 500 milliseconds = 0.5 seconds

setInterval(() => {
    isOn = !isOn; // Flips the state on or off
    digitalWrite(D2, isOn); // D2 is the blue LED on the ESP8266 boards
}, interval);
```
