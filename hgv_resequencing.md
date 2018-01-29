HGSC HgV: Human Resequencing Protocol

HgV is the HGSC workflow management system for all primary and secondary genome sequence analysis. HgV implements tiered XML protocols, manages LIMS communications, and provides verbose logging and stable reproducibility. All single-lane and multiplexed HiSeq X sequencing events are subject to the HgV Human Resequencing Protocol, which includes conversion to FASTQ, BWA-MEM mapping, GATK recalibration and realignment, and ATLAS SNV and indel variant calling. Complete protocol details are supplied below.

Protocol Summary: All WGS FASTQs are aligned via BWA-MEM to the GRCh37 decoy reference. The resulting BAMs are first assessed for contamination with VerifyBAMID using a set of HapMap-derived MAFs. BAMs are then GATK-recalibrated and realigned using dbSNP142b37, 1KGP Phase 1 and Mills gold standard indels. SNVs and indels are called independently by ATLAS and annotated by Cassandra. BAM files are "finished" by stripping multiple tags and binning the quality scores, resulting in final deliverables of ~60 GB BAM and ~1 GB SNV and indel VCFs per sample.

1. Reference files

    1. Human Reference Genome [GRCh37d](ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/phase2_reference_assembly_sequence/)

    2. dbSNP [142b37](http://www.ncbi.nlm.nih.gov/SNP/snp_summary.cgi?view+summary=view+summary&build_id=142)

    3. 1KGP_Indels [phase1.indels.b37](http://gatkforums.broadinstitute.org/gatk/discussion/1213/what-s-in-the-resource-bundle-and-how-can-i-get-it)

    4. Mills_Indels [gold_standard.indels.b37](http://gatkforums.broadinstitute.org/gatk/discussion/1213/what-s-in-the-resource-bundle-and-how-can-i-get-it)

    5. VIPbedfile (provided by TOPmed, available upon request)

2. Program Versions

    6. Picard [1.128](https://github.com/broadinstitute/picard/releases/tag/1.128)

    7. FastQC [0.11.2](http://www.bioinformatics.babraham.ac.uk/projects/download.html#fastqc)

    8. BWA [0.7.12](http://sourceforge.net/projects/bio-bwa/files/)

    9. verifyBAMID [1.1.0](https://github.com/statgen/verifyBamID/releases/tag/v1.1.0)

    10. sambamba [0.5.9](https://github.com/lomereiter/sambamba/releases/tag/v0.5.9)

    11. GATK [3.4.0](https://github.com/broadgsa/gatk-protected/releases/tag/3.4)

    12. ATLAS v0.0.1-rc7

    13. Cassandra [15.4.29](https://www.hgsc.bcm.edu/software/cassandra)

    14. bamUtil [1.0.13](https://github.com/statgen/bamUtil/releases/tag/v1.0.13)

3. Human Resequencing Protocol

    15. Base Stats

        1. Picard Sequence Analyzer

        2. FastQC --disableSeqIDCheck

    16. Mapping

        3. BWA mem -M -t 8 -R |

            1. samtools view -F 256 -Sbh

            2. samtools fixmate

        4. sambamba markdup -t 8 -p

        5. sambamba index -t 8

    17. Contamination

        6. verifyBamID --vcf HapMapMAF.vcf --verbose --ignoreRG

    18. GATK Recal/Realign

        7. GenomeAnalysisTK.jar -T RealignerTargetCreator /

            3. -R hs37d5.fa 

            4. -nt 8 

            5. --downsampling_type NONE

            6. -known 1KGP_Indels

            7. -known Mills_Indels

        8. GenomeAnalysisTK.jar -T IndelRealigner /

            8. -R hs37d5.fa

            9. --downsampling_type NONE

            10. --consensusDeterminationModel USE_READS

            11. -knownSites dbsnp_142_b37

            12. -knownSites 1KGP_Indels

            13. -knownSites Mills_Indels

        9. GenomeAnalysisTK.jar -T BaseRecalibrator /

            14. -R hs37d5.fa 

            15. -nct 8 

            16. --downsampling_type NONE

            17. -knownSites 1KGP_Indels

            18. -knownSites Mills_Indels

            19. -cov ReadGroupCovariate

            20. -cov QualityScoreCovariate 

            21. -cov CycleCovariate 

            22. -cov ContextCovariate

        10. GenomeAnalysisTK.jar -T PrintReads /

            23. -R hs37d5.fa

            24. --emit_original_quals 

            25. --downsampling_type NONE

            26. -nct 4

    19. SNP/Indel Calling

        11. Atlas --ref hs37d5.fa  --runGL true

    20. Annotation

        12. Cassandra.jar -t Annotate -n 8 -a

    21. Finishing

        13. bamUtil/bam squeeze /

            27. --keepDups 

            28. --binMid 

            29. --rmTags "BI:Z;BD:Z;PG:Z" 

            30. --binQualS 2,3,10,20,25,30,35,40,50

