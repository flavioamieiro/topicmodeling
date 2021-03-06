#topicmodeling


Library containing tools for topic modeling and related NLP tasks.

It brings together implementations from various authors, slightly modified by me as well as a new visualization tools
to help inspect the results.

I have also added a fair ammount of tests, mainly to guide my refactoring of
the code. Tests are still sparse, but will grow as the rest of the codebase sees more usage and refactoring.

###Running the tests

After you clone this repository, you can run the tests by going into the tests directory and running nosetests (nose required).

Quick tutorial
--------------

###Online LDA


The sub-package onlineldavb is currently  the most used/tested.
Here is a quick example of its usage:
Assume you have a set of documents you want to extract the most representative topics from. 

The first thing you need is a vocabulary list for these, i.e., valid informative words you may want to use 
to describe topics. I generally use a spellchecker to find these plus a list of stopwords.
*NLTK* and *PyEnchant* can help us with that

```python
import nltk
import enchant
from string import punctuation
from enchant.checker import SpellChecker

sw = nltk.corpus.stopwords.words('english')
checker=SpellChecker('en_US')

docset = ['...','...',...] # your corpus
```
Now, for every document in your corpus you can run the following code to define its vocabulary.
```python
checker.set_text(text)
errors = [err.word for err in checker]
vocab = [word.strip(punctuation) for word in nltk.wordpunct_tokenize(text) if word.strip(punctuation) not in sw+errors]
vocab = list(set(vocab))
```
Now that you have a vocabulary, which the union of all the vocabularies of each document, you can run the 
LDA analysis. You have to specify the number of topics you expect to find (K below)
```python
K=10
D = 100 #Number of documents in the docset
olda = onlineldavb.OnlineLDA(vocab, K, D, 1./K, 1./K, 1024, 0.7)
for doc in docset:
  gamma, bound = olda.update_lambda(doc)
  wordids, wordcts = onlineldavb.parse_doc_list(doc,olda._vocab)
  perwordbound = bound * len(docset) / (D*sum(map(sum,wordcts)))
np.savetxt('lambda.dat',olda._lambda)
```

Finally you can visualize the resulting topics as a Word Cloud:
```python
cloud = GenCloud(vocab,lamb)
for i in range(K):
  cloud.gen_image(i)
```
If you have done everything right you should see 10 figures just like this:

![topic_cloud](https://raw.github.com/NAMD/topicmodeling/master/tests/topic_0.png?raw=true)


### Turbotopics

Turbo topics from Blei & Lafferty (2009) is also part of this package. As with the rest of the code it has been
refactored for better compliance to PEP 8, as well as to provide a better integration to the Topics package.

Here is a simpl usage example:

```python
from Topics.visualization.ngrams import compute
from Topics.visualization import lda_topics

compute('mydoc_utf8.txt', 0.001,False,'unigrams.txt',stopw=sw)
```

After executing the code above, two files will be generated on disk: "unigrams.txt" and "ngrams_count,csv".

Now we can load them and create nice word clouds:

```python
from collections import OrderedDict
with codecs.open('ngram_counts.csv', encoding='utf8') as f:
    ngrams = f.readlines()
ng = OrderedDict()
for l in ngrams:
    w,c = l.split('|')
    if float(c.strip()) >100:
        continue
    ng[w.strip()] = float(c.strip())

counts = np.array(ng.values())
counts.shape = 1,len(counts)
ngcloud = GenCloud(ng.keys(),counts)
ng.values()

ngcloud.gen_image(0,'ngrams')
```

if we want to include only the ngrams with more than one word, we can remove those from the dictionary *ng*, above.
