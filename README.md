# Metagenomic assembly pipeline

Assembly exercise workflow for Nordic Summer School on Computational Microbiome Research, 2024. 

## Data

Illumina, Nanopore and Pacbio RS II sequencing data from 12 species mock community: 
[Shotgun metagenome data of a defined mock community using Oxford Nanopore, PacBio and Illumina technologies](https://doi.org/10.1038/s41597-019-0287-z)

__Original data__

| Accession   | Platform | Data in Gb |
| -------- | ------- | --- 
| SRR8073715 | Pacbio RS II | 1.1 |
| SRR8073714 | PacBio RS II | 1.5 |
| SRR8073713 | Nanopore | 3.7 |
| SRR8073716 | Illumina | 64 |

## Data download 

Download read files and pack them.

```bash
 for acc in $(cat accessions.txt)
 do	
  fasterq-dump -e $SLURM_CPUS_PER_TASK -O data --split-3 --skip-technical --progress $acc
  pigz ${acc}*.fastq
 done
```

## Subsample Illumina data

Keep 10 % of original reads (~6.4Gb).

```bash
 seqkit sample -s 100 data/SRR8073716_1.fastq.gz  -p 0.1 |pigz -c > data/SRR8073716_sub_1.fastq.gz
 seqkit sample -s 100 data/SRR8073716_2.fastq.gz  -p 0.1 |pigz -c > data/SRR8073716_sub_2.fastq.gz
```

## Assemblies

Illumina only with spades

```bash
 spades.py --meta -1 data/SRR8073716_sub_1.fastq.gz -2 data/SRR8073716_sub_2.fastq.gz --threads $SLURM_CPUS_PER_TASK --only-assembler -o illumina_only
```

Pacbio only with metaflye (use both pacbio runs). 

```bash
 ~/projappl/flye/bin/flye --meta --pacbio-raw data/SRR8073715.fastq.gz data/SRR8073714.fastq.gz --threads $SLURM_CPUS_PER_TASK --out-dir pacbio_only
```

Nanopore only with metaflye.

```bash
 ~/projappl/flye/bin/flye --meta --nano-raw data/SRR8073713.fastq.gz --threads $SLURM_CPUS_PER_TASK --out-dir nanopore_only
 ```

Illumina + Pacbio hybrid with spades

 ```bash
 spades.py --meta -1 data/SRR8073716_sub_1.fastq.gz -2 data/SRR8073716_sub_2.fastq.gz --pacbio data/SRR8073715.fastq.gz --pacbio data/SRR8073714.fastq.gz --threads $SLURM_CPUS_PER_TASK --only-assembler -o pacbio_hybrid
 ```

Illumina + Nanopore hybrid with spades.

 ```bash
 spades.py --meta -1 data/SRR8073716_sub_1.fastq.gz -2 data/SRR8073716_sub_2.fastq.gz --nanopore data/SRR8073713.fastq.gz --threads $SLURM_CPUS_PER_TASK --only-assembler -o nanopore_hybrid
 ```

 ## Download links for the assemblies

Illumina only: https://a3s.fi/antkark-2001183-pub/illumina_only.fasta
Nanopore only: https://a3s.fi/antkark-2001183-pub/nanopore_only.fasta
Pacbio only: https://a3s.fi/antkark-2001183-pub/pacbio_only.fasta
Nanopore hybrid: https://a3s.fi/antkark-2001183-pub/nanopore_hybrid.fasta
Pacbio hybrid: https://a3s.fi/antkark-2001183-pub/pacbio_hybrid.fasta

## Assembly stats
 
 Assembly statistics with metaquast for all assemblies. 

 ```bash 
 metaquast.py -o assembly_stats --threads $SLURM_CPUS_PER_TASK  --max-ref-number 0 */contigs.fasta */assembly.fasta
```

## Software versions:
- fasterq-dump v.3.0.0
- seqkit v.0.16.0
- spades v.4.0.0
- flye v.2.9.4-b1799
- metaquast v.5.2.0
