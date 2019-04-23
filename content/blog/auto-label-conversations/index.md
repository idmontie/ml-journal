---
title: Auto-label Conversations
date: "2019-04-22T22:55:38.875Z"
---

## Problem

Given a text conversation between two people, can we auto-label the conversation based on keywords?

## Prior Work

### TextBlob

### pytextrank

## Data

### Cleaning data

## Results

### TextBlob

Using TextBlob to count the most used noun phrases:

```python
#!/usr/bin/env python

from textblob import TextBlob

txt = """RAW CONVERSATION TEXT"""

blob = TextBlob(txt)
key_phrase = (-1, '')

for noun in blob.noun_phrases:
  size = blob.noun_phrases.count(noun)

  if (key_phrase[0] < size):
    key_phrase = (size, noun)

print('key phrase: ', key_phrase)
```

TODO

### pytextrank

Using pytextrank to generate key phrases:

TODO

### Using TextBlob and pytextrank ranking

Use `noun_phrases` from TextBlob and ranking algorithm from pytextrank:

TODO

### Custom

spacy noun_chunks (ORGs, no PERSONs, no CARDINALS)

spacy noun_chunks with ranking algorithm

spacy ents (ORGs, no PERSONs, no CARDINALS)

## Wrapping It Up