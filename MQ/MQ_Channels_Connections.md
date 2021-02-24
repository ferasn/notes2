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
     - NORMAL: Nonpersistent messages on a channel are transferred within transactions.  No message is lost in transit.
     - FAST (Default value): Nonpersistent messages on a channel are not transferred within transactions.  unit of work, syncpoint, transaction.  Only if the batch sent are all non-persistent.  This why it is good practice to use seperate channels for persistent and non-persitent.

* The queue that holds the synchronization data for channels is SYSTEM.CHANNEL.SYNCQ

* Stopping a receiver channel: it is stuck waiting to a message from sender.  Two ways to get this message:
  1. The sender sends a message batch
  2. The sedner sends a heartbeat

* Sender channel "retry" vs receiver channel "message retry"

* How to simulator in-doubt sender channel? using three queues and the third queue with depth 1 and sending messages to the three queues in sequence and makding batch size three.  When the third message (the last in the batch) is sent by sender channel, it start the confirm flow and enters in-doubt.  The receiver upon receiving the thrid message will fail to put the message in thrid queue and enters "message" retry loop. 

* 

* Channel parameters that negotiated:
  1. heartbeat: the max value of both sender and receiver is taken.



* In-doubt channels and channel sequence numbers:


* a persistent message may be lost if GOT outside a syncpoint.  If the message is removed from queue but a crash happens before app receices the message
