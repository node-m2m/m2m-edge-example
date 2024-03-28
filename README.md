
## Edge devices communicating through a private local area network 

<br>

In this example, we will setup endpoint devices communicating with each other through a local area network using tcp. 

The remote devices must be connected to the internet for authentication ensuring only authorized users can access the edge devices. You can also configure and develop your applications using a browser interface. 

![](assets/m2m-edge.svg)

## Edge Server 1 Setup

### 1. Create a project directory and install m2m.
```js
$ npm install m2m
```
### 2. Save the code below as device.js in your project directory.
```js
const m2m = require('m2m')

// simulated voltage data source
function voltageSource(){
  return 50 + Math.floor(Math.random() * 10)
}

let m2mServer = new m2m.Server(100)
let edge = new m2m.Edge()

m2m.connect()
.then(console.log) // success
.then(() => {
    // m2m server 1 - internet bound communication
  
    m2mServer.publish('m2m-voltage', (ws) => {
      let vs = voltageSource()
      ws.send({id:ws.id, topic:ws.topic, type:'voltage', value:vs})
    })
})
.then(() => {
    // edge tcp server - private LAN communication
  
    let port = 8125		// port must be open from your endpoint
    let host = '192.168.0.113' 	// use the actual ip of your endpoint
    
    edge.createServer(port, host, (server) => {
      console.log('edge server started :', host, port)
      
      server.publish('edge-voltage', (tcp) => { // using default 5 secs or 5000 ms polling interval
         let vs = voltageSource()
         tcp.send({port:port, topic:tcp.topic, type:'voltage', value:vs})
      })
  
      server.dataSource('data-source-1', (tcp) => {
         if(tcp.payload){
            tcp.send({port:port, topic:tcp.topic, payload:tcp.payload})
         }
      })
    })
})
.catch(console.log)
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
const m2m = require('m2m')

// simulated temperature data source
function tempSource(){
  return 20 + Math.floor(Math.random() * 4)
}

let m2mServer = new m2m.Server(200)
let edge = new m2m.Edge()

m2m.connect()
.then(console.log) // success
.then(() => {
    // m2m server 2 - internet bound communication

    m2mServer.publish('m2m-temperature', (ws) => {
      let ts = tempSource()
      ws.send({id:ws.id, topic:ws.topic, type:'temperature', value:ts})
    })
})
.then(() => {
    // edge tcp server - private LAN communication
   
    let port = 8125		// port must be open from your endpoint
    let host = '192.168.0.142' 	// use the actual ip of your endpoint 
    
    let server = edge.createServer(port, host)
    console.log('tcp server started :', host, port)
      
    server.publish('edge-temperature', (tcp) => {
      let ts = tempSource()
      tcp.polling = 10000;	// set polling interval to 10000 ms or 10 secs instead of the default 5000 ms
      tcp.send({port:port, topic:tcp.topic, type:'temperature', value:ts})
    });

    server.dataSource('current-temp', (tcp) => {
      let ts = tempSource()
      tcp.send({port:port, topic:tcp.topic, value:ts})
    })
})
.catch(console.log)

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
const m2m = require('m2m') 

let client = new m2m.Client()

let edge = new m2m.Edge()

m2m.connect()
.then(console.log) // success
.then(() => {
    // m2m client - access m2m servers through a public internet

    // m2m server 1 
    let server1 = client.access(100)
    
    server1.subscribe('m2m-voltage', (data) => {
      console.log('m2m device 100 voltage', data)
    })

    // m2m server 2 
    let server2 = client.access(200)
  
    server2.subscribe('m2m-temperature', (data) => {
      console.log('m2m device 200 temperature', data)
    })
})
.then(async () => {
    // edge tcp clients - access edge servers through a private local network
  
    // edge client 1 
    let ec1 = new edge.client(8125, '192.168.0.113')
    
    ec1.subscribe('edge-voltage', (data) => {
      console.log('edge server 1 edge-voltage', data)
    })

    let wd = await ec1.write('data-source-1', 'node-edge')
    console.log('edge server 1 data-source1', wd)
  
    // edge client 2
    let ec2 = new edge.client(8125, '192.168.0.142')
    
    ec2.subscribe('edge-temperature', (data) => {
      console.log('edge server 2 edge-temperature', data)
    })

    ec2.read('current-temp')
    .then(console.log)
})
.catch(console.log)
```
### 3. Start your application.
```js
$ node client.js
```

### 4. The expected output should be similar as shown below.
```js
$

edge server 1 edge-voltage {port:8125, topic:'edge-voltage', type:'voltage', value:16}
edge server 1 data-source-1 {port:8125, topic:'data-source-1', payload:'node-edge'}
m2m device 200 temperature {id:200, topic:'m2m-temperature', type:'temperature', value:25
m2m device 100 voltage {id:100, topic:'m2m-voltage', type:'voltage', value:4
edge server 2 edge-temperature {port:8125, topic:'edge-temperature', type:'temperature', value:25}
{port:8125, topic:'current-temp', value:25}

```


