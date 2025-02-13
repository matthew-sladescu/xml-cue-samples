# XML to CUE Encoding Proposal

## Problem
Many users would benefit from using CUE with their XML files, however CUE does not have an encoding that supports XML.

## Purpose of this document
This document puts forward a proposal for an XML to CUE mapping that can be used to add an XML encoding to CUE.

## Objectives
This mapping in this proposal aims to:

- allow users to write CUE constraints against XML files (XML Support)
- provide a mapping that makes it easy to understand what a CUE constraint refers to relative to the XML it is describing (Readability)

## Proposed Mapping

The proposed mapping follows a convention that is inspired by the [Badgerfish convention](http://www.sklar.com/badgerfish/), with devations for compatibility with CUE and increased readability.

This new mapping will be called `rcXML` and follows the following rules:

1. Each XML element maps to a CUE struct, with the struct key being the element name.
2. Each nested XML element becomes a nested CUE struct.
3. Each XML attribute maps to a CUE struct property, where:
   - the property key is the attribute name prefixed with `$`, and
   - the property belongs to the struct that is mapped from the XML attribute's parent element.
4. The text content of an XML element maps to a CUE property keyed as `$`, where that property belongs to the struct that is mapped from the XML content's parent element.
5. Multiple XML elements at the same level map to multiple CUE structs forming part of a CUE list at that level.
6. Each XML attribute that defines a namespace maps to a CUE struct property in the same way that other XML attributes are mapped.
7. When an XML element name contains a namespace label, the corresponding CUE struct property will be keyed by the same name.
8. Values of XML attributes and elements will be typed in the corresponding CUE value when the type is infered to be either int, float, boolean, null, or string.

The examples below illustrate each of these rules:

### 1. Elements

*1.xml*
```
<note>
	<to>Tove</to>
	<from>Jani</from>
	<heading>Reminder</heading>
	<body>Don't forget me this weekend!</body>
</note>
```

*1.cue*
```
{
	note: {
		to: {
			$$content: "Tove"
		}
		from: {
			$$content: "Jani"
		}
		heading: {
			$$content: "Reminder"
		}
		body: {
			$$content: "Don't forget me this weekend!"
		}
	}
}
```

## 

Shows the mapping for simple XML elements and content.



## 2.xml -> 2.cue : Attribute Mapping

Shows how **attributes** are mapped using the "alpha" attribute as an example.

*2.xml*
```
<note alpha="abcd">
	<to>Tove</to>
	<from>Jani</from>
	<heading>Reminder</heading>
	<body>Don't forget me this weekend!</body>
</note>
```

*2.cue*
```
{
	note: {
		body:    "Don't forget me this weekend!"
		$alpha:  "abcd"
		to:      "Tove"
		from:    "Jani"
		heading: "Reminder"
	}
}
```


## 3.xml -> 3.cue : Attribute & Element with same name

Shows what the mapping looks like when an element and an attribute have the same name.

*3.xml*
```
<note alpha="abcd">
	<to>Tove</to>
	<from>Jani</from>
	<heading>Reminder</heading>
	<body>Don't forget me this weekend!</body>
	<alpha>efgh</alpha>
</note>
```

*3.cue*
```
{
	note: {
		alpha:   "efgh"
		$alpha:  "abcd"
		to:      "Tove"
		from:    "Jani"
		heading: "Reminder"
		body:    "Don't forget me this weekend!"
	}
}
```

## 4.xml -> 4.cue : Mapping for content when an attribute exists

Shows how the **content** for an element is modeled when an attribute is present.

*4.xml*
```
<note alpha="abcd">
	hello
</note>
```

*4.cue*
```
{
	note: {
		$$content: "hello"
		$alpha:    "abcd"
	}
}
```

## 5.xml -> 5.cue : Nested Element

Shows the mapping for a nested **note** element within a **notes** element.

*5.xml*
```
<notes>
	<note alpha="abcd">hello</note>
</notes>
```

*5.cue*
```
{
	notes: {
		note: {
			$$content: "hello"
			$alpha:    "abcd"
		}
	}
}
```

## 6.xml -> 6.cue : Collection with more than 1 element

Shows how an xml **collection with more than 1 element** is mapped.

*6.xml*
```
<notes>
	<note alpha="abcd">hello</note>
	<note alpha="abcdef">goodbye</note>
</notes>
```

*6.cue*
```
{
	notes: {
		note: [{
			$$content: "hello"
			$alpha:    "abcd"
		}, {
			$$content: "goodbye"
			$alpha:    "abcdef"
		}]
	}
}
```


## 7.xml -> 7.cue : Mixed text/sub-element content within an element

Shows how **mixed content** within an element is represented. ie: An element that has both text content and nested elements.

*7.xml*
```
<notes>
	<note alpha="abcd">hello</note>
	<note alpha="abcdef">goodbye</note>
	yay
	tfd
</notes>
```

*7.cue*
```
{
	notes: {
		$$content: """
			yay
			tfd
			"""
		note: [{
			$$content: "hello"
			$alpha:    "abcd"
		}, {
			$$content: "goodbye"
			$alpha:    "abcdef"
		}]
	}
}
```


## 8.xml -> 8.cue : Interleaving Element types

Shows how a collection is represented when it has an interleaved element of a different type in the middle.

*8.xml*
```
<notes>
	<note alpha="abcd">hello</note>
	<note alpha="abcdef">goodbye</note>
	<book>mybook</book>
	<note alpha="ab">goodbye</note>
	<note>direct</note>
</notes>
```

*8.cue*
```
{
	notes: {
		note: [{
			$$content: "hello"
			$alpha:    "abcd"
		}, {
			$$content: "goodbye"
			$alpha:    "abcdef"
		}, {
			$$content: "goodbye"
			$alpha:    "ab"
		}, {
			$$content: "direct"
		}]
		book: {
			$$content: "direct"
		}
	}
}
```


## 9.xml -> 9.cue : Namespaces

Shows how namespaces and associated member elements are modeled

*9.xml*
```
<h:table xmlns:h="http://www.w3.org/TR/html4/">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
  </h:tr>
</h:table>
```

*9.cue*
```
{
	"h:table": {
		"$xmlns:h": "http://www.w3.org/TR/html4/"
		"h:tr": {
			"h:td": [
				{ $$content: "Apples"}, 
				{ $$content: "Bananas"}
			]
		}
	}
}
```


## 10.xml -> 10.cue : Mixed Namespaces

Shows how XML documents where multiple namespaces are declared are represented, and how members elements of these different namespaces are represented in CUE.

*10.xml*
```
<h:table xmlns:h="http://www.w3.org/TR/html4/" xmlns:r="d">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
    <r:blah>e3r</r:blah>
  </h:tr>
</h:table>
```

*10.cue*
```
{
	"h:table": {
		"$xmlns:h": "http://www.w3.org/TR/html4/"
		"$xmlns:r": "d"
		"h:tr": {
			"h:td": [
				{ $$content: "Apples"}, 
				{ $$content: "Bananas"}
			]
			"r:blah": { $$content: "e3r" }
		}
	}
}
```


## 11.xml -> 11.cue : Elements with same name but different namespaces

Shows how elements with the same name, but under different namespaces are represented.
In the example below these are ```h:td``` and ```r:td```

*11.xml*
```
<h:table xmlns:h="http://www.w3.org/TR/html4/" xmlns:r="d">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
    <r:td>e3r</r:td>
  </h:tr>
</h:table>
```

*11.cue*
```
{
	"h:table": {
		"$xmlns:h": "http://www.w3.org/TR/html4/"
		"$xmlns:r": "d"
		"h:tr": {
			"h:td": [
				{ $$content: "Apples"}, 
				{ $$content :"Bananas"}
			]
			"r:td": { $$content: "e3r"}
		}
	}
}
```


## 12.xml -> 12.cue : Collection of elements, where elements have optional properties

Shows how elements of a collection (in this case of type ```book```) having optional properties (here the optional ```volume``` property) are represented in CUE.

*12.xml*
```
<books>
    <book>
        <title>title</title>
        <author>John Doe</author>
    </book>
    <book>
        <title>title2</title>
        <author>Jane Doe</author>
    </book>
    <book>
        <title>Lord of the rings</title>
        <author>JRR Tolkien</author>
        <volume>
            <title>Fellowship</title>
            <author>JRR Tolkien</author>
        </volume>
        <volume>
            <title>Two Towers</title>
            <author>JRR Tolkien</author>
        </volume>
        <volume>
            <title>Return of the King</title>
            <author>JRR Tolkien</author>
        </volume>
    </book>
</books>
```

*12.cue*
```
{
	books: {
		book: [{
			title:  "title"
			author: "John Doe"
		}, {
			title:  "title2"
			author: "Jane Doe"
		}, {
			title:  "Lord of the rings"
			author: "JRR Tolkien"
			volume: [{
				title:  "Fellowship"
				author: "JRR Tolkien"
			}, {
				title:  "Two Towers"
				author: "JRR Tolkien"
			}, {
				title:  "Return of the King"
				author: "JRR Tolkien"
			}]
		}]
	}
}
```

## 13.xml -> 13.cue : Representing types 

Shows examples of how types are modeled in CUE when element contents look like they have an implicit type.

*13.xml*
```
<data>
	<int>54</int>
	<float>43.12</float>
	<string>hello</string>
	<bool1>TRUE</bool1>
	<bool2>true</bool2>
</data>
```

*13.cue*
```
{
	data: {
		"int":    54
		"float":  43.12
		"string": "hello"
		bool1:  true
		bool2:  true
	}
}
```
