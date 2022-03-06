---
title: Streams
---
## Advanced Location

1. Callback Pattern

readmore about process.nextTick();

```js
function hideString2(str) {
  return str.replace(/[a-zA-Z]/g, "X");
}
function hideString(str, done) {
  done(str.replace(/[a-zA-Z]/g, "X"));
}
const hidden = hideString2("yousef");
hideString("meska", (hidden) => {
  console.log(hidden);
});
console.log(hidden);
console.log("end");

/* the code above is still act synchrously, becuase callbacks in nature are synchrounus, we need a way to make it async */
// we are going to use process.nextTick()
/* process.nextTick():
tell nodej to invoke the function we send to nextTick in the next loop of the Even loop, so it' goint to happen async 
*/

function hideString3(str, done) {
  process.nextTick(() => {
    done(str.replace(/[a-zA-Z]/g, "X"));
  });
}
hideString3("hello world", (hidden) => {
  console.log(hidden);
});

function delay(seconds, callback) {
  setTimeout(callback, seconds * 1000);
}
console.log("starting delays");
delay(2, () => {
  console.log("two seconds");
  delay(1, () => {
    console.log("three seconds ");
    delay(1, () => {
      console.log("four seconds");
    });
  });
});
console.log("end of delay");
```

2. Resolving Promises
  
  read more about Promises
  
  ```js
  const delay = (seconds) =>
    new Promise((resolve, reject) => {
      // setTimeout(resolve, seconds * 1000);
      setTimeout(() => {
        resolve("The delay has been ended");
      }, seconds * 1000);
    });
  
  /*delay(2).then((message) => {
    console.log(message);
  });*/
  
  delay(2)
    .then(console.log)
    .then(() => console.log("another mesage"))
    .then(() => console.log("another message"))
    .then(() => console.log("another message"))
    .then(() => console.log("another message"));
  
  delay(1)
    .then(console.log)
    .then(() => 42)
    .then((number) => console.log(`the number is ${number}`));
  console.log(`end first tick`);
  ```
  
3. Promisify function
  
  read more about promisify
  

```js
const { promisify } = require("util");
const delay = (seconds, callback) => {
  if (seconds > 3) {
    callback(new Error(`${seconds} seconds is too long1`));
  } else {
    setTimeout(() => {
      callback(null, `the ${seconds} second delay is over`);
    }, seconds * 1000);
  }
};
/*delay(2, (error, message) => {
  if (error) {
    console.log(error.message);
  } else {
    console.log(message);
  }
});*/

const promiseDelay = promisify(delay);
promiseDelay(2)
  .then(console.log)
  .catch((error) => console.log(`error: ${error.message}`));
```

## Streams

readmore about streams and buffers

and also read more about garbage collection on node [MarkSweep, Scavenge]

![](file://C:\Users\ncm\AppData\Roaming\marktext\images\2022-02-12-18-27-35-image.png)

on streams: we are not loading the entire file before sending it, we're sending it bit by bit as the server process it or recieve it

```js
const fs = require('fs');
const http = require('http');
const file = './my-vide.mp4';
http.createServer((req, res)=>{
    res.writeHead(200, {'Content-Type': 'video/mp4'});
    fs.createReadStream(file).pipe(res).on('error', console.error);
}).listen(3000, ()=>console.log('streams - http://localhost:3000'));
```

if you debug this application using node --trace_gc streams.js

you will notice there are no markSweep but only a schavenge which is not a big problem

using buffer

```js
const fs = require("fs");
const http = require("http");

const file = "./powder-day.mp4";

http
  .createServer((req, res) => {
    fs.readFile(file, (error, data) => {
      if (error) {
        console.log("hmmm: ", error);
      }
      res.writeHead(200, { "Content-Type": "video/mp4" });
      res.end(data);
    });
  })
  .listen(8000, () => {
    console.log(`buffer -http://localhost:3000`);
  });
```

streams use eventEmitter meaning they can raise events

### Readable streams

streams have two modes:

1. binary mode
  
2. object mode
  

```js
const {Readable}  = require('streams'); //Redable interface
const peaks = ["Tallac", "Rose", "Freel Peal", "Castle Peak"];
class StreamFromArray extends Redable{

    constructor(array){
    super({encoding: "UTF-8"});
    this.array = array || [];
    this.index = 0;
}
_read(){
    if(this.index <= this.array.length){
    const chunk = this.array[this.index];
this.push(chunk); // pushing it to the readable streams [first chucnk]
this.index +=1 
} else {
    this.push(null);
}
}
}

const peakStream = new StreamFromArray(peaks);
peakStream.on('data', (chunk)=>console.log(chunk));
```

super can accept {objectMode: true} which make the stream read any type of objects

```js
const chunk = {
    data: this.array[this.index],
    index: this.index
}
```

nodeJs comes with many types of readable streams ranging from

http request on the server

file system

zipping/unzipping

tcp sockets

and so on.

## Streams:

1. Readable Stream
  
2. Writable Stream
  
3. Piping Stream
  
4. Duplex Stream
  
5. Transform Stream