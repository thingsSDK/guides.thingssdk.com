# Using an HTTP API

TL;DR - [jump to the final code](#final-code).

In this chapter we'll cover the basics of making an HTTP request to an API.

For this example, we don't need any wiring, as we will use the REPL interactive console to display the data from the API. 

Let's go to write some code!


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

##The Code
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

### Create the web server
Ok, now we are connected to Internet, let's begin to write the real code!
To create a new server, we need to use the **http** library. 


```javascript
let handleRequest=((req,res)=>{
	console.log("Somebody is connected!");
})
require("http").createServer(handleRequest).listen(80);
console.log(`I'm ready on ${ wifi.getIP().ip }:80`);
```

This will permit the Espruino to listen to port 80 on his assigned IP address (we are using wifi.getIP() to know the IP address).

### Build a response
Right now we don't serve a response, so the client will try to load the website forever. 
The handleRequest function has 2 object params pretty similar to any node.js server, **Request** and **Response**. **Request** contains the URL, the method used, and the paramethers sent to it, while the **Response** permits to serve data to the client. 
For example, if we want to return a simple HTML page, we should do something like this:

```javascript
const handleRequest=(req, res) => {
  res.writeHead(200, {'Content-Type': 'text/html'});
  res.end(`<html>
  		<head>Test Page</head>
  		<body>
  			<h1>Hello World!</h1>
  		</body>	
  	</html>`);
}
```

This will return "Hello World!".

### Handle different methods
Let's create a simple form that will send the name of the user to another page. We need to differentiate between the GET request (where we will display the form) and the POST request (where we will get the user name and show it on a different page.

```javascript
const handleRequest=(req, res) => {
  res.writeHead(200, {'Content-Type': 'text/html'});
  if(req.method=="GET"){
	  res.end(`<html>
	  	 <head>Test Page</head>
	  	 <body>
	  	   <h1>Write your name here:</h1>
	  		<form method="POST" action="/">
		  	  <input type="text" name="name_user">
	  	     <input type="submit" value="Send">
	  	   </form>
	  	 </body>	
	  </html>`); 	
  }else{
	  let params=parseRequestData(req.read());
  	  let name=obj.name_user;
   	  res.end(`<html>
  		<head>Test Page</head>
  		<body>
  		  <h1>Hello, ${name}</h1>
  		</body>	
	  </html>`); 	
  }
}

//function to return an object with all request paramethers 
const parseRequestData =(str)=>{
  return str.split("&").reduce(function(prev, curr, i, arr) {
    var p = curr.split("=");
    prev[decodeURIComponent(p[0])] = decodeURIComponent(p[1]);
    return prev;
  }, {});
}

```


### Keep the HTML/CSS/JS slim!
Remember: you have a limited amount of memory on Espruino. Everytime you add new fancy CSS attributes or lots of JS functions, you are using memory and this can drive to a **Out Of Memory!** error.
Be sure to put some **print(process.memory());** inside your code to monitor the amount of free memory still available.


## Final Code
```javascript
const wifi = require('Wifi')
const WIFI_NAME= "...";
const WIFI_PASSWORD= "...";

function main(){
  // wifi.connect(ssid, options object, callback)
  wifi.connect(WIFI_NAME, { password: WIFI_PASSWORD }, error => {
    if (error){
     console.error(error)
    }else{
  	  console.log(`Connected to: ${ wifi.getIP().ip }`)
  	  require("http").createServer(handleRequest).listen(80);
    }
  })
}

const handleRequest= (req, res) =>{
  res.writeHead(200, {'Content-Type': 'text/html'});
  if(req.method=="GET"){
	  res.end(`<html>
	  	 <head>Test Page</head>
	  	 <body>
	  	   <h1>Write your name here:</h1>
	  		<form method="POST" action="/">
		  	  <input type="text" name="name_user">
	  	     <input type="submit" value="Send">
	  	   </form>
	  	 </body>	
	  </html>`); 	
  }else{
	  let params=parseRequestData(req.read());
  	  let name=obj.name_user;
   	  res.end(`<html>
  		<head>Test Page</head>
  		<body>
  		  <h1>Hello, ${name}</h1>
  		</body>	
	  </html>`); 	
  }
}

//function to return an object with all request paramethers 
const parseRequestData = (str) =>{
  return str.split("&").reduce(function(prev, curr, i, arr) {
    var p = curr.split("=");
    prev[decodeURIComponent(p[0])] = decodeURIComponent(p[1]);
    return prev;
  }, {});
}


```

