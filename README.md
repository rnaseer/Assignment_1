# UNIX Assignment


## _Data Inspection_

```
wc -l fang_et_al_genotypes.txt snp_position.txt
```
Output (number of lines): 2783 fang_et_al_genotypes.txt, 984 snp_position.txt, 3767 total

```
awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
```
```
awk -F "\t" '{print NF; exit}' snp_position.txt
```
Output (number of columns): 986, 15

```
ls -lh fang_et_al_genotypes.txt
```
```
ls -l snp_positions.txt
```
Output (file size in bytes): 11M, 81K
## _Data Processing_

I began by extracting the appropriate data for Maize and Teosinte separately. At this point, I kept the header from `fang_et_al_genotypes.txt`. This data is not yet transposed.

```
awk 'NR==1 || $3 == "ZMMIL" || $3 == "ZMMLR" || $3 == "ZMMMR"{print}' fang_et_al_genotypes.txt > maize.txt
```
```
awk 'NR==1 || $3 == "ZMPBA" || $3 == "ZMPIL" || $3 == "ZMPJA"{print}' fang_et_al_genotypes.txt > teosinte.txt
```

Next, I transposed both output text files using `transpose.awk` provided.

```
awk -f transpose.awk teosinte.txt > transposed_teosinte.txt
```
```
awk -f transpose.awk maize.txt > transposed_maize.txt
```

After double-checking that the files are properly transposed using `head` and `cut -f 1`, I continued on to begin sorting.

I sorted `snp_positions.txt` using 
```
tail -n +2 snp_position.txt | sort -k1 > sorted_snp_position.txt
```
which disregards the header line. 

To sort the genotype files, I used 
```
awk 'NR==1{next} {print | "sort"}' transposed_teosinte.txt > sorted_transposed_teosinte.txt
```
which omits the first line, same with ` awk 'NR==1{next} {print | "sort"}' transposed_maize.txt > sorted_transposed_maize.txt`.

Files were joined using 
```
join sorted_snp_position.txt sorted_transposed_maize.txt > joined_maize.txt
```
```
join sorted_snp_position.txt sorted_transposed_teosinte.txt > joined_teosinte.txt
```
I once again double-checked the success of this using `wc -l` and `head -1`.

The next step was to extract data for each chromosome, which was done with 
```
awk '$3 == 1{print}' joined_maize.txt > chrom_1.txt
```
for each of the ten chromosomes, as well as for the `joined_teosinte.txt` file. 

To sort each file by increasing position I sorted by column four
```
sort -k4,4 chrom_1.txt > sorted_chrom_1.txt
```
for each chromosome file, within each group.

By default, missing data is encoded with '?', so to encode by '-' instead, I used the `sed` command to replace every '?' with a '-' in all the sorted chromosome files: 
```
sed 's/?/-/g' sorted_chrom_1.txt > replaced_chrom_1.txt
```

I subset the data again for 'missing' and 'unknown' locations:
```
awk '$3 == "unknown"' joined_teosinte.txt > chrom_unknown.txt
```
```
awk '$3 == "multiple"' joined_teosinte.txt > chrom_multiple.txt
```

All the files were organized into folders then the whole folder was added using `git add` and pushed to the origin master after a `git commit`.

## _Finding Files_

Within the `maize_files` or `teosinte_files` folders, you will find `chrom_multiple.txt` which includes data with multiple positions, and `chrom_unknown.txt` which includes data with unknown positions.

The `sorted_chrom_data` folder includes sorted chromosome files.

The `missing_replaced_chrom_data` folder includes sorted chromosome files with the missing data replaced with a '-'.
