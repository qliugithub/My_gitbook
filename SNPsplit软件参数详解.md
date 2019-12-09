```{bash}
[qliu@sg22 SNPsplit-0.3.4]$ ./SNPsplit_genome_preparation --help

  SYNOPSIS:

SNPsplit_genome_preparation is designed to read in a variant call files from the Mouse Genomes Project (e.g. this latest
file: 'mgp.v5.merged.snps_all.dbSNP142.vcf.gz') and generate new genome versions where the strain SNPs are either incorporated
into the new genome (full sequence) or masked by the ambiguity nucleo-base 'N' (N-masking).

SNPsplit_genome_preparation may be run in two different modes:

Single strain mode:

   1) The VCF file is read and filtered for high-confidence SNPs in the strain specified with --strain <name>
     # 读取 VCF 文件并对其进行筛选，以获得使用 --strain name 指定的品种中的高置信度 SNP
   2) The reference genome (given with --reference_genome <genome>) is read into memory, and the filtered high-
      confidence SNP positions are incorporated either as N-masking (default) or full sequence (option --full_sequence)
     # 参考基因组(--reference_genome genome 中给出的)被读入存储器，并且过滤后的高置信度 SNP 位置被合并为 N-masked ( 默认 )或全序列(选项 --full_sequence ) 

Dual strain mode:
# 双品种模型

   1) The VCF file is read and filtered for high-confidence SNPs in the strain specified with --strain <name>
   
   2) The reference genome (given with --reference_genome <genome>) is read into memory, and the filtered high-
      confidence SNP positions are incorporated as full sequence and optionally as N-masking
      
   3) The VCF file is read one more time and filtered for high-confidence SNPs in strain 2 specified with --strain2 <name>
   # 再次读取 VCF 文件并筛选 --strain2 name 指定的品种 2 中的高置信度 SNP
   4) The filtered high-confidence SNP positions of strain 2 are incorporated as full sequence and optionally as N-masking
   # 品种 2 过滤后的高置信度 SNP 位置被合并为全序列，并且可选地合并为 N-masked
   5) The SNP information of strain and strain 2 relative to the reference genome build are compared, and a new Ref/SNP
      annotation is constructed whereby the new Ref/SNP information will be Strain/Strain2 (and no longer the standard
      reference genome strain Black6 (C57BL/6J))
   # 比较相对于参考基因组构建的品种/品种 2 的 SNP 信息，并构建新的 Ref/SNP 注释，由此新的 Ref/SNP 信息将是品种/品种 2 (并且不再是标准参考基因组品种 Black6 (C57BL/6J) )   
   6) The full genome sequence given with --strain <name> is read into memory, and the high-confidence SNP positions between
      Strain and Strain2 are incorporated as full sequence and optionally as N-masking
   # 用 --strain name 给出的全基因组序列被读入存储器，并且品种和品种 2 之间的高置信度 SNP 位置被合并为全序列，并且可选地合并为 N-masked

The resulting .fa files are ready to be indexed with your favourite aligner. Proved and tested aligners include Bowtie2,
Tophat, STAR, Hisat2, HiCUP and Bismark. Please note that STAR and Hisat2 may require you to disable soft-clipping, please
refer to the SNPsplit manual for more details

Both the SNP filtering as well as the genome preparation write out little report files for record keeping.
Please note that the SNPsplit genome preparation writes out files and creates new folders for the SNPs and new genomes into
the current working directory, so move there before invoking SNPsplit_genome_preparation.


  USAGE:    SNPsplit_genome_preparation  [options] --vcf_file <file> --reference_genome /path/to/genome/ --strain <strain_name>


--vcf_file <file>             Mandatory file specifying SNP information for mouse strains from the Mouse Genomes Project
                              (http://www.sanger.ac.uk/science/data/mouse-genomes-project). The file used and approved is 
                              called 'mgp.v5.merged.snps_all.dbSNP142.vcf.gz'. Please note that future versions
                              of this file or entirely different VCF files might not work out-of-the-box but may require some
                              tweaking. SNP calls are read from the VCF files, and high confidence SNPs are written into
                              a folder in the current working directory called SNPs_<strain_name>/chr<chromosome>.txt,
                              in the following format:

                                          SNP-ID     Chromosome  Position    Strand   Ref/SNP
                              example:   33941939        9       68878541       1       T/G


--strain <strain_name>        The strain you would like to use as SNP (ALT) genome. Mandatory. For an overview of strain names
                              just run SNPsplit_genome_preparation selecting '--list_strains'.

--list_strains                Displays a list of strain names present in the VCF file for use with '--strain <strain_name>'.

--dual_hybrid                 Optional. The resulting genome will no longer relate to the original reference specified with '--reference_genome'.
                              Instead the new Reference (Ref) is defined by '--strain <strain_name>' and the new SNP genome
                              is defined by '--strain2 <strain_name>'. '--dual_hybrid' automatically sets '--full_sequence'.

                              This will invoke a multi-step process:
                                 1) Read/filter SNPs for first strain (specified with '--strain <strain_name>')
                                 2) Write full SNP incorporated (and optionally N-masked) genome sequence for first strain
                                 3) Read/filter SNPs for second strain (specified with '--strain2 <strain_name>')
                                 4) Write full SNP incorporated (and optionally N-masked) genome sequence for second strain
                                 5) Generate new Ref/Alt SNP annotations for Strain1/Strain2
                                 6) Set first strain as new reference genome and construct full SNP incorporated (and optionally 
                                    N-masked) genome sequences for Strain1/Strain2
                                                            

--strain2 <strain_name>       Optional for constructing dual hybrid genomes (see '--dual_hybrid' for more information). For an
                              overview of strain names just run SNPsplit_genome_preparation selecting '--list_strains'.

--reference_genome            The path to the reference genome, typically the strain 'Black6' (C57BL/6J), e.g.
                              '--reference_genome /bi/scratch/Genomes/Mouse/GRCm38/'. Expects one or more FastA files in this folder
                              (file extension: .fa or .fasta).

--skip_filtering              This option skips reading and filtering the VCF file. This assumes that a folder named
                              'SNPs_<Strain_Name>' exists in the working directory, and that text files with SNP information
                              are contained therein in the following format:

                                          SNP-ID     Chromosome  Position    Strand   Ref/SNP
                              example:   33941939        9       68878541       1       T/G

--nmasking                    Write out a genome version for the strain specified where Ref bases are replaced with 'N'. In the
                              Ref/SNP example T/G the N-masked genome would now carry an N instead of the T. The N-masked genome
                              is written to a folder called  '<strain_name>_N-masked/'. Default: ON.

--full_sequence               Write out a genome version for the strain specified where Ref bases are replaced with the SNP base.
                              In the Ref/SNP example T/G the full sequence genome would now carry a G instead of the T. The full
                              sequence genome is written out to folder called '<strain_name>_full_sequence/'. May be set in
                              addition to '--nmasking'. Default: OFF. 

--no_nmasking                 Disable N-masking if it is not desirable. Will automatically set '--full_sequence' instead.

--genome_build [name]         Name of the mouse genome build, e.g. GRCm38. Will be incorporated into some of the output files.
                              Defaults to 'GRCm38'.

--help                        Displays this help information and exits.

--version                     Displays version information and exits.


                                                             Last modified: 1 March 2017
```



