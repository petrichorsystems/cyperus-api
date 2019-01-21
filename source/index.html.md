---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Cyperus API. 

This is an open-sound-control api that's used to generate musically-inclined dsp graphs in realtime.

For now we only support the UC Berkeley CNMAT's open-sound-control protocol, if you're unfamiliar please see [opensoundcontrol.org](http://opensoundcontrol.org/introduction-osc).

<aside class="notice">
Each unique object in the Cyperus API posseses a Version 4 UUID separated by a unique token for that type.
</aside>
# Client-Server

To communicate with Cyperus, an Open-Sound-Control client-server is required. That means your process must be capable of both sending messages and receiving the corresponding response message. In the examples below, "pyliblo" is the module used for communication using Python and "osc-min" with JavaScript, but of course any language extended with an Open-Sound-Control interface may be used.

The 'Address' section contains a full example illustrating the send/receive flow in both Python and JavaScript, the following sections only contain an example send and printed response.

## Send messages

> To send an OSC message, you must know the destination host and port. A non-existent message namespace is used as an example:

```python
import liblo
dest = liblo.Address('10.0.0.181', 97211)
liblo.send(dest, "/cyperus/message")
```

```javascript
var osc = require('osc-min'),
    dgram = require('dgram'),
    remote;

// listen for OSC messages and print them to the console
var udp = dgram.createSocket('udp4', function() {});

// build message with a few different OSC args
var msg_address = osc.toBuffer({
    oscType: 'message',
    address: '/cyperus/message',
    args: []
});
udp.send(msg_address, 0, msg_address.length, 3000, '10.0.0.181');

```

## Receive messages

> To receive an OSC message, you must know which port to listen on:

```python
import liblo
from liblo import *
import queue, sys, time

responses = queue.Queue()

class OscServer(ServerThread):
    def __init__(self):
        ServerThread.__init__(self, 97217)
        
    @make_method('/cyperus/message')
    def osc_message_handler(self, path, args):
        s = args
        responses.put(s)
        print("received '/cyperus/message'")
        
    @make_method(None, None)
    def fallback(self, path, args):
        print("fallback, received '{}'".format(path))

if __name__ == '__main__':
    #incoming server
    server = OscServer()

    server.start()

    input("press enter to quit...\n")
```

```javascript
var osc = require('osc-min'),
    dgram = require('dgram'),
    remote;

// listen for OSC messages and print them to the console
var udp = dgram.createSocket('udp4', function(msg, rinfo) {

  // save the remote address
  remote = rinfo.address;

  try {
    console.log(osc.fromBuffer(msg));
  } catch (err) {
    console.log('Could not decode OSC message');
  }

});

udp.bind(3001);
console.log('Listening for OSC messages on port 3001');

```

# Address

## Configure send address

> To change the send address of the running instance of Cyperus:

```python

import liblo
from liblo import *
import queue, sys, time

responses = queue.Queue()

class OscServer(ServerThread):
    def __init__(self):
        ServerThread.__init__(self, 3001)
        
    @make_method('/cyperus/address', 'ss')
    def osc_address_handler(self, path, args):
        s = args
        responses.put(s)
        print("received '/cyperus/address'")

    @make_method(None, None)
    def fallback(self, path, args):
        print("fallback, received '{}'".format(path))
        
def test_address_msg(dest):
    liblo.send(dest, "/cyperus/address", "10.0.0.126", "3001")
    response = responses.get()
    print('/cyperus/address: ', response)

if __name__ == '__main__':
    #outgoing connection
    dest = liblo.Address('10.0.0.181', 3000)

    #incoming server
    server = OscServer()

    server.start()

    test_address_msg(dest)

    input("press enter to quit...\n")
```

```javascript
var osc = require('osc-min'),
    dgram = require('dgram'),
    remote;

// listen for OSC messages and print them to the console
var udp = dgram.createSocket('udp4', function(msg, rinfo) {

  // save the remote address
  remote = rinfo.address;

  try {
    console.log(osc.fromBuffer(msg));
  } catch (err) {
    console.log('Could not decode OSC message');
  }

});

function send() {
  // build message with a few different OSC args

    var msg_address = osc.toBuffer({
	oscType: 'message',
	address: '/cyperus/address',
	args: [{
	    type: 'string',
	    value: '10.0.0.126'
	},
	{
	    type: 'string',
	    value: '3001'
	}]
    });
    
    udp.send(msg_address, 0, msg_address.length, 3000, '10.0.0.181');
}

udp.bind(3001);

send.call()

console.log('Listening for OSC messages on port 3001');
```

> The above namespace returns the host and port to confirm:


```python
["10.0.0.4", "97217"]
```

```javascript
{ address: '/cyperus/address',
  args: 
   [ { type: 'string', value: '10.0.0.126' },
     { type: 'string', value: '3001' } ],
  oscType: 'message' }
```

This namespace configures the send host and port.

### OSC Namespace

`/cyperus/address ss`

### Message Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
host | s | "10.0.0.126"
port | s | "3001"

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
host | s | "10.0.0.126"
port | s | "3001"

# Mains

## List mains

> To get a list of Cyperus' current main inputs and outputs:

```python
liblo.send(dest, "/cyperus/list/main")
```

```javascript
    var msg_address = osc.toBuffer({
        oscType: 'message',
        address: '/cyperus/list/main',
        args: []
    });

    udp.send(msg_address, 0, msg_address.length, 3000, '10.0.0.181');
```

> The above namespace returns a new-line separated list of inputs and outputs like:

```python
['in:\n/mains{3ca74782-b379-446e-8a59-1f96323d4b89\n/mains{cddbb832-eea0-4aa7-a427-36eb6fdba529\n/mains{9ef2ead1-e7ab-4b00-8454-52dc1be82a15\n/mains{ba78868f-47b2-442f-b6e7-de6ed54efc0a\n/mains{fe0663ec-08af-4a01-822e-05cf4312d775\n/mains{0258a013-8d9e-4655-94ee-0ac973350c2d\n/mains{4ebcb35e-7ae5-4c97-bb31-4c02a72cd53b\n/mains{fb1418ac-a93d-4562-9aae-0b4e9d7c6f13\nout:\n/mains}e5417ebc-76a0-4b9b-9afa-e8decb9561a8\n/mains}8b0fa997-9143-405a-84f8-d55f79d47cfd\n/mains
}ac60fe34-b3be-4068-92eb-b508fc1fb3da\n/mains}f06b4ab4-1705-45d0-be41-c346822b3aee\n/mains}f5721f2a-0a32-4fe7-9500-b3afba48eaed\n/mains}dcf2ab1c-bd42-4860-89ba-a277f1da4262\n/mains}54c09d68-9d1e-4dd7-8c9e-480e559042ce\n/mains}114e0933-e8d8-4742-a7ef-f4193734f14e\n']
```

```javascript
{ address: '/cyperus/list/main',
  args: 
   [ { type: 'string',
       value: 'in:\n/mains{74752fe4-18ce-42e1-a174-21decacd6b58\n/mains{e072b0e0-bd93-40b4-88dc-3892ec212ea7\n/mains{4e40c809-3a92-48ec-a69b-87e986143c49\n/mains{619f0dcf-a02f-4895-9f30-bc277662bfd7\n/mains{c9d46d73-9485-4d63-a52d-3f0b7eb140e8\n/mains{5ecb0227-323d-452c-b275-1f68e7c79dec\n/mains{2a136665-2e77-4832-aa7d-89ab68e48306\n/mains{9749dd29-8cb5-46a9-94d4-26570f84d99c\nout:\n/mains}671f726c-4f92-4d92-bc99-3e0ac28a29da\n/mains}03a136e4-009b-4475-bff7-1cb73091119a\n/mains}740efebc-1ba0-4fcb-8228-8c02adfa90c2\n/mains}6f15f834-11f9-4dee-8a16-c78a22402245\n/mains}70983545-5cc5-4d22-9e38-fd7225bc91cb\n/mains}20cead11-c4bf-44fc-914b-9329bcd44698\n/mains}1087acbc-ea2c-4ea7-b129-22bb265e2971\n/mains}0f2e48eb-d8cb-4191-baa3-3582f98dafa3\n' } ],
  oscType: 'message' }
```

This namespace lists all main inputs/outputs in the form of a newline-separated string.

### OSC Namespace

`/cyperus/list/main`

### Input ID Separator
`{`

### Output ID Separator
`}`


### Message Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
n/a | n/a | n/a

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
mains list | s | "in:\n/mains{3ca74782-b379-446e-8a59-1f96323d4b89\nout:\n/mains}e5417ebc-76a0-4b9b-9afa-e8decb9561a8\n"
# DSP Buses

## Add root-level bus

> To add a new dsp bus to the root-level:

```python
liblo.send(dest, "/cyperus/add/bus", "/", "main0", "in", "out")
```

```javascript
    var msg_address = osc.toBuffer({
        oscType: 'message',
        address: '/cyperus/add/bus',
        args: [{
            type: 'string',
            value: '/'
        },
        {
            type: 'string',
            value: 'main0'
        },
        {
            type: 'string',
            value: 'in'
        },
        {
            type: 'string',
            value: 'out'
        }]

    });

    udp.send(msg_address, 0, msg_address.length, 3000, '10.0.0.181');
```

> Response is sent back containing original arguments and additionally `0` for success and `1` for fail:

```python
["/", "main0", "in", "out", 0]
```

```javascript
{ address: '/cyperus/add/bus',
  args:
   [ { type: 'string', value: '/' },
     { type: 'string', value: 'main0' },
     { type: 'string', value: 'in' },
     { type: 'string', value: 'out' },
     { type: 'integer', value: 0 } ],
  oscType: 'message' }

```

Adds a new root-level dsp bus given a path.

### OSC Namespace

`/cyperus/add/bus ssss`

### ID Separator
`/`

### Message Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
path | s | "/"
name | s | "main0"
input bus-port names | s | "in"
output bus-port names | s | "out"

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
path | s | "/"
name | s | "main0"
input bus-port names | s | "in"
output bus-port names | s | "out"
success | i | 0

## List first root-level bus peer

> To list the first root-level bus peer:

```python
liblo.send(dest, "/cyperus/list/bus", "/", 0)
```

```javascript
    var msg_address = osc.toBuffer({
        oscType: 'message',
        address: '/cyperus/list/bus',
        args: [{
            type: 'string',
            value: '/'
        },
        {
            type: 'integer',
            value: 0
        }]

    });

    udp.send(msg_address, 0, msg_address.length, 3000, '10.0.0.181');
```

> Response is sent back containing original arguments and additionally `0` for success and `1` for fail:

```python
['/', 1, 0, 'a264cd84-dad1-40d3-8e83-48b0f55434bd|main0|1|1\n']
```

```javascript
{ address: '/cyperus/list/bus',
  args:
   [ { type: 'string', value: '/' },
     { type: 'integer', value: 0 },
     { type: 'integer', value: 0 },
     { type: 'string',
       value: '910ba248-f715-41d1-a620-d458fc2a649e|main0|1|1\n' } ],
  oscType: 'message' }

```

Lists the first root-level bus peer

### OSC Namespace

`/cyperus/list/bus si`

### ID Separator
`/`

### Message Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
path | s | "/"
list-type | i | 0

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
path | s | "/"
list-type | i | 0
success | i | 0
result | s | a264cd84-dad1-40d3-8e83-48b0f55434bd|main0|1|1\n

# DSP Modules

## List Module Ports

> To list the ports of a module in a given path:

```python
liblo.send(dest, "/cyperus/list/module/port", "/a7788829-f087-4631-92fb-6f39d2e67ba9?aa628862-33f9-43e3-ab6b-61b36f6f503a")
```

```javascript
    var msg_address = osc.toBuffer({
        oscType: 'message',
        address: '/cyperus/add/module/sine',
        args: [{
            type: 'string',
            value: '/'
        },
        {
            type: 'string',
            value: 'main0'
        },
        {
            type: 'string',
            value: 'in'
        },
        {
            type: 'string',
            value: 'out'
        }]

    });

    udp.send(msg_address, 0, msg_address.length, 3000, '10.0.0.181');
```

> Response is sent back containing a newline-separated list of a module's ports:

```python
['/a7788829-f087-4631-92fb-6f39d2e67ba9?aa628862-33f9-43e3-ab6b-61b36f6f503a', 'in:\n3c841b45-d29d-44de-8abe-6a7c8a632964|in\nout:\n95f76f45-be0e-4b4b-9d00-945da1ea7af8|out\n']
```

```javascript
{ address: '/cyperus/add/bus',
  args:
   [ { type: 'string', value: '/' },
     { type: 'string', value: 'main0' },
     { type: 'string', value: 'in' },
     { type: 'string', value: 'out' },
     { type: 'integer', value: 0 } ],
  oscType: 'message' }

```

Adds a new sine generator module to a given path.

### OSC Namespace

`/cyperus/list/module/port s`

### ID Separator
`?`

### Message Arguments


Argument | TypeTag | Example Data
--------- | ------- | -----------
module-path | s | "/a7788829-f087-4631-92fb-6f39d2e67ba9?aa628862-33f9-43e3-ab6b-61b36f6f503a"

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
module-path | s | "/a7788829-f087-4631-92fb-6f39d2e67ba9?aa628862-33f9-43e3-ab6b-61b36f6f503a"
result | s | "in:\n3c841b45-d29d-44de-8abe-6a7c8a632964|in\nout:\n95f76f45-be0e-4b4b-9d00-945da1ea7af8|out\n"

## Add Sine Module

> To add a new sine module to a given path:

```python
liblo.send(dest, "/cyperus/add/module/sine", "/", "main0", "in", "out")
```

```javascript
    var msg_address = osc.toBuffer({
        oscType: 'message',
        address: '/cyperus/add/module/sine',
        args: [{
            type: 'string',
            value: '/'
        },
        {
            type: 'string',
            value: 'main0'
        },
        {
            type: 'string',
            value: 'in'
        },
        {
            type: 'string',
            value: 'out'
        }]

    });

    udp.send(msg_address, 0, msg_address.length, 3000, '10.0.0.181');
```

> Response is sent back containing original arguments and additionally `0` for success and `1` for fail:

```python
["/", "main0", "in", "out", 0]
```

```javascript
{ address: '/cyperus/add/bus',
  args:
   [ { type: 'string', value: '/' },
     { type: 'string', value: 'main0' },
     { type: 'string', value: 'in' },
     { type: 'string', value: 'out' },
     { type: 'integer', value: 0 } ],
  oscType: 'message' }

```

Adds a new sine generator module to a given path.

### OSC Namespace

`/cyperus/add/module/sine sfff`

### ID Separator
`?`

### Message Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
bus path | s | "/"
frequency (hertz) | f | 100.0
amplitude | f | 1.0
phase | f | 0.0

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
module id | s | "e5c39cc4-7932-4186-a1be-ecb78fd25234"
frequency (hertz) | f | 100.0
amplitude | f | 1.0
phase | f | 0.0
