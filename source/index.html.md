---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python

toc_footers:
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:

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

To communicate with Cyperus, an Open-Sound-Control client-server is required. That means your process must be capable of both sending messages and receiving the corresponding response message. In the examples below, "pyliblo" is the module used for communication using Python, but of course any language extended with an Open-Sound-Control interface may be used.

The 'Address' section contains a full example illustrating the send/receive flow in Python, the following sections only contain an example send and printed response.

## Send messages

> To send an OSC message, you must know the destination host and port. A non-existent message namespace is used as an example:

```python
import liblo
dest = liblo.Address('10.0.0.181', 97211)
liblo.send(dest, "/cyperus/message")
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


# Address

## Configure Send Address

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


> The above namespace returns the host and port to confirm:


```python
["10.0.0.126", "3001"]
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

## List Mains

> To get a list of Cyperus' current main inputs and outputs:

```python
liblo.send(dest, "/cyperus/list/main")
```

> The above namespace returns a new-line separated list of inputs and outputs like:

```python
['in:\n/mains{3ca74782-b379-446e-8a59-1f96323d4b89\n/mains{cddbb832-eea0-4aa7-a427-36eb6fdba529\n/mains{9ef2ead1-e7ab-4b00-8454-52dc1be82a15\n/mains{ba78868f-47b2-442f-b6e7-de6ed54efc0a\n/mains{fe0663ec-08af-4a01-822e-05cf4312d775\n/mains{0258a013-8d9e-4655-94ee-0ac973350c2d\n/mains{4ebcb35e-7ae5-4c97-bb31-4c02a72cd53b\n/mains{fb1418ac-a93d-4562-9aae-0b4e9d7c6f13\nout:\n/mains}e5417ebc-76a0-4b9b-9afa-e8decb9561a8\n/mains}8b0fa997-9143-405a-84f8-d55f79d47cfd\n/mains
}ac60fe34-b3be-4068-92eb-b508fc1fb3da\n/mains}f06b4ab4-1705-45d0-be41-c346822b3aee\n/mains}f5721f2a-0a32-4fe7-9500-b3afba48eaed\n/mains}dcf2ab1c-bd42-4860-89ba-a277f1da4262\n/mains}54c09d68-9d1e-4dd7-8c9e-480e559042ce\n/mains}114e0933-e8d8-4742-a7ef-f4193734f14e\n']
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

## List Bus Ports

> To list the ports of a bus in a given path:

```python
liblo.send(dest, "/cyperus/list/bus_port", "/a2a9fce1-0aeb-4c9f-bbd9-7a934690d0ed")
```

> Response is sent back containing a newline-separated list of a bus' ports. Each port is pipe-separated as `<bus port id>|<bus port name>`:

```python
['/a2a9fce1-0aeb-4c9f-bbd9-7a934690d0ed', 'in:\n6b005def-39a7-409f-97c3-b9bb21d91e8c|in\nout:\nbe9c4d40-7064-4147-a67d-b12f3b81befa|out\n']
```

Lists ports belonging to a bus in a given path.

### OSC Namespace

`/cyperus/list/bus_port s`

### ID Separator
`:`

### Message Arguments


Argument | TypeTag | Example Data
--------- | ------- | -----------
module-path | s | "/a2a9fce1-0aeb-4c9f-bbd9-7a934690d0ed"

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
module-path | s | "/a2a9fce1-0aeb-4c9f-bbd9-7a934690d0ed"
result | s | "in:\n6b005def-39a7-409f-97c3-b9bb21d91e8c|in\nout:\nbe9c4d40-7064-4147-a67d-b12f3b81befa|out\n"


## Add Root-Level Bus

> To add a new dsp bus to the root-level:

```python
liblo.send(dest, "/cyperus/add/bus", "/", "main0", "in", "out")
```

> Response is sent back containing original arguments and additionally `0` for success and `1` for fail:

```python
["/", "main0", "in", "out", 0]
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

## List First Root-Level Bus Peer

> To list the first root-level bus peer:

```python
liblo.send(dest, "/cyperus/list/bus", "/", 0)
```

> Response is sent back containing original arguments and additionally `0` for success and `1` for fail:

```python
['/', 1, 0, 'a264cd84-dad1-40d3-8e83-48b0f55434bd|main0|1|1\n']
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

## List Modules

> To list the ports of a module in a given path:

```python
liblo.send(dest, "/cyperus/list/module", "/a7788829-f087-4631-92fb-6f39d2e67ba9")
```


> Response is sent back containing a newline-separated list of modules:

```python
['aa628862-33f9-43e3-ab6b-61b36f6f503a\n95f76f45-be0e-4b4b-9d00-945da1ea7af8']
```


Lists modules belonging to a bus, in a given path.

### OSC Namespace

`/cyperus/list/module s`

### ID Separator
`/`

### Message Arguments


Argument | TypeTag | Example Data
--------- | ------- | -----------
bus-path | s | "/a7788829-f087-4631-92fb-6f39d2e67ba9"

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
result | s | "aa628862-33f9-43e3-ab6b-61b36f6f503a\n95f76f45-be0e-4b4b-9d00-945da1ea7af8"
    
## List Module Ports

> To list the ports of a module in a given path:

```python
liblo.send(dest, "/cyperus/list/module/port", "/a7788829-f087-4631-92fb-6f39d2e67ba9?aa628862-33f9-43e3-ab6b-61b36f6f503a")
```


> Response is sent back containing a newline-separated list of a module's ports:

```python
['/a7788829-f087-4631-92fb-6f39d2e67ba9?aa628862-33f9-43e3-ab6b-61b36f6f503a', 'in:\n3c841b45-d29d-44de-8abe-6a7c8a632964|in\nout:\n95f76f45-be0e-4b4b-9d00-945da1ea7af8|out\n']
```


Lists ports belonging to a module in a given path.

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
liblo.send(dest, "/cyperus/add/module/sine", "/a7788829-f087-4631-92fb-6f39d2e67ba9", 100, 1.0, 0.0)
```

> Response is sent back containing new module id and creation parameters::

```python
["/", "e5c39cc4-7932-4186-a1be-ecb78fd25234", 100, 1.0, 0.0]
```

Adds a new sine generator module to a given path.

### OSC Namespace

`/cyperus/add/module/sine sfff`

### ID Separator
`?`

### Message Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
bus path | s | "/a7788829-f087-4631-92fb-6f39d2e67ba9"
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

## Add Delay Module

> To add a new delay module to a given path:

```python
liblo.send(dest, "/cyperus/add/module/delay", "/a7788829-f087-4631-92fb-6f39d2e67ba9", 1.0, 1.0, 0.8)
```

> Response is sent back containing new module id and creation parameters:

```python
['e5c39cc4-7932-4186-a1be-ecb78fd25234', 1.0, 1.0, 0.8]
```

Adds a new delay module to a given path.

### OSC Namespace

`/cyperus/add/module/delay sfff`

### ID Separator
`?`

### Message Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
bus path | s | "/a7788829-f087-4631-92fb-6f39d2e67ba9"
amplitude | f | 1.0
time (sec) | f | 1.0
feedback | f | 0.8

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
module id | s | "e5c39cc4-7932-4186-a1be-ecb78fd25234"
amplitude | f | 1.0
time (sec) | f | 1.0
feedback | f | 0.8

## Add Envelope Follower Module

> To add a new envelope follower module to a given path:

```python
liblo.send(dest, "/cyperus/add/module/envelope_follower", "/a7788829-f087-4631-92fb-6f39d2e67ba9", 1.0, 1.0, 1.0)
```

> Response is sent back containing new module id and creation parameters:

```python
['e5c39cc4-7932-4186-a1be-ecb78fd25234', 1.0, 1.0, 1.0]
```

Adds a new envelope follower module to a given path.

### OSC Namespace

`/cyperus/add/module/envelope follower sfff`

### ID Separator
`?`

### Message Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
bus path | s | "/a7788829-f087-4631-92fb-6f39d2e67ba9"
attack (ms) | f | 1.0
decay (ms) | f | 1.0
scale | f | 1.0

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
module id | s | "e5c39cc4-7932-4186-a1be-ecb78fd25234"
attack (ms) | f | 1.0
decay (ms) | f | 1.0
scale | f | 1.0

# DSP Connections

## Add Connection

> To add a connection between dsp ports and/or dsp bus ports:

```python
liblo.send(dest, "/cyperus/add/connection", "/f38e9de7-d26b-4aba-b7e9-bc069e293d51:eefd199b-eb1b-41eb-8f9c-381f683a8b91", "/f38e9de7-d26b-4aba-b7e9-bc069e293d51?7e8ffe83-6afd-4ad2-bc33-c4ca9c3377d0<7bb05e05-e43f-4f5d-bee3-4400c71e1999")
```

> Response is sent back containing attempted connection ports and success result of `0`

```python
['/f38e9de7-d26b-4aba-b7e9-bc069e293d51:eefd199b-eb1b-41eb-8f9c-381f683a8b91', '/f38e9de7-d26b-4aba-b7e9-bc069e293d51?7e8ffe83-6afd-4ad2-bc33-c4ca9c3377d0<7bb05e05-e43f-4f5d-bee3-4400c71e1999', 0]
```

Adds a connection between dsp ports and/or dsp bus ports.

### OSC Namespace

`/cyperus/add/connection ss`

### Input Port ID Separator
`<`

### Output Port ID Separator
`>`

### Bus Port ID Separator
`:`  

### Message Arguments


Argument | TypeTag | Example Data
--------- | ------- | -----------
out-path | s | "/f38e9de7-d26b-4aba-b7e9-bc069e293d51:eefd199b-eb1b-41eb-8f9c-381f683a8b91"
in-path | s |  "/f38e9de7-d26b-4aba-b7e9-bc069e293d51?7e8ffe83-6afd-4ad2-bc33-c4ca9c3377d0<7bb05e05-e43f-4f5d-bee3-4400c71e1999"

### Response Arguments

Argument | TypeTag | Example Data
--------- | ------- | -----------
out-path | s | "/f38e9de7-d26b-4aba-b7e9-bc069e293d51:eefd199b-eb1b-41eb-8f9c-381f683a8b91"
in-path | s |  "/f38e9de7-d26b-4aba-b7e9-bc069e293d51?7e8ffe83-6afd-4ad2-bc33-c4ca9c3377d0<7bb05e05-e43f-4f5d-bee3-4400c71e1999"
success | i | 0
