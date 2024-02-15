<h1><b>QIIME2 Tutorial for 18S rRNA Data</h1></b>
<p>This repository contains a step-by-step tutorial for analyzing 18S rRNA data using QIIME2. The tutorial covers various aspects of data processing, 
including quality filtering, denoising, taxonomic classification, and phylogenetic analysis. Below is a detailed guide outlining each step of the tutorial.</p>
<h2>Step 1: Define the Path to Sequences Folder</h2>
<pre style="background-color: #000; color: #fff;">d="/Path_to_sequences_folder/"</pre>
<h2>Step 2: Counting Files and Directories</h2>
<pre style="background-color: #000; color: #fff;">t=$(ls $d | wc -l);</pre>
<h2>Step 3: Generating Sample Metadata</h2>
<pre style="background-color: #000; color: #fff;">paste <(ls $d) <(perl -le 'sub p{my $l=pop @_;unless(@_){return map [$_],@$l;}return map { my $ll=$_; map [@$ll,$_],@$l} p(@_);} @a=[A,C,G,T]; print join("", @$_) for p(@a,@a,@a,@a,@a,@a,@a,@a);' | awk -v k=$t 'NR<=k{print}') | awk 'BEGIN{print "sample-id\tbarcode-sequence\n#q2:types\tcategorical"}1' > sample_metadata.tsv
</pre>
<h2>Step 4: Generate Barcode Sequences</h2>
<pre style="background-color: #000; color: #fff;">(for i in $(ls $d); do bc=$(awk -v k=$i '$1==k{print $2}' sample_metadata.tsv); bioawk -cfastx -v k=$bc '{print "@"$1" "$4"\n"k"\n+";for(i=0;i< length(k);i++) {printf "#"};printf "\n"}' $d/$i/Raw/*_R1.fastq; done) > barcodes.fastq
</pre>
<h2>Step 5: Concatenate Forward Reads</h2>
<pre style="background-color: #000; color: #fff;">(for i in $(ls $d); do cat $d/$i/Raw/*_R1.fastq ; done) > forward.fastq</pre>
<h2>Step 6: Concatenate Reverse Reads</h2>
<pre style="background-color: #000; color: #fff;">(for i in $(ls $d); do cat $d/$i/Raw/*_R2.fastq ; done) > reverse.fastq</pre>

<h2>Step 7: Count Sequences in FASTQ File</h2>
<pre style="background-color: #000; color: #fff;">bioawk -cfastx 'END{print NR}' forward.fastq</pre>

<h2>Step 8: Compress FASTQ Files</h2>
<pre style="background-color: #000; color: #fff;">gzip *.fastq</pre>

<h2>Step 9: Organizing Compressed Files</h2>
<pre style="background-color: #000; color: #fff;">mkdir emp-paired-end-sequences; mv *.gz emp-paired-end-sequences/.</pre>

<h2>Step 10: Enable Qiime2 on cluster</h2>
<pre style="background-color: #000; color: #fff;">export PATH=/home/opt/miniconda2/bin:$PATH
source activate qiime2-2019.7</pre>

<h2>Step 11: Importing EMP Paired-End Sequences in Qiime2</h2>
<pre style="background-color: #000; color: #fff;">qiime tools import --type EMPPairedEndSequences --input-path emp-paired-end-sequences --output-path emp-paired-end-sequences.qza</pre>

<h2>Step 12: Demultiplexing and Error Correction of Paired-End Sequences in Qiime2</h2>
<pre style="background-color: #000; color: #fff;">qiime demux emp-paired --p-no-golay-error-correction --i-seqs emp-paired-end-sequences.qza --m-barcodes-file sample_metadata.tsv --m-barcodes-column barcode-sequence --o-per-sample-sequences demux.qza --o-error-correction-details demux-details.qza</pre>
<h2>Step 13: Generating Summary of Demultiplexed Sequences</h2>
<pre style="background-color: #000; color: #fff;">qiime demux summarize --i-data ./demux.qza  --o-visualization ./demux.qzv</pre>
<h2>Step 14: Unset MAFFT_BINARIES</h2>
<pre style="background-color: #000; color: #fff;">unset MAFFT_BINARIES</pre>
<h2>Step 15: Quality Filtering of Sequences</h2>
<pre style="background-color: #000; color: #fff;">qiime quality-filter q-score --i-demux demux.qza --o-filtered-sequences demux-filtered.qza --o-filter-stats demux-filter-stats.qza --p-min-quality 20</pre>
<h2>Step 16: Running Deblur Denoising with QIIME 2</h2>
<pre style="background-color: #000; color: #fff;">qiime deblur denoise-other --i-demultiplexed-seqs demux-filtered.qza --i-reference-seqs /home/opt/qiime2_databases/silva-138-99-seqs.qza --p-trim-length 145 --p-min-size 2 --p-min-reads 2 --o-representative-sequences rep-seqs-deblur.qza --o-table table-deblur.qza --p-sample-stats --o-stats deblur-stats.qza --p-jobs-to-start 5</pre>

<h2>Step 17: Assigning Taxonomy to Sequences</h2>
<pre style="background-color: #000; color: #fff;">qiime feature-classifier classify-consensus-vsearch --i-query rep-seqs-deblur.qza --i-reference-reads /home/opt/qiime2_databases/silva-138-99-seqs.qza --i-reference-taxonomy /home/opt/qiime2_databases/silva-138-99-tax.qza --p-perc-identity 0.99 --o-classification taxonomy.qza --p-threads 0</pre>

<h2>Step 18: Generate Taxonomy Bar Plot</h2>
<pre style="background-color: #000; color: #fff;">qiime taxa barplot --i-table table-deblur.qza --i-taxonomy taxonomy.qza --m-metadata-file sample_metadata.tsv --o-visualization taxa-bar-plots.qzv</pre>
<h2>Step 19: Generate Interactive Taxonomy Bar Plot</h2>
<pre style="background-color: #000; color: #fff;">qiime metadata tabulate --m-input-file taxa-bar-plots.qzv --o-visualization taxa-bar-plots.qzv</pre>
<h2>Step 20: Export Results</h2>
<pre style="background-color: #000; color: #fff;">
<p>qiime tools export --input-path table-deblur.qza --output-path exported/ </p>
<p>qiime tools export --input-path rep-seqs-deblur.qza --output-path exported/</p>
<p>qiime tools export --input-path taxonomy.qza --output-path exported/</p>
<p>qiime tools export --input-path taxa-bar-plots.qzv --output-path exported/</p>
</pre>
<h2>Step 21: Visualizing Taxonomy Bar Plot</h2>
<pre style="background-color: #000; color: #fff;">qiime tools view taxa-bar-plots.qzv</pre>
<h2>Step 22: Convert Qiime2 Artifact to Biom Format</h2>
<pre style="background-color: #000; color: #fff;">biom convert -i exported/feature-table.biom -o exported/feature-table.biom.tsv --to-tsv</pre>
<h2>Step 23: Convert Qiime2 Artifact to BIOM Format</h2>
<pre style="background-color: #000; color: #fff;">qiime tools export --input-path rep-seqs-deblur.qza --output-path exported/rep-seqs-deblur</pre>

<h2>Step 24: Generate Phylogenetic Tree</h2>
<pre style="background-color: #000; color: #fff;">
<p>qiime alignment mafft --i-sequences exported/rep-seqs-deblur/dna-sequences.fasta --o-alignment exported/aligned-rep-seqs.qza </p>
<p>qiime alignment mask --i-alignment exported/aligned-rep-seqs.qza --o-masked-alignment exported/masked-aligned-rep-seqs.qza </p>
<p>qiime phylogeny fasttree --i-alignment exported/masked-aligned-rep-seqs.qza --o-tree exported/unrooted-tree.qza </p>
<p>qiime phylogeny midpoint-root --i-tree exported/unrooted-tree.qza --o-rooted-tree exported/rooted-tree.qza </p>
</pre>

<h2>Step 25: Exporting Phylogenetic Tree</h2>
<pre style="background-color: #000; color: #fff;">qiime tools export --input-path exported/rooted-tree.qza --output-path exported/rooted-tree</pre>

<h2>Step 26: Visualizing Phylogenetic Tree</h2>
<pre style="background-color: #000; color: #fff;">qiime tools view exported/rooted-tree/tree.nwk</pre>
<h2>Step 27: Export Visualization</h2>
<pre style="background-color: #000; color: #fff;">qiime tools export --input-path taxa-bar-plots.qzv --output-path exported/taxa-bar-plots</pre>

