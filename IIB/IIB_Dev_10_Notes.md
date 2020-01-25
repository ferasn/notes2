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
    
