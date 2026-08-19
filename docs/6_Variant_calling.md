# 6. Variant calling
!!! info ""

- To detect both small and large variants among paths from the pangenome graph, we utilized the Variation Graph (VG) toolkit to deconstruct these variants into VCF files.
- To decompose the graph into a VCF file, we need to choose one path as a reference for comparison with the others (any path can serve as this reference).
- In fact, during the pangenome graph construction process, when the parameter -V 'NC_017518.1:#' is activated, the output file includes the VCF based on the NC_017518.1 reference.

## `vg deconstruct` graph to get the variations in vcf 
!!! info ""

Set up directory for VG and GFA files.

!!! terminal "code"
    
    - Return to working directory
    ```bash
    cd ~/pg_workshop
    ```

    - Create VG directory
    ```bash
    mkdir -p vg_deconstruct
    ```

    - Copy the gfa file to the directory
    ```bash
    cp ./5NM_2Kb94/5NM*.gfa ./vg_deconstruct/5NM_2Kb94.gfa
    cd ./vg_deconstruct
    ```
    
Load the necessary modules for an example run.
!!! terminal "code"

    ```bash
    module purge
    module load vg/1.46.0
    module load BCFtools/1.15.1-GCC-11.3.0
    ```

An example run to obtain VCF files from GFA.

!!! terminal "code"

    - check the paths in the graph using tail, which depends on the number of genomes. We have five input genomes for the 5NM dataset. 
    ```bash
    tail -5 5NM_2Kb94.gfa | less -S
    ```
    !!! success "Output"
        
        ```
		P       NC_003112.2#1#1   1+,2+,4+,5+,7+,8+,10+,12+,13+,14+,16+,18+,19+,21+,22+,23+,25+,26+,28+>
		P       NC_017518.1#1#1   1+,3+,4+,5+,7+,8+,10+,11+,13+,14+,16+,17+,19+,20+,22+,24+,25+,27+,28+>
		P       NZ_CP007668.1#1#1 1+,3+,4+,5+,7+,8+,10+,11+,13+,14+,16+,17+,19+,20+,22+,24+,25+,27+,28+>
		P       NZ_CP016880.1#1#1 1+,2+,4+,6+,7+,9+,10+,12+,13+,15+,16+,18+,19+,21+,22+,24+,25+,27+,28+>
		P       NZ_CP020423.2#1#1 1+,2+,4+,6+,7+,9+,10+,12+,13+,14+,16+,18+,19+,21+,22+,24+,25+,27+,28+>
        ```

!!! terminal "code"

    ```bash
    vg deconstruct -h
    ```
    
    !!! success "Output"
    	The arguments we are interested in are -e -a.
    	```text
    	use vg deconstruct the graph into VCF based on the first path NC_003112.2
    	-e, --path-traversals    Only consider traversals that correspond to paths in the graph.
    	-a, --all-snarls         Process all snarls, including nested snarls (by default only top-level snarls reported).
    	```
    
    - vg deconstruct for the 5NM_2Kb94.gfa using the path NC_003112.2 as reference 
    ```bash
    vg deconstruct -p "NC_003112.2#1#1#0" -a -e ./5NM_2Kb94.gfa > 5NM_2Kb94aep1.vcf
    bgzip 5NM_2Kb94aep1.vcf
    tabix 5NM_2Kb94aep1.vcf.gz
    ```

    - use vg deconstruct the graph into VCF based on the second path NC_017518.1
    ```bash
    vg deconstruct -p "NC_017518.1#1#1#0" -a -e ./5NM_2Kb94.gfa > 5NM_2Kb94aep2.vcf
	bgzip 5NM_2Kb94aep2.vcf
    tabix 5NM_2Kb94aep2.vcf.gz
    ```
    
    - Inspect the vcf files
    ```bash
    less 5NM_2Kb94aep1.vcf.gz
    ```
    
    ??? success "Output"
        
        ```
		##fileformat=VCFv4.2
		##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
		##INFO=<ID=CONFLICT,Number=.,Type=String,Description="Sample names for which there are multiple paths in the graph with conflicting alleles">
		##INFO=<ID=AC,Number=A,Type=Integer,Description="Total number of alternate alleles in called genotypes">
		##INFO=<ID=AF,Number=A,Type=Float,Description="Estimated allele frequency in the range (0,1]">
		##INFO=<ID=NS,Number=1,Type=Integer,Description="Number of samples with data">
		##INFO=<ID=AN,Number=1,Type=Integer,Description="Total number of alleles in called genotypes">
		##INFO=<ID=LV,Number=1,Type=Integer,Description="Level in the snarl tree (0=top level)">
		##INFO=<ID=PS,Number=1,Type=String,Description="ID of variant corresponding to parent snarl">
		##INFO=<ID=AT,Number=R,Type=String,Description="Allele Traversal as path in graph">
		##contig=<ID=NC_003112.2#1#1,length=2272360>
		#CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  NC_017518.1#1#1   NZ_CP007668.1#1#1 NZ_CP016880.1#1#1 NZ_CP020423.2#1#1
		1       152     >1>4    C       T       60      .       AC=2;AF=0.5;AN=4;AT=>1>2>4,>1>3>4;NS=4;LV=0     GT      1       1       0       0
		1       510     >4>7    A       G       60      .       AC=2;AF=0.5;AN=4;AT=>4>5>7,>4>6>7;NS=4;LV=0     GT      0       0       1       1
		1       558     >7>10   A       G       60      .       AC=2;AF=0.5;AN=4;AT=>7>8>10,>7>9>10;NS=4;LV=0   GT      0       0       1       1
		1       954     >10>13  G       A       60      .       AC=2;AF=0.5;AN=4;AT=>10>12>13,>10>11>13;NS=4;LV=0       GT      1       1       0       0
		1       1139    >13>16  A       G       60      .       AC=1;AF=0.25;AN=4;AT=>13>14>16,>13>15>16;NS=4;LV=0      GT      0       0       1       0
		1       1411    >16>19  G       A       60      .       AC=2;AF=0.5;AN=4;AT=>16>18>19,>16>17>19;NS=4;LV=0       GT      1       1       0       0
		1       1539    >19>22  T       C       60      .       AC=2;AF=0.5;AN=4;AT=>19>21>22,>19>20>22;NS=4;LV=0       GT      1       1       0       0
		1       1561    >22>25  A       G       60      .       AC=4;AF=1;AN=4;AT=>22>23>25,>22>24>25;NS=4;LV=0 GT      1       1       1       1
		1       1630    >25>28  G       C       60      .       AC=4;AF=1;AN=4;AT=>25>26>28,>25>27>28;NS=4;LV=0 GT      1       1       1       1
		1       1674    >28>31  T       C       60      .       AC=4;AF=1;AN=4;AT=>28>29>31,>28>30>31;NS=4;LV=0 GT      1       1       1       1
		1       1781    >31>34  A       G       60      .       AC=4;AF=1;AN=4;AT=>31>32>34,>31>33>34;NS=4;LV=0 GT      1       1       1       1
		1       1809    >34>36  AT      A       60      .       AC=4;AF=1;AN=4;AT=>34>35>36,>34>36;NS=4;LV=0    GT      1       1       1       1
		1       1819    >36>38  AT      A       60      .       AC=4;AF=1;AN=4;AT=>36>37>38,>36>38;NS=4;LV=0    GT      1       1       1       1
		1       1894    >38>41  G       C       60      .       AC=4;AF=1;AN=4;AT=>38>39>41,>38>40>41;NS=4;LV=0 GT      1       1       1       1
		1       1909    >41>44  A       G       60      .       AC=4;AF=1;AN=4;AT=>41>42>44,>41>43>44;NS=4;LV=0 GT      1       1       1       1
		1       1974    >44>47  A       G       60      .       AC=4;AF=1;AN=4;AT=>44>45>47,>44>46>47;NS=4;LV=0 GT      1       1       1       1
		1       2069    >47>50  A       G       60      .       AC=4;AF=1;AN=4;AT=>47>48>50,>47>49>50;NS=4;LV=0 GT      1       1       1       1
		1       2073    >50>53  T       C       60      .       AC=2;AF=0.5;AN=4;AT=>50>51>53,>50>52>53;NS=4;LV=0       GT      0       0       1       1
		1       2079    >53>56  T       C       60      .       AC=4;AF=1;AN=4;AT=>53>54>56,>53>55>56;NS=4;LV=0 GT      1       1       1       1
		1       2084    >56>59  AT      GG      60      .       AC=4;AF=1;AN=4;AT=>56>57>59,>56>58>59;NS=4;LV=0 GT      1       1       1       1
		1       2089    >59>62  T       C       60      .       AC=4;AF=1;AN=4;AT=>59>60>62,>59>61>62;NS=4;LV=0 GT      1       1       1       1
		1       2099    >62>64  A       AC      60      .       AC=4;AF=1;AN=4;AT=>62>64,>62>63>64;NS=4;LV=0    GT      1       1       1       1
		1       2178    >64>67  G       A       60      .       AC=4;AF=1;AN=4;AT=>64>65>67,>64>66>67;NS=4;LV=0 GT      1       1       1       1
		1       2200    >67>70  A       G       60      .       AC=4;AF=1;AN=4;AT=>67>68>70,>67>69>70;NS=4;LV=0 GT      1       1       1       1
		1       2290    >70>73  C       T       60      .       AC=4;AF=1;AN=4;AT=>70>71>73,>70>72>73;NS=4;LV=0 GT      1       1       1       1
		1       2359    >73>76  T       C       60      .       AC=4;AF=1;AN=4;AT=>73>74>76,>73>75>76;NS=4;LV=0 GT      1       1       1       1
		1       2362    >76>79  C       T       60      .       AC=4;AF=1;AN=4;AT=>76>77>79,>76>78>79;NS=4;LV=0 GT      1       1       1       1
		1       2392    >79>84  CG      AG,AA   60      .       AC=2,2;AF=0.5,0.5;AN=4;AT=>79>80>82>84,>79>81>82>84,>79>81>83>84;NS=4;LV=0      GT      2       2       1       1
		1       2491    >84>87  G       A       60      .       AC=2;AF=0.5;AN=4;AT=>84>85>87,>84>86>87;NS=4;LV=0       GT      0       0       1       1
		1       2494    >87>90  T       C       60      .       AC=2;AF=0.5;AN=4;AT=>87>88>90,>87>89>90;NS=4;LV=0       GT      0       0       1       1
		1       2503    >90>93  A       C       60      .       AC=4;AF=1;AN=4;AT=>90>91>93,>90>92>93;NS=4;LV=0 GT      1       1       1       1
        ```

!!! terminal-2 "check the statistics of vcf files"

	- use bcftools stats to check the statistics for the vcf file 
    ```bash
    bcftools stats 5NM_2Kb94aep1.vcf.gz > 5NM_2Kb94aep1.vcf_stats
    bcftools stats 5NM_2Kb94aep2.vcf.gz > 5NM_2Kb94aep2.vcf_stats
    ```

    ```bash
    less -S 5NM_2Kb94aep1.vcf_stats
    ```
    ??? success "Output"
        
        ```
		# This file was produced by bcftools stats (1.15.1+htslib-1.23.1) and can be plotted using plot-vcfstats.
		# The command line was: bcftools stats  5NM_2Kb94aep1.vcf.gz
		#
		# Definition of sets:
		# ID    [2]id   [3]tab-separated file names
		ID      0       5NM_2Kb94aep1.vcf.gz
		# SN, Summary numbers:
		#   number of records   .. number of data rows in the VCF
		#   number of no-ALTs   .. reference-only sites, ALT is either "." or identical to REF
		#   number of SNPs      .. number of rows with a SNP
		#   number of MNPs      .. number of rows with a MNP, such as CC>TT
		#   number of indels    .. number of rows with an indel
		#   number of others    .. number of rows with other type, for example a symbolic allele or
		#                          a complex substitution, such as ACT>TCGA
		#   number of multiallelic sites     .. number of rows with multiple alternate alleles
		#   number of multiallelic SNP sites .. number of rows with multiple alternate alleles, all SNPs
		# 
		#   Note that rows containing multiple types will be counted multiple times, in each
		#   counter. For example, a row with a SNP and an indel increments both the SNP and
		#   the indel counter.
		# 
		# SN    [2]id   [3]key  [4]value
		SN      0       number of samples:      4
		SN      0       number of records:      75703
		SN      0       number of no-ALTs:      0
		SN      0       number of SNPs: 66313
		SN      0       number of MNPs: 6901
		SN      0       number of indels:       3286
		SN      0       number of others:       1009
		SN      0       number of multiallelic sites:   3757
		SN      0       number of multiallelic SNP sites:       1363
        ```
        
!!! terminal-2 "check the complex variation in vcf files"

    ```bash
    zcat 5NM_2Kb94aep1.vcf.gz | grep -v '^##' | awk 'length($4) > 2' | head -100 | less -S 
    ```
    ??? success "Output"
            
            ```
			##INFO=<ID=CONFLICT,Number=.,Type=String,Description="Sample names for which there are multiple paths in the graph with conflicting alleles">
			##INFO=<ID=AC,Number=A,Type=Integer,Description="Total number of alternate alleles in called genotypes">
			##INFO=<ID=NS,Number=1,Type=Integer,Description="Number of samples with data">
			##INFO=<ID=AN,Number=1,Type=Integer,Description="Total number of alleles in called genotypes">
			##INFO=<ID=LV,Number=1,Type=Integer,Description="Level in the snarl tree (0=top level)">
			##INFO=<ID=PS,Number=1,Type=String,Description="ID of variant corresponding to parent snarl">
			##INFO=<ID=AT,Number=R,Type=String,Description="Allele Traversal as path in graph">
			#CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  NC_017518.1#1#1   NZ_CP007668.1#1#1 NZ_CP016880.1#1#1 NZ_CP020423.2#1#1
			1       3441    >250>600        CCAACCTCGCCAAAGTCCGCAAACAAGTAACTGCTTTGTGCAATAAATACCCCGTTTACGGCGCGTAAGCCTTTTTAAAAATATTCCGCCAAGCAATCCAATGCCGCCTGAAATCTCATAATGTTTCAGGCGGAAACCTTTGCAAAAATCCCCAAAATCCCCTAAATT>
			1       4164    >392>397        GCG     AAG,AAA 60      .       AC=1,1;AF=0.5,0.5;AN=2;AT=>392>393>395>397,>392>394>395>397,>392>394>396>397;NS=2;LV=1;PS=>250>600      GT      2       1       .       .
			1       4178    >397>402        ATCAAGAAAAACGGC AT,ACAAAGAAAAACGGC      60      .       AC=1,1;AF=0.5,0.5;AN=2;AT=>397>398>399>401>402,>397>398>402,>397>400>401>402;NS=2;LV=1;PS=>250>600      GT      >
			1       4394    >426>431        TTG     TTA,CAG 60      .       AC=1,1;AF=0.5,0.5;AN=2;AT=>426>427>429>431,>426>427>430>431,>426>428>429>431;NS=2;LV=1;PS=>250>600      GT      2       1       .       .
			1       4918    >518>520        TCC     T       60      .       AC=1;AF=0.5;AN=2;AT=>518>519>520,>518>520;NS=2;LV=1;PS=>250>600 GT      1       0       .       .
			1       5116    >529>553        TCCCGTCATTCCCGCGCAGGCGGGAATCTAGGTTTGTCGGCACGGAAACTTATCGGGTAAAACGGTTTCTTTAGATTTTACGTTCTAGATTCCCGCCTGCGCGGGAATGACGATGAAAAGATTGTTGTCGCTTCGGATAAATTTTTGTCGCGTTGGGTTCTAGATTCC>
			1       5441    >553>556        TCCCC   T,TCCC  60      .       AC=1,1;AF=0.5,0.5;AN=2;AT=>553>554>555>556,>553>556,>553>555>556;NS=2;LV=1;PS=>250>600  GT      2       1       .       .
			1       6462    >600>603        ACT     AA      60      .       AC=3;AF=0.75;AN=4;AT=>600>601>603,>600>602>603;NS=4;LV=0        GT      1       0       1       1
			1       8056    >938>1174       AGCTGCGCCGCCAGCGTGCAGAACGCCGCAACGTACAGGGACAAGTCAACTTCAAGCTCGATAGCGGTGAGAAAAGTGGCAAAATCATCGCCGAATTGGAACACGCTTCGTTTGCCTATGGCGGCAAAGTCATTATGGACAAATTCTCCGCTATCTTGCAGCGCGGCG>
			1       40957   >1264>1267      CACCCAGTTGCGCCAAAGCTGCGCCATCCCGCTC      TACACAGTTACGCCAAAGTTGCGCCATCCCGCTT      60      .       AC=2;AF=0.5;AN=4;AT=>1264>1266>1267,>1264>1265>1267;NS=4;LV=0   GT      >
			1       42763   >1445>1450      TGG     CAA,CAG 60      .       AC=2,1;AF=0.5,0.25;AN=4;AT=>1445>1448>1449>1450,>1445>1446>1447>1450,>1445>1446>1449>1450;NS=4;LV=0     GT      2       0       1       1
			1       42797   >1465>1470      CGCG    CATA,TGCG       60      .       AC=2,1;AF=0.5,0.25;AN=4;AT=>1465>1466>1469>1470,>1465>1466>1467>1470,>1465>1468>1469>1470;NS=4;LV=0     GT      0       2       >
			1       43307   >1650>1653      GAG     TTC     60      .       AC=1;AF=0.25;AN=4;AT=>1650>1652>1653,>1650>1651>1653;NS=4;LV=0  GT      0       1       0       0
			1       45378   >1733>1738      GGCCCGCTTCAGGGGC        GGCGCGCTTCAGGGGC,G      60      .       AC=2,2;AF=0.5,0.5;AN=4;AT=>1733>1734>1736>1737>1738,>1733>1734>1735>1737>1738,>1733>1738;NS=4;LV=0      >
			1       47380   >1813>1815      TCG     T       60      .       AC=4;AF=1;AN=4;AT=>1813>1814>1815,>1813>1815;NS=4;LV=0  GT      1       1       1       1
			1       47406   >1837>1839      AACGGAAT        A       60      .       AC=4;AF=1;AN=4;AT=>1837>1838>1839,>1837>1839;NS=4;LV=0  GT      1       1       1       1
			1       47431   >1851>1854      AAA     CTT     60      .       AC=4;AF=1;AN=4;AT=>1851>1853>1854,>1851>1852>1854;NS=4;LV=0     GT      1       1       1       1
			1       47447   >1861>1863      ACG     A       60      .       AC=4;AF=1;AN=4;AT=>1861>1862>1863,>1861>1863;NS=4;LV=0  GT      1       1       1       1
			1       47451   >1863>1866      TCCG    TT      60      .       AC=4;AF=1;AN=4;AT=>1863>1865>1866,>1863>1864>1866;NS=4;LV=0     GT      1       1       1       1
			1       47457   >1866>1869      GAA     TCC     60      .       AC=4;AF=1;AN=4;AT=>1866>1868>1869,>1866>1867>1869;NS=4;LV=0     GT      1       1       1       1
			1       47464   >1872>1874      TCCATTGCCGCATTGTCCAAGCAGTTTCCCTTGCGGGACATACTCTGAACCAGACCGTTGCCTTTCAACTGCTTTTG   T       60      .       AC=4;AF=1;AN=4;AT=>1872>1873>1874,>1872>1874;NS=4;LV=0  GT      >
			1       48557   >1944>1949      TCG     CGG,CGA 60      .       AC=2,1;AF=0.5,0.25;AN=4;AT=>1944>1946>1948>1949,>1944>1945>1948>1949,>1944>1945>1947>1949;NS=4;LV=0     GT      0       2       1       1
			1       48645   >1982>1985      CAC     AGA     60      .       AC=1;AF=0.25;AN=4;AT=>1982>1984>1985,>1982>1983>1985;NS=4;LV=0  GT      0       0       0       1
			1       48936   >2036>2040      TAT     TGT,TG  60      .       AC=2,1;AF=0.5,0.25;AN=4;AT=>2036>2038>2039>2040,>2036>2037>2039>2040,>2036>2037>2040;NS=4;LV=0  GT      0       1       2       1
			1       48942   >2040>2042      CCT     C       60      .       AC=1;AF=0.25;AN=4;AT=>2040>2041>2042,>2040>2042;NS=4;LV=0       GT      0       0       1       0
			1       49330   >2124>2129      TGT     TGG,ATT 60      .       AC=3,1;AF=0.75,0.25;AN=4;AT=>2124>2126>2128>2129,>2124>2126>2127>2129,>2124>2125>2128>2129;NS=4;LV=0    GT      1       2       1       1
			1       49436   >2170>2275      ACTTGTCTGACATGGAAAAATCCCTGTATTGAATTAAAAATCAATACAGGGATTGTAGGAAAGGCCGTCTGACTAAGCCTTTAATACGGGTTAAAACTTAATCAGTAGAGAGAGATGTGAGGATGATTTTTTTAGGCTTACGAGAGCCATTTTGCTTTAAGTCAAACT>
			1       50804   >2210>2213      AGAT    A       60      .       AC=1;AF=1;AN=1;AT=>2210>2211>2212>2213,>2210>2213;NS=1;LV=1;PS=>2170>2275       GT      1       .       .       .
			1       50904   >2230>2233      ATTC    GGCA    60      .       AC=1;AF=1;AN=1;AT=>2230>2232>2233,>2230>2231>2233;NS=1;LV=1;PS=>2170>2275       GT      1       .       .       .
			1       53387   >2355>2358      CAA     ATG     60      .       AC=1;AF=0.25;AN=4;AT=>2355>2357>2358,>2355>2356>2358;NS=4;LV=0  GT      1       0       0       0
			1       53393   >2361>2364      ACAA    TTTC    60      .       AC=1;AF=0.25;AN=4;AT=>2361>2362>2364,>2361>2363>2364;NS=4;LV=0  GT      1       0       0       0
			1       53414   >2381>2383      CGAT    C       60      .       AC=1;AF=0.25;AN=4;AT=>2381>2382>2383,>2381>2383;NS=4;LV=0       GT      1       0       0       0
			1       53471   >2404>2407      AGCG    CCAC    60      .       AC=1;AF=0.25;AN=4;AT=>2404>2406>2407,>2404>2405>2407;NS=4;LV=0  GT      1       0       0       0
			1       53493   >2416>2419      TCAA    CTTG    60      .       AC=1;AF=0.25;AN=4;AT=>2416>2418>2419,>2416>2417>2419;NS=4;LV=0  GT      1       0       0       0
			1       53604   >2467>2469      TGGGCTG T       60      .       AC=1;AF=0.25;AN=4;AT=>2467>2468>2469,>2467>2469;NS=4;LV=0       GT      1       0       0       0    
            ```

## Extract distance among paths

!!! terminal "code"

    ```bash
    module purge
	module load pggb/0.5.3-Miniconda3

    odgi similarity -i 5NM_2Kb94.gfa -d > 5NM_2Kb94.gfa_similarity
    cut -f 1,2,6 5NM_2Kb94.gfa_similarity > 5NM_2Kb94.gfa_similarity_cut
    ```
!!! terminal "code"

    ```bash
    #Using R for distanc clustering
    module purge
    module load R/4.0.1-gimkl-2020a
    R
    ```
!!! r-project "code"
    
    ```r linenums="1"
    library(reshape)
    library(ape)
 
    # read in the data
    dat=read.csv("./5NM_2Kb94.gfa_similarity_cut",sep="\t")
    dat
    
    # use reshape's cast function to change to matrix
    m <- cast(dat, group.a ~ group.b)
    m
    
    # set the row names
    rownames(m) <- m[,1]
    rownames(m)
 
    # change the matrix to a distance matrix
    d <- dist(m)
    d
 
    # do hierarchical clustering
    h <- hclust(d)
    h
    
    # plot the dendrogram
	pdf("5NM_2Kb94_dendrogram.pdf", width = 8, height = 6)
	plot(h,
     main = "5NM Similarity Dendrogram",
     xlab = "",
     sub  = "",
     ylab = "Height",
     cex  = 0.9)        # label text size, shrink if names are long/crowded	
    dev.off()
    
    ```

