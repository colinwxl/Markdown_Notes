## Project Processing
### <2017-10-18>
1. ~/work/DATA/STRING
> awk '{if($3 >=800) print $1"\t"$2"\t"$3}' 9606.protein.links.v10.5.txt > links_800.txt
> awk '{if($3 >=850) print $1"\t"$2"\t"$3}' 9606.protein.links.v10.5.txt > links_850.txt
> create "string.py" to convert string\_id to entrez\_id **some string_id map to no entrez_id**
> get links_850_entrez.txt; links_800_entrez.txt
