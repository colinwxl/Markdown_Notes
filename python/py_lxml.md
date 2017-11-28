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

