# NLP Image Configuration

**Parent:** `./AGENTS.md`

## OVERVIEW
NLP-focused Ruby image built on `base`. Adds Ruby 3.3.8, IRuby kernel, and 100+ gems.

## FILES
| File | Purpose |
|------|---------|
| `Containerfile` | NLP image definition (182 lines) |
| `Gemfile` | Ruby gems list |
| `Gemfile.lock` | Pinned versions (required for reproducible builds) |
| `respond_to_missing.patch` | Fix for ruby-spacy/pycall compat |

## KEY CONFIG
- Ruby: 3.3.8 (pinned)
- Multi-stage build: builder stage → runtime stage
- Native gems: compiled during build (nmatrix, rbplotly, ferret)
- Patch: `respond_to_missing.patch` applied to ruby-spacy

## ADDING GEMS
```bash
# Edit nlp/Gemfile
echo "gem 'new-gem'" >> nlp/Gemfile

# Update lockfile (in container or locally with matching Ruby)
bundle install

# Rebuild
rake build/nlp
```

## GEM CATEGORIES
- **NLP**: ruby-spacy, tokenizers, lingua, wordnet, tf-idf-similarity
- **LLM**: langchainrb, ruby-openai, groq, hugging-face, chroma-db
- **Data**: daru, sequel, ohm, redis, pgvector
- **TTY**: tty-prompt, tty-table, tty-spinner, tty-screen

## NOTES
- Always update `Gemfile.lock` after gem changes
- Native gems may need build deps added to Containerfile
- Test locally: `podman build -t test-nlp -f nlp/Containerfile .`
