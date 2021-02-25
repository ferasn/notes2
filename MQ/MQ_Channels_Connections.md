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

* Stopping a distributed QM sender channel to "INACTIVE" state does not inhibit the transmission queue and the messages in transmission queue are browsable.
* Stopping a distributed QM sender channel to "STOPPED" (disabled) inihibts "GET" of transmission queue and the queue is not browsable.
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


--------
message duplication case: if you resolve channel rollback and reset channel sequence number

NPMSPEED: FAST sender channel gets non-persistent messages outside of a transaction (syncpoint). In case, the batch contains persistent and non-persistent messages, only persistent message are GET in a syncpoint.   If the channel becomes in-doubt and resolved to back-out, the transaction of the batch is backed-out and persistent messages are available again in the queue.  Non-persistent message outside of syncpoint are destructively read by channel MCA and are permanently lost if the remote MCA failed to write them to their destination queues.

if a batch contains only non-persistent messages and NPMSPEED is FAST, the sender channel never becomes in-doubt on this batch because there is no unit-of-work started for the batch. Also, the sequence number is updated to the latest one regardless of the status of the batch at the receiver channel.
----------

Test two scenarios with Receiver failing to put message to detination queue:
1- message batch is GET in a transaction (either the messages are persistent or NPMSPEED is NORMAL). What happens if destination fails to put messages to queue and retry attemps are exhausted.
2- message batch is GET in a transaction (either the messages are persistent or NPMSPEED is NORMAL). What happens if destination fails to put messages to queue and sender channel is stopped before retry attempts are exhausted.
3- message batch is GET without a transaction (batch are all non-persistent messages and NPMSPEE is FAST).  What happens if destination fails to put messages to queue and retry attemps are exhausted.

Test two scenarios with Receiver failing to put message to destination queue but eventually succeeds after sender is stopped so receiver could not confirm to sender:
1- message batch is GET in a transaction (either the messages are persistent or NPMSPEED is NORMAL). : Sender channel is in-doubt and sequence number is not the same beteween sender and receiver.  What happens when sender is started again? in-doubt is resolved automatically and sequence number is updatd.
2- message batch is GET in a transaction (either the messages are persistent or NPMSPEED is NORMAL). : Sender channel is in-doubt and sequence number is not the same beteween sender and receiver.  What happens when sender is resolved to backout and then sender is started? 
3- message batch is GET without a transaction (batch are all non-persistent messages and NPMSPEE is FAST). : Sender does not become in-doubt and sequence number is updated to the latest. What happens when sender is started again?
