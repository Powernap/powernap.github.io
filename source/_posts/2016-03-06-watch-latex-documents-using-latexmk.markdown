---
layout: post
title: "Watch LaTeX Documents using latexmk"
date: 2016-03-06 16:54:47 +0100
comments: true
categories: 
---

**THIS ARTICLE IS UNDER CONSTRUCTION**

**tl,dr:** The `Watch Document` functionality of Textmate automatically compiles Tex documents when saved. This post describes how to achieve this behavior with any editor. It works for Unix-based systems, such as OSX or Linux.

## Introduction: Switch from Textmate

I've been using Textmate to write and compile my LaTeX Documents since 2007. [This Blog post](http://jann.is/daily/archives/756-LaTex-Live-PDF-preview-with-TextMate-and-PDFView.html) by Jannis Hermanns basically defined my workflow since then. The neat `Watch Document` feature compiles the document and looks for changes. It recompiles the document as soon as it is saved.

{% img center /media/2016-03-06-watch-latex-documents-using-latexmk/textmate_watch_document.gif %}

This also works with LaTeX projects consisting of multiple files. Simply put the watch on the project master file and it will also look for changes in the associated files. This even extends to included images.

If you even use a proper PDF viewer that supports [SyncTex](http://itexmac.sourceforge.net/SyncTeX.html), such as [Skim](http://skim-app.sourceforge.net/), you can even jump to the current position of the cursor from the Tex document to the PDF and vice versa.

Once I got used to this convenience, compiling by hand felt like a chore. Unfortunately, using Textmate began to feel likewise. Since I do all my coding in Sublime Text, I [looked for a similar workflow for that editor](http://tex.stackexchange.com/questions/276973/pendant-to-textmate-watch-document-function-in-sublime-text-latextools).
Turns out, it doesn't matter which editor you use, you can simply incorporate [`latexmk`](https://www.ctan.org/pkg/latexmk/) to achieve this functionality. Event better, `latexmk` is shipped per default with all large LaTeX distributions, so you should have it installed already.

## Compile LaTeX Projects using Latexmk

Consider the following LaTeX document `article.tex`.
{% codeblock article.tex lang:latex %}
\documentclass[12pt]{article}
\begin{document}
Hi!
\end{document}
{% endcodeblock %}

Using `latexmk`, you can compile this into a PDF file by entering the following command.

{% codeblock lang:bash %}
latexmk -pdf article.tex
{% endcodeblock %}

In order to watch for changes, you can simply add the `-pvc` option, which stands for "preview continuously":

{% codeblock lang:bash %}
latexmk -pdf -pvc article.tex
{% endcodeblock %}

As you can see, the command will not terminate back into the prompt, as `latexmk` watches for updated files (*targets*) associated with the previewed file. This is already close to the desired behavior. I suggest adding the following options:

- `-bibtex` to compile associated BibTeX files,
- `-f` to continue compiling albeit found errors,
- `-quiet` to make `latexmk` less verbose and show only important messages,
- `-use-make` tries to resolve missing files after the `pdflatex` call using custom dependencies, and
- `-pdflatex="pdflatex -synctex=1 -interaction=nonstopmode"`  to add options to the pdflatex command, most important the `synctex` command to allow jumping between the source PDF and the text editor.

For a precise description of each command, refer to the [latexmk readme](http://ftp.fernuni-hagen.de/ftp-dir/pub/mirrors/www.ctan.org/support/latexmk/latexmk.pdf). All options together yield the following command:

{% codeblock lang:bash %}
latexmk -quiet -bibtex -pvc -f -pdf -pdflatex="pdflatex -synctex=1 -interaction=nonstopmode" article.tex
```
{% endcodeblock %}

This is exactly the behavior I was looking for. And as a big plus, it is independent of your editor choice. Using it with [Sublime Text](https://www.sublimetext.com/) and the [LaTeXTools Plugin](https://www.sublimetext.com/) along with a Skim even allows to jump between source and compiled PDF using the enabled `synctex` feature.

*insert gif of thesis with jumping in between files*

## Use the Power of `make` with `latexmk`

Utilizing this knowledge with `makefiles` is easy. I know that for most people, makefiles are often scary due to their seemingly obscure syntax, but actually they are easy to set up and will spare you *a lot* of work. If you want to get a quick introduction to `make`, I strongly recommend Mike Bostocks [beautiful blog post](https://bost.ocks.org/mike/make/) on it. The commands applied here, however, are basic and should be self explanatory.

The `makefile` I propose is an adapted version from [this stackoverflow post](https://tex.stackexchange.com/questions/40738/how-to-properly-make-a-latex-project).

This comes down to personal preference, I built the makefile to work like this:

- `make` compiles the document, but does *not* watch for changes,
- `make preview` compiles the document and watches the document for changes using the `latexmk -pvc` command, and
- `make clean` removes all files produced using the LaTeX compilation process including the resulting PDF.

Here is the `makefile`:

{% codeblock make lang:make %}
# File adapted from this stackoverflow question: https://tex.stackexchange.com/questions/40738/how-to-properly-make-a-latex-project

# The first rule in a Makefile is the one executed by default ("make"). It
# should always be the "all" rule, so that "make" and "make all" are identical.
all: Interactive\ Visual\ Analysis\ of\ Population\ Study\ Data.pdf

# MAIN LATEXMK RULE

# -pdf tells latexmk to generate PDF directly (instead of DVI).
# -pdflatex="" tells latexmk to call a specific backend with specific options.
# -use-make tells latexmk to call make for generating missing files.
# -interaction=nonstopmode keeps the pdflatex backend from stopping at a
# missing file reference and interactively asking you for an alternative.
# -synctex=1 is required to jump between the source PDF and the text editor
# -pvc causes latexmk to watch the file directory for changes. Removing this command causes a single build
# -quiet suppresses most status messages (https://tex.stackexchange.com/questions/40783/can-i-make-latexmk-quieter)
Interactive\ Visual\ Analysis\ of\ Population\ Study\ Data.pdf: Interactive\ Visual\ Analysis\ of\ Population\ Study\ Data.tex
    latexmk -quiet -bibtex $(PREVIEW) -f -pdf -pdflatex="pdflatex -synctex=1 -interaction=nonstopmode" -use-make Interactive\ Visual\ Analysis\ of\ Population\ Study\ Data.tex

# The .PHONY rule keeps make from doing something with a file named preview or clean.
.PHONY: preview
# Set the preview variable to -pvc to switch latexmk into the preview continuously mode
preview: PREVIEW=-pvc
preview: Interactive\ Visual\ Analysis\ of\ Population\ Study\ Data.pdf

.PHONY: clean
# Adding the -bibtex also removes the .bbl files (http://tex.stackexchange.com/a/83384/79184)
clean:
    latexmk -CA -bibtex
{% endcodeblock %}

The `all` task is straightforward to create the target file "article.pdf", which is created using `latexmk` later on. You'll notice the `$(PREVIEW)` where the `-pvc` option should be. `$(PREVIEW)` refers to the variable `PREVIEW`, which is empty for all calls except for `make preview`, where `PREVIEW` is set to `-pvc`. The `make clean` command removes all latex output files including the BibTeX files as well as the PDF.

## Closing Remarks

*Note: If you ran `make` and now want to run `make preview`, it will most likely say "Nothing to be done for `preview`", since it sees that the target pdf is up to date. The `-B` option forces make to create the targets, so `make -B preview` will work.*
