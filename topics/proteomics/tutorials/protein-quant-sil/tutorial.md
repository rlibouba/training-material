---
layout: tutorial_hands_on

title: "Peptide and Protein Quantification via Stable Isotope Labelling (SIL)"
zenodo_link: "https://zenodo.org/record/1051552"
level: Advanced
questions:
  - "What are MS1 features?"
  - "How to quantify based on MS1 features?"
  - "How to map MS1 features to MS2 identifications?"
  - "How to evaluate and optimize the results?"
objectives:
  - "MS1 feature quantitation and mapping of quantitations to peptide and protein IDs."
time_estimation: "1h"
requirements:
-
  type: "internal"
  topic_name: proteomics
  tutorials:
    - database-handling
    - protein-id-oms
key_points:
  - "Peptides labelled with stable isotopes result in multiple parallel MS1 ion traces."
  - "MS1 features can be used for relative protein quantitation."
  - "Quantitations have to be mapped to PSMs."
  - "PSM quantitations can be used to calculate protein quantitations."
  - "Proper quantitation and mapping needs careful evaluation and optimization."
contributions:
  authorship:
  - stortebecker
  - bgruening
  - matthias313
  funding:
  - elixir-europe
  - deNBI
  - uni-freiburg
subtopic: id-quant
tags: [DDA, SILAC]
---


To compare protein amounts in different samples from MS/MS data, two different experiment setups exist. Firstly, unmodified proteins can be measured in separate runs at one sample per MS-run. Secondly, proteins of samples to compare can be labelled with small chemical tags, mixed, and measured side-by-side in a single MS-run.
There are two types of chemical tags:
  1. Isobaric tags display the same mass on first hand, but fragment during the generation of the MS/MS spectra to yield reporter ions of different mass. The intensity of those reporter ions can be compared in MS/MS spectra. There are two types of isobaric tags commercially available: *tandem mass tags* (TMT) and *isobaric tags for relative and absolute quantitation* (iTRAQ).
  2. Isotopic tags are chemically identical, but differ in their mass due to incorporated stable isotopes. Examples of different isotopic tags for stable isotope labelling (SIL) are ICAT, SILAC, dimethylation, or heavy oxygen (<sup>18</sup>O).

This tutorial deals with protein quantitation via stable isotope labelling (SIL). For isotopic tags, quantitation can be achieved by comparing the intensity of MS1 peptide mass traces. The whole MS1 profile of a peptide, i.e. the intensities of all its isotopic peaks over time, is called a *peptide feature* (Figure 1a).
Incorporation of stable isotopes results in different peptide masses on MS1 level, which give rise to coeluting ion traces in the TIC with a mass difference typical for each different chemical tag (Figure 1b). Figure originally published in [Nilse et al, 2015](http://www.ncbi.nlm.nih.gov/pubmed/25931027).

In this tutorial, we will use tools of the [OpenMS suite](http://www.openms.org) to identify and quantify peptides and proteins.

![ms1 feature](../../images/protein-quant-sil_ms1feature.png "MS1 mass traces. A) Two peptide features of co-eluting SIL peptides. B) MS1 spectra at a given RT. C) XIC monoisotopic peak light peptide. D) XIC monoisotopic peak heavy peptide.")

## Prerequisites


If you are in the planning phase of your quantitative proteomics experiment, you may want to consider our tutorial on different [quantitation methods]({{site.baseurl}}/topics/proteomics/tutorials/labelfree-vs-labelled/tutorial.html) first.

To learn about *protein identification* in Galaxy, please consider our OpenMS-based [peptide ID tutorial]({% link topics/proteomics/tutorials/protein-id-oms/tutorial.md %}).

> <hands-on-title>Introduction</hands-on-title>
> In the hands-on section of this tutorial, we will use a quantitative comparison of HEK cell lysate as a test dataset. In this experiment, HEK cells were once labelled with light, once with heavy SILAC. Both cultures were lysed simultaneously and the cell lysates were mixed in a certain ratio.
> A detailed description of the full dataset is available in the [PRIDE archive](https://www.ebi.ac.uk/pride/).
>
> Your objective in this hands-on-tutorial is to find out the correct mixing ratio of the test sample.
>
> To speed up analysis, the input dataset was filtered to include only those data acquired in second 2000-3000 of the original LC gradient.
{: .hands_on}

> <agenda-title></agenda-title>
>
> In this tutorial, we will deal with:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Peptide and Protein Identification

In this tutorial, peptide identification will be performed using the workflow of the previous [Peptide ID Tutorial]({% link topics/proteomics/tutorials/protein-id-oms/tutorial.md %}). Alternatively one can perform the protein identification step by step in the [Peptide ID Tutorial]({% link topics/proteomics/tutorials/protein-id-oms/tutorial.md %}) using the SILAC dataset from [zenodo](https://zenodo.org/record/1051552) **but** beware to specify the labels in the `param_variable_modifications` of **XTandemAdapter** {% icon tool %}.

A common problem in mass spectrometry are misassigned mono-isotopic precursor peaks. Most search engines allow for some adaptation of the monoisotopic peak and we will use this by leaving `By default, misassignment to the first and second isotopic 13C peak are also considered` at `No`.

[//]: # TODO: Read about monoisotopic peak problem, give citation to review!

> <hands-on-title>Data upload</hands-on-title>
>
> 1. Create a new history for this SILAC Proteome exercise
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>
> 2. Import the mzml file, containing the measured mass spectra from [Zenodo](https://doi.org/10.5281/zenodo.1051552) or a data library:
>
>    ```
>    https://zenodo.org/record/1051552/files/HEK_SILAC-K6R6_ST905_part.mzml
>    ```
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
>    {% snippet faqs/galaxy/datasets_import_from_data_library.md %}
>
>    > <comment-title></comment-title>
>    > The data have been preprocessed during the conversion from the machine raw file. We used background removal on MS1 and MS2 level, and MS2 deisotoping.
>    {: .comment}
>
> 2. Import the human protein database (including cRAP contaminants and decoys) from [zenodo](https://zenodo.org/record/892005/files/Human_database_including_decoys_%28cRAP_added%29.fasta) or a data library:
>
>    ```
>    https://zenodo.org/record/892005/files/Human_database_including_decoys_%28cRAP_added%29.fasta
>    ```
>
> 3. Import the [workflow]({% link topics/proteomics/tutorials/protein-id-oms/workflows/workflow.ga %})
>
>    ```
>    {{ site.url }}/{{site.baseurl}}/topics/proteomics/tutorials/protein-id-oms/workflows/workflow.ga
>    ```
>
> 4. Modify the workflow
>
>   1. Connect the `mzML` input directly to the **XTandemAdapter** {% icon tool %} node and delete **PeakPickerHiRes**.
>   2. Change the **XTandemAdapter** {% icon tool %} parameters: Add the **param_variable_modifications** `Label:13C(6) (K)` and `Label:13C(6) (R)`
>
> 4. Run the workflow with
>   - the mzML dataset `1: mzML dataset`
>   - the human FASTA database `2: protein FASTA database`
>
>   > <tip-title>Using Galaxy Workflows</tip-title>
>   > If you want to learn more about Galaxy workflows, please consult the [Galaxy Introduction]({{site.baseurl}}/topics/introduction/tutorials/galaxy-intro-101/tutorial.html#the-workflow-editor)
>   {: .tip}
>
{: .hands_on}

> <question-title></question-title>
> 1. How many peptides and proteins were successfully identified?
>
> > <solution-title></solution-title>
> > 1.    2217 non-redundant peptides and 763 proteins were identified.
> > Numbers may slightly vary depending on the versions of the tools and the used FASTA file.
> {: .solution }
{: .question}

# MS1 Feature Detection

MS1 feature detection is a critical step in quantitative workflows. In principle, there are two different ways to define features:
1. Feature detection solely based upon MS1 data (mzML/raw files) without prior knowledge of peptide identifications (IDs).
    - Advantage: Feature results can be used to assist in peptide identification.
    - Drawback: Not all peptide identifications can be mapped to features, thus not every identified peptide can be quantified.
2. Feature detection based upon peptide IDs.
    - Advantage: Most peptide IDs trigger a feature detection.
    - Drawback: Feature results cannot be used to improve peptide ID.

The OpenMS suite provides several tools (FeatureFinders) for MS1 feature detection. For SIL we have to use **FeatureFinderMultiplex**, which does not take peptide IDs as an input.

> <hands-on-title>MS1 Feature Detection</hands-on-title>
>
> 1. Run {% tool [FeatureFinderMultiplex](toolshed.g2.bx.psu.edu/repos/galaxyp/openms_featurefindermultiplex/FeatureFinderMultiplex/2.6+galaxy0) %} with
>    - {% icon param-file %} *"LC-MS dataset in centroid or profile mode"*: mzML file
>    - In *"algorithmic parameters"*
>      - *"Labels used for labelling the samples"*: `[ ][Arg6,Lys6]`
>      - *"m/z tolerance for search of peak patterns"*: `10`
>      - *"Maximum number of missed cleavages due to incomplete digestion"*: `1`
>    - *"Optional outputs"*: `out_multiplets (Optional output file containing all detected peptide groups`
>
{: .hands_on}

> <comment-title>Multiple labels per peptide</comment-title>
> When using SILAC-KR or dimethyl-labelling and trypsin digestion, exactly one labelled amino acid per peptide is expected. The only labelled amino acids are lysine (K) and arginine (R) and trypsin cuts after each of them. However, a small percentage of missed cleavage normally occur also in those datasets. Setting *"Maximum number of missed cleavages due to incomplete digestion"* to `1` will be sufficient to deal with most missed cleavages.
>
> When using other enzymes (e.g. Lys-C) or other labels (e.g. $$^{18}O$$), several labelled amino acids per peptide are expected. You can search for those features by increasing the parameter *"Maximum number of missed cleavages due to incomplete digestion"*.
{: .comment}

# Mapping Identifications to Features

We now have feature quantifications for MS1 elution peaks, peptide identifications for the MS2 spectra and protein identifications.
The next step is to map the MS2-based peptide identifications to the quantified MS1 precursor peaks ("peptide features"). This will enable the quantification of identified peptides.
For labelled data, it is necessary to map peptide identifications to *consensus* features (i.e. a pair of one light peptide feature with one matching heavy feature in the correct m/z distance).
For `consensusXML`, IDMapper uses the consensus centroids, not the feature boundaries for mapping. Therefore, the RT tolerance has to be set higher than for mapping to `featureXML`. A good starting value is half the expected typical retention time.

Sometimes several peptide identifications are mapped to a feature. The tool [IDConflictResolver](http://ftp.mi.fu-berlin.de/pub/OpenMS/release-documentation/html/TOPP_IDConflictResolver.html) filters the mapping so that only the identification with the best score is associated to each feature.
Another refinement of the quantitative result is obtained by removing falsely mapped identifications e.g. light identification mapped to heavy feature. This step is performed by the **MultiplexResolver** {% icon tool %} that returns a first file with the correctly mapped peptides and as a second output the falsly mapped peptides.

Finally, the correctly mapped peptides will be combined into protein quantifications with the **ProteinQuantifier** {% icon tool %}.

> <hands-on-title>Quant to ID matching</hands-on-title>
>
> 1. Run {% tool [IDMapper](toolshed.g2.bx.psu.edu/repos/galaxyp/openms_idmapper/IDMapper/2.6+galaxy0) %} with
>    - {% icon param-file %} *"Protein/peptide identifications file"*: output of **FalseDiscoveryRate**
>    - {% icon param-file %} *"Feature map/consensus map file"*: output of **FeatureFinderMultiplex**
>    - *"RT tolerance (in seconds) for the matching of peptide identifications and (consensus) features"*: `20`
>    - *"m/z tolerance (in ppm or Da) for matching of peptide identifications and (consensus) features"*: `10`
>    - In *"Additional options for consensusXML input"*
>      - *"Match using RT and m/z of sub-features instead of consensus RT and m/z"*: `Yes`
>      - *"Store the map index of the sub-feature in the peptide ID"*: `Yes`
>
> 2. Run {% tool [IDConflictResolver](toolshed.g2.bx.psu.edu/repos/galaxyp/openms_idconflictresolver/IDConflictResolver/2.6+galaxy0) %} with
>    - {% icon param-file %} *"Input file"*: output of **IDMapper**
>
> 3. Run {% tool [MultiplexResolver](toolshed.g2.bx.psu.edu/repos/galaxyp/openms_multiplexresolver/MultiplexResolver/2.5+galaxy0) %} with
>    - {% icon param-file %} *"Peptide multiplets with assigned sequence information"*: output of **IDConflictResolver**
>    - *"Labels used for labelling the samples"*: `[ ][Arg6,Lys6]`
>    - *"Maximum number of missed cleavages due to incomplete digestion"*: `1`
>
> 4. Run {% tool [ProteinQuantifier](toolshed.g2.bx.psu.edu/repos/galaxyp/openms_proteinquantifier/ProteinQuantifier/2.6+galaxy0) %} with
>   - {% icon param-file %} *"Input file"*: first output of **MultiplexResolver**
>   - {% icon param-file %} *"Protein inference results"*: output of **FalseDiscoveryRate**
>   - *"Calculate protein abundance from this number of proteotypic peptides (most abundant first; '0' for all)"*: `0`
>   - *"Averaging method used to compute protein abundances from peptide abundances"*: `sum`
>   - *"Add the log2 ratios of the abundance values to the output"*: `Yes`
>   - *"Optional outputs"*: `out (Output file for protein abundances)` and `peptide_out (Output file for peptide abundances)`
>
{: .hands_on}

> <comment-title>ProteinQuantifier parameters</comment-title>
> Peptide quantitation algorithms are more precise for high abundant peptides. Therefore, it is recommended to base protein quantitations on those peptides. In ProteinQuantifier, you may restrict the calculation of protein abundances to the most abundant peptides by using the option "Calculate protein abundance from this number of proteotypic peptides".
> However, we recommend to use the averaging method `sum` instead. By using this option, protein ratios are based on the sum of all peptide abundances. Thus, highly abundant peptides thus have more influence on protein abundance calculation than low abundant peptides.
> A simple sum-of-intensities algorithm provided the best estimates of true protein ratios in a comparison of several protein quantitation algorithms ([Carrillo et al., Bioinformatics, 2009](https://www.ncbi.nlm.nih.gov/pubmed/19892804)).
{: .comment}

# Descriptive Statistics and Plotting of Protein Quantitations

**ProteinQuantifier** produces two output tables: the first one gives information about the quantified proteins, the second one gives information about the quantified peptides.
For proteins, we added a log-transformed ratio to the output, which is saved in column 8 of the protein table. The ratio is calculated as **log2 (abundance2/abundance1)**, which is sometimes called the *fold change (FC)* ratio.

To get a quick overview of the results, you can calculate basic descriptive statistics and plot the data as a histogram.
Comment lines in the beginning of a `tabular` file may sometimes cause errors, therefore we will remove them with the tool **Select last lines from a dataset (tail)**.

> <hands-on-title>Descriptive Statistics</hands-on-title>
> 1. Run {% tool [Summary Statistics](Summary_Statistics1) %} with
>   - {% icon param-file %} *"Summary statistics on"*: protein table output (first file) of **ProteinQuantifier**
>   - *"Column or expression"*: `c8`
>
> 2. Run {% tool [Select last lines from a dataset (tail)](toolshed.g2.bx.psu.edu/repos/bgruening/text_processing/tp_tail_tool/1.1.0) %} with
>   - {% icon param-file %} *"Text file"*: protein table output (first file) of **ProteinQuantifier**
>   - *"Operation"*: `Keep everything from this line on`
>   - *"Number of lines"*: `4`
>
> 3. Run {% tool [Histogram](toolshed.g2.bx.psu.edu/repos/devteam/histogram/histogram_rpy/1.0.4) %} with
>   - {% icon param-file %} *"Dataset"*: output of **Select last**
>   - *"Numerical column for x axis"*: `Column: 8`
>   - *"Number of breaks (bars)"*: `20`
>   - *"Plot title"* and *"Label for x axis"* to something meaningful.
>
>   > <comment-title>Calculating descriptive statistics for peptides</comment-title>
>   > The peptide table output of **ProteinQuantifier** does not give the log-transformed ratio for each peptide. Nonetheless, you may calculate basic statistics of the FC values by running **Summary Statistics** with *"Column or expression"* set to `log(c6/c5,2)`.
>   {: .comment}
>
{: .hands_on}

> <question-title></question-title>
> 1. How many peptides and proteins were successfully quantified?
> 2. What might have been the mixing ratio of the input dataset?
> 3. In the histogram, there is a second local maximum at about FC 0. What might that mean?
>
> > <solution-title></solution-title>
> > 1.    With the above parameters, you should have quantified 870 peptides and 421 proteins (slight variations are expected depending on tool version or fasta file)
> > 2. In the histogram, you see that the peak of the density curve is between -1.1 and -1.2. In the summary statistics, you can see that the mean protein ratio was -0.98. An FC of -1 indicates that the unlabelled proteins were twice as abundant as their heavy-labelled counterparts. Indeed, the mixing ratio of the dataset was 2 parts light labelled HEK cell lysate and 1 part heavy labelled HEK cell lysate.
> > 3. Some proteins were quantified with an FC close to 0. These may stem from incomplete SILAC labelling. Even after two weeks of cell culture in SILAC medium, some proteins with a very low turnover rate may remain unlabelled.
> {: .solution }
{: .question}

# Evaluation and Optimization of Quantitation Results

Protein quantitation is a multi-step procedure. Many parameters of different steps influence the final results. Therefore, it is recommended to optimize the tool parameters for each dataset and to carefully evaluate quantitation results. While the total number of quantified proteins is a first important parameter for optimization, it is also necessary to visualize the results and check for correct feature finding and ID mapping.

Galaxy does not provide a tool for proteomics visualization, we recommend to use the OpenMS Viewer **TOPPView**.
Basic TOPPView tutorials are available as [videos](https://www.openms.de/getting-started/command-line-and-visualisations/) and a more comprehensive tutorial as [HTML](http://ftp.mi.fu-berlin.de/pub/OpenMS/release-documentation/html/TOPP_TOPPView.html) or [PDF](http://ftp.mi.fu-berlin.de/pub/OpenMS/release-documentation/TOPP_tutorial.pdf).

For the optimization of tool parameters, it is recommended not to work with a complete LC-MS/MS run. Instead, we will use **FileFilter** to extract a small *RT-slice* of our input dataset, i.e. a fraction of the original dataset that was measured during a short period of time. Reducing the test data reduces the time needed for analysis and facilitates visual examination of the data.

Using Galaxy Workflows enables us to quickly re-run a full analysis with changed parameters. To learn about Galaxy Workflows, consult this [tutorial]({% link topics/galaxy-interface/tutorials/history-to-workflow/tutorial.md %}).

**Cave:** Be aware that only very small parts of your dataset can be checked by visual examination. To minimize biases, try to look at the same areas / features of each result file.

> <hands-on-title>Data reduction and visual evaluation with TOPPView</hands-on-title>
>
> 1. Run **FileFilter** {% icon tool %} with
>   - {% icon param-file %} *"Input file"*: `mzML` file
>   - *"Retention time range to extract"*: `2000:2200`
>
> 2. Extract a workflow out of your history or import the [premade workflow]({% link topics/proteomics/tutorials/protein-id-oms/workflows/workflow.ga %})
>
> 3. Run the whole workflow again with default settings on the reduced `mzML` file.
>
> 4. Run **FileFilter** {% icon tool %} with
>   - {% icon param-file %} *"Input file"*: **IDConflictResolver** output
>   - *"Remove features without annotations"*: `Yes`
>
> 5. Rename the **FileFilter** output to `Annotated features`
>
>    {% snippet faqs/galaxy/datasets_rename.md %}
>
> 6. Run **FileFilter** {% icon tool %} with
>   - {% icon param-file %} *"Input file"*: **IDConflictResolver** output
>   - *"Remove features without annotations"*: `Yes`
>
> 7. Rename the **FileFilter** output to `UNannotated features`
>
> 8. Download locally the following files:
>   - Spectra: `mzML` file
>   - peptide IDs: **IDScoreSwitcher** `idXML` output file
>   - features: **FeatureFinderMultiplex** `featureXML` output file
>   - consensus features: **FileFilter** `consensusXML` output files ("Annotated" and "UNannotated" features)
>
> 9. Open **TOPPView**
> 10. Open the `mzML` file in **TOPPView** with
>   - **Open as** set to `new window`
>   - **Map view** set to `2D`
>   - **Low intensity cutoff** set to `off`
> 11. Open all other downloaded files in **TOPPView** with
>   - **Open as** set to `new layer`
> 12. Activate the `mzML` layer and click on `Show projections`
>
>   ![showProjections](../../images/protein-quant-sil_showProjections.png "The projections display intensities plotted against RT (top panel) and plotted against m/z (right panel).")
>
> 13. Activate the `consensusXML` layers and click on `Show consensus feature element positions`
>
>   ![showConsensus](../../images/protein-quant-sil_showConsensus.png "Arrows between linked light and heavy features centroids are displayed.")
>
> 14. Evaluate your data analysis, by
>   - zooming into a specific region (hold `Ctrl` and use the mouse to zoom)
>   - measuring m/z and RT distances (select the `mzML` layer, hold `Shift` and use the mouse to measure)
>   - displaying an area in 3D view (`right-click` into the 2D View and select `Switch to 3D view`)
>   - switching on and off the display of single layers (`left-click` at the tick-boxes in the window "Layers")
{: .hands_on}

## Examples

1. **Displaying annotated vs. UNannotated features**: visualize annotated (= mapped) and unannotated (= unmapped) features by switching between activating only the "Annotated_features.consensusxml" or only the "UNannotated_features.consensusxml" layer

    ![displaying feature annotations](../../images/protein-quant-sil_example_featureAnnotation.png "The feature below was mapped to and annotated with a peptide identification, the feature above was not mapped. A) Annotated layer is active. B) UNannotated layer is active.")

2. **Correct mapping**: a feature was detected, a peptide was identified and the two were mapped.

    ![Correct mapping of a high-abundant peptide](../../images/protein-quant-sil_example_highAbundant.png "A high-abundant peptide. A) 2D View. B) 3D View.")
    ![Correct mapping of a low-abundant peptide](../../images/protein-quant-sil_example_lowAbundant.png "A low-abundant peptide. A) 2D View. B) 3D View.")

3. **No feature detected for a contaminant.** Contaminants are often not labelled, but occur only in their unlabelled isoform. Therefore, they do not give rise to a consensus feature in FeatureFinderMultiplex.

    ![contaminant](../../images/protein-quant-sil_example_contaminant.png "A contaminant. There are no elution peaks for the heavy labelled peptide isoform. A) 2D View. B) 3D View.")

> <hands-on-title>Check a possible contaminant</hands-on-title>
> 1. Run **TextExporter** {% icon tool %} with
>   - {% icon param-file %} *"Input file"*: **IDFilter** output file
> 2. Run **Search in textfiles (grep)** {% icon tool %} with
>   - {% icon param-file %}*"Select lines from"*: output of **TextExporter**
>   - *"Regular Expression"*: the peptide sequence (e.g. `MFLSFPTTK`)
> 3. Check if the peptide was mapped to a protein marked with "CONTAMINANT".
{: .hands_on}

## Typical Problems

Three problems typically hamper correct peptide mapping:

1. **A feature is detected, but no peptide identification is nearby.**

    - *Possible cause*: This may be caused by imperfect peptide identification. However, it is never expected that every single MS2-spectrum leads to an identification. The protein might be missing in the database, or the peptide may carry a modification that was not included in the search.
    - *Possible solution*: Improve your search engine settings.

    ![noID](../../images/protein-quant-sil_problem_noID.png "Several features without peptide IDs. Several MS2 spectra were generated (fragment scan precursors marked with red circles), but did not lead to a peptide identification.")

2. **A peptide was identified, but no feature is nearby.**

    - *Possible cause*:
        1. The elution peaks of the peptide may be distorted. This is typical for low intensity peptides. If a lot of peptides have distorted elution peaks this may be a sign of spray instability.
        2. The peptide is a contaminant.
    - *Possible solution*:
        1. Lower the FeatureFinderMultiplex parameters **Two peptides in a multiplet are expected to have the same isotopic pattern** and/or **The isotopic pattern of a peptide should resemble the averagine model at this m/z position** or broaden the **Range of isotopes per peptide in the sample** (in **Advanced Options**).
        2. No optimization of parameters is needed (see [example above](#examples))

    ![noID](../../images/protein-quant-sil_problem_noFeature.png "A peptide was identified in its unlabelled and in its labelled isoform, but no feature was detected. A) 2D View. No third isotopic peak was detected for the labelled peptide. B) 3D View.")

3. **A peptide was identified and a feature was detected nearby, but the two are not mapped to each other.**

    - *Possible cause*:
        1. The MS2 event and the feature are too far apart to be mapped.
        2. The precursor of the MS2 was not correctly assigned to the mono-isotopic peak.
        3. The detected feature is too small in RT dimension and covers only a part of the peptide peaks.
    - *Possible solution*:
        1. Increase the IDMapper parameter **RT tolerance (in seconds) for the matching of peptide identifications and (consensus) features**.
        2. Feature size in RT dimension cannot be directly corrected, use solution 1 instead.

    ![noMapping](../../images/protein-quant-sil_problem_noMapping.png "A feature was detected, but the RT dimension is shorter than the peptides's elution peak. Also, the 2nd isotopic peak was fragmented and was not corrected due to the short feature. A) 2D View. B) 3D View.")

Two problems typically disturb correct peptide quantitation:

1. **A peptide is mapped to the wrong feature.**
    - *Possible cause*: Co-eluting peptides of a similar mass may be falsely mapped to a nearby feature, if the correct peptide did not lead to an identification or was identified only with a low score. In *high-resolution data*, this problem of is very limited. Co-eluting peptides can normally be distinguished by slightly different m/z values. In *low-resolution data*, wrong assignment may occur more often.
    - *Possible solution*: If a high value is used for the precursor mass tolerance, try to keep the RT tolerance low to avoid false mapping.
2. **Background noise (1) or co-eluting peptides (2) are incorporated in a feature.**
    - *Possible solution*:
        1. Use noise-filtering either during pre-processing or by increasing the FeatureFinderMultiplex parameter **Lower bound for the intensity of isotopic peaks**
        2. Reduce the FeatureFinderMultiplex parameter **m/z tolerance for search of peak patterns**.

> <question-title></question-title>
> 2. How many peptides could not be mapped to MS1 features? (Click on the **IDMapper** output and look at the tool's infobox.)
> 3. How many features could not be mapped to a peptide identification? (Click on the **ProteinQuantifier** output and look at the tool's infobox.)
> 1. Which problems are most prominent in the test dataset?
>
> > <solution-title></solution-title>
> > 1. 1,395 peptide IDs could not be mapped to a feature.
> > 2. 1,898 features, corresponding to 949 consensus features could not be mapped to a peptide identification.
> > 3. The mapping of peptide IDs to features seems to have worked mostly fine. The main problems seem to be (1) missing peptide identifications, (2) missing features where a peptide was identified and (3) features that span a shorter RT range than the corresponding peptide's elution peak.
> {: .solution }
{: .question}

## Optimization of Quantitation Results

For optimization, it is critical to modify **only one parameter at a time**.
Also, it is recommended to optimize the tools in the order of their position in the workflow.

In the test dataset, several peptides were identified, but not quantified. Some of the peptides were even identified in the unlabelled, as well as in the labelled form. To optimize the feature detection, we will relax the parameters of FeatureFinderMultiplex.

> <hands-on-title>Optimize Feature Detection</hands-on-title>
>
> 1. Run the whole workflow again:
>    - Change the **FeatureFinderMultiplex** parameter *"Range of isotopes per peptide in the sample"* from `3:6` to `2:6`.
> 2. Integrate the **HighResPrecursorMassCorrector** {% icon tool %} into the workflow.
>    - Use the mzML file and the featureXML from the **FeatureFinderMultiplex** {% icon tool %} as input
>    - Set **Additional retention time tolerance added to feature boundaries** between `0.0` and `10.0`
>    - Connect the output with the search engine.
> 3. Compare the number of identified proteins, unmatched features and unmapped peptides for each parameter setting.
> 4. Visualize the results with TOPPView to check for correct feature detection and feature-to-peptide mapping.
>
>   > <tip-title>Sending workflow results to new history</tip-title>
>   > When running a workflow, you may send the results to a new history. This helps keeping track of different parameter settings.
>   {: .tip}
{: .hands_on}

> <question-title></question-title>
> Which parameter improved the number of quantified proteins?
>
> > <solution-title></solution-title>
> > Both changes led to more quantified proteins. Increasing the isotope range led to 26 \% more protein quantitations, increasing the RT tolerance led to 7 \% more protein quantitations.
> {: .solution }
{: .question}
