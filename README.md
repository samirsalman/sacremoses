# Sacremoses

[![Build Status](https://travis-ci.org/alvations/sacremoses.svg?branch=master)](https://travis-ci.org/alvations/sacremoses)
[![Build status](https://ci.appveyor.com/api/projects/status/bwgmj4axw9pdk1oq?svg=true)](https://ci.appveyor.com/project/alvations/sacremoses)

# License

MIT License.

# Install

```
pip install -U sacremoses
```

NOTE: Sacremoses only supports Python 3 now (`sacremoses>=0.0.41`). If you're using Python 2, the last possible version is `sacremoses==0.0.40`.

# Usage (Python)

## Tokenizer and Detokenizer

```python
>>> from sacremoses import MosesTokenizer, MosesDetokenizer
>>> mt = MosesTokenizer(lang='en')
>>> text = 'This, is a sentence with weird\xbb symbols\u2026 appearing everywhere\xbf'
>>> expected_tokenized = 'This , is a sentence with weird \xbb symbols \u2026 appearing everywhere \xbf'
>>> tokenized_text = mt.tokenize(text, return_str=True)
>>> tokenized_text == expected_tokenized
True


>>> mt, md = MosesTokenizer(lang='en'), MosesDetokenizer(lang='en')
>>> sent = "This ain't funny. It's actually hillarious, yet double Ls. | [] < > [ ] & You're gonna shake it off? Don't?"
>>> expected_tokens = ['This', 'ain', '&apos;t', 'funny', '.', 'It', '&apos;s', 'actually', 'hillarious', ',', 'yet', 'double', 'Ls', '.', '&#124;', '&#91;', '&#93;', '&lt;', '&gt;', '&#91;', '&#93;', '&amp;', 'You', '&apos;re', 'gonna', 'shake', 'it', 'off', '?', 'Don', '&apos;t', '?']
>>> expected_detokens = "This ain't funny. It's actually hillarious, yet double Ls. | [] < > [] & You're gonna shake it off? Don't?"
>>> mt.tokenize(sent) == expected_tokens
True
>>> md.detokenize(tokens) == expected_detokens
True
```


## Truecaser

```python
>>> from sacremoses import MosesTruecaser, MosesTokenizer

# Train a new truecaser from a 'big.txt' file.
>>> mtr = MosesTruecaser()
>>> mtok = MosesTokenizer(lang='en')

# Save the truecase model to 'big.truecasemodel' using `save_to`
>> tokenized_docs = [mtok.tokenize(line) for line in open('big.txt')]
>>> mtr.train(tokenized_docs, save_to='big.truecasemodel')

# Save the truecase model to 'big.truecasemodel' after training
# (just in case you forgot to use `save_to`)
>>> mtr = MosesTruecaser()
>>> mtr.train('big.txt')
>>> mtr.save_model('big.truecasemodel')

# Truecase a string after training a model.
>>> mtr = MosesTruecaser()
>>> mtr.train('big.txt')
>>> mtr.truecase("THE ADVENTURES OF SHERLOCK HOLMES")
['the', 'adventures', 'of', 'Sherlock', 'Holmes']

# Loads a model and truecase a string using trained model.
>>> mtr = MosesTruecaser('big.truecasemodel')
>>> mtr.truecase("THE ADVENTURES OF SHERLOCK HOLMES")
['the', 'adventures', 'of', 'Sherlock', 'Holmes']
>>> mtr.truecase("THE ADVENTURES OF SHERLOCK HOLMES", return_str=True)
'the ADVENTURES OF SHERLOCK HOLMES'
>>> mtr.truecase("THE ADVENTURES OF SHERLOCK HOLMES", return_str=True, use_known=True)
'the adventures of Sherlock Holmes'
```

## Normalizer

```python
>>> from sacremoses import MosesPunctNormalizer
>>> mpn = MosesPunctNormalizer()
>>> mpn.normalize('THIS EBOOK IS OTHERWISE PROVIDED TO YOU "AS-IS."')
'THIS EBOOK IS OTHERWISE PROVIDED TO YOU "AS-IS."'
```

# Usage (CLI)

Since version `0.0.42`, the pipeline feature for CLI is introduced, thus there
are global options that should be set first before calling the commands:

 - language
 - processes
 - encoding
 - quiet

```shell
$ pip install -U sacremoses>=0.0.42

$ sacremoses --help
Usage: sacremoses [OPTIONS] COMMAND1 [ARGS]... [COMMAND2 [ARGS]...]...

Options:
  -l, --language TEXT      Use language specific rules when tokenizing
  -j, --processes INTEGER  No. of processes.
  -e, --encoding TEXT      Specify encoding of file.
  -q, --quiet              Disable progress bar.
  --version                Show the version and exit.
  -h, --help               Show this message and exit.

Commands:
  detokenize
  detruecase
  normalize
  tokenize
  train-truecase
  truecase
```

## Pipeline

Example to chain the following commands:

 - `normalize` with `-c` option to remove control characters.
 - `tokenize` with `-a` option for aggressive dash split rules.
 - `truecase` with `-a` option to indicate that model is for ASR 
   - if `big.truemodel` exists, load the model with `-m` option,
   - otherwise train a model and save it with `-m` option to `big.truemodel` file.
 - save the output to console to the `big.txt.norm.tok.true` file.

```shell
cat big.txt | sacremoses -l en -j 4 \
    normalize -c tokenize -a truecase -a -m big.truemodel \
    > big.txt.norm.tok.true
```

## Tokenizer

```shell
$ sacremoses tokenize --help
Usage: sacremoses tokenize [OPTIONS]

Options:
  -a, --aggressive-dash-splits   Triggers dash split rules.
  -x, --xml-escape               Escape special characters for XML.
  -p, --protected-patterns TEXT  Specify file with patters to be protected in
                                 tokenisation.
  -c, --custom-nb-prefixes TEXT  Specify a custom non-breaking prefixes file,
                                 add prefixes to the default ones from the
                                 specified language.
  -h, --help                     Show this message and exit.


 $ sacremoses -l en -j 4 tokenize  < big.txt > big.txt.tok
100%|██████████████████████████████████| 128457/128457 [00:05<00:00, 24363.39it/s

 $ wget https://raw.githubusercontent.com/moses-smt/mosesdecoder/master/scripts/tokenizer/basic-protected-patterns
 $ sacremoses -l en -j 4 tokenize -p basic-protected-patterns < big.txt > big.txt.tok
100%|██████████████████████████████████| 128457/128457 [00:05<00:00, 22183.94it/s
```

## Detokenizer

```shell
$ sacremoses detokenize --help
Usage: sacremoses detokenize [OPTIONS]

Options:
  -x, --xml-unescape  Unescape special characters for XML.
  -h, --help          Show this message and exit.

 $ sacremoses -l en -j 4 detokenize < big.txt.tok > big.txt.tok.detok
100%|██████████████████████████████████| 128457/128457 [00:16<00:00, 7931.26it/s]
```

## Truecase

```shell
$ sacremoses truecase --help
Usage: sacremoses truecase [OPTIONS]

Options:
  -m, --modelfile TEXT            Filename to save/load the modelfile.
                                  [required]
  -a, --is-asr                    A flag to indicate that model is for ASR.
  -p, --possibly-use-first-token  Use the first token as part of truecase
                                  training.
  -h, --help                      Show this message and exit.

$ sacremoses -j 4 truecase -m big.model < big.txt.tok > big.txt.tok.true
100%|██████████████████████████████████| 128457/128457 [00:09<00:00, 14257.27it/s]
```

## Detruecase

```shell
$ sacremoses detruecase --help
Usage: sacremoses detruecase [OPTIONS]

Options:
  -j, --processes INTEGER  No. of processes.
  -a, --is-headline        Whether the file are headlines.
  -e, --encoding TEXT      Specify encoding of file.
  -h, --help               Show this message and exit.

$ sacremoses -j 4 detruecase  < big.txt.tok.true > big.txt.tok.true.detrue
100%|█████████████████████████████████| 128457/128457 [00:04<00:00, 26945.16it/s]
```

## Normalize

```shell
$ sacremoses normalize --help
Usage: sacremoses normalize [OPTIONS]

Options:
  -q, --normalize-quote-commas  Normalize quotations and commas.
  -d, --normalize-numbers       Normalize number.
  -p, --replace-unicode-puncts  Replace unicode punctuations BEFORE
                                normalization.
  -c, --remove-control-chars    Remove control characters AFTER normalization.
  -h, --help                    Show this message and exit.

$ sacremoses -j 4 normalize < big.txt > big.txt.norm
100%|██████████████████████████████████| 128457/128457 [00:09<00:00, 13096.23it/s]
```
