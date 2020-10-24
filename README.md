# Overview

Suppose we have the acronym _CSS_. It can mean "Cascading Style Sheets" or "Chirp Spread Spectrum" depending on the context. This work trains an ML model to disambiguate acronyms based on the context.

This work is derived from the work of [Varma and Gardner (2017)](https://github.com/maya124/AcronymLookup) with some updates done by [Reddy (2020)](https://github.com/teja0508/AcronymLookup). In particular, this work is concerned with domains relevant to Devopedia (Computer Science, Electronics, Telecommunications).


# Installation

If running in Google Colab, simply run `Acronyms.ipynb`. Else, following this procedure:
- Run `pip install -r requirements.txt` to set up Python dependencies.
- Install and configure Postgres. DB schema is in file `setUpDb.sql`. On Linux, these can be done via commands:
```
    sudo apt-get -y -qq update
    sudo apt-get -y -qq install postgresql
    sudo service postgresql start
    sudo -u postgres psql -U postgres -c "ALTER USER postgres PASSWORD 'pgpwd';"
    sudo -u postgres psql -U postgres -c 'DROP DATABASE IF EXISTS acronyms;'
    sudo -u postgres psql -U postgres -c 'CREATE DATABASE acronyms;'
    sudo -u postgres psql -U postgres -d acronyms -f postgres/setUpDb.sql
```


# Process

- Data Collection:
    - Wikipedia is used as the data source. We start with a seed in `data/seed.json`.
    - Articles in the seed have been pre-selected from a Wikipedia page on [Computing & IT abbreviations](https://en.wikipedia.org/wiki/List_of_computing_and_IT_abbreviations).
    - From the seed, obtain Wikipedia page titles and URLs: `python get_urls.py`. Output is saved in `data/data.csv`.
    - For all URLs in `data/data.csv`, download and save content: `python download.py`. Downloaded files are saved in `data/train` and `data/test` folders.
- Data Pre-processing:
    - Extract acronym definitions and context by treating this as a Constraint Satisfaction Problem (CSP): `python csp/main.py`. 
    - Extracted data is saved in `definitions.csv`.
    - **TODO**: This extraction is not working very well at the moment. File `definitions.csv` has been manually edited.
- Model Training and Validation:
    - Data is read from database. Downloaded content is also used.
    - Train by calling `python train.py`. Multiple classifier models are saved as a `trained_models/*.pkl` files.
- Model Use:
    - Call `python serve.py {model} {some string with acronym}`, such as `python serve.py svc 'ALU is an essential part of a computer along with memory and peripherals.'`
