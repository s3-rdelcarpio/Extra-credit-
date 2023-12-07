# Extra-credit-
This experiment was done in multiple labs to explain the refernces to other labs. I also named my subdirectory personal.

Creating a blast search for my protein

First download the query sequence in FAFSTA format
``ncbi-acc-download -F fasta -m protein NP_001121697.2``

KEY: PROTEOMES FILE was already included 

Uncompress proteomes with 
``gunzip proteomes/*.gz``

Then put them into a single file 
``cat  proteomes/*.faa > allprotein.fas``

Make a BLAST database with 
``makeblastdb -in allprotein.fas -dbtype prot``

Create a BLAST search against a protein database with information about top hit 
``blastp -db ../allprotein.fas -query NP_001121697.2.fa -outfmt 0 -max_hsps 1 -out personal.blastp.typical.out``

Create a more detailed output including specific information about alignment like length and e-value. 
``blastp -db ../allprotein.fas -query NP_001121697.2.fa  -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle"  -max_hsps 1 -out personal.blastp.detail.out``

Filter BLAST for high scoring homologs

Filter out file to a e-value of less than 1e-35 allowing only output to include less than the specified treshold
``awk '{if ($6< 1e-35)print $1 }' personal.blastp.detail.out > personal.blastp.detail.filtered.out``

Obtain sequences in BLAST output by 
``seqkit grep --pattern-file ~/labs/lab03-$MYGIT/personal/personal.blastp.detail.filtered.out ~/labs/lab03-$MYGIT/allprotein.fas > ~/labs/lab04-$MYGIT/personal/personal.homologs.fas``

Align multiple sequences using muscle 
``muscle -in ~/labs/lab04-$MYGIT/personal/personal.homologs.fas -out ~/labs/lab04-$MYGIT/personal/personal.homologs.al.fas``

Find the max likelihood tree estimate by using IQ tree
``iqtree -s ~/labs/lab05-$MYGIT/personal/personal.homologs.al.fas -bb 1000 -nt 2 ``

Rerrot the tree in accordance with midpoint rooting 
``gotree reroot midpoint -i ~/labs/lab05-$MYGIT/personal/personal.homologs.al.fas.treefile -o ~/labs/lab05-$MYGIT/personal/personal.homologs.al.mid.treefile ``

Save this with
``nw_order -c n ~/labs/lab05-$MYGIT/personal/personal.homologs.al.mid.treefile | nw_display -w 1000 -b 'opacity:0' -s  >  ~/labs/lab05-$MYGIT/personal/personal.homologs.al.mid.treefile.svg -``

Reconciling gene and species tree
``java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s ~/labs/lab06-$MYGIT/species.tre -g ~/labs/lab06-$MYGIT/personal/personal.homologs.al.mid.treefile --reconcile --speciestag prefix --savepng --events --outputdir ~/labs/lab06-$MYGIT/personal/``

To see the internal nodes names use
``grep NOTUNG-SPECIES-TREE ~/labs/lab06-$MYGIT/globins/globins.homologs.al.mid.treefile.reconciled | sed -e "s/^\[&&NOTUNG-SPECIES-TREE//" -e "s/\]/;/" | nw_display -``

Then use thirdkind to recreate the genreconciliation with species tree reconciliation
``thirdkind -Iie -D 40 -f ~/labs/lab06-$MYGIT/globins/globins.homologs.al.mid.treefile.reconciled.xml -o  ~/labs/lab06-$MYGIT/globins/globins.homologs.al.mid.treefile.reconciled.svg``

Domain Identification 
Using raw unaligned sequence remove stop codon 
``sed 's/*//' ~/labs/lab04-$MYGIT/personal/personal.homologs.fas > ~/labs/lab08-$MYGIT/personal/personal.homologs.fas``

Download pfam database
``wget -O ~/data/Pfam_LE.tar.gz ftp://ftp.ncbi.nih.gov/pub/mmdb/cdd/little_endian/Pfam_LE.tar.gz && tar xfvz ~/data/Pfam_LE.tar.gz  -C ~/data``

run rps blast
``rpsblast -query ~/labs/lab08-$MYGIT/personal/personal.homologs.fas -db ~/data/Pfam -out ~/labs/lab08-$MYGIT/personal/personal.rps-blast.out  -outfmt "6 qseqid qlen qstart qend evalue stitle" -evalue .0000000001``

plot pfam domain predicitions from rps-blast by using 
``sudo /usr/local/bin/Rscript  --vanilla ~/labs/lab08-$MYGIT/plotTreeAndDomains.r ~/labs/lab08-$MYGIT/personal/personal.homologs.al.mid.treefile ~/labs/lab08-$MYGIT/personal/personal.rps-blast.out ~/labs/lab08-$MYGIT/personal/personal.tree.rps.pdf``

