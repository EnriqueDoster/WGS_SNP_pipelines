# Overview of process for installing WGS tools

## To run all 4 pipelines on MSI
```
module load bowtie2
module load perl/modules.centos7.5.26.1
module load  ncbi_toolkit
# make sure samtools v1.9 is in your PATH
```


## CFSAN SNP

```
##################
conda install -c bioconda snp-pipeline
#https://snp-pipeline.readthedocs.io/en/latest/usage.html#step-by-step-workflows


# Dependencies
Make sure you have the following tools in your path
  * samtools v1.9


# On server, or singularity container, you have to specify the following CLASSPATHs
export CLASSPATH=/s/angus/index/common/tools/VarScan.jar:$CLASSPATH
export CLASSPATH=/s/angus/index/common/tools/picard.jar:$CLASSPATH
https://software.broadinstitute.org/gatk/download/archive
export CLASSPATH=/s/angus/index/common/tools/GenomeAnalysisTK.jar:$CLASSPATH

# ALso need to load bowtie2
module load bowtie2

## Run the pipeline using the main_combined_pipelines.nf script which uses the following command to run cfsansnp
cfsan_snp_pipeline run -m soft -o outputDirectory -s 63_samples /media/AngusWorkspace/WGS_test/ref_NC_003197.fasta

#
##
### If there is an error, you can find the working directory in the .nextflow.log and try running the individual steps of the pipeline by manually
##
# https://snp-pipeline.readthedocs.io/en/latest/usage.html


# Step 1 - Align the samples to the reference:
cat CFSAN_snp_results/sampleFullPathNames.txt | xargs -n 2 -L 1 cfsan_snp_pipeline map_reads --threads 30 CFSAN_snp_results/reference/ref_Ecoli_NC_000913.fasta

# Step 2 - Find the sites having high-confidence SNPs:
# Process the samples in parallel using all CPU cores
export SamtoolsMpileup_ExtraParams="-q 0 -Q 13 -A"
export VarscanMpileup2snp_ExtraParams="--min-var-freq 0.90"
cat sampleDirectories.txt | xargs -n 1 -P $numCores cfsan_snp_pipeline call_sites reference/lambda_virus.fasta

```




 ## Lyve-set 

 
```
 #https://github.com/lskatz/lyve-SET
# anaconda installation did not work correctly and was limited to v2 of lyve-SET instead of the more stable 1.4v
# Created singularity container that works correctly as of November 25, 2020
singularity pull shub://TheNoyesLab/WGS_SNP_pipelines:lyveset

```

Lyve-SET installation on UMN's servers

```

module load  ncbi_toolkit
# Load the correct perl module (the default does not support perl module installation)
module load perl/modules.centos7.5.26.1

# On MSI, installing perl modules had to be done from shell
perl -MCPAN -e shell
install XML::DOM::XPath
install Bio::FeatureIO

# Installation of perl modules
perl -MCPAN -Mlocal::lib -e 'CPAN::install(File::Slurp)'
perl -MCPAN -Mlocal::lib -e 'CPAN::install(URI::Escape)'
perl -MCPAN -Mlocal::lib -e 'CPAN::install(Bio::DB::EUtilities)'

# Download lyve SET
wget https://github.com/lskatz/lyve-SET/archive/v2.0.1.zip
unzip v2.0.1.zip
cd lyve-SET-2.0.1
make install-config
make install-perlModules
make install-smalt
make install-CGP
make install-SGELK
make install-vcftools
make install-samtools
make install-varscan
make install-snpEff
make install-bcftools
make install-raxml
make install-snap
make env

## Very important to make these two changes:
# Error with location of "varscan.sh"
cp /home/noyes046/edoster/.conda/envs/lyve_set_conda/bin/varscan /home/noyes046/edoster/.conda/envs/lyve_set_conda/bin/varscan.sh
# error with location of /panfs/roc/groups/11/noyes046/edoster/.conda/envs/lyve_set_conda/bin/VarScan.jar
cp /home/noyes046/edoster/.conda/envs/lyve_set_conda/share/varscan-2.4.4-0/VarScan.jar /home/noyes046/edoster/.conda/envs/lyve_set_conda/bin/VarScan.jar
```

Troubleshooting errors with Lyve-set

I can't seem to explain what caused the error below, as it wsa fixed by simply increased the number of threads from 4 to 14
```
set_findPhages.pl: main: Tempdir is /tmp/phast0yqfYw
set_findPhages.pl: phast: Running blastx against /usr/local/lyve-SET/scripts/../lib/phast/phast.faa
cat: '/tmp/phast0yqfYw/phastfg2fmJ/*.fna.bls': No such file or directory
set_findPhages.pl: main::phast: ERROR with cat on /tmp/phast0yqfYw/phastfg2fmJ/*.fna.bls at /usr/local/lyve-SET/scripts/set_findPhages.pl line 69, <GEN0> line 1.
QSUB ERROR
256
launch_set.pl: Schedule::SGELK::command: ERROR with command: Inappropriate ioctl for device
 
```



## Enterobase
* According to the enterobase documentation, you should be able to use a single command to download all software requirements (except Usearch). However, this doesn't work and I get errors for multiple tools.
```
   # Download EToKi github repository
    git clone https://github.com/zheminzhou/EToKi.git
    chmod +x /usr/local/EToKi/EToKi.py
    echo 'export PATH=$PATH:/usr/local/EToKi/' >> $SINGULARITY_ENVIRONMENT
    
    
    git clone https://github.com/EnriqueDoster/sing_biotools.git
    
    # Etoki installation
    python3 /usr/local/EToKi/EToKi.py configure --install --usearch /usr/local/sing_biotools/bin/usearch11.0.667_i86linux32
```

