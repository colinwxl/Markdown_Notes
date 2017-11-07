## Rank Aggregation [Bioinformatics]
> Let X = {x1...xk} with xi ∈ Rn denote a set of samples (profile vectors) of a certain class of objects. A rank aggregation method can be seen as a function X → θ, where θ is a so-called consensus ranking. A ranking is thereby a permutation of the numbers 1,...,n. Π   = {π1...πk},where πi is a ranking of the values of xi . 
### Borda Score
The Borda method (borda) calculates a score for each element (gene) of a profile. This score is based on a summary statistic (e.g., mean) of the ranks the gene achieved in the rankings of Π. The consensus ranking is constructed by ranking the scores of all genes. 
### Copeland's Score
In Copeland aggregation (copeland), the score of an element depends on its relative position in relationship to the other elements, A high score is assigned to those elements which are better ranked in the majority of the rankings in Π than the other elements.
### Pick-a-Perm
a consensus ranking can be found by selecting one from the available input rankings θ ∈ Π.
### Spearman’s Footrule
Spearman’s footrule (spearman) is defined as the sum of rank differences of all elements in two different rankings.