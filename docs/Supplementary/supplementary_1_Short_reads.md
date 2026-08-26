# Mapping short reads with `vg giraffe`

NGS reads can be mapped to a pangenome graph rather than to a single linear reference. In this workshop we use [`vg giraffe`](https://github.com/vgteam/vg/wiki/Mapping-short-reads-with-Giraffe), the haplotype-aware short-read mapper in `vg`.

`vg giraffe` is designed to map short reads efficiently to pangenome graphs while using known haplotype paths to improve mapping through variable regions.

!!! info "Why `vg giraffe` rather than `vg map`?"
    `vg map` is the original general-purpose graph mapper, whereas `vg giraffe` is specifically designed for fast short-read mapping to haplotype-aware pangenome graphs. For short-read mapping, `vg giraffe` is the recommended approach.

## Learning objectives

!!! quote ""

    - Build the indexes required for `vg giraffe` from a pangenome graph.
    - Map paired-end NGS reads to the graph.
    - Assess mapping quality using `vg stats`.

## Before you start

For this workshop we already have a PGGB graph in GFA format:

```text
5NM.fa.fefc7f5.417fcdf.e2ae00b.smooth.final.gfa
```

The graph contains the five *N. meningitidis* assemblies used earlier in the workshop.

!!! warning "Use a recent version of `vg`"
    `vg` is under active development. The current `vg` documentation recommends using a recent release because important fixes and improvements are made regularly. 
    
    We are using a very old version of vg (v1.46.0).
---

## Build the Giraffe indexes

### Prepare the working directory

!!! terminal "code"
    ```bash
    cd ~/pg_workshop/Short_Read_Mapping

    # Copy the PGGB graph to the working directory
    cp ../5NM_2Kb94/5NM.fa.fefc7f5.417fcdf.e2ae00b.smooth.final.gfa ./5NM.gfa
    ```

Load `vg`:

!!! terminal "code"
    ```bash
    module purge
    module load vg/1.46.0
    ```

!!! tip
    Record the `vg` version used to construct the indexes and perform the mapping. Using the same version for index construction and mapping avoids compatibility problems.

### Convert P-lines to W-lines with `vg`

!!! warning "P-lines and W-lines"
    This graph's `.gfa` file uses **P-lines** (GFA v1.0) to encode paths, not **W-lines** (GFA v1.1). PGGB writes P-lines. Some downstream tools expect or prefer W-lines to auto-detect reference vs. haplotype paths, so a conversion step is required before those steps.

!!! terminal "code"
	```bash
    vg convert -g 5NM.gfa -f > 5NM.walk.gfa
	```

    - `-g` reads the GFA input
    - `-f` writes GFA output (W-lines by default; pass `-W` instead to force P-lines back out)

    Path names should follow [PanSN-spec](https://github.com/pangenome/PanSN-spec) (`sample#haplotype#contig`) so `vg` can correctly assign each W-line's SampleId/HapIndex/SeqId fields and distinguish reference paths from haplotypes.
	To convert W-lines to P-lines use `vg convert -g 5NM.walk.gfa -f -W > 5NM.path.gfa`


### Build the indexes with `vg autoindex`

For a GFA graph containing haplotype paths, the simplest approach is to let `vg autoindex` construct the Giraffe indexes:

!!! terminal "code"
    ```bash
    vg autoindex \
        --workflow giraffe \
        -g 5NM.walk.gfa \
        -p 5NM_pangenome
    ```

This produces the files required by `vg giraffe`, including:

```text
5NM_pangenome.giraffe.gbz
5NM_pangenome.dist
5NM_pangenome.min
```

`vg autoindex` handles the required graph preparation and index construction, including chopping long nodes where necessary. The GBZ contains the graph together with haplotype information. The other files provide the distance and minimizer indexes used by Giraffe.

!!! info "Long nodes"
    vg does not work well with graph nodes longer than 1024 bp when constructing minimizer indexes. When converting a GFA to GBZ, `vg` automatically chops long nodes as required. The resulting GBZ retains a translation back to the original graph coordinates.

!!! warning
    From vg v1.63.0 a .zipcodes index will also be created (and expected for downstream tools).

---

## Map short-read WGS reads

Say we have a sample with short-read WGS which we want to aligned to the pangenome, in this case a [*Neisseria meningitidis* isolate from a NZ hospital](https://www.ncbi.nlm.nih.gov/bioproject/592848):

```text
SRR10610805_1.fastq.gz
SRR10610805_2.fastq.gz
```

The recommended Giraffe command is:

!!! terminal "code"
    ```bash
    vg giraffe \
        -p \
        -t 8 \
        -Z 5NM_pangenome.giraffe.gbz \
        -d 5NM_pangenome.dist \
        -m 5NM_pangenome.min \
        -f SRR10610805_1.fastq.gz \
        -f SRR10610805_2.fastq.gz \
        -b default \
        > SRR10610805.5NM.gam
    ```

The important options are:

| Option | Purpose |
|---|---|
| `-p` | Print progress information |
| `-t 8` | Use 8 mapping threads |
| `-Z` | Giraffe GBZ graph |
| `-d` | Distance index |
| `-m` | Short-read minimizer index |
| `-f` | FASTQ input; specify twice for paired-end reads |
| `-b default` | Default short-read mapping preset |

!!! terminal "code"
    ```bash
	vg view --align-in SRR10610805.5NM.gam | less
    ```

---

## Evaluate the mapping

`vg stats` can be used to obtain basic alignment statistics:

!!! terminal "code"
    ```bash
    vg stats -a SRR10610805.5NM.gam > SRR10610805.5NM.stats
    cat SRR10610805.5NM.stats
    ```

Useful statistics include:

```text
Total alignments: 1139592
Total primary: 1139592
Total secondary: 0
Total aligned: 1139258
Total perfect: 1018448
Total gapless (softclips allowed): 1133796
Total paired: 1139592
Total properly paired: 1137260
Insertions: 8652 bp in 2525 read events
Deletions: 11447 bp in 5107 read events
Substitutions: 199048 bp in 199048 read events
Softclips: 670286 bp in 17479 read events
```

For paired-end data, a high proportion of reads should normally be properly paired, although the expected value depends on the sample and graph.

---

### Surjecting alignments to a linear reference

If a BAM/CRAM/SAM representation is required, Giraffe can project alignments onto reference paths in the GBZ graph. To do this we first need to set a path to be the reference.


!!! terminal "code"
    ```bash
    vg paths --metadata -x 5NM_pangenome.giraffe.gbz
    
	vg gbwt \
		-Z \
		--set-tag "reference_samples=NC_003112.2" \
    	--gbz-format \
    	-g 5NM_pangenome.giraffe.ref.gbz \
    	5NM_pangenome.giraffe.gbz

    vg paths --metadata -x 5NM_pangenome.giraffe.ref.gbz

    vg surject \
    	-x 5NM_pangenome.giraffe.ref.gbz \
    	--into-path NC_003112.2#1#1 \
    	--bam-output \
    	SRR10610805.5NM.gam \
    	> SRR10610805.5NM.bam
    ```
    
	```bash
    module load SAMtools/1.21-GCC-12.3.0
    samtools view SRR10610805.5NM.bam | less
    ```

!!! warning "Left-aligning"
	For best results, indel left-alignment is recommended (e.g. bamleftalign from FreeBayes, or vg surject --left-align in newer version), see methods section [Surjection to GRCh38 and indel realignment](https://www.nature.com/articles/s41586-023-05896-x)

!!! tip "Multiple Chromosomes"
    To use a sample with multiple paths as the reference, collate those paths and input as a file `--into-paths <ref_paths.txt>`

---

### Is this any better than just using a linear reference???

We can compare how a 'flat' reference performs compared to our alignments against the pangenome 

!!! terminal "code"
    ```bash
	awk '/^>/{if (seen++) exit} {print}'  ../5NM.fa > NC_003112.2.fa

	vg construct -r NC_003112.2.fa -m 1024 > NC_003112.2_flat.vg
	vg convert -f NC_003112.2_flat.vg > NC_003112.2_flat.gfa
	
	vg autoindex \
    	--workflow giraffe \
     	-g NC_003112.2_flat.gfa \
     	-p NC_003112.2_flat
    ```

!!! tip "Alternative ways to make a pangenome with vg"
	`vg construct` can be used to make a pangenome from a reference `-r` with variants called against that reference `-v`. This won't represent any complexity that isn't already captured in your variant calls.
     
!!! terminal "code"
    ```bash
	vg giraffe \
        -p \
        -t 8 \
        -Z NC_003112.2_flat.giraffe.gbz \
        -d NC_003112.2_flat.dist \
        -m NC_003112.2_flat.min \
        -f SRR10610805_1.fastq.gz \
        -f SRR10610805_2.fastq.gz \
        -b default \
        > SRR10610805.NC_003112.2.gam
        
    vg stats -a SRR10610805.NC_003112.2.gam > SRR10610805.NC_003112.2.stats
    cat SRR10610805.NC_003112.2.stats
	```
	
| Metric | 5NM (pangenome) | NC_003112.2 (single ref) |
|---|---|---|
| Total alignments | 1,139,592 | 1,139,592 |
| Total primary | 1,139,592 | 1,139,592 |
| Total secondary | 0 | 0 |
| Total aligned | 1,139,258 | 1,102,540 |
| Total perfect | 1,018,448 | 315,227 |
| Total gapless (softclips allowed) | 1,133,796 | 1,047,260 |
| Total paired | 1,139,592 | 1,139,592 |
| Total properly paired | 1,137,260 | 1,102,328 |
| Insertions | 8,652 bp / 2,525 events | 97,424 bp / 39,816 events |
| Deletions | 11,447 bp / 5,107 events | 120,580 bp / 42,322 events |
| Substitutions | 199,048 bp | 3,011,833 bp |
| Softclips | 670,286 bp / 17,479 events | 5,874,728 bp / 145,895 events |
| Total time | 1,063.94 s | 384.855 s |
| Speed | 1,071.11 reads/s | 2,961.1 reads/s |


## Summary

The recommended short-read workflow is:

```text
PGGB GFA
   │
   ▼
vg autoindex --workflow giraffe
   │
   ├── GBZ graph
   ├── distance index
   └── minimizer index
          │
          ▼
   vg giraffe
          │
          ├── GAM
		  ▼
   vg surject
          └── BAM/CRAM
```
