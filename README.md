# Download the gene's file from NCBI

Use this to download a fasta file of the gene's scaffold

      ncbi-acc-download -F fasta -m protein XP_032222198.1

# Prepare the proteomes given by Dr. Rest

Uncompress the proteomes from the folder given by Dr. Rest

      gunzip proteomes/*.gz
      
Then put all of these files into a single file
      
      cat  proteomes/* > allprotein.fas

# Set up the BLAST database using the 7 proteomes

Use the following command

      makeblastdb -in allprotein.fas -parse_seqids -dbtype prot
      
# BLAST the gene against the database to identify homologs
      
       blastp -db allprotein.fas -query XP_032222198.1.fa -outfmt 0 -max_hsps 1 -out ABC.blastp.typical.out

# Filtering the BLAST output for an e-value of <1e^-31

      awk '{if ($6<0.0000000000000000000000000000001)print $1 }' ABC.blastp.detail.out > ABC.blastp.detail.filtered.out
      
# Align the gene family sequences using MUSCLE
      
       muscle -in ABC.blastp.detail.filtered.fas -out ABC.blastp.detail.filtered.aligned.fas
       
# Remove aligned sequences with too many gaps

      t_coffee -other_pg seq_reformat -in ABC.blastp.detail.filtered.aligned.fas -action +rm_gap 50 -out ABC.blastp.detail.filtered.aligned.r50.fas
      
# Replace spaces in annotations with underscores

      sed "s/ /_/g" ABC.blastp.detail.filtered.aligned.fas > ABC.blastp.detail.filtered.aligned_.fas

# Create a phylogeny tree using IQTREE

Use the following command to create an unrooted tree

      iqtree -s ABC.blastp.detail.filtered.aligned_.fas -nt 2

Then, use the following command to create a midpoint rooted tree

      gotree reroot midpoint -i ABC.blastp.detail.filtered.aligned_.fas.treefile -o ABC.blastp.detail.filtered.aligned_.fas.midpoint.treefile

Look at the midpoint rooted treefile

      nw_display -s  ABC.blastp.detail.filtered.aligned_.fas.midpoint.treefile -w 1000 -b 'opacity:0' >  ABC.blastp.detail.filtered.aligned_.fas.midpoint.treefile.svg

# Use Notung to reconcile the tree

      java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g ABC.genes.tre --reconcile --speciestag prefix --savepng --events

Use the following command to print out the tree with Notung support

      nw_display -s  maguk.genes.ufboot.MendozaRoot.treefile -w 1000 -b 'opacity:0' >  maguk.genes.ufboot.MendozaRoot.treefile.svg

# Evaluate branch support with bootstrap

Use the following command to first evaluate branch support

      iqtree -s ABC.blastp.detail.filtered.aligned_.fas -bb 1000 -nt 2 -m VT+F+R4 -t ABC.genes.tre -pre ABC.genes.ufboot

Then, use the following command to reroot to midpoint rooting

      gotree reroot midpoint -i ABC.genes.ufboot.treefile -o ABC.midpoint.ufboot

# Run iprscan5

Set up a directory to help organize your output using the following command
      
      mkdir ~/labs/lab8-L03-JackMcCarron/ABC
      
Run iprscan5

      iprscan5   --email jack.mccarron@stonybrook.edu  --multifasta --useSeqId --sequence   ~/labs/lab5-L03-JackMcCarron/maguk.blastp.detail.filtered.fas

Concatenate all files into a single file

      cat ~/labs/lab8-L03-JackMcCarron/ABC/*.tsv.tsv > ~/labs/lab8-L03-JackMcCarron/ABC.domains.all.tsv

Use the following command to focus only on domains defined by the pfam database

      grep Pfam ~/labs/lab8-myusername/maguk.domains.all.tsv >  ~/labs/lab8-myusername/maguk.domains.pfam.tsv

# Using Evolview to plot pfam domains

Use the following code to rearrange the interproscan's output

      awk 'BEGIN{FS="\t"} {print $1"\t"$3"\t"$7"@"$8"@"$5}' ~/labs/lab8-L03-JackMcCarron/ABC.domains.pfam.tsv | datamash -sW --group=1,2 collapse 3 | sed 's/,/\t/g' | sed 's/@/,/g' > ~/labs/lab8-L03-JackMcCarron/ABC.domains.pfam.evol.tsv

Go to the evolview website using the following link

      https://www.evolgenius.info/evolview/
      
Load your tree into Evolview

      Click on the file folder at the top left of the screen.  
      Give the tree a name
      Click upload, upload this file "ABC.blastp.detail.filtered.aligned_.fas.midpoint.treefile"

Upload the annotation file

      

      yourgenefamily.domains.pfam.evol.tsv









