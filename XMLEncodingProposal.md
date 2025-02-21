# XML to CUE Encoding Proposal

## Problem
Many users would benefit from using CUE with their XML files, however CUE does not currently have an encoding that supports XML.

## Purpose of this document
This document puts forward a proposal for an XML to CUE mapping called `koala` that can be used to add an XML encoding to CUE.

Given XML has constructs like attributes and namespaces that don't have identical analogues in CUE, there are many approaches for mapping from XML to CUE, with other future XML encodings possible. Examples of future options could include encodings that use a schema-guided approach (similar to the [textproto](https://pkg.go.dev/cuelang.org/go@v0.12.0/encoding/protobuf/textproto) encoding), and raw low-level AST style encodings that model each XML construct as a node that associates to abstractions like attributes, tags, and other content.

## Objectives
The mapping in this proposal aims to:

- allow users to write CUE constraints against XML files (XML Support)
- provide a mapping that makes it easy to understand what a CUE constraint refers to relative to the XML it is describing (Readability)
- allow the conversion of an XML file into CUE, and to also go back from CUE to a semantically equivalent XML file.

## Non-Objective
- This mapping does not aim to represent a lossless mapping. It is known for instance that unless otherwise specified in the rules below, order and XML comments are not preserved.

## Proposed Mapping

The proposed mapping follows a convention that is inspired by the [Badgerfish convention](http://www.sklar.com/badgerfish/), with deviations for compatibility with CUE and increased readability.

This new mapping will be called `koala` and follows the rules below:

1. Each XML element maps to a CUE struct, with the struct key being the element name.
2. Each nested XML element becomes a nested CUE struct.
3. Each XML attribute maps to a CUE struct property, where:
   - the property key is the attribute name prefixed with `$`, and
   - the property belongs to the struct that is mapped from the XML attribute's parent element.
4. The text content of an XML element maps to a CUE property keyed as `$$`, where that property belongs to the struct that is mapped from the XML content's parent element.
5. Multiple XML elements at the same level with the same tag text map to multiple CUE structs forming part of a CUE list at that level. When this is the case:
   - the tag text is used as the property key of the list
   - the order of structs in the list corresponds to the order of elements at the same level
6. Each XML attribute that defines a namespace maps to a CUE struct property in the same way that other XML attributes are mapped.
7. When an XML element name includes a namespace label as a prefix, the corresponding CUE struct property will be keyed by the same name and include the same prefix.
8. Values of XML attributes and elements will be typed in the corresponding CUE value when the type is inferred to be either int, float, boolean, null, or string. The type is inferred as described in the next section.

### Type mapping
Conceptually, the type of a given value is inferred as described in the sequence below:

1. If this is an element defined as `xsi:nil="true"` where `xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"`, then the type is inferred to be null.

If not null, the leading and trailing whitespace of the value are trimmed. In the order below, if the trimmed string can be converted to:  

2. an integer by [strconv.Atoi](https://pkg.go.dev/strconv#Atoi), then the type is inferred to be int.  
3. a float by [strconv.ParseFloat](https://pkg.go.dev/strconv#example-ParseFloat), then the type is inferred to be float.  
4. a bool by [strconv.ParseBool](https://pkg.go.dev/strconv#ParseBool), then the type is inferred to be bool.  

5. If a type is not yet determined, then the type is inferred to be string.  


### Sample CUE constraints for XML using `koala`

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
notes: quantity: $$: int
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

The content of the `note` XML element below maps to the value of the `$$` property of the `note` struct in CUE.

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
		$$:     "hello"
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
			$$:     "hello"
		}, {
			$alpha: "abcdef"
			$$:     "goodbye"
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
				$$: "Apples"
			}, {
				$$: "Bananas"
			}]
			"r:blah": {
				$$: "e3r"
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
			$$: 54
		}
		float: {
			$$: 43.12
		}
		string: {
			$$: "hello"
		}
		bool1: {
			$$: true
		}
		bool2: {
			$$: true
		}
	}
}
```

## Alternative Conventions Considered

Although no known conventions exist to map from XML to CUE, there are a number of known mappings that take XML to JSON, which we can take inspiration from.

### Parker and Spark Conventions

The Parker and Spark conventions use a very simplistic model where XML elements are mapped to object properties, and attributes are ignored.

We wish to maintain attribute information so we cannot use these mapping conventions as a whole.

### Badgerfish 

The Badgerfish convention maps elements, attributes, and content from XML to JSON. We follow the many of the rules in the Badgerfish convention, described [here](http://www.sklar.com/badgerfish/), with the modifications below to allow for mapping to CUE and for increased readability:

- XML attributes map to CUE properties starting with a `$` prefix instead of an `@` prefix, given `@` is already reserved in CUE for CUE attributes. Although we could still use the `@` prefix using quotes in CUE, we do not want to overload the usage of `@` for two concepts (ie: for XML attribute prefixes and for CUE attributes). Using the `$` prefix will also provide a less verbose notation given quotes do not need to be used with this prefix.

- Given a single `$` is [not a valid identifier in CUE](https://cuelang.org/docs/reference/spec/#identifiers), we use `$$` as the property to model element text content instead of `$`. Although we could use a quoted `"$"` as the key, we avoid this to prevent ambiguity with usage of `$` in other contexts, (such as "root element" in JSONPath), and to minimize usage of quotes.

- For namespaces, we do not recursively define namespaces in nested objects as this would un-necessarily increase verbosity in the mapped CUE. Instead we align more closely to how namespaces are defined in the XML, and only define namespaces in the CUE at the same level as they are declared in the XML. To illustrate how `koala` simplifies the mapping, we provide the example below (Badgerfish mapping taken from [here](http://www.sklar.com/badgerfish/)):

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

*koala*
```
{ 
	alice: {
		"$xmlns:charlie": "http://some-other-namespace"
		$xmlns:           "http://some-namespace"
		bob: {
			$$: "david"
		}
		"charlie:edgar": {
			$$: "frank"
		}
	}
}
```

### GData

The [GData convention](https://developers.google.com/gdata/docs/json?csw=1) is similar to Badgerfish, however makes no distinction between identifiers used for elements and those used for attributes. 

Unlike the Badgerfish convention, if one were to use this convention to map from XML to CUE, it would mean that it becomes ambiguous whether you are referring to an attribute or to an element when writing a CUE constraint. Further, it is not clear from the rules specified [here](https://developers.google.com/gdata/docs/json?csw=1) what happens when there is a collision between an element name and an attribute name.

### Abdera 

[This convention](https://wiki.open311.org/JSON_and_XML_Conversion/) is similar to the GData convention, however, it uses separate `children` and `attribute` abstractions when both nested XML elements and attributes are mentioned. Having to mention `children` and/or `attributes` in CUE constraints, as well as integer indexes for `children` arrays increases verbosity and complexity, which goes against the readability objective of this paper. To illustrate this with an example for Abdera:

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
			myAttr:  "attrVal"
			attrTwo: "two"
		}
		children: [ {
			point:  "example"
			sample: "other"
		}]
	}		
}
```


### JsonML 

Short for JSON Markup Language, [this convention](http://www.jsonml.org/xml/) makes heavy use of arrays to ensure an order-preserving mapping, where each element maps to an array entry, and each attribute also maps to an array entry. An example mapping is shown [here](https://wiki.open311.org/JSON_and_XML_Conversion/).

Having to work out (count) integer indexes when writing a CUE constraint rather than just simply using the element and attribute identifiers found in the XML makes this mapping too unwieldy to use for the purposes of our mapping.


## Testing Plan

The XML to CUE mapping scenarios required are covered by the examples described [here](https://github.com/matthew-sladescu/xml-cue-samples/blob/main/README.md). We will consider the solution complete once it can both decode and encode the examples shown there, along with any other test cases requested by the CUE maintainer team.

## Deployment Plan

The new `koala` encoding will not be the default XML encoding, but rather an opt-in encoding. Users will be able to use this from the command line using a command similar to:

`cue vet schema.cue xml+koala: data.xml`

Given this is not the default encoding, the command below would not work:

`cue vet schema.cue data.xml`

This will initially be an experimental encoding, which will be specified in the documentation, however given that it requires opt-in when the xml encoding to be used is specified, it does not need to be toggled using the `CUE_EXPERIMENT` variable as other experimental features are.

We also note that embedded XML within CUE will not be supported on day 1.
