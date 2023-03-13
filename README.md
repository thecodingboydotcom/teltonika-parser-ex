# README

Introduced a condition to handle ioElements 257 (Crash trace data) and sample code to convert hex to real axis value (mG)

### Source

Forked from: https://www.npmjs.com/package/teltonika-parser-ex

### Installation

Run console command

`npm i teltonika-parser-ex`

Note: Please copy the codec8ex.js file and replace it with the one located in the node_modules\teltonika-parser-ex\codecs directory

### Usage example

```const net = require('net');
 const Parser = require('teltonika-parser-ex');
 const binutils = require('binutils64');


function groupIntoArray(str) {
    const arr = [];
    for (let i = 0; i < str.length; i += 4) {
        arr.push(str.slice(i, i + 4));
    }
    return arr;
}

 let server = net.createServer((c) => {
     console.log("client connected");
     c.on('end', () => {
         console.log("client disconnected");
     });

     c.on('data', (data) => {

         let buffer = data;
         let parser = new Parser(buffer);
         if(parser.isImei){
             c.write(Buffer.alloc(1, 1));
         }else {
             let avl = parser.getAvl();

             let writer = new binutils.BinaryWriter();
             writer.WriteInt32(avl.number_of_data);

             let response = writer.ByteBuffer;
             c.write(response);
             
             
             //sample code to extract ioElement 257
             if (parser._avlObj.records != null) {
                 parser._avlObj.records.forEach(element => {

                     if (element.ioElements) {
                         element.ioElements.forEach(function (ioelement) {
                             if (ioelement.id == "257") {
                                 var _array_mG = [];
                                 
                                 const myArray = groupIntoArray(ioelement.value.toString('hex'));

                                 for (const hexValue of myArray) {
                                     // Convert hex to signed decimal
                                     const decimalValue = parseInt(hexValue, 16);
                                     const signedDecimalValue = decimalValue > 32767 ? decimalValue - 65536 : decimalValue;

                                     // Scale the decimal value to get the real axis value in mG
                                     const sensitivity = 1; // LSB/mG
                                     const axisValue = signedDecimalValue * sensitivity;
                                     _array_mG.push(axisValue);
                                 }
                                 ioelement.value = JSON.stringify(_array_mG);
                             }
                         });
                     };
                 });
             }             
             
         }
     });
 });

 server.listen(5000, () => {
     console.log("Server started");
 });
```
