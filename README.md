# PPTe

**PPTe** (PolymorPhic Transposable elements simulator) is a Python toolkit for simulating polymorphic transposable elements (pTEs) in genomes. It supports generating TE sequence pools, simulating real pTEs, generating simulated VCF and genome sequences, and comparing predicted VCFs from other TE genotyping tools with the simulated VCF file.

---

## Installation

You can install PPTEs from source using `pip`:

```bash
conda install bioconda::mason
conda install bioconda::pbsim3
pip install PPTE
```
##  Dependencies  
Read simulation:
- Mason2 (short reads)
- pbsim3 (long reads)
  
Python libraries:
- Python 3.8+
- numpy
- biopython
- pysam

## Quick start
Example data can be found in the **testData** directory   

**1. Simulate 100 pTE from known TE insertions and deletions**
```bash
ppte TEreal --knownINS MEI.fa --knownDEL rmsk.txt --CHR 21 --nTE 100
```
- `MEI.fa` is known pTE insertion, from paper [Logsdon, G.A. et al. Nature, 2025](https://www.nature.com/articles/s41586-025-09140-6)  
- `rmsk.txt` is known repeats annotation from UCSC hgTables.

**2. Simulate 100 pTE from known TE deletions and random TE insertions**
```bash
ppte TErandom --consensus TEconsensus.fa --knownDEL rmsk.txt --CHR chr21 --nTE 100
```
- `TEconsensus.fa` is human TE consensus sequences from Dfam

**3. Simulate 100 genomes with 100 pTE**  
```bash
ppte simulate --ref chr21_tiny.fa --bed real.bed --num 2 --pool MEI.fa
ppte simulate --ref chr21_tiny.fa --bed real.bed --num 2 --pool MEI.fa --diverse --diverse_config diverse.config
```
- `chr21_tiny.fa` is the reference sequence
- `real.bed` is the position of pTE events that generated from `ppte TEreal`
- `diverse` : Introduce sequence diversity among individuals for the same TE event (which is suitable for evaluating methods that require a TE panel as input)
- `diverse_config` : A configuration file of parameters for introducing sequence diversity among individuals for the same TE event (optional)

**4. Generate sequencing reads from simulated genome** 
```bash
ppte readsim --type short --genome random.fa --depth 1 
ppte readsim --type long --genome random.fa --depth 1
```
- `type` : short reads or long reads

## Flowchart
![flowchart](https://github.com/JanMiao/PPTE/blob/main/flowchart.png)
- The known TE deletion information can be obtained from [UCSC annotaion file (.txt)](https://genome.ucsc.edu/cgi-bin/hgTables) or [repeatmasker annotation (.out)  ](https://www.repeatmasker.org/genomicDatasets/RMGenomicDatasets.html)
- The known TE insertion position can be obtained from our pre-built dataset (data/MEI_Callset_GRCh38.ALL.20241211.fasta). Any TE insertion sequence is acceptable , as long as the sequence ID follows the naming format **CHR-POS-ID**, e.g., **chr1-683234-AluSp**

## Usage
PPTEs provides five main command-line subcommands:
```bash
ppte <subcommand> [options]
```

### 1. TErandom
Generate pTE position from known deletion sites and random TE insertion.

**Required arguments:**
- `consensus` : Path to TE consensus FASTA file
- `knownDEL` : Input known TE deletion file (RepeatMasker .out or UCSC .txt)
- `nTE` : Number of polymorphic TE (pTE) insertions to simulate (default: 500)
- `ins-ratio` : Proportion of insertion events among all simulated pTE (0-1, default: 0.4)
- `CHR` : Chromosome to simulate TE insertions on (e.g., chr21 or 21)
  
**Optional arguments:**
- `num` : Number of TE sequences to generate (default: 1000)
- `outprefix` : Output prefix for TE pool FASTA (default: TEpool)
- `snp-rate` : SNP mutation rate per base (default: 0.02)
- `indel-rate` : INDEL mutation rate per base (default: 0.005)
- `indel-ins` : Proportion of indels that are insertions (default: 0.4)
- `indel-geom-p` : Geometric distribution parameter for indel lengths (default: 0.7)
- `truncated-ratio` : Proportion of sequences to truncate (default: 0.3)
- `truncated-max-length` : Maximum proportion of sequence to truncate (default: 0.5)
- `polyA-ratio` : Proportion of sequences to add polyA tail (default: 0.8)
- `polyA-min` : Minimum polyA length (default: 5)
- `polyA-max` : Maximum polyA length (default: 20)
- `seed` : Random seed (default: None)
- `verbose` : Disable verbose logging (default: True)


### 2. TEreal
Automatically generate pTE positions from RepeatMasker or UCSC repeat annotations.

**Required arguments:**  
- `knownINS` : Known TE insertion file (FASTA)  
- `knownDEL` : Known TE deletion file (from RepeatMasker `.out` or UCSC `.txt`)  
- `CHR` : Chromosome used to simulate pTE  

**Optional arguments:**  
- `outprefix` : Output prefix for BED file (default: real)  
- `nTE` : Number of pTE insertions (default: 500)  
- `ins-ratio` : Proportion of insertion events (default: 0.4)  
- `seed` : Random seed (default: None)  
- `verbose` : Disable verbose logging (default: True)  


### 3. simulate
Simulate pTE insertions/deletions and generate VCF and modified genome FASTA.

**Required arguments:**
- `ref` : Reference genome FASTA  
- `te-pool` : TE pool FASTA  
- `bed` : BED file of TE positions (can be generated by `ppte TEreal`)  
- `num` : Number of simulated genomes

**Optional arguments:**  
- `diverse` : Introduce sequence diversity among individuals for the same TE event, which is suitable for evaluating methods that require a TE panel as input(default: False)
- `diverse_config` : Path to a configuration file for introducing sequence diversity among individuals for the same TE event (default: None; requires --diverse)
- `outprefix` : Output prefix (default: Sim)  
- `af-min / --af-max` : Min/max allele frequency (default: 0.1/0.9)  
- `tsd-min / --tsd-max` : Min/max TSD length (default: 5/20)  
- `sense-strand-ratio` : Proportion of sense-strand insertions (default: 0.5)  
- `seed` : Random seed (default: None)  
- `verbose` : Disable verbose logging (default: True)  


### 4. readsim
Generate short or long reads from the simulated genome.

**Required arguments:**
- `type` : Type of reads to simulate (short or long)  
- `genome` : Reference genome file (FASTA) where reads will be simulated from  
- `depth` : Depth of simulated reads    

**Optional arguments(long reads only):**  
- `Lerror` : Sequencing error rate for long reads (default: 0.15)  
- `Lmean` : Average read length for long reads (default: 9000)   
- `Lstd` : Standard deviation of read length for long reads (default: 7000)    

**Optional arguments(short reads only):**  
- `length` : Read length (default: 150)  
- `Fmean` : Average fragment length (default: 300)   
- `Fstd` : Fragment length standard deviation (default: 30)    

**Optional arguments(general):**  
- `seed` : Random seed for reproducibility (default: None)

### 5. compare
Compare predicted VCF to the simulated VCF.

**Required arguments:**  
- `truth` : Ground truth VCF (generated by `ppte simulate`)  
- `pred` : Predicted VCF  
- `outprefix` : the Matched variants in two VCF file  
- `truthID` : Sample ID in truth VCF  
- `predID` : Sample ID in predicted VCF  

**Optional arguments:**  
- `nHap` : Ploidy level, e.g., 2 for Humans  (default: 2)  
- `max_dist` : Maximum allowed distance for variant matching (default: 100 bp)  

**Example**:
```bash
ppte compare --truth sim.vcf --pred variants.vcf --truthID Hap1_Hap2 --predID Sample
```
In simulation files, genomes are named Hap1, Hap2, etc.; for polyploids, combine haplotype IDs with `_` for one individual.  

