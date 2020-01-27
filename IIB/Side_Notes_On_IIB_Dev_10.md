These notes are from the course but while going through the course and researching certain areas of IIB:

* Message Parsing:
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
