XSLT template precedence:
https://stackoverflow.com/questions/5709581/xsl-template-precedence

XSLT strip spaces:
https://stackoverflow.com/questions/1134318/xslt-xslstrip-space-does-not-work

How XSLT works:
http://lenzconsulting.com/how-xslt-works/


<copy> : copy current node and any namespace node declared in it.  No child is copied (no attribute or ther child elements).  The namespace nodes are copied because the element could be in that namespace so the declarations need to be copied with it.
<copy-of> : copy selected node and all its children (child attributes are included)

An XSLT is processed in recursively through <xsl:apply-templates> to select a set of nodes and then finding a matching <xsl:template> for each node and process that template.  If the template contains <xsl:apply-templates> the same cycle repeats therefore recurseivly.  The set of nodes to select from are from the children of the current context node. the "select" attributes provides the filter to select from this set. 

In the absence of a select attribute, the <xsl:apply-templates> instruction processes all of the children of the current node (emitting select attribute is the same as providing it with select all type nodes: <xsl:apply-templates select="node()"> ).  Please note that this does not include attribute nodes because attribute nodes are not children of element nodes and to include them they have to be selected explicitly  <xsl:apply-templates select="node() | @*">

Note: <xsl:apply-templates> is done implicitly first and the set of nodes selected contains only the Document node.  Therefore, usually first template match is <xsl:template match="/"> to match the document node.


* Built-in templates:
=====================
There is a built-in template rule to allow recursive processing to continue in the absence of a successful pattern match by an explicit template rule in the stylesheet.  This template rule applies to both element nodes and the root node. The following shows the equivalent of the built-in template rule:

<xsl:template match="*|/">
  <xsl:apply-templates/>
</xsl:template>

In other words, if an <xsl:apply-templates> selects a set of nodes, and there are one or more nodes that have no templates matching them, the built-in templates will be applied.  For document node and element nodes, it means recursively select all child nodes and apply templates to them.

There is also a built-in template rule for each mode, which allows recursive processing to continue in the same mode in the absence of a successful pattern match by an explicit template rule in the stylesheet. This template rule applies to both element nodes and the root node. The following shows the equivalent of the built-in template rule for mode m.

<xsl:template match="*|/" mode="m">
  <xsl:apply-templates mode="m"/>
</xsl:template>

There is also a built-in template rule for text and attribute nodes that copies text through:

<xsl:template match="text()|@*">
  <xsl:value-of select="."/>
</xsl:template>
The built-in template rule for processing instructions and comments is to do nothing.

<xsl:template match="processing-instruction()|comment()"/>
The built-in template rule for namespace nodes is also to do nothing. There is no pattern that can match a namespace node; so, the built-in template rule is the only template rule that is applied for namespace nodes.

The built-in template rules are treated as if they were imported implicitly before the stylesheet and so have lower import precedence than all other template rules. Thus, the author can override a built-in template rule by including an explicit template rule.




* Effect of built-in templates on transformation output:
========================================================
Consider this input document:
<?xml version="1.0" ?>
<a d="abc">
	<b>xyz</b>
</a>

Root element 'a' has four children:
1- Text node: new line + spaces
2- Element node "b"
3- Text node: new line
The attribute node "d" is not listed in the child nodes list becuase attribute nodes are not children of element nodes.

and element 'b' has one child:
1- text node: "xyz"

Transforming it with this XSLT:
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:template match="/">
		<xsl:apply-templates select="*" />
	</xsl:template>

	<xsl:template match="*">
		<xsl:copy>
			<xsl:apply-templates />
		</xsl:copy>
	</xsl:template>
</xsl:stylesheet>

The <xsl:apply-templates /> inside the <xsl:copy> element selects all child nodes. In the first iteration it is applied to the children of element node "a" and this will select the four child nodes of "a".  The available templates only match element node types <xsl:template match="*"> and document node <xsl:template match="/">.  So, one would expect to find only element "a" with child element "b".  However the resulted XML is as follows:

<?xml version="1.0" encoding="UTF-8"?><a>
	<b>xyz</b>
</a>

All child text nodes are included in the output XML.  This is because text nodes that do not have a matching template in XSLT will match the built-in template for text and attribute nodes <xsl:template match="text()|@*">.  This template copies the text to the output XML.  One interesting observation is that the output XML does not have the attribute "d" from original XML, although the built-in template <xsl:template match="text()|@*"> matches attribute nodes.  This is becuase the node set resulted from <xsl:apply-templates /> includes all child nodes and attribute nodes are not considered children and only those nodes in the set will be matched against avaialable templates.  To include attribute nodes, the <xsl:apply-templates /> has to be changed to <xsl:apply-templates select = "node() | @*" />

To prevent text nodes from appearing in the output XML, one can override the built-in template with custom template that matches text nodes but doe not do anything:  <xsl:template match="text()" /> 

Adding this template to XSLT will result in the following output XML:

<?xml version="1.0" encoding="UTF-8"?><a><b/></a>

Also, another way to do this is to limit the nodes in the nodes set to element nodes.  So, <xsl:apply-templates /> will change to <xsl:apply-templates select="*"/>.  This change will result in the same output XML:

<?xml version="1.0" encoding="UTF-8"?><a><b/></a>

The above XSLT can also be simplified to remove the tempalte that matches document node, since there is a built-in template that matches document node and applies templates to its children.  This XSLT will produce exactly the same XML output:

<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	
	<xsl:template match="*">
		<xsl:copy>
			<xsl:apply-templates />
		</xsl:copy>
	</xsl:template>
</xsl:stylesheet>

