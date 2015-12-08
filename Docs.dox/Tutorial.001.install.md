Installation {#build}
=====================

This section contains instructions on downloading, compiling, and
installing the UCAM-SMT tools.  

There are three sets of tools that can be installed:

   * \ref install_binaries --  HiFST C++ binaries and dependencies (OpenFst, KenLM, ...)
   * \ref tutorial_install -- HiFST tutorial based on the UCAM WMT'13 Russian-English system [\ref Pino2013]
   * \ref rulextract_install -- Hadoop-based translation grammar extraction

The \ref install_binaries is **required** to run the \ref
tutorial_. However the \ref tutorial_ does **not require** the
translation grammar - translation grammars, language models, etc., are
provided in the tutorial.  However, the Hadoop-based translation
grammar extraction may be useful if you wish to extract your own
translation grammars from aligned parallel text.

\section install_binaries HiFST Binary Installation

**Note**:The following instructions are for the Bash shell.

The code can be cloned from the following GitHub address:

    > git clone https://github.com/ucam-smt/ucam-smt.git

In the following, `HiFSTROOT` designates the cloned directory,
i.e. the following should be a complete path to the cloned directory

    > export HiFSTROOT=complete_path_to_hifst_cloned_directory

As a check, the following command should find the `README.md` file downloaded from github:

    > ls $HiFSTROOT/README.md

Once downloaded, the tools and libraries are compiled in the directory
`$HiFSTROOT` by running the script `build-test.sh`.  

**Note** that the
environment variable `TGTBINMK` specific compilation option
(e.g. optimisation level, static vs dynamic, etc.) can be set to a value of your choosing 
before running the script.  See `build-test.sh` for the supported options.

    > cd $HiFSTROOT
    > export TGTBINMK=O2 # change this as appropriate for your environment. If you omit this line, the default is `export TGTBINMK=O2`.
    > ./build-test.sh

This should download and install necessary dependencies,
compile the code and run tests. The `README.md` in the cloned directory also
contains useful information for the installation.

\section hifst_paths HiFST Paths and Environment Variables

After HiFST is successfully built and tested,  the file $HiFSTROOT/Makefile.inc
will contain environment variable settings needed to run the HiFST
binaries and the OpenFST tools using the HiFST libraries.  To set these,
simply run

    > source $HiFSTROOT/Makefile.inc
    > export PATH=$HiFSTROOT/bin:$OPENFST_BIN:$PATH
    > export LD_LIBRARY_PATH=$HiFSTROOT/bin:$OPENFST_LIB:$BOOST_LIB:$LD_LIBRARY_PATH

You should make sure that $HiFSTBINDIR is added first on the path and
the library path and that it preceeds the OpenFst directories.
If the LD\_LIBRARY\_PATH variable is not set correctly, you will see the message

    ERROR: GenericRegister::GetEntry : tropical_LT_tropical-arc.so: cannot open shared object file: No such file or directory
    ERROR: ReadFst : unknown arc type "tropical_LT_tropical" : standard input

Sourcing `Makefile.inc` sets the environment variable 
`TGTBINMK` to point to the HiFST binaries; for example, the following should find the main HiFST binary:

   > ls $HiFSTROOT/bin/hifst.${TGTBINMK}.bin

It is possible to use multiple builds of HiFST by changing the `TGTBINMK` variable.

\section rulextract_install Installation of the Hadoop-based Grammar Extraction Tools
**Note:** These are not needed to run the basic HiFST \ref tutorial_.

Requirements for building the rule extraction software:
  + [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html "Download Java")
  + [SBT 13.8 or greater](http://www.scala-sbt.org/download.html "Download SBT") 

The rule extraction code will be in the `$HiFSTROOT/java/ruleXtract` directory. Rule extraction is built as part of the full build. To build separatly use the folloing commands.

    > cd $HiFSTROOT/java/ruleXtract
    > sbt assembly

You will obtain a jar file named `ruleXtract.jar`
located at `$HiFSTROOT/java/ruleXtract/target/ruleXtract.jar` .
We use the variable `$RULEXTRACTJAR` to designate this
jar from now on:

    > RULEXTRACTJAR=$HiFSTROOT/java/ruleXtract/target/ruleXtract.jar

\section tutorial_install Tutorial Installation

Files for this tutorial can be downloaded from the following GitHub address:

    > git clone https://github.com/ucam-smt/demo-files.git
    > cd demo-files; gunzip wmaps/*.gz  ## Uncompress big wordmap files.

The tutorial files and sources for this tutorial will be
in the `demo-files` directory. The variable `$DEMO`
should be set to point to this director:

    > export DEMO=complete_path_to_demo-files_cloned_directory

As a check, the following command should find the `README` file downloaded from github:

    > ls $DEMO/README

**Language Models** The language models needed for this tutorial can be downloaded from
<http://mi.eng.cam.ac.uk/~wjb31/data/hifst.release.May14/> .  These are not on github, due to their size.
There are two files:
   * [interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.withoptions.mmap.gz](http://mi.eng.cam.ac.uk/~wjb31/data/hifst.release.May14/interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.withoptions.mmap.gz) -- 766M
   * [interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.union.mmap.gz](http://mi.eng.cam.ac.uk/~wjb31/data/hifst.release.May14/interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.union.mmap.gz) -- 5.8G

To run this tutorial you **must** download the smaller of the two LMs:

    > cd $DEMO/M/
    > wget http://mi.eng.cam.ac.uk/~wjb31/data/hifst.release.May14/interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.withoptions.mmap.gz
    > gunzip interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.withoptions.mmap.gz

The big LM is needed for some language model rescoring operations, but otherwise is not required for the tutorial.

    > cd $DEMO/M/
    > wget http://mi.eng.cam.ac.uk/~wjb31/data/hifst.release.May14/interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.union.mmap.gz
    > gunzip interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.union.mmap.gz

At this point, the following command should find the smaller of the two LMs:

    > ls $DEMO/M/interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.withoptions.mmap

**Word Maps** The Russian and English wordmap files should be uncompressed (see \ref wmappedfiles ):

    > gunzip -k wmaps/wmt13.ru.wmap.gz wmaps/wmt13.en.wmap.gz


**Documentation**
As an optional
step, if you wish to regenerate the HTML code for this
tutorial, make sure you have doxygen (1.8+
for markdown support) and
latex (for formulas) installed and run the following commands:

    > cd $DEMO/Docs.dox
    > doxygen


