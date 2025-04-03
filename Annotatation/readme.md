Step 1: Obtain Annotation Files
You'll need:
-Reference genome (FASTA)
-Gene annotation (GTF/GFF3)
Step 2: Convert PLINK SNPs to a Mappable Format (VCF)
```bash
plink --bfile buffalo_ld_filtered --recode vcf --out buffalo_ld_filtered
```
