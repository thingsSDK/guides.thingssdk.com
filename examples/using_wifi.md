# Using WiFi
TL;DR - <a href="#example-code">See the code</a>

## Barebones Example
While blinking lights and triggering button effects is fun, we are building the _internet_ of things. At some point that means we need to get access to the internet. There are a few different ways to use wifi on a JavaScript thing. Espruino has an awesome core library we'll use here called "Wifi".

We can include it in our projects like so:
```javascript
// main.js
const wifi = require('Wifi')  // This is one of our magic, native Espruino friends.
```
Remember that - by default - main.js is always our entry point for a ThingsSDK app.
Cool, now we've got a friendly helper. How do we connect to the internet? Probably the easiest way is to connect directly using your own home wifi network.

**DISCLAIMER**: You need to be careful where you put your personal home wifi credentials. You don't want malicious parties to have access to them. My advice for this simple exercise is to put your credentials in a seperate file like so:

```javascript
// wifi.config.js
module.exports = {
  name: "YOUR_NETWORK_NAME",
  password: "YOUR_WIFI_PASSWORD"
}
```
You can just write them in by hand as you would when getting wifi access on your home computer or mobile device. For example, if my home network appears in the wireless menu on my Mac as _JavaScriptRULEZ_, and my password is _n0DOu8T!_ I would enter `name: "JavaScriptRULEZ", password: "n0DOu8T!"` in wifi.config.js.

Now if you plan to publish your code anywhere (like Github), you should add wifi.config.js to a .gitignore file in your project, or add it to your global .gitignore settings. See [the official Github article on ignoring files](https://help.github.com/articles/ignoring-files/) for more help using gitignore settings.

So now we can import our configuration like so:
```javascript
// main.js
const wifi = require('Wifi')
const { name: WIFI_NAME, password: WIFI_PASSWORD } = require('./wifi.config.js')
```

A method on the wifi module accepts our configuration parameters and handles connecting to the wifi for us.
```javascript
// main.js
...
// wifi.connect(ssid, options object, callback)
wifi.connect(WIFI_NAME, { password: WIFI_PASSWORD }, error => {
  if (error) console.error(error)
  else console.log(`Connected to: ${ wifi.getIP().ip }`)
})
```

Now's a good time for a sanity-check. Save the file and try running `npm run push` from your project root. You should see something like this in your terminal:
```bash
$ espruino:   _____                 _ 
|   __|___ ___ ___ _ _|_|___ ___ 
|   __|_ -| . |  _| | | |   | . |
|_____|___|  _|_| |___|_|_|_|___|
          |_| http://espruino.com
 1v87 Copyright 2016 G.Williams

Espruino is Open Source. Our work is supported
espruino:  only by sales of official boards and donations:
http://espruino.com/Donate
Flash map 4MB:512/512, manuf 0xef chip 0x4016

espruino:  >echo(0);
espruino:  =undefined

> Wifi_Example@0.0.0 repl /Users/joe/Experimental/Things/Wifi_Example
> node ./scripts/repl

Connecting REPL...
espruino:  Espruino Command-line Tool 0.0.21
-----------------------------------

espruino:  Connecting to '/dev/cu.SLAB_USBtoUART'
espruino:  Connected
Connected to: 192.168.1.131
```

Tada! Technically you're done! That last line is all the evidence we need of our device having wifi access. We are now free to run off and do all kinds of internet-connected things like GET from and POST to external APIs, store data, send messages, whatever!

## Adding Some Niceties
Let's spend a bit more time here though. Do we really want to run all of our application logic from inside this wifi.connect callback?

In one sense, we have no choice. But we can make this code a bit more composable by adding some functions and leveraging native promises.

First, let's wrap our current `wifi.connect` code with a new function that will accept our personal wifi parameters and return a new promise with the results of `wifi.connect`.

```javascript
// main.js
...
const connect = (networkName, options) => {
  console.log(`attempting to connect to network named: ${ networkName }`)
  return new Promise((resolve, reject) => wifi.connect(networkName, options, error => {
      const ipAddress = wifi.getIP().ip
      if (error) reject(error)
      else resolve(ipAddress)
    })
  )
}
```

That buys us at least two things.
1. We are now notified in the repl when the device first starts its connection attempt.
2. We get a Promise back that we can chain from and pass around as arguments to other functions. This gives us a lot more freedom over when our code runs and how it's written.

Before we `npm run push` again, we need to call our new function.
```javascript
// main.js
...
connect(WIFI_NAME, { password: WIFI_PASSWORD })
  .then(ip => {
    console.log(`successfully connected to ${ ip }`)
  })
  .catch(error => console.error(error))
```

Since we `resolve()`'d our Promise in `connect` with `ipAddress` as its value, we have access to that inside our `then()` block. There are other properties you might be interested in accessing, but for our purposes here we don't really care. We only want to confirm that we connected successfully so we can forget about and do our cool internet stuff.

`$ npm run push` should show something like this toward the end of the terminal output.

```bash
successfully connected to 192.168.1.131

> Wifi_Example@0.0.0 repl /Users/joe/Experimental/Things/Wifi_Example
> node ./scripts/repl

Connecting REPL...
espruino:  Espruino Command-line Tool 0.0.21
-----------------------------------

espruino:  Connecting to '/dev/cu.SLAB_USBtoUART'
espruino:  Connected
```

## Tracking Our Wifi Connection
The wifi module gives us a few more handy features we'll likely want. For example, it gives us a convenient way to do stuff at the moment we connect, disconnect and disconnect. Let's have our D2 LED light keep track of our current connected status, just like our home wifi routers. This is just one example of chaining app logic onto our connection promise.

Let's start with some convenient functions for toggling the LED light. I've chosen to write two different functions to avoid a state variable somewhere else in the program that tracks whether the light is currently on or off.
```javascript
// main.js
...
const lightOn = () =>{
  digitalWrite(D2, isOn = false) // on my Feather Huzzah board, the led is v0, so it's counterintuitive: `false` means on
  console.log('A blue LED should be on...')
 }

const lightOff = () => {
  digitalWrite(D2, isOn = true)
  console.log(`The blue light should be off...`)
}

connect(WIFI_NAME, { password: WIFI_PASSWORD })
  .then(ip => {
    console.log(`successfully connected to ${ ip }`)
    lightOn()
  })
  .catch(error => console.error(error))
```

`$ npm run push` and you should see your LED light on! Cool, let's use an event listener to turn it off when we lose signal. A good place to set up these wifi lifecycle listeners is after we've connected. So since we have a nice promise chain running, we'll just do that in the next `then()` block.

```javascript
// main.js
...
connect(WIFI_NAME, { password: WIFI_PASSWORD })
  .then(ip => {
    console.log(`successfully connected to ${ ip }`)
    lightOn() // Again, this is just a reminder that we are wifi connected now
  })
  .then(() => {
    wifi.on('disconnected', () => {
      lightOff()
      console.log(`successfully disconnected...`)
    })
  })
  .then(() => setTimeout(wifi.disconnect, 3000) )
  .catch(error => {
    console.error(error)
  })
```
`wifi.on('disconnected',())` come out-of-the-box with the Espruino Wifi module. If we wanted we could wrap these event listeners in promises too, but in this case flipping the light is simple enought hat we don't need to.

Ok, so the light turns on when we connect, and it's supposed to turn off when we disconnect. Let's add a final `then()` block to test it out.

```javascript
// main.js
...
connect(WIFI_NAME, { password: WIFI_PASSWORD })
  .then(ip => {
    console.log(`successfully connected to ${ ip }`)
    lightOn() // Again, this is just a reminder that we are wifi connected now
  })
  .then(() => {
    wifi.on('disconnected', () => {
      lightOff()
      console.log(`successfully disconnected...`)
    })
  })
  .then(() => setTimeout(wifi.disconnect, 3000) )
  .catch(error => {
    console.error(error)
  })
```

`wifi.disconnect()` is the built-in counterpart to `wifi.connect()`. Here we say that once we've connected and our listeners are in place, wait 3 seconds and then disconnect.

`$ npm run push` and you should your light turn on once connected, and turn off after three seconds!

One last thing we want to do is add a special line that tells our device _NOT_ to act as a wifi access point for other devices (which it currently does in Espruino by default).

```javascript
// main.js
...
wifi.stopAP() // Don't act as a Wifi access point for other devices
```

Not too exciting since we're not actually using the internet to do any internet stuff here. See [Using and HTTP API](https://github.com/thingsSDK/guides.thingssdk.com/blob/master/examples/using_an_http_api.md) for more on that part.

<h2 id="example-code">Example Code</h2>
```javascript
// main.js
const wifi = require('Wifi')  // This is one of our magic, native Espruino friends.

// If you plan to publish your code,
// it's a good idea to keep your wifi name and password in a secure file that you do not version control.
// In this case I've named my file "Wifi_Config.js"
const { name: WIFI_NAME, password: WIFI_PASSWORD } = require('./wifi.config.js')


// Wrap the call to wifi.connect in a native Promise so it's a bit easier to deal with later
const connect = (networkName, options) => {
  console.log(`attempting to connect to network named: ${ networkName }`)
  return new Promise((resolve, reject) => wifi.connect(networkName, options, error => {
      const ipAddress = wifi.getIP().ip
      if (error) reject(error)
      else resolve(ipAddress)
    })
  )
}

const lightOn = () =>{
  digitalWrite(D2, isOn = false)
  console.log('A blue LED should be on...')
 }

const lightOff = () => {
  digitalWrite(D2, isOn = true)
  console.log(`The blue light should be off...`)
}

// Save a reference to the attempt that we can pass around.
const connectionAttempt = connect(WIFI_NAME, { password: WIFI_PASSWORD })
  .then(ip => {
    console.log(`successfully connected to ${ ip }`)
    lightOn() // Again, this is just a reminder that we are wifi connected now
  })
  .then(() => {
    // From here on you have wifi access.
    // You can do all your app stuff in this or subsequent .then() blocks.
    // One good idea is to set up a "disconnected" event listener that allows you to handle
    // lost wifi
    wifi.on('disconnected', () => {
      lightOff()
      console.log(`successfully disconnected...`)
    })
  })
  // Here's an example of some app code that comes after we've set up all our wifi stuff.
  // In this case I just trigger a disconnect for demonstration purposes.
  // But you could have all your app's business logic playout here.
  .then(() => setTimeout(wifi.disconnect, 3000) )
  .catch(error => {
    console.error(error)
  })

wifi.stopAP() // Don't act as a Wifi access point for other devices

```
