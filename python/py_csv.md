# python [csv](https://docs.python.org/3.6/library/csv.html)
<2017-1127> by Colin Wu (colinabc@qq.com)

**Version: python3.6**

CSV: Comma Separated Values
## Module Contents
### csv.reader(csvfile, dialect='excel', **fmtparams)
csvfile can be any object which supports the iterator protocol and returns a string each time its `__next__()` method is called — file objects and list objects are both suitable. If csvfile is a file object, it should be opened with `newline=''`. An optional dialect parameter can be given which is used to define a set of parameters specific to a particular CSV dialect. It may be an instance of a subclass of the [`Dialect`](https://docs.python.org/3.6/library/csv.html#csv.Dialect) class or one of the strings returned by the [`list_dialects()`](https://docs.python.org/3.6/library/csv.html#csv.list_dialects) function. The other optional fmtparams keyword arguments can be given to override individual formatting parameters in the current dialect. For full details about the dialect and formatting parameters, see section [Dialects and Formatting Parameters](https://docs.python.org/3.6/library/csv.html#csv-fmt-params).
> A dialect is a subclass of the Dialect class having a set of specific methods and a single `validate()` method. When creating `reader` or `writer` objects, the programmer can specify a string or a subclass of the `Dialect` class as the dialect parameter. In addition to, or instead of, the dialect parameter, the programmer can also specify individual formatting parameters, which have the same names as the attributes defined below for the `Dialect` class.

`delimiter=','`; `doublequote=True`; `escapechar=None`; `lineterminator='\r\n'`: the string used to terminate lines produced by the writer; `quotechar='"'`; `quoting=QUOTE_MINIMAL`; `skipinitialspace=False`: when True, whitespace immediately following the delimiter is ignored; `strict=False`: when True, raise exception Error on bad CSV input.

```
>>> import csv
>>> with open('eggs.csv', newline='') as csvfile:
...     spamreader = csv.reader(csvfile, delimiter=' ', quotechar='|')
...     for row in spamreader:
...         print(', '.join(row))
Spam, Spam, Spam, Spam, Spam, Baked Beans
Spam, Lovely Spam, Wonderful Spam
```

### csv.writer(csvfile, dialect='excel', **fmtparams)
csvfile can be any object with a write() method. If csvfile is a file object, it should be opened with `newline=''`. To make it as easy as possible to interface with modules which implement the DB API, the value None is written as the empty string.
```
import csv
with open('eggs.csv', 'w', newline='') as csvfile:
    spamwriter = csv.writer(csvfile, delimiter=' ',
                            quotechar='|', quoting=csv.QUOTE_MINIMAL)
    spamwriter.writerow(['Spam'] * 5 + ['Baked Beans'])
    spamwriter.writerow(['Spam', 'Lovely Spam', 'Wonderful Spam'])
```

### csv.register_dialect(name[, dialect[, **fmtparams]])
用来自定义格式的
Associate dialect with name. name must be a string. The dialect can be specified either by passing a sub-class of `Dialect`, or by fmtparams keyword arguments, or both, with keyword arguments overriding parameters of the dialect. For full details about the dialect and formatting parameters, see section Dialects and Formatting Parameters.

### csv.unregister_dialect(name)
Delete the dialect associated with name from the dialect registry. An `Error` is raised if name is not a registered dialect name.

### csv.get_dialect(name)
Return the dialect associated with name. An `Error` is raised if name is not a registered dialect name. This function returns an immutable `Dialect`.

### csv.list_dialect
Return the names of all registered dialects.
```
>>> csv.list_dialects()
['excel', 'excel-tab', 'unix']
```

### csv.field_size_limit([new_limit])
Returns the current maximum field size allowed by the parser. If new_limit is given, this becomes the new limit.

### class csv.DictReader(f, fieldnames=None, restkey=None, restval=None, dialect='excel', *args, **kwds)
Create an object that operates like a regular reader but maps the information in each row to an [`OrderedDict`](https://docs.python.org/3.6/library/collections.html#collections.OrderedDict) whose keys are given by the optional fieldnames parameter.
The fieldnames parameter is a sequence. If fieldnames is omitted, the values in the first row of file f will be used as the fieldnames. Regardless of how the fieldnames are determined, the ordered dictionary preserves their original ordering.
If a row has more fields than fieldnames, the remaining data is put in a list and stored with the fieldname specified by restkey (which defaults to `None`). If a non-blank row has fewer fields than fieldnames, the missing values are filled-in with None.
All other optional or keyword arguments are passed to the underlying [reader](https://docs.python.org/3.6/library/csv.html#csv.reader) instance.
```
>>> import csv
>>> with open('names.csv', newline='') as csvfile:
...     reader = csv.DictReader(csvfile)
...     for row in reader:
...         print(row['first_name'], row['last_name'])
...
Eric Idle
John Cleese

>>> print(row)
OrderedDict([('first_name', 'John'), ('last_name', 'Cleese')])
```

### class csv.DictWriter(f, fieldnames, restval='', extrasaction='raise', dialect='excel', *args, **kwds)
Create an object which operates like a regular writer but maps dictionaries onto output rows. The fieldnames(not optional) parameter is a `sequence` of keys that identify the order in which values in the dictionary passed to the `writerow()` method are written to file f. The optional restval parameter specifies the value to be written if the dictionary is missing a key in fieldnames. If the dictionary passed to the `writerow()` method contains a key not found in fieldnames, the optional extrasaction parameter indicates what action to take. If it is set to `'raise'`, the default value, a `ValueError` is raised. If it is set to `'ignore'`, extra values in the dictionary are ignored. Any other optional or keyword arguments are passed to the underlying `writer` instance.
```
import csv

with open('names.csv', 'w', newline='') as csvfile:
    fieldnames = ['first_name', 'last_name']
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

    writer.writeheader()
    writer.writerow({'first_name': 'Baked', 'last_name': 'Beans'})
    writer.writerow({'first_name': 'Lovely', 'last_name': 'Spam'})
    writer.writerow({'first_name': 'Wonderful', 'last_name': 'Spam'})
```

### class csv.Dialect
The Dialect class is a container class relied on primarily for its attributes, which are used to define the parameters for a specific `reader` or `writer` instance.

### class csv.excel (registered with name 'excel'); class csv.excel_tab (registered with name 'excel-tab'); class csv.unix_dialect (registered with name 'unix')

### class csv.Sniffer
The `Sniffer` class is used to deduce the format of a CSV file.
Methods: `sniff(sample, delimiters=None)`; `hsa_header(sample)`

### constans:
`csv.QUOTE_ALL`; `csv.QUOTE_MINIMAL`; `csv.QUOTE_NONNUMERIC`; `csv.QUOTE_NONE`

### exception
`csv.Error`

## Dialects and Formatting Parameters
1. Dialect.delimiter
A one-character string used to separate fields. It defaults to `','`.

2. Dialect.doublequote
Controls how instances of quotechar appearing inside a field should themselves be quoted. When `True`, the character is doubled. When `False`, the escapechar is used as a prefix to the quotechar. It defaults to `True`.

 On output, if doublequote is False and no escapechar is set, Error is raised if a quotechar is found in a field.

3. Dialect.escapechar
A one-character string used by the writer to escape the delimiter if quoting is set to `QUOTE_NONE` and the quotechar if doublequote is `False`. On reading, the escapechar removes any special meaning from the following character. It defaults to `None`, which disables escaping.

4. Dialect.lineterminator
The string used to terminate lines produced by the writer. It defaults to `'\r\n'`.

 > Note The reader is hard-coded to recognise either '\r' or '\n' as end-of-line, and ignores lineterminator. This behavior may change in the future.

5. Dialect.quotechar
A one-character string used to quote fields containing special characters, such as the delimiter or quotechar, or which contain new-line characters. It defaults to `'"'`.

6. Dialect.quoting
Controls when quotes should be generated by the writer and recognised by the reader. It can take on any of the QUOTE_* constants (see section Module Contents) and defaults to `QUOTE_MINIMAL`.

7. Dialect.skipinitialspace
When `True`, whitespace immediately following the delimiter is ignored. The default is `False`.

8. Dialect.strict
When True, raise exception `Error` on bad CSV input. The default is False.

## Reader Objects
Methods: `__next__()`; `dialect`; `line_num`; `fieldnames`

## Writer Objects
Methods: `writerow(row)`; `writerows(rows)`; `dialect`; `writeheader()`

## Examples
```
import csv
with open('passwd', newline='') as f:
    reader = csv.reader(f, delimiter=':', quoting=csv.QUOTE_NONE)
    for row in reader:
        print(row)
```
Since `open()` is used to open a CSV file for reading, the file will by default be decoded into unicode using the system default encoding (see [`locale.getpreferredencoding()`](https://docs.python.org/3.6/library/locale.html#locale.getpreferredencoding)). To decode a file using a different encoding, use the encoding argument of open:
```
import csv
with open('some.csv', newline='', encoding='utf-8') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```
```
import csv, sys
filename = 'some.csv'
with open(filename, newline='') as f:
    reader = csv.reader(f)
    try:
        for row in reader:
            print(row)
    except csv.Error as e:
        sys.exit('file {}, line {}: {}'.format(filename, reader.line_num, e))
```