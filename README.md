# FlashPCA (_alpha version_)

FlashPCA performs fast principal component analysis (PCA) of single nucleotide
polymorphism (SNP) data, similar to smartpca from EIGENSOFT
(http://www.hsph.harvard.edu/alkes-price/software/) and shellfish
(https://github.com/dandavison/shellfish). FlashPCA is based on the
[https://github.com/yixuan/spectra/](Spectra) library.

Main features:

* Fast: partial PCA of 500,000 individuals with 100,000 SNPs in &lt;6h using 2GB RAM
* Memory requirements are bounded
* Highly accurate results 
* Natively reads PLINK bed/bim/fam files
* Easy to use

## Help

Google Groups: [https://groups.google.com/forum/#!forum/flashpca-users](https://groups.google.com/forum/#!forum/flashpca-users)

## Contact

Gad Abraham, gad.abraham@unimelb.edu.au

## Citation
G. Abraham and M. Inouye, Fast Principal Component Analysis of Large-Scale
Genome-Wide Data, PLos ONE 9(4): e93766. [doi:10.1371/journal.pone.0093766](http://www.plosone.org/article/info:doi/10.1371/journal.pone.0093766)

(preprint: http://biorxiv.org/content/early/2014/03/11/002238)

## License
This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 3 of the License, or
(at your option) any later version.

Copyright (C) 2014-2016 Gad Abraham. All rights reserved.

Portions of this code are based on SparSNP
(https://github.com/gabraham/SparSNP), Copyright (C) 2011-2012 Gad Abraham
and National ICT Australia (http://www.nicta.com.au).

## Download statically linked version (stable versions only, not for alpha versions)

* We recommend compiling from source for best performance.
* To get the devel version, you'll need to compile yourself

See [Releases](https://github.com/gabraham/flashpca/releases) for statically-linked version for Linux x86-64 &ge; 2.6.15

### System requirements
* 64-bit Linux or Mac

## Building from source

To get the latest version:
   ```
   git clone git://github.com/gabraham/flashpca
   ```

### Requirements

On Linux:

* 64-bit OS
* g++ compiler
* Eigen (http://eigen.tuxfamily.org), v3.2 or higher
   (if you get a compile error ``error: no match for 'operator/' in '1 / ((Eigen::MatrixBase...`` you'll need a more recent Eigen)
* Spectra (https://github.com/yixuan/spectra/)
* Boost (http://www.boost.org/), specifically boost_program_options/boost_program_options-mt.
* libgomp for openmp support
* Recommended: plink2 (https://www.cog-genomics.org/plink2) for SNP
   thinning

On Mac:

* Homebrew (http://brew.sh) to install gcc/g++ and boost
* Eigen, as above
* Spectra, as above
* Set CXX to whatever g++ version you're using before calling make, e.g.:
```
export CXX=/usr/local/bin/g++-4.7
```

### To install

The [Makefile](Makefile) contains three variables that need to be set according to where you have installed the Eigen
headers and Boost headers and libraries on your system. The default values for these are: 
   ```
   EIGEN_INC=/usr/local/include/eigen
   BOOST_INC=/usr/local/include/boost
   BOOST_LIB=/usr/local/lib
   SPECTRA_INC=spectra
   ```
   
 If your system has these libraries and header files in those locations, you can simply run make:
   ```
   cd flashpca
   make all
   ```
   
 If not, you can override their values on the make command line. For example, if you have the Eigen source in `/opt/eigen-3.2.5` and Boost 1.59.0 installed into `/opt/boost-1.59.0`, you could run: 
   ```
   cd flashpca
   make all EIGEN_INC=/opt/eigen-3.2.5 BOOST_INC=/opt/boost-1.59.0/include BOOST_LIB=/opt/boost-1.59.0/lib
   ```
 
Note: the compilation process will first look for a local directory named
Eigen. It should contain the file signature_of_eigen3_matrix_library. Next,
it will look for the directory /usr/include/eigen3 (Debian/Ubuntu location
for Eigen), although those available through apt-get tend to be older versions.

## Quick start

First thin the data by LD (highly recommend
[plink2](https://www.cog-genomics.org/plink2) for this):
   ```
   plink --bfile data --indep-pairwise 1000 50 0.05 --exclude range exclusion_regions_hg19.txt
   plink --bfile data --extract plink.prune.in --make-bed --out data_pruned
   ```
where [exclusion_regions_hg19.txt](exclusion_regions_hg19.txt) contains:
   ```
   5 44000000 51500000 r1
   6 25000000 33500000 r2
   8 8000000 12000000 r3
   11 45000000 57000000 r4
   ```
(You may need to change the --indep-pairwise parameters to get a suitable
number of SNPs for you dataset, 10,000-50,000 is usually enough.)

To run on the pruned dataset:
   ```
   ./flashpca --bfile data_pruned
   ```

To append a custom suffix '_mysuffix.txt' to all output files:
   ```
   ./flashpca --suffix _mysuffix.txt ...
   ```

To see all options
   ```
   ./flashpca --help 
   ```

## Output

flashpca produces the following files:

* `eigenvectors.txt`: the top k eigenvectors of the covariance
   X X<sup>T</sup> / p, same as matrix U from the SVD of the genotype matrix
   X/sqrt(p)=UDV<sup>T</sup> (where p is the number of SNPs).
* `pcs.txt`: the top k principal components (the projection of the data on the
eigenvectors, scaled by the eigenvalues,  same as XV (or UD). This is the file
you will want to plot the PCA plot from.
* `eigenvalues.txt`: the top k eigenvalues of X X<sup>T</sup> / p. These are the
    square of the singular values D (square of sdev from prcomp).
* `pve.txt`: the proportion of total variance explained by *each of the top k*
   eigenvectors (the total variance is given by the trace of the covariance
   matrix X X<sup>T</sup> / p, which is the same as the sum of all eigenvalues).
   To get the cumulative variance explained, simply
   do the cumulative sum of the variances (`cumsum` in R).

## Warning

You must perform quality control using PLINK (at least filter using --geno, --mind,
--maf, --hwe) before running flashpca on your data. You will likely get
spurious results otherwise.

### <a name="scca"></a>Sparse Canonical Correlation Analysis (SCCA)

* flashpca now experimentally supports sparse CCA
   ([Parkhomenko 2009](http://dx.doi.org/10.2202/1544-6115.1406),
   [Witten 2009](http://dx.doi.org/10.1093/biostatistics/kxp008)),
   between SNPs and multivariate phenotypes.
* The phenotype file is the same as PLINK phenotype file:
   `FID, IID, pheno1, pheno2, pheno3, ...`
   except that there must be no header line. The phenotype file *must be in the same order as
   the FAM file*.
* The L1 penalty for the SNPs is `--lambda1` and for the phenotypes is
 `--lambda2`.

#### Quick example
   ```
   ./flashpca --scca --bfile data --pheno pheno.txt \
   --lambda1 1e-3 --lambda2 1e-2 --ndim 10 --numthreads 8
   ```

* The file eigenvectorsX.txt are the left eigenvectors of X<sup>T</sup> Y, with size (number of
  SNPs &times; number of dimensions), and eigenvectorsY.txt are the right
  eigenvectors of X<sup>T</sup> Y, with size (number of phenotypes &\times; number of
  dimensions).

#### Example scripts to tune the penalties via split validation

We optimise the penalties by finding the values that maximise the correlation
of the canonical components cor(X U, Y V) in independent test data.

* Wrapper script [scca.sh](scca.sh) ([GNU
   parallel](http://www.gnu.org/software/parallel) is recommended)
* R code for plotting the correlations [scca_pred.R](scca_pred.R)

# <a name="R"></a>flashpcaR: flashpca in R

flashpca is now available as an independent R package.

## _Sep 13 2016: flashpcaR is not yet available for this alpha version of flashpca_


## Prebuilt R packages

Ssee [Releases](https://github.com/gabraham/flashpca/releases) for prebuilt
Mac/Windows binary packages and a source package for Linux.

## Building from source

### Requirements

* R packages: Rcpp, RcppEigen, BH
* C++ compiler

As of version v1.2.5, flashpcaR will compile on Mac with either clang++ or g++.
However, OpenMP multi-threading won't work with clang (see
https://github.com/gabraham/flashpca/issues/5).

### Several ways to install from source:

* If you downloaded the Release source code:
   ```
   R CMD INSTALL flashpcaR_1.2.5.tar.gz
   ```

* To install the latest (potentially unstable) version on Mac or Linux,
   you can also use devtools::install_github:
   ```
   library(devtools)
   install_github("gabraham/flashpca/flashpcaR")
   ```

* Alternatively, after cloning the git archive, install using:
   ```
   R CMD INSTALL flashpcaR
   ```


## PCA

Example usage, assuming `X` is a 100-sample by 1000-SNP matrix in dosage
coding (0, 1, 2) (an actual matrix, not a path to PLINK data)
   ```
   dim(X)
   [1]  100 1000
   library(flashpcaR)
   r <- flashpca(X, do_loadings=TRUE, verbose=TRUE, stand="binom", ndim=10,
   nextra=100)
   ```

PLINK data can be loaded into R either by recoding the data into raw format (`recode A`) or using package [plink2R](https://github.com/gabraham/plink2R).

Output:
   * `values`: eigenvalues
   * `vectors`: eigenvectors
   * `projection`: projection of sample onto eigenvectors (X V)
   * `loadings`: SNP loadings, if using a linear kernel

## Sparse CCA

Sparse CCA of matrices X and Y, with 5 components, penalties lambda1=0.1 and lambda2=0.1:

   ```
   dim(X)
   [1]  100 1000
   dim(Y)
   [1]  100 50
   r <- scca(X, Y, ndim=5, lambda1=0.1, lambda2=0.1)
   ```

# LD-pruned HapMap3 example data

See the [HapMap3](HapMap3) directory

# Changelog (stable versions only)

See [CHANGELOG.txt](CHANGELOG.txt)

