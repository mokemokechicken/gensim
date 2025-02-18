.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_auto_examples_core_run_corpora_and_vector_spaces.py>`     to download the full example code
    .. rst-class:: sphx-glr-example-title

    .. _sphx_glr_auto_examples_core_run_corpora_and_vector_spaces.py:


Corpora and Vector Spaces
=========================

Demonstrates transforming text into a vector space representation.

Also introduces corpus streaming and persistence to disk in various formats.


.. code-block:: default


    import logging
    logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)








First, let’s create a small corpus of nine short documents [1]_:

.. _second example:

From Strings to Vectors
------------------------

This time, let's start from documents represented as strings:



.. code-block:: default

    documents = [
        "Human machine interface for lab abc computer applications",
        "A survey of user opinion of computer system response time",
        "The EPS user interface management system",
        "System and human system engineering testing of EPS",
        "Relation of user perceived response time to error measurement",
        "The generation of random binary unordered trees",
        "The intersection graph of paths in trees",
        "Graph minors IV Widths of trees and well quasi ordering",
        "Graph minors A survey",
    ]








This is a tiny corpus of nine documents, each consisting of only a single sentence.

First, let's tokenize the documents, remove common words (using a toy stoplist)
as well as words that only appear once in the corpus:


.. code-block:: default


    from pprint import pprint  # pretty-printer
    from collections import defaultdict

    # remove common words and tokenize
    stoplist = set('for a of the and to in'.split())
    texts = [
        [word for word in document.lower().split() if word not in stoplist]
        for document in documents
    ]

    # remove words that appear only once
    frequency = defaultdict(int)
    for text in texts:
        for token in text:
            frequency[token] += 1

    texts = [
        [token for token in text if frequency[token] > 1]
        for text in texts
    ]

    pprint(texts)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    [['human', 'interface', 'computer'],
     ['survey', 'user', 'computer', 'system', 'response', 'time'],
     ['eps', 'user', 'interface', 'system'],
     ['system', 'human', 'system', 'eps'],
     ['user', 'response', 'time'],
     ['trees'],
     ['graph', 'trees'],
     ['graph', 'minors', 'trees'],
     ['graph', 'minors', 'survey']]




Your way of processing the documents will likely vary; here, I only split on whitespace
to tokenize, followed by lowercasing each word. In fact, I use this particular
(simplistic and inefficient) setup to mimic the experiment done in Deerwester et al.'s
original LSA article [1]_.

The ways to process documents are so varied and application- and language-dependent that I
decided to *not* constrain them by any interface. Instead, a document is represented
by the features extracted from it, not by its "surface" string form: how you get to
the features is up to you. Below I describe one common, general-purpose approach (called
:dfn:`bag-of-words`), but keep in mind that different application domains call for
different features, and, as always, it's `garbage in, garbage out <http://en.wikipedia.org/wiki/Garbage_In,_Garbage_Out>`_...

To convert documents to vectors, we'll use a document representation called
`bag-of-words <http://en.wikipedia.org/wiki/Bag_of_words>`_. In this representation,
each document is represented by one vector where each vector element represents
a question-answer pair, in the style of:

- Question: How many times does the word `system` appear in the document?
- Answer: Once.

It is advantageous to represent the questions only by their (integer) ids. The mapping
between the questions and ids is called a dictionary:


.. code-block:: default


    from gensim import corpora
    dictionary = corpora.Dictionary(texts)
    dictionary.save('/tmp/deerwester.dict')  # store the dictionary, for future reference
    print(dictionary)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-30 12:28:00,819 : INFO : adding document #0 to Dictionary(0 unique tokens: [])
    2020-09-30 12:28:00,820 : INFO : built Dictionary(12 unique tokens: ['computer', 'human', 'interface', 'response', 'survey']...) from 9 documents (total 29 corpus positions)
    2020-09-30 12:28:00,821 : INFO : saving Dictionary object under /tmp/deerwester.dict, separately None
    2020-09-30 12:28:00,822 : INFO : saved /tmp/deerwester.dict
    Dictionary(12 unique tokens: ['computer', 'human', 'interface', 'response', 'survey']...)




Here we assigned a unique integer id to all words appearing in the corpus with the
:class:`gensim.corpora.dictionary.Dictionary` class. This sweeps across the texts, collecting word counts
and relevant statistics. In the end, we see there are twelve distinct words in the
processed corpus, which means each document will be represented by twelve numbers (ie., by a 12-D vector).
To see the mapping between words and their ids:


.. code-block:: default


    print(dictionary.token2id)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    {'computer': 0, 'human': 1, 'interface': 2, 'response': 3, 'survey': 4, 'system': 5, 'time': 6, 'user': 7, 'eps': 8, 'trees': 9, 'graph': 10, 'minors': 11}




To actually convert tokenized documents to vectors:


.. code-block:: default


    new_doc = "Human computer interaction"
    new_vec = dictionary.doc2bow(new_doc.lower().split())
    print(new_vec)  # the word "interaction" does not appear in the dictionary and is ignored





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    [(0, 1), (1, 1)]




The function :func:`doc2bow` simply counts the number of occurrences of
each distinct word, converts the word to its integer word id
and returns the result as a sparse vector. The sparse vector ``[(0, 1), (1, 1)]``
therefore reads: in the document `"Human computer interaction"`, the words `computer`
(id 0) and `human` (id 1) appear once; the other ten dictionary words appear (implicitly) zero times.


.. code-block:: default


    corpus = [dictionary.doc2bow(text) for text in texts]
    corpora.MmCorpus.serialize('/tmp/deerwester.mm', corpus)  # store to disk, for later use
    print(corpus)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-30 12:28:01,181 : INFO : storing corpus in Matrix Market format to /tmp/deerwester.mm
    2020-09-30 12:28:01,182 : INFO : saving sparse matrix to /tmp/deerwester.mm
    2020-09-30 12:28:01,182 : INFO : PROGRESS: saving document #0
    2020-09-30 12:28:01,182 : INFO : saved 9x12 matrix, density=25.926% (28/108)
    2020-09-30 12:28:01,183 : INFO : saving MmCorpus index to /tmp/deerwester.mm.index
    [[(0, 1), (1, 1), (2, 1)], [(0, 1), (3, 1), (4, 1), (5, 1), (6, 1), (7, 1)], [(2, 1), (5, 1), (7, 1), (8, 1)], [(1, 1), (5, 2), (8, 1)], [(3, 1), (6, 1), (7, 1)], [(9, 1)], [(9, 1), (10, 1)], [(9, 1), (10, 1), (11, 1)], [(4, 1), (10, 1), (11, 1)]]




By now it should be clear that the vector feature with ``id=10`` stands for the question "How many
times does the word `graph` appear in the document?" and that the answer is "zero" for
the first six documents and "one" for the remaining three.

.. _corpus_streaming_tutorial:

Corpus Streaming -- One Document at a Time
-------------------------------------------

Note that `corpus` above resides fully in memory, as a plain Python list.
In this simple example, it doesn't matter much, but just to make things clear,
let's assume there are millions of documents in the corpus. Storing all of them in RAM won't do.
Instead, let's assume the documents are stored in a file on disk, one document per line. Gensim
only requires that a corpus must be able to return one document vector at a time:



.. code-block:: default

    from smart_open import open  # for transparently opening remote files


    class MyCorpus(object):
        def __iter__(self):
            for line in open('https://radimrehurek.com/gensim/mycorpus.txt'):
                # assume there's one document per line, tokens separated by whitespace
                yield dictionary.doc2bow(line.lower().split())








The full power of Gensim comes from the fact that a corpus doesn't have to be
a ``list``, or a ``NumPy`` array, or a ``Pandas`` dataframe, or whatever.
Gensim *accepts any object that, when iterated over, successively yields
documents*.


.. code-block:: default


    # This flexibility allows you to create your own corpus classes that stream the
    # documents directly from disk, network, database, dataframes... The models
    # in Gensim are implemented such that they don't require all vectors to reside
    # in RAM at once. You can even create the documents on the fly!








Download the sample `mycorpus.txt file here <./mycorpus.txt>`_. The assumption that
each document occupies one line in a single file is not important; you can mold
the `__iter__` function to fit your input format, whatever it is.
Walking directories, parsing XML, accessing the network...
Just parse your input to retrieve a clean list of tokens in each document,
then convert the tokens via a dictionary to their ids and yield the resulting sparse vector inside `__iter__`.


.. code-block:: default


    corpus_memory_friendly = MyCorpus()  # doesn't load the corpus into memory!
    print(corpus_memory_friendly)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    <__main__.MyCorpus object at 0x125b5a128>




Corpus is now an object. We didn't define any way to print it, so `print` just outputs address
of the object in memory. Not very useful. To see the constituent vectors, let's
iterate over the corpus and print each document vector (one at a time):


.. code-block:: default


    for vector in corpus_memory_friendly:  # load one vector into memory at a time
        print(vector)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    [(0, 1), (1, 1), (2, 1)]
    [(0, 1), (3, 1), (4, 1), (5, 1), (6, 1), (7, 1)]
    [(2, 1), (5, 1), (7, 1), (8, 1)]
    [(1, 1), (5, 2), (8, 1)]
    [(3, 1), (6, 1), (7, 1)]
    [(9, 1)]
    [(9, 1), (10, 1)]
    [(9, 1), (10, 1), (11, 1)]
    [(4, 1), (10, 1), (11, 1)]




Although the output is the same as for the plain Python list, the corpus is now much
more memory friendly, because at most one vector resides in RAM at a time. Your
corpus can now be as large as you want.

Similarly, to construct the dictionary without loading all texts into memory:


.. code-block:: default


    from six import iteritems
    # collect statistics about all tokens
    dictionary = corpora.Dictionary(line.lower().split() for line in open('https://radimrehurek.com/gensim/mycorpus.txt'))
    # remove stop words and words that appear only once
    stop_ids = [
        dictionary.token2id[stopword]
        for stopword in stoplist
        if stopword in dictionary.token2id
    ]
    once_ids = [tokenid for tokenid, docfreq in iteritems(dictionary.dfs) if docfreq == 1]
    dictionary.filter_tokens(stop_ids + once_ids)  # remove stop words and words that appear only once
    dictionary.compactify()  # remove gaps in id sequence after words that were removed
    print(dictionary)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-30 12:28:02,652 : INFO : adding document #0 to Dictionary(0 unique tokens: [])
    2020-09-30 12:28:02,653 : INFO : built Dictionary(42 unique tokens: ['abc', 'applications', 'computer', 'for', 'human']...) from 9 documents (total 69 corpus positions)
    Dictionary(12 unique tokens: ['computer', 'human', 'interface', 'response', 'survey']...)




And that is all there is to it! At least as far as bag-of-words representation is concerned.
Of course, what we do with such a corpus is another question; it is not at all clear
how counting the frequency of distinct words could be useful. As it turns out, it isn't, and
we will need to apply a transformation on this simple representation first, before
we can use it to compute any meaningful document vs. document similarities.
Transformations are covered in the next tutorial
(:ref:`sphx_glr_auto_examples_core_run_topics_and_transformations.py`),
but before that, let's briefly turn our attention to *corpus persistency*.

.. _corpus-formats:

Corpus Formats
---------------

There exist several file formats for serializing a Vector Space corpus (~sequence of vectors) to disk.
`Gensim` implements them via the *streaming corpus interface* mentioned earlier:
documents are read from (resp. stored to) disk in a lazy fashion, one document at
a time, without the whole corpus being read into main memory at once.

One of the more notable file formats is the `Market Matrix format <http://math.nist.gov/MatrixMarket/formats.html>`_.
To save a corpus in the Matrix Market format:

create a toy corpus of 2 documents, as a plain Python list


.. code-block:: default

    corpus = [[(1, 0.5)], []]  # make one document empty, for the heck of it

    corpora.MmCorpus.serialize('/tmp/corpus.mm', corpus)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-30 12:28:02,781 : INFO : storing corpus in Matrix Market format to /tmp/corpus.mm
    2020-09-30 12:28:02,782 : INFO : saving sparse matrix to /tmp/corpus.mm
    2020-09-30 12:28:02,783 : INFO : PROGRESS: saving document #0
    2020-09-30 12:28:02,783 : INFO : saved 2x2 matrix, density=25.000% (1/4)
    2020-09-30 12:28:02,783 : INFO : saving MmCorpus index to /tmp/corpus.mm.index




Other formats include `Joachim's SVMlight format <http://svmlight.joachims.org/>`_,
`Blei's LDA-C format <http://www.cs.princeton.edu/~blei/lda-c/>`_ and
`GibbsLDA++ format <http://gibbslda.sourceforge.net/>`_.


.. code-block:: default


    corpora.SvmLightCorpus.serialize('/tmp/corpus.svmlight', corpus)
    corpora.BleiCorpus.serialize('/tmp/corpus.lda-c', corpus)
    corpora.LowCorpus.serialize('/tmp/corpus.low', corpus)






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-30 12:28:02,842 : INFO : converting corpus to SVMlight format: /tmp/corpus.svmlight
    2020-09-30 12:28:02,844 : INFO : saving SvmLightCorpus index to /tmp/corpus.svmlight.index
    2020-09-30 12:28:02,844 : INFO : no word id mapping provided; initializing from corpus
    2020-09-30 12:28:02,844 : INFO : storing corpus in Blei's LDA-C format into /tmp/corpus.lda-c
    2020-09-30 12:28:02,844 : INFO : saving vocabulary of 2 words to /tmp/corpus.lda-c.vocab
    2020-09-30 12:28:02,845 : INFO : saving BleiCorpus index to /tmp/corpus.lda-c.index
    2020-09-30 12:28:02,904 : INFO : no word id mapping provided; initializing from corpus
    2020-09-30 12:28:02,905 : INFO : storing corpus in List-Of-Words format into /tmp/corpus.low
    2020-09-30 12:28:02,906 : WARNING : List-of-words format can only save vectors with integer elements; 1 float entries were truncated to integer value
    2020-09-30 12:28:02,906 : INFO : saving LowCorpus index to /tmp/corpus.low.index




Conversely, to load a corpus iterator from a Matrix Market file:


.. code-block:: default


    corpus = corpora.MmCorpus('/tmp/corpus.mm')





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-30 12:28:02,968 : INFO : loaded corpus index from /tmp/corpus.mm.index
    2020-09-30 12:28:02,969 : INFO : initializing cython corpus reader from /tmp/corpus.mm
    2020-09-30 12:28:02,970 : INFO : accepted corpus with 2 documents, 2 features, 1 non-zero entries




Corpus objects are streams, so typically you won't be able to print them directly:


.. code-block:: default


    print(corpus)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    MmCorpus(2 documents, 2 features, 1 non-zero entries)




Instead, to view the contents of a corpus:


.. code-block:: default


    # one way of printing a corpus: load it entirely into memory
    print(list(corpus))  # calling list() will convert any sequence to a plain Python list





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    [[(1, 0.5)], []]




or


.. code-block:: default


    # another way of doing it: print one document at a time, making use of the streaming interface
    for doc in corpus:
        print(doc)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    [(1, 0.5)]
    []




The second way is obviously more memory-friendly, but for testing and development
purposes, nothing beats the simplicity of calling ``list(corpus)``.

To save the same Matrix Market document stream in Blei's LDA-C format,


.. code-block:: default


    corpora.BleiCorpus.serialize('/tmp/corpus.lda-c', corpus)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-30 12:28:03,395 : INFO : no word id mapping provided; initializing from corpus
    2020-09-30 12:28:03,397 : INFO : storing corpus in Blei's LDA-C format into /tmp/corpus.lda-c
    2020-09-30 12:28:03,397 : INFO : saving vocabulary of 2 words to /tmp/corpus.lda-c.vocab
    2020-09-30 12:28:03,398 : INFO : saving BleiCorpus index to /tmp/corpus.lda-c.index




In this way, `gensim` can also be used as a memory-efficient **I/O format conversion tool**:
just load a document stream using one format and immediately save it in another format.
Adding new formats is dead easy, check out the `code for the SVMlight corpus
<https://github.com/piskvorky/gensim/blob/develop/gensim/corpora/svmlightcorpus.py>`_ for an example.

Compatibility with NumPy and SciPy
----------------------------------

Gensim also contains `efficient utility functions <http://radimrehurek.com/gensim/matutils.html>`_
to help converting from/to numpy matrices


.. code-block:: default


    import gensim
    import numpy as np
    numpy_matrix = np.random.randint(10, size=[5, 2])  # random matrix as an example
    corpus = gensim.matutils.Dense2Corpus(numpy_matrix)
    # numpy_matrix = gensim.matutils.corpus2dense(corpus, num_terms=number_of_corpus_features)








and from/to `scipy.sparse` matrices


.. code-block:: default


    import scipy.sparse
    scipy_sparse_matrix = scipy.sparse.random(5, 2)  # random sparse matrix as example
    corpus = gensim.matutils.Sparse2Corpus(scipy_sparse_matrix)
    scipy_csc_matrix = gensim.matutils.corpus2csc(corpus)








What Next
---------

Read about :ref:`sphx_glr_auto_examples_core_run_topics_and_transformations.py`.

References
----------

For a complete reference (Want to prune the dictionary to a smaller size?
Optimize converting between corpora and NumPy/SciPy arrays?), see the :ref:`apiref`.

.. [1] This is the same corpus as used in
       `Deerwester et al. (1990): Indexing by Latent Semantic Analysis <http://www.cs.bham.ac.uk/~pxt/IDA/lsa_ind.pdf>`_, Table 2.


.. code-block:: default


    import matplotlib.pyplot as plt
    import matplotlib.image as mpimg
    img = mpimg.imread('run_corpora_and_vector_spaces.png')
    imgplot = plt.imshow(img)
    _ = plt.axis('off')



.. image:: /auto_examples/core/images/sphx_glr_run_corpora_and_vector_spaces_001.png
    :alt: run corpora and vector spaces
    :class: sphx-glr-single-img






.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  3.219 seconds)

**Estimated memory usage:**  10 MB


.. _sphx_glr_download_auto_examples_core_run_corpora_and_vector_spaces.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: run_corpora_and_vector_spaces.py <run_corpora_and_vector_spaces.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: run_corpora_and_vector_spaces.ipynb <run_corpora_and_vector_spaces.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
