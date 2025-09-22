# Fhb1 CDS vs Glenn CDS BLAST Workflow

This workflow aligns **gene of interest CDS sequences** against the **Wheat high-confidence (HC) and low-confidence (LC) CDS sets** using BLAST+.  
Designed for HPC environments (Atlas/CCAST), but portable to any BLAST+ installation.

---

## Input files

- **Queries (Genes of interest CDS)**
/directory/this/saved/geneofinterest_gene_sequences/geneofinterest_cds_gene_sequences_wheat

- **Wheat high-confidence CDS FASTA**
/directory/this/saved/8-hc-lc-genes/wheatsubject.high.cds.fa

- **Glenn low-confidence CDS FASTA** *(optional)*
/directory/this/saved/8-hc-lc-genes/wheatsubject.low.cds.fa


---

## Environment setup
```bash
# Load BLAST module (Atlas/CCAST)
ml blast-plus/2.14.1
```

## 1. Create working folder
```bash
mkdir -p /directory/this/saved/blast-fhb1-wheat1-wheat2/geneofinterest_vs_wheat_cds
cd /directory/this/saved/blast-fhb1-wheat1-wheat2/geneofinterest_vs_wheat_cds
```

## 2. Build BLAST databases
```bash
# High-confidence CDS
makeblastdb \
  -in /directory/this/saved/8-hc-lc-genes/wheatsubject.high.cds.fa \
  -dbtype nucl -parse_seqids -out wheat_HC_cds

# Low-confidence CDS (if available)
makeblastdb \
  -in /directory/this/saved/8-hc-lc-genes/wheatsubject.low.cds.fa \
  -dbtype nucl -parse_seqids -out wheat_LC_cds
```

## 3. Run BLAST
### Against Wheat HC CDS
```bash
blastn \
  -query /directory/this/saved/geneofinterest_gene_sequences/geneofinterest_cds_gene_sequences_wheat \
  -db wheat_HC_cds \
  -task dc-megablast \
  -evalue 1e-20 \
  -perc_identity 85 \
  -max_hsps 1 \
  -max_target_seqs 10 \
  -num_threads 16 \
  -outfmt '6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore qlen slen' \
  -out geneofinterest_vs_wheatHC.txt
```

### Against Wheat LC CDS (optional)
```bash
blastn \
  -query /directory/this/saved/geneofinterest_cds_gene_sequences/geneofinterest_cds_gene_sequences_wheat \
  -db wheat_LC_cds \
  -task dc-megablast \
  -evalue 1e-20 \
  -perc_identity 85 \
  -max_hsps 1 \
  -max_target_seqs 10 \
  -num_threads 16 \
  -outfmt '6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore qlen slen' \
  -out geneofinterest_vs_wheatLC.tab
```

## Post-processing
### Tag results and merge HC+LC
```bash
awk 'BEGIN{OFS="\t"}{print $0,"HC"}' geneofinterest_vs_wheatHC.tab > geneofinterest_vs_wheatHC.tagged.tab
awk 'BEGIN{OFS="\t"}{print $0,"LC"}' geneofinterest_vs_wheatLC.tab > geneofinterest_vs_wheatLC.tagged.tab
cat geneofinterest_vs_wheatHC.tagged.tab geneofinterest_vs_wheatLC.tagged.tab > geneofinterest_vs_wheat_HC_LC.all.txt
```

### Extract top hit per query (by bitscore)
```bash
# HC only
sort -k12,12nr -k11,11g geneofinterest_vs_wheatHC.tab | awk '!seen[$1]++' > geneofinterest_vs_wheatHC.top1.tab

# HC+LC combined
sort -k12,12nr -k11,11g geneofinterest_vs_wheat_HC_LC.all.tab | awk '!seen[$1]++' > geneofinterest_vs_wheat_HC_LC.top1.tab
```

Maintainer:

Ruby Mijan



