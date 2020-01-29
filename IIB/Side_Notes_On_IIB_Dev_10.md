These notes are from the course but while going through the course and researching certain areas of IIB:

Message Parsing:
===============

* Parse timing and validation:
    - Message Validation:
      * Validation done can be either on input message or output message depending on the node type. See: https://www.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/ac00400_.html
      * The following options are available:
        a. "None": The default value. No validation is performed.
        b. "Content", "Content and Value" : are the same for SOAP, DFDL, and XMLNSC domains.  It makes a difference for MRM domain.
        c. "Inherit" : (The default for output nodes) available in some nodes other than input nodes. It inherites the validation properties of the input message.
    - Parse Timing:
        a. "On Demand": (partial parsing) message is parsed as elements of the message are accessed.  Also, validation is done the same way.
        b. "Immediate", "Complete": are the same for all domains except MRM.  The whole message is parsed and validated (if validation is not "None").
        
* Validation Failure actions:
    - Exception: The default value. An exception is thrown on the first validation failure encountered. The resulting exception list is shown below. The failure is also logged in the user trace if you have asked for user tracing of the message flow, and validation stops. Use this setting if you want processing of the message to halt as soon as a failure is encountered.
    - Exception List : it does not stop at first validation excpetion but continues until current parsing is completed and generates a list of all validation exceptions and processing stops. Each failure is also logged in the user trace if you have asked for user tracing of the message flow. This property is affected by the Parse Timing property; when "On Demand" parsing is selected the current parsing operation parses only a portion of an input message, so only the validation failures in that portion of the message are reported.
    - User Trace: Logs all validation failures to the user trace, even if you have not asked for user tracing of the message flow. Use this setting if you want processing of the message to continue regardless of validation failures.
    - Local Error Log: Logs all validation failures to the error log (for example, syslog on Unix or the Event Log on Windows). Use this setting if you want processing of the message to continue regardless of validation failures.


        
* Every message carries with it two meta-data:
  1. Parsing properties: The type of the parser (XMLNSC, DFDL, JSON, ..etc) and parser timing ( On Demand, Immeidate, Complete) .
  2. Validation properties: Type of validation (None, Content, Content and Value) and validation failure actions.

* Input Nodes:
    - "on-demand" parsing: input node does not parse the message but set parser properties for the message.  When the message elements are accessed and referenced later in the flow, the message is parsed up to the point needed to access the referenced message elements.  With "on-demand" parsing, the message propagtes from input node in its bitstream form unparsed. Also, input node sets the validation properties for the message if validation is specified (not "None").  Validation is performed "On-Demand" as parsing is taking place.  So, in this case, validation errors can be generated later in the flow as elements are accessed. This also has implications that validation errors might never be detected if a portion of the message is never parse.
    - However, if parse timing is set other than "on-demand", the parser properties are set and the message is parsed to build the logical message tree. Also, validation properties are set for the message and validation is performed if specified (not "None").
    - Example Test Scenario:
        1. Create a message flow with two nodes: "MQInput" and "MQOutput"
        2. Connect "out" terminal of "MQInput" to "in" terminal of "MQOutput".
        3. Create two queue "IN.Q" and "OUT.Q" and assign them to "MQInput" and "MQOutput" nodes respectively.
        4. In "MQInput" set message domain to "XMLNSC" and parse timing to "On Demand".
        5. Deply the flow and put a text message "12345" in "IN.Q". This message is not a valid XML message. You will notice that the message appears in "OUT.Q".  This means that message just propagated in its bitstream form from "MQInput" to "MQOutput" node.  If it had been parsed in "MQInput", the parser would have thrown an error and the message would not propogate.
        6. Change the parse timing to "Complete" in "MQInput" and put the same message. You will notice that no message appears in "MQOutput" and the syslog (Event Viewer in Windows) logs the error "An XML parsing error ''An invalid XML character (Unicode: 0x54) was found in the prolog of the document".
    - Note: IIB Toolkit Flow Exerciser intercepts the message along the paths between nodes and does serialization of the message tree in order to able to display it to the user. This serialization involves full parsing of the message first if not already parsed. This changes the behaviour of the flow in case of "On Demand" parse timing as the message bitstream is fully parsed as soon as it leaves the input node and before it reaches the next node.  So, if you try the above test scenario in Flow Exerciser with "On Demand" parse timing, the parsing that occurs after the message bitstream leaves the "out" terminal of input node generates an error and this error goes back to the input node and is propogated to "catch" terminal if connected.
    
* Compute Node:
   - Like Parser Copies: 
     a. When a tree is copied between two parsers that are the same, for example in ESQL:
       SET OutputRoot.XMLNSC.myTesData.Data = InputRoot.XMLNSC.myInputData.myData
     b. In this case, the integration node can be certain that the tree can be fully represented by the target parser and so all parser information is copied. The information specific to the parser that is added to the elements (e.g. namespaces, types specific to the parser like : XMLNSC.XmlDeclaration, XMLNSC.Attribute, ..etc.)
   - Unlike Parser Copies:
     a. When you copy a tree between two different parsers, for example in ESQL:
       SET OutputRoot.JSON.Data = InputRoot.XMLNSC;
     b. In this case, the integration node cannot know whether the source tree can be fully represented by the target parser. This behavior is because parsers do not all support the same type of structure and content information. After an unlike parser copy, the target tree consists only of Name-Value pairs under the target parser with no other parser information (e.g. namespaces, types specific to the parser like : XMLNSC.XmlDeclaration, XMLNSC.Attribute, ..etc.).  So, an element that is an XMLNSC.Attribute in the tree of XMLNSC parser will be copied as a simple Name-Value element to tree of JSON parser.
    Reference: https://www.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/ac70570_.html
    
    - Note: in "Compute Node", a statement like "SET OutputRoot.XMLNSC = InputRoot.XMLNSC" does not reference any element in the input message so if the input to "Compute Node" is an unparsed bitstream, this statement will not cause the bitstream to be parsed and the bitstream will be copied as is to OutputRoot. So, if you pass an invalid XML message like in the example test scenario above, it will pass through Compute Node to "out" terminal without causing parsing error.  However, if the parser is changed like "SET OutputRoot.JSON.Data = InputRoot.XMLNSC", this will cause current parser of input message to parse the whole message to produce the logical message tree in order for it to be copied to OutputRoot.  Here bitstream cannot be copied to OutputRoot because it will fail once JSON parser tries to parse XML bitstream later in the flow. So, the logical tree (which is indepedent from bitstream format) has to be constructed to be copied if parser is changed.
    
    - Creating new parser and setting its properties and validation properties explicitly through "CREATE" statement in ESQL:
    https://www.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/ak04950_.html
    Test: Check if Compute Node validation options override the valiation options set using CREATE statement in ESQL.
    
* Java Compute Node:
    - Whatever mentioned above for Compute node also applies to Java Compute Node.


* ResetContentDescriptor Node:
   - It changes the parser properties and validation properties for the input message.  A new parser gets associated with the message.
   - There are two cases this node can encouter with regard to the input message:
        1. The message is unparsed and still in its bitstream form. Chaning the parser in this case is straightforward. This case does not have any performance penality.
        2. The message has been parsed and contents of the message changed, then this node has to recreate the bitstream from the message tree by using the current parser associated with the message.  If the content added to the message tree makes it invalid according the current parser and its model, then parser generates an error that it failed to serialize the message to bitstream and ResetContentDescriptor node raises an exception.  This case incurs performance overhead in serializing the message tree and converting it to bitstream.
   - In both cases, once bitstream is there, the way the message and its new parser work are the same as explained in input nodes above.  If ResetContentDescriptor node has parse timing "On Demand", the message propogates from this node to the next node in its bitstream form.  Later in the flow when the elements of the message are referenced, the parser set by this node starts parsing the message sufficiently enough for the reference.  If However parse timing is set otherwise, the message is parsed by this node and the logical message tree is propogated to the next node.
   - Note: this node does not do transformation. That means this node will not convert the message from XML to JSON by just changing the parser.  It only changes the parser.  What happens if one changes the parser from XML to JSON is that once the bitstream of XML message is recreated (if it is not alraedy in bitstream form), the JSON parser will try to parse it and will throw this error "A JSON parsing error occurred. A message that is not a valid JSON message was found in the input bit stream.".  So, for this node to work both the old parser and the new parser have to able to parse the same bitstream.  Example: from BLOB parser to XMLNSC parser.


References:
================
* This page in knowledge center about parsing strategies only exists in v9.0 documentation but not in later versions:
https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.scenarios.doc/gop_03/topics/bj60039_.htm
