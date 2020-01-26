These notes are a summary of IIB Dev 10 course.

* Output terminal: A node output terminal can have multiple connections, which result in a duplication of the message, or multiple different messages. The order in which any of the output paths are taken is random, but the paths are run sequentially.
* Input terminal: Multiple connections to the same input terminal are not message aggregation.
* Input Node: converts incoming data from a stream of bits into a logical tree structure through a "parsing" operation.
* Output Node: creates outgoing data (in the bit stream) from the tree structure by a "serialization" operation.
* Logical message tree (or message assembly):
  * contains the original data, plus other information about the runtime environment, message transport properties, and other “control” information.
  * Transport type determines the logical tree headers that are included in the tree
  * The tree structure begins with a node called Root. Under Root, other nodes and subtrees are created:
    - message properties
    - headers
    - Body
  * If a runtime error occurs, a special tree that is called an Exception List that contains information about the error, is constructed and attached to the Root tree node.
* IIB Administration Tools:
  * IBM Integration Toolkit, IBM Integration web user interface, Command interface, Custom written tools.
  * All access integration bus node through "Integration API".
  
* The three containers of resources in IBM Integration Toolkit:
  * Integration Project: specialized container in which you create and maintain all the resources that are associated with one or more message flows. In the toolkit, it is shown under "Independent Resources". It contains messages flows and their resources:
    •Message flows
    •Subflows
    •Message maps
    •ESQL files
    •Database definitions
    •BAR files
  * Application:
    * can contain any Integration Bus resource. It can also contain references to message flow dependencies, such as a Java project or message model. It can also reference libraries of reusable components
    * You can deploy multiple applications to an integration server, and the visibility of a resource is restricted to the application in which it is contained.
  * Library:
    * Logical grouping of related reusable resources (message models, maps, and subflows) that an application uses
    * Can be shared or static
    * Static Library:
      - Application gets its own copy of the library and the resources that it contains
      - To update a library, every application that uses the library must be repackaged and redeployed with the updates
    * Shared Library:
      - Shared libraries are deployed directly to the integration server
      - Message flows are not allowed in a shared library
      - Shared libraries do not have a running state; they cannot be started or stopped and no runtime threads are assigned to shared libraries
      - Shared libraries can reference other shared libraries, but not static libraries. Similarly, a static library cannot reference a shared library. An integration project cannot reference a shared library.
      - Applications automatically recognize updates to shared library
      - Application deployment fails if a required shared library is not already deployed
      - Cannot remove a shared library from an integration server while deployed applications are referencing it
* Broker schemas (namespaces): subdirectory that provides a way to organize objects into a namespace to prevent duplicate names in the workspace. The namesapces is appended to the name of the object with "." as seperator in the integration node.
* Message flow node properties:
  * Mandatory: You must set values for these properties at "design time".
  * Optional: You can set values for these properties at "design time"; otherwise, a default is used.
  * Configurable: You can override values in the "deployment file (BAR)" or at "run time".
  * Promoted: Elevates the property to the message flow level instead of the individual node level; the property value can then be changed at the message flow level. Also, in the "deployment file (BAR)" or at "run time".
* Patterns: Catalog of IBM Integration Bus patterns is available on OT4I GitHub Pattern Repository.
* Flow exerciser: Show the path that each message took.  View the structure and content of the logical message tree at any point in a message flow. Save recorded messages for replay.
* BAR file creation and deployment:
  * When you add a message flow to a BAR file, you can add the flow as a .msgflow file, or as a compiled message flow (defined in a .cmf file). You cannot add the same message flow to a BAR file as both a .cmf file and a .msgflow file.
  * Deploying all the resources in the compiled form requires that all the subflows are implemented in .msgflow files. As a result, all the flow, subflow, and ESQL code is in-lined into the parent .cmf file. This method does not allow the same flexibility as the source deployment, but it does provide an unambiguous runtime behavior.
  * If the BAR file contains a mixture of resources that are compiled and resources that are not compiled, you might see unexpected results.
  * The BAR file contains the broker.xml file. This file is called the deployment descriptor. This file, in XML format, is in the META-INF folder of the .zip file and can be modified by using a text editor or shell script.
  * When creating a new bar file in the toolkit, the BAR file is created in the "BarFiles" project by default. The default can be changed in preferences.
  * If you elect to include source files in the BAR file (by selecting the option "Add workspace project source files" in the prepare tab), these resources are stored in a separate folder, which is named src in the bar file.
* The IBM Integration web user interface is enabled and assigned to port 4414 by default.

Unit 4: Connecting to IBM MQ
* MQInput, MQOutput, MQGet, and MQReply nodes can connect to local or remote queue managers.
* MQInput nodes can receive messages from multiple queues on multiple queue managers.
* Globally coordinated transaction control requires that MQInput and MQOutput nodes use same "local" queue manager. This queue manager is the global transaction manager, and no other IBM MQ resources can be used in the message flow.
* You can specify a connection from an IBM MQ node to a specific local or remote queue manager by using connection properties on the node, including the destination queue manager name, host name, port, and channel. Alternatively, you can use an "MQEndpoint policy" to control the values of IBM MQ node connection properties at run time, or to specify an IBM MQ broker for event publication.
