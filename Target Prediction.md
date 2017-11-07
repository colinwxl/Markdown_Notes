## Target Prediction [work]
### ID Conversion
1. unirot/entrez
```
wget ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/idmapping/by_organism/HUMAN_9606_idmapping.dat.gz
gzip -d HUMAN_9606_idmapping.dat.gz
grep "GeneID" HUMAN_9606_idmapping.dat > 

```