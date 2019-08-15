Reference
---------
P. Gopalan, L. Charlin, D.M. Blei, Content-based recommendations with Poisson factorization, NIPS 2014.

[Paper PDF](https://papers.nips.cc/paper/5360-content-based-recommendations-with-poisson-factorization.pdf)


Installation
------------

Required libraries: gsl, gslblas, pthread

On Linux/Unix run

 ./configure
 make; make install

***Note: We have NOT tested the code on Mac. Please use Linux.***

On Mac OS, the location of the required gsl, gslblas and pthread
libraries may need to be specified:

 ./configure LDFLAGS="-L/opt/local/lib" CPPFLAGS="-I/opt/local/include"
 make; make install

The binary 'collabtm' will be installed in /usr/local/bin unless a
different prefix is provided to configure. (See INSTALL.)

COLLABTM: Nonnegative Collaborative Topic Modeling tool
--------------------------------------------------------

**collabtm** [OPTIONS]

    -dir <string>            path to dataset directory with files described under INPUT below
 
    -mdocs <int>	     number of documents

    -nuser <int>	     number of users

    -nvocab <int>	     size of vocabulary
    	    
    -k <int>                 latent dimensionality

    -fixeda                  fix the document length correction factor ('a') to 1
    
    -lda-init                use LDA based initialization (see below)
    
OPTIONAL:    

    -binary-data             treat observed ratings data as binary; if rating > 0 then rating is treated as 1

    -doc-only                use document data only

    -ratings-only            use ratings data only
    
    -content-only            use both data, but predict only with the topic affinities (i.e., topic offsets are 0)

EXPERIMENTAL:

    -online                  use stochastic variational inference
    
    -seq-init -doc-only	     use sequential initialization for document only fits
    
    
    
RECOMMENDED
-----------

We recommend running CTPF using the following options:

~/src/collabtm -dir <path-to>/mendeley -nusers 80278 -ndocs 261248 -nvocab 10000 -k 100 -lda-init -fixeda

If the document lengths are expected to vary significantly, we recommend additionally running without the "-fixeda" option above.

The above options depend on LDA-based fits being available for the document portion of the model. See below.

INPUT 
-----

The following files must be present in the data directory (as indicated by the
'-dir' switch): 

train.tsv, test.tsv, validation.tsv, test_users.tsv

train/valid/test files contain triplets in the following format (one per line): 
userID itemID rating

where tab characters separate the fields. 

test_users.tsv contains the userIDs of all users that are tested on (one per
line). 

The new files additionally needed are mult.dat and vocab.dat.  (They are really text files.) This is the "document" portion of the data. Each line of mult.dat is a document and has the following format:

     <number of words> <word-id0:count0> <word-id1:count1>....

Each line of vocab.dat is a word. Note that both the word index and the document index starts at 0. So a word-id in vocab.dat can be 0 and the document id "rated" in train.tsv can be 0.

EXAMPLE
-------

Run two versions -- with the correction scalar 'a' inferred and one with 'a' fixed at a 1.  One of these fits might be better than the other. The "-fixeda" option specifies that the documents are of similar lengths.

***Always use LDA-based initialization.***

~/src/collabtm -dir <path-to>/mendeley -nusers 80278 -ndocs 261248 -nvocab 10000 -k 100 -lda-init

~/src/collabtm -dir <path-to>/mendeley -nusers 80278 -ndocs 261248 -nvocab 10000 -k 100 -fixeda -lda-init


LDA BASED INITIALIZATION
------------------------

1. Run Chong's gibbs sampler to obtain LDA fits on the word frequencies
(see below for details)

2. Create a directory "lda-fits" within the "dataset directory" above and put
two files in it: the topics beta-lda-k<K>.tsv and the memberships
theta-lda-k<K>.tsv.  If K=100, these files will be named beta-lda-k100.tsv and
theta-lda-k100.tsv, respectively.

3. Run collabtm inference with the -lda-init option as follows (the -fixeda option fixes 'a' at 1):

~/src/collabtm -dir <path-to>/mendeley -nusers 80278 -ndocs 261248 -nvocab 10000 -k 100 -lda-init

~/src/collabtm -dir <path-to>/mendeley -nusers 80278 -ndocs 261248 -nvocab 10000 -k 100 -lda-init -fixeda


CHONG's GIBBS SAMPLER
---------------------

The LDA code is provided under the "lda" directory.

For example, run LDA with parameters
    - 50 topics
    - the topic Dirichlet set to 0.01
    - the topic proportion Dirichlet set to 0.1
as follows:

./lda --directory fit_50/ --train_data ~/arxiv/dat/mult_lda.dat --num_topics 50 --eta 0.01 --alpha 0.1 --max_iter -1 --max_time -1

mult_lda.dat contains the documents (see the David Blei's lda-c package for
the exact format: http://www.cs.princeton.edu/~blei/lda-c/index.html)

***Note*** The values of eta and alpha need to reflect those used when loading the LDA fits 
in CTPF (see collabtm.cc:initialize()).

The output directory ("fit_50/" in the above example) will contain the fit files which 
can be used to initialize CTPF with -lda-init option. Specifically *.topics corresponds 
to beta-lda-k<K>.tsv, and *.doc.states corresponds to theta-lda-k<K>.tsv.

---
### To run in debug mode
This fork enables proper debugging to better understand the working of this software.\
To enable debugging it doesn't suffice to simply specify `./configure --enable-debug make`. It requires changes to `Makefile.am` and `Makefile.in` and this fork has implemented those required changes.
`.vscode\` has example `launch.json` and `tasks.json` that were actually used to run this software in debug mode in Windows 10 WSL Visual Studio Code.
