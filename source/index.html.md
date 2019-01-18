---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - c
  - python

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

# Mains

## List Mains

> To get a list of Cyperus' current main inputs and outputs, use this namespace:

```c
#include <lo/lo.h>
lo_address t = lo_address_new("127.0.0.1", "97211")
lo_send(t, "/cyperus/list/main", NULL);
```

```python
import liblo
dest = liblo.Address(97211)
liblo.send(dest, "/cyperus/list/main")
```

> The above namespace returns a new-line separated list of inputs and outputs like:

```c
in:
/mains{3ca74782-b379-446e-8a59-1f96323d4b89
/mains{cddbb832-eea0-4aa7-a427-36eb6fdba529
/mains{9ef2ead1-e7ab-4b00-8454-52dc1be82a15
/mains{ba78868f-47b2-442f-b6e7-de6ed54efc0a
/mains{fe0663ec-08af-4a01-822e-05cf4312d775
/mains{0258a013-8d9e-4655-94ee-0ac973350c2d
/mains{4ebcb35e-7ae5-4c97-bb31-4c02a72cd53b
/mains{fb1418ac-a93d-4562-9aae-0b4e9d7c6f13
out:
/mains}e5417ebc-76a0-4b9b-9afa-e8decb9561a8
/mains}8b0fa997-9143-405a-84f8-d55f79d47cfd
/mains}ac60fe34-b3be-4068-92eb-b508fc1fb3da
/mains}f06b4ab4-1705-45d0-be41-c346822b3aee
/mains}f5721f2a-0a32-4fe7-9500-b3afba48eaed
/mains}dcf2ab1c-bd42-4860-89ba-a277f1da4262
/mains}54c09d68-9d1e-4dd7-8c9e-480e559042ce
/mains}114e0933-e8d8-4742-a7ef-f4193734f14e

```

```python
['in:\n/mains{3ca74782-b379-446e-8a59-1f96323d4b89\n/mains{cddbb832-eea0-4aa7-a427-36eb6fdba529\n/mains{9ef2ead1-e7ab-4b00-8454-52dc1be82a15\n/mains{ba78868f-47b2-442f-b6e7-de6ed54efc0a\n/mains{fe0663ec-08af-4a01-822e-05cf4312d775\n/mains{0258a013-8d9e-4655-94ee-0ac973350c2d\n/mains{4ebcb35e-7ae5-4c97-bb31-4c02a72cd53b\n/mains{fb1418ac-a93d-4562-9aae-0b4e9d7c6f13\nout:\n/mains}e5417ebc-76a0-4b9b-9afa-e8decb9561a8\n/mains}8b0fa997-9143-405a-84f8-d55f79d47cfd\n/mains
}ac60fe34-b3be-4068-92eb-b508fc1fb3da\n/mains}f06b4ab4-1705-45d0-be41-c346822b3aee\n/mains}f5721f2a-0a32-4fe7-9500-b3afba48eaed\n/mains}dcf2ab1c-bd42-4860-89ba-a277f1da4262\n/mains}54c09d68-9d1e-4dd7-8c9e-480e559042ce\n/mains}114e0933-e8d8-4742-a7ef-f4193734f14e\n']
```

This namepsace lists all main inputs/outpus.

### OSC Namespace

/cyperus/list/main

### Input ID Separator
`{`

### Output ID Separator
`}`

# Buses

## List Root-Level Bus

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

