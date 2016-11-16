# Getting Started with thingsSDK CLI

thingsSDK CLI is a command line application to manage Internet of Things applications written in JavaScript.

## Prerequisites 

You will need an ESP8266 development board flashed with the Espruino JavaScript runtime ([see the Flasher.js guides](../flasher.js/index.md)). You can think of Espruino as the Node.js of microcontrollers.

## Installation 

Install `thingssdk-cli` with npm globally.

 ```bash
 $ npm install thingssdk-cli -g
 ```
 
## Create a Project

To create a thingsSDK project use the `new` command followed by the project name. Replace `<project>` in the following command with a project name.

```bash
$ thingssdk new <project>
```

Then change in to the project folder.

```bash
$ cd <project>`
```
Then install the dependencies.`

```bash
$ npm install
```

## Hello World
Each thingsSDK project starts as with the Hello World code. All thingsSDK applications require a `main()` function to be implemented. The Hello World application blinks the blue LED on the device. It blinks every half a second.

```javascript
let isOn = false;
const interval = 500; // 500 milliseconds = 0.5 seconds

/**
 * The `main` function gets executed when the board is initialized.
 * Development: npm run dev
n * Production: npm run deploy
 */
function main() {
    setInterval(() => {
        isOn = !isOn; // Flips the state on or off
        digitalWrite(D2, isOn); // D2 is the blue LED on the ESP8266 boards
    }, interval);
}
```

To send the code to the device in development mode use the following npm script:

```bash
$ npm run dev
```

The `dev` command also starts an interactive REPL on the device so you can debug. Try typing `isOn` and seeing the value change overtime.

```
>isOn
=false
>isOn
=true
```

The development environment doesn't save the code on the device. In other words, if you unplug the device and plug it back in again the code you just sent is gone.

To send the code to the device and save it to memory in production mode use the `deploy` command.

```bash
$ npm run deploy
```
When you unplug the ESP8266 from your computer and plug it in to an external power source it will still have your code on the device.

## Other Commands

To connect to the device without sending any code use the `repl` command.

```bash
$ npm run repl
```

If you're cloning the project on another machine or working as a team you'll want to generate a `devices.json` machine for the new environment using the thingsSDK `devices` command.

```bash
$ thingssdk devices
```
