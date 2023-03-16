
## Setup edge devices communicating through a private local area network 

<br>

In this example, we will setup endpoint devices communicating to each other through a local area network using tcp. 

The devices are also accessible from the cloud so you can configure and develop your applications using a browser interface. 

![](assets/m2m-edge.svg)

## Edge Server 1 Setup

### 1. Create a project directory and install m2m.
```js
$ npm install m2m
```
### 2. Save the code below as device.js in your project directory.
```js
const m2m = require('m2m');

// simulated voltage sensor data source
function voltageSource(){
  return 50 + Math.floor(Math.random() * 10);
}

/***
 * edge tcp server (communication through a private local network)
 */
    
let port = 8125;		// port must be open from your endpoint
let host = '192.168.0.113'; 	// use the actual ip of your endpoint

m2m.edge.createServer(port, host, (server) => {
  console.log('edge server started :', host, port);
  
  server.publish('edge-voltage', (data) => { // using default 5 secs polling interval
     let vs = voltageSource();
     tcp.send(vs);
  });
  
  server.dataSource('data-source-1', (data) => {
     if(data.payload){
        data.send(data.payload);
     }
  });
  
});

/***
 * m2m device (communication through a public internet)
 */
let device = new m2m.Device(100); // using deviceId of 100

device.connect(() => {
    device.publish('m2m-voltage', (data) => {
      let vs = voltageSource();
      data.send(vs);
    });
});

```
### 3. Start your application.
```js
$ node device.js
```

## Edge Server 2 Setup

### 1. Create a project directory and install m2m.
```js
$ npm install m2m
```
### 2. Save the code below as device.js in your project directory.
```js
const m2m = require('m2m');

// simulated temperature sensor data source
function tempSource(){
  return 20 + Math.floor(Math.random() * 4);
}

/***
 * edge tcp server (communication through a private local network)
 */
    
let port = 8125;		// port must be open from your endpoint
let host = '192.168.0.142'; 	// use the actual ip of your endpoint

m2m.edge.createServer(port, host, (server) => {
  console.log('tcp server started :', host, port);
  
  server.publish('edge-temperature', (data) => {
     let ts = tempSource();
     data.polling = 9000; 	// set polling interval to check data source for any changes
     data.send(ts);
  });
  
  server.dataSource('current-temp', (data) => {
     data.send(tempSource()); 
  })
  
});

/***
 * m2m device (communication through a public internet)
 */
let device = new m2m.Device(200);

device.connect(() => {
    device.publish('m2m-temperature', (data) => {
      let ts = tempSource();
      data.send(ts);
    });
});

```
### 3. Start your application.
```js
$ node device.js
```

## Edge Client Setup

### 1. Create a project directory and install m2m.
```js
$ npm install m2m
```
### 2. Save the code below as client.js in your project directory.
```js
const m2m = require('m2m'); 

let client = new m2m.Client();

client.connect(() => {
    
    /********************************************************
     * m2m client (communication through a public internet)
     ********************************************************/
     
    // method 1
    let device1 = client.accessDevice(100)    // subscribe from m2m device 100
    
    device1.subscribe('m2m-voltage', (data) => {
      console.log('m2m device 100 voltage', data);
    });
    
    let device2 = client.accessDevice(200)    // subscribe from m2m device 200

    device2.subscribe('m2m-temperature', (data) => {
      console.log('m2m device 200 temperature', data);
    });
    
    // or 
    
    // method 2
    client.subscribe({id:100, channel:'m2m-voltage'}, (data) => {
      console.log('m2m device 100 voltage', data);
    });
    
    client.subscribe({id:200, channel:'m2m-temperature'}, (data) => {
      console.log('m2m device 200 temperature', data);
    });

    /********************************************************************
     * edge tcp clients (communication through a private local network)
     ********************************************************************/
     
    // edge client 1 
    let ec1 = new m2m.edge.client(8125, '192.168.0.113');
    
    ec1.subscribe('edge-voltage', (data) => {
      console.log('edge server 1 voltage', data.toString());
    });
    
    ec2.write('data-source-1', 'node-edge', (data) => {
      console.log('edge server 1 data-source-1 value', data.toString());
    });

    // edge client 2
    let ec2 = new m2m.edge.connect(8125, '192.168.0.142');
    
    ec2.subscribe('edge-temperature', (data) => {
      console.log('edge server 2 temperature', data.toString());
    });
    
    ec2.read('current-temp', (data) => {
      console.log('edge server 2 current temperature', data.toString());
    });
    
});

```
### 3. Start your application.
```js
$ node client.js
```

### 4. The expected output should be similar as shown below.
```js
$

edge server 1 voltage 16
edge server 1 data-source-1 value node-edge
m2m device 200 temperature 25
m2m device 100 voltage 9
edge server 2 temperature 25
edge server 2 current temperature 25

```


