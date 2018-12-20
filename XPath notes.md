XPath works on XML object model.

Types of XML Nodes:
element,
attribute, 
text,
processing-instruction, 
comment, 
document

XPath expressions return a list of nodes.  Depending on the expression, there could be one node only in the list.

1- select document node (top-most level)
/

2- select element nodes (in this case would return one node that is the opening element of the XML document)
/*

3- select attribute nodes (in this case it would not return any node, because document node cannot have child attributes)
/@*

4- select all nodes except attribute nodes (in this case would return one node that is the opening element of the XML document)
/node()



If XML elements have a namesapce, you need to prefix the elements name with namespace:
To do this you need to register a prefix with the namespace and use that prefix:
/x:List/x:Fields/x:Field

You can register (or declare) a namespace prefix if you are using an XPath engine programmatically (use the engine APIs to register the prefix) or in XSLT.  
However, in other cases where there is no way to register a prefix (like in Eclipse XML tools Xpath View or Datapower debug probe filter),
you can refer to elements by name this way:

/*[name()='List']/*[name()='Fields']/*[name()='Field']

Reference: https://stackoverflow.com/questions/5239685/xml-namespace-breaking-my-xpath
