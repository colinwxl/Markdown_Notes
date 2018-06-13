## [Linux] [Bioconda](https://bioconda.github.io/index.html)
### Introduction
Bioconda is a channel for the conda package manager specializing in bioinformatics software. Bioconda consists of:
- a repository of recipes hosted on GitHub
- a build system that turns these recipes into conda packages
- a [repository of more than 3000 bioinformatics packages](https://anaconda.org/bioconda/repo) ready to use with `conda install`
- Over 250 contributors that add, modify, update and maintain the recipes

**Browse packages in the Bioconda channel:** [Available packages](https://bioconda.github.io/recipes.html#recipes)
**Browse BioContainer packages:** [Biocontainers Registry UI](https://biocontainers.pro/registry/#/)

### Using Bioconda
1. Install (Mini)conda
2. Set up channels
```sh
conda config --add channels defaults
conda config --add channels conda-forge
conda config --add channels bioconda
```
3. Install packages
```sh
conda install [package name]
conda install -n [env name] [package names]
```
