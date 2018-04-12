R to D3 rendering tools
================

# r2d3: R interface to D3 visualizations

<img src="tools/README/r2d3-hex.png" width=180 align="right" style="margin-right: 20px;"/>

The **r2d3** package provides a suite of tools for using [D3
visualizations](https://d3js.org/) with R, including:

  - Translating R data frames into D3 friendly data structures

  - Rendering D3 scripts within the RStudio Viewer and [R
    Notebooks](https://rmarkdown.rstudio.com/r_notebooks.html)

  - Incorporating D3 scripts into [R
    Markdown](https://rmarkdown.rstudio.com/) reports, presentations,
    and dashboards

  - Creating interacive D3 applcations with
    [Shiny](https://shiny.rstudio.com/)

  - Publishing D3 based [htmlwidgets](http://www.htmlwidgets.org) in R
    packages.

With **r2d3**, you can bind data from R to D3 visualizations like the
ones found on the [D3 Gallery](https://github.com/d3/d3/wiki/Gallery),
[Blocks](https://bl.ocks.org/), or [VIDA](https://vida.io/explore). D3
visualizations you create work just like R plots within RStudio, R
Markdown documents, and Shiny applications.

## Installation

First, install the package from GitHub as follows:

``` r
devtools::install_github("rstudio/r2d3")
```

Next, install the [daily build](https://dailies.rstudio.com) of RStudio
(you need this version of RStudio to take advantage of various
integrated tools for authoring D3 scripts with r2d3):

[![](images/daily_build.png)](https://dailies.rstudio.com)

## D3 Scripts

To use **r2d3**, write a D3 script and then pass data to it using the
`rd23()` function. For example, here’s a simple D3 script that draws a
bar chart (“bars.js”):

``` js
var barHeight = Math.floor(height / data.length);

svg.selectAll('rect')
  .data(data)
  .enter().append('rect')
    .attr('width', function(d) { return d * width; })
    .attr('height', barHeight)
    .attr('y', function(d, i) { return i * barHeight; })
    .attr('fill', 'steelblue');
```

To render the script within R you call the `r2d3()` function:

``` r
library(r2d3)
r2d3(data=c(0.3, 0.6, 0.8, 0.95, 0.40, 0.20), script = "bars.js")
```

Which results in the following visualization:

![](images/bar_chart.png)

Note that data is provided to the script script using the `data`
argument. This data is then automatically made available to the D3
script. There are a number of other special variables available within
D3 scripts, including:

  - **data**: The R data converted to JavaScript.
  - **svg**: The svg element with the right dimensions.
  - **width**: The width of the svg.
  - **height**: The height of the svg.
  - **options**: Additional options provided from R.

## RStudio D3 Preview

The [daily build](https://dailies.rstudio.com) of RStudio includes
support for previewing D3 scripts as you write them. To try this out,
create a D3 script using the new file menu:

![](images/new_d3_script.png)

A simple template for a D3 script (the bars.js example shown above) is
provided by default. Note that the template includes a special comment
at the top of the script:

``` js
// !preview r2d3 data=c(0.3, 0.6, 0.8, 0.95, 0.40, 0.20)
```

This comment includes a `data` attribute, which enables RStudio to
provide a preview of the script’s output when the **Preview** command
(Ctrl+Shift+Center) is executed or when the document is saved:

![](images/rstudio_preview.png)

## R Markdown

You can include D3 visualizations in an R Markdown document or R
Notebook. For example:

<pre><code>---
output: html_document
---

&#96``{r}
library(r2d3)
r2d3(data=c(0.3, 0.6, 0.8, 0.95, 0.40, 0.20), script = "bars.js")
&#96``</code></pre>

![](images/bar_chart.png)

You can also include D3 visualization code inline using the `d3` chunk
type:

<pre><code>&#96``{r setup}
library(r2d3)
bars &lt;- c(10, 20, 30)
&#96``</code></pre>

<pre><code>&#96``{d3 data=bars, options=list(color = 'orange')}
svg.selectAll('rect')
  .data(data)
  .enter()
    .append('rect')
      .attr('width', function(d) { return d * 10; })
      .attr('height', '20px')
      .attr('y', function(d, i) { return i * 22; })
      .attr('fill', options.color);
&#96``</code></pre>

![](images/rmarkdown-1.png)

## Shiny

`r2d3` provides `renderD3()` and `d3Output()` to render under Shiny
apps:

``` r
library(shiny)
library(r2d3)

ui <- fluidPage(
  inputPanel(
    sliderInput("bar_max", label = "Max:",
      min = 10, max = 110, value = 10, step = 20)
  ),
  d3Output("d3")
)

server <- function(input, output) {
  output$d3 <- renderD3({
    r2d3(
      floor(runif(5, 5, input$bar_max)),
      system.file("baranims.js", package = "r2d3")
    )
  })
}

shinyApp(ui = ui, server = server)
```

<img src="images/baranim-1.gif" class="illustration" width=550/>

We can also render D3 in a Shiny document as follows:

<pre><code>---
runtime: shiny
output: html_document
---

&#96``{r setup}
library(r2d3)
&#96``

&#96``{r echo=FALSE}
inputPanel(
  sliderInput("bar_max", label = "Max:",
    min = 10, max = 110, value = 10, step = 20)
)

bars &lt;- reactive({
   floor(runif(5, 5, input$bar_max))
})
&#96``

&#96``{d3 data=bars}
var bars = svg.selectAll('rect')
  .data(data);
    
bars.enter()
  .append('rect')
    .attr('width', function(d) { return d * 10; })
    .attr('height', '20px')
    .attr('y', function(d, i) { return i * 22; })
    .attr('fill', 'steelblue');

bars.exit().remove();

bars.transition()
  .duration(250)
  .attr("width", function(d) { return d * 10; });
&#96``</code></pre>

<img src="images/baranim-1.gif" class="illustration" width=550/>
