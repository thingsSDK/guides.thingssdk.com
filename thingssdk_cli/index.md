# thingsSDK CLI

__thingsSDK CLI__ is a tool to help professional JavaScript developers build IoT projects in a familiar way.

Projects created by thingsSDK CLI are set up to use the Espruino JavaScript runtime on the IoT device.

## Installation

```bash
$ npm install thingsSDK/thingssdk-cli -g
```

**TODO: Use correct package name on release!
**

## Usage

Creating a project:

```bash
$ thingssdk new <project_name>
```

Changing in to the project directory you'll have access to the following scripts `push` and `repl`.

Deploying project to device:

```bash
$ npm run push
```

Running a REPL on the connected device:

```bash
$ npm run repl
```


## Guides

* [Getting Started](./getting_strated.md)
