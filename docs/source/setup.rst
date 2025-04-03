.. include:: substitutions.rst
.. _setup:

Running |tool|
==================================

Requirements
------------------------------

To run |tool| from scratch locally or on a remote server, you’ll need the following:

1. The |tool| source code, available from |git|.
2. A running instance of Neo4j.
3. A GFA graph file.
4. Gene annotation file (optional).

.. toctree::
   :maxdepth: 1

   setup/neo4j

|tool| subcommands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

pangyplot setup
######################

  Initial configuration of |tool|.

pangyplot status
######################

  Check the databases currently available.

pangyplot run
######################

  Launch the web application.

  **Options:**

  - ``--db``: Database name (default: ``default``)  
  - ``--port``: Port to serve the app (default: ``5700``)

pangyplot add
######################

  Add a dataset to the database.

  **Required:**

  - ``--ref``: Reference name  
  - ``--gfa``: Path to rGFA file  
  - ``--layout``: Path to layout TSV file  
  - ``--positions``: Position TSV file

pangyplot annotate
######################

  Add gene annotation to a reference.

  **Required:**

  - ``--ref``: Reference name  
  - ``--gff3``: Path to GFF3 annotation file

pangyplot drop
######################

  Drop tables from the database.

  **Options:**

  - ``--db``: Drop from a specific database  
  - ``--annotations``: Drop annotation tables  
  - ``--all``: Drop all stored data


Setting up with HPRC data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To test |tool| with Human Pangenome Reference Consortium (HPRC) data, follow the steps below.

- **Resource**: `https://github.com/human-pangenomics/hpp_pangenome_resources <https://github.com/human-pangenomics/hpp_pangenome_resources>`_
- **Download directory**: `hprc-v1.1-mc-grch38.chroms <https://s3-us-west-2.amazonaws.com/human-pangenomics/index.html?prefix=pangenomes/freeze/freeze1/minigraph-cactus/hprc-v1.1-mc-grch38/hprc-v1.1-mc-grch38.chroms>`_

**Example: Setup for chromosome 7**

.. code-block:: bash

   THREADS=16

   OUT=./data
   mkdir -p $OUT
   PREFIX=./${OUT}/chr7.d9

   wget https://s3-us-west-2.amazonaws.com/human-pangenomics/pangenomes/freeze/freeze1/minigraph-cactus/hprc-v1.1-mc-grch38/hprc-v1.1-mc-grch38.chroms/chr7.d9.vg 
   mv chr7.d9.vg ${PREFIX}.vg

   vg convert ${PREFIX}.vg -W -f > ${PREFIX}.gfa

   odgi build -t $THREADS -g ${PREFIX}.gfa -O -o ${PREFIX}.unsorted.og
   odgi sort -t $THREADS -Y -i ${PREFIX}.unsorted.og -o ${PREFIX}.sorted.og
   odgi normalize -t $THREADS -i ${PREFIX}.sorted.og -o ${PREFIX}.og

   # --------------- LAYOUT FILE ----------------------------
   odgi layout -t $THREADS -i ${PREFIX}.og --tsv ${PREFIX}.lay.tsv -o ${PREFIX}.lay

   # --------------- GFA FILE ----------------------------
   odgi view -t $THREADS -i ${PREFIX}.og -g > ${PREFIX}.gfa

   cat ${PREFIX}.gfa | grep ^P | cut -f 2 | grep CHM13 > ${OUT}/reference_paths.txt
   cat ${PREFIX}.gfa | grep ^S | cut -f2 > ${OUT}/segment_starts.txt
   cat ${PREFIX}.gfa | grep ^S | awk '{print $2 "," length($3)-1}' > ${OUT}/segment_ends.txt

   odgi position -t $THREADS -i ${PREFIX}.og --ref-paths ${OUT}/reference_paths.txt --graph-pos-file ${OUT}/segment_starts.txt > ${OUT}/start_positions.txt
   odgi position -t $THREADS -i ${PREFIX}.og --ref-paths ${OUT}/reference_paths.txt --graph-pos-file ${OUT}/segment_ends.txt > ${OUT}/end_positions.txt

   awk -F"[,\t]" '{print $1 "\t" $4 ":" $5+1}' ${OUT}/start_positions.txt | grep -v ^"#" | sort -k1,1 > tmp1.txt
   awk -F"[,\t]" '{print $1 "\t" $4 ":" $5+1}' ${OUT}/end_positions.txt | grep -v ^"#" | sort -k1,1 > tmp2.txt

   # --------------- POSITION FILE ----------------------------
   join -t $'\t' tmp1.txt tmp2.txt > ${OUT}/node_positions.txt
   rm tmp1.txt ; rm tmp2.txt
