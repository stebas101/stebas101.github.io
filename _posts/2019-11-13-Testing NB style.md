---
layout: nbpost
title: Notebook style MD test
date: 2019-11-13 21:50:28 +0000
categories: jekyll update
---
### A visual example

Let us put this concept to work and create some practical examples using Python, [pandas], and [Matplotlib]. I am assuming that you have at least some basic Python knowledge and know what a DataFrame and a Series object are. If this is not the case, you can find a gentle introduction [here]. I also suggest that you use a [Jupyter] notebook to follow along with the following code. Here you can find a handy [Jupyter tutorial]. However, you always execute the code in your favorite way, through an IDE or the interactive prompt. We start by loading the required libraries and checking their versions:

[here]: https://www.dataquest.io/course/python-for-data-science-fundamentals/
[pandas]: https://pandas.pydata.org/
[Matplotlib]: https://matplotlib.org/
[Jupyter]: https://jupyter.org/
[Jupyter tutorial]: https://www.dataquest.io/blog/jupyter-notebook-tutorial/


  <div class="input" markdown="0">
  <div class="prompt input_prompt" markdown="0">
  In [1]:
  </div>
  <div class="input_area" markdown="1">
  
```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib as mpl
import sys

print('Python version: ' + sys.version)
print('pandas version: ' + pd.__version__)
print('matplotlib version: ' + mpl.__version__)
```

  </div>
  </div>
  
  {:.output_stream}</p>

<pre><code>  Python version: 3.7.4 (default, Aug 13 2019, 15:17:50) 
[Clang 4.0.1 (tags/RELEASE_401/final)]
pandas version: 0.25.1
matplotlib version: 3.1.1
</code></pre>
<p>
In **Jupyter**, it's always a good idea to display the chart within the notebook:


  <div class="input" markdown="0">
  <div class="prompt input_prompt" markdown="0">
  In [2]:
  </div>
  <div class="input_area" markdown="1">
  
```python
%matplotlib inline
```

  </div>
  </div>
  