# Phase imputation for missing data on a linkage map
### preparing data for use in random forest

The goal of this program is to leverage the new linkage mapping technique we are employing, 
in order to create an input dataset of high quality for qtl analysis, and various
statistical modelling techniques I will employ. These methods are not tolerant of missing
genotypes, so we need to impute missing data for certain instances. Since data exists
in a linkage map, we have more information to work with than programs such as fastPHASE
which make imputations informed by LD and haplotype clustering. 

Markers placed in clusters lend themselves to making a consensus haplotype for the given
cluster. Consider the following hypothetical zero recombination cluster, ZRC_1. Any one
markers has missing genotypes and lacks a full phase vector necessary for modelling. These
markers all sit in the same cluster (cM location) on the linkage map, so their genotypes
can be merged to create a full phase vector for the given ZRC. 

### ZRC_1
### Member markers:

	M1 H-HAHAAAAA-HAHH-
	M2 HAHAH-AAAAHHAHHH
	M3 HAHAHAAAAAHH-HHH

### Consensus phase:

	ZRC_1  HAHAHAAAAAHHAHHH


## General idea of this algorithm
1. make a consensus phase for each of the clusters in the map. If the cluster is ambigious
	(i.e. all have missing data for a given column, or the two phases are equal in number)
	an NaN is returned, and imputed in step two.
2. For remaining missing data, fill the phase of the above and below clusters if they are equal.
	If the above or below clusters have non-matching phases (or one is also an NaN) then impute the 
	value by considering the number of matches with the above and below groups, and then impute
	the phase for the closer matching cluster.

I've tried to make a thorough consideration of fringe cases, such as when: 
1. There is disagreement between markers for an individual's phase in a cluster
	- mode will be taken, ties will be dealt with as well
2. All markers in a cluster lack a genotype for a given individual
	- will look at the clusters up and downstream to see if phase matches
	- if they do not, poll the other individuals for matches up and down
3. there is only one marker in the cluster
	- need to look at the adjacent clusters in manner of 2.
4. the flanking phases are both NaN
	- look one cluster further in either direction (2 away).
	- At this point you should be very careful, as there may be more
		missing data then one can deal with properly. So I've not
		coded this up to impute data from further than 3+ locations away
		(as at this point you may just be making things up).


## notes:
example_data folder has data in the format needed to run phase imputation.

support_functions folder contains scripts to help you move a linkage map into the format necessary to run phase imputation and shell scripts to execute phase imputation for numerous linkage groups at once.



