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
```
root = etree.Element("root")
etree.SubElement(root, "child").text = "Child 1"
etree.SubElement(root, "child").text = "Child 2"
etree.SubElement(root, "another").text = "Child 3"
print(etree.tostring(root, pretty_print=True))
for element in root.iter():
	print("%s - %s" % (element.tag, element.text))
for element in root.iter("child"):
	print("%s - %s" % (element.tag, element.text))

for element in root.iter("another", "child"):
	print("%s - %s" % (element.tag, element.text))
child - Child 1
child - Child 2
another - Child 3
```
By default, iteration yields all nodes in the tree, including ProcessingInstructions, Comments and Entity instances. If you want to make sure only Element objects are returned, you can pass the Element factory as tag parameter:
```
root.append(etree.Entity("#234"))
root.append(etree.Comment("some comment"))
for element in root.iter():
	if isinstance(element.tag, basestring): # or ’str’ in Python 3
	print("%s - %s" % (element.tag, element.text))
	else:
		print("SPECIAL: %s - %s" % (element, element.text))

for element in root.iter(tag=etree.Element):
	print("%s - %s" % (element.tag, element.text))

for element in root.iter(tag=etree.Entity):
	print(element.text)
```
Note that passing a wildcard "*" tag name will also yield all Element nodes (and only elements).

### Serialisation
Serialisation commonly uses the `tostring()` function that returns a string, or the `ElementTree.write()` method that writes to a file, a file-like object, or a URL (via FTP PUT or HTTP POST). Both calls accept the same keyword arguments like `pretty_print` for formatted output or `encoding` to select a specific output encoding other than plain ASCII:
```
root = etree.XML('<root><a><b/></a></root>')
etree.tostring(root)
print(etree.tostring(root, xml_declaration=True))
print(etree.tostring(root, encoding='iso-8859-1'))
print(etree.tostring(root, pretty_print=True))
```
You can serialise to HTML or extract the text content by passing the method keyword:
```
root = etree.XML('<html><head/><body><p>Hello<br/>World</p></body></html>')
etree.tostring(root) # default: method = 'xml'
etree.tostring(root, method='xml')
etree.tostring(root, method='html', pretty_print=True)
etree.tostring(root, method='text') # b'HelloWorld'
```
As for XML serialisation, the default encoding for plain text serialisation is ASCII:
```
br = next(root.iter('br'))
br.tail = u’W\xf6rld’
etree.tostring(root, method=’text’) # raise UnicodeEncodeError
etree.tostring(root, method=’text’, encoding="UTF-8") # b’HelloW\xc3\xb6rld’
# serialising to a Python unicode string instead of a byte string might become handy.
etree.tostring(root, encoding=’unicode’, method=’text’) # ’HelloW\xf6rld’
```
The W3C has a good article about the [Unicode character set and character encodings](http://www.w3.org/International/tutorials/tutorial-char-enc/).

### The ElementTree class
An `ElementTree` is mainly a document wrapper around a tree with a root node. An `ElementTree` is also what you get back when you call the `parse()` function to parse files or file-like
objects
```
root = etree.XML('''\
<?xml version="1.0"?>
<!DOCTYPE root SYSTEM "test" [ <!ENTITY tasty "parsnips"> ]>
<root>
<a>&tasty;</a>
</root>
''')
tree = etree.ElementTree(root)
print(tree.docinfo.xml_version) # 1.0
>>> print(tree.docinfo.doctype) # <!DOCTYPE root SYSTEM "test">
tree.docinfo.public_id = ’-//W3C//DTD XHTML 1.0 Transitional//EN’
tree.docinfo.system_url = ’file://local.dtd’
print(tree.docinfo.doctype)
<!DOCTYPE root PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "file://local.dtd">

print(etree.tostring(tree.getroot()))
```

### Parsing from strings and files
`lxml.etree` supports parsing XML in a number of ways and from all important sources, namely strings, files, URLs (http/ftp) and file-like objects. The main parse functions are `fromstring()` and `parse()`, both called with the source as first argument.

#### The fromstring() function
```
some_xml_data = "<root>data</root>"
root = etree.fromstring(some_xml_data)
print(root.tag); etree.tostring(root)
```

#### The XML() funtion
```
root = etree.XML("<root>data</root>")
root = etree.HTML("<p>data</p>")
etree.tostring(root) # b'<html><body><p>data</p></body></html>'
```

#### The parse() function
As an example of such a file-like object, the following code uses the `BytesIO` class for reading from a string instead of an external file. That class comes from the `io` module in Python 2.6 and later. Note that `parse()` returns an `ElementTree` object, not an `Element` object as the string parser functions. See `help(etree.XMLParser)` to find out about the available parser options.
```
from io import BytesIO
some_file_or_file_like_object = BytesIO(b"<root>data</root>")
tree = etree.parse(some_file_or_file_like_object)
```
The `parse()` function supports any of the following sources:
- an open file object (make sure to open it in binary mode)
- a file-like object taht has a .read(byte_count) method returning a byte string on each call
- a filename string
- an HTTP or FTP URL string

#### Parser object
```
parser = etree.XMLParser(remove_blank_text=True)
root = etree.XML("<root> <a/> <b>  </b> </root>", parser)
etree.tostring(root) # b'<root><a/><b>  </b></root>'
```
content at leaf elements tends to be data content (even if blank).
```
for element in root.iter("*"):
	if element.text is not None and not element.text.strip():
		element.text = None
etree.tostring(root) # b'<root><a/><b/></root>'
```

#### Incremental parsing
`lxml.etree` provides two ways for incremental step-by-step parsing. One is through file-like objects, where it calls the `read()` method repeatedly. This is best used where the data arrives from a source like `urllib` or any other file-like object that can provide data on request. Note that the parser will block and wait until data becomes available in this case:
```
class DataSource:
	data = [ b"<roo", b"t><", b"a/", b"><", b"/root>" ]
	def read(self, requested_size):
		try:
			return self.data.pop(0)
		except IndexError:
			return b''
tree = etree.parse(DataSource())
```
The second way is through a feed parser interface, given by the `feed(data)` and `close()` methods:
```
parser = etree.XMLParser()
parser.feed("<roo")
parser.feed("t><")
parser.feed("a/")
parser.feed("><")
parser.feed("/root>")
root = parser.close()
```
After calling the `close()` method (or when an exception was raised by the parser), you can reuse the parser by calling its `feed()` method again:
```
parser.feed("<root/>")
root = parser.close()
etree.tostring(root) # b’<root/>’
```

#### Event-driven parsing
Sometimes, all you need from a document is a small fraction somewhere deep inside the tree, `lxml.etree` supports this use case with two event-driven parser interfaces, one that generates parser events while building the tree (iterparse), and one that does not build the tree at all, and instead calls feedback methods on a target object in a SAX-like fashion.
Here is a simple `iterparse()` example:
```
some_file_like = BytesIO(b"<root><a>data</a></root>")
for event, element in etree.iterparse(some_file_like):
	print("%s, %4s, %s" % (event, element.tag, element.text))
```
By default, `iterparse()` only generates events when it is done parsing an element, but you can control this through the `events` keyword argument:
```
some_file_like = BytesIO(b"<root><a>data</a></root>")
for event, element in etree.iterparse(some_file_like, events=("start", "end")):
	print("%5s, %4s, %s" % (event, element.tag, element.text))
```
It also allows you to `.clear()` or modify the content of an Element to save memory. So if you parse a large tree and you want to keep memory usage small, you should clean up parts of the tree that you no longer need:
```
some_file_like = BytesIO(b"<root><a><b>data</b></a><a><b/></a></root>")
for event, element in etree.iterparse(some_file_like):
	if element.tag == 'b':
		print(element.text)
	elif element.tag == 'a':
		print("** cleaning up the subtree")
		element.clear()
```
use `interparse()` to parsing generated XML files, e.g database dumps.
```
xml_file = BytesIO(b'''\
<root>
  <a><b>ABC</b><c>abc</c></a>
  <a><b>MORE DATA</b><c>more data</c></a>
  <a><b>XYZ</b><c>xyz</c></a>
</root>''')
for _, element in etree.iterparse(xml_file, tag='a'):
	print('%s -- %s' % (element.findtext('b'), element[1].text))
	element.clear()
```
SAX-like events:
```
class ParserTarget:
	events = []
	close_count = 0
	def start(self, tag, attrib):
		self.events.append(("statr", tag, attrib))
	def close(self):
		events, self.events = self.events, []
		self.close_count += 1
		return events

parser_target = ParserTarget()
parser = etree.XMLParser(target= parser_target)
events = etree.fromstring('<root test="treue"/>', parser)
print(parser_target.close_count)
for event in events:
	print('event: %s - tag: %s' % (event[0], event[1]))
	for attr, value in envent[2].items():
		print(' * %s = %s' % (attr, value))
```
You can reuse the parser and its target as often as you like, so you should take care that the `.close()` method really resets the target to a usable state (also in the case of an error!).

### Namespaces
The ElementTree API avoids [namespace prefixes](http://www.w3.org/TR/xml-names/#ns-qualnames) wherever possible and deploys the real namespace (the URI) instead:
```
xhtml = etree.Element("{http://www.w3.org/1999/xhtml}html")
body = etree.SubElement(xhtml, "{http://www.w3.org/1999/xhtml}body")
body.text = "Hello World"

print(etree.tostring(xhtml, pretty_print=True))
<html:html xmlns:html="http://www.w3.org/1999/xhtml">
  <html:body>Hello World</html:body>
</html:html>
```
It is common practice to store a namespace URI in a global variable. To adapt the namespace prefixes for serialisation, you can also pass a mapping to the Element factory function, e.g. to define the default namespace:
```
XHTML_NAMESPACE = "http://www.w3.org/1999/xhtml"
XHTML = "{%s}" % XHTML_NAMESPACE
NSMAP = {None : XHTML_NAMESPACE} # the default namespace (no prefix)
xhtml = etree.Element(XHTML + "html", nsmap=NSMAP)
body = etree.SubElement(xhtml, XHTML + "body")
body.text = "Hello World"
print(etree.tostring(xhtml, pretty_print=True))
b'<html xmlns="http://www.w3.org/1999/xhtml"><body>Hello World</body></html>'
xhtml.nsmap # {None: 'http://www.w3.org/1999/xhtml'}
```
You can also use the `QName` helper class to build pr split qualified tag names:
```
tag = etree.QName('http://www.w3.org/1999/xhtml', 'html')
print(tag.localname) # html
print(tag.namespace) # http://www.w3.org/1999/xhtml
print(tag.text) # {http://www.w3.org/1999/xhtml}html
tag = etree.QName('{http://www.w3.org/1999/xhtml}html')
print(tag.localname) # html
print(tag.namespace) # http://www.w3.org/1999/xhtml

root = etree.Element('{http://www.w3.org/1999/xhtml}html')
tag = etree.QName(root)
print(tag.localname)
tag = etree.QName(root, 'script')
print(tag.text) # {http://www.w3.org/1999/xhtml}script
tag = etree.QName('{http://www.w3.org/1999/xhtml}html', 'script')
print(tag.text)
```
`.nsmap`:
```
root = etree.Element('root', nsmap={'a': ’http://a.b/c’})
>>> child = etree.SubElement(root, 'child', nsmap={'b': 'http://b.c/d'})
len(root.nsmap) # 1
len(child.nsmap) # 2
child.nsmap['a'] # 'http://a.b/c'
child.nsmap['b'] # 'http://b.c/d'
```
`lxml.etree` will ensure that the attribute uses a prefixed
namespace declaration.
```
body.set(XHTML + "bgcolor", "#CCFFAA")
print(etree.tostring(xhtml, pretty_print=True))
print(body.get("bgcolor")) # None
body.get(XHTML + "bgcolor") # '#CCFFAA'
```
use XPath with fully qualified names:
```
find_xhtml_body = etree.ETXPath("//{%s}body" % XHTML_NAMESPACE)
results = find_xhtml_body(xhtml)
print(results[0].tag)
```
wildcards "*":
```
for el in xhtml.iter('*'): print(el.tag)
for el in xhtml.iter('{http://www.w3.org/1999/xhtml}*'): print(el.tag)
for el in xhtml.iter('{*}body'): print(el.tag)
```
look for elements taht do not have a namespace:
```
[ el.tag for el in xhtml.iter('{http://www.w3.org/1999/xhtml}body') ]
[ el.tag for el in xhtml.iter('body') ]
[ el.tag for el in xhtml.iter('{}body') ]
[ el.tag for el in xhtml.iter('{}*') ]
```

### The E-factory
The `E-factory` provides a simple and compact syntax for generating XML and HTML:
```
from lxml.builder import E
def CLASS(*args):
	return {"class": ' '.join(args)}
html = page = (
	E.html(
		E.head(
			E.title("This is a sample document")
		),
		E.body(
			E.h1("hello!", CLASS("title")),
			E.p("This is a paragraph, with ", E.b("bold"), " text in it!"),
			E.p("This is another paragraph, with a", "\n      ",
				E.a("link", href="http://wwww.python.org"), "."),
			E.p("Here are some reserved characters: <spam&egg>."),
			etree.XML("<p>And finally an embedded XHTML fragment.</p>"),
		)
	)
)
print(etree.tostring(page, pretty_print=True, encoding="unicode"))
```
Element creation based on attribute access makes it easy to build up a simple vocabulary for an XML language:
```
from lxml.builder import ElementMaker
E = ElementMaker(namespace="http://my.de/fault/namespace", nsmap={'p : "http://my.de/fault/namespace"})
DOC = E.doc
TITLE = E.title
SECTION = E.section
PAR = E.par
my_doc = DOC(
	TITLE("The dog and the hog"),
	SECTION(
		TITLE("The dog"),
		PAR("Once upon a time, ..."),
		PAR("And then ...")
	),
	SECTION(
		TITLE("The hog"),
		PAR("Sooner or later ...")
	)
)
print(etree.tostring(my_doc, pretty_print=True))
```

### ElementPath
The ElementTree library comes with a simple XPath-like path language called ElementPath. The main difference is that you can use the {namespace}tag notation in ElementPath expressions. However, advanced features like value comparison and functions are not available.
The API provides four methods here that you can find on Elements and ElementTrees:
- iterfind() iterates over all Elements that match the path expression
- findall() returns a list of matching Elements
- find() efficiently returns only the first match
- findtext() returns the .text content of the first match

```
root = etree.XML("<root><a x='123'>aText<b/><c/><b/></a></root>")
# find a child of an Element:
print(root.find("b")) # None
print(root.find("a").tag) # a
# find an Element anywhere in the tree:
print(root.find(".//b").tag) # b
[ b.tag for b in root.iterfind(".//b") ] # ['b', 'b']
# find Elements with a certain attribute:
print(root.findall(".//a[@x]")[0].tag) # a
print(root.findall(".//a[@y]")) # []

tree = etree.ElementTree(root)
a = root[0]
print(tree.getelementpath(a[0])) # a/b[1]
print(tree.getelementpath(a[1])) # a/c
print(tree.getelementpath(a[2])) # a/b[2]
tree.find(tree.getelementpath(a[2])) == a[2] # True
```
The .iter() method is a special case that only finds specific tags in the tree by their name, not based on a path.
```
print(root.find(".//b").tag) # b
print(next(root.iterfind(".//b")).tag) # b
print(next(root.iter("b")).tag) # b
```
Note that the .find() method simply returns None if no match is found, whereas the other two examples would raise a `StopIteration` exception.

## Chapter 8 APIs specifuc to lxml.etree
For a compele reference of API, see the [generated API documentation](http://lxml.de/api/index.html).
lxml is extremely extensible through [XPath functions in Python](http://lxml.de/extensions.html), custom [Python element classes](http://lxml.de/element_classes.html), custom [URL resolvers](http://lxml.de/resolvers.html) and even [at the C-level](http://lxml.de/capi.html).

### lxml.etree
lxml.etree tries to follow the [ElementTree API](lxml.etree tries to follow the ElementTree API wherever it can.) wherever it can.
`lxml.etree.LXML_VERSION`; `lxml.etree.LIBXML_VERSION` and `lxml.etree.LIBXSLT_VERSION`
the following examples based on: `from lxml import etree`

### Other Element APIs
...

### Trees and Documents
Compared to the original ElementTree API, lxml.etree has an extended tree model. It knows about parents and siblings of elements: `getparent()`; `getnext()`; `getprevious()`
```
root = etree.Element("root")
a = etree.SubElement(root, "a")
b = etree.SubElement(root, "b")
c = etree.SubElement(root, "c")
d = etree.SubElement(root, "d")
e = etree.SubElement(d,    "e")
b.getparent() == root # True
print(b.getnext().tag) # c
print(c.getprevious().tag) # b
```
You can retrieve an ElementTree for the root node of a document from any of its elements.
```
tree = d.getroottree()
print(tree.getroot().tag) # root
```
You can use ElementTrees to create XML trees with an explicit root node:
```
tree = etree.ElementTree(d)
print(tree.getroot().tag) # d
etree.tostring(tree) # b'<d><e/></d>'
```
The rule is that all operations that are applied to Elements use either the Element itself as reference point, or the absolute root of the document that contains this Element (e.g. for absolute XPath expressions). All operations on an ElementTree use its explicit root node as reference.
```
element = tree.getroot()
print(element.tag) # d
print(element.getparent().tag) # root
print(element.getroottree().getroot().tag) # root
```

### Iteration
```
[ child.tag for child in root ]
[ el.tag for el in root.iter() ] # tree traversal
[ child.tag for child in root.iterchildren() ]
[ child.tag for child in root.iterchildren(reversed=True) ]
[ sibling.tag for sibling in b.itersiblings() ] # ['c', 'd']
[ sibling.tag for sibling in c.itersiblings(preceding=True) ] # ['b', 'a']
[ ancestor.tag for ancestor in e.iterancestors() ] # ['d', 'root']
[ el.tag for el in root.iterdescendants() ]
```
All of these iterators support one (or more, since lxml 3.0) additional arguments that filter the generated elements by tag name:
`[ el.tag for el in root.iter('d', 'a') ]`
The most common way to traverse an XML tree is depth-first. While there is no dedicated method for breadth-first traversal:
```
root = etree.XML('<root><a><b/><c/></a><d><e/></d></root>')
print(etree.tostring(root, pretty_print=True, encoding='unicode'))
queue = deque([root])
while queue:
	el = queue.popleft()
	queue.extend(el)
	print(el.tag)
```

### Error handling on exceptions
```
etree.clear.clear_error_log()
broken_xml = '<root><a></root>'
try:
	etree.parse(StringIO(broken_xml))
except etree.XMLSyntaxError, e:
	pass

log = e.error_log.filter_from_level(etree.ErrorLevels.FATAL)
print(log)
entry = log[0]
print(entry.domain_name) # PARSER
print(entry.type_name) # ERR_TAG_NAME_MISMATCH
print(entry.filename) # <string>

entry = e.error_log.last_error
print(entry.domain_name) # PARSER
print(entry.type_name) # ERR_TAG_NAME_MISMATCH
print(entry.filename) # <string>
```

### Error logging
...

### Serialisation
By default, lxml (just as ElementTree) outputs the XML declaration only if it is required by the standard:
```
unicode_root = etree.Element( u"t\u3120st" )
unicode_root.text = u"t\u0A0Ast"
etree.tostring(unicode_root, encoding="utf-8")
b'<t\xe3\x84\xa0st>t\xe0\xa8\x8ast</t\xe3\x84\xa0st>'

print(etree.tostring(unicode_root, encoding="iso-8859-1"))
<?xml version='1.0' encoding='iso-8859-1'?>
<t&#12576;st>t&#2570;st</t&#12576;st>
```
Also see the general remarks on [Unicode support](http://lxml.de/parsing.html#python-unicode-strings).
You can enable or disable the declaration explicitly by passing another keyword argument for the serialisation:
```
print(etree.tostring(root, xml_declaration=True))
<?xml version='1.0' encoding='ASCII'?>
<root><test/></root>

unicode_root.clear()
etree.tostring(unicode_root, encoding="UTF-16LE", xml_declaration=False)
b'<\x00t\x00 1s\x00t\x00/\x00>\x00'
```

### Incremental XML generation
```
f = BytesIO()
with etree.xmlfile(f) as xf:
	with xf.element('abc'):
		xf.write('text')

print(f.getvalue().decode('utf-8')) # <abc>text</abc>
```
```
f = BytesIO()
with etree.xmlfile(f) as xf:
	with xf.element('abc'):
		with xf.element('in'):
			for value in '123':
				# construct a really complex XML tree
				el = etree.Element('xyz', attr=value)
				xf.write(el)
				# no longer needed, discard it right away!
				el = None

print(f.getvalue().decode('utf-8'))
<abc><in><xyz attr="1"/><xyz attr="2"/><xyz attr="3"/></in></abc>
```
```
def writer(out_stream):
    with xmlfile(out_stream) as xf:
        with xf.element('{http://etherx.jabber.org/streams}stream'):
            while True:
                el = (yield)
                xf.write(el)
                xf.flush()
w = writer(stream)
next(w)   # start writing (run up to 'yield')
# whenever XML elements are available for writing, call
w.send(element)
# when done:
w.close()
```

### CDATA
By default, lxml's parser will strip CDATA sections from the tree and replace them by their plain text content. You can instruct the parser to leave CDATA sections in the document:
```
parser = etree.XMLParser(strip_cdata=False)
root = etree.XML('<root><![CDATA[test]]></root>', parser)
root.text # 'test'

etree.tostring(root) # b'<root><![CDATA[test]]></root>'
```
```
root.text = 'test'
root.text # 'test'
etree.tostring(root) # b'<root>test</root>'

root.text = etree.CDATA(root.text)
root.text # 'test'
etree.tostring(root) # b'<root><![CDATA[test]]></root>'
```

### Xinclude and ElemmentInculude
You can let lxml process xinclude statements in a document by calling the `xinclude()` method on a tree:
```
data = StringIO('''\
<doc xmlns:xi="http://www.w3.org/2001/XInclude">
<foo/>
<xi:include href="doc/test.xml" />
</doc>''')

tree = etree.parse(data)
tree.xinclude()
print(etree.tostring(tree.getroot()))
<doc xmlns:xi="http://www.w3.org/2001/XInclude">
<foo/>
<a xml:base="doc/test.xml"/>
</doc>
```

### write_c14n on ElementTree
The lxml.etree.ElementTree class has a method `write_c14n`, which takes a file object as argument. This file object will receive an UTF-8 representation of the canonicalized form of the XML, following the W3C C14N recommendation. For example:
```
f = StringIO('<a><b/></a>')
tree = etree.parse(f)
f2 = StringIO()
tree.write_c14n(f2)
print(f2.getvalue().decode("utf-8")) # <a><b></b></a>
```

## Chapter 9 Parsing XML and HTML with lxml
lxml supports one-step parsing as well
as step-by-step parsing using an event-driven API (currently only for XML).
The following examples base on:
```
from lxml import etree
from io import StringIO, BytesIO
```
### Parsers
There is support for parsing both XML and (broken) HTML. Note that XHTML is best parsed as XML. lxml can parse from a local file, an HTTP URL or an FTP URL. It also auto-detects and reads gzip-compressed XML files (.gz).
```
xml = '<a xmlns="test"><b xmlns="test"/></a>'
root = etree.fromstring(xml)
tree = etree.parse(StringIO(xml))
etree.tostring(tree.getroot())
tree = etree.parse("doc/test.xml")
```
If you want to parse from memory and still provide a base URL for the document (e.g. to support relative paths in an XInclude), you can pass the `base_url` keyword argument:
`root = etree.fromstring(xml, base_url="http://where.it/is/from.xml")`

#### Parser options
Available boolean keyword arguments:
- `attribute_defaults` - read the DTD (if referenced by the document) and add the default attributes from it
- `dtd_validation` - validate while parsing (if a DTD was referenced)
- `load_dtd` - load and parse the DTD while parsing (no validation is performed)
- `no_network` - prevent network access when looking up external documents (on by default)
- `ns_clean` - try to clean up redundant namespace declarations
- `recover` - try hard to parse through broken XML
- `remove_blank_text` - discard blank text nodes between tags, also known as ignorable whitespace. This is best used together with a DTD or schema (which tells data and noise apart), otherwise a heuristic will be applied.
- `remove_comments` - discard comments
- `remove_pis` - discard processing instructions
- `strip_cdata` - replace CDATA sections by normal text content (on by default)
- `resolve_entities` - replace entities by their text value (on by default)
- `huge_tree` - disable security restrictions and support very deep trees and very long text content (only affects libxml2 2.7+)
- `compact` - use compact storage for short text content (on by default)
- `collect_ids` - collect XML IDs in a hash table while parsing (on by default). Disabling this can substantially speed up parsing of documents with many different IDs if the hash lookup is not used afterwards.
Other keyword arguments:
- `encoding` - override the document encoding
- `target` - a parser target object that will receive the parse events (see [The target parser interface](http://lxml.de/parsing.html#the-target-parser-interface))
- `schema` - an XMLSchema to validate against (see [validation](http://lxml.de/validation.html#xmlschema))
```
parser = etree.XMLParser(ns_clean=True)
tree = etree.parse(StringIO(xml), parser)
etree.tostring(tree.getroot()) # b'<a xmlns="test"><b/></a>'
``` 

#### Error log
Parsers have an `error_log` property taht lists the errors and warnings of the last parser run:
```
parser = etree.XMLParser()
print(len(parser.error_log)) # 0
tree = etree.XML("<root>\n</b>", parser) # raise lxml.etree.XMLSyntaxError
print(len(parser.error_log)) # 1
error = parser.error_log[0]
print(error.message)
print(error.line) # 2
print(error.column) # 5
```
Each entry in the log has the following properties:
- `message`: the message text
- `domain`: the domain ID (see the lxml.etree.ErrorDomains class)
- `type`: the message type ID (see the lxml.etree.ErrorTypes class)
- `level`: the log level ID (see the lxml.etree.ErrorLevels class)
- `line`: the line at which the message originated (if applicable)
- `column`: the character column at which the message originated (if applicable)
- `filename`: the name of the file in which the message originated (if applicable)
For convenience, there are also three properties that provide readable names for the ID values:
- domain_name
- type_name
- level_name
To filter for a specific kind of message, use the different `filter_*()` methods on the error log (see the lxml.etree._ListErrorLog class).

#### Parsing HTML
The parsers have a `recover` keyword argument that the HTMLParser sets by default. It lets libxml2 try its best to return a valid HTML tree with all content it can manage to parse. It will not raise an exception on parser errors.
```
broken_html = "<html><head><title>test<body><h1>page title</h3>"
parser = etree.HTMLParser()
tree = etree.parse(StringIO(broken_html), parser)
result = etree.tostring(tree.getroot(), pretty_print=True, method="html")
print(result)
<html>
  <head>
    <title>test</title>
  </head>
  <body>
    <h1>page title</h1>
  </body>
</html>

# lxml has a HTML function, similar to the XML shortcut known from ElementTree:
html = etree.HTML(broken_html)
result = etree.tostring(html, pretty_print=True, method="html")
print(result)
```
XML forbids double hyphens in comments, which the HTML parser will happily accept in recovery mode.

#### Doctype information
```
pub_id = "-//W3C//DTD XHTML 1.0 Transitional//EN"
sys_url = "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"
doctype_string = '<!DOCTYPE html PUBLIC "%s" "%s">' % (pub_id, sys_url)
xml_header = '<?xml version="1.0" encoding="ascii"?>'
xhtml = xml_header + doctype_string + '<html><body></body></html>'
tree = etree.parse(StringIO(xhtml))
docinfo = tree.docinfo
print(docinfo.public_id) # -//W3C//DTD XHTML 1.0 Transitional//EN
print(docinfo.system_url) #http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd
docinfo.doctype == doctype_string # True
print(docinfo.xml_version) # 1.0
print(docinfo.encoding) # ascii
docinfo.system_url = None
docinfo.public_id = None
print(etree.tostring(tree))
<!DOCTYPE html>
<html><body/></html>
```

### The target parser interface
```
class EchoTarget(object):
	def start(self, tag, attrib):
		print("start %s %r" % (tag, dict(attrib)))
	def end(self, tag):
		print("end %s" % tag)
	def data(self, data):
		print("data %r" % data)
	def comment(self, text):
		print("comment %s" % text)
	def close(self):
		print("close")
		return "closed!"
parser = etree.XMLParser(target = EchoTarget())
result = etree.XML(XML("<element>some<!--comment-->text</element>", parser)
start element {}
data u’some’
comment comment
data u’text’
end element
close

print(result)
closed!
```
It is important for the `.close()` method to reset the parser target to a usable state, so that you can reuse the parser as often as you like:
```
result = etree.XML("<element>some<!--comment-->text</element>", parser)
print(result)
```
The `.close()` method will also be called in the error case.
Note that the parser does not build a tree when using a parser target.
```
parser = etree.XMLParser(target = etree.TreeBuilder())
result = etree.XML("<element>some<!--comment-->text</element>", parser)
print(result.tag) # element
print(result[0].text) # comment
```

### The feed parser interface
```
parser = etree.XMLParser()
for data in (’<?xml versio’, ’n="1.0"?’, ’><roo’, ’t><a’, ’/></root>’):
	parser.feed(data)
root = parser.close() # must
print(root.tag); print(root[0].tag) 
```
The feed parser has its own error log called `feed_error_log`. Errors in the feed parser do not show up in the normal `error_log` and vice versa.
You can also combine the feed parser interface with the target parser:
```
parser = etree.XMLParser(target = EchoTarget())
parser.feed("<eleme")
parser.feed("nt>some text</elem")
start element {}
data u’some text’
parser.feed("ent>")
end element
result = parser.close()
close
print(result)
closed!
```

### Incremental event parsing
```
parser = etree.XMLPullParser(events=(’start’, ’end’))
def print_events(parser):
for action, element in parser.read_events():
	print(’%s: %s’ % (action, element.tag))
parser.feed('<root>some text')
print_events(parser)
start: root
print_events(parser) # well, no more events, as before ...
parser.feed(’<child><a />’)
print_events(parser)
start: child
start: a
end: a
parser.feed(’</child></roo’)
print_events(parser)
end: child
parser.feed(’t>’)
print_events(parser)
end: root
root = parser.close()
etree.tostring(root) # b’<root>some text<child><a/></child></root>’
```
#### Event types
The parse events are tuples (event-type, object). The event types supported by ElementTree and lxml.etree are the strings 'start', 'end', 'start-ns' and 'end-ns'. The value of the 'start-ns' event is a tuple (prefix, namespaceURI) that designates the beginning of a prefixnamespace mapping. The 'end-ns' event does not have a value (None).
```
def print_events(events):
	for action, obj in events:
		if action in ('start', 'end'):
			print("%s: %s" % (action, obj.tag))
		elif action == 'start-ns':
			print("%s: %s" % (action, obj))
		else:
			print(action)
event_types = ("start", "end", "start-ns", "end-ns")
parser = etree.XMLPullParser(event_types)
events = parser.read_events()
parser.feed('<root><element>')
print_events(events)
start: root
start: element
parser.feed('text</element><element>text</element>')
print_events(events)
end: element
start: element
end: element
parser.feed('<empty-element xmlns="http://testns/" />')
print_events(events)
start-ns: (’’, ’http://testns/’)
start: {http://testns/}empty-element
end: {http://testns/}empty-element
end-ns
parser.feed(’</root>’)
print_events(events)
end: root
```
#### Modifying the tree
You can modify the element and its descendants when handling the 'end' event.
```
parser = etree.XMLPullParser()
events = parser.read_events()
parser.feed('<root><element key="value">text</element>')
parser.feed('<element><child /></element>')
for action, elem in events:
	print('%s: %d' % (elem.tag, len(elem)))
	elem.clear()  # delete children

parser.feed('<empty-element xmlns="http://testns/" /></root>')
for action, elem in events:
	print('%s: %d' % (elem.tag, len(elem))) 
	elem.clear()

root = parser.close()
etree.tostring(root) # b'<root/>'
```
**WARNING**: During the 'start' event, any content of the element, such as the descendants, following siblings or text, is not yet available and should not be accessed. Only attributes are guaranteed to be set. During the 'end' event, the element and its descendants can be freely modified, but its following siblings should not be accessed. During either of the two events, you must not modify or move the ancestors (parents) of the current element. You should also avoid moving or discarding the element itself. The golden rule is: do not touch anything that will have to be touched again by the parser later on.
```
for event, element in parser.read_events():
	# ... do something with the element
	element.clear() # clean up children
	while element.getprevious() is not None:
		del element.getparent()[0]
```

#### Selective tag events
```
parser = etree.XMLPullParser(tag="element")
parser.feed('<root><element key="value">text</element>')
parser.feed('<element><child /></element>')
parser.feed('<empty-element xmlns="http://testns/" /></root>')
for action, elem in parser.read_events():
	print("%s: %s" % (action, elem.tag))
# end: element
# end: element

event_types = ("start", "end")
parser = etree.XMLPullParser(event_types, tag="{http://testns/}*")
parser.feed('<root><element key="value">text</element>')
parser.feed('<element><child /></element>')
parser.feed('<empty-element xmlns="http://testns/" /></root>')
for action, elem in parser.read_events():
	print("%s: %s" % (action, elem.tag))
# start: {http://testns/}empty-element
# end: {http://testns/}empty-element
```

#### Comments and PIs
The `XMLPullParser` in lxml.etree also supports the event types 'comment' and 'pi' for the respective XML structures.
```
event_types = ("start", "end", "comment", "pi")
parser = etree.XMLPullParser(event_types)
parser.feed('<?some pi ?><!-- a comment --><root>')
parser.feed('<element key="value">text</element>')
parser.feed('<!-- another comment -->')
parser.feed('<element>text</element>tail')
parser.feed('<empty-element xmlns="http://testns/" />')
parser.feed('</root>')
for action, elem in parser.read_events():
	if action in ('start', 'end'):
		print("%s: %s" % (action, elem.tag))
	elif action == 'pi':
		print("%s: -%s=%s-" % (action, elem.target, elem.text))
	else: # 'comment'
		print("%s: -%s-" % (action, elem.text))
# pi: -some=pi -
# comment: - a comment -
# start: root
# start: element
# end: element
# comment: - another comment -
# start: element
# end: element
# start: {http://testns/}empty-element
# end: {http://testns/}empty-element
# end: root
root = parser.close()
print(root.tag)
# root
```

#### Events with custom targets
You can combine the pull parser with a parser target. In that case, it is the target’s responsibility to generate event values.
```
class Target(object):
	def start(self, tag, attrib):
		print('-> start(%s)' % tag)
		return '>>START: %s<<' % tag
	def end(self, tag):
		print('-> end(%s)' % tag)
		return '>>END: %s<<' % tag
	def close(self):
		print('-> close()')
		return "CLOSED!"
event_types = ('start', 'end')
parser = etree.XMLPullParser(event_types, target=Target())
parser.feed('<root><child1 /><child2 /></root>')
# -> start(root)
# -> start(child1)
# -> end(child1)
# -> start(child2)
# -> end(child2)
# -> end(root)
for action, value in parser.read_events():
	print(’%s: %s’ % (action, value))
# start: >>START: root<<
# start: >>START: child1<<
# end: >>END: child1<<
# start: >>START: child2<<
# end: >>END: child2<<
# end: >>END: root<<
print(parser.close())
# -> close()
# CLOSED!
```
You will want to make your custom target inherit from the TreeBuilder class in order to have it build a tree that you can process normally.
```
class AttributeFilter(etree.TreeBuilder):
	def start(self, tag, attrib):
		attrib = dict(attrib)
		if 'evil' in attrib:
			del attrib['evil']
		return super(AttributeFilter, self).start(tag, attrib)
parser = etree.XMLPullParser(target=AttributeFilter())
parser.feed('<root><child1 test="123" /><child2 evil="YES" /></root>')
for action, element in parser.read_events():
	print('%s: %s(%r)' % (action, element.tag, element.attrib))
# end: child1({'test': '123'})
# end: child2({})
# end: root({})
root = parser.close()
```

### iterparse and iterwalk
As known from ElementTree, the `iterparse()` utility function returns an iterator that generates parser events for an XML file (or file-like object), while building the tree.
```
xml = '''
<root>
	<element key='value'>text</element>
	<element>text</element>tail
	<empty-element xmlns="http://testns/" />
</root>
'''
context = etree.iterparse(StringIO(xml))
for action, elem in context:
	print("%s: %s" % (action, elem.tag))
# end: element
# end: element
# end: {http://testns/}empty-element
# end: root

# After parsing, the resulting tree is available through the 'root' property of the iterator:
context.root.tag
# 'root'

# The other event types can be activated with the 'events' keyword argument:
events = ("start", "end")
context = etree.iterparse(StringIO(xml), events=events)
for action, elem in context:
	print("%s: %s" % (action, elem.tag))
# start: root
# start: element
# end: element
# start: element
# end: element
# start: {http://testns/}empty-element
# end: {http://testns/}empty-element
# end: root
```

#### iterwalk
`interwalk()` works on Elements and ElementTrees. It can iterate over the resulting in-memory tree parsed by `iterparse()`. The `iterwalk` iterator can be instructed to skip over an entire subtree with its `.skip_subtree()` method.
```
root = etree.XML('''
<root>
	<a> <b /> </a>
	<c />
</root>
''')
context = etree.iterwalk(root, events=("start", "end"))
for action, elem in context:
	print("%s: %s" % (action, elem.tag))
	if action == 'start' and elem.tag == 'a':
		context.skip_subtree() # ignore <b>
# start: root
# start: a
# end: a
# start: c
# end: c
# end: root
```
Note that `.skip_subtree()` only has an effect when handling start or start-ns events.

### Serialising to Unicode strings
If you want to save the result to a file or pass it over the network, you should use `write()` or `tostring()` with a byte encoding (typically UTF-8) to serialize the XML. The main reason is that unicode strings returned by `tostring(encoding='unicode')` are not byte streams and they never have an XML declaration to specify their encoding. These strings are most likely not parsable by other XML libraries.

## Chapter 10 Validation with lxml
A [document type definition (DTD)](https://en.wikipedia.org/wiki/Document_type_definition) is a set of markup declarations that define a document type for an SGML-family markup language (SGML, XML, HTML).
### [Validation at parse time](http://lxml.de/validation.html#validation-at-parse-time)
### [DTD](http://lxml.de/validation.html#id1)
### [RelaxNG](http://lxml.de/validation.html#relaxng)
### [Schematron](http://lxml.de/validation.html#id2)
### [(Pre-ISO-Schematron)](http://lxml.de/validation.html#id3)

## Chapter 11 XPath and XSLT with lxml
lxml supports XPath 1.0, XSLT 1.0 and the EXSLT extensions through libxml2 and libxslt in a standards compliant way.
### XPath
For more about [XPath](http://www.w3school.com.cn/xpath/).
#### The xpath() method
For ElementTree, the xpath method performs a global XPath query against the document (if absolute) or against the root node (if relative):
```
f = StringIO('<foo><bar></bar></foo>')
tree = etree.parse(f)
r = tree.xpath('/foo/bar')
len(r) # 1
r[0].tag # 'bar'
r = tree.xpath('bar')
r[0].tag # 'bar'
```
When `xpath()` is used on an Element, the XPath expression is evaluated against the element (if relative) or against the root tree (if absolute):
```
root = tree.getroot()
r = root.xpath('bar')
r[0].tag #'bar'

bar = root[0]
r = bar.xpath('/foo/bar')
r[0].tag # 'bar'

tree = bar.getroottree()
r = tree.xpath('/foo/bar')
r[0].tag # 'bar'
```
The `xpath()` method has support for XPath variables:
```
expr = "//*[local-name() = $name]"
print(root.xpath(expr, name = "foo")[0].tag) # foo

print(root.xpath(expr, name = "bar")[0].tag) # bar

print(root.xpath("$text", text = "Hello World!")) # Hello World!
```

#### Namespaces and prefixes
If your XPath expression uses namespace prefixes, you must define them in a prefix mapping. To this end, pass a dictionary to the namespaces keyword argument that maps the namespace prefixes used in the XPath expression to namespace URIs:
```
f = StringIO('''\
<a:foo xmlns:a="http://codespeak.net/ns/test1"
       xmlns:b="http://codespeak.net/ns/test2">
   <b:bar>Text</b:bar>
	</a:foo>
''')
doc = etree.parse(f)

r = doc.xpath('/x:foo/b:bar', namespaces={'x': 'http://codespeak.net/ns/test1', 'b': 'http://codespeak.net/ns/test2'})
len(r) # 1
r[0].tag # '{http://codespeak.net/ns/test2}bar'
r[0].text # 'Text'
```
Note that XPath does not have a notion of a default namespace. The empty prefix is therefore undefined for XPath and cannot be used in namespace prefix mappings.

#### XPath return values
The return value types of XPath evaluations vary, depending on the XPath expression used:
- True or False, when the XPath expression has a boolean result
- a float, when the XPath expression has a numeric result (integer or float)
- a 'smart' string (as described below), when the XPath expression has a string result.
- a list of items, when the XPath expression has a list as result. The items may include Elements (also comments and processing instructions), strings and tuples. Text nodes and attributes in the result are returned as 'smart' string values. Namespace declarations are returned as tuples of strings: (prefix, URI).

XPath string results are 'smart' in that they provide a `getparent()` method that knows their origin:
- for attribute values, `result.getparent()` returns the Element that carries them. An example is `//foo/@attribute`, where the parent would be a foo Element.
- for the `text()` function (as in `//text()`), it returns the Element that contains the text or tail that was returned.

You can distinguish between different text origins with the boolean properties is_text, is_tail and is_attribute.
Note that getparent() may not always return an Element. For example, the XPath functions string() and concat() will construct strings that do not have an origin. For them, getparent() will return None.
```
root = etree.XML("<root><a>TEXT</a></root>")
find_text = etree.XPath("//text()")
text = find_text(root)[0]
print(text) # TEXT
print(text.getparent().text) # TEXT

find_text = etree.XPath("//text()", smart_strings=False)
text = find_text(root)[0]
print(text) # TEXT
hasattr(text, 'getparent') # False
```

#### Generating XPath expressions
ElementTree objects have a method `getpath(element)`, which returns a structural, absolute XPath expression to find that element:
```
a  = etree.Element("a")
b  = etree.SubElement(a, "b")
c  = etree.SubElement(a, "c")
d1 = etree.SubElement(c, "d")
d2 = etree.SubElement(c, "d")

tree = etree.ElementTree(c)
print(tree.getpath(d2)) # /c/d[2]
tree.xpath(tree.getpath(d2)) == [d2] # True
```

#### The `XPath` class
The `XPath` class compiles an XPath expression into a callable function:
```
root = etree.XML("<root><a><b/></a><b/></root>")
find = etree.XPath("//b")
print(find(root)[0].tag) # b
```
Just like the `xpath()` method, the `XPath` class supports XPath variables. Prefix-to-namespace mappings can be passed as second parameter:
```
root = etree.XML("<root xmlns='NS'><a><b/></a><b/></root>")
find = etree.XPath("//n:b", namespaces={'n':'NS'})
print(find(root)[0].tag) # {NS}b
```

#### Regular expressions in XPath
By default, XPath supports regular expressions in the [**EXSLT**](http://www.exslt.org/) namespace:
```
regexpNS = "http://exslt.org/regular-expressions"
find = etree.XPath("//*[re:test(., '^abc$', 'i')]", namespaces={'re':regexpNS})

root = etree.XML("<root><a>aB</a><b>aBc</b></root>")
print(find(root)[0].text) # aBc
```

#### The XPathEvaluator classes
`lxml.etree` provides two other efficient XPath evaluators that work on ElementTrees or Elements respectively: `XPathDocumentEvaluator` and `XPathElementEvaluator`. They are automatically selected if you use the `XPathEvaluator` helper for instantiation:
```
root = etree.XML("<root><a><b/></a><b/></root>")
xpatheval = etree.XPathEvaluator(root)
print(isinstance(xpatheval, etree.XPathElementEvaluator)) # True
print(xpatheval("//b")[0].tag) # b
```

#### ETXPath
One of the main differences between XPath and ElementPath is that the XPath language requires an indirection through prefixes for namespace support, whereas ElementTree uses the Clark notation ({ns}name) to avoid prefixes completely. The other major difference regards the capabilities of both path languages. Where XPath supports various sophisticated ways of restricting the result set through functions and boolean expressions, ElementPath only supports pure path traversal without nesting or further conditions. So, while the ElementPath syntax is self-contained and therefore easier to write and handle, XPath is much more powerful and expressive.
lxml.etree bridges this gap through the class ETXPath, which accepts XPath expressions with namespaces in Clark notation. It is identical to the XPath class, except for the namespace notation. Normally, you would write:
```
root = etree.XML("<root xmlns='ns'><a><b/></a><b/></root>")
find = etree.XPath("//p:b", namespaces={'p' : 'ns'})
print(find(root)[0].tag) # {ns}b
```
ETXPath allows you to change this to:
```
find = etree.ETXPath("//{ns}b")
print(find(root)[0].tag) # {ns}b
```

#### Error handling
lxml.etree raises exceptions when errors occur while parsing or evaluating an XPath expression. lxml will also try to give you a hint what went wrong, so if you pass a more complex expression. During evaluation, lxml will emit an XPathEvalError on errors.

### [XSLT](http://www.w3school.com.cn/xsl/)
lxml.etree introduces a new class, `lxml.etree.XSLT`. The class can be given an ElementTree or Element object to construct an XSLT transformer:
```
xslt_root = etree.XML('''\
<xsl:stylesheet version="1.0"
	xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:template match="/">
		<foo><xsl:value-of select="/a/b/text()" /></foo>
	</xsl:template>
</xsl:stylesheet>''')
transform = etree.XSLT(xslt_root)
```
You can then run the transformation on an ElementTree document by simply calling it, and this results in another ElementTree object:
```
f = StringIO('<a><b>Text</b></a>')
doc = etree.parse(f)
result_tree = transform(doc)
```
By default, XSLT supports all extension functions from libxslt and libexslt as well as Python regular expressions through the [EXSLT regexp functions](http://www.exslt.org/regexp/).

#### XSLT result objects
The result of an XSL transformation can be accessed like a normal ElementTree document:
```
 root = etree.XML('<a><b>Text</b></a>')
result = transform(root)
result.getroot().text # 'Text'
```
but, as opposed to normal ElementTree objects, can also be turned into an (XML or text) string by applying the str() function:
```
str(result)
# '<?xml version="1.0"?>\n<foo>Text</foo>\n'
```
You can use other encodings at the cost of multiple recoding. Encodings that are not supported by Python will result in an error. While it is possible to use the `.write()` method (known from ElementTree objects) to serialise the XSLT result into a file, it is better to use the `.write_output()` method. The latter knows about the `<xsl:output>` tag and writes the expected data into the output file.
```
 xslt_root = etree.XML('''\
<xsl:stylesheet version="1.0"
	xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:output method="text" encoding="utf8" />
	<xsl:template match="/">
		<foo><xsl:value-of select="/a/b/text()" /></foo>
	</xsl:template>
</xsl:stylesheet>''')
transform = etree.XSLT(xslt_root)
result = transform(doc)
result.write_output("output.txt.gz", compression=9) # doctest: +SKIP
```

#### Stylesheet parameters
It is possible to pass parameters, in the form of XPath expressions, to the XSLT template:
```
xslt_tree = etree.XML('''\
<xsl:stylesheet version="1.0"
	xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:param name="a" />
	<xsl:template match="/">
		<foo><xsl:value-of select="$a" /></foo>
	</xsl:template>
</xsl:stylesheet>''')
transform = etree.XSLT(xslt_tree)
doc_root = etree.XML('<a><b>Text</b></a>')

result = transform(doc_root, a="5")
str(result)
# '<?xml version="1.0"?>\n<foo>5</foo>\n'

result = transform(doc_root, a="/a/b/text()")
str(result)
# '<?xml version="1.0"?>\n<foo>Text</foo>\n'

result = transform(doc_root, a="'A'")
str(result)
# '<?xml version="1.0"?>\n<foo>A</foo>\n'

# To pass a string that (potentially) contains quotes, you can use the .strparam() class method. 
plain_string_value = etree.XSLT.strparam(""" It's "Monty Python" """)
result = transform(doc_root, a=plain_string_value)
str(result)
# '<?xml version="1.0"?>\n<foo> It\'s "Monty Python" </foo>\n'

# to pass parameters that are not legal Python identifiers, pass them inside of a dictionary:
transform = etree.XSLT(etree.XML('''\
<xsl:stylesheet version="1.0"
	xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:param name="non-python-identifier" />
	<xsl:template match="/">
		<foo><xsl:value-of select="$non-python-identifier" /></foo>
	</xsl:template>
</xsl:stylesheet>'''))

result = transform(doc_root, **{'non-python-identifier': '5'})
str(result)
# '<?xml version="1.0"?>\n<foo>5</foo>\n'
```

#### Errors and messages
 XSLT provides an error log that lists messages and error output from the last run. 
```
xslt_root = etree.XML('''\
<xsl:stylesheet version="1.0"
	xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:template match="/">
		<xsl:message terminate="no">STARTING</xsl:message>
		<foo><xsl:value-of select="/a/b/text()" /></foo>
		<xsl:message terminate="no">DONE</xsl:message>
	</xsl:template>
</xsl:stylesheet>''')
transform = etree.XSLT(xslt_root)
doc_root = etree.XML('<a><b>Text</b></a>')
result = transform(doc_root)
str(result)
# '<?xml version="1.0"?>\n<foo>Text</foo>\n'

print(transform.error_log)
# <string>:0:0:ERROR:XSLT:ERR_OK: STARTING
# <string>:0:0:ERROR:XSLT:ERR_OK: DONE

for entry in transform.error_log:
	print('message from line %s, col %s: %s' % (entry.line, entry.column, entry.message))
	print('domain: %s (%d)' % (entry.domain_name, entry.domain))
	print('type: %s (%d)' % (entry.type_name, entry.type))
	print('level: %s (%d)' % (entry.level_name, entry.level))
	print('filename: %s' % entry.filename)
# message from line 0, col 0: STARTING
# domain: XSLT (22)
# type: ERR_OK (0)
# level: ERROR (2)
# filename: <string>
# message from line 0, col 0: DONE
# domain: XSLT (22)
# type: ERR_OK (0)
# level: ERROR (2)
# filename: <string>
```
Note that there is no way in XSLT to distinguish between user messages, warnings and error messages that occurred during the run.

#### The xslt() tree method
```
result = doc.xslt(xslt_tree, a="'A'")
str(result)
# '<?xml version="1.0"?>\n<foo>A</foo>\n'
transform = etree.XSLT(xslt_tree)
result = transform(doc, a="'A'")
str(result)
# '<?xml version="1.0"?>\n<foo>A</foo>\n'
```

#### Dealing with stylesheet complexity
Some applications require a larger set of rather diverse stylesheets. lxml.etree allows you to deal with this in a number of ways. Here are some ideas to try.
The most simple way to reduce the diversity is by using XSLT parameters that you pass at call time to configure the stylesheets.
You may also consider creating stylesheets programmatically.
A third thing to remember is the support for [custom extension functions](http://lxml.de/extensions.html#xpath-extension-functions) and [XSLT extension elements](http://lxml.de/extensions.html#xslt-extension-elements).

#### Profiling
If you want to know how your stylesheet performed, pass the profile_run keyword to the transform:
```
result = transform(doc, a="/a/b/text()", profile_run=True)
profile = result.xslt_profile

del result.xslt_profile
```

## Chapter 12 lxml.objectify
lxml supports an alternative API similar to the [Amara](http://uche.ogbuji.net/tech/4suite/amara/) bindery or [gnosis.xml.objectify](http://gnosis.cx/download/) through a [custom Element implementation](http://lxml.de/element_classes.html). 
```
from lxml import objectify
import lxml.usedoctest
```
### [The lxml.objectify API](http://lxml.de/objectify.html#the-lxml-objectify-api)
### [Asserting a Schema](http://lxml.de/objectify.html#asserting-a-schema)
### [ObjectPath](http://lxml.de/objectify.html#objectpath)
### [Python data types](http://lxml.de/objectify.html#python-data-types)
### [How data types are matched](http://lxml.de/objectify.html#how-data-types-are-matched)
### [What is different from lxml.etree?](http://lxml.de/objectify.html#what-is-different-from-lxml-etree)

## Chapter 13 lxml.html
### Parsing HTML
#### Parsing HTML fragments
There are several functions available to parse HTML:
- parse(filename_url_or_file)
- document_fromstring(string)
- fragment_fromstring(string, create_parent=False)
- fragments_fromstring(string)
- fromstring(string)

#### Really broken pages
A way to deal with this is [ElementSoup](http://lxml.de/elementsoup.html), which deploys the well-known [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/) parser to build an lxml HTML tree.

### HTML Element Methods
HTML elements have all the methods that come with ElementTree, but also include some extra methods:
- .drop_tree()
- .drop_tag()
- .find_class(class_name)
- .find_rel_links(rel)
- .get_element_by_id(id, default=None)
- .text_content()
- .cssselect(expr)
- .label
- .base_url
- .classes
- .set(key, value=None)

### Running HTML doctests
lxml provides the `lxml.doctestcompare` module that supports relaxed comparison of XML and HTML pages and provides a readable diff in the output when a test fails. The HTML comparison is most easily used by importing the `usedoctest` module in a doctest:
```
import lxml.html.usedoctest
import lxml.html
html = lxml.html.fromstring('''\
<html><body onload="" color="white">
	<p>Hi  !</p>
</body></html>
''')
print(lxml.html.tostring(html))
# <html><body onload="" color="white"><p>Hi !</p></body></html>
```

### Creating HTML with the E-factory
lxml.html comes with a predefined HTML vocabulary for the [E-factory](http://online.effbot.org/2006_11_01_archive.htm#et-builder).
```
from lxml.html import builder as E
from lxml.html import usedoctest
html = E.HTML(
	E.HEAD(
		E.LINK(rel="stylesheet", href="great.css", type="text/css"),
		E.TITLE("Best Page Ever")
	),
	E.BODY(
		E.H1(E.CLASS("heading"), "Top News"),
		E.P("World News only on this page", style="font-size: 200%"),
		"Ah, and here's some more text, by the way.",
		lxml.html.fromstring("<p>... and this is a parsed fragment ...</p>")
	)
)

print lxml.html.tostring(html)
'''
<html>
  <head>
    <link href="great.css" rel="stylesheet" type="text/css">
    <title>Best Page Ever</title>
  </head>
  <body>
    <h1 class="heading">Top News</h1>
    <p style="font-size: 200%">World News only on this page</p>
    Ah, and here's some more text, by the way.
    <p>... and this is a parsed fragment ...</p>
  </body>
</html>
```

#### Viewing your HTML
A handy method for viewing your HTML: `lxml.html.open_in_browser(lxml_doc)` will write the document to disk and open it in a browser (with the [webbrowser module](http://python.org/doc/current/lib/module-webbrowser.html)).

### Working with links
There are several methods on elements that allow you to see and modify the links in a document.
- .iterlinks()
- .resolve_base_href()
- .make_links_absolute(base_href, resolve_base_href=True)
- .rewrite_links(link_repl_func, resolve_base_href=True, base_href=None)

#### Functions
In addition to these methods, there are corresponding functions:
- iterlinks(html)
- make_links_absolute(html, base_href, ...)
- rewrite_links(html, link_repl_func, ...)
- resolve_base_href(html)

These functions will parse html if it is a string, then return the new HTML as a string. If you pass in a document, the document will be copied (except for `iterlinks()`), the method performed, and the new document returned.

### Forms
Any <form> elements in a document are available through the list doc.forms (e.g., doc.forms[0]). **Form**, **input**, **select**, and **textarea** elements each have special methods.
Input elements (including 'select' and 'textarea') have these attributes:
- .name
- .value

Select attributes:
- .value_options
- .multiple

Input attributes:
- .type()
- .checkable()
- .checked()

The form itself has these attributes:
- .inputs
- .fields
- .form_values()
- .action
- .method

#### Form Filling Example
```
from lxml.html import fromstring, tostring
form_page = fromstring('''<html><body><form>
	Your name: <input type="text" name="name"> <br>
	Your phone: <input type="text" name="phone"> <br>
	Your favorite pets: <br>
	Dogs: <input type="checkbox" name="interest" value="dogs"> <br>
	Cats: <input type="checkbox" name="interest" value="cats"> <br>
	Llamas: <input type="checkbox" name="interest" value="llamas"> <br>
<input type="submit"></form></body></html>''')
form = form_page.forms[0]
form.fields = dict(
	name='John Smith',
	phone='555-555-3949',
	interest=set(['cats', 'llamas']))
print tostring(form)
'''
<html>
  <body>
    <form>
    Your name:
      <input name="name" type="text" value="John Smith">
      <br>Your phone:
      <input name="phone" type="text" value="555-555-3949">
      <br>Your favorite pets:
      <br>Dogs:
      <input name="interest" type="checkbox" value="dogs">
      <br>Cats:
      <input checked name="interest" type="checkbox" value="cats">
      <br>Llamas:
      <input checked name="interest" type="checkbox" value="llamas">
      <br>
      <input type="submit">
    </form>
  </body>
</html>
'''
```

#### Form Submission
You can submit a form with `lxml.html.submit_form(form_element)`. This will return a file-like object (the result of urllib.urlopen()).
If you have extra input values you want to pass you can use the keyword argument `extra_values`, like extra_values={'submit': 'Yes!'}. This is the only way to get submit values into the form, as there is no state of "submitted" for these elements.
You can pass in an alternate opener with the `open_http` keyword argument, which is a function with the signature open_http(method, url, values).
Example:
```
from lxml.html import parse, submit_form
page = parse('http://tinyurl.com').getroot()
page.forms[0].fields['url'] = 'http://lxml.de/'
result = parse(submit_form(page.forms[0])).getroot()
[a.attrib['href'] for a in result.xpath("//a[@target='_blank']")]
# ['http://tinyurl.com/2xae8s', 'http://preview.tinyurl.com/2xae8s']
```

### Cleaning uo HTML
The module lxml.html.clean provides a Cleaner class for cleaning up HTML pages. It supports removing embedded or script content, special tags, CSS style annotations and much more.
An evil web page:
```
html = '''\
<html>
  <head>
    <script type="text/javascript" src="evil-site"></script>
    <link rel="alternate" type="text/rss" src="evil-rss">
    <style>
      body {background-image: url(javascript:do_evil)};
      div {color: expression(evil)};
    </style>
  </head>
  <body onload="evil_function()">
    <!-- I am interpreted for EVIL! -->
    <a href="javascript:evil_function()">a link</a>
    <a href="#" onclick="evil_function()">another link</a>
    <p onclick="evil_function()">a paragraph</p>
    <div style="display: none">secret EVIL!</div>
    <object> of EVIL! </object>
    <iframe src="evil-site"></iframe>
    <form action="evil-site">
      Password: <input type="password" name="password">
    </form>
    <blink>annoying EVIL!</blink>
    <a href="evil-site">spam spam SPAM!</a>
    <image src="evil!">
  </body>
</html>'''
```
To remove the all suspicious content from this unparsed document, use the clean_html function:
```
from lxml.html.clean import clean_html
print(clean_html(html))
'''
<div><style>/* deleted */</style><body>

   <a href="">a link</a>
   <a href="#">another link</a>
   <p>a paragraph</p>
   <div>secret EVIL!</div>
    of EVIL!


     Password:
   annoying EVIL!<a href="evil-site">spam spam SPAM!</a>
   <img src="evil!"></body></div>
```
The Cleaner class supports several keyword arguments to control exactly which content is removed:
```
from lxml.html.clean import Cleaner
cleaner = Cleaner(page_structure=False, links=False)
print(cleaner.clean_html(html))
'''
<html>
  <head>
    <link rel="alternate" src="evil-rss" type="text/rss">
    <style>/* deleted */</style>
  </head>
  <body>
    <a href="">a link</a>
    <a href="#">another link</a>
    <p>a paragraph</p>
    <div>secret EVIL!</div>
    of EVIL!
    Password:
    annoying EVIL!
    <a href="evil-site">spam spam SPAM!</a>
    <img src="evil!">
  </body>
</html>
'''
cleaner = Cleaner(style=True, links=True, add_nofollow=True, page_structure=False, safe_attrs_only=False)
print(cleaner.clean_html(html))
'''
<html>
  <head>
  </head>
  <body>
    <a href="">a link</a>
    <a href="#">another link</a>
    <p>a paragraph</p>
    <div>secret EVIL!</div>
    of EVIL!
    Password:
    annoying EVIL!
    <a href="evil-site" rel="nofollow">spam spam SPAM!</a>
    <img src="evil!">
  </body>
</html>
'''
```
You can also whitelist some otherwise dangerous content with **Cleaner(host_whitelist=['www.youtube.com'])**, which would allow embedded media from YouTube, while still filtering out embedded media from other sites.
See the docstring of Cleaner for the details of what can be cleaned.

#### autolink
In addition to cleaning up malicious HTML, lxml.html.clean contains functions to do other things to your HTML. This includes autolinking:
```
autolink(doc, ...)
autolink_html(html, ...)
```
This finds anything that looks like a link (e.g., http://example.com) in the text of an HTML document, and turns it into an anchor. It avoids making bad links.
Links in the elements 'textarea', 'pre', 'code', anything in the head of the document. You can pass in a list of elements to avoid in avoid_elements=['textarea', ...]
Links to some hosts can be avoided. By default links to localhost*, example.* and 127.0.0.1 are not autolinked. Pass in avoid_hosts=[list_of_regexes] to control this.
Elements with the nolink CSS class are not autolinked. Pass in avoid_classes=['code', ...] to control this.
The autolink_html() version of the function parses the HTML string first, and returns a string.

#### wordwrap
You can also wrap long words in your html:
```
word_break(doc, max_width=40, ...)
word_break_html(html, ...)
```
This finds any long words in the text of the document and inserts &#8203; in the document (which is the Unicode zero-width space).
This avoids the elements 'pre', 'textarea', and 'code'. You can control this with avoid_elements=['textarea', ...].
It also avoids elements with the CSS class nobreak. You can control this with avoid_classes=['code', ...].
Lastly you can control the character that is inserted with break_character=u'\u200b'. However, you cannot insert markup, only text.
word_break_html(html) parses the HTML document and returns a string.

### HTML Diff
The module lxml.html.diff offers some ways to visualize differences in HTML documents. 
There are two ways to view differences: htmldiff and html_annotate. One shows differences with `<ins>` and `<del>`, while the other annotates a set of changes similar to svn blame. Both these functions operate on text, and work best with content fragments (only what goes in <body>), not complete documents.
Example of htmldiff:
```
from lxml.html.diff import htmldiff, html_annotate
doc1 = '''<p>Here is some text.</p>'''
doc2 = '''<p>Here is <b>a lot</b> of <i>text</i>.</p>'''
doc3 = '''<p>Here is <b>a little</b> <i>text</i>.</p>'''
print(htmldiff(doc1, doc2))
<p>Here is <ins><b>a lot</b> of <i>text</i>.</ins> <del>some text.</del> </p>
print(html_annotate([(doc1, 'author1'), (doc2, 'author2'), (doc3, 'author3')]))
<p><span title="author1">Here is</span>
   <b><span title="author2">a</span>
   <span title="author3">little</span></b>
   <i><span title="author2">text</span></i>
   <span title="author2">.</span></p>
```
The `html_annotate` function can also take an optional second argument, `markup`. This is a function like markup(text, version) that returns the given text marked up with the given version. The default version, the output of which you see in the example, looks like:
```
def default_markup(text, version):
    return '<span title="%s">%s</span>' % (
        cgi.escape(unicode(version), 1), text)
```

### Examples
#### Microformat Example
This example parses the [hCard](http://microformats.org/wiki/hcard) microformat.
First we get the page:
```
import urllib
from lxml.html import fromstring
url = 'http://microformats.org/'
content = urllib.urlopen(url).read()
doc = fromstring(content)
doc.make_links_absolute(url)
```
Then we create some objects to put the information in:
```
class Card(object):
	def __init__(self, **kw):
		for name, value in kw:
		setattr(self, name, value)
class Phone(object):
	def __init__(self, phone, types=()):
		self.phone, self.types = phone, types
```
And some generally handy functions for microformats:
```
def get_text(el, class_name):
	els = el.find_class(class_name)
	if els:
		return els[0].text_content()
	else:
		return ''
def get_value(el):
	return get_text(el, 'value') or el.text_content()
def get_all_texts(el, class_name):
	return [e.text_content() for e in els.find_class(class_name)]
def parse_addresses(el):
	# Ideally this would parse street, etc.
	return el.find_class('adr')
```
Then the parsing:
```
for el in doc.find_class('hcard'):
	card = Card()
	card.el = el
	card.fn = get_text(el, 'fn')
	card.tels = []
	for tel_el in card.find_class('tel'):
		card.tels.append(Phone(get_value(tel_el), get_all_texts(tel_el, 'type')))
	card.addresses = parse_addresses(el)
```

## Chapter 14 lxml.cssselect
lxml supports a number of interesting languages for tree traversal and element selection. The most important is obviously **XPath**, but there is also **ObjectPath** in the `lxml.objectify` module. The newest child of this family is [**CSS selection**](http://www.w3.org/TR/CSS21/selector.html), which is made available in form of the lxml.cssselect module.
Although it started its life in lxml, [**cssselect**](https://cssselect.readthedocs.io/en/latest/) is now an independent project. It translates CSS selectors to XPath 1.0 expressions that can be used with lxml's XPath engine. lxml.cssselect adds a few convenience shortcuts into that package.

### The CSSSelector class
The most important class in the `lxml.cssselect` module is `CSSSelector`. It provides the same interface as the XPath class, but accepts a CSS selector expression as input:
```
from lxml.cssselect import CSSSelector
sel = CSSSelector('div.content')
sel  #doctest: +ELLIPSIS
# <CSSSelector ... for 'div.content'>
sel.css
# 'div.content'
sel.path
# "descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' content ')]"
```
To use the selector, simply call it with a document or element object:
```
from lxml.etree import fromstring
h = fromstring('''<div id="outer">
<div id="inner" class="content body">
	text
</div></div>''')
[e.get('id') for e in sel(h)]
# ['inner']
```
Using `CSSSelector` is equivalent to translating with cssselect and using the XPath class:
```
from cssselect import GenericTranslator
from lxml.etree import XPath
sel = XPath(GenericTranslator().css_to_xpath('div.content'))
```
CSSSelector takes a `translator` parameter to let you choose which translator to use. It can be 'xml' (the default), 'xhtml', 'html' or a [**Translator object**](http://packages.python.org/cssselect/#cssselect.GenericTranslator).

### The cssselect method
lxml Element objects have a `cssselect` convenience method.
`h.cssselect('div.content') == sel(h) # True`
The method also accepts a translator parameter. On HtmlElement objects, the default is changed to 'html'.

### Supported Selectors
Most [Level 3](http://www.w3.org/TR/2011/REC-css3-selectors-20110929/) selectors are supported. The details are in the [cssselect documentation](http://packages.python.org/cssselect/#supported-selectors).

### Namespaces
In CSS you can use `namespace-prefix|element`, similar to `namespace-prefix:element` in an XPath expression. In fact, it maps one-to-one, and the same rules are used to map namespace prefixes to namespace URIs: the CSSSelector class accepts a dictionary as its namespaces argument.

## Chapter15 BeautifulSoup Parser
A very nice feature of BeautifulSoup is its excellent [support for encoding detection](http://www.crummy.com/software/BeautifulSoup/bs4/doc/#unicode-dammit) which can provide better results for real-world HTML pages that do not (correctly) declare their encoding.
lxml interfaces with BeautifulSoup through the `lxml.html.soupparser` module. It provides three main functions: `fromstring()` and `parse()` to parse a string or file using BeautifulSoup into an lxml.html document, and `convert_tree()` to convert an existing BeautifulSoup tree into a list of top-level Elements.

### Parsing with the soupparser
The functions fromstring() and parse() behave as known from lxml. The first returns a root Element, the latter returns an ElementTree.
```
tag_soup = '''
<meta/><head><title>Hello</head><body onload=crash()>Hi all<p>'''
from lxml.html.soupparser import fromstring
root = fromstring(tag_soup)
from lxml.etree import tostring
print(tostring(root, pretty_print=True).strip())
'''
<html>
  <meta/>
  <head>
    <title>Hello</title>
  </head>
  <body onload="crash()">Hi all<p/></body>
</html>
'''
```
To control how Element objects are created during the conversion of the tree, you can pass a makeelement factory function to parse() and fromstring(). By default, this is based on the HTML parser defined in lxml.html.

### Entity handling
By default, the BeautifulSoup parser also replaces the entities it finds by their character equivalent.
```
tag_soup = '<body>&copy;&euro;&#45;&#245;&#445;<p>'
body = fromstring(tag_soup).find('.//body')
body.text # u'\xa9\u20ac-\xf5\u01bd'

tostring(body)
# '<body>&#169;&#8364;-&#245;&#445;<p/></body>'
tostring(body, method="html")
# '<body>&#169;&#8364;-&#245;&#445;<p></p></body>'
```

### Using soupparser as a fallback
The downside of using this parser is that it is much slower than the C implemented HTML parser of libxml2 that lxml uses.
```
 tag_soup = '''\
<meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
<html>
	<head>
		<title>Hello W\xc3\xb6rld!</title>
	</head>
	<body>Hi all</body>
</html>'''

import lxml.html
import lxml.html.soupparser

root = lxml.html.fromstring(tag_soup)
try:
	ignore = tostring(root, encoding='unicode')
except UnicodeDecodeError:
	root = lxml.html.soupparser.fromstring(tag_soup)
```

### Using only the encoding detection
Even if you prefer lxml's fast HTML parser, you can still benefit from BeautifulSoup's [support for encoding detection](http://www.crummy.com/software/BeautifulSoup/bs4/doc/#unicode-dammit) in the UnicodeDammit class. Once it succeeds in decoding the data, you can simply pass the resulting Unicode string into lxml's parser.
```
try:
	from bs4 import UnicodeDammit
	def decode_html(html_string):
		converted = UnicodeDammit(html_string)
		if not converted.unicode_markup:
			raise UnicodeDecodeError("Failed to detect encoding, tried [%s]",
			', '.join(converted.tried_encodings))
		# print converted.original_encoding
		return converted.unicode_markup
except ImportError:
	from BeautifulSoup import UnicodeDammit # BeautifulSoup 3
	def decode_html(html_string):
		converted = UnicodeDammit(html_string, isHTML=True)
		if not converted.unicode:
			raise UnicodeDecodeError("Failed to detect encoding, tried [%s]",
			', '.join(converted.triedEncodings))
		# print converted.originalEncoding
		return converted.unicode

root = lxml.html.fromstring(decode_html(tag_soup))
```

## Chapter 16 html5lib Parser
[html5lib](http://code.google.com/p/html5lib/) is a Python package that implements the HTML5 parsing algorithm which is heavily influenced by current browsers and based on the [WHATWG HTML5 specification](http://www.whatwg.org/specs/web-apps/current-work/).
lxml can benefit from the parsing capabilities of html5lib through the lxml.html.html5parser module. It provides a similar interface to the lxml.html module by providing fromstring(), parse(), document_fromstring(), fragment_fromstring() and fragments_fromstring() that work like the regular html parsing functions.

### Differences to regular HTML parsing
html5lib normalizes some elements and element structures to a common format. For example even if a tables does not have a tbody html5lib will inject one automatically:
```
from lxml.html import tostring, html5parser
tostring(html5parser.fromstring("<table><td>foo"))
'<table><tbody><tr><td>foo</td></tr></tbody></table>'
```

### Function Reference
- parse(filename_url_or_file)
- document_fromstring(html, guess_charset=True)
- fragment_fromstring(string, create_parent=False, guess_charset=False)
- fragments_fromstring(string, no_leading_text=False, parser=None)
- fromstring(string)

# Part III Extending lxml
## Chapter 17 [Document loading and URL resolving](http://lxml.de/resolvers.html)
## Chapter 18 [Python extensions for XPath and XSLT](http://lxml.de/extensions.html)
##  Chapter 19 [Using custom Element classes in lxml](http://lxml.de/element_classes.html)
## Chapter 20 [Sax support](http://lxml.de/sax.html)
## Chapter 21 [The public C-API of lxml.etree](http://lxml.de/capi.html)

# Part IV Developing lxml
## Chapter 22 [How to build lxml from source](http://lxml.de/build.html)
## Chapter 23 [How to read the source of lxml](http://lxml.de/lxml-source-howto.html)
## Chapter 24 Credits