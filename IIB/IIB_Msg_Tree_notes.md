Message are copied between trees 

* InputTree -> OutputTree
* InputTree -> Environment

Some fields are owned by a parser


* The process of creating the output message is referred to as serialization or flattening of the message tree


Tow experiments:
* ESQL node that propogates multiple messages and then exists.  The control remains within the node while the flow on the output terminal run multiple times per msg
* Modification of Input message tree
* Message rolling backing to its original input at catch terminal



-------------------------------

Throughout the the msg flow there are multiple trees created.


|--------T1----------|---------------T2------------|
MQInput				ESQL-Transform				
