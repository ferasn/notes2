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
4. Queues:
  a) Local queue: only local queues can hold messages.
  b) Remote queue.
  c) Alias queue.
  d) Model queue: a template for creating a dynamic local queue.
5. Channels:
  * Channels are the processes that move messages to and from queue managers.
  * Categories of channels:
    a) Message Channel:
      a.1) One-way communications link between two queue managers
      a.2) Defined in pairs: at the sending queue manager and the receiving queue manager.
      a.3) A message channel connects two queue managers by using message channel agents (MCAs).The MCA on one end takes messages from local transmission queue and puts them on communication link. The MCA on other end receives the messages and puts them on target queue.
    b) Client (MQI) Channel:
      b.1) Two-way communications link
      b.2) Connects an application (MQI client) to a queue manager
6. IBM MQ Client: An MQ client is a component of MQ that allows an application that is running on one system to send MQI calls to a queue manager that is running on another system.
7. IBM MQ Administration Interface:
  a) MQ Explorer
  b) MQ script commands (MQSC)
  c) MQ control commands: command line interface.
8. IBM MQ Deployment Options:
  a) IBM MQ and IBM MQ Advanced.
  b) IBM MQ Appliance: MQ in a physical appliance with premium hardware including SSDs. Pre-optimized for simple setup and easy maintenance with firmeware updates.
  c) IBM MQ Cloud Options: Plug into a range of services on IBM’s PaaS, IBM Bluemix with IBM Message Hub.  Access repeatable solutions, with IBM MQ on PureApplication and SoftLayer and other Cloud and virtualized environments.
9. IBM MQ Advanced:
  a) IBM MQ + managed file transfer, advanced message security and telemetry (MQTT).
  

Unit 2 IBM MQ Installation and Deployment Options:
