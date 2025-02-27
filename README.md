# XML to CUE Mapping Samples

This document uses examples to illustrate a proposed mapping from XML to CUE.

## 1.xml -> 1.cue : Simple Mapping

Shows the mapping for simple XML elements and content.

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
			$$: "Tove"
		}
		from: {
			$$: "Jani"
		}
		heading: {
			$$: "Reminder"
		}
		body: {
			$$: "Don't forget me this weekend!"
		}
	}
}
```

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
		$alpha: "abcd"
		to: {
			$$: "Tove"
		}
		from: {
			$$: "Jani"
		}
		heading: {
			$$: "Reminder"
		}
		body: {
			$$: "Don't forget me this weekend!"
		}
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
		$alpha: "abcd"
		to: {
			$$: "Tove"
		}
		from: {
			$$: "Jani"
		}
		heading: {
			$$: "Reminder"
		}
		body: {
			$$: "Don't forget me this weekend!"
		}
		alpha: {
			$$: "efgh"
		}
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
		$alpha: "abcd"
		$$:     "hello"
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
			$alpha: "abcd"
			$$:     "hello"
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
			$alpha: "abcd"
			$$:     "hello"
		}, {
			$alpha: "abcdef"
			$$:     "goodbye"
		}]
	}
}
```

## 7.xml -> 7.cue : Interleaving Element types

Shows how a collection is represented when it has an interleaved element of a different type in the middle.

*7.xml*
```
<notes>
	<note alpha="abcd">hello</note>
	<note alpha="abcdef">goodbye</note>
	<book>mybook</book>
	<note alpha="ab">goodbye</note>
	<note>direct</note>
</notes>
```

*7.cue*
```
{
	notes: {
		note: [{
			$alpha: "abcd"
			$$:     "hello"
		}, {
			$alpha: "abcdef"
			$:      "goodbye"
		}, {
			$alpha: "ab"
			$$:     "goodbye"
		}, {
			$$:	"direct"
		}]
		book: {
			$$: 	"mybook"
		}
	}
}
```


## 8.xml -> 8.cue : Namespaces

Shows how namespaces and associated member elements are modeled

*8.xml*
```
<h:table xmlns:h="http://www.w3.org/TR/html4/">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
  </h:tr>
</h:table>
```

*8.cue*
```
{
	"h:table": {
		"$xmlns:h": "http://www.w3.org/TR/html4/"
		"h:tr": {
			"h:td": [{
				$$: "Apples"
			}, {
				$$: "Bananas"
			}]
		}
	}
}
```


## 9.xml -> 9.cue : Mixed Namespaces

Shows how XML documents where multiple namespaces are declared are represented, and how members elements of these different namespaces are represented in CUE.

*9.xml*
```
<h:table xmlns:h="http://www.w3.org/TR/html4/" xmlns:r="d">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
    <r:blah>e3r</r:blah>
  </h:tr>
</h:table>
```

*9.cue*
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


## 10.xml -> 10.cue : Elements with same name but different namespaces

Shows how elements with the same name, but under different namespaces are represented.
In the example below these are ```h:td``` and ```r:td```

*10.xml*
```
<h:table xmlns:h="http://www.w3.org/TR/html4/" xmlns:r="d">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
    <r:td>e3r</r:td>
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
			"h:td": [{
				$$: "Apples"
			}, {
				$$: "Bananas"
			}]
			"r:td": {
				$$: "e3r"
			}
		}
	}
}
```


## 11.xml -> 11.cue : Collection of elements, where elements have optional properties

Shows how elements of a collection (in this case of type ```book```) having optional properties (here the optional ```volume``` property) are represented in CUE.

*11.xml*
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

*11.cue*
```
{
	books: {
		book: [{
			title: {
				$$: "title"
			}
			author: {
				$$: "John Doe"
			}
		}, {
			title: {
				$$: "title2"
			}
			author: {
				$$: "Jane Doe"
			}
		}, {
			title: {
				$$: "Lord of the rings"
			}
			author: {
				$$: "JRR Tolkien"
			}
			volume: [{
				title: {
					$$: "Fellowship"
				}
				author: {
					$$: "JRR Tolkien"
				}
			}, {
				title: {
					$$: "Two Towers"
				}
				author: {
					$$: "JRR Tolkien"
				}
			}, {
				title: {
					$$: "Return of the King"
				}
				author: {
					$$: "JRR Tolkien"
				}
			}]
		}]
	}
}
```

```
