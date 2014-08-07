# Code supplement for Devisetty et al., 2014

## SNP Detection

Perl modules and scripts:

- [SNPtools](https://github.com/mfcovington/SNPtools) v1.5.0 (at [commit 4b8a284](https://github.com/mfcovington/SNPtools/tree/4b8a284d6e38bb508385f46572b89a4df2f035c1))

Reference genome FASTA file:

- `B.rapa_genome_sequence_0830.fa`

Input alignment files:

- `R500.20140121.bam`
- `IMB211.20140121.bam`

### Set environmental variables

```sh
THREADS=    # Number of processors to use
BIN=        # Path to SNPtools/bin
OUT_DIR=    # Path to output directory

BAM_DIR=$OUT_DIR/bam
FA_DIR=$OUT_DIR/fa/B.rapa_genome_sequence_0830.fa

PAR1=R500
PAR2=IMB211
SEQ_LIST=A01,A02,A03,A04,A05,A06,A07,A08,A09,A10
```

### Find SNPs/indels

```sh
for ID in $PAR2 $PAR1; do
    $BIN/SNPfinder/snp_finder.pl \
      --id        $ID \
      --bam       $BAM_DIR/$ID.20140121.bam \
      --fasta     $FA \
      --seq_list  $SEQ_LIST \
      --out_dir   $OUT_DIR \
      --snp_min   0.33 \
      --indel_min 0.66 \
      --threads   $THREADS \
      --verbose

    for SNP_FILE in $OUT_DIR/snps/$ID.*.snps.nogap.gap.csv; do
        $BIN/SNPfinder/02.0.filtering_SNPs_by_pos.pl $SNP_FILE
    done
done

$BIN/Coverage/reciprocal_coverage.pl \
  --bam      $BAM_DIR/$PAR1.20140121.bam \
  --par1     $PAR1 \
  --par2     $PAR2 \
  --par1_bam $BAM_DIR/$PAR1.20140121.bam \
  --par2_bam $BAM_DIR/$PAR2.20140121.bam \
  --seq_list $SEQ_LIST \
  --out_dir  $OUT_DIR \
  --threads  $THREADS \
  --verbose

for CHR in A0{1..9} A10; do
    $BIN/SNPfinder/coverage_filter.pl \
      --chr  $CHR \
      --snp1 $OUT_DIR/snps/$PAR1.$CHR.snps.nogap.gap.FILTERED.csv \
      --snp2 $OUT_DIR/snps/$PAR2.$CHR.snps.nogap.gap.FILTERED.csv \
      --par1 $PAR1 \
      --par2 $PAR2 \
      --out  $OUT_DIR
done

Rscript --vanilla $BIN/SNPfinder/classify-snps.r $OUT_DIR/master_snp_lists > $OUT_DIR/classify.2014-02-02.log

mkdir $OUT_DIR/snp_master
for CHR in A0{1..9} A10; do
    grep -ve "NOT" \
    $OUT_DIR/master_snp_lists/master_snp_list.$PAR1.vs.$PAR2.$CHR.classified \
    > $OUT_DIR/snp_master/polyDB.$CHR &
done
```

### Noise-reduction

```sh
$BIN/Genotype/genotype_parents+nr.pl \
  --id1         $PAR1 \
  --id2         $PAR2 \
  --bam1        $BAM_DIR/$PAR1.20140121.bam \
  --bam2        $BAM_DIR/$PAR2.20140121.bam \
  --fasta       $FA \
  --seq_list    $SEQ_LIST \
  --out_dir     $OUT_DIR \
  --nr_ratio    0.9 \
  --threads     $THREADS
```