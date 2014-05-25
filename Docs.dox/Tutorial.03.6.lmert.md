
\section lmert Lattice MERT

This section describes how to: 

- generate lattices for use with LMERT [\ref Macherey2008]
- run the HiFST implementations of LMERT for iterative parameter estimation  [\ref Waite2012]

\subsection lmert_run HiFST_lmert

This HiFST release includes an implementation of LMERT [\ref Waite2012].  The
script `HiFST_lmert` runs several iterations of lattice generation and
parameter estimation using the `lmert` tool.

Prior to running the script, make sure to download and uncompress the
LM `interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.withoptions.mmap`
into the `M/` directory (see \ref build).

    > scripts/HiFST_lmert

This script runs 4 iterations of LMERT, with each iteration consisting of the four steps that follow in the sections below.   
In our experience, on a X86_64 Linux computer with 4 2.8GHz CPUs and 24GB RAM, each iteration takes ca. 3 hours.  

Output from each iteration is written to files `log/log.lmert.[1,2,3,4]`.  For example, 

    > tail -8 log/log.lmert.bak/log.lmert.1 
    Thu May 15 14:33:52 2014 optimization result:
    Thu May 15 14:33:52 2014   start: 0.310724 0.996479
    Thu May 15 14:33:52 2014   final: 0.316637
    Thu May 15 14:33:52 2014 writing final parameters to file: output/exp.lmert/params.1
    Thu May 15 14:33:52 2014   1.000000,1.080643,0.516691,1.694245,0.096786,-0.436836,24.789537,-4.856992,-3.419495,0.488006,0.121445,-0.064946
    ==Params
    Thu May 15 14:33:52 BST 2014
    1.000000,1.080643,0.516691,1.694245,0.096786,-0.436836,24.789537,-4.856992,-3.419495,0.488006,0.121445,-0.064946

    > tail -8 log/log.lmert.bak/log.lmert.2 
    Thu May 15 19:08:00 2014 optimization result:
    Thu May 15 19:08:00 2014   start: 0.316392 0.968629
    Thu May 15 19:08:00 2014   final: 0.322178
    Thu May 15 19:08:00 2014 writing final parameters to file: output/exp.lmert/params.2
    Thu May 15 19:08:00 2014   1.000000,1.471600,0.534356,2.443535,-1.373008,0.631498,27.626422,-5.952960,-2.638954,0.699155,0.182260,-0.018291
    ==Params
    Thu May 15 19:08:00 BST 2014
    1.000000,1.471600,0.534356,2.443535,-1.373008,0.631498,27.626422,-5.952960,-2.638954,0.699155,0.182260,-0.018291

This indicates:
- The initial set of parameters yields a tuning set BLEU score of 0.310724, with brevity penalty 0.996479
- The first iteration of LMERT improves the tuning set BLEU score to 0.316637 over the tuning set lattices generated with the initial parameters
- Retranslation with the parameters found at iteration 1 yields a tuning set BLEU score of 0.316392, with brevity penalty 0.968629
- The second iteration of LMERT improves the tuning set BLEU score to 0.322178 over the tuning set lattices generated with the parameters from iteration 1

Note that the n-gram language model is set by default to
`M/interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.withoptions.mmap`.
This is a quantized version of
`M/interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.union.mmap`.
Slightly higher tuning set BLEU scores can be gotten with the unquantized
LM, although memory use in tuning will be higher.

Note also that the script `HiFST_lmert` can be modified to perform
LMERT over only the first (e.g.) 100 tuning set sentences.  This can
be done for debugging / demonstration, in that processing will be much
faster, although the estimated parameters will not be as robust.

   

\subsection lmert_hyps 1. Hypotheses for LMERT

Note that this step is also done in MERT (\ref mert), although the settings here are slightly different.

The following command is run at iteration `$it` ,  and will generate lattices for `$M` files.

- Input: 
  - `$it` -- lmert iteration (1, 2, ...)
  - `$M` -- number of sentences to process 
  - `RU/RU.tune.idx` -- tuning set source language sentences
  - `G/rules.shallow.vecfea.all.gz` -- translation grammar 
  - `M/interp.4g.arpa.newstest2012.tune.corenlp.ru.idx.union.mmap` -- target language model
  - language model and translation grammar feature weights, provided via command line option
- Output 
  - `output/exp.lmert/$it/LATS/?.fst.gz` -- word lattices (WFSAs), determinized and minimized
  - `output/exp.lmert/$it/hyps` -- translation hyps (discarded) 

The feature weights are gathered into a single vector, and set via the `--featureweights` command line option

- The first parameter is the grammar scale factor (in the MERT demo, this is set via `lm.featureweights`)
- The remaining parameters are weights for the grammar features (in the MERT demo, these are set via  `grammar.featureweights`)

The initial parameters are

    FW=1.0,0.697263,0.396540,2.270819,-0.145200,0.038503,29.518480,-3.411896,-3.732196,0.217455,0.041551,0.060136

HiFST is run in translation mode as

    > hifst.O2 --config=configs/CF.lmert.hyps --range=1:$M --featureweights=$FW --target.store=output/exp.lmert/$it/hyps --hifst.lattice.store=output/exp.lmert/$it/LATS/?.fst.gz      


\subsection lmert_veclats 2. Guided Translation / Forced Alignment

- Input:
  - `$it` -- lmert iteration (1, 2, ...)
  - `$M` -- number of sentences to process 
  - `output/exp.lmert/$it/LATS/?.fst.gz` -- word lattices (WFSAs), determinized and minimized (from \ref lmert_hyps)
- Output:
  - `output/exp.lmert/$it/ALILATS/?.fst.gz` -- transducers mapping derivations to translations (e.g. Fig. 7, [\ref deGispert2010])
  - `output/exp.lmert/$it/hyps` -- translation hyps (discarded)

HiFST is run in alignment mode.  Lattices (from `output/exp.lmert/$it/LATS/?.fst.gz`)
read and transformed into substring acceptors used to constrain the
space of alignments

    > hifst.O2 --config=configs/CF.lmert.alilats --range=1:$M --referencefilter.load=output/exp.lmert/$it/LATS/?.fst.gz --target.store=output/exp.lmert/$it/hyps --hifst.lattice.store=output/exp.lmert/$it/ALILATS/?.fst.gz 

Note the following parameters in the configuration file:

    [referencefilter]
    prunereferenceweight=4 
    # pruning threshold to be applied to input lattice prior to alignment
    prunereferenceshortestpath=10000
    # extract n-best list from input lattice to use as hypotheses

Reference lattices are read from `output/exp.lmert/$it/LATS/?.fst.gz`. For each lattice:
- an N-Best list of depth 10000 is extracted using `fstshortestpath` 
- the lattices are pruned with threshold `prunereferenceweight=4`
- the pruned lattice is unioned with the N-Best list
- the resulting WFSA is transformed (after removing weights, minimization and determinization) into a substring acceptor to be used in alignment

A simpler approach could be simply to prune the reference lattices.
However in practice it can be difficult to find a global pruning threshold that always
yields a reference lattice that is big enough, but not too big.
Including the n-best list ensures that there will always be a rich set of candidate hypotheses.

\subsection lmert_alilats 3. WFSAs with Unweighted Feature Vectors

- Input:
  - `$it` -- lmert iteration (1, 2, ...)
  - `$M` -- number of sentences to process
  - language model and translation grammar feature weights, provided via command line options
  - `output/exp.lmert/$it/ALILATS/?.fst.gz` -- transducers mapping derivations to translations 
- Output: 
  - output/exp.lmert/$it/VECFEA/?.fst.gz -- translation lattices (WFSAs) with unweighted feature vectors

`alilats2splats` transforms ALILATS alignment lattices to sparse
vector weight lattices; see Section 2.3.1, [\ref deGispert2010] for a
detailed explanation.  A single output WFSA with sparse vector weights
is written for each translation.  Note that this is different from the
MERT case, where two N-best lists of hypotheses
and features are written for each translation.

The HiFST `alilats2splats` command is

    > alilats2splats.O2 --config=configs/CF.lmert.vecfea --range=1:$M --featureweights=$FW --sparseweightvectorlattice.loadalilats=output/exp.lmert/$it/ALILATS/?.fst.gz --sparseweightvectorlattice.store=output/exp.lmert/$it/VECFEA/?.fst.gz 

\subsection lmert_lmert 4. LMERT

- Input:
  - `$it` -- lmert iteration (1, 2, ...)
  - `$M` -- number of sentences to process
  - `output/exp.lmert/$it/VECFEA/?.fst.gz` -- translation lattices (WFSAs) with unweighted feature vectors (from \ref lmert_alilats)
  - `EN/EN.tune.idx` -- target language references in integer format
- Output:
  - `output/exp.lmert/params.$it` -- reestimated feature vector under LMERT with BLEU

`latmert` runs as follows

    > latmert.O2 --search=random --random_axes --random_directions=28 --direction=axes --threads=24 --cache_lattices --error_function=bleu --algorithm=lmert --idxlimits=1:$M --print_precision=6 --lats=output/exp.lmert/$it/VECFEA/%idx%.fst.gz --lambda=$FW --write_parameters=output/exp.lmert/params.$it  EN/EN.tune.idx 

\subsection lmert_veclats_tst Notes on Tropical Sparse Tuple Vector Weights

The output lattices in `output/exp.lmert/$it/lats/VECFEA` are `tropicalsparsetuple` vector weight lattices.   

    > zcat output/exp.mert/1/lats/VECFEA/1.fst.gz | fstinfo | head -n 2
    fst type                                          vector
    arc type                                          tropicalsparsetuple
        
The scores in these lattices are unweighted by the feature vector
weights, i.e. they are the raw feature scores against which L/MERT finds
the optimal parameter vector values.  Distances under these unweighted
vectors do not agree with the initial translation hypotheses, e.g. the
shortest-path does not agree with the best translation:

    > unset TUPLEARC_WEIGHT_VECTOR
    > zcat output/exp.mert/1/lats/VECFEA/1.fst.gz | fstshortestpath | fsttopsort | fstpush --to_final --push_weights | fstprint -isymbols=wmaps/wmt13.en.all.wmap 
    Warning: cannot find parameter vector. Defaulting to flat parameters
    Warning: cannot find parameter vector. Defaulting to flat parameters
    0       1       <s>     1
    1       2       parliament      50
    2       3       not     20
    3       4       supports        1463
    4       5       amendment       245
    5       6       ,       4
    6       7       gives   1145
    7       8       freedom 425
    8       9       tymoshenko      23899
    9       10      </s>    2
    10      0,10,1,35.6919899,2,6.59277344,3,14.2285156,4,-10,5,-10,6,-6,8,-1,10,-9,11,8.2109375,12,13.7412109,

The sparse vector weight format is 

    0,N,idx_1,fea_1,...,idx_N,fea_N

where N is the number of non-zero elements in that weight vector.  

To compute semiring costs correctly, the `TUPLEARC_WEIGHT_VECTOR`
environment variable should be set to contain the correct feature
vector weight; this should be the same feature vector weight applied
in translation in steps 1 and 2:

    TUPLEARC_WEIGHT_VECTOR=[s_1 ... s_m w_1 ... w_n]

which in this particular example is 

    > export TUPLEARC_WEIGHT_VECTOR="1,0.697263,0.396540,2.270819,-0.145200,0.038503,29.518480,-3.411896,-3.732196,0.217455,0.041551,0.060136"

The shortest path found through the vector lattice is then
the same hypothesis produced under the initial parameter settings:

    > zcat output/exp.mert/1/lats/VECFEA/1.fst.gz | fstshortestpath | fsttopsort | fstpush --to_final --push_weights | fstprint -isymbols=wmaps/wmt13.en.all.wmap 
    0       1       <s>     1
    1       2       parliament      50
    2       3       supports        1463
    3       4       amendment       245
    4       5       giving  803
    5       6       freedom 425
    6       7       tymoshenko      23899
    7       8       </s>    2
    8       0,10,1,20.7773838,2,7.80957031,3,17.8671875,4,-8,5,-8,6,-5,9,-1,10,-7,11,20.0175781,12,18.2978516,
    

Note that printstrings can be used to extract n-best lists from the vector lattices,
if the TUPLEARC_WEIGHT_VECTOR is correctly set:

    > zcat output/exp.mert/1/lats/VECFEA/1.fst.gz | printstrings.O2 --semiring=tuplearc --nbest=10 --unique -w -m wmaps/wmt13.en.all.wmap --tuplearc.weights=$TUPLEARC_WEIGHT_VECTOR 2>/dev/null
    <s> parliament supports amendment giving freedom tymoshenko </s>        20.7778,7.80957,17.8672,-8,-8,-5,0,0,-1,-7,20.0176,18.2979
    <s> parliament supports amendment gives freedom tymoshenko </s>         20.7773,8.48828,20.9248,-8,-8,-4,0,-1,0,-7,14.1162,14.1016
    <s> parliament supports amendment giving freedom timoshenko </s>        20.7778,9.70703,17.7393,-8,-8,-5,0,0,-1,-7,22.166,18.2529
    <s> parliament supports correction giving freedom tymoshenko </s>       20.7768,9.15527,18.6689,-8,-8,-4,0,0,-1,-7,22.4062,20.7334
    <s> parliament supports amendment giving liberty tymoshenko </s>        20.7768,10.2051,18.3838,-8,-8,-5,0,0,-1,-7,22.6582,19.4707
    <s> parliament supports amendment gives freedom timoshenko </s>         20.7773,10.1602,21.0596,-8,-8,-4,0,-1,0,-7,16.2646,14.0566
    <s> parliament supports amendment enables freedom tymoshenko </s>       20.7768,8.48828,20.0742,-8,-8,-4,0,-1,0,-7,50.2627,17.0137
    <s> parliament supports amendment enable freedom tymoshenko </s>        20.7768,8.48828,20.6904,-8,-8,-4,0,-1,0,-7,50.2627,15.3457
    <s> parliament not supports amendment giving freedom tymoshenko </s>    29.3873,5.82324,12.8096,-9,-9,-6,0,0,-1,-8,16.1914,17.9443
    <s> parliament supports amendment providing freedom tymoshenko </s>     20.7769,8.48828,21.1689,-8,-8,-4,0,-1,0,-7,50.2627,17.3027

These should agree with n-best lists generated directly by alilats2splats

- N.B. There can be significant numerical differences between
computations under the tropical lexicographic semiring vs the tuplearc
semiring: printstrings and alilats2splats might not give exactly the same results.
In such cases, the alilats2splats result is probably the better choice.

