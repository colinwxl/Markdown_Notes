# python [lxml](http://lxml.de/index.html)
version: lxml-4.1.1

Download: https://pypi.python.org/pypi/lxml/

[PDF documentation](http://lxml.de/4.1/lxmldoc-4.1.1.pdf)

## Chapter 1 lxml
lxml is the most feature-rich and easy-to-use library for processing XML and HTML in the Python language.
The lxml XML toolkit is a Pythonic binding for the C libraries libxml2 and libxslt. It is unique in that it combines the speed and XML feature completeness of these libraries with the simplicity of a native Python API, mostly compatible but superior to the well-known ElementTree API. The latest release works with all CPython versions from 2.6 to 3.6.

## Chapter 2 Why lxml?
- Standards-compliant XML support.
- Support for (broken) HTML.
- Full-featured.
- Actively maintained by XML experts.
- fast. fast! FAST!

## Chapter 3 Installing lxml
> Debian/Ubuntu:
`sudo apt-get install python3-lxml`
For MacOS-X:
`sudo port install py27-lxml`

### Requirements
- libxml2 version 2.7.0 or later (libxml2-dev)
- libxslt version 2.9.2 or later (libxslt-dev)

### Installation
`pip install lxml==4.1.1`
`sudo pip install lxml` install lxml globally
disable the C compiler optimisations: `CFLAGS="-O0" pip install lxml`

## Chapter 4 Benchmarks and Speed
`bench_etree.py`; `bench_xpath.py`; `bench_objectify,py`

## Chapter 5 ElementTree compatibility of lxml.etree

## Chapter 6 lxml FAQ
1. tutorial? [lxml.etree Tutorial](http://lxml.de/tutorial.html); [extended etree API](http://lxml.de/api.html)
2. implemented standards? http://xmlsoft.org/
3. bugs reporting?
```
import sys
from lxml import etree
print("%-20s: %s" % ('Python', sys.version_info))
print("%-20s: %s" % ('lxml.etree', etree.LXML_VERSION))
print("%-20s: %s" % ('libxml used', etree.LIBXML_VERSION))
print("%-20s: %s" % ('libxml compiled', etree.LIBXML_COMPILED_VERSION))
print("%-20s: %s" % ('libxslt used', etree.LIBXSLT_VERSION))
print("%-20s: %s" % ('libxslt compiled', etree.LIBXSLT_COMPILED_VERSION))
```
4. use threads to concurrently access the lxml API? yes
5. lxml can't parse XML from unicode strings?
> XML is explicitly defined as a stream of bytes. [XML specification](http://www.w3.org/TR/REC-xml/)
6. [Xpath specification](http://www.w3.org/TR/xpath)
7. `pretty_print` option dose not reformat XML output?

## Chapter 7 The lxml.etree Tutorial
```
try:
	from lxml import etree
	print("running with lxml.etree")
except ImportError:
	try:
		# Python 2.5
		import xml.etree.cElementTree as etree
		print("running with cElementTree on Python 2.5+")
	except ImportError:
		try:
			# Python 2.5
			import xml.etree.ElementTree as etree
			print("running with ElementTree on Python 2.5+")
		except ImportError:
			try:
				# normal cElementTree install
				import cElementTree as etree
				print("running with cElementTree")
			except ImportError:
				try:
					# normal ElementTree install
					import elementtree.ElementTree as etree
					print("running with ElementTree")
				except ImportError:
					print("Failed to import ElementTree from any known place")
```
`from lxml import etree`

### The Element class
```
from lxml import etree

root = etree.Element("root")
root.append(etree.Element("child1")) # create child element
child2 = etree.SubElement(root, "child2") # a more efficient way

# To see this in really XML, serialise the tree:
print(etree.tostring(root, pretty_print=True))
```

### Elements are like lists
elements mimic the behaviour of normal Python lists as closely as possible:
```
child = root[0]
print(child.tag)
print(len(root))
root.index(root[1])
children = list(root)
root.insert(0, etree.Element(("child0"))
print(etree.iselement(root))

if len(root):
	...
root[0] = root[-1] # this moves the element in lxml.etree
root is root[0].getparent() # Element in lxml.etree has exactly one parent
```
to copy an element to a different position in lxml.etree
```
from copy import deepcopy
element = etree.Element("neu")
element.append(deepcopy(root[1]))
```
siblings(or neighbours) of an element are accessed as next and previous elements:
```
root[0] is root[1].getprevious()
root[1] is root[0].getnext()
```

### Elements carry attributes as a dict
```
root = etree.Element("root", interesting="totally")
etree.tostring(root) # b’<root interesting="totally"/>’
print(root.get("interesting"))
print(root.get("hello")) # None
root.set("hello", "Huhu")
print(root.get("hello")) # Huhu
sorted(root.keys())
for name, value in sorted(root.items()):
	print('%s = %r' % (name, value))
```
to get a 'real' dictionary-like object:
```
attributes = root.attrib
print(attributes["interesting"])
attributes["hello"] = "Guten Tag"
print(root.get("hello")) # Guten Tag
```
to get an independent snapshot of the attributes that does not depend on the XML tree, copy it into a dict:
```
d = dict(root.attrib)
```

### Element contain text
```
root = etree.Element("root")
root.text = "TEXT"
etree.tostring(root) # b'<root>TEXT</root>'
```
document-style or mixed-content XML:
```
html = etree.Element("html")
body = etree.SubElement(html, "body")
body.text = "TEXT"
br = etree.SubElement(body, "br")
br.tail = "TAIL"
etree.tostring(html)
b'<html><body>TEXT<br/>TAIL</body></html>'
```
do not always want its tail text:
```
etree.tostring(br) # b'<br/>TAIL'
etree.tostring(br, with_tail=False) # n'<br/>'
```
to read only the text, without any intermediate tags
```
etree.tostring(html, method="text") # b'TEXTTAIL'
```

### Using XPath to find text
```
print(html.xpath("string()")) # TEXTTAIL
print(html.xpath("//text()")) # ['TEXT', 'TAIL']
```
to wrap Xpath in function:
```
build_text_list = etree.XPath("//text()")
print(build_text_list(html)) # ['TEXT', 'TAIL']
```
a result returned by XPath is a special 'smart' object that knows about its origins:
```
texts = build_text_list(html)
print(texts[0]).getparent().tag) # 'body'
print(texts[1]).getparent().tag) # 'br'
print(texts[0].is_text) # True
print(texts[1].is_text) # False
print(texts[1].is_tail) # True
```
this works for results of the `text()` function instead of `string()` or `concat()`

### Tree iteration