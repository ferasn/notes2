* MQ Connections:
  1. Two types:
    a) Server connection: connection made through IPC binding.
    b) Client connection: connection made through MQ client.
    
* MCA binding

* Server channel sharing converstation

* Channel States:
* 
* 
* A channel can be in two states:
  1- Inactive
  2- Current.  This state has substates: Stopped, Starting, Retrying, Active.  So a CURRENT channel is ACTIVE unless it is in RETRYING, STOPPED, or STARTING state.
    The seven possible states of an active channel (INITIALIZING, BINDING, SWITCHING, REQUESTING, RUNNING, PAUSED, or STOPPING).  When a channel is active, it is consuming resource and a process or thread is running.
    ... MAXchannels MAXActiveChannels....


* Channel properties: 
  1. Nonpersistent message speed (NPMSPEED): This property applies to all channel types (senders and receivers) that trasmit messages between queue managers. It is not applicable to Server-connection channels.
     - NORMAL: Nonpersistent messages on a channel are transferred within transactions. Message delivery between queue managers is acknowledged. No message is lost in transit.
     - FAST (Default value): Nonpersistent messages on a channel are not transferred within transactions. No acknowledgement. The advantage is speed. The disadvatnage is messages might be lost if there is a transmission failure or if the channel stops when the messages are in transit.

* The queue that holds the synchronization data for channels is SYSTEM.CHANNEL.SYNCQ

* Stopping a receiver channel: it is stuck waiting to a message from sender.  Two ways to get this message:
  1. The sender sends a message batch
  2. The sedner sends a heartbeat

* In-doubt channels and channel sequence numbers:
