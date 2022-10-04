Untitled
================
Chuanni He
2022-10-03

``` r
rmarkdown::render("test.md","github_document",output_file = "README.md")
```

    ## "C:/Program Files/RStudio/bin/quarto/bin/tools/pandoc" +RTS -K512m -RTS test.md --to gfm+tex_math_dollars-yaml_metadata_block --from markdown+autolink_bare_uris+tex_math_single_backslash --output README.md --template "C:\Users\hechu\Documents\R\win-library\4.1\rmarkdown\rmarkdown\templates\github_document\resources\default.md" 
    ## "C:/Program Files/RStudio/bin/quarto/bin/tools/pandoc" +RTS -K512m -RTS README.md --to html4 --from gfm+tex_math_dollars --output README.html --self-contained --highlight-style pygments --template "C:\Users\hechu\Documents\R\win-library\4.1\rmarkdown\rmarkdown\templates\github_document\resources\preview.html" --variable "github-markdown-css:C:\Users\hechu\Documents\R\win-library\4.1\rmarkdown\rmarkdown\templates\github_document\resources\github.css" --metadata pagetitle=PREVIEW --mathjax

    ## 
    ## Preview created: README.html

    ## 
    ## Output created: README.md

## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax
for authoring HTML, PDF, and MS Word documents. For more details on
using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that
includes both content as well as the output of any embedded R code
chunks within the document. You can embed an R code chunk like this:

``` r
summary(cars)
```

    ##      speed           dist       
    ##  Min.   : 4.0   Min.   :  2.00  
    ##  1st Qu.:12.0   1st Qu.: 26.00  
    ##  Median :15.0   Median : 36.00  
    ##  Mean   :15.4   Mean   : 42.98  
    ##  3rd Qu.:19.0   3rd Qu.: 56.00  
    ##  Max.   :25.0   Max.   :120.00

## Including Plots

You can also embed plots, for example:

![](README_files/figure-gfm/pressure-1.png)<!-- -->

Note that the `echo = FALSE` parameter was added to the code chunk to
prevent printing of the R code that generated the plot.
