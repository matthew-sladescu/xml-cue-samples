# XML to CUE Encoding Proposal

## Problem
Many users would benefit from using CUE with their XML files, however CUE does not have an encoding that supports XML.

## Purpose of this document
This document puts forward a proposal for an XML to CUE mapping that can be used to add an XML encoding to CUE.

## Objectives
This mapping in this proposal aims to:

- allow users to write CUE constraints against XML files (XML Support)
- provide a mapping that makes it easy to understand what a CUE constraint refers to relative to the XML it is describing (Readability)
- allow one to go back from the mapped json to XML. Note: Element / Attribute order preservation at the same level is not required.

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

The XML `note` element below maps to the note struct in CUE.

*xml*
```
<note>
</note>
```

*cue*
```
{
	note: { }
}
```

### 2. Nested Elements

Nesting an XML `to` element to the `note` element from the first example results in a nested CUE `to` struct inside the `note` struct.

*xml*
```
<note>
	<to></to>
</note>
```

*cue*
```
{
	note: {
		to: {}
	}
}
```

### 3. Attributes

The `alpha` attribute of the `note` element in XML below maps to the `$` prefixed `$alpha` property of the `note` struct in CUE.

*xml*
```
<note alpha="abcd">
</note>
```

*cue*
```
{
	note: {
		$alpha: "abcd"
	}
}
```

### 4. Content

The content of the `note` XML element below maps to the value of the `$` property of the `note` struct in CUE.

*xml*
```
<note alpha="abcd">
	hello
</note>
```

*cue*
```
{
	note: {
		$alpha: "abcd"
		$:      "hello"
	}
}
```

### 5. XML Lists

The multiple XML `note` elements at the same level map to a list of `note` structs in CUE.

*xml*
```
<notes>
	<note alpha="abcd">hello</note>
	<note alpha="abcdef">goodbye</note>
</notes>
```

*cue*
```
{
	notes: {
		note: [{
			$alpha: "abcd"
			$:      "hello"
		}, {
			$alpha: "abcdef"
			$:      "goodbye"
		}]
	}
}
```

### 6 and 7. Namespace Definitions and References

The `h` and `r` XML namespace definitions declared in the `table` XML element are declared as properties of the `h:table` strcut in CUE.
Note how the namspace prefixed XML element names like `h:table`, `h:tr`, `h:td` and `r:blah` carry across to the key names of their corresponding CUE structs.

*xml*
```
<h:table xmlns:h="http://www.w3.org/TR/html4/" xmlns:r="d">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
    <r:blah>e3r</r:blah>
  </h:tr>
</h:table>
```

*cue*
```
{
	"h:table": {
		"$xmlns:h": "http://www.w3.org/TR/html4/"
		"$xmlns:r": "d"
		"h:tr": {
			"h:td": [{
				$: "Apples"
			}, {
				$: "Bananas"
			}]
			"r:blah": {
				$: "e3r"
			}
		}
	}
}
```
### 8. Typing

Note how the int, float, string, and boolean types are inferred in the CUE below from the values in the XML element content.

*xml*
```
<data>
	<int>54</int>
	<float>43.12</float>
	<string>hello</string>
	<bool1>TRUE</bool1>
	<bool2>true</bool2>
</data>
```

*cue*
```
{
	data: {
		int: {
			$: 54
		}
		float: {
			$: 43.12
		}
		string: {
			$: "hello"
		}
		bool1: {
			$: true
		}
		bool2: {
			$: true
		}
	}
}
```

## Example CUE constraints definitions against an XML file

Given an XML file with a set of note elements and a book element like the one shown below, we could write a CUE schema as shown below"

XML
```
<notes>
	<note alpha="abcd">hello</note>
	<quantity>5</quantity>
</notes>
```

CUE constraints
```
notes: {
	note:{
 		$alpha: string
   		$: string
  	}
   	quantity:$:int
}
```
## Alternative Conventions Considered

Although no current mappings between XML and CUE exist, there are a number of existing mappsings that take XML to JSON, which we take inspiration from.

### Parker and Spark Conventions

The Parker and Spark conventions use a very simplistic model where XML attributes are ignored.
We wish to maintain attribute information when mapping from XML to CUE, however we take inspiration from this convention of mapping elements to object properties.

### Badgerfish 

The Badgerfish convention maps elements, attributes, and content from XML to JSON. We follow the majority of the rules in the badgerfish convention, described [here](http://www.sklar.com/badgerfish/), with the modifications below to allow for ampping to CUE and for increased readability:

- XML attributes map to CUE properties starting with a "$" prefix instead of a "@" prefix, given "@" is already reserved in CUE for CUE attributes.
- The mapping proposed in this document improves readability by not recursively defining namespaces in nested objects, but rather only defining namespaces at the same level where they are declared in XML. 

To illustrate how rcXML simplifies the mapping, we provide the example below (taken from [here](http://www.sklar.com/badgerfish/)):

*xml*
```
<alice xmlns="http://some-namespace" xmlns:charlie="http://some-other-namespace"> 
    <bob>david</bob> 
    <charlie:edgar>frank</charlie:edgar> 
</alice>
```

*Badgerfish*
```
{ 
    alice : 
    { 
        bob : 
        { 
            $ : "david" , 
            "@xmlns" : 
            {
                charlie : "http://some-other-namespace" , 
                $ : "http://some-namespace"
            } 
        } , 
        "charlie:edgar" : 
        { 
            $ : "frank" , 
            "@xmlns" : 
            {
                "charlie":"http://some-other-namespace", 
                "$" : "http://some-namespace"
            } 
        }, 
        "@xmlns" : 
        { 
            charlie : "http://some-other-namespace", 
            $ : "http://some-namespace"
        } 
    } 
}
```

*rcXML*
```
{ 
    alice : 
    { 
        "$xmlns:charlie" : "http://some-other-namespace",
        $xmlns : "http://some-namespace",
        bob : 
        { 
            $ : "david" 
        }, 
        "charlie:edgar" : 
        { 
            $ : "frank" 
        }
    } 
}
```

### GData

The [GData convention](https://developers.google.com/gdata/docs/json?csw=1) is similar to Badgerfish, however makes no distincion between identifiers used for elements and those used for attributes. 

Unlike the Badferfish convention, if one were to use this convention to map from XML to CUE, it would mean that it becomes ambiguous whether you are referring to an attribute or to an element when writing a CUE constraint. Further, the rules specified [here](https://developers.google.com/gdata/docs/json?csw=1) do not specify what happens when there is a collision between an element name and an attribute name.

### Abdera 

This convention is similar to the GData convention, however, it uses a "children" array and "attributes" object when both nested XML elements and attributes are mentioned. Having to mention these array and object labels along with associated indexes to access children makes it too unwieldy to write CUE constraints with. For example:

*XML*
```
<note myAttr="attrVal" attrTwo="two">
	<point>example</point>
	<sample>other</other>
</note>
```

*CUE*
```
{
	note: {
		attributes:  {
			myAttr: "attrVal"
			attrTwo: "example"
		}
		children: [ {
			$
		}]
	}		

}
```

### JsonML 

Short for JSON Markup Language, this convention makes heavy use of arrays to ensure an order-preserving mapping, where each element maps to an array entry, and each attribute also maps to an array entry.

Having to use integer indexes when writing a CUE constraint rather than just simply use the element and attribute identifiers found in the XML makes this mapping too unwieldy to use.



