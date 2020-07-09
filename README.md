# A JEO-Batch tutorial (for first time users)

The tutorial is written in markdown which is then converted to HTML via mkdocs

## Generate the static website

```
python3 -mvenv .venv
source .venv/bin/activate
poetry istall
mkdocs build
```

The `mkdocs build` command will generate the `site` directory with the contents of the website.

To test that it works:

```
cd site
python3 -m http.server 9999
```

and visit http://localhost:9999

## Deploy

Just copy the contents of `site` to an EOS path that is being served to the internet

## Developing

```
python3 -mvenv .venv
source .venv/bin/activate
poetry istall
mkdocs serve
```

This will launch a development server that re-renders the website every time you make a change
in the markdown files.
