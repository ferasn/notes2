* Message Assembly: For logical Trees:
	* Root
	* Environment
	* LocalEnvironment
	* ExceptionList

* When an MQInput node receives a message, a message assembly of four different message trees is created. While the parsed message is written into the Root tree, the other three trees are initially empty. The trees can be populated:
	•By certain processing nodes (the LocalEnvironment)
	•By the broker when an exception occurs (ExceptionList)
	•By ESQL or Java statements that are issued from JavaCompute, Compute, Filter, or Database nodes (all trees).

* Elements in the tree are addressed by using XPath or dotted name notation. Example: TRANSACTION.CUSTOMER.FIRST_NAME

* Each leaf node in the tree contains both a label and a value.  The data types that are supported in a leaf node come from a limited and simple set such as string, integer, and float.  Element values are stored in Unicode to facilitate code conversions

* Message Domain = parser.  A parser might or might not use a model when parsing.
* Different parsers (domains) each parse a different part of the message (MQMD, other headers, and the body). As a parser completes its part of the message, it passes to the next. The information that is passed includes where the unparsed data starts so that the next parser knows where to start. This results in a series of logical “trees” that can then be used to refer to specific elements within the message. In addition, the message has some properties carried with it. The properties never exist when the message is on a queue. They accompany messages only as they go through a message flow.
* It is only the parser that needs to understand the physical format of the message. The logical message tree is independent of the physical format of the message bit stream. This decoupling of physical format from transformation logic is a key architectural feature of broker message flows.
* MQ Root Tree:
	|-> Properties
	|-> MQMD
	|-> MQRFH2
	|-> ....
	|-> (Application Data)
	
* Self-defining (programmatic) message domains:
	* XMLNSC: Can optionally use model (XML schema) to validate XML documents.  you can specify to parse elements opaquely by XMLNSC if the elements in a message are never referenced by the message flow. Opaque parsing reduces the costs of parsing and writing the message, and might improve performance in other parts of the message flow.
	* JMSMAP and JMSSTREAM
* Predefined (model-driven) message domains:
	* MRM.  (stands for: Message Repository Manager)
	* SOAP
	
* In the BAR file, only XSL files and Java code (in a *.jar file) are visible. ESQL and mappings code is included in the compiled message flow (*.cmf file).

ESQL:
* ESQL correlation names: Pointers into message tree:
	* Root, InputRoot, OutputRoot
	* Body (=Root.*[<]), InputBody (=InputRoot.*[<]).  Body is a shortcut for “the last child of Root.”
	* LocalEnvironment, Environment, ExceptionList
	* Root is the highest level identifier in the logical message tree. This concept is still true in nodes that cannot alter the message (such as the Filter node). In the Compute node, however, since two different messages are in use (input and output), you need to prefix them with “Input” or “Output”. However, OutputBody is NOT a valid correlation name (like InputBody) because the broker cannot determine the message domain (parser) to use to build the body. Thus, you must always write explicitly: SET OutputRoot.XMLNSC.a.Field, or SET OutputRoot.MRM.b.Field.
	* Reference: https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ac00510_.htm

* Data types:
	- Boolean: TRUE, FALSE, UNKNOWN
	- Datetime: DATE, TIME, GMTTIME, TIMESTAMP, GMTTIMESTAMP, INTERVAL
	- NULL
	- Numeric: DECIMAL, FLOAT, INTEGER, INT
	- String: BIT, BLOB, CHARACTER
	- REFERENCE TO: creates a dynamic reference type.
	- ROW
		* When declaring a ROW variable, a single field is created.  You can assign a value to the field and add children.
			Example:	
				DECLARE myrow ROW;
				SET myrow = 'Hello';  -- set the value of the field pointed to by myrow to 'Hello'
				SET myrow.abc = 'World'; -- create a child abc and set its value to 'World'
		* ROW() constructor function creates a parent field with no value and the function parameters as children fields.
			Example: ROW('granary' AS bread, 'riesling' AS wine, 'stilton' AS cheese);
	- LIST
		* LIST constructor function:  creates a list of values
			Example: LIST{'red', 'green', 'blue'};
		* The fields in a message tree can be obtained as a list using the array field reference (indicated by [] suffixed to the last element of the reference).
		* Assignment of LIST of array field reference: https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ak05640_.htm

* Field references and reference variables (dynamic references):
	- Static references: such as correlation names like Root, and ROW variables.
	- Dynamic references: by declaring a REFERENCE variable, you obtain a dynamic reference or message cursor that can be moved in the message tree.
		* if a dynamic reference is a pointer to InputRoot, no modifications can be done (you cannot SET).
		* If you declare a REFERENCE variable to a non-existent field, the message cursor always points to Root.  You need to check the success of reference or reference move by using LASTMOVE function or checking the name of the field the reference is pointing to with FIELDNAME function
			Example:
			
			DECLARE refCust REFERENCE TO InputBody.OrderMsg.Customer;
		
			1) IF LASTMOVE(refCust) THEN...
			2) IF FIELDNAME(refCust) = ‘Customer’ THEN... .
	
	
	- dynamic field reference vs Index array index:
		WHILE count < 32 DO
			 SET TOTAL = TOTAL + InputRoot.XMLNS.Data.Invoice[count].Amount;
			 SET COUNT = COUNT + 1 
		END WHILE;
		
		Use this kind of construct with care, because it implies that the broker must count the fields from the beginning each time round the loop. If the repeat count is large, performance will be poor. In such cases, a better alternative is to use a field reference variable.
		
	- Dynamic field references allow you to cut and paste parts of the logical message using DETACH and ATTACH:
		DETACH myRef;
		ATTACH myRef TO OutputRoot.XMLNSC.OrderMsg AS FIRSTCHILD;
	
	- Resources for field references:
	https://www.ibm.com/support/knowledgecenter/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ak04861_.htm
	https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ak04864_.htm
	https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ak04862_.htm
	https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ak04863_.htm
	https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.etools.mft.doc/ac00510_.htm
	
	
		
* ROW and LIST comparison:
	- Rule: when a field reference is matched against the field another reference, only the value is matched but not the name:
		example: a.b = a.c -- evaluates to TRUE only value of b is equal to value of c.  The field names b and c are not matched.  This does not apply to children of b and c.  It only applied to the directly referenced fields.
	- The same also applies in assignment: SET a.b = a.c; -- sets the value of b to the value of c.  the field with name b does not change its name to c.
		to change a field name use NAME clause in SET statement:  SET a.b NAME = 'c'; -- changes the name of field b to c
	- ROW:
		* the root field values are compared and the children names and values one to one.
		Example: consider this input message:
			<a>
				<b>
					<x1>1</x1>
					<x2>2</x2>
				</b>
				<c>
					<x1>1</x1>
					<x2>2</x2>
				</c>
			</a>
			
			a.b = a.c; -- evaluates to TRUE.  the filed name b is different from field name c but root tree field names in comparison are not matched
	- LIST: the field values are compared regardless of names. It is as if you are comparing the corresponding field references directly.
		Example:
			<a>
				<b>
					<b1>1</b2>
					<b2>2</b2>
				</b>
				<c>
					<c1>1</c1>
					<c2>2</c2>
				</c>
			</a>
			
			a.b.*[] = a.c.*[] -- evaluates to TRUE.  This will be handled as if you are doing: a.b.b1 = a.c.a1 and a.b.b2 = a.c.c2.  The values are matched but not field names.
		
* Variables:
	* Scalar and fixed data type
	* Tree structures (ROW and REFERENCE).  ROW variables can be used to create a modifiable copy of the input message.
	Examples of variable declarations:
		- DECLARE var, I INTEGER 1;
		- DECLARE addr NAMESPACE ‘http://www.ibm.com/address’;
		- DECLARE course CONSTANT CHAR ‘WM663’;
		- DECLARE rowCust ROW InputRoot.XMLNSC.OrderMsg.Customer;
		
	* Long-lifetime variables: have an extended lifetime beyond the lifetime of a single message passing through a node. They are shared between threads and exist for the life of a message flow. Strictly speaking, the lifetime is the time between configuration changes to a message flow).  Long-lifetime variables are created by using the SHARED keyword on the DECLARE statement.  
		Example: DECLARE s_counter SHARED INT 1;
		Tech article: https://www.ibm.com/developerworks/websphere/library/techarticles/1405_dunn/1405_dunn.html
	* ATOMIC statement: Only one instance of a message flow (that is, one thread) is allowed to execute the statements of a specific BEGIN ATOMIC... END statement (identified by its schema and label), at any one time.
	* It is not necessary to use the BEGIN ATOMIC construct in flows that are never deployed with more than one instance. It is also unnecessary to use the BEGIN ATOMIC construct on reads and writes to shared variables. The broker always safely writes a new value to, and safely reads the latest value from a shared variable. ATOMIC is only required when the application is sensitive to seeing intermediate results (i.e: updates to more than one shared variable where all to happen atomically).

* ESQL SPECIAL VALUES:
	* UNKNOWN: synonymous with NULL. It is used mainly for BOOLEAN when the value is neither TRUE or FALSE.  Example: as the value of RETURN in Main function of a module.
		- If you assign UNKNOWN to a variable and then test the variable equality to NULL (IS NULL), the result is TRUE.
	* NULL: A value of NULL means that the value is unknown, undefined, or uninitialized.
		- All ESQL data types (except REFERENCE) support the concept of the null value.
		- Null values can arise when you refer to message fields that do not exist, access database columns for which no data has been supplied, or use the keyword NULL, which supplies a null literal value.
		- If an expression returns a null value its data type is not, in general, known. This can be regarded as their belonging to the data type NULL , which is a data type that can have just one value, null.
		- General rule in comparing two operands using a comparison operator: if either operand is NULL, the result of comparison is also NULL.
		- Assigning NULL to a field, deletes the field. Example: SET a.b.c = NULL; -- this deletes field c
		- To assign NULL value to a field use VALUE.  Example: SET a.b.c VALUE = NULL -- this set to value of c to NULL
		- To test if a field exists or not, use FIELDTYPE.  Example: IF FIELDTYPE(a.b.c) IS NULL THEN -- if the field c does not exist FIELDTYPE will return NULL
			Note: IF a.b.c IS NULL THEN -- this does not test if field c exists but the value of field c
	

* PROPAGATE statement can be used to:
	– Generate multiple output messages
	– Route message to different output terminals: Out, Out1, Out2, Out3, Out4

* Schemas:  A broker schema is a symbol space that defines the scope of uniqueness of the names of resources defined within it. The resources include message flows and other optional resources such as ESQL files, Java files, and mapping files.  A broker schema is implemented as a folder, or subdirectory, within the project, and provides organization within that project. You can also use project references to spread the scope of a single broker schema across multiple projects.  You can then reference any ESQL object from any resource within the same broker schema whether in the same project in any referenced project. If you want to reuse a function or procedure globally in an ESQL file, either provide a fully-qualified reference, or include a PATH statement that defines the path.

* Compute Node: Compute node is associated with only one ESQL module.  Each module contains a function called Main() which is the entry point at which the processing of the node starts.
	* RETURN value controls message propagation to output terminals:
		– RETURN; and RETURN TRUE; : Propagates to OUT terminal
		– RETURN FALSE or UNKNOWN  : Message is not propagated

* Procedures:
	- Subroutines with IN, OUT, and INOUT parameters and optional RETURNS value
	- Can be used recursively
	- Language can be ESQL, Java, or database (stored procedure)
	- Output message trees cannot be accessed directly within the body; they must be passed in as a Reference parameter.

	CREATE PROCEDURE navigate (IN root REFERENCE, INOUT answer CHARACTER)
		LANGUAGE ESQL
		BEGIN
			......
		END;
	
	
* XMLNSC domain:
	* The message tree fields specific types are:
		- XMLNSC.Folder
		- XMLNSC.Field
		- XMLNSC.Attribute

	* ROW type: when a ROW type is assigned to or from a field in a XMLNSC message tree:
		- if a ROW has direct value, it is converted to XMLNSC.Field in the message tree.  All children of ROW are ignored since an XML field is a leaf node and cannot have children.
		- if ROW has no direct value and has children, it is converted to XMLNSC.Folder in the message tree.  All children are converted to XMLNSC.Field if they have a direct value.

	* Typically, the XML parsers (XMLNSC, XMLNS, and XML) do not create null values in the message tree; an empty element or an empty attribute value merely produces an empty string value in the message tree.
		Reference: https://www.ibm.com/support/knowledgecenter/en/SSKM8N_8.0.0/com.ibm.etools.mft.doc/ad21050_.htm
	
* Parsers in output tree:
	- Parsers must be created when not copying to OutputRoot:
		https://www.ibm.com/support/knowledgecenter/en/SSMKHH_9.0.0/com.ibm.scenarios.doc/gop_03/topics/bj60033_.htm
	
	
XPath:
* Dot notation is used to traverse elements in message tree in ESQL. XPath 1.0 is also used to reference and manipulate elements in message tree in nodes such as JavaCompute, XSLTransform, Mapping, Route, Transport Header nodes.  * There is a set of preinstalled set of XPath variables that are commonly used in Broker Development, such as $Body, $Environment, and so on

* Combining both Java and ESQ:
	* MbSQLStatement executes ESQL within JavaCompute node.  Best for accessing databases with ODBC.
	* call static Java methods from ESQL:
		public class WM80 {
			public static void sleep(Long millis) {
				try {
					Thread.sleep(millis.longValue());
				} catch (InterruptedException e) {}
			}
		}
		* Wrap Java method as ESQL procedure:
			CREATE PROCEDURE PauseFlow (IN millisecs INT)
				LANGUAGE JAVA
				EXTERNAL NAME "myPackage.WM80.sleep";
		* Use it like any other procedure in ESQL or map:
			CALL PauseFlow(10000)
			
			
* ESQL Sample Code:
	- construct a new message from input message and database:
		SET OutputRoot.XMLNSC.OrderMsg.OrderItem[]=
			(SELECT I.Nr, D.DESCR AS Description 
				FROM InputBody.OrderMsg.OrderItem[] as I,
					Database.PRODUCTS as D
				WHERE I.Nr = D.PRODNO);
				
				
	- Using 'VALUE' keyword in 'SET' statement:
		- input message: <a><b name="Hello">World</b></a>
		- First statement: SET OutputRoot.XMLNSC.Result.x = InputRoot.XMLNSC.a.b;
			* Result: <Result><x name="Hello">World</x></Result>
		- Second statement: SET OutputRoot.XMLNSC.Result.x VALUE = InputRoot.XMLNSC.a.b;
			* Result: <Result><x>World</x></Result>
			
			
Resources:
=========
https://developer.ibm.com/integration/blog/2014/06/16/new-iib-technical-presentations-from-impact-2014/
