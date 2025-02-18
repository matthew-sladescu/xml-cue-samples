# XML to CUE Encoding Proposal

## Problem
Many users would benefit from using CUE with their XML files, however CUE does not currently have an encoding that supports XML.

## Purpose of this document
This document puts forward a proposal for an XML to CUE mapping called `cXML` that can be used to add an XML encoding to CUE. Given there are many approaches for mapping from XML to CUE, this could be one of many XML encodings provided by CUE.

## Objectives
The mapping in this proposal aims to:

- allow users to write CUE constraints against XML files (XML Support)
- provide a mapping that makes it easy to understand what a CUE constraint refers to relative to the XML it is describing (Readability)
- allow one to go back from the mapped CUE to XML.

## Non-Objective
- Order preservation at the same level.

## Proposed Mapping

The proposed mapping follows a convention that is inspired by the [Badgerfish convention](http://www.sklar.com/badgerfish/), with deviations for compatibility with CUE and increased readability.

This new mapping will be called `cXML` and follows the rules below:

1. Each XML element maps to a CUE struct, with the struct key being the element name.
2. Each nested XML element becomes a nested CUE struct.
3. Each XML attribute maps to a CUE struct property, where:
   - the property key is the attribute name prefixed with `$`, and
   - the property belongs to the struct that is mapped from the XML attribute's parent element.
4. The text content of an XML element maps to a CUE property keyed as `$`, where that property belongs to the struct that is mapped from the XML content's parent element.
5. Multiple XML elements at the same level map to multiple CUE structs forming part of a CUE list at that level.
6. Each XML attribute that defines a namespace maps to a CUE struct property in the same way that other XML attributes are mapped.
7. When an XML element name includes a namespace label as a prefix, the corresponding CUE struct property will be keyed by the same name and include the same prefix.
8. Values of XML attributes and elements will be typed in the corresponding CUE value when the type is inferred to be either int, float, boolean, null, or string.
9. When text is interspersed at multiple locations between the sub-element(s) of an XML element, we map this text content to a CUE list as follows. Within an XML element:
	- the order with which non white-space text appears determines the 0-based **index** of each non white-space text
	- if there are no sub-elements, then the text in that element is mapped to the value of `$`.
	- if text is found at more than one index, then the value of `$` is a CUE list. This CUE list contains an indexed collection of text found within that XML element.

### Sample CUE constraints for XML using `cXML`

Using the rules above, one would be able to write CUE constraints for XML as shown below:

Given an XML file with a `note` element and a `book` element, we could write a CUE schema to define types as shown below:

*XML*
```
<notes>
	<note alpha="abcd">hello</note>
	<quantity>5</quantity>
</notes>
```

*CUE constraints*
```
//the alpha attribute of notes:note should be a string
notes: note: $alpha: string

//the text within the notes:quantity element should be an integer
notes: quantity: $: int
```

## Mapping examples

The examples below illustrate each of the mapping rules defined above:

### 1. Elements

The XML `note` element below maps to the `note` struct in CUE.

*XML*
```
<note>
</note>
```

*CUE*
```
{
	note: { }
}
```

### 2. Nested Elements

Nesting an XML `to` element to the `note` element from the first example results in a nested CUE `to` struct inside the `note` struct.

*XML*
```
<note>
	<to></to>
</note>
```

*CUE*
```
{
	note: {
		to: {}
	}
}
```

### 3. Attributes

The `alpha` attribute of the `note` element in XML below maps to the `$` prefixed `$alpha` property of the `note` struct in CUE.

*XML*
```
<note alpha="abcd">
</note>
```

*CUE*
```
{
	note: {
		$alpha: "abcd"
	}
}
```

### 4. Content

The content of the `note` XML element below maps to the value of the `$` property of the `note` struct in CUE.

*XML*
```
<note alpha="abcd">
	hello
</note>
```

*CUE*
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

*XML*
```
<notes>
	<note alpha="abcd">hello</note>
	<note alpha="abcdef">goodbye</note>
</notes>
```

*CUE*
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

The `h` and `r` XML namespace definitions declared in the `table` XML element are declared as properties of the `h:table` struct in CUE.
Note how the namespace prefixed XML element names like `h:table`, `h:tr`, `h:td` and `r:blah` carry across to the key names of their corresponding CUE structs.

*XML*
```
<h:table xmlns:h="http://www.w3.org/TR/html4/" xmlns:r="d">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
    <r:blah>e3r</r:blah>
  </h:tr>
</h:table>
```

*CUE*
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

*XML*
```
<data>
	<int>54</int>
	<float>43.12</float>
	<string>hello</string>
	<bool1>TRUE</bool1>
	<bool2>true</bool2>
</data>
```

*CUE*
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

### 9. Interspersed text between different sub-elements

When there is text interspersed between different sub-elements, as described in mapping rule 9, then it can be mapped as shown in the example below:

*XML*
```
<notes>
	<note alpha="x">hi</hello>
	textA
	<note alpha="abcd">hello</note>
	textB
</notes>
```
maps to:

*CUE*
```
{
	notes: {
 		$: ["""
	textA""", """
	textB"""],
 		note: {
   			$alpha: "abcd"
      			$: "hello"
	 	}
   	}
}
```

In the above example, you could write constraints in CUE for the text segments as follows:

```
notes:$:[=~"^[[:space:]]*textA$", =~"^[[:space:]]*textB$"]
```


## Alternative Conventions Considered

Although no known conventions exist to map from XML to CUE, there are a number of known mappings that take XML to JSON, which we can take inspiration from.

### Parker and Spark Conventions

The Parker and Spark conventions use a very simplistic model where XML elements are mapped to object properties, and attributes are ignored.

We wish to maintain attribute information so we cannot use these mapping conventions as a whole.

### Badgerfish 

The Badgerfish convention maps elements, attributes, and content from XML to JSON. We follow the majority of the rules in the Badgerfish convention, described [here](http://www.sklar.com/badgerfish/), with the modifications below to allow for mapping to CUE and for increased readability:

- XML attributes map to CUE properties starting with a `$` prefix instead of an `@` prefix, given `@` is already reserved in CUE for CUE attributes.
  
- The mapping proposed in this document improves readability by not recursively defining namespaces in nested objects, but rather only defining namespaces at the same level where they are declared in XML. 

To illustrate how cXML simplifies the mapping, we provide the example below (taken from [here](http://www.sklar.com/badgerfish/)):

*XML*
```
<alice xmlns="http://some-namespace" xmlns:charlie="http://some-other-namespace"> 
    <bob>david</bob> 
    <charlie:edgar>frank</charlie:edgar> 
</alice>
```

*Badgerfish*
```
{ 
	alice: {
		bob: {
			$: "david"
			"@xmlns": {
				charlie: "http://some-other-namespace"
				$:       "http://some-namespace"
			}
		}
		"charlie:edgar": {
			$: "frank"
			"@xmlns": {
				charlie: "http://some-other-namespace"
				$:       "http://some-namespace"
			}
		}
		"@xmlns": {
			charlie: "http://some-other-namespace"
			$:       "http://some-namespace"
		}
	}
}
```

*cXML*
```
{ 
	alice: {
		"$xmlns:charlie": "http://some-other-namespace"
		$xmlns:           "http://some-namespace"
		bob: {
			$: "david"
		}
		"charlie:edgar": {
			$: "frank"
		}
	}
}
```

We also extend Badgerfish by adding the 9th mapping rule, which allows us to add CUE constraints around specific text elements where an element has text interspersed between sub-elements.

### GData

The [GData convention](https://developers.google.com/gdata/docs/json?csw=1) is similar to Badgerfish, however makes no distinction between identifiers used for elements and those used for attributes. 

Unlike the Badgerfish convention, if one were to use this convention to map from XML to CUE, it would mean that it becomes ambiguous whether you are referring to an attribute or to an element when writing a CUE constraint. Further, it is not clear from the rules specified [here](https://developers.google.com/gdata/docs/json?csw=1) what happens when there is a collision between an element name and an attribute name.

### Abdera and Cobra

[These conventions](https://readthedocs.org/projects/xmljson/downloads/pdf/stable/) are similar to the GData convention, however, they use a "children" array and "attributes" object when both nested XML elements and attributes are mentioned. Having to mention `children` and/or `attributes` in CUE constraints increases verbosity, which goes against the readability objective of this paper. To illustrate this with an example for Abdera:

*XML*
```
<note myAttr="attrVal" attrTwo="two">
	<point>example</point>
	<sample>other</other>
</note>
```
would map to:

*CUE*
```
{
	note: {
		attributes:  {
			myAttr: "attrVal"
			attrTwo: "two"
		}
		children: [ {
			point: "example"
			sample: "other"
		}]
	}		

}
```


### JsonML 

Short for JSON Markup Language, [this convention](http://www.jsonml.org/xml/) makes heavy use of arrays to ensure an order-preserving mapping, where each element maps to an array entry, and each attribute also maps to an array entry. An example mapping is shown [here](https://wiki.open311.org/JSON_and_XML_Conversion/).

Having to work out (count) integer indexes when writing a CUE constraint rather than just simply using the element and attribute identifiers found in the XML makes this mapping too unwieldy to use.


## Testing Plan

The XML to CUE mapping scenarios required are covered by the examples described [here](https://github.com/matthew-sladescu/xml-cue-samples/tree/XMLEncodingProposal). We will consider the solution complete once it can both decode and encode the examples shown there, along with any other test cases requested by the CUE maintainer team.

## Deployment Plan

We aim to release the first versions of the `cXML` encoding as an "experimental" feature of CUE, which can be switched on via existing `CUE_EXPERIMENT` flag. New releases can bring stability improvements to this encoding, with the encoding coming out of the "experimental" phase once the maintainers have deemed this feature stable. Examples of other experimental features are the embedding and topological sort features outlined [here](https://github.com/cue-lang/cue/releases/tag/v0.12.0)
