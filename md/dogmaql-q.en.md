# DogmaQL/Q (queue)

*Reading time: 5min*

**DogmaQL/Q** is the **DogmaQL** spec for messaging services based on queue.

A **messaging service** is a communication system for exchanging messages between two o more software components.

Its main uses are the following:

- Implement chats.
- Real-time notification as, for example, events, alerts, news, changes of price, etc.
  Once the notification received, the client can perform an action as, for example, e-mail sends or data views.

The main features of a messaging service are:

- Component decouple.
  The message emitter aren't connected to the receivers.
  Both are connected to a third party: the proxy or broker.
  So well, the extremes aren't connected directly, maintaining decoupled.
- System scalability when the load ups or downs.

## Messaging models

Usually, the services are implemented under one of two **messaging models**, that is, a manner to conceive the system.
It defines the components to appear in the system, how to connect them and how to exchange the messages.
These are the point-to-point model and the pub/sub model.

Under the **point-to-point model**, the emitters send the messages to the broker, which will transmit to only one receiver.
Whereas when the **pub/sub model** used, the broker transmits the message to all the receivers.

A **channel**, **topic** or **queue** is a container to publish messages.
It's where the emitters publish their messages and from where the receivers read it.
The broker is the responsible component of administrating them.
Once a message has been transmitted to the receiver(s), this is removed automatically by the broker.

Not needed be created.
When a client is subscribed to the channel or when a message is published, the broker creates it.
And it is removed when no subscriber exists and no message is published pending to transmit.

## insert statement

In **DogmaQL/Q**, the `insert` statement is used for publishing one or multiple messages in a channel.

Example:

```
insert in queue concerts(
  {
    artist = "Simple Minds"
    city = "Valencia"
    date = "Jun 29, 2018"
  }
  {
    artist = "Echo and the Bunnymen"
    city = "London"
    date = "Jun 1, 2018"
  }
)
```

Have to indicate the `queue` keyword after `in`/`into`.
The channel or queue is `concerts`.
The messages to publish are the given data objects.

## set statement

Not supported in **DogmaQL/Q**.

# delete statement

Not supported in **DogmaQL/Q**.

# update statement

Not supported in **DogmaQL/Q**.


# select statement

Not supported in **DogmaQL/Q**.
