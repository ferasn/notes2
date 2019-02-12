Java Computer Node:
===================

* The node class must extend 'MbJavaComputeNode'.
* Must implement 'evaluate()' method, which the broker calls once for each message processed.
* The incoming message is passed into the evaluate method as part of a message assembly. 'MbMessageAssembly' encapsulates four 'MbMessage' objects.  'MbMessageAssembly' is proporgated to output terminal.
* 'MbElement' represents a “parsed” logical part of the message.  It has three main methods: 
	- getType(): Generic type that returns one of NAME, VALUE or NAME/VALUE
	- getName()
	- getValue(): returns an object of parser determined type
	In addition to other properties such as namespace URI, parser-specific type
