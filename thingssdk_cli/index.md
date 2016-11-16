# thingsSDK CLI

__thingsSDK CLI__ is a tool to help professional JavaScript developers build Internet of Things (IoT) projects in a familiar way.

Projects created by thingsSDK CLI are set up to use the Espruino JavaScript runtime on the IoT device. thingsSDK CLI currently supports ESP8266 development boards.

## Installation

```bash
$ npm install thingssdk-cli -g
```

## Usage

Creating a project:

```bash
$ thingssdk new <project_name>
```

Changing in to the project directory, and install the libraries:

```
$ npm install
```

You'll have access to the following scripts `dev`, `deploy` and `repl`.

To push your project to device in development use the `dev` command:

```bash
$ npm run dev
```

In development the code isn't saved to the device. It allows you to debug and test your code before commiting to it. Once you're happy with your code you can use the `deploy` command:

```bash
$ npm run deploy
```

Deploying means you code is saved and will be available when the device reboots.

Run a REPL on the connected device with `repl`:

```bash
$ npm run repl
```


## Guides

* [Getting Started](./getting_started.md)
