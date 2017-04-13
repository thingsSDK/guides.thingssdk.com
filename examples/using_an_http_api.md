# Using an HTTP API

In this chapter we'll cover the basics of making an HTTP request to an API.

For this example, we don't need any wiring, as we will use the REPL interactive console to display the data from the API. 

## Setup
### New ThingsSDK Project
The first step is start a new Things project. Plug your device into your desired USB port. Then in your terminal run:
```bash
$ thingssdk new HTTP_API
```

You should see this in your console:
```bash
thingssdk new HTTP_API
? Select a port: (Use arrow keys)
? Select a port: # This will show your port
? Select the baud rate: (Use arrow keys)
? Select the baud rate: 115200
To install the project dependencies:
    cd HTTP_API && npm install
To upload to your device:
    cd HTTP_API && npm run deploy
Project successfully created
```

Follow the prompts to get our dependencies installed:

```bash
$ cd HTTP_API && npm install
```

When everything is installed, `$ npm run dev` sends the code to your device and runs it. I like to perform a quick sanity check with the default hello world code just to make sure everything is correctly connected.

```bash
$ npm run dev
```

## The Code
### Connect to WIFI
There is an [entire chapter of this guide](https://guides.thingssdk.com/examples/using_wifi.html) dedicated to how to connect to a WIFI network.

Here's some boilerplate code:

```javascript
// main.js

const wifi = require('Wifi')


// wifi.connect(ssid, options object, callback)
wifi.connect(WIFI_NAME, { password: WIFI_PASSWORD }, error => {
  if (error){
  	console.error(error)
  }else{
  	console.log(`Connected to: ${ wifi.getIP().ip }`)
  	//you are connected to WIFI!
  }
})
```

### Make a GET Request

Making a request is really easy: here we will use the Yahoo Weather APIs to get the current condition in Dallas, TX. (I'm using one of the examples in the Yahoo APIs Documentation: https://developer.yahoo.com/weather/).


```javascript

const url="https://query.yahooapis.com/v1/public/yql?q=select%20item.condition.text%20from%20weather.forecast%20where%20woeid%20in%20(select%20woeid%20from%20geo.places(1)%20where%20text%3D%22dallas%2C%20tx%22)&format=json&env=store%3A%2F%2Fdatatables.org%2Falltableswithkeys";

require("http").get(url, function(res) {
    let c = "";
    res.on('data', function(data) {
      c += data;
    });
    res.on('close', function(data) {
      let content=JSON.parse(c)
      console.log("The weather in Dallas is... "+content.results.channel.item.condition.text);
    });
  });
  

```

As you can see, we are using the `http` module to make our request. The `get` function requires 2 params: the URL and a function to handle the response. 

#### Handle the response
The `response` object can emit 2 events: `data` and `close`. 

The `data` event is called when a chunk of information is sent from the server to the client. If the response is small, it could be all inside a single chunk, but don't rely on it. The best way is to concatenate the chunk inside a String variable (here `c`)

The `close` event is called when there are no more chunks left: know it's save to read the full response (stored in the `c` variable in this example).


### Make a POST/PUT/DELETE Requests
For other methods different than GET, we can use the `request` method. 


```javascript

let options={
	host: 'example.com', 	// change this
	port: 80,            	// (optional) port, defaults to 80
	path: '/example_path',	// path sent to server
	method: 'POST',       	// HTTP command sent to server (must be uppercase 'GET', 'POST', etc)
	headers: { key : value, key : value } // (optional) HTTP headers
}

let data=JSON.stringify({
	a:5 //data to send in POST request
})

require("http").request(options, function(res) {
    let c = "";
    res.on('data', function(data) {
      c += data;
    });
    res.on('close', function(data) {
      let content=JSON.parse(c)
      console.log("The server responded:", content)
    });
  }).end(data);
  
```

As you can see, the `request` method is pretty similar to `get` method, except you use a options object inside of the string url as first paramether and then you add a `.end(data)` to start the request with the POST/PUT/PATCH data to send to the server. 

N.B.: You can also call a GET URL with `request`, just use `method:"GET"` and `.end({})`. 


### HTTPS

Unfortunately as today (13 April 2017), the HTTPS is supported only on Espruino Pico and WiFi. 

