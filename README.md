# flo - basic gff annotations lift over using chain files

Lift over is a way of mapping annotations from one genome assembly to another.
The idea "lift over" is same as what tools like UCSC LiftOver, NCBI's LiftUp
web service do. However, NCBI and UCSC's web services are available only for
a limited number of species.

To perform lift over locally, one can use UCSC chain files ([Kent et al 2003][kent2003])
with programs such as UCSC's liftOver or [CrossMap][crossmap]. A chain file captures
large, homologous segments between two genomes as chains of gapless blocks of
alignment. One way of generating chain files is using [this bash script][kent-script]
and [UCSC tools][ucsc-tools].

flo is an implementation of the above script in Ruby programming language. Further,
both liftOver and CrossMap process GFF files line by line instead of transcripts as
a whole. This results in some non-biologically meaningful output. flo provides a
basic filtering of UCSC liftOver's GFF output.

We created flo for our work on the fire ant genome. If you use flo, please cite
the following paper:

> The fire ant social chromosome supergene variant Sb shows low diversity but
> high divergence from SB. 2017. R Pracana, A Priyam, I Levantis, Y Wurm.
> [Molecular Ecology, doi: 10.1111/mec.14054](http://onlinelibrary.wiley.com/doi/10.1111/mec.14054/full).

[Using flo](#using-flo) | [Results & discussion](#results--discussion) | [Tweaking flo](#tweak-flo)

## Using flo

To use flo you must have Ruby 2.0 or higher and the BioRuby gem. Ruby 2.0 can
be installed through package managers on Linux and is available by default on
Mac. To install BioRuby gem:

    sudo gem install bio

flo additionally requires a few programs from [UCSC tools][ucsc-tools], [GNU
Parallel][gnu-parallel] and [genometools][genometools]. These can be
installed in any directory by running 'scripts/install.sh' script
after you have downloaded flo:

    wget -c https://github.com/yeban/flo/archive/master.tar.gz -O flo.tar.gz
    tar xvf flo.tar.gz
    mv flo-master flo

It's best to run flo in a new directory - we will call it project dir:

    mkdir flo_species_name
    cd flo_species_name

Copy over example configuration file from where you installed flo to
project dir:

    cp /path/to/flo/opts_example.yaml flo_opts.yaml

Install flo's dependencies in `ext/` directory in the project dir:

    /path/to/flo/scripts/install.sh

Now edit `opts.yaml` to indicate:
1. Location of source and target assembly in FASTA format (required).
2. Location of GFF3 file(s) containing annotations on the source
   assembly. If this is omitted, flo will stop after generating
   the chain file.
3. BLAT parameters (optional). By default the target assembly is
   assumed to be of the same species. If the target assembly is
   a different (but closely related) species, you may want to
   lower `minIdentity`.
4. Number of CPU cores to use (required - not auto detected). This  
   cannot be greater than the number of scaffolds in the target assembly.

Here, it's important to note that flo can only work with transcripts
and their child exons and CDS. Transcripts can be annotated as: mRNA,
transcript, or gene. However, if you have a 'gene' annotation for
each transcript, you will need to remove that:

    /path/to/flo/gff_remove_feats.rb gene xx_genes.gff \
    > xx_transcripts.gff

Alternatively, if you have more than one transcript annotated for
each gene, you can select the longest transcript for each gene to
work with:

    /path/to/flo/gff_longest_transcripts.rb xx_genes.gff \
    > xx_longest_transcripts.gff

Finally, run flo as:

    rake -f /path/to/flo/Rakefile

A common problem encountered is that 1st column of GFF file doesn't match
chromosome, or scaffold, or contig id in the source assembly. In this case
`liftOver` will generate an empty output file. flo stops at this point. You
can fix the GFF file and resume flo by running the above command.

flo writes all output to a directory called `run/` in the current directory.
The chain file generated by flo can be found at `run/liftover.chn`. If flo
completed successfully, a directory is created for each given GFF3 file in
'run/' that contains:
1. `lifted.gff3` and `unlifted.gff3` - liftOver's output
2. `lifted_cleaned.gff` - lifted.gff3 cleaned by flo -> final output
3. `unmapped.txt` - id of all transcripts that couldn't be lifted.
   Transcripts present in this list and also found in output gff
   should be considered partial.

## Results & discussion
Both strengths and weaknesses of flo largely reflect that of the underlying
tools - the chain file and UCSC liftOver. In general, gaps and errors in
assemblies may split a long chain. Gene models that are split across
different chains as well as those that are duplicated in the target
assembly are not lifted.

- For an ant genome (~350 Mb) we saw 90% annotations map identically to
the new assembly (unpublished result).
- flo has been used in:
  - [_De novo_ genome assembly of a _Plasmodium falciparum_ NF54 clone using single-molecule real-time sequencing. Bryant et al 2018. Genome Announcements 6(5): e01479-17.](http://genomea.asm.org/content/6/5/e01479-17.short)
  - [Chromosome-scale scaffolding of the black raspberry (_Rubus occidentalis_ L.) genome based on chromatin interaction data. Jibran et al 2018. Horticulture Research 5(1): 8.](https://www.nature.com/articles/s41438-017-0013-y)
  - [First draft assembly and annotation of the genome of a California Endemic Oak _Quercus lobata_ Née (Fagaceae). Sork et al 2016. G3: Genes, Genomes, Genetics 6(11): 3485-3495.](https://doi.org/10.1534/g3.116.030411)

## Tweak flo
If you would like to optimise how chain files are created:
- UCSC wiki and website is an amazing resource to learn about BLAT and
  chain files. Don't forget to read Kent 2003 paper cited above first.
- Read the `Rakefile` from top to bottom. Ruby is similar, yet simpler
  compared to Perl and bash.

You can test things by lifting annotations between the same assembly.

---
Copyright 2017 Anurag Priyam, Queen Mary University of London

[kent-script]: http://hgwdev.cse.ucsc.edu/~kent/src/unzipped/hg/doc/liftOver.txt
[kent2003]: http://www.pnas.org/content/100/20/11484.full
[ucsc-tools]: http://hgdownload.cse.ucsc.edu/admin/exe/
[gnu-parallel]: https://www.gnu.org/software/parallel/
[genometools]: http://genometools.org/
[crossmap]: http://crossmap.sourceforge.net/
