---
layout: post
author: Jonathan Chung
title: Reproducible research with pandoc, knitr, and word
header-img: img/grey_block.jpg
---

# Whats the big idea?

The problem: My PI is a die hard Word fan. I love plain text. Can things ever
work out between us?

# If I can do it, so can you!

I really enjoy using R and LaTeX for creating reports, papers, presentations, and (we will see how this goes) posters. The problem is that I am the only one in my lab who uses LaTeX and everyone else is a die hard Word fan. Since I'm the one under pressure, I need to adapt to survie.

Normally, I would write LaTeX documents with R code embeded and use `Knitr` to weave everything together into a nice little (or big) pdf, with all of my references, links, figure numbers, tables, TOC.... But converting from LaTeX to `.docx` is a legendary boss that no one has yet defeated.

Thats where pandoc comes in. (Pandoc)[http://johnmacfarlane.net/pandoc/] is a great program that cross converts several document formats. I'm interested in the markdown to Word conversion. It took a lot of work and hair pulling, but I think I've finally figured it out. Using `Knitr`, `rmarkdown`, and pandoc, I can finally get a working `.docx` with references, links, figure numbers, tables... almost everything I need to create a reproducible document.

# How to do it

## What's needed?

`R`, with the `knitr` and `rmarkdown` packages, pandoc, and Word.

I use the wonderful (Rstudio)[http://www.rstudio.com/] IDE which comes with document generation built in. (See here)[http://rmarkdown.rstudio.com/] for a great guide into using markdown in R.
