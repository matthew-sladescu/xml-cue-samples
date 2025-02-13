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

*xml*
```
<note>
	<to>Tove</to>
	<from>Jani</from>
	<heading>Reminder</heading>
	<body>Don't forget me this weekend!</body>
</note>
```

*cue*
```
{
	note: {
		to: {
			$: "Tove"
		}
		from: {
			$: "Jani"
		}
		heading: {
			$: "Reminder"
		}
		body: {
			$: "Don't forget me this weekend!"
		}
	}
}
```
