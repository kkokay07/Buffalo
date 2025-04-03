# Buffalo

1. Install gdown
```bash
pip install gdown
```
2. Download from google drive using gdown
```bash
gdown "https://drive.google.com/uc?id=1WamXZwP-UymzvwwK3Y01xxCx14WNM-9g"
```
3. Extract
```bash
tar -xzvf NGS_P2025_02_238.Buffalo.tar.gz
```
4. Convert to B files
```bash
/home/info_lab/tools/plink --vcf TotalRawSNPs.vcf.gz --make-bed --out buffalo --allow-extra-chr
```
5. Check no. of SNPs in the file
```bash
wc -l buffalo.bim
```
6. Convert RefSeq ID to Chromosomes and make lenght of allele to 1 nucleotide
```bash
sbcb$ awk 'NR==FNR {map[$2] = $1; next} 
{
    if ($1 in map) $1 = map[$1]; 
    if (length($5) > 1 || length($6) > 1) { $5 = "N"; $6 = "N" } 
    print $0
}' chrom_map.txt buffalo.bim > buffalo_fixed.bim
```
Example chrom_map.txt
(The following file can be directly used if reference assembly used is NDDB_SH_1)
```bash
1 NC_059157.1
2 NC_059158.1
3 NC_059159.1
4 NC_059160.1
5 NC_059161.1
6 NC_059162.1
7 NC_059163.1
8 NC_059164.1
9 NC_059165.1
10 NC_059166.1
11 NC_059167.1
12 NC_059168.1
13 NC_059169.1
14 NC_059170.1
15 NC_059171.1
16 NC_059172.1
17 NC_059173.1
18 NC_059174.1
19 NC_059175.1
20 NC_059176.1
21 NC_059177.1
22 NC_059178.1
23 NC_059179.1
24 NC_059180.1
X NC_059181.1
MT NC_049568.1
```
Before applying genotype/individual missingness filters, first clean the dataset by running:
```bash
plink --bfile buffalo_fixed --make-bed --out buffalo_cleaned
```
Apply Missing Genotype & Individual Filters
```bash
plink --bfile buffalo_cleaned --geno 0.05 --make-bed --out buffalo_filtered_geno
```
```bash
plink --bfile buffalo_filtered_geno --mind 0.10 --make-bed --out buffalo_final
```
Apply minor allele frequency (maf) criteria
```bash
plink --bfile buffalo_final --maf 0.05 --make-bed --out buffalo_maf_filtered
```
Generate summary on maf filtering
```bash
plink --bfile buffalo_maf_filtered --freq --out maf_summary
```
Perform LD Pruning
```bash
plink --bfile buffalo_maf_filtered --indep-pairwise 50 5 0.2 --out buffalo_ld
```
>--indep-pairwise 50 5 0.2

>50 → Window size (SNPs)

>5 → Step size (SNPs)

>0.2 → R² threshold (SNP pairs with R² > 0.2 are pruned)

>This creates two output files:

>buffalo_ld.prune.in → List of SNPs to keep.

>buffalo_ld.prune.out → List of SNPs to remove.
use the .prune.in file to retain only independent SNPs
```bash
plink --bfile buffalo_maf_filtered --extract buffalo_ld.prune.in --make-bed --out buffalo_ld_filtered
```
Remove "N" (missing or ambiguous nucleotides)
```bash
plink --bfile buffalo_ld_filtered --exclude <(awk '$5=="N" || $6=="N" {print $2}' buffalo_ld_filtered.bim) --make-bed --out buffalo_ld_noN
```


