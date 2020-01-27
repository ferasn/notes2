These notes are from the course but while going through the course and researching certain areas of IIB:

Message Parsing:
===============
* Input Nodes:
    - input node does not parse the message but set parser properties of the message if parse timing is "on-demand".
when the message nodes are accessed and referenced later in the flow, the message is parsed up to the point needed to access the referenced message nodes.  With "on-demand" parsing, the message propagtes from input node in its bitstream form unparsed.
    - However, if parse timing is set other than "on-demand", the parser properties are set and the message is parsed to build the logical message tree.
    - Example Test Scenario:
        1. Create a message flow with two nodes: "MQInput" and "MQOutput"
        2. Connect "out" terminal of "MQInput" to "in" terminal of "MQOutput".
        3. Create two queue "IN.Q" and "OUT.Q" and assign them to "MQInput" and "MQOutput" nodes respectively.
        4. In "MQInput" set message domain to "XMLNSC" and parse timing to "On Demand".
        5. Deply the flow and put a text message "12345" in "IN.Q". This message is not a valid XML message. You will notice that the message appears in "OUT.Q".  This means that message just propagated in its bitstream form from "MQInput" to "MQOutput" node.  If it had been parsed in "MQInput", the parser would have thrown an error and the message would not propogate.
        6. Change the parse timing to "Complete" in "MQInput" and put the same message. You will notice that no message appears in "MQOutput" and the syslog (Event Viewer in Windows) logs the error "An XML parsing error ''An invalid XML character (Unicode: 0x54) was found in the prolog of the document".
    - Note: IIB Toolkit Flow Exerciser overrides "On Demand" and does full parsing in input node. It does this to be able to show you the full message tree when viewing the message path.  So, if you try the above test scenario in Flow Exerciser with "On Demand" parse timing, you will get a different result.

* ResetContentDescriptor Node:
   - It changes the parser properties of the message and associates a new parser with the message.
   - There are two cases where this node can encouter with regard to the input message:
        1. The message is unparsed and still in its bitstream form, then chaning the parser is straightforward. This case does not have any performance penality.
        2. The message has been parsed and contents of the message changed, then this node has to recreate the bitstream from the message tree by using the current parser associated with the message.  If the content added to the message tree makes it invalid according the current parser and its model, then parser generates an error that it failed to serialize the message to bitstream and ResetContentDescriptor node raises an exception.  This case incurs performance overhead in serializing the message tree and converting it to bitstream.
   - In both cases, once bitstream is there the way the message and its parser work are the same as explained in input nodes above.  If ResetContentDescriptor node has parse timing "On Demand", the message propogates from this node to the next node in its bitstream form.  Later in the flow when the elements of the message are referenced, the parser set by this node starts parsing the message sufficiently enough for the reference.  If However parse timing is set otherwise, the message is parsed by this node the logical message tree is propogated to the next node.
   - Note: this node does not do transformation. So, this node will not change from XML to JSON by just changing the parser.  It only changes the parser.  What happens if one changes the parser from XML to JSON is that once the bitstream of XML message is recreated (it is not alraedy in bitstream form), the JSON parser will try to parse and will throw this error "A JSON parsing error occurred. A message that is not a valid JSON message was found in the input bit stream."


IIB Toolkit Flow Excerciser:
============================
* In Flow Exerciser, if you send a message through the flow and click "view path" to show the path and the message tree and it says "no data retrieved for this flow", it means that the message failed in the input node and did not propogate through any terminal.
