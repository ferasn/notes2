IIB Nodes:
==============

1- Filter Node:
	* evaluates ESQL expression to return BOOLEAN value. 
	* Has four output terminals. Depending on the value of the BOOLEAN value, message is propagated to the appropriate output terminal:
		–TRUE
		–FALSE
		–UNKNOWN (test field not present)
		–FAILURE (runtime error during node processing): if it is wired; otherwise, default error handling takes place
	* The output message is the input message (cannot alter the message)
	* If you code RETURN without an expression (RETURN;) or with a null expression, the node propagates the message to the Unknown terminal.
	* The Filter node can access a relational database table.
	
2- Route Node:
	* Direct messages down various paths, based on filter criteria
	* The filter is an XPath expression. The filters are populated in a filter table, with each filter corresponding to an output terminal.
	* All XPath expressions must start with $Root, $Body, $Properties, $LocalEnvironment, $DestinationList, $ExceptionList, or $Environment. Expressions are evaluated in the order in which they are displayed in the table.
	* Minimum of three output terminals; can add more:
		- "Match" terminal when filter evaluates to true
		– "Default" terminal if none of the filter expressions are true
		– "Failure" terminal if runtime error occurs (if wired)
	* Can propagate to multiple terminals, depending on distribution mode property of the node:
		- First: the Route node terminates after encountering the first match in the table
		- All: all the table is scanned for matches
		
		
3- Explicit Error Handling:
	* ExceptionList: part of the logical message tree and contains nested "RecoverableException" structures. The innermost structure contains information that is the original cause of the exception.
	* Error handling in input nodes:
		- If the node detects an internal error (e.g. parsing the message) before the message is propagated to the Out terminal, the node always propagates the message and an exception list to the Failure terminal if the node has a Failure terminal and if you have connected a fail flow. If the Failure terminal is not connected or an exception occurs downstream of the failure terminal, the transaction is rolled back (or message is discarded if the flow is not transactional).
		- If you connect the Catch terminal (if the node has one), this indicates that you want to handle all exceptions that are generated in the out flow. If you do not connect the Catch terminal, or the node does not have a Catch terminal, or an exception occurs downstream of the Catch terminal the current transaction is rolled back (or message is discarded if the flow is not transactional).
		- Any internal exceptions that occur in the node after the message has been propagated to the Out terminal cause a rollback of the transaction. This situation is rare but could happen for some input nodes.
		Reference: https://www.ibm.com/support/knowledgecenter/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ac18890_.htm
		MQInput Error Handling: https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ac00414_.htm

	* TryCatch Node:
		- has two output terminals "Try" and "Catch". Message is passed to Try branch; runs to completion unless exception occurs.
		- If exception occurs in Try branch:
			* Broker updates ExceptionList
			* Existing ExceptionList information is written to local error log
			* Root and LocalEnvironment reset to state at start of TryCatch node
			* Database changes and MQPUTs in Try branch before the exception occurred are not rolled back. To force a rollback, you must explicitly throw an exception on the Catch branch.
			* Enter Catch branch; if not connected, return control to Input node
			
	* Throwing and logging exception:
		- Throw node: You often wire a Throw node as the last node in a Catch path. It allows a user-defined exception to be thrown. You can configure the Throw node to output specific diagnostic information to the syslog or the ExceptionList. The Throw node has a single input terminal, but no outputs.
		- ESLQ THROW function:
			* in Compute, Filter, or Database node.
			* Database return codes can be retrieved by using ESQL functions if advanced property "Throw exception on database error" is not selected in the node.
		- JavaCompute node class "MbService":
			* Provides Event log and syslog methods logError(), logWarning(), logInformation()
			* Message text is heldin Java resource bundles
			
	* Flow order node:
		- Explicitly controls the order of message processing along two branches of a flow: "First" then "Second" output terminals.
		- The same message tree that was sent through First terminal is sent through Second terminal, disregarding the changes done through the first terminal.
		- If a runtime exception occurs while processing the First branch,the Second branch is not executed.
		- FlowOrder often used in error handling flow where First branch does some logging and event publishing and the second node has "Throw" node connected to backout the message to the input node.
		
	* Backout threshold "BOTHRESH"( property of the MQ queue):
		- If the catch terminal of the MQInput node is connected, the message is propagated there. ExceptionList entries are available. Root and LocalEnvironment are reset to their initial values. Otherwise, if it is not connected, the transaction status of the message is considered:
			* If the message is not transactional, the message is discarded.
			* if it is transactional, the message is returned to the input queue, and it is read again, whereupon the backout count (property in MQMD header) is checked.
		- When the message is read again from input queue, the backout threshold is checked. If it is exceeded:
			* If the failure terminal of the MQInput node is connected, the message it propagated to the failure terminal. In this case, the exception list does not contain the original exceptions that occurred in the flows connected to the Out or Catch terminals; the exception list contains new exceptions that cover the reason that the message has gone through the Failure terminal (in this case: the backout threshold was reached). If exception happens down the failure terminal path, retries are attempted until 2*BOTHRESH. after that, the message is put backout queue or DLQ if one is not defined.
			* If the failure terminal is not connected, then the message is put in the backout queue if it is defined. If it is not defined, it is put in dead letter queue (DLQ).  If neither are available, or put to these queues fail, the message is backed out to the input queue and retries continue.
		- For the message to be retried at least once, the BOTHRESH property has be set to 2 or greater. Setting BOTHRESH to one or the default zero means the message will be processed once but not retried.
			
