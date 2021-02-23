* MQ Connections:
  1. Two types:
    a) Server connection: connection made through IPC binding.
    b) Client connection: connection made through MQ client.
    
* MCA binding

* Server channel sharing converstation

* A channel can be in two states:
  1- Inactive
  2- Current.  This state has substates: Stopped, Starting, Retrying, Active.  So a CURRENT channel is ACTIVE unless it is in RETRYING, STOPPED, or STARTING state.
    The seven possible states of an active channel (INITIALIZING, BINDING, SWITCHING, REQUESTING, RUNNING, PAUSED, or STOPPING).  When a channel is active, it is consuming resource and a process or thread is running.
    ... MAXchannels MAXActiveChannels....
