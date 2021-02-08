Unit 1 IBM MQ Review:
1. IBM MQ uses messages and queues to provide program-to-program communication. Multiple applications can put messages on the same queue to request the same service.  Each requesting application can have its own reply-to queue so that each client application can receive its replies separately from other client applications. 
IBM MQ provides:
  a) Assured message delivery: messages do not get lost or duplicated.
  b) Time-independent processing: communicating applications do not have to be active at the same time.
  c) Application parallelism.
2. Messages contain:
  a) message descriptor (MQMD) header with message control information. The application that puts the messages on the queue sets some of the fields in the message descriptor; the queue manager sets others on behalf of the application.
  b) optional additional headers
  c) applicaiton data or payload
3. Message Persistence: persistence is specified by the application in MQMD
  a) Persistent messages: Queue manager keeps a failure-tolerant recovery log of all actions on messages. These message are recoverable after MQ restarts.
  b) Non-persistent messages: Stored in system memory only. Discarded after MQ restarts. Used when data loss is tolerable and performance is more important.
