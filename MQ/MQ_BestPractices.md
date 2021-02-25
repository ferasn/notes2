* Best performance of GET and PUT of messages:
  - Persistent messages: GET and PUT persistent messages in a syncpoint even it is one message.
  - Non-persistent messages: GET and PUT non-persistent message outside a syncpoint.
    the message you get it in a transaction (syncpoint) in case of failure the message is rolled back.  
  - References:
    1. https://www.ibm.com/support/knowledgecenter/SSFKSJ_9.2.0/com.ibm.mq.dev.doc/q023850_.html
    2. Section 1.3.1.2 Concurrency, Syncpointing, and Queue Locking: https://ibm-messaging.github.io/mqperf/mqio_v1.pdf 


* Considerations to use a syncpoint:
  - You need a group of operations to be atomic (a group of messages put together will be visible for consumers at the same time).
  - When doing GET, you do not want to lose messages:
    1. Non-persistent: 
    2. Persistent: GETs of a persistent message outside of syncpoint do not make any logical sense, as MQ cannot verify that a message arrived at the application after it has been destructively read. A GET of a persistent message should not complete until the application has verified receipt of the message (via a call to MQCMIT).
    3. Reference: Page10 https://ibm-messaging.github.io/mqperf/mqio_v1.pdf