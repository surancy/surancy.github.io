---
title: "Word Co-occurrence Matrix Implementation and Visualization"
layout: page
date: 2019-06-20 14:57
image: /assets/images/blog/2019-06-20-co-occurrence-matrix-visualization/0620-gelphi2.png
category: [Tech Notes, NLP]
author: Rainy
description: Graph visualization on representing sementic text relationships
comments: true
---

## Word Co-occurance Matrix in Python and Network Visualization in Gelphi

Co-occurrence matrices analyze text in context. Word embeddings and vector semantics 
are ways to understand words in their context, namely the **semantics analysis** in NLP 
(compare to syntax analysis such as language modeling using ngram, Part-of-Speech (POS) taggings, Named Entity Recognition (NER)).

### Word embeddings and vector semantics

Thesaurus-based meaning has problems:  
thesaurus for every language  
time evolves and meaning changes  
problems with recall: missing words, works less well for v,adv  

#### Sparse vector representations

1. Mutual-information weighted word co-occurrence matrices

   - Term-document matrix
   - Term-term matrix (word-word co-occurrence matrix / word-context matrix)

   **First-order co-occurrence (syntagmatic association)**:  
       • They are typically nearby each other.  
       • *wrote* is a first-order associate of *book* or *poem*.  
   **Second-order co-occurrence (paradigmatic association)**:  
       • They have similar neighbors.  
       • *wrote* is a second- order associate of words like *said* or *remarked*.  

#### Dense vector representations

1. Singular value decomposition (and Latent Semantic Analysis)
2. Neural-network-inspired models (skip-grams, CBOW)
3. ELMo and BERT
4. Brown clusters

---

We'll focus on implementing the word-word co-occurrence matrix in Python and do the visualization of word's network in Gelphi. 

#### Fetch Data

The data we'll be using is from a news API:  
https://newsapi.org/  

```python
import requests
url = ('https://newsapi.org/v2/everything?'
       'q=bitcoin&'
       'from=2019-06-20&'
       'sortBy=popularity&'
       'language=en&'
       'apiKey=[YOUR API KEY]')

response = requests.get(url)
json = response.json()
```

There are 20 news articles fetched in today.
{% highlight raw %}
Number of Articles Fetched:  20

'Florida city gives in to $600,000 bitcoin ransomware demand',
 'Florida city will pay hackers $600,000 to get its computer systems back - The Washington Post',
 'US wants to extradite Swede over $11M Bitcoin investment scam',
 'Bitcoin healthy AF following Zuckbuck reveal, hash rate hits all-time high',
 'Messaging app Line is reportedly launching a cryptocurrency exchange in Japan soon',
 'Facebook’s logo drama is a problem and for more reasons than you think',
 'Here Are All The Deets On Facebook’s New Cryptocurrency, Libra',
 'Facebook’s logo drama is a problem and for more reasons than you think',
 'Florida city agrees to pay hackers $600,000 in bitcoin to get its computer files back',
 'This father of three put everything into bitcoin. Here’s what happened next.',
 "Libra: four reasons to be extremely cautious about Facebook's new currency",
 'What does Facebook’s Libra currency mean for Africa?',
 'With Cryptocurrency Launch, Facebook Sets Its Path Toward Becoming an Independent Nation',
 'What Do Mark Zuckerberg, Marco Rubio, and Patrick Drahi Have in Common?',
 'Florida City Ransom Payment Could Open Door to More Attacks',
 'Facebook cryptocurrency: what it aims to be, why it has led to concern',
 'Post-Ransomware Attack, Florida City Pays $600K',
 'Facebook’s New Libra Digital Currency, Trust Issues (Many), and Sovereignty',
 "Libra: An Interesting Idea, If Only Facebook Weren'tInvolved",
 "Facebook's Libra Needs To Answer Three Questions"
{% endhighlight raw %}

#### Preprocessing

```python
def preprocessing(corpus):
    # initialize
    clean_text = []

    for row in corpus:
        # tokenize
        tokens = nltk.tokenize.word_tokenize(row)
        # lowercase
        tokens = [token.lower() for token in tokens]
        # isword
        tokens = [token for token in tokens if token.isalpha()]
        clean_sentence = ''
        clean_sentence = ' '.join(token for token in tokens)
        clean_text.append(clean_sentence)
        
    return clean_text
    
all_text = preprocessing(all_text);
```

#### Count Vector Representation

We'll use the CountVectorizer in sklearn.

```python
# sklearn countvectorizer
from sklearn.feature_extraction.text import CountVectorizer
# Convert a collection of text documents to a matrix of token counts
cv = CountVectorizer(ngram_range=(1,1), stop_words = 'english')
# matrix of token counts
X = cv.fit_transform(all_text)
Xc = (X.T * X) # matrix manipulation
Xc.setdiag(0) # set the diagonals to be zeroes as it's pointless to be 1
```

#### Convert into matrix table

```python
import pandas as pd
names = cv.get_feature_names() # This are the entity names (i.e. keywords)
df = pd.DataFrame(data = Xc.toarray(), columns = names, index = names)
df.to_csv('to gephi.csv', sep = ',')
```

Quick look at the matrix table
![0620-table.png](/assets/images/blog/2019-06-20-co-occurrence-matrix-visualization/0620-table.png)


#### Visualization

Visualization of the co-occurrence matrix network is done by using Gelphi, a open-source software to visualize network. 
![0620-gelphi1.png](/assets/images/blog/2019-06-20-co-occurrence-matrix-visualization/0620-gelphi1.png)  

Drawing layout: Fruchterman Reingold (Force-directed graph drawing)  

![0620-gelphi2.png](/assets/images/blog/2019-06-20-co-occurrence-matrix-visualization/0620-gelphi2.png)   

![0620-gelphi3.png](/assets/images/blog/2019-06-20-co-occurrence-matrix-visualization/0620-gelphi3.png)   


Seems like the Libra is the hottest topic of cryptocurrency on 06/20 today!


