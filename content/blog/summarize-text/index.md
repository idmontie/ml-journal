---
title: Summarize Text
date: "2019-04-22T18:59:05.476Z"
---

## Problem

I run a newsletter [Stellar](https://stellar.clarityhub.io) that lets subscribers know about active, beginner-friendly issues that they can contribute to to help the open source community and also improve their programming skills.

Part of this newsletter currently involves manually taking an issue's description and distilling it down to a 1 to 2 sentence human-readable summary. Because of all the different formats that repos and contributors write their issues in, it is very difficult to simply take the issue description markdown and pluck sentences from it.

What we would like to do is:

**Take Github issue descriptions and summarize them into 1 to 2 sentence summaries.**

These issue descriptions are markdown and may contain HTML comments, images, code, and other markdown that cannot be summarized into text. The issue descriptions may be all structured differently, so looking for keywords like "Description" or "Feature request" does not consistently give us the section that best summarizes the Github issue.

## Prior Work

Before trying to come up with my own novel approach, I looked at other tools that already exist to summarize text like articles and see if there are open source frameworks I can use to quickly summarize Github Issues.

### SMMRY

[SMMRY](https://smmry.com/) powers a popular bot on Reddit that takes news articles that are linked to by other posts and attempts to summarize the data.

My guess is that this "free-service" is a way to also train the algorithm by using upvotes and downvotes as validation that a summarization was good or bad

The algorithm is not open source, but the steps used in the algorithm are listed on their website:

1. Rank sentences by importances
    1. Associate words with their grammatical counterparts (ie, citie -> city)
    2. Calculate the occurrence of each word in the text
    3. Assign each word with points depending on their popularity
    4. Detect which periods represent the end of a sentence (Mr. Clause is not two sentences)
    5. Split up the text into individual sentences.
    6. Rank sentences by the sum of their words' points.
    7. Return X of th emost highly ranked sentences in chronological order.
2. Reorganize the summary to focus on a typic
3. Remove transition phrases
4. Remove unnecessary clauses.
5. Remove excessive examples.

### pytextrank

[pytextrank](https://github.com/DerwenAI/pytextrank) uses multiple stages to extract keywords and sentences from a given article.

### Article-Summarizer

[Article-Summarizer](https://github.com/LazoCoder/Article-Summarizer) uses frequency analysis to summarize text.

### text-summarization-tensorflow

[text-summarization-tensorflow](https://github.com/dongjun-Lee/text-summarization-tensorflow) is a simple Tensorflow implementation of text summarization using seq2seq library.

## Data

What I'll do is create an Excel sheet with the issue links of issues we have sent via the newsletter, get their full markdown, and a column of the user created summaries, and then have columns for each methods' summary.

Something like the following:

| Issue Link | Raw Markdown Description | User Summary | Method 1 Summary | Method 2 Summary | etc. |
|---------|---------|---------|------|------|-----|
|         |         |         |      |      |     |

### Clean Data

Before we attempt to summarize the data, we should go through and clean the data to give the summarization algorithms the best chance at generating a readable excerpt.

1. Remove markdown that is not necessary:
    1. Remove comments
    2. Remove images and code tags

Here is the code I used to pull down issue data and clean it:

```js
const fs = require('fs');
const Octokit = require('@octokit/rest');
const removeMd = require('remove-markdown');

const octokit = new Octokit();

const issues = [
  'https://github.com/30-seconds/30-seconds-of-code/issues/829',
  'https://github.com/mui-org/material-ui/issues/13695',
  'https://github.com/grommet/grommet/issues/2981',
  'https://github.com/30-seconds/30-seconds-of-code/issues/774',
  'https://github.com/aws-amplify/amplify-js/issues/1538',
];

function clean (text) {
  // Remove comments
  let clean = text.replace(/<!--[\s\S]*?-->/g, '');

  // remove code blocks (but not inline code)
  clean = clean.replace(/(`{3,})(.*?)\1/gm, '');

  // remove tables
  clean = clean.replace(/^\|(.*)\|$/gm, '');

  // remove headings
  clean = clean.replace(/^\#+ (.*)$/gm, '$1');

  console.log(clean);

  return removeMd(clean);
}

async function run() {
  const issuesData = await Promise.all(issues.map(async (issue) => {
    const parts = issue.split('/');
    owner = parts[3];
    repo = parts[4];
    issueNumber = parts[6];
    const id = `${owner}/${repo}/${issueNumber}`;

    const result = await octokit.issues.get({
      owner,
      repo,
      issue_number: issueNumber
    });

    return {
      id,
      text: clean(result.data.body),
    };
  }));

  return await new Promise((resolve, reject) => {
    fs.writeFile('./data.json', JSON.stringify(issuesData, null, 4), (err) => {
      if (err) {
        reject(err);
      }
      resolve();
    });
  });
}

run();

```

## Summary Attempts

The following are summaries of different attempts I took when trying to summarize Github issues.

### Using SMMRY

Most Github issue body texts are too short for SMMRY. After cleaning the data, most result in "TEXT IS TOO SHORT" from SMMRY's online tool.

### Using pytextrank

Since Github issue body texts are short, pytextrank will return almost the entire body text. This results in summaries that are about the same length as the input text.

### Using Article Summarizer

Article Summarizer does well summarizing the issue descriptions to 3 sentences. However, a more sophistacted way of determining sentences would be needed since it splits naively on periods. Newlines do not seem to be taken into account. There is a package called [TextBlob](https://textblob.readthedocs.io/en/dev/) that might improve parsing sentences.

### Using text-summarization-tensorflow

Text Summarization with Tensorflow is mainly geared to generating novel article titles based on article bodies. This is not exactly what we are looking to do with the issue summaries. However, it was interesting to see the resulting one sentence titles of issues based on their bodies.

```
https://github.com/mui-org/material-ui/issues/13695
> Actual Title: Rendering issue when using badges and noWrap text in List in IE11
> Generated Title: 
```

## Moving Forward

What I ended up doing is going with a modified version of Article Summarizer as a starting point for summarizing Github issues. I changed the sentence parsing to use [TextBlob](https://textblob.readthedocs.io/en/dev/) which provided a nice wrapping API around [NLTK](https://www.nltk.org/api/nltk.tokenize.html). Changes to the repo can be found [here](https://github.com/idmontie/Article-Summarizer).

I'll need to continue messing with the scoring algorithm to fit Github issue instead of scoring articles. Using something like `text-summarization-tensorflow` to generate novel sentences may be required as well since Github issues typically have lots of partial sentences as well.

