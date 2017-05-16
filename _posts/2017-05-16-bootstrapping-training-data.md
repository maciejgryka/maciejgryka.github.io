---
layout: post
title:  "Bootstrapping training data, or free lunch"
date:   2017-05-16 7:30:00 +0200
---
A while ago our CTO, [Russ](https://twitter.com/rhs), asked me about bootstraping some training data to play with one ML thing or other that he had in mind. The idea was to avoid the effort of manual labeling (he's kind of lazy...) by coming up with a bunch of rules that would classify the data well enough to feed into a supervised training algorithm later.

My immediate reaction was: sure, go ahead and play around, but don't expect to get anything better than your initial rules out of this! Your new ML classifier will try very hard to fit whatever training data you give it and all the biases and inaccuracies of your initial rule-based approach will be embedded within. You might as well just use the rules and forget about all [the hidden costs of using ML in production](https://research.google.com/pubs/pub43146.html).

Well, it turns out I overlooked some nuance and a recent post from [David Robinson](https://twitter.com/drob), talking about [how they build TaggerNews](http://varianceexplained.org/programming/tagger-news/) showed me the light.

See, while it's true that using rules to generate your training data is not ideal, there are ways you can make it work well enough to solve specific problems. One of the tricks is to do the rule-based classification using one representation of your data and compute your feature vector for the ML algo using another one. The second representation can be more descriptive, but at the same time more difficult to create rules for. As long as your rules are good enough to cover some reasonable subset of the training data, you might be able to get away with a mostly-free lunch by automatically labeling your samples.

Take the example of TaggerNews: they used regex to create a training set, where each article was assigned a class label based on title alone. This method is not perfect: besides biases, which are always difficult to avoid, it is unable to classify many articles entirely. As an example, a title like "Tim Cook gets a YubiKey" would get tagged neither with "Apple", nor with "Security", because it doesn't contain any of the keywords the regexes are looking for. The cool thing is, it doesn't matter! The training data contains many other, appropriately rule-tagged, Apple- and Security-related articles whose actual feature vectors, based on the full body of the article, will likely be similar to each other.

The lunch is still not entirely free: David and his team did a bunch of work to iterate on their initial rules to make sure they're good enough: adjusting some regexes and discarding some labels completely to not feed the final classifier junk. Still, this is much less time-consuming (and much more enjoyable!) than hand-labeling 10k data points.

I might try this next time I'm playing with an unlabeled dataset - and now you also have my blessing Russ!