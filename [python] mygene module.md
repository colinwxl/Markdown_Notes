## mygene [python module]
1. Usage: ID mapping
> **mygene** is essentially a convenient Python module to access MyGene.info gene query web services.

2. quick start

```
import mygene
mg = mygene.MyGeneInfo()
# mapping gene symbols to Entrez gene ids
xli = ['DDX26B', 'CCDC83', 'MAST3', 'FLOT1', 'RPL11', 'ZDHHC20', 'LUC7L3', 'SNORD49A', 'CTSH', 'ACOT8']
out = mg.querymany(xli, scopes='symbol', fields='entrezgene', species='human')
# The mapping result is returned as a list of dictionaries. 

# mapping gene symbols to Ensembl gene ids
mg.querymany(xli, scopes='symbol', fields='ensembl.gene', species='human')

mg.querymany(xli, scopes='symbol,reporter,accession', fields='entrezgene,uniprot', species='human')

mg.getgene(1017, 'name,symbol,refseq.rna')
mg.getgenes([1017,1018,'ENSG00000148795'], as_dataframe=True)
mg.query('symbol:cdk2', species='human')

# query all human kinases using fetch_all parameter:
kinases = mg.query('name:kinase', species='human', fetch_all=True)
# kinases is a Python generator, now you can loop through it to get all 1073 hits:
for gene in kinases:
     print gene['_id'], gene['symbol']
```

3. [class mygene.MyGeneInfo(url='http://mygene.info/v3')](http://mygene-py.readthedocs.io/en/latest/)
`mg = mygene.MyGeneInfo()`

<1> getgene(geneid, fields='symbol, name, taxid, entrezgene', **kwargs)
	Return the gene object for the give geneid. This is a wrapper for GET query of “/gene/<geneid>” service.

> **Parameters:**	
	**geneid** – entrez/ensembl gene id, entrez gene id can be either a string or integer
	**fields** – fields to return, a list or a comma-separated string. If fields=”all”, all available fields are returned
	**species** – optionally, you can pass comma-separated species names or taxonomy ids
	**email** – optionally, pass your email to help us to track usage
	**filter** – alias for fields parameter
**Returns:**	
	a gene object as a dictionary, or None if geneid is not valid.
Note that field name supports dot notation for nested data structure as well, e.g. you can pass “refseq.rna” or “pathway.kegg”.

``
<2> getgenes(geneids, fields='symbol, name, taxid, entrezgene', **kwargs)
	Return the list of gene objects for the given list of geneids. This is a wrapper for POST query of “/gene” service.

> **Parameters:**
	**geneids** – a list/tuple/iterable or comma-separated entrez/ensembl gene ids
	**fields** – fields to return, a list or a comma-separated string. If fields=”all”, all available fields are returned
	**species** – optionally, you can pass comma-separated species names or taxonomy ids
	**email** – optionally, pass your email to help us to track usage
	**filter** – alias for fields
	**as_dataframe** – if True, return object as DataFrame (requires Pandas).
	**df_index** – if True (default), index returned DataFrame by ‘query’, otherwise, index by number. Only applicable if as_dataframe=True.
**Returns:**	
	a list of gene objects or a pandas DataFrame object (when as_dataframe is True)

<3> query(q, **kwargs)
	Return the query result. This is a wrapper for GET query of “/query?q=<query>” service.
> **Parameters:**	
	**q** – a query string, detailed query syntax here
	**fields** – fields to return, a list or a comma-separated string. If fields=”all”, all available fields are returned
	**species** – optionally, you can pass comma-separated species names or taxonomy ids. Default: human,mouse,rat.
	**size** – the maximum number of results to return (with a cap of 1000 at the moment). Default: 10.
	**skip** – the number of results to skip. Default: 0.
	**sort** – Prefix with “-” for descending order, otherwise in ascending order. Default: sort by matching scores in decending order.
	**entrezonly** – if True, return only matching entrez genes, otherwise, including matching Ensemble-only genes (those have no matching entrez genes).
	**email** – optionally, pass your email to help us to track usage
	**as_dataframe** – if True, return object as DataFrame (requires Pandas).
	**df_index** – if True (default), index returned DataFrame by ‘query’, otherwise, index by number. Only applicable if as_dataframe=True.
	**fetch_all** – if True, return a generator to all query results (unsorted). This can provide a very fast return of all hits from a large query. Server requests are done in blocks of 1000 and yielded individually. Each 1000 block of results must be yielded within 1 minute, otherwise the request will expire on the server side.
**Returns:**	
	a dictionary with returned gene hits or a pandas DataFrame object (when as_dataframe is True)

<4> querymany(qterms, scopes=None, **kwargs)
	Return the batch query result. This is a wrapper for POST query of “/query” service.
> **Parameters:**
	**qterms** – a list/tuple/iterable of query terms, or a string of comma-separated query terms.
	**scopes** – type of types of identifiers, either a list or a comma-separated fields to specify type of input qterms, e.g. “entrezgene”, “entrezgene,symbol”, [“ensemblgene”, “symbol”]. Refer to official MyGene.info docs for full list of fields.
	**fields** – fields to return, a list or a comma-separated string. If fields=”all”, all available fields are returned
	**species** – optionally, you can pass comma-separated species names or taxonomy ids. Default: human,mouse,rat.
	**entrezonly** – if True, return only matching entrez genes, otherwise, including matching Ensemble-only genes (those have no matching entrez genes).
	**returnall** – if True, return a dict of all related data, including dup. and missing qterms
	**verbose** – if True (default), print out infomation about dup and missing qterms
	**email** – optionally, pass your email to help us to track usage
	**as_dataframe** – if True, return object as DataFrame (requires Pandas).
	**df_index** – if True (default), index returned DataFrame by ‘query’, otherwise, index by number. Only applicable if as_dataframe=True.
**Returns:**	
	a list of gene objects or a pandas DataFrame object (when as_dataframe is True)