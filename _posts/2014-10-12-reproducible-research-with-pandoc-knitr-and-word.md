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

Normally, I would write LaTeX documents with R code embeded and use [knitr] to weave everything together into a nice little (or big) pdf, with all of my references, links, figure numbers, tables, TOC.... But converting from LaTeX to `.docx` is a legendary boss that no one has yet defeated.

Thats where pandoc comes in. [Pandoc] is a great program that cross converts several document formats. I'm interested in the markdown to Word conversion. It took a lot of work and hair pulling, but I think I've finally figured it out. Using `knitr`, `rmarkdown`, and pandoc, I can finally get a working `.docx` with references, links, figure numbers, tables... almost everything I need to create a reproducible document.

# How to do it

## What's needed?

`R`, with the `knitr` and `rmarkdown` packages, pandoc, and Word.

I use the wonderful [Rstudio]() IDE which comes with document generation built in. [See here]() for a great guide into using markdown in R.

## The setup

`rmarkdown` files consist of markdown with `R` code embedded in chunks. `knitr` takes this file and weaves the normal text with code chunks to produce a plain `markdown` file that is then processed into whatever format you want using pandoc.

I won't get into a full runthrough of how to write a reproducible document using `rmarkdown` and `knitr` because there are already a lot of great resources for that. Instead, I will go over a problem that I have been trying to overcome for a while. 

One of the key requirements for me was a way to keep track of figure and table numbers. Coming from LaTeX, I was used to using `\label{}` to define a reference marker and `\ref{}` to refer to a marker. So I reated some functions in `R` to emulate these LaTeX commands.

```r
#' Functions to create and reference table and figure numbers.
#'
#' Takes a named list of markers and their number.
#' Adds \code{marker_name} to \code{marker_obj} with proper number.
#' The \code{marker_name} is parsed to determine the marker type, i.e. figure
#' or table based on the marker prefix.
#' 
#' Each marker prefix is given a seperate counter.
#' 
#' @param marker_obj Named \code{list} with caption marker and numbers.
#' @param marker_name \code{character} string with marker name. marker name
#'    should have the format <type>.<marker> where <type> is the figure or
#'    table type. For example \code{"fig.car_plot"} or
#'    \code{"tab.p_values"}.
#' @return Named \code{list} with new marker and number
label <- function(marker_obj, marker_name){
    if (grepl("\\.", marker_name)){
        marker_type <- strsplit(marker_name, "\\.")[[1]][1]
    } else {
        stop("marker name incorrect format. Use '<type>.<marker_name>'.")
	}

	if (marker_name %in% names(marker_obj)){
		warning("marker, '", marker_name, "' already present!")
	} else {
		previous_markers <- grep(glob2rx(paste0(marker_type, ".*")), 
								names(marker_obj))
		new_marker_number <- length(previous_markers) + 1
		marker_obj[[marker_name]] <- new_marker_number
	}

	return(marker_obj)
}

#' Reference a predefined marker and print the assigned number.
#' 
#' @param marker_obj Named \code{list} containing markers and their numbers.
#' @param marker \code{character} string of marker to reference.
#' @return \code{character} containing marker number or "??" for undefined
#'  reference.
ref <- function(marker_obj, marker){
	output_text <- marker_obj[[marker]]
	if (is.null(output_text)){
		output_text <- "??"
		warning("No marker called ", marker)
	}

	return(output_text)
}
```

## What can I do with this?

This system allows me to create code chunks and assign `markers` to these chunks. The basic idea is that there is a single named list of `markers` (the `marker_list`) and their assigned numbers. I can add new markers using the `label()` function and refer back to them using `ref()`.

`markers` have a two part syntax, `<type>.<marker_name>` where `<type>` is the type of element you want to refer to (i.e. a figure or table) and can be any arbitrary string. `<marker_name>` is the name of the element you want to mark. Using this system, the `marker_list` can contain a mixture of figures, tables, supplemental documents... anything as long as you keep the `<type>` consistent.

{% highlight r linenos %}
```{r, chunk1}
# Intialize the marker_list
marker_list <- list()

# Create a plot from www.statmethods.net
attach(mtcars)
plot(wt, mpg, main = "Scatterplot Example",
     xlab = "Car wight", ylab = "Miles Per Gallon",
     ph = 19)

# Add the first marker
marker_list <- label(marker_list, "fig.scatter_plot")
```
{% endhighlight %}

Later on in the document, I can refer back to the mark using `label()`. Either in a code block or even inline.

    For example, as seen in Figure `r label(marker_list, "fig.scatter_plot")`, mileage is negatively correlated with car weight.

Which will produce the following output:

> For example, as seen in Figure 1, mileage is negatively correlated with car weight.


A key feature of this system is that I can create markers and references to supplemental figures that are not normally included in a document like a journal submission.

{% highlight r linenos %}
```{r, chunk2}
marker_list <- label(marker_list, "sup_fig.boring_supplemental_figure")
makrer_list <- label(marker_list, "sup_fig.a_not_so_boring_figure")
```
{% endhighlight %}

I can then refer to these `markers` in my main text like before. 

    We did some routine but necessary experiments (Figure S`r ref(marker_list, "sup_fig.boring_supplemental_figure")`. We also found some interesting things that didn't quite make the cut (Figure S`r ref(maker_list, "sup_fig.a_not_so_boring_figure")`.

## Whats the catch?

The major things to keep in mind is that you must define a `marker` before your refer to it.

More examples to come.

[Pandoc]: http://johnmacfarlane.net/pandoc/
[knitr]: http://yihui.name/knitr/
[Rstudio]: http://www.rstudio.com/
