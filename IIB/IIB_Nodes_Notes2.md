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
		
