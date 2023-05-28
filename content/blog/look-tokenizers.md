---
title: 'Taking a look at (some) tokenizers'
date: 2023-05-28
Tags: [English]
Categories: [article]
draft: false
---

Recently I have been working on writing a tokenizer from scratch in Rust. 
In the process, I wanted to _really_ understand the implementation of some commonly used 
tokenizers. Fun, I know.


### Moses Tokenizer 
_Webpage http://www2.statmt.org/ and the implementation that I'll talk about 
[tokenizer.perl](https://github.com/moses-smt/mosesdecoder/blob/master/scripts/tokenizer/tokenizer.perl)._

If you already know Moses, you know. Silent nod. For everyone else, Moses is an NLP framework written in Perl, focused on statistical machine translation. 
It is super easy to install anywhere and everyone loves it because of it.
Aside from language models and statistical machine translation utilities, it also includes tools to clean text, namely punctuation normalization, tokenization and cleaning corpora by limiting sentence length.
From these I am concerned with the normalize-punctuation script and the tokenizer itself, because I don't remember ever **not** using punctuation normalization along with the tokenizer.

The normalize-punctuation script ([normalize-punctuation.perl](https://github.com/moses-smt/mosesdecoder/blob/master/scripts/tokenizer/normalize-punctuation.perl)
) removes carriage return characters (`\r`) and normalizes spacing around parentheses and punctuation marks. 
After that it normalizes punctuation marks (e.g. different types of quotation marks, hyphens and pseudo-spaces). 
There are a few special cases, like `« »` being the de-facto quotes for French as indicated in the code, which are normalized to `" "`. This is also the case for Spanish, where English quotation marks are only supposed to be used in nested quotations (`«Gritó "NOOOOOOO" cuando le pidieron instalar Moses », dijo su antiguo supervisor`).


Now let's go in depth about each step of the process in the tokenizer. Feel free to follow along with me, we are going to go through [tokenizer.perl](https://github.com/moses-smt/mosesdecoder/blob/master/scripts/tokenizer/tokenizer.perl). We start in line 227!
Right at the beginning there is an optional call to using Penn Treebank tokenization. Let's ignore this for now since it is a specific tokenization strategy not exclusive to Moses. 
It used to be more popular, but I was born a little late to know exactly why I don't see it pop up much anywhere. Do you know?
1. **Simple cleaning** (line 235-240):
   1. The input text is chomp(ed) to remove the trailing newline character, and then spaces are added at the beginning and end of the text.
   2. ASCII junk characters (with ASCII values between 0 and 31) are removed from the text.
2. **Tag protected patterns** (lines 243-255):
   1. The function searches for protected patterns in the text. It iterates over a list of protected patterns stored in the @protected_patterns array and captures matches in the text using regular expressions. The matched patterns are stored in the @protected array.
   2. The function replaces the protected patterns in the text with unique substitution strings (e.g., THISISPROTECTED000, THISISPROTECTED001, etc.) to temporarily remove them from further processing.
3. **Cleaning up multiple, leading and trailing spaces** (lines 256-258):
   1. Multiple consecutive spaces are replaced with a single space. 
   2. Leading and trailing spaces are removed.
4. **Separate out all remaining special characters** (lines 261-283):
   Special characters are separated from the surrounding text by spaces. The specific rules for separating special characters depend on the language being processed. For example, in Finnish and Swedish, a colon may be separated unless immediately followed by lowercase characters. 
5. **Optional aggressive hyphen splitting** (lines 286-289):
   Only performed if the $AGGRESSIVE flag is true. It separates a hyphen surrounded by alphanumeric characters with spaces. 
6. **Multi-dot tagging** (lines 292-297):
   1. Multiple consecutive dots (e.g., "...") are replaced with a placeholder string DOTMULTI followed by the number of dots. 
   2. Multi-dots at the end of the tokenized sentences are split into a sequence of DOTDOTMULTI followed by a dot and a space. 
7. **Comma management** (lines 307-311):
   Commas are separated from the surrounding text by spaces, except when they are within numbers (e.g., "5,300"). 
8. **Split contractions** (lines 323-351):
Apostrophes and single quotation marks are separated from the surrounding text by spaces, following language-specific rules for contractions and word boundaries. The specific strategy depends on the language chosen, with the general case being to add a space before and after any apostrophe.
9. **Word tokenization** (lines 354-380): Word tokenization is performed by splitting the text into an array of words using whitespace as the delimiter. Then, specific rules are applied to each word to handle abbreviations, contractions, and punctuation. 
10. **Remove extraneous spaces from the tokenized text** (lines 383-385) 
11. **Handle `.'` at the end of sentence** (line 388): A special case is handled to ensure that a period followed by an apostrophe at the end of a sentence is properly separated. 
12. **Restore protected patterns** (lines 391-394): The previously protected patterns, represented by substitution strings, are restored. 
13. **Restore multi-dots** (lines 397-401): Any remaining placeholder strings (DOTMULTI and DOTDOTMULTI) are replaced with the appropriate number of dots. 
14. **Escape special chars** (line 404-414): If the $NO_ESCAPING flag is false, certain special characters are escaped using HTML/XML entity references. 
15. **Ensure final line break** (line 417)


The tokenizer has special cases when separating special characters for English, French, Italian, Gaelic, Catalan, Somali, and Tetun. For any other language, it inserts a space before and after any character that is not alphanumeric, space, period, apostrophe, backtick, comma, or hyphen.
This seems like a short, odd collection of languages (at first it looked that way to me too). But keep in mind that the tokenizer is pretty much exclusively operating with special characters and punctuation marks.
I also feel that it assumes that you have used normalize-punctuation.perl prior to this, since it is standard practice. 

Overall, Moses' tokenizer is relatively simple. 
As with any tokenizer, do not make the mistake of thinking that the order of operations does not matter (it does).
However, it is also somewhat limited, especially because it requires to be fed a file with newline-separated text (because it is very much the norm in machine translation datasets). 
Inputting a giant chunk of text in a single line will result in loading in memory the full string at once _and_ not using any parallelization, since it is based on splitting all the lines in the text in batches and distributing those.
And even if you were passing a nicely formatted text with each sentence in a different lines, Perl uses by default interpreted-based threads. 
In principle these are underlain by system threads so the OS may allocate them in different cores if possible. But I am honestly not sure, I have barely worked with Perl.
You can try to start looking around from here https://www.perlmonks.org/?node_id=710237. I love the energy of this website. The basic HTML. The snarky remarks in questions.
The uncertainty of whether you are being advised by Josh from an office that uses Windows 98 or a member of the C++ Committee.

Okay a sidebar.

This thread https://www.perlmonks.org/?node_id=11151615. 
- "That is just the tip of the iceberg in my experience, with the younger guys eager to use a multitude of Infrastructure as code tools, such as Puppet and Ansible ... and new DevOps tools, often requiring JVM languages, such as Groovy ... and Microservices ... and trendy new statically typed languages, such as Golang and Rust".

Ouch.
- "For a brief moment, I even considered learning some Python for these two purposes but I quickly came to my senses and wrote a Perl module to do what I wanted with OpenAI".

Fortunately, he quickly came to his senses.


### SpaCy Tokenizer
_Official page https://spacy.io/api/tokenizer.

I happen to be very familiar with SpaCy because it is used in my workplace. 
When it comes to text processing, especially tokenization or sentence segmentation, it is probably as good as they come.
Other than in cases where your use case is very specifically machine translation, I don't really see a reason to use Moses over SpaCy for text cleaning purposes. 
That said, like many other things in Python and SpaCy, it is slower than it could theoretically be, in spite of being implemented in Cython. And with very little parallelization. 
But that's how we roll in NLP, and otherwise I quite like SpaCy because they gave me shiny stickers once.

The root tokenizer in SpaCy is located here [spacy/tokenizer.pyx](https://github.com/explosion/spaCy/blob/master/spacy/tokenizer.pyx). 
Let's go over its lovely Cython code. The main tokenization function `__call__` is divided in two steps:

##### 1. Tokenize affixes

As per their documentation:
> The task here is much like string.split, but not quite. 
We find spans of whitespace and non-space characters, and ignore spans that are exactly `' '`. 
So, our sequences will all be separated by either `' '` or nothing.

But enough literature, here's a breakdown of the implementation 🤓:

- Iterate over each `uc`character in the input string.
  - Check if the current character's whitespace status (whether it is a classified whitespace according to Unicode standards) is different from the previous character's whitespace status `in_ws`.
    - If `start` (the variable that indicates the last word boundary position) is < `i` (the current position in the string). If so, a word boundary has been encountered.
      - Extract the span of characters from `start` to `i`, which becomes a new `span`. Calculate a hash value for the `span` using the `hash_string` function and assigns it to the `key` variable.
      - It checks if the current span has special characters or other tokens not cached in the current vocabulary. If so, it includes them in the vocabulary using the `_tokenize` function.
    - Check if `uc` is a literal space character `' '`.
      - If so, set the attribute `spacy` (which indicates wether the token is a space token) of the last token in `doc` to `True` and `start` becomes equal to the next position in the loop (`i + 1`).
      - Else, only set `start` to the value of the current position `i`.
    - Negate `in_ws` (the variable that stores the whitespace value of the previous character).
  - Increment current position `i` by 1.
- After iterating through all characters, if `start < i` (the last word boundary position is behind the last position of the string), then there's a leftover span to process. 
  - Again, it checks if the current span has special characters or other tokens not cached in the current vocabulary. If so, it includes them in the vocabulary using the `_tokenize` function.
  - Set the attribute `spacy` of the last character of `doc` to `True` if the last character of the string is a space and the penultimate character on it is not a space.

The check-chached-tokens step has this piece of comment in it 
```
# When we want to make this fast, get the data buffer once
# with PyUnicode_AS_DATA, and then maintain a start_byte
# and end_byte, so we can call hash64 directly. That way
# we don't have to create the slice when we hit the cache.
```

Does not seem like it has been done because data buffer is still a string slice, and it's not using hash64. Something that anyone can add?

##### 2. Apply special cases

Described as:
> Retokenize the doc according to special cases

- Find matches for special cases (with `_special_matcher.find_matches`) with and stores the results in `c_matches`. If there are no matches, the function finishes.
- Filter special cases in `c_matches` with `_filter_special_spans`. This function checks if each span overlaps with previously seen tokens. If a span does not overlap, it is added to the filtered list. This filtering ensures that only non-overlapping spans are considered for modification.
- The function calls the `_prepare_special_spans` method to prepare the special spans for modification. This method creates a dictionary `span_data` that contains information about each special span, such as its start index, end index, length difference, and the original text. It also determines whether modifications never increase doc length (`modify_in_place`).
- If `modify_in_place` then do so, otherwise create a separate array to store modified tokens.
- The `_retokenize_special_spans` method is called to modify the tokenization according to the filtered special cases. It copies the tokens from the document into the `tokens` array, adjusting the offsets if necessary.
- Additional memory is allocated for the document if needed.
- If modifications were not made in place, the modified tokens are copied back to the document.

You will have noticed that SpaCy's tokenizer also has to deal with maintaining a vocabulary and an object with the information about the parsed spans.
This tokenizer does not exist in a vacuum: SpaCy's intended use is to parse text and obtain a `Doc` object containing lots of information for each of the tokens, such as lemma, original form, lowercase...
As a result, I do not exactly super like this tokenizer to exemplify how they generally work or use as a reference. 
Much of the action is going on in `_prepare_special_spans` and `_retokenize_special_spans`, which I did not break down line-for-line like in Moses' case. It is already hard enough to read this text as it stands.

And the reality of many tokenizers is that they exist as an accessory to a larger NLP suite, or they complement another tool like a sentence splitter, or a lemmatizer. 
Often they are concerned with things other than throughput. Moses and SpaCy have pure rule-based tokenizers, arguably the most intuitive implementation of a tokenizer.
However, HuggingFace's `transformers` library offers an array of tokenizers that are focused on subword tokenization, 
be it through Byte-Pair Encoding or Wordpiece (Miko Schuster and Kaisuke Nakajima, 2012)[^fn1] or forego pretokenization entirely, like in the case of SentencePiece. 
All of which are much better than rule-based tokenizers at generating smaller vocabularies that are less memory-hungry, a very important feature when working with large language models.

The latter is especially useful for languages that do not use spaces to separate words,
such as Chinese, Japanese and Thai. Moses is completely helpless for these languages, and SpaCy, for instance defaults to using SudachiPy (https://github.com/WorksApplications/SudachiPy) mode A for Japanese, 
I am guessing because there is no convenient way to integrate parsing for Japanese into the standard flow. 
In general any rule-based tokenizer is going to suffer and have to implement radically different logics for certain languages.
Subword-based tokenizers (pure BPE, WordPiece, SentencePiece, etc.) are much more flexible and will do alright for pretty much any language.
But their output is not suitable for many tasks that do **not** involve a large neural network. Is SentencePiece even tokenization when it's mostly concerned with producing a nice vocabulary?
I had an full-blown existential crisis over this during my master's thesis that made me start a Rafa Nadal highlights loop and join my university's tennis classes the next day.
Also an aside, BPE may be one of the most used misnomers in NLP. We all cite this paper (Sennrich et al., 2015)[^fn2]. But BPE, the _name_ was originally conceived as a basic data compression technique which operated by finding the most frequently occurring pairs of adjacent bytes in the data and replacing all instances of the pair with a byte that was not in the original data, repeating the process until no further compression is possible. 
Yes, applied to text compression, this algorithm functions at the character level. But like, we should have given it some name. Now BPE is an NLP-only thing. Whatever.


I have been thinking too much this afternoon. You get to decide what is and what is not tokenization. Bye.


[^fn1]: Japanese and Korean Voice Search - Miko Schuster and Kaisuke Nakajima - https://static.googleusercontent.com/media/research.google.com/ja//pubs/archive/37842.pdf
[^fn2]: Neural Machine Translation of Rare Words with Subword Units - Rico Sennrich, Barry Haddow, Alexandra Birch - https://arxiv.org/pdf/1508.07909.pdf
