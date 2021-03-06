* Non-persistent messages will never be written to the Recovery Log – thus they can (and will) be lost or discarded, even in nonfailure situations. This is true even if the non-persistent messages are put under syncpoint – even if the UOW includes both persistent and non-persistent messages. All transaction state for non-persistent messages is held in memory – never written to the recovery log. So including non-persistent messages in a transaction will not result in any log I/O – and thus will not be recovered after a failure.
  Reference: MQTC_v2016_More_Mysteries_of_the_IBM_MQ_Logger_final.pdf  page 10
  
* Persistent messages are always written to the recovery log, except in one unique circumstance – when it is put outside syncpoint to a waiting getter that is also not using syncpoint.  Reference: MQTC_v2016_More_Mysteries_of_the_IBM_MQ_Logger_final.pdf

* GETs of a persistent message outside of syncpoint do not make sense from a business logic point of view, as MQ cannot verify that a message arrived at the application after it has been destructively read (if the message is lost at the network layer, MQ will already have destroyed it). A GET of a persistent message should not complete until the application has verified receipt of the message (via a call to MQCMIT).
  References: 
    1- https://ibm-messaging.github.io/mqperf/mqio_v1.pdf
    2- http://ibm-messaging.github.io/mqperf/MQ_for_Windows_V910_Performance.pdf

  
* Hardening messages: means writing them to the recovery log.

* Transaction log Triple write:
  1- Log records are written to pages of size 4k. The log buffer and file are organized into pages of 4k. When a transaction (or transactions) commit, the log buffer pages are written to log file (hardened).
  2- The last page may be not be full of log records and still has free space.  Still, the whole page is written to log file.
  3- New transactions will fill the empty space in this partially full page.  When those transactions commit, this page has be written once again to the log file replacing the old page.
  4- This situation presents a problem: if the storage cannot guarantee writing the 4k page to persistent storage atomically (that is either all of the 4k page is written or none.  Prtial writing can corrupt the data in the page as in the case when mechanical disk writes 4k data as a series of 512 sector writes), the log records of previously committed transactions int the page are corrupted and not recoverable.
  5- This is why triple write is employed just for the case of this last 4k page.
  References:
    1- https://www.ibm.com/developerworks/websphere/library/techarticles/0712_dunn/0712_dunn.html
    2- MQTC_v2016_More_Mysteries_of_the_IBM_MQ_Logger_final.pdf

Resources:
1. MQTC_v2016_More_Mysteries_of_the_IBM_MQ_Logger_final.pdf
2. MQTC_2017_Whats_New_in_MQ_Logging.pdf
3. MQTC_2018_MQ91_LoggerEnhancements_final.pdf
4. IBM MQ - Linear and Circular logging: https://www.ibm.com/support/pages/node/5737737
5. Webcast replay: IBM WebSphere MQ - An Introduction to Logging: https://www.ibm.com/support/pages/node/321037
6. Tool and instructions to use the MQ recovery log to find out what happened to your persistent MQ messages: https://www.ibm.com/support/pages/wheres-my-message-tool-and-instructions-use-mq-recovery-log-find-out-what-happened-your-persistent-mq-messages-distributed-platforms

7- https://ibm-messaging.github.io/mqperf/mqio_v1.pdf
8- http://ibm-messaging.github.io/mqperf/MQ_for_Windows_V910_Performance.pdf
9- https://www.ibm.com/support/pages/how-calculate-total-amount-disk-space-recovery-logs-one-queue-manager
10- Configuring and tuning WebSphere MQ for performance on Windows and UNIX: https://www.ibm.com/developerworks/websphere/library/techarticles/0712_dunn/0712_dunn.html
11- https://www.ibm.com/support/knowledgecenter/SSFKSJ_9.2.0/com.ibm.mq.con.doc/q018410_.html
12- https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.2.0/com.ibm.mq.con.doc/q018860_.html
13- https://www.ibm.com/support/pages/how-calculate-total-amount-disk-space-recovery-logs-one-queue-manager
