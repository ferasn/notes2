XPath types:
* Result Tree fragements: Type introduced by XSLT.  It is created with xslt variables.  
A variable may be bound to a result tree fragment instead of one of the four basic XPath data-types (string, number, boolean, node-set).
A result tree fragement type is like a node-set that contains just a single root node except that you cannot write XPath
location path expressions on values of this type (/, //, and [] operators are not permitted).
Expressions can only return values of type result tree fragment by referencing variables of type result tree fragment or calling extension functions that return a result tree fragment or getting a system property whose value is a result tree fragment.

String and Boolean values of types:
* Result Tree fragement;
  - Boolean value: always true.  Boolean for types that have nodes is true if there is at least one node. For result tree fragement
  values, there is always a node (root node).  Contrast this with node-set where you can have a value of node-set that is empty.
  - String value: The concatenation of all decendant text nodes that are not white space. White space text nodes are excluded.
  Contrast this with node-set where all text nodes are concatenated includeing white space text nodes. Note: this is tested in Xalan XSLT engine
  by outputing the result of <xsl:value-of select="$n"> where $n is a variable with result tree fragement value.
  - Number: ... test this in Xalan: provide tree that contain text nodes that are number (also test with numbers in multiple text
  nodes so that they become concatenated) and text nodes that are not numbers.
  
* Node-set:
  - Boolean value: if node-set is empty: false, otherwise: true.
  - String value: ....
  

XPath predicate evaluation:
- Type of expression:
  * Boolean:
  * Number
  * Others: apply Boolean function
  
------------------------------------------
* <xsl:value-of>
  - Boolean:
  - String
  - Node-set: (apply string function for first node) get string-value of the first node only. (Conctenation of all text nodes that are decedendants of the first node)
  - Result tree fragement: (apply string function to the root node) get string-value of the root node. (Concatenation of all decedendant text nodes that are not white spaces)

* <xsl:copy-of>
XSLT 1.0 section 11.3

Names of variables and parameters:
===================================
XSLT 1.0 specs section 11

Values of Variables and Parameters:
===================================
XSLT 1.0 Specs section 11.2
- difference between <xsl:variable name="x" select="1"/> and <xsl:variable name="x" select="'1'"/>
- <xsl:variable name="n">2</xsl:variable> : this binds the variable 'n' to a result tree fragement value that cotains one 
text node.
- 
<xsl:variable name="n">2</xsl:variable>
<xsl:value-of select="item[$n]"/>

The expression predicate in "item[$n]" evaluation to a value of type reslut tree fragement. Since it is not boolean or number, the Boolean
function is applied to it, this always produces true. This results in all 'item' elements to be selected in the resulted node-set.
<xsl:value-of> gets the string-value of the first node in the node-set.
So, rather than getting the string-value of the second element, we get the string-value of the first one.

To rectify this, we can force the result tree fragement to convert to number by putting it in an expression that compares it
againts a number:
<xsl:value-of select="item[position()=$n]"/>
In this case, the select expression will result in a node-set that contains the second element only.




Interesting use of Result Tree Nodes:
=====================================
- create a tree to hold values to be used later in compuation:
https://www.xml.com/pub/a/2003/07/16/nodeset.html



* Put the example that converts XML text nodes from escaped characterts to resolved characters.




