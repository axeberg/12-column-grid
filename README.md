# 12-column-grid
A super simple CSS column grid that can get you started with most layouts.

## How grid is calculated

### Single column width
`ekb = (100 – (m * (mk – 1))) / mk`

* **ekb** = single column width
* **m** = margin (1.6%)
* **mk** = maximal number of columns (12)

### Column span widths

`kb = (ekb * ks) + (m * (ks – 1))`

* **kb**  = column width
* **ekb** = single column width (6.86666666667%)
* **ks**  = column span (1-12)
* **m**   = margin (1.6%)

### Examples

Structure your grids like so to get the desired effect, and remember that each `.row` needs to have enough columns to always reach the maximum number of columns (i.e. 12).

```HTML
<section  class="row">
    <article class="column column-4"></article>
    <article class="column column-4"></article>
    <article class="column column-4"></article>
</section>

<section  class="row">
    <article class="column column-2"></article>
    <article class="column column-4"></article>
    <article class="column column-4"></article>
    <article class="column column-2"></article>
</section>
```
