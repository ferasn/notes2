XML entities:
https://www.ibm.com/developerworks/library/x-entities/index.html

There are 5 pre-defined entity references in XML:

&lt;	<	less than
&gt;	>	greater than
&amp;	&	ampersand 
&apos;	'	apostrophe
&quot;	"	quotation mark


Datapower character entities:
http://www-01.ibm.com/support/docview.wss?uid=swg21409699



XPath works on XML object model.

Types of XML Nodes:
element,
attribute, 
text,
processing-instruction, 
comment, 
document
namespace : a namespace node cannot be selected by an XPath expression. However, any namespace declared in an element is considered a child of that element.  In fact it is also considered a child of all elements nested under that element.  So, if you use <copy-of> in XSLT on an element that has namespace declarations or on any of its child elements, the namespace declarations will be copied as well, because they are considered children of the element node. It make sence that XML declaration is considered child of every element in the heirarchy starting from the elements where they are declared becuase their scope cover all these elements.

<?xml version="1.0" encoding="UTF-8"?>
<a d="abc">
	<b>xyz</b>
</a>


* The result of XPath expressions is a set of nodes.  The set could contain one or more nodes.

* Attribute nodes: an attribute node has a parent node but it is not a child of its parent.  This is why if you select the child nodes of an element node that has an attribute, the attribute node will not be in the node set.  to select the attribute node you need to explicity select it /@attribName

1- select document node (top-most level)
/

2- select element nodes (in this case would return the root element of XML document)
/*

3- select attribute nodes (in this case it would not return any node, because document node cannot have attributes as children)
/@*

4- select all nodes except attribute nodes (child nodes of the document node.  Could contain: the XML declaration, text node representing new line, the root element)
/node()



If XML elements have a namesapce, you need to prefix the elements name with namespace:
To do this you need to register a prefix with the namespace and use that prefix:
/x:List/x:Fields/x:Field

You can register (or declare) a namespace prefix if you are using an XPath engine programmatically (use the engine APIs to register the prefix) or in XSLT.  
However, in other cases where there is no way to register a prefix (like in Eclipse XML tools Xpath View or Datapower debug probe filter),
you can refer to elements by name this way:

/*[name()='List']/*[name()='Fields']/*[name()='Field']

Reference: https://stackoverflow.com/questions/5239685/xml-namespace-breaking-my-xpath


============

<?xml version="1.0" encoding="UTF-8"?>
<t:a xmlns:t="asdfasdf" ddd="abc">
<b>abc</b>
</t:a>

If an XML element has a namespace prefix, the prefix is part of the element name (e.g. t:a ).  
To find an element with namespace prefix:
/*[name() = "t:a"]

If you define a namespace prefix "x" with URI "asdfasdf" you can navigate to the element node using:
/x:a
This will return the root element "a" but still its name is "t:a"

If you want to match with element name but you do not know the prefix in XML document, you can use "local-name()":
/*[local-name() = "a"]

XPath 2.0:
It is possible to place asterisk in place of the namespace to match any namespace:
/*:a


XML, XPath, XSLT summary notes:
https://www.scss.tcd.ie/Owen.Conlan/4d2/



Eclipse plugins for XPath evaluation:
1- eclipse WTP tools has XPath view.  It only support XPath 1.0.  Also, you cannot declare namespace prefixes with it.
2- https://marketplace.eclipse.org/content/eclipse-xpath-evaluation-plugin:  supports XPath 2.0 and it allowes you to register namespace prefixes