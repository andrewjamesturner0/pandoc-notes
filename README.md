# PANDOC NOTES


# 1. Pandoc basics

Use pandoc to convert markdown files into a variety of formats, (e.g. html, pdf,
doc):

    pandoc Input.md -o Output.html

Pandoc is more powerful than this. It can convert to and from many different
formats, with a large range of extra options. These notes cover converting from
markdown to html, pdf, and doc, as well as how to include citations and
bibliographies.  Essentially, everything you need to be able to write a paper.

For more information about writing markdown for pandoc, see:
<http://pandoc.org/demo/example9/pandocs-markdown.html>

For much more complete information, see:

* <http://pandoc.org/README.html>
* <http://pandoc.org/demos.html>

#### Generate html

    pandoc Notes.md -o Notes.html

#### Generate PDFs

    pandoc Notes.md --latex-engine=pdflatex -o Notes.pdf

#### Generate office documents

    pandoc Notes.md -o Notes.{odt,doc,docx}


# 2. Extra options

#### Common pandoc flags

`--standalone` creates a standalone file by adding an appropriate header
and footer from pandoc's built in templates.

`-smart` intelligently interprets quotes, brackets, em dashes etc.

`--toc` adds a table of contents
 
`--filter pandoc-citproc` processes citations (see below).

`--template` allows you to supply a custom template file.  To view the built in
templates:

    pandoc -D $DOCTYPE

Or see: <https://github.com/jgm/pandoc-templates>.

Custom templates should be placed in a `templates` directory in the user data
directory (default: `$HOME/.pandoc`) i.e. `~/.pandoc/templates/`. These can then
be referred to simply by filename, without having to specify the full path the
to file.

`--reference-doc template.docx`  similarly allows a template to be specified
for office documents.

`--css` allows you to supply a css file for formatting html output. This will
add a link to the css file in the html. Alternatively, if you wish to create a
fully standalone html document (i.e. with the css embedded in the document) then
use the `-H` option instead. Note however that the css file you pass to the `-H`
option must be enclosed between `<style> </style>` tags.

#### Variables

Further document options can be specified by using the `--variable` or `-V`
flag. For example:

    pandoc Input.md \
      --variable title="Title of Document" \
      --variable geometry:margin=2cm \
      --latex-engine=pdflatex \
      -o Output.pdf

##### What variables are available to be set?

Any variables used by the document template. For example, pandoc's html template
includes the `$css$` variable, and pandoc's latex template contains the
`$fontfamily$` variable.

For a complete list of what variables are available for each type of document,
see: <http://pandoc.org/README.html#templates>

#### YAML header

Variables can also be included within a YAML header. For example:

    ---
    title: Title of Document
    author: Andrew
    date: 2015-01-01
    abstract: Abstract goes here
              and continues.
    geometry: margin=2cm
    fontfamily: libertine
    bibliography: ref.bib
    ---

This header can be included in the markdown file itself, or in a separate file.
If placed in a separate file, the final document is generated by:

    pandoc Input.md metadata.yaml -o Output.html


# 3. Citations

If you want to include citations in `Notes.md`, then you can reference a .bib
file.

#### Markdown formatting

In text citations are inserted inside square brackets using the `@` symbol
followed by the correct identifier from the .bib file.  Multiple citations are
separated by semicolons and can each be given a prefix, location and a suffix.
To suppress the author, simply use `-@`. For instance:

    Early definitions of Evidence-Based Medicine [see, for example:
    @Sackett_1996]

    Sackett et al [-@Sackett_1996] defined EBM as "the conscientious...
    
    The first appearance of the term Evidence-Based Medicine occurred in an
    article in JAMA by the EBM Working Group [-@EBM_working_group_1992, p. 2420]

#### Generating the references

To generate the references with pandoc the bibliography file and the citation
style file can be pre-specified in the document's yaml header. Or alternatively,
using the following flags in the pandoc command:

    pandoc Notes.md
      --filter pandoc-citeproc \
      --bibliography=/path/to/ref.bib \
      --csl=vancouver.csl \
      -o Notes.html


# 4. Further formatting

## Latex

$\latex$ formatting can be inserted into markdown documents by enclosing it with
`$`. For instance:

     $\forall a \in D: X(a) \quad$

No extra option or filter needs to be passed to the pandoc command to interpret
this.

## Images

Images can be inserted into markdown documents in the standard way. For example:

    ![Alt-text](/path/to/image.png "Title")

# 5. Example Makefile

    VERSION = 0.0.1
    FILENAME = notes
    PANDOCFLAGS = --standalone \
                  -smart \
                  --variable version=$(VERSION) \
                  --filter=pandoc-citeproc \
                  --bibliography=$(FILENAME).bib \
                  --csl=vancouver.csl
    PANDOC_PDF_FLAGS = --variable fontfamily=libertine \
                       --variable geometry:margin=3cm \
                       --template=/path/to/template
    PANDOC_HTML_FLAGS = --include-in-header=style.css

    all: $(FILENAME)_v$(VERSION).pdf \
         $(FILENAME)_v$(VERSION).html \
         $(FILENAME)_v$(VERSION).docx

    pdf: $(FILENAME)_v$(VERSION).pdf

    html: $(FILENAME)_v$(VERSION).html

    docx: $(FILENAME)_v$(VERSION).docx

    %.pdf: $(FILENAME).md $(FILENAME).bib
        pandoc $< $(PANDOC_FLAGS) $(PANDOC_PDF_FLAGS) -o $@

    %.html: $(FILENAME).md $(FILENAME).bib
        pandoc $< $(PANDOC_FLAGS) $(PANDOC_HTML_FLAGS) -o $@

    %.docx: $(FILENAME).md $(FILENAME).bib
        pandoc $< $(PANDOC_FLAGS) -o $@

    clean: cleanhtml cleanpdf cleandocx

    cleanhtml:
        rm $(FILENAME)_v$(VERSION).html

    cleanpdf:
        rm $(FILENAME)_v$(VERSION).pdf

    cleandocx:
        rm $(FILENAME)_v$(VERSION).docx
