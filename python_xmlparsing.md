## XML parsing in Python

### Element: tag and attributes

Every **element** has a `<tag>` and a dictionary of attributes. An **attribute** has the form of `{‘key’:’value’}`.
This applies also to the root element.

XML Example:

```xml
<?xml version="1.0"?>
<data>
    <country name="Liechtenstein">
        <rank>1</rank>
        <year>2008</year>
        <gdppc>141100</gdppc>
        <neighbor name="Austria" direction="E"/>
        <neighbor name="Switzerland" direction="W"/>
    </country>
    <country name="Singapore">
        <rank>4</rank>
        <year>2011</year>
        <gdppc>59900</gdppc>
        <neighbor name="Malaysia" direction="N"/>
    </country>
    <country name="Panama">
        <rank>68</rank>
        <year>2011</year>
        <gdppc>13600</gdppc>
        <neighbor name="Costa Rica" direction="W"/>
        <neighbor name="Colombia" direction="E"/>
    </country>
</data>
```

Getting the tag and attributes of root and children elements:

``` python
# parse XML file
import xml.etree.ElementTree as ET
tree = ET.parse('country_data.xml')

# get the root element
root = tree.getroot()

# get root tag and attributes
>>> root.tag
'data'
>>> root.attrib
{}

# get tag and attributes of root children
>>> for child in root:
...     print(child.tag, child.attrib)
...
country {'name': 'Liechtenstein'}
country {'name': 'Singapore'}
country {'name': 'Panama'}

# we can access specific child nodes by index
>>> root[0][1].text
'2008'
```



### Element methods

- `.tag` delivers the tag of the element
- `.attrib` delivers all the attributes of the element
  [for some reason in a file with namespaces I didn’t get all the attributes of the root (that is, all the namespaces)]
- `.iter()` searches for all elements with a specific name

```python
>>> for neighbor in root.iter('neighbor'):
...     print(neighbor.attrib)
...
{'name': 'Austria', 'direction': 'E'}
{'name': 'Switzerland', 'direction': 'W'}
{'name': 'Malaysia', 'direction': 'N'}
{'name': 'Costa Rica', 'direction': 'W'}
{'name': 'Colombia', 'direction': 'E'}
```

- `.findall()` searches for elements which are **direct** children of the given element

```python
root.findall(".")
```

- `.find()` finds the **first** child with a particular tag
- `.text` accesses the element’s text content
- `.get()` accesses the value of an attribute

```python
>>> for country in root.findall('country'):
...     rank = country.find('rank').text
...     name = country.get('name')
...     print(name, rank)
...
Liechtenstein 1
Singapore 4
Panama 68
```



### XPath syntax

| Syntax              | Meaning                                                      |
| ------------------- | ------------------------------------------------------------ |
| `.`                 | Selects the current node.  This is mostly useful at the beginning of the path, to indicate that it’s a relative path. |
| `//`                | Selects **all levels** beneath the current  element.         |
| `..`                | Selects the parent element.                                  |
| `[@attrib]`         | Selects all elements that have the given attribute.          |
| `[@attrib='value']` | Selects all elements for which the given attribute has the given value. |
| `[tag]`             | Selects all elements that have an immediate **child** named `tag`. |
| `[.='text']`        | Selects all elements whose complete **text content** equals the given `text`. |
| `[tag='text']`      | Selects all elements that have an immediate child named `tag` whose complete text content equals the given `text`. |
| `[position]`        | Selects all elements that are located at the given position.  The position can be: <br />- an integer **(1 is the first position)** <br />- the expression `last()` (for the last position)<br />- a position relative to the last position, e.g. `last()-1`. |

Examples:

```python
# Top-level elements
root.findall(".")

# All 'neighbor' grand-children of 'country' children of the top-level
# elements
root.findall("./country/neighbor")

# Nodes with name='Singapore' that have a 'year' child
root.findall(".//year/..[@name='Singapore']")

# 'year' nodes that are children of nodes with name='Singapore'
root.findall(".//*[@name='Singapore']/year")

# All 'neighbor' nodes that are the second child of their parent
root.findall(".//neighbor[2]")
```



### XML with namespaces

XML example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mets:mets xmlns:mets="http://www.loc.gov/METS/"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:mods="http://www.loc.gov/mods/v3"
           xmlns:dv="http://dfg-viewer.de/"
           xmlns:xlink="http://www.w3.org/1999/xlink"
           xsi:schemaLocation="http://www.loc.gov/mods/v3 http://www.loc.gov/standards/mods/v3/mods-3-7.xsd http://www.loc.gov/METS/ http://www.loc.gov/standards/mets/version17/mets.v1-7.xsd">
   <mets:metsHdr CREATEDATE="2020-09-10T08:51:57">
      <mets:agent ROLE="CREATOR" TYPE="ORGANIZATION">
         <mets:name>Verbundzentrale des GBV (VZG)</mets:name>
      </mets:agent>
   </mets:metsHdr>
  # here comes more code ...
</mets:mets>  
```

XPath syntax with namespaces:

| XML          | XPath                                                        |
| ------------ | ------------------------------------------------------------ |
| `prefix:tag` | `{uri}tag`<br />If there is a default namespace, its URI gets prepended to all of the non-prefixed tags. |

For example:

```python
>>> agent = root.find(".//{http://www.loc.gov/METS/}agent")
>>> agent.attrib
{'ROLE': 'CREATOR', 'TYPE': 'ORGANIZATION'}
```

A **better way** to search the namespaced XML example is to create a dictionary with your own prefixes and use those in the search functions:

```python
>>> ns = {'mets':'http://www.loc.gov/METS/'}
>>> agent = root.find('.//mets:agent', ns)
>>> agent.attrib
{'ROLE': 'CREATOR', 'TYPE': 'ORGANIZATION'}
```

This **doesn’t work** though for namespaces in attributes!

see also: https://stackoverflow.com/questions/14853243/parsing-xml-with-namespace-in-python-via-elementtree



### Other links / tutorials

https://www.datacamp.com/community/tutorials/python-xml-elementtree

https://www.geeksforgeeks.org/xml-parsing-python/