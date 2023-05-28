---
title: 'Taking a look at (some) tokenizers'
date: 2023-06-05
Tags: [English]
Categories: [article]
draft: true
---

Recently I have been working on writing a tokenizer from scratch in Rust. 
In the process, I wanted to _really_ understand the implementation of some commonly used 
tokenizers. Fun, I know.


### Moses Tokenizer 
_Webpage http://www2.statmt.org/ and the implementation that I'll talk about 
https://github.com/moses-smt/mosesdecoder/blob/master/scripts/tokenizer/tokenizer.perl._

If you already know Moses, you know. Silent nod. For everyone else, Moses is an NLP framework written in Perl, focused on statistical machine translation. 
It is super easy to install anywhere and everyone loves it because of it.
Aside from language models and statistical machine translation utilities, it also includes tools to clean text, namely punctuation normalization, tokenization and cleaning corpora by limiting sentence length.
From these I am concerned with the normalize-punctuation script and the tokenizer itself, because I don't remember ever **not** using punctuation normalization along with the tokenizer.

The normalize-punctuation script (https://github.com/moses-smt/mosesdecoder/blob/master/scripts/tokenizer/normalize-punctuation.perl
) removes carriage return characters (`\r`) and normalizes spacing around parentheses and punctuation marks. 
After that it normalizes punctuation marks (e.g. different types of quotation marks, hyphens and pseudo-spaces). 
There are a few special cases, like `« »` being the de-facto quotes for French as indicated in the code, which are normalized to `" "`. This is also the case for Spanish, where English quotation marks are only supposed to be used in nested quotations (`«Gritó "NOOOOOOO" cuando le pidieron instalar Moses », dijo su antiguo supervisor`).


Now let's go in depth about each step of the process in the tokenizer. Feel free to follow along with me, we are going to go through https://github.com/moses-smt/mosesdecoder/blob/master/scripts/tokenizer/tokenizer.perl. We start in line 227!
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
_Official page https://spacy.io/api/tokenizer and code that I will be referencing https://github.com/explosion/spaCy/tree/master_

I happen to be very familiar with SpaCy because it is used in my workplace. 
When it comes to text processing, especially tokenization or sentence segmentation, it is probably as good as they come.
Other than in cases where your use case is very specifically machine translation, I don't really see a reason to use Moses over SpaCy for text cleaning purposes. 
That said, like many other things in Python and SpaCy, it is slower than it could theoretically be, in spite of being implemented in Cython. And with very little parallelization. 
But that's how we roll in NLP, and otherwise I quite like SpaCy because they gave me shiny stickers once.

The root tokenizer in SpaCy is located here https://github.com/explosion/spaCy/blob/master/spacy/tokenizer.pyx. 
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

>  When we want to make this fast, get the data buffer once
   with PyUnicode_AS_DATA, and then maintain a start_byte
   and end_byte, so we can call hash64 directly. That way
   we don't have to create the slice when we hit the cache.

##### 2. Apply special cases


You will have notices that SpaCy's tokenizer also has to deal with maintaining a vocabulary and an object with the information about the parsed spans



### Other
Prob HuggingFace transformers tokenizer is quite used nowadays

From https://huggingface.co/transformers/v3.4.0/tokenizer_summary.html 


spaCy and Moses are two popular rule-based tokenizers. On the text above, they’d output something like:

["Do", "n't", "you", "love", "🤗", "Transformers", "?", "We", "sure", "do", "."]
Space/punctuation-tokenization and rule-based tokenization are both examples of word tokenization, which is splitting a sentence into words. While it’s the most intuitive way to separate texts in smaller chunks, it can have a problem when you have a huge corpus: it usually yields a very big vocabulary (the set of all unique tokens used). Transformer XL for instance uses space/punctuation-tokenization, and has a vocabulary size of 267,735!

A huge vocabulary size means a huge embedding matrix at the start of the model, which will cause memory problems. TransformerXL deals with it by using a special kind of embeddings called adaptive embeddings, but in general, transformers models rarely have a vocabulary size greater than 50,000, especially if they are trained on a single language.



