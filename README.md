# OpenIO

OpenIO is an opensource, openhardware I/O control initiative.
OpenIO focusses on creating an abstract, cross platform way of controlling devices.

Currently openIO is only a concept, but I'm thinking of creating a NodeJS implementation of it.

# Architecture

OpenIO uses a client-server model, to ensure QoS and simplify management.

# How does it work?

## Server
There is one server in a network. Any client can either be manually attached or by auto-discovery.
On the server, one can set triggers for different events. These triggers lead to actions if all requirements are met.

### Connection handler
The server implements different "transmission layers" (as per OSI) within a connection handler. Basically this means that the server can start up an UDP server, HTTP server, TCP/IP server, MQTT server, websocket server, connect to an USB RF device, connect to an IP or USB RS485/RS232 bus/device or basically anything. The server should, if possible automatically invoke these connection layers from a folder. But this may not be possible in some types of programming languages. (Usually compiled languages)

### Message handler
The message handler will identify from which device the message comes and translates it into an event.
It will also built up a list of devices and possible events/triggers and actions, if these can't be automatically detected, they will be read from a configuration file or database.

### I to the O
A user can link events/triggers to certain actions. The user can also specify a guard/requirement. When an event is triggered AND the requirements are met, the action will be executed. If the requirements are not met an "exception action" can be executed.
Multiple actions can be added to one event, to give the idea of grouping.

For example:

```
Trigger - Smart switch ON
Requirements - Time of day is (8:00-19:00) not night 
Action - Turn all lights on in the room 100% brigthness
Exception - Turn only side light on for 50%
```

The "usecase" above is set up according to a real life situation.
I would like a system where I can control the lighting in my bedroom with a simple press of the button.
But I don't want to get blinded by my spots if I have to get up in the middle of the night.

I could also specify the lights to go on whenever my android phones alarm goes of (if I hook to the alarm app to send a HTTP message to my server). Bonus, if I use the local address of my server, my lights won't go on when I'm not at home (not connected to wifi).

## Client

A client can be any device and doesn't need any specific protocol or connection. Since the server can be extended to accept any form of communication and can be configured to understand messages, it allows for any device to be connected. Making it possible to use any existing hardware without having to reprogram it. Optionally, clients have autodiscovery, but this is required since not all devices have such a thing implemented.

A client can generate an event and/or execute an action. It is possible to have a client that can do both, or only one of these.

### Event
A client can generate events. For example a press of a button or a change in temperature. The server will handle these events.

### Action
A client can execute actions. For example setting a relay, controlling an LED strip, send an IR command. The server will send these actions.

### Autoconf

In the future, it may be possible to let devices configurate themselves automatically. A device can send a message where it specifies which triggers it will give (and how) and what actions it can execute.

For example:

```
deviceName: "Smart switch",
connection: [{type:"HTTP",ip:"192.168.1.12"},{type:"BROADCAST"}]
triggers: [{name:"Button Pressed", message:"input high 1"}, {name:"Button Released",message:"input low 1"}]
actions:  [{name:"Set indication LED on", message:"output=high&id=1", connection:{type:"HTTP",method:"GET"}}, {name:"Set indication LED off", message:"output=low&id=1", connection:{type:"HTTP",method:"GET"}} ]
```

This object (probably in JSON, but we could make an abstract parser that can also handle xml) will be sent to the server by the client. In this way the server knows which transmission layer to use and what messages to send, to get the desired behaviour.
