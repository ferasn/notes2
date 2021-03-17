* https://www.ibm.com/support/pages/which-mq-process-indication-active-instance-running-multi-instance-queue-manager
* https://www.ibm.com/support/pages/node/219139
* mqconfig: Check if opearting system meets minimum MQ requirements
* runmqras: Gather diagnostic information together into a single archive, for example to submit to IBM Support. https://www.ibm.com/support/pages/read-first-collect-ibm-mq-mustgather-data-linux-unix-windows-and-ibm-i
* https://www.ibm.com/support/pages/there-way-summarize-contents-fdc-files-yes-using-mq-utility-ffstsummary
* https://www.ibm.com/support/pages/ibm-mq-error-log-ffst-and-dump-locations

FFST: 
The terms FFST and FDC are sometimes used synonymously. These are files written by MQ to record errors and dump information about what occurred at the time MQ encountered the error. 
FFST stands for First Failure Support Technology, and is technology within WebSphere MQ designed to create detailed reports with information about the current state of a queue manager together with historical data. These errors are written to files named as AMQxxxxx.y.FDC where xxxxx is the process ID (PID) that encountered the error. Hence, the *FDC files include the FFST reports. All threads in a process will append their FFSTs to the same file.
https://www.ibm.com/support/pages/there-way-summarize-contents-fdc-files-yes-using-mq-utility-ffstsummary

IBM APARs:
An Authorized Program Analysis Report, or APAR, is a formal report to IBM development of a problem caused by a suspected defect in a current release of an IBM program.
If IBM development is able to confirm the existence of the defect, the APAR record will be updated with any known workarounds.
The APAR record will then be published so that it is visible to supported customers.
https://www.ibm.com/support/pages/open-apars-ibm-products-available-web
