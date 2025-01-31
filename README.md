# Systematic error workflow example

Here we provide files and commands necessary to reproduce the analyses described in virologival post when live. 

## Commands 

the snakefile is provided in the usher repository.  

    git clone https://github.com/yatisht/usher.git  
    cd usher/workflows/

get the files from this example and rename them as appropriate to match expectations of snakefile

    wget http://hgdownload.soe.ucsc.edu/goldenPath/wuhCor1/UShER_SARS-CoV-2/2021/10/26/public-2021-10-26.all.masked.pb.gz
    mv public-2021-10-26.all.masked.pb.gz public-latest.all.masked.pb.gz
    wget https://raw.githubusercontent.com/russcd/systematic_error_workflow_example/main/sanger_v4_samples.txt
    wget https://github.com/russcd/systematic_error_workflow_example/raw/main/sanger_v4_samples.fa.xz && unxz https://github.com/russcd/systematic_error_workflow_example/raw/main/sanger_v4_samples.fa.xz
  
get all samples from the latest public tree.  

    matUtils summary -i public-latest.all.masked.pb.gz -s public-latest.samples.txt
  
match up sanger IDs with sample IDs in the public tree.  
here “sanger_v4_samples.txt” is a text file indicating the IDs of V4 samples
    
    join <( paste <( perl -pe 's/\//\t/g' public-latest.samples.txt | cut -f2 ) <( cut -f1 public-latest.samples.txt ) | sort ) <( sort sanger_v4_samples.txt ) | cut -f2 -d' '> sanger_v4_sample_ids_to_prune.txt 
  
now prune the sanger ARTIC V4 samples from the tree.  
    
    matUtils extract -i public-latest.all.masked.pb.gz -s sanger_v4_sample_ids_to_prune.txt -p -o public-latest.pruned_sanger_v4.pb.gz
  
swap tree file name to match the input expected from our Snakefile.  
    
    mv public-latest.pruned_sanger_v4.pb.gz public-latest.all.masked.pb.gz
  
now run our workflow but add the sanger samples.  
    
    snakemake --use-conda --cores 4 --config FASTA="sanger_v4_samples.fa" RUNTYPE="systematic"
  
## Acknowledgements 
We thank COG-UK for providing the FASTA sequences. 
