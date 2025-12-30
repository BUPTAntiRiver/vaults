Tokenization takes an important part in large language models. A lot of problems come from it, but we cannot avoid it.
Tokenization is about how to transform the words or sequences that is understandable to human into a form that is easier to be perceived by machine.
An simple way is we set an index for every character, that would be easy if only for English, but when considering other languages like Chinese, the vocabulary will become super large.
The modern method is usually byte level tokenization. We only need 256 index for all representation, actually, index is just the value itself. But the problem is this will make *sequence very long*, the original sequence with length 1000 may become 1500. And in transformer complexity grows quadratically with sequence length.
So we use byte pair encoding tokenizer.

## What is new?

We consider pair of bytes, some specific byte pairs may occur more often, so we group them together and assign them a new index to represent them together. After we update the vocab, we also update the original text (in index form), we merge pair indices and do this repeatedly, until we reach the desiring vocabulary size.

## How to implement?

This is where the demons come out. We are operating with terabytes of data, even more. It would be very slow if we just update the whole text to do the merge.

### Pre-tokenization

We can break the text into coarse tokens first like word-level, and track their count, so that we can train the tokenizer base on these unique words which will be much faster.

### Special tokens

There are pre-defined special tokens like `end-of-text` or `start-of-text`  to provide extra information. They are also raw bytes! But we should now train the tokenizer on them. So that when doing pre-tokenization, instead of building a separating pattern, we also need to protect the special tokens and neglect them when building the count table.

### Merging

This becomes the simplest part, just count and update.