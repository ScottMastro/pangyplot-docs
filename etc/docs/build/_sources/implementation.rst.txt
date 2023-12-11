.. _implementation:
.. include:: substitutions.rst

Implementation
==================================

VikNGS will output the p-values calculated to a text file in the current directory (i.e. ".") by default. Output directory can be changed from the user interface or using ``-o [DIRECTORY]`` on the command line version. 

In addition to p-values, VikNGS can also produce a file detailing which variants were filtered out prior to testing and an explanation why. In future versions of the software, we hope to provide more types of output in addition to these files.

.. figure:: _images/vcf_layout.png
   :alt: Multsample VCF Layout
   :align: center

   The general layout of a multi-sample VCF file.


Back-end
------------------------------

|tool| uses the `D3 Force Graph library 🔗 <https://github.com/vasturiano/force-graph>`_ and `BubbleGun 🔗 <https://github.com/fawaz-dabbaghieh/bubble_gun>`_

``Chromosome  Position  Reference Alternative P-value TestName  CollapseGroupID``

``CollapseGroupID`` is only present if variants were collapsed into groups (single p-value per collapsed group).


Front-end
------------------------------

