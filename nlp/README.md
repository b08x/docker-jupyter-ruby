# NLP Docker Image for Jupyter Ruby Kernel

This directory contains the configuration to build a Docker image specifically tailored for Natural Language Processing (NLP) tasks using the IRuby kernel in Jupyter environments.

## Overview

The `Dockerfile` sets up a Ruby environment (Ruby 3.3.8) with necessary system dependencies and installs a curated list of Ruby gems useful for NLP, data manipulation, AI interactions, and general development within a Jupyter notebook.

## Included Gems

This image comes pre-installed with the following Ruby gems (specified in `Gemfile`):

### Core & Jupyter Integration

* **irb**: Interactive Ruby shell. (Built-in)
* **[iruby](https://github.com/SciRuby/iruby)**: Jupyter kernel for Ruby.
* **[pycall](https://github.com/mrkn/pycall.rb)**: Allows calling Python functions from Ruby.
* **[bundler](https://github.com/rubygems/bundler)**: Manages Ruby application's gem dependencies.

### NLP & Text Processing

* **[bm25f](https://github.com/catflip/bm25f-ruby)**: A fast implementation of the BM25F ranking algorithm for information retrieval systems.
* **[commonmarker](https://github.com/gjtorikian/commonmarker)**: A fast, safe CommonMark parser and renderer.
* **[fuzzy_tools](https://rubygems.org/gems/fuzzy_tools)**: Tools for fuzzy string matching and searching.
* **[kramdown](https://github.com/gettalong/kramdown)**: A fast, pure Ruby Markdown superset converter.
* **[langchainrb](https://github.com/patterns-ai-core/langchainrb)**: Ruby implementation of the LangChain framework for building applications with LLMs.
* **[lingua](https://github.com/dbalatero/lingua)**: Language detection library.
* **[linguistics](https://github.com/ged/linguistics)**: Framework for linguistic analysis of English texts.
* **[linkparser](https://rubygems.org/gems/linkparser)**: Ruby interface to the Link Grammar Parser.
* **[loofah](https://github.com/flavorjones/loofah)**: Library for manipulating and sanitizing HTML fragments.
* **[mimemagic](https://github.com/mimemagicrb/mimemagic)**: Detects the MIME type of a file by its content.
* **[poppler](https://rubygems.org/gems/poppler)**: Ruby bindings for the Poppler PDF rendering library.
* **[pragmatic_segmenter](https://github.com/diasks2/pragmatic_segmenter)**: Rule-based sentence boundary detection.
* **[pragmatic_tokenizer](https://github.com/diasks2/pragmatic_tokenizer)**: Text tokenizer.
* **[ruby-spacy](https://github.com/yohasebe/ruby-spacy)**: Ruby wrapper for the spaCy NLP library (requires Python installation).
* **[srt](https://github.com/cpetersen/srt)**: Library for parsing and manipulating SubRip (.srt) subtitle files.
* **[tf-idf-similarity](https://github.com/jpmckinney/tf-idf-similarity)**: Calculates TF-IDF and cosine similarity between documents.
* **[tokenizers](https://github.com/ankane/tokenizers-ruby)**: Ruby bindings for the Hugging Face Tokenizers library.
* **[treetop](https://github.com/cjheath/treetop)**: Ruby-based parsing expression grammar (PEG) parser generator.
* **[webvtt](https://github.com/jronallo/webvtt)**: Library for parsing and manipulating WebVTT (.vtt) subtitle files.
* **[wordnet](https://hg.sr.ht/~ged/ruby-wordnet)**: Ruby interface to the WordNet lexical database.
* **[wordnet-defaultdb](https://rubygems.org/gems/wordnet-defaultdb)**: Default database files for the `wordnet` gem.

### AI & Machine Learning Interfaces

* **[aia](https://github.com/MadBomber/aia)**: Interface for interacting with various AI models/APIs.
* **[chroma-db](https://github.com/mariochavez/chroma)**: Ruby client for the Chroma vector database.
* **[google-cloud-ai_platform-v1](https://rubygems.org/gems/google-cloud-ai_platform-v1)**: Google Cloud AI Platform client library.
* **[google_custom_search_api](https://github.com/wiseleyb/google_custom_search_api)**: Wrapper for the Google Custom Search JSON API.
* **[google_search_results](https://github.com/serpapi/google-search-results-ruby)**: Scrapes Google Search Results using SerpApi.
* **[groq](https://github.com/drnic/groq-ruby)**: Ruby client for the Groq API.
* **[hugging-face](https://github.com/alchaplinsky/hugging-face)**: Ruby client for the Hugging Face Hub API.
* **[ruby-nano-bots](https://github.com/icebaker/ruby-nano-bots)**: Small, focused bots for specific tasks, often AI-related.
* **[numpy](https://github.com/mrkn/numpy.rb)**: (via `pycall`) Interface to the NumPy Python library for numerical computing.
* **[omniai-google](https://github.com/ksylvest/omniai-google)**: OmniAI provider for Google AI models.
* **[omniai-mistral](https://github.com/ksylvest/omniai-mistral)**: OmniAI provider for Mistral AI models.
* **[omniai-openai](https://github.com/ksylvest/omniai-openai)**: OmniAI provider for OpenAI models.
* **[open_router](https://github.com/OlympiaAI/open_router)**: Ruby client for the OpenRouter API.
* **[pgvector](https://github.com/pgvector/pgvector-ruby)**: PostgreSQL vector similarity search extension support for Ruby.
* **[ruby-openai](https://github.com/alexrudall/ruby-openai)**: Ruby client library for the OpenAI API.
* **[sublayer](https://github.com/sublayerapp/sublayer)**: Ruby client for the Sublayer API.

### Data Handling & Manipulation

* **[algorithms](https://github.com/kanwei/algorithms)**: Ruby implementations of various algorithms and data structures.
* **[daru](https://github.com/SciRuby/daru)**: Data Analysis in RUby (DataFrame implementation).
* **[daru-view](https://github.com/SciRuby/daru-view)**: Interactive plotting library for Daru DataFrames.
* **[hashie](https://github.com/hashie/hashie)**: Collection of classes and mixins that make hashes more powerful.
* **json**: Standard JSON library for Ruby. (Built-in)
* **[jsonl](https://github.com/zenizh/jsonl)**: Library for reading and writing JSON Lines format.
* **[jsonlint](https://github.com/dougbarth/jsonlint)**: Checks JSON files for correct syntax and no silly mistakes.
* **[oj](https://github.com/ohler55/oj)**: A fast JSON parser and object serializer.
* **[ohm](https://github.com/soveran/ohm)**: Object-hash mapping library for Redis.
* **[ohm-contrib](https://github.com/soveran/ohm-contrib)**: Community contributions for the Ohm library.
* **[psych](https://github.com/ruby/psych)**: YAML parser and emitter.
* **[redic](https://github.com/amakawa/redic)**: Lightweight Redis client.
* **[redis](https://github.com/redis/redis-rb)**: Ruby client library for Redis.
* **[sequel](https://github.com/jeremyevans/sequel)**: Database toolkit for Ruby.
* **[yajl-ruby](https://github.com/brianmario/yajl-ruby)**: Ruby bindings for the YAJL JSON parsing library.
* **yaml**: Standard YAML library for Ruby (often uses `psych`). (Built-in, uses psych)
* **[rubyzip](https://github.com/rubyzip/rubyzip)**: Library for working with Zip archives.

### Development & Utilities

* **[amazing_print](https://github.com/amazing-print/amazing_print)**: Pretty prints Ruby objects with proper indentation and colors.
* **[dotenv](https://github.com/bkeepers/dotenv)**: Loads environment variables from `.env` files.
* **[jongleur](https://gitlab.com/RedFred7/Jongleur)**: Task runner utility.
* **[logging](https://github.com/TwP/logging)**: Flexible logging framework for Ruby.
* **[minitest](https://github.com/minitest/minitest)**: Complete suite of testing facilities supporting TDD, BDD, mocking, and benchmarking.
* **open3**: Access to the standard input, standard output, and standard error of subprocesses. (Built-in)
* **[open4](https://github.com/ahoward/open4)**: Variant of `popen3` that allows specifying the execution environment.
* **open-uri**: Easy-to-use wrapper for Net::HTTP, Net::HTTPS and Net::FTP. (Built-in)
* **[openssl](https://github.com/ruby/openssl)**: Ruby bindings for OpenSSL.
* **[os](https://github.com/rdp/os)**: Simple and easy way to access operating system information.
* **[parallel](https://github.com/grosser/parallel)**: Run blocks of code in parallel processes or threads.
* **[pastel](https://github.com/piotrmurach/pastel)**: Terminal string styling library.
* **[pry](https://github.com/pry/pry)**: Powerful alternative to the standard IRB shell.
* **[pry-doc](https://github.com/pry/pry-doc)**: Provides documentation browsing capabilities within Pry.
* **[rake](https://github.com/ruby/rake)**: Ruby build program with capabilities similar to Make.
* **[ratelimit](https://github.com/ejfinneran/ratelimit)**: Generic rate limiter implementation.
~~**[sad_panda](https://github.com/mattThousand/sad_panda)**: Sentiment analysis library.~~
* **[sinatra](https://github.com/sinatra/sinatra)**: DSL for quickly creating web applications in Ruby.
* **timeout**: Auto-terminate potentially long-running operations. (Built-in)
* **[tool_tailor](https://github.com/kieranklaassen/tool_tailor)**: A Gem to convert methods to openai JSON schemas for use with tools.

### TTY toolkit

* **[tty-box](https://github.com/piotrmurach/tty-box)**: Draw boxes, frames, and borders in the terminal.
* **[tty-command](https://github.com/piotrmurach/tty-command)**: Execute external commands with enhanced feedback.
* **[tty-config](https://github.com/piotrmurach/tty-config)**: Define and manage application configuration.
* **[tty-editor](https://github.com/piotrmurach/tty-editor)**: Opens a file or text in the user's preferred editor.
* **[tty-file](https://github.com/piotrmurach/tty-file)**: File manipulation utility methods.
* **[tty-font](https://github.com/piotrmurach/tty-font)**: Write text in large stylized characters using a variety of terminal friendly fonts.
* **[tty-link](https://github.com/piotrmurach/tty-link)**: Detects whether the terminal supports hyperlinks and creates them ready for display in the console.
* **[tty-logger](https://github.com/piotrmurach/tty-logger)**: A readable, structured and beautiful logging for the terminal.
* **[tty-markdown](https://github.com/piotrmurach/tty-markdown)**: Convert Markdown documents or strings into terminal-friendly output.
* **[tty-option](https://github.com/piotrmurach/tty-option)**: A declarative command-line parser.
* **[tty-prompt](https://github.com/piotrmurach/tty-prompt)**: Interactive command-line prompting.
* **[tty-screen](https://github.com/piotrmurach/tty-screen)**: Get terminal screen size and color depth.
* **[tty-spinner](https://github.com/piotrmurach/tty-spinner)**: A terminal spinner for tasks that have non-deterministic time frame.
* **[tty-tree](https://github.com/piotrmurach/tty-tree)**: Print directory or structured data in a tree like format.
* **[tty-table](https://github.com/piotrmurach/tty-table)**: Draw structured tables in the terminal.

## Build Instructions

To build the Docker image, navigate to the root directory of this repository and run:

```bash
docker build -t your-image-name:latest -f nlp/Dockerfile .
```

*(Replace `your-image-name:latest` with your desired image name and tag)*

## Usage

Once built, you can run this image as part of a Jupyter environment (e.g., using Docker Compose with JupyterHub or JupyterLab). The IRuby kernel should be automatically available when creating new notebooks.

## Patches

* **`respond_to_missing.patch`**: This patch is applied to the `ruby-spacy` gem during the build process to fix compatibility issues with newer versions of the `pycall` gem regarding the `respond_to_missing?` method signature.
