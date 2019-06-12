---
layout: post
author: tin

title:      Quick Service to copy code format and reserve syntax highlight
subtitle:  An enhanced robust routine to copy-paste code block via macOS System Services, but the core is using the ubiquitous Python and its syntax highlighter Pygment

categories: [Tutorial, macOS, Productivity, Command Line Tools]
tags: [Syntax Highlight, Programming, Services, macOS, Python, Pygment]

gif: /assets/blogs/2018-12-23-Quick-Service-to-copy-code-format-and-reserve-syntax-highlight/preview.gif

featured: true
hidden: false
portfolio: false
---

* TOC
{:toc}

# Overview

This short tutorial aims to simplify the process of **reserve the syntax highlight** property of any code blocks and paste them into other application such as _Microsoft Word_, _Keynote_, etc. conveniently with **System Service routines** and couple of clicks.


# Prerequisite
  
1\.  Install Python-powered Syntax Highlighter *Pygments* via Terminal app

```python
# Python 2.7
pip install pygments

# Python 3.x
pip3 install pygments
```

* `pip/ pip3`: Python Package Managers

**Side note:** _Python 2.7_ end of life is on January $$1^{st}$$, 2020 and *Python 3.x* is **backward-incompatible**, i.e. you cannot keep using obsolete modules. It is highly recommended to install _Python 3.x_ instead.

2\. (Optional) In case you don't have Python installed yet. Download Python 3 Latest Release [here][1]. Or follow the below script to install via command line.

```bash
#! /bin/bash

# One-liner install homebrew using default Ruby shipped with your macOS Version
mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew

# Then update the default PATH to use homebrew with your Shell
# Or other config file, E.g. '.bashrc'
test -f ~/.zshrc && echo "export PATH="/usr/local/bin:/usr/local/sbin:$PATH"" >> ~/.zshrc && source ~/.zshrc

# Verify successful installation
brew --version

# Install Python via Brew
brew install python

# Or specify a version
brew install python@2

# Lastly, install the Syntax Highlighter as in step 1
pip3 install pygments
```
**Side note:** Have a look into [similar tutorials]({% post_url 2019-01-18-Define-Compiling-workflow-for-Arduino-sketch-with-Makefile %}) for Python installation and settings.

# About Pygments

> Python is an intepreted, high-level and without a doubt, the most common general-purpose programming language as of 2018.

And Pygments is simply a syntax highlighter suitable for _code hosting_, _forums_, _wikis_ or other applications that need to **prettify source code**.

From the Terminal, use command `pygmentize` to initiate and provide _input flags_ to process your code block. If you have experience with Python, head to the next section on how to use Pygment in Python Environment.

```python
touch test.py && echo 'print("Success") # Just a comment' > test.py

pygmentize -f html -O style=friendly,full=True -o result.html test.py 

open result.html
```

* `touch`: Create a file named _test.py_ 
* `echo`: Insert '{string}' of text to designated file \> {output}. This would also create file (if not exist) when invoked
* `open`: Open input file

Pygment flags explanation. Type `pygmentize --help` for a detailed manual.
1. `-f`: Format flag
2. `-O`: Option flags, comma separated values
3. `-o`: Output file (_stdout_)

Lastly, the _test.py_ file was given as input (_stdin_) for the command.

![][image-1]
Grab another example [array.py]({{ site.baseurl }}{% link /assets/blogs/2018-12-23-Quick-Service-to-copy-code-format-and-reserve-syntax-highlight/array.py %}) and try it yourself.
{: .caption}

# Automate with Services

Now that you have an initial idea of how _Pygments_ works, let's try to automate copy code format with _a couple of clicks_.

![][image-2]
Services provide convenient routines visibly on a right click. Screen captured on **macOS Mojave 10.14**
{: .caption}

Essentially, *Services* are _pre-defined routines_ that aim to automate tasks using **AppleScript**. Open _Automator_ app via _Spotlight_ using  `⌘-Space Bar` keystroke.

![][image-3]
Step 1 – Select Quick Action workflow
{: .caption}

Simply put, the flow is to _Copy the selected text_ (Step 1) $$\rightarrow$$ Prompt for highlighting language (Step 2) $$\rightarrow$$ Set variable value (Step 3) $$\rightarrow$$ Run command to 

![][image-4]
Step 2 – Select Input as Text. On the left panel, search & drag `Copy to Clipboard` action
{: .caption}

**Note:** The transition between Step 2 and 3. The _Input(1) and Copy(2)_ actions are inter-connected (Above, middle), whereas the _Copy(2) and Ask for Text (3)_ actions below are not connected.

![][image-5]
Step 3 – Similarly, search & drag `Set Value of Variable` action, then select New Variable. This will serve as input for the final command
{: .caption}

Ultimately, the whole workflow should look similar to this.

![][image-6]
Step 4 – Finally, Add a `Run Shell Script` action, select your Shell and make sure it _Pass input_ `as arguments`
{: .caption}

* `pbpaste` & `pbcopy`: Command lines to provide text from Clipboard and add text to Clipboard. This setup serves our purpose of copy code blocks and later able to paste a _highlighted version_ of it. 

New Pygments flags are added as well as the full path in order to ensure all cases.
1. `-g`: Automatically detect Lexer from the content, i.e. the programming language.
2. `-l`: Manually input Lexer, i.e. the user-input in Step 3

**Note:** The `-f` formatter flag now utilizes rtf (Rich Text Format) instead.

```bash
export LC_CTYPE=UTF-8; 
pbpaste | /usr/local/bin/pygmentize -g -O style=colorful -f rtf -l  $1 | pbcopy
```
The command here has been revised and is a little bit different than the 'raw' one from previous section. Most importantly, the _export_ command ensures that the copied text is in **UTF-8** encode in order for this to work.

# Command Line variant

As a bonus, if you're more comfortable working with CLI, there is a quick function for you. Usage is simply as passing any file into your CLI, for instance `highlight array.py` from the previous example. 

```bash
echo -e 'function highlight() {/usr/local/bin/pygmentize -g -O style=colorful -f rtf $1 | pbcopy}' >> ~/.zshrc
source ~/.zshrc
```
Again replace the config file based on your Shell, the default is usually bashrc
{: .caption}

⚠️ **Important:** Use of `echo` command with `>>` will append to file. **Do not** use `>`, this will replace all your text.<span class="spoiler"> I learned that the hard way !</span>

The result will be copy onto your _Clipboard_ and is ready to paste in any other application that support _RTF_.

**Side note:** With the `-g` flag enabled and file extension, there is no need to provide a _Lexer_ as Pygments will automatically detect the language and highlight accordingly.

# Explore highlight style

To find more about available _color scheme_, simply type `python3` to access Python Environment or use _Python3 IDLE_ if you use a Graphical UI application and the following code

```python
from pygments.styles import get_all_styles
styles = list(get_all_styles())
print(styles)
```

All default themes have been composed below _at your conveniences_.

{% capture text-capture %}

<html><head><title>Pygments style gallery!</title>
</head><body>
<h3>monokai</h3>
<style type="text/css">.monokai .hll { background-color: #49483e }
.monokai  { background: #272822; color: #f8f8f2 }
.monokai .c { color: #75715e } /* Comment */
.monokai .err { color: #960050; background-color: #1e0010 } /* Error */
.monokai .k { color: #66d9ef } /* Keyword */
.monokai .l { color: #ae81ff } /* Literal */
.monokai .n { color: #f8f8f2 } /* Name */
.monokai .o { color: #f92672 } /* Operator */
.monokai .p { color: #f8f8f2 } /* Punctuation */
.monokai .cm { color: #75715e } /* Comment.Multiline */
.monokai .cp { color: #75715e } /* Comment.Preproc */
.monokai .c1 { color: #75715e } /* Comment.Single */
.monokai .cs { color: #75715e } /* Comment.Special */
.monokai .ge { font-style: italic } /* Generic.Emph */
.monokai .gs { font-weight: bold } /* Generic.Strong */
.monokai .kc { color: #66d9ef } /* Keyword.Constant */
.monokai .kd { color: #66d9ef } /* Keyword.Declaration */
.monokai .kn { color: #f92672 } /* Keyword.Namespace */
.monokai .kp { color: #66d9ef } /* Keyword.Pseudo */
.monokai .kr { color: #66d9ef } /* Keyword.Reserved */
.monokai .kt { color: #66d9ef } /* Keyword.Type */
.monokai .ld { color: #e6db74 } /* Literal.Date */
.monokai .m { color: #ae81ff } /* Literal.Number */
.monokai .s { color: #e6db74 } /* Literal.String */
.monokai .na { color: #a6e22e } /* Name.Attribute */
.monokai .nb { color: #f8f8f2 } /* Name.Builtin */
.monokai .nc { color: #a6e22e } /* Name.Class */
.monokai .no { color: #66d9ef } /* Name.Constant */
.monokai .nd { color: #a6e22e } /* Name.Decorator */
.monokai .ni { color: #f8f8f2 } /* Name.Entity */
.monokai .ne { color: #a6e22e } /* Name.Exception */
.monokai .nf { color: #a6e22e } /* Name.Function */
.monokai .nl { color: #f8f8f2 } /* Name.Label */
.monokai .nn { color: #f8f8f2 } /* Name.Namespace */
.monokai .nx { color: #a6e22e } /* Name.Other */
.monokai .py { color: #f8f8f2 } /* Name.Property */
.monokai .nt { color: #f92672 } /* Name.Tag */
.monokai .nv { color: #f8f8f2 } /* Name.Variable */
.monokai .ow { color: #f92672 } /* Operator.Word */
.monokai .w { color: #f8f8f2 } /* Text.Whitespace */
.monokai .mf { color: #ae81ff } /* Literal.Number.Float */
.monokai .mh { color: #ae81ff } /* Literal.Number.Hex */
.monokai .mi { color: #ae81ff } /* Literal.Number.Integer */
.monokai .mo { color: #ae81ff } /* Literal.Number.Oct */
.monokai .sb { color: #e6db74 } /* Literal.String.Backtick */
.monokai .sc { color: #e6db74 } /* Literal.String.Char */
.monokai .sd { color: #e6db74 } /* Literal.String.Doc */
.monokai .s2 { color: #e6db74 } /* Literal.String.Double */
.monokai .se { color: #ae81ff } /* Literal.String.Escape */
.monokai .sh { color: #e6db74 } /* Literal.String.Heredoc */
.monokai .si { color: #e6db74 } /* Literal.String.Interpol */
.monokai .sx { color: #e6db74 } /* Literal.String.Other */
.monokai .sr { color: #e6db74 } /* Literal.String.Regex */
.monokai .s1 { color: #e6db74 } /* Literal.String.Single */
.monokai .ss { color: #e6db74 } /* Literal.String.Symbol */
.monokai .bp { color: #f8f8f2 } /* Name.Builtin.Pseudo */
.monokai .vc { color: #f8f8f2 } /* Name.Variable.Class */
.monokai .vg { color: #f8f8f2 } /* Name.Variable.Global */
.monokai .vi { color: #f8f8f2 } /* Name.Variable.Instance */
.monokai .il { color: #ae81ff } /* Literal.Number.Integer.Long */</style>
<div class="monokai"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>manni</h3>
<style type="text/css">.manni .hll { background-color: #ffffcc }
.manni  { background: #f0f3f3; }
.manni .c { color: #0099FF; font-style: italic } /* Comment */
.manni .err { color: #AA0000; background-color: #FFAAAA } /* Error */
.manni .k { color: #006699; font-weight: bold } /* Keyword */
.manni .o { color: #555555 } /* Operator */
.manni .cm { color: #0099FF; font-style: italic } /* Comment.Multiline */
.manni .cp { color: #009999 } /* Comment.Preproc */
.manni .c1 { color: #0099FF; font-style: italic } /* Comment.Single */
.manni .cs { color: #0099FF; font-weight: bold; font-style: italic } /* Comment.Special */
.manni .gd { background-color: #FFCCCC; border: 1px solid #CC0000 } /* Generic.Deleted */
.manni .ge { font-style: italic } /* Generic.Emph */
.manni .gr { color: #FF0000 } /* Generic.Error */
.manni .gh { color: #003300; font-weight: bold } /* Generic.Heading */
.manni .gi { background-color: #CCFFCC; border: 1px solid #00CC00 } /* Generic.Inserted */
.manni .go { color: #AAAAAA } /* Generic.Output */
.manni .gp { color: #000099; font-weight: bold } /* Generic.Prompt */
.manni .gs { font-weight: bold } /* Generic.Strong */
.manni .gu { color: #003300; font-weight: bold } /* Generic.Subheading */
.manni .gt { color: #99CC66 } /* Generic.Traceback */
.manni .kc { color: #006699; font-weight: bold } /* Keyword.Constant */
.manni .kd { color: #006699; font-weight: bold } /* Keyword.Declaration */
.manni .kn { color: #006699; font-weight: bold } /* Keyword.Namespace */
.manni .kp { color: #006699 } /* Keyword.Pseudo */
.manni .kr { color: #006699; font-weight: bold } /* Keyword.Reserved */
.manni .kt { color: #007788; font-weight: bold } /* Keyword.Type */
.manni .m { color: #FF6600 } /* Literal.Number */
.manni .s { color: #CC3300 } /* Literal.String */
.manni .na { color: #330099 } /* Name.Attribute */
.manni .nb { color: #336666 } /* Name.Builtin */
.manni .nc { color: #00AA88; font-weight: bold } /* Name.Class */
.manni .no { color: #336600 } /* Name.Constant */
.manni .nd { color: #9999FF } /* Name.Decorator */
.manni .ni { color: #999999; font-weight: bold } /* Name.Entity */
.manni .ne { color: #CC0000; font-weight: bold } /* Name.Exception */
.manni .nf { color: #CC00FF } /* Name.Function */
.manni .nl { color: #9999FF } /* Name.Label */
.manni .nn { color: #00CCFF; font-weight: bold } /* Name.Namespace */
.manni .nt { color: #330099; font-weight: bold } /* Name.Tag */
.manni .nv { color: #003333 } /* Name.Variable */
.manni .ow { color: #000000; font-weight: bold } /* Operator.Word */
.manni .w { color: #bbbbbb } /* Text.Whitespace */
.manni .mf { color: #FF6600 } /* Literal.Number.Float */
.manni .mh { color: #FF6600 } /* Literal.Number.Hex */
.manni .mi { color: #FF6600 } /* Literal.Number.Integer */
.manni .mo { color: #FF6600 } /* Literal.Number.Oct */
.manni .sb { color: #CC3300 } /* Literal.String.Backtick */
.manni .sc { color: #CC3300 } /* Literal.String.Char */
.manni .sd { color: #CC3300; font-style: italic } /* Literal.String.Doc */
.manni .s2 { color: #CC3300 } /* Literal.String.Double */
.manni .se { color: #CC3300; font-weight: bold } /* Literal.String.Escape */
.manni .sh { color: #CC3300 } /* Literal.String.Heredoc */
.manni .si { color: #AA0000 } /* Literal.String.Interpol */
.manni .sx { color: #CC3300 } /* Literal.String.Other */
.manni .sr { color: #33AAAA } /* Literal.String.Regex */
.manni .s1 { color: #CC3300 } /* Literal.String.Single */
.manni .ss { color: #FFCC33 } /* Literal.String.Symbol */
.manni .bp { color: #336666 } /* Name.Builtin.Pseudo */
.manni .vc { color: #003333 } /* Name.Variable.Class */
.manni .vg { color: #003333 } /* Name.Variable.Global */
.manni .vi { color: #003333 } /* Name.Variable.Instance */
.manni .il { color: #FF6600 } /* Literal.Number.Integer.Long */</style>
<div class="manni"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>rrt</h3>
<style type="text/css">.rrt .hll { background-color: #0000ff }
.rrt  { background: #000000; }
.rrt .c { color: #00ff00 } /* Comment */
.rrt .k { color: #ff0000 } /* Keyword */
.rrt .cm { color: #00ff00 } /* Comment.Multiline */
.rrt .cp { color: #e5e5e5 } /* Comment.Preproc */
.rrt .c1 { color: #00ff00 } /* Comment.Single */
.rrt .cs { color: #00ff00 } /* Comment.Special */
.rrt .kc { color: #ff0000 } /* Keyword.Constant */
.rrt .kd { color: #ff0000 } /* Keyword.Declaration */
.rrt .kn { color: #ff0000 } /* Keyword.Namespace */
.rrt .kp { color: #ff0000 } /* Keyword.Pseudo */
.rrt .kr { color: #ff0000 } /* Keyword.Reserved */
.rrt .kt { color: #ee82ee } /* Keyword.Type */
.rrt .s { color: #87ceeb } /* Literal.String */
.rrt .no { color: #7fffd4 } /* Name.Constant */
.rrt .nf { color: #ffff00 } /* Name.Function */
.rrt .nv { color: #eedd82 } /* Name.Variable */
.rrt .sb { color: #87ceeb } /* Literal.String.Backtick */
.rrt .sc { color: #87ceeb } /* Literal.String.Char */
.rrt .sd { color: #87ceeb } /* Literal.String.Doc */
.rrt .s2 { color: #87ceeb } /* Literal.String.Double */
.rrt .se { color: #87ceeb } /* Literal.String.Escape */
.rrt .sh { color: #87ceeb } /* Literal.String.Heredoc */
.rrt .si { color: #87ceeb } /* Literal.String.Interpol */
.rrt .sx { color: #87ceeb } /* Literal.String.Other */
.rrt .sr { color: #87ceeb } /* Literal.String.Regex */
.rrt .s1 { color: #87ceeb } /* Literal.String.Single */
.rrt .ss { color: #87ceeb } /* Literal.String.Symbol */
.rrt .vc { color: #eedd82 } /* Name.Variable.Class */
.rrt .vg { color: #eedd82 } /* Name.Variable.Global */
.rrt .vi { color: #eedd82 } /* Name.Variable.Instance */</style>
<div class="rrt"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>perldoc</h3>
<style type="text/css">.perldoc .hll { background-color: #ffffcc }
.perldoc  { background: #eeeedd; }
.perldoc .c { color: #228B22 } /* Comment */
.perldoc .err { color: #a61717; background-color: #e3d2d2 } /* Error */
.perldoc .k { color: #8B008B; font-weight: bold } /* Keyword */
.perldoc .cm { color: #228B22 } /* Comment.Multiline */
.perldoc .cp { color: #1e889b } /* Comment.Preproc */
.perldoc .c1 { color: #228B22 } /* Comment.Single */
.perldoc .cs { color: #8B008B; font-weight: bold } /* Comment.Special */
.perldoc .gd { color: #aa0000 } /* Generic.Deleted */
.perldoc .ge { font-style: italic } /* Generic.Emph */
.perldoc .gr { color: #aa0000 } /* Generic.Error */
.perldoc .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.perldoc .gi { color: #00aa00 } /* Generic.Inserted */
.perldoc .go { color: #888888 } /* Generic.Output */
.perldoc .gp { color: #555555 } /* Generic.Prompt */
.perldoc .gs { font-weight: bold } /* Generic.Strong */
.perldoc .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.perldoc .gt { color: #aa0000 } /* Generic.Traceback */
.perldoc .kc { color: #8B008B; font-weight: bold } /* Keyword.Constant */
.perldoc .kd { color: #8B008B; font-weight: bold } /* Keyword.Declaration */
.perldoc .kn { color: #8B008B; font-weight: bold } /* Keyword.Namespace */
.perldoc .kp { color: #8B008B; font-weight: bold } /* Keyword.Pseudo */
.perldoc .kr { color: #8B008B; font-weight: bold } /* Keyword.Reserved */
.perldoc .kt { color: #a7a7a7; font-weight: bold } /* Keyword.Type */
.perldoc .m { color: #B452CD } /* Literal.Number */
.perldoc .s { color: #CD5555 } /* Literal.String */
.perldoc .na { color: #658b00 } /* Name.Attribute */
.perldoc .nb { color: #658b00 } /* Name.Builtin */
.perldoc .nc { color: #008b45; font-weight: bold } /* Name.Class */
.perldoc .no { color: #00688B } /* Name.Constant */
.perldoc .nd { color: #707a7c } /* Name.Decorator */
.perldoc .ne { color: #008b45; font-weight: bold } /* Name.Exception */
.perldoc .nf { color: #008b45 } /* Name.Function */
.perldoc .nn { color: #008b45; text-decoration: underline } /* Name.Namespace */
.perldoc .nt { color: #8B008B; font-weight: bold } /* Name.Tag */
.perldoc .nv { color: #00688B } /* Name.Variable */
.perldoc .ow { color: #8B008B } /* Operator.Word */
.perldoc .w { color: #bbbbbb } /* Text.Whitespace */
.perldoc .mf { color: #B452CD } /* Literal.Number.Float */
.perldoc .mh { color: #B452CD } /* Literal.Number.Hex */
.perldoc .mi { color: #B452CD } /* Literal.Number.Integer */
.perldoc .mo { color: #B452CD } /* Literal.Number.Oct */
.perldoc .sb { color: #CD5555 } /* Literal.String.Backtick */
.perldoc .sc { color: #CD5555 } /* Literal.String.Char */
.perldoc .sd { color: #CD5555 } /* Literal.String.Doc */
.perldoc .s2 { color: #CD5555 } /* Literal.String.Double */
.perldoc .se { color: #CD5555 } /* Literal.String.Escape */
.perldoc .sh { color: #1c7e71; font-style: italic } /* Literal.String.Heredoc */
.perldoc .si { color: #CD5555 } /* Literal.String.Interpol */
.perldoc .sx { color: #cb6c20 } /* Literal.String.Other */
.perldoc .sr { color: #1c7e71 } /* Literal.String.Regex */
.perldoc .s1 { color: #CD5555 } /* Literal.String.Single */
.perldoc .ss { color: #CD5555 } /* Literal.String.Symbol */
.perldoc .bp { color: #658b00 } /* Name.Builtin.Pseudo */
.perldoc .vc { color: #00688B } /* Name.Variable.Class */
.perldoc .vg { color: #00688B } /* Name.Variable.Global */
.perldoc .vi { color: #00688B } /* Name.Variable.Instance */
.perldoc .il { color: #B452CD } /* Literal.Number.Integer.Long */</style>
<div class="perldoc"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>borland</h3>
<style type="text/css">.borland .hll { background-color: #ffffcc }
.borland  { background: #ffffff; }
.borland .c { color: #008800; font-style: italic } /* Comment */
.borland .err { color: #a61717; background-color: #e3d2d2 } /* Error */
.borland .k { color: #000080; font-weight: bold } /* Keyword */
.borland .cm { color: #008800; font-style: italic } /* Comment.Multiline */
.borland .cp { color: #008080 } /* Comment.Preproc */
.borland .c1 { color: #008800; font-style: italic } /* Comment.Single */
.borland .cs { color: #008800; font-weight: bold } /* Comment.Special */
.borland .gd { color: #000000; background-color: #ffdddd } /* Generic.Deleted */
.borland .ge { font-style: italic } /* Generic.Emph */
.borland .gr { color: #aa0000 } /* Generic.Error */
.borland .gh { color: #999999 } /* Generic.Heading */
.borland .gi { color: #000000; background-color: #ddffdd } /* Generic.Inserted */
.borland .go { color: #888888 } /* Generic.Output */
.borland .gp { color: #555555 } /* Generic.Prompt */
.borland .gs { font-weight: bold } /* Generic.Strong */
.borland .gu { color: #aaaaaa } /* Generic.Subheading */
.borland .gt { color: #aa0000 } /* Generic.Traceback */
.borland .kc { color: #000080; font-weight: bold } /* Keyword.Constant */
.borland .kd { color: #000080; font-weight: bold } /* Keyword.Declaration */
.borland .kn { color: #000080; font-weight: bold } /* Keyword.Namespace */
.borland .kp { color: #000080; font-weight: bold } /* Keyword.Pseudo */
.borland .kr { color: #000080; font-weight: bold } /* Keyword.Reserved */
.borland .kt { color: #000080; font-weight: bold } /* Keyword.Type */
.borland .m { color: #0000FF } /* Literal.Number */
.borland .s { color: #0000FF } /* Literal.String */
.borland .na { color: #FF0000 } /* Name.Attribute */
.borland .nt { color: #000080; font-weight: bold } /* Name.Tag */
.borland .ow { font-weight: bold } /* Operator.Word */
.borland .w { color: #bbbbbb } /* Text.Whitespace */
.borland .mf { color: #0000FF } /* Literal.Number.Float */
.borland .mh { color: #0000FF } /* Literal.Number.Hex */
.borland .mi { color: #0000FF } /* Literal.Number.Integer */
.borland .mo { color: #0000FF } /* Literal.Number.Oct */
.borland .sb { color: #0000FF } /* Literal.String.Backtick */
.borland .sc { color: #800080 } /* Literal.String.Char */
.borland .sd { color: #0000FF } /* Literal.String.Doc */
.borland .s2 { color: #0000FF } /* Literal.String.Double */
.borland .se { color: #0000FF } /* Literal.String.Escape */
.borland .sh { color: #0000FF } /* Literal.String.Heredoc */
.borland .si { color: #0000FF } /* Literal.String.Interpol */
.borland .sx { color: #0000FF } /* Literal.String.Other */
.borland .sr { color: #0000FF } /* Literal.String.Regex */
.borland .s1 { color: #0000FF } /* Literal.String.Single */
.borland .ss { color: #0000FF } /* Literal.String.Symbol */
.borland .il { color: #0000FF } /* Literal.Number.Integer.Long */</style>
<div class="borland"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>colorful</h3>
<style type="text/css">.colorful .hll { background-color: #ffffcc }
.colorful  { background: #ffffff; }
.colorful .c { color: #808080 } /* Comment */
.colorful .err { color: #F00000; background-color: #F0A0A0 } /* Error */
.colorful .k { color: #008000; font-weight: bold } /* Keyword */
.colorful .o { color: #303030 } /* Operator */
.colorful .cm { color: #808080 } /* Comment.Multiline */
.colorful .cp { color: #507090 } /* Comment.Preproc */
.colorful .c1 { color: #808080 } /* Comment.Single */
.colorful .cs { color: #cc0000; font-weight: bold } /* Comment.Special */
.colorful .gd { color: #A00000 } /* Generic.Deleted */
.colorful .ge { font-style: italic } /* Generic.Emph */
.colorful .gr { color: #FF0000 } /* Generic.Error */
.colorful .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.colorful .gi { color: #00A000 } /* Generic.Inserted */
.colorful .go { color: #808080 } /* Generic.Output */
.colorful .gp { color: #c65d09; font-weight: bold } /* Generic.Prompt */
.colorful .gs { font-weight: bold } /* Generic.Strong */
.colorful .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.colorful .gt { color: #0040D0 } /* Generic.Traceback */
.colorful .kc { color: #008000; font-weight: bold } /* Keyword.Constant */
.colorful .kd { color: #008000; font-weight: bold } /* Keyword.Declaration */
.colorful .kn { color: #008000; font-weight: bold } /* Keyword.Namespace */
.colorful .kp { color: #003080; font-weight: bold } /* Keyword.Pseudo */
.colorful .kr { color: #008000; font-weight: bold } /* Keyword.Reserved */
.colorful .kt { color: #303090; font-weight: bold } /* Keyword.Type */
.colorful .m { color: #6000E0; font-weight: bold } /* Literal.Number */
.colorful .s { background-color: #fff0f0 } /* Literal.String */
.colorful .na { color: #0000C0 } /* Name.Attribute */
.colorful .nb { color: #007020 } /* Name.Builtin */
.colorful .nc { color: #B00060; font-weight: bold } /* Name.Class */
.colorful .no { color: #003060; font-weight: bold } /* Name.Constant */
.colorful .nd { color: #505050; font-weight: bold } /* Name.Decorator */
.colorful .ni { color: #800000; font-weight: bold } /* Name.Entity */
.colorful .ne { color: #F00000; font-weight: bold } /* Name.Exception */
.colorful .nf { color: #0060B0; font-weight: bold } /* Name.Function */
.colorful .nl { color: #907000; font-weight: bold } /* Name.Label */
.colorful .nn { color: #0e84b5; font-weight: bold } /* Name.Namespace */
.colorful .nt { color: #007000 } /* Name.Tag */
.colorful .nv { color: #906030 } /* Name.Variable */
.colorful .ow { color: #000000; font-weight: bold } /* Operator.Word */
.colorful .w { color: #bbbbbb } /* Text.Whitespace */
.colorful .mf { color: #6000E0; font-weight: bold } /* Literal.Number.Float */
.colorful .mh { color: #005080; font-weight: bold } /* Literal.Number.Hex */
.colorful .mi { color: #0000D0; font-weight: bold } /* Literal.Number.Integer */
.colorful .mo { color: #4000E0; font-weight: bold } /* Literal.Number.Oct */
.colorful .sb { background-color: #fff0f0 } /* Literal.String.Backtick */
.colorful .sc { color: #0040D0 } /* Literal.String.Char */
.colorful .sd { color: #D04020 } /* Literal.String.Doc */
.colorful .s2 { background-color: #fff0f0 } /* Literal.String.Double */
.colorful .se { color: #606060; font-weight: bold; background-color: #fff0f0 } /* Literal.String.Escape */
.colorful .sh { background-color: #fff0f0 } /* Literal.String.Heredoc */
.colorful .si { background-color: #e0e0e0 } /* Literal.String.Interpol */
.colorful .sx { color: #D02000; background-color: #fff0f0 } /* Literal.String.Other */
.colorful .sr { color: #000000; background-color: #fff0ff } /* Literal.String.Regex */
.colorful .s1 { background-color: #fff0f0 } /* Literal.String.Single */
.colorful .ss { color: #A06000 } /* Literal.String.Symbol */
.colorful .bp { color: #007020 } /* Name.Builtin.Pseudo */
.colorful .vc { color: #306090 } /* Name.Variable.Class */
.colorful .vg { color: #d07000; font-weight: bold } /* Name.Variable.Global */
.colorful .vi { color: #3030B0 } /* Name.Variable.Instance */
.colorful .il { color: #0000D0; font-weight: bold } /* Literal.Number.Integer.Long */</style>
<div class="colorful"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>default</h3>
<style type="text/css">.default .hll { background-color: #ffffcc }
.default  { background: #f8f8f8; }
.default .c { color: #408080; font-style: italic } /* Comment */
.default .err { border: 1px solid #FF0000 } /* Error */
.default .k { color: #008000; font-weight: bold } /* Keyword */
.default .o { color: #666666 } /* Operator */
.default .cm { color: #408080; font-style: italic } /* Comment.Multiline */
.default .cp { color: #BC7A00 } /* Comment.Preproc */
.default .c1 { color: #408080; font-style: italic } /* Comment.Single */
.default .cs { color: #408080; font-style: italic } /* Comment.Special */
.default .gd { color: #A00000 } /* Generic.Deleted */
.default .ge { font-style: italic } /* Generic.Emph */
.default .gr { color: #FF0000 } /* Generic.Error */
.default .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.default .gi { color: #00A000 } /* Generic.Inserted */
.default .go { color: #808080 } /* Generic.Output */
.default .gp { color: #000080; font-weight: bold } /* Generic.Prompt */
.default .gs { font-weight: bold } /* Generic.Strong */
.default .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.default .gt { color: #0040D0 } /* Generic.Traceback */
.default .kc { color: #008000; font-weight: bold } /* Keyword.Constant */
.default .kd { color: #008000; font-weight: bold } /* Keyword.Declaration */
.default .kn { color: #008000; font-weight: bold } /* Keyword.Namespace */
.default .kp { color: #008000 } /* Keyword.Pseudo */
.default .kr { color: #008000; font-weight: bold } /* Keyword.Reserved */
.default .kt { color: #B00040 } /* Keyword.Type */
.default .m { color: #666666 } /* Literal.Number */
.default .s { color: #BA2121 } /* Literal.String */
.default .na { color: #7D9029 } /* Name.Attribute */
.default .nb { color: #008000 } /* Name.Builtin */
.default .nc { color: #0000FF; font-weight: bold } /* Name.Class */
.default .no { color: #880000 } /* Name.Constant */
.default .nd { color: #AA22FF } /* Name.Decorator */
.default .ni { color: #999999; font-weight: bold } /* Name.Entity */
.default .ne { color: #D2413A; font-weight: bold } /* Name.Exception */
.default .nf { color: #0000FF } /* Name.Function */
.default .nl { color: #A0A000 } /* Name.Label */
.default .nn { color: #0000FF; font-weight: bold } /* Name.Namespace */
.default .nt { color: #008000; font-weight: bold } /* Name.Tag */
.default .nv { color: #19177C } /* Name.Variable */
.default .ow { color: #AA22FF; font-weight: bold } /* Operator.Word */
.default .w { color: #bbbbbb } /* Text.Whitespace */
.default .mf { color: #666666 } /* Literal.Number.Float */
.default .mh { color: #666666 } /* Literal.Number.Hex */
.default .mi { color: #666666 } /* Literal.Number.Integer */
.default .mo { color: #666666 } /* Literal.Number.Oct */
.default .sb { color: #BA2121 } /* Literal.String.Backtick */
.default .sc { color: #BA2121 } /* Literal.String.Char */
.default .sd { color: #BA2121; font-style: italic } /* Literal.String.Doc */
.default .s2 { color: #BA2121 } /* Literal.String.Double */
.default .se { color: #BB6622; font-weight: bold } /* Literal.String.Escape */
.default .sh { color: #BA2121 } /* Literal.String.Heredoc */
.default .si { color: #BB6688; font-weight: bold } /* Literal.String.Interpol */
.default .sx { color: #008000 } /* Literal.String.Other */
.default .sr { color: #BB6688 } /* Literal.String.Regex */
.default .s1 { color: #BA2121 } /* Literal.String.Single */
.default .ss { color: #19177C } /* Literal.String.Symbol */
.default .bp { color: #008000 } /* Name.Builtin.Pseudo */
.default .vc { color: #19177C } /* Name.Variable.Class */
.default .vg { color: #19177C } /* Name.Variable.Global */
.default .vi { color: #19177C } /* Name.Variable.Instance */
.default .il { color: #666666 } /* Literal.Number.Integer.Long */</style>
<div class="default"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>murphy</h3>
<style type="text/css">.murphy .hll { background-color: #ffffcc }
.murphy  { background: #ffffff; }
.murphy .c { color: #606060; font-style: italic } /* Comment */
.murphy .err { color: #F00000; background-color: #F0A0A0 } /* Error */
.murphy .k { color: #208090; font-weight: bold } /* Keyword */
.murphy .o { color: #303030 } /* Operator */
.murphy .cm { color: #606060; font-style: italic } /* Comment.Multiline */
.murphy .cp { color: #507090 } /* Comment.Preproc */
.murphy .c1 { color: #606060; font-style: italic } /* Comment.Single */
.murphy .cs { color: #c00000; font-weight: bold; font-style: italic } /* Comment.Special */
.murphy .gd { color: #A00000 } /* Generic.Deleted */
.murphy .ge { font-style: italic } /* Generic.Emph */
.murphy .gr { color: #FF0000 } /* Generic.Error */
.murphy .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.murphy .gi { color: #00A000 } /* Generic.Inserted */
.murphy .go { color: #808080 } /* Generic.Output */
.murphy .gp { color: #c65d09; font-weight: bold } /* Generic.Prompt */
.murphy .gs { font-weight: bold } /* Generic.Strong */
.murphy .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.murphy .gt { color: #0040D0 } /* Generic.Traceback */
.murphy .kc { color: #208090; font-weight: bold } /* Keyword.Constant */
.murphy .kd { color: #208090; font-weight: bold } /* Keyword.Declaration */
.murphy .kn { color: #208090; font-weight: bold } /* Keyword.Namespace */
.murphy .kp { color: #0080f0; font-weight: bold } /* Keyword.Pseudo */
.murphy .kr { color: #208090; font-weight: bold } /* Keyword.Reserved */
.murphy .kt { color: #6060f0; font-weight: bold } /* Keyword.Type */
.murphy .m { color: #6000E0; font-weight: bold } /* Literal.Number */
.murphy .s { background-color: #e0e0ff } /* Literal.String */
.murphy .na { color: #000070 } /* Name.Attribute */
.murphy .nb { color: #007020 } /* Name.Builtin */
.murphy .nc { color: #e090e0; font-weight: bold } /* Name.Class */
.murphy .no { color: #50e0d0; font-weight: bold } /* Name.Constant */
.murphy .nd { color: #505050; font-weight: bold } /* Name.Decorator */
.murphy .ni { color: #800000 } /* Name.Entity */
.murphy .ne { color: #F00000; font-weight: bold } /* Name.Exception */
.murphy .nf { color: #50e0d0; font-weight: bold } /* Name.Function */
.murphy .nl { color: #907000; font-weight: bold } /* Name.Label */
.murphy .nn { color: #0e84b5; font-weight: bold } /* Name.Namespace */
.murphy .nt { color: #007000 } /* Name.Tag */
.murphy .nv { color: #003060 } /* Name.Variable */
.murphy .ow { color: #000000; font-weight: bold } /* Operator.Word */
.murphy .w { color: #bbbbbb } /* Text.Whitespace */
.murphy .mf { color: #6000E0; font-weight: bold } /* Literal.Number.Float */
.murphy .mh { color: #005080; font-weight: bold } /* Literal.Number.Hex */
.murphy .mi { color: #6060f0; font-weight: bold } /* Literal.Number.Integer */
.murphy .mo { color: #4000E0; font-weight: bold } /* Literal.Number.Oct */
.murphy .sb { background-color: #e0e0ff } /* Literal.String.Backtick */
.murphy .sc { color: #8080F0 } /* Literal.String.Char */
.murphy .sd { color: #D04020 } /* Literal.String.Doc */
.murphy .s2 { background-color: #e0e0ff } /* Literal.String.Double */
.murphy .se { color: #606060; font-weight: bold; background-color: #e0e0ff } /* Literal.String.Escape */
.murphy .sh { background-color: #e0e0ff } /* Literal.String.Heredoc */
.murphy .si { background-color: #e0e0e0 } /* Literal.String.Interpol */
.murphy .sx { color: #f08080; background-color: #e0e0ff } /* Literal.String.Other */
.murphy .sr { color: #000000; background-color: #e0e0ff } /* Literal.String.Regex */
.murphy .s1 { background-color: #e0e0ff } /* Literal.String.Single */
.murphy .ss { color: #f0c080 } /* Literal.String.Symbol */
.murphy .bp { color: #007020 } /* Name.Builtin.Pseudo */
.murphy .vc { color: #c0c0f0 } /* Name.Variable.Class */
.murphy .vg { color: #f08040 } /* Name.Variable.Global */
.murphy .vi { color: #a0a0f0 } /* Name.Variable.Instance */
.murphy .il { color: #6060f0; font-weight: bold } /* Literal.Number.Integer.Long */</style>
<div class="murphy"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>vs</h3>
<style type="text/css">.vs .hll { background-color: #ffffcc }
.vs  { background: #ffffff; }
.vs .c { color: #008000 } /* Comment */
.vs .err { border: 1px solid #FF0000 } /* Error */
.vs .k { color: #0000ff } /* Keyword */
.vs .cm { color: #008000 } /* Comment.Multiline */
.vs .cp { color: #0000ff } /* Comment.Preproc */
.vs .c1 { color: #008000 } /* Comment.Single */
.vs .cs { color: #008000 } /* Comment.Special */
.vs .ge { font-style: italic } /* Generic.Emph */
.vs .gh { font-weight: bold } /* Generic.Heading */
.vs .gp { font-weight: bold } /* Generic.Prompt */
.vs .gs { font-weight: bold } /* Generic.Strong */
.vs .gu { font-weight: bold } /* Generic.Subheading */
.vs .kc { color: #0000ff } /* Keyword.Constant */
.vs .kd { color: #0000ff } /* Keyword.Declaration */
.vs .kn { color: #0000ff } /* Keyword.Namespace */
.vs .kp { color: #0000ff } /* Keyword.Pseudo */
.vs .kr { color: #0000ff } /* Keyword.Reserved */
.vs .kt { color: #2b91af } /* Keyword.Type */
.vs .s { color: #a31515 } /* Literal.String */
.vs .nc { color: #2b91af } /* Name.Class */
.vs .ow { color: #0000ff } /* Operator.Word */
.vs .sb { color: #a31515 } /* Literal.String.Backtick */
.vs .sc { color: #a31515 } /* Literal.String.Char */
.vs .sd { color: #a31515 } /* Literal.String.Doc */
.vs .s2 { color: #a31515 } /* Literal.String.Double */
.vs .se { color: #a31515 } /* Literal.String.Escape */
.vs .sh { color: #a31515 } /* Literal.String.Heredoc */
.vs .si { color: #a31515 } /* Literal.String.Interpol */
.vs .sx { color: #a31515 } /* Literal.String.Other */
.vs .sr { color: #a31515 } /* Literal.String.Regex */
.vs .s1 { color: #a31515 } /* Literal.String.Single */
.vs .ss { color: #a31515 } /* Literal.String.Symbol */</style>
<div class="vs"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>trac</h3>
<style type="text/css">.trac .hll { background-color: #ffffcc }
.trac  { background: #ffffff; }
.trac .c { color: #999988; font-style: italic } /* Comment */
.trac .err { color: #a61717; background-color: #e3d2d2 } /* Error */
.trac .k { font-weight: bold } /* Keyword */
.trac .o { font-weight: bold } /* Operator */
.trac .cm { color: #999988; font-style: italic } /* Comment.Multiline */
.trac .cp { color: #999999; font-weight: bold } /* Comment.Preproc */
.trac .c1 { color: #999988; font-style: italic } /* Comment.Single */
.trac .cs { color: #999999; font-weight: bold; font-style: italic } /* Comment.Special */
.trac .gd { color: #000000; background-color: #ffdddd } /* Generic.Deleted */
.trac .ge { font-style: italic } /* Generic.Emph */
.trac .gr { color: #aa0000 } /* Generic.Error */
.trac .gh { color: #999999 } /* Generic.Heading */
.trac .gi { color: #000000; background-color: #ddffdd } /* Generic.Inserted */
.trac .go { color: #888888 } /* Generic.Output */
.trac .gp { color: #555555 } /* Generic.Prompt */
.trac .gs { font-weight: bold } /* Generic.Strong */
.trac .gu { color: #aaaaaa } /* Generic.Subheading */
.trac .gt { color: #aa0000 } /* Generic.Traceback */
.trac .kc { font-weight: bold } /* Keyword.Constant */
.trac .kd { font-weight: bold } /* Keyword.Declaration */
.trac .kn { font-weight: bold } /* Keyword.Namespace */
.trac .kp { font-weight: bold } /* Keyword.Pseudo */
.trac .kr { font-weight: bold } /* Keyword.Reserved */
.trac .kt { color: #445588; font-weight: bold } /* Keyword.Type */
.trac .m { color: #009999 } /* Literal.Number */
.trac .s { color: #bb8844 } /* Literal.String */
.trac .na { color: #008080 } /* Name.Attribute */
.trac .nb { color: #999999 } /* Name.Builtin */
.trac .nc { color: #445588; font-weight: bold } /* Name.Class */
.trac .no { color: #008080 } /* Name.Constant */
.trac .ni { color: #800080 } /* Name.Entity */
.trac .ne { color: #990000; font-weight: bold } /* Name.Exception */
.trac .nf { color: #990000; font-weight: bold } /* Name.Function */
.trac .nn { color: #555555 } /* Name.Namespace */
.trac .nt { color: #000080 } /* Name.Tag */
.trac .nv { color: #008080 } /* Name.Variable */
.trac .ow { font-weight: bold } /* Operator.Word */
.trac .w { color: #bbbbbb } /* Text.Whitespace */
.trac .mf { color: #009999 } /* Literal.Number.Float */
.trac .mh { color: #009999 } /* Literal.Number.Hex */
.trac .mi { color: #009999 } /* Literal.Number.Integer */
.trac .mo { color: #009999 } /* Literal.Number.Oct */
.trac .sb { color: #bb8844 } /* Literal.String.Backtick */
.trac .sc { color: #bb8844 } /* Literal.String.Char */
.trac .sd { color: #bb8844 } /* Literal.String.Doc */
.trac .s2 { color: #bb8844 } /* Literal.String.Double */
.trac .se { color: #bb8844 } /* Literal.String.Escape */
.trac .sh { color: #bb8844 } /* Literal.String.Heredoc */
.trac .si { color: #bb8844 } /* Literal.String.Interpol */
.trac .sx { color: #bb8844 } /* Literal.String.Other */
.trac .sr { color: #808000 } /* Literal.String.Regex */
.trac .s1 { color: #bb8844 } /* Literal.String.Single */
.trac .ss { color: #bb8844 } /* Literal.String.Symbol */
.trac .bp { color: #999999 } /* Name.Builtin.Pseudo */
.trac .vc { color: #008080 } /* Name.Variable.Class */
.trac .vg { color: #008080 } /* Name.Variable.Global */
.trac .vi { color: #008080 } /* Name.Variable.Instance */
.trac .il { color: #009999 } /* Literal.Number.Integer.Long */</style>
<div class="trac"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>tango</h3>
<style type="text/css">.tango .hll { background-color: #ffffcc }
.tango  { background: #f8f8f8; }
.tango .c { color: #8f5902; font-style: italic } /* Comment */
.tango .err { color: #a40000; border: 1px solid #ef2929 } /* Error */
.tango .g { color: #000000 } /* Generic */
.tango .k { color: #204a87; font-weight: bold } /* Keyword */
.tango .l { color: #000000 } /* Literal */
.tango .n { color: #000000 } /* Name */
.tango .o { color: #ce5c00; font-weight: bold } /* Operator */
.tango .x { color: #000000 } /* Other */
.tango .p { color: #000000; font-weight: bold } /* Punctuation */
.tango .cm { color: #8f5902; font-style: italic } /* Comment.Multiline */
.tango .cp { color: #8f5902; font-style: italic } /* Comment.Preproc */
.tango .c1 { color: #8f5902; font-style: italic } /* Comment.Single */
.tango .cs { color: #8f5902; font-style: italic } /* Comment.Special */
.tango .gd { color: #a40000 } /* Generic.Deleted */
.tango .ge { color: #000000; font-style: italic } /* Generic.Emph */
.tango .gr { color: #ef2929 } /* Generic.Error */
.tango .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.tango .gi { color: #00A000 } /* Generic.Inserted */
.tango .go { color: #000000; font-style: italic } /* Generic.Output */
.tango .gp { color: #8f5902 } /* Generic.Prompt */
.tango .gs { color: #000000; font-weight: bold } /* Generic.Strong */
.tango .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.tango .gt { color: #a40000; font-weight: bold } /* Generic.Traceback */
.tango .kc { color: #204a87; font-weight: bold } /* Keyword.Constant */
.tango .kd { color: #204a87; font-weight: bold } /* Keyword.Declaration */
.tango .kn { color: #204a87; font-weight: bold } /* Keyword.Namespace */
.tango .kp { color: #204a87; font-weight: bold } /* Keyword.Pseudo */
.tango .kr { color: #204a87; font-weight: bold } /* Keyword.Reserved */
.tango .kt { color: #204a87; font-weight: bold } /* Keyword.Type */
.tango .ld { color: #000000 } /* Literal.Date */
.tango .m { color: #0000cf; font-weight: bold } /* Literal.Number */
.tango .s { color: #4e9a06 } /* Literal.String */
.tango .na { color: #c4a000 } /* Name.Attribute */
.tango .nb { color: #204a87 } /* Name.Builtin */
.tango .nc { color: #000000 } /* Name.Class */
.tango .no { color: #000000 } /* Name.Constant */
.tango .nd { color: #5c35cc; font-weight: bold } /* Name.Decorator */
.tango .ni { color: #ce5c00 } /* Name.Entity */
.tango .ne { color: #cc0000; font-weight: bold } /* Name.Exception */
.tango .nf { color: #000000 } /* Name.Function */
.tango .nl { color: #f57900 } /* Name.Label */
.tango .nn { color: #000000 } /* Name.Namespace */
.tango .nx { color: #000000 } /* Name.Other */
.tango .py { color: #000000 } /* Name.Property */
.tango .nt { color: #204a87; font-weight: bold } /* Name.Tag */
.tango .nv { color: #000000 } /* Name.Variable */
.tango .ow { color: #204a87; font-weight: bold } /* Operator.Word */
.tango .w { color: #f8f8f8; text-decoration: underline } /* Text.Whitespace */
.tango .mf { color: #0000cf; font-weight: bold } /* Literal.Number.Float */
.tango .mh { color: #0000cf; font-weight: bold } /* Literal.Number.Hex */
.tango .mi { color: #0000cf; font-weight: bold } /* Literal.Number.Integer */
.tango .mo { color: #0000cf; font-weight: bold } /* Literal.Number.Oct */
.tango .sb { color: #4e9a06 } /* Literal.String.Backtick */
.tango .sc { color: #4e9a06 } /* Literal.String.Char */
.tango .sd { color: #8f5902; font-style: italic } /* Literal.String.Doc */
.tango .s2 { color: #4e9a06 } /* Literal.String.Double */
.tango .se { color: #4e9a06 } /* Literal.String.Escape */
.tango .sh { color: #4e9a06 } /* Literal.String.Heredoc */
.tango .si { color: #4e9a06 } /* Literal.String.Interpol */
.tango .sx { color: #4e9a06 } /* Literal.String.Other */
.tango .sr { color: #4e9a06 } /* Literal.String.Regex */
.tango .s1 { color: #4e9a06 } /* Literal.String.Single */
.tango .ss { color: #4e9a06 } /* Literal.String.Symbol */
.tango .bp { color: #3465a4 } /* Name.Builtin.Pseudo */
.tango .vc { color: #000000 } /* Name.Variable.Class */
.tango .vg { color: #000000 } /* Name.Variable.Global */
.tango .vi { color: #000000 } /* Name.Variable.Instance */
.tango .il { color: #0000cf; font-weight: bold } /* Literal.Number.Integer.Long */</style>
<div class="tango"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>fruity</h3>
<style type="text/css">.fruity .hll { background-color: #333333 }
.fruity  { background: #111111; color: #ffffff }
.fruity .c { color: #008800; font-style: italic; background-color: #0f140f } /* Comment */
.fruity .err { color: #ffffff } /* Error */
.fruity .g { color: #ffffff } /* Generic */
.fruity .k { color: #fb660a; font-weight: bold } /* Keyword */
.fruity .l { color: #ffffff } /* Literal */
.fruity .n { color: #ffffff } /* Name */
.fruity .o { color: #ffffff } /* Operator */
.fruity .x { color: #ffffff } /* Other */
.fruity .p { color: #ffffff } /* Punctuation */
.fruity .cm { color: #008800; font-style: italic; background-color: #0f140f } /* Comment.Multiline */
.fruity .cp { color: #ff0007; font-weight: bold; font-style: italic; background-color: #0f140f } /* Comment.Preproc */
.fruity .c1 { color: #008800; font-style: italic; background-color: #0f140f } /* Comment.Single */
.fruity .cs { color: #008800; font-style: italic; background-color: #0f140f } /* Comment.Special */
.fruity .gd { color: #ffffff } /* Generic.Deleted */
.fruity .ge { color: #ffffff } /* Generic.Emph */
.fruity .gr { color: #ffffff } /* Generic.Error */
.fruity .gh { color: #ffffff; font-weight: bold } /* Generic.Heading */
.fruity .gi { color: #ffffff } /* Generic.Inserted */
.fruity .go { color: #444444; background-color: #222222 } /* Generic.Output */
.fruity .gp { color: #ffffff } /* Generic.Prompt */
.fruity .gs { color: #ffffff } /* Generic.Strong */
.fruity .gu { color: #ffffff; font-weight: bold } /* Generic.Subheading */
.fruity .gt { color: #ffffff } /* Generic.Traceback */
.fruity .kc { color: #fb660a; font-weight: bold } /* Keyword.Constant */
.fruity .kd { color: #fb660a; font-weight: bold } /* Keyword.Declaration */
.fruity .kn { color: #fb660a; font-weight: bold } /* Keyword.Namespace */
.fruity .kp { color: #fb660a } /* Keyword.Pseudo */
.fruity .kr { color: #fb660a; font-weight: bold } /* Keyword.Reserved */
.fruity .kt { color: #cdcaa9; font-weight: bold } /* Keyword.Type */
.fruity .ld { color: #ffffff } /* Literal.Date */
.fruity .m { color: #0086f7; font-weight: bold } /* Literal.Number */
.fruity .s { color: #0086d2 } /* Literal.String */
.fruity .na { color: #ff0086; font-weight: bold } /* Name.Attribute */
.fruity .nb { color: #ffffff } /* Name.Builtin */
.fruity .nc { color: #ffffff } /* Name.Class */
.fruity .no { color: #0086d2 } /* Name.Constant */
.fruity .nd { color: #ffffff } /* Name.Decorator */
.fruity .ni { color: #ffffff } /* Name.Entity */
.fruity .ne { color: #ffffff } /* Name.Exception */
.fruity .nf { color: #ff0086; font-weight: bold } /* Name.Function */
.fruity .nl { color: #ffffff } /* Name.Label */
.fruity .nn { color: #ffffff } /* Name.Namespace */
.fruity .nx { color: #ffffff } /* Name.Other */
.fruity .py { color: #ffffff } /* Name.Property */
.fruity .nt { color: #fb660a; font-weight: bold } /* Name.Tag */
.fruity .nv { color: #fb660a } /* Name.Variable */
.fruity .ow { color: #ffffff } /* Operator.Word */
.fruity .w { color: #888888 } /* Text.Whitespace */
.fruity .mf { color: #0086f7; font-weight: bold } /* Literal.Number.Float */
.fruity .mh { color: #0086f7; font-weight: bold } /* Literal.Number.Hex */
.fruity .mi { color: #0086f7; font-weight: bold } /* Literal.Number.Integer */
.fruity .mo { color: #0086f7; font-weight: bold } /* Literal.Number.Oct */
.fruity .sb { color: #0086d2 } /* Literal.String.Backtick */
.fruity .sc { color: #0086d2 } /* Literal.String.Char */
.fruity .sd { color: #0086d2 } /* Literal.String.Doc */
.fruity .s2 { color: #0086d2 } /* Literal.String.Double */
.fruity .se { color: #0086d2 } /* Literal.String.Escape */
.fruity .sh { color: #0086d2 } /* Literal.String.Heredoc */
.fruity .si { color: #0086d2 } /* Literal.String.Interpol */
.fruity .sx { color: #0086d2 } /* Literal.String.Other */
.fruity .sr { color: #0086d2 } /* Literal.String.Regex */
.fruity .s1 { color: #0086d2 } /* Literal.String.Single */
.fruity .ss { color: #0086d2 } /* Literal.String.Symbol */
.fruity .bp { color: #ffffff } /* Name.Builtin.Pseudo */
.fruity .vc { color: #fb660a } /* Name.Variable.Class */
.fruity .vg { color: #fb660a } /* Name.Variable.Global */
.fruity .vi { color: #fb660a } /* Name.Variable.Instance */
.fruity .il { color: #0086f7; font-weight: bold } /* Literal.Number.Integer.Long */</style>
<div class="fruity"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>autumn</h3>
<style type="text/css">.autumn .hll { background-color: #ffffcc }
.autumn  { background: #ffffff; }
.autumn .c { color: #aaaaaa; font-style: italic } /* Comment */
.autumn .err { color: #F00000; background-color: #F0A0A0 } /* Error */
.autumn .k { color: #0000aa } /* Keyword */
.autumn .cm { color: #aaaaaa; font-style: italic } /* Comment.Multiline */
.autumn .cp { color: #4c8317 } /* Comment.Preproc */
.autumn .c1 { color: #aaaaaa; font-style: italic } /* Comment.Single */
.autumn .cs { color: #0000aa; font-style: italic } /* Comment.Special */
.autumn .gd { color: #aa0000 } /* Generic.Deleted */
.autumn .ge { font-style: italic } /* Generic.Emph */
.autumn .gr { color: #aa0000 } /* Generic.Error */
.autumn .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.autumn .gi { color: #00aa00 } /* Generic.Inserted */
.autumn .go { color: #888888 } /* Generic.Output */
.autumn .gp { color: #555555 } /* Generic.Prompt */
.autumn .gs { font-weight: bold } /* Generic.Strong */
.autumn .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.autumn .gt { color: #aa0000 } /* Generic.Traceback */
.autumn .kc { color: #0000aa } /* Keyword.Constant */
.autumn .kd { color: #0000aa } /* Keyword.Declaration */
.autumn .kn { color: #0000aa } /* Keyword.Namespace */
.autumn .kp { color: #0000aa } /* Keyword.Pseudo */
.autumn .kr { color: #0000aa } /* Keyword.Reserved */
.autumn .kt { color: #00aaaa } /* Keyword.Type */
.autumn .m { color: #009999 } /* Literal.Number */
.autumn .s { color: #aa5500 } /* Literal.String */
.autumn .na { color: #1e90ff } /* Name.Attribute */
.autumn .nb { color: #00aaaa } /* Name.Builtin */
.autumn .nc { color: #00aa00; text-decoration: underline } /* Name.Class */
.autumn .no { color: #aa0000 } /* Name.Constant */
.autumn .nd { color: #888888 } /* Name.Decorator */
.autumn .ni { color: #800000; font-weight: bold } /* Name.Entity */
.autumn .nf { color: #00aa00 } /* Name.Function */
.autumn .nn { color: #00aaaa; text-decoration: underline } /* Name.Namespace */
.autumn .nt { color: #1e90ff; font-weight: bold } /* Name.Tag */
.autumn .nv { color: #aa0000 } /* Name.Variable */
.autumn .ow { color: #0000aa } /* Operator.Word */
.autumn .w { color: #bbbbbb } /* Text.Whitespace */
.autumn .mf { color: #009999 } /* Literal.Number.Float */
.autumn .mh { color: #009999 } /* Literal.Number.Hex */
.autumn .mi { color: #009999 } /* Literal.Number.Integer */
.autumn .mo { color: #009999 } /* Literal.Number.Oct */
.autumn .sb { color: #aa5500 } /* Literal.String.Backtick */
.autumn .sc { color: #aa5500 } /* Literal.String.Char */
.autumn .sd { color: #aa5500 } /* Literal.String.Doc */
.autumn .s2 { color: #aa5500 } /* Literal.String.Double */
.autumn .se { color: #aa5500 } /* Literal.String.Escape */
.autumn .sh { color: #aa5500 } /* Literal.String.Heredoc */
.autumn .si { color: #aa5500 } /* Literal.String.Interpol */
.autumn .sx { color: #aa5500 } /* Literal.String.Other */
.autumn .sr { color: #009999 } /* Literal.String.Regex */
.autumn .s1 { color: #aa5500 } /* Literal.String.Single */
.autumn .ss { color: #0000aa } /* Literal.String.Symbol */
.autumn .bp { color: #00aaaa } /* Name.Builtin.Pseudo */
.autumn .vc { color: #aa0000 } /* Name.Variable.Class */
.autumn .vg { color: #aa0000 } /* Name.Variable.Global */
.autumn .vi { color: #aa0000 } /* Name.Variable.Instance */
.autumn .il { color: #009999 } /* Literal.Number.Integer.Long */</style>
<div class="autumn"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>bw</h3>
<style type="text/css">.bw .hll { background-color: #ffffcc }
.bw  { background: #ffffff; }
.bw .c { font-style: italic } /* Comment */
.bw .err { border: 1px solid #FF0000 } /* Error */
.bw .k { font-weight: bold } /* Keyword */
.bw .cm { font-style: italic } /* Comment.Multiline */
.bw .c1 { font-style: italic } /* Comment.Single */
.bw .cs { font-style: italic } /* Comment.Special */
.bw .ge { font-style: italic } /* Generic.Emph */
.bw .gh { font-weight: bold } /* Generic.Heading */
.bw .gp { font-weight: bold } /* Generic.Prompt */
.bw .gs { font-weight: bold } /* Generic.Strong */
.bw .gu { font-weight: bold } /* Generic.Subheading */
.bw .kc { font-weight: bold } /* Keyword.Constant */
.bw .kd { font-weight: bold } /* Keyword.Declaration */
.bw .kn { font-weight: bold } /* Keyword.Namespace */
.bw .kr { font-weight: bold } /* Keyword.Reserved */
.bw .s { font-style: italic } /* Literal.String */
.bw .nc { font-weight: bold } /* Name.Class */
.bw .ni { font-weight: bold } /* Name.Entity */
.bw .ne { font-weight: bold } /* Name.Exception */
.bw .nn { font-weight: bold } /* Name.Namespace */
.bw .nt { font-weight: bold } /* Name.Tag */
.bw .ow { font-weight: bold } /* Operator.Word */
.bw .sb { font-style: italic } /* Literal.String.Backtick */
.bw .sc { font-style: italic } /* Literal.String.Char */
.bw .sd { font-style: italic } /* Literal.String.Doc */
.bw .s2 { font-style: italic } /* Literal.String.Double */
.bw .se { font-weight: bold; font-style: italic } /* Literal.String.Escape */
.bw .sh { font-style: italic } /* Literal.String.Heredoc */
.bw .si { font-weight: bold; font-style: italic } /* Literal.String.Interpol */
.bw .sx { font-style: italic } /* Literal.String.Other */
.bw .sr { font-style: italic } /* Literal.String.Regex */
.bw .s1 { font-style: italic } /* Literal.String.Single */
.bw .ss { font-style: italic } /* Literal.String.Symbol */</style>
<div class="bw"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>emacs</h3>
<style type="text/css">.emacs .hll { background-color: #ffffcc }
.emacs  { background: #f8f8f8; }
.emacs .c { color: #008800; font-style: italic } /* Comment */
.emacs .err { border: 1px solid #FF0000 } /* Error */
.emacs .k { color: #AA22FF; font-weight: bold } /* Keyword */
.emacs .o { color: #666666 } /* Operator */
.emacs .cm { color: #008800; font-style: italic } /* Comment.Multiline */
.emacs .cp { color: #008800 } /* Comment.Preproc */
.emacs .c1 { color: #008800; font-style: italic } /* Comment.Single */
.emacs .cs { color: #008800; font-weight: bold } /* Comment.Special */
.emacs .gd { color: #A00000 } /* Generic.Deleted */
.emacs .ge { font-style: italic } /* Generic.Emph */
.emacs .gr { color: #FF0000 } /* Generic.Error */
.emacs .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.emacs .gi { color: #00A000 } /* Generic.Inserted */
.emacs .go { color: #808080 } /* Generic.Output */
.emacs .gp { color: #000080; font-weight: bold } /* Generic.Prompt */
.emacs .gs { font-weight: bold } /* Generic.Strong */
.emacs .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.emacs .gt { color: #0040D0 } /* Generic.Traceback */
.emacs .kc { color: #AA22FF; font-weight: bold } /* Keyword.Constant */
.emacs .kd { color: #AA22FF; font-weight: bold } /* Keyword.Declaration */
.emacs .kn { color: #AA22FF; font-weight: bold } /* Keyword.Namespace */
.emacs .kp { color: #AA22FF } /* Keyword.Pseudo */
.emacs .kr { color: #AA22FF; font-weight: bold } /* Keyword.Reserved */
.emacs .kt { color: #00BB00; font-weight: bold } /* Keyword.Type */
.emacs .m { color: #666666 } /* Literal.Number */
.emacs .s { color: #BB4444 } /* Literal.String */
.emacs .na { color: #BB4444 } /* Name.Attribute */
.emacs .nb { color: #AA22FF } /* Name.Builtin */
.emacs .nc { color: #0000FF } /* Name.Class */
.emacs .no { color: #880000 } /* Name.Constant */
.emacs .nd { color: #AA22FF } /* Name.Decorator */
.emacs .ni { color: #999999; font-weight: bold } /* Name.Entity */
.emacs .ne { color: #D2413A; font-weight: bold } /* Name.Exception */
.emacs .nf { color: #00A000 } /* Name.Function */
.emacs .nl { color: #A0A000 } /* Name.Label */
.emacs .nn { color: #0000FF; font-weight: bold } /* Name.Namespace */
.emacs .nt { color: #008000; font-weight: bold } /* Name.Tag */
.emacs .nv { color: #B8860B } /* Name.Variable */
.emacs .ow { color: #AA22FF; font-weight: bold } /* Operator.Word */
.emacs .w { color: #bbbbbb } /* Text.Whitespace */
.emacs .mf { color: #666666 } /* Literal.Number.Float */
.emacs .mh { color: #666666 } /* Literal.Number.Hex */
.emacs .mi { color: #666666 } /* Literal.Number.Integer */
.emacs .mo { color: #666666 } /* Literal.Number.Oct */
.emacs .sb { color: #BB4444 } /* Literal.String.Backtick */
.emacs .sc { color: #BB4444 } /* Literal.String.Char */
.emacs .sd { color: #BB4444; font-style: italic } /* Literal.String.Doc */
.emacs .s2 { color: #BB4444 } /* Literal.String.Double */
.emacs .se { color: #BB6622; font-weight: bold } /* Literal.String.Escape */
.emacs .sh { color: #BB4444 } /* Literal.String.Heredoc */
.emacs .si { color: #BB6688; font-weight: bold } /* Literal.String.Interpol */
.emacs .sx { color: #008000 } /* Literal.String.Other */
.emacs .sr { color: #BB6688 } /* Literal.String.Regex */
.emacs .s1 { color: #BB4444 } /* Literal.String.Single */
.emacs .ss { color: #B8860B } /* Literal.String.Symbol */
.emacs .bp { color: #AA22FF } /* Name.Builtin.Pseudo */
.emacs .vc { color: #B8860B } /* Name.Variable.Class */
.emacs .vg { color: #B8860B } /* Name.Variable.Global */
.emacs .vi { color: #B8860B } /* Name.Variable.Instance */
.emacs .il { color: #666666 } /* Literal.Number.Integer.Long */</style>
<div class="emacs"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>vim</h3>
<style type="text/css">.vim .hll { background-color: #222222 }
.vim  { background: #000000; color: #cccccc }
.vim .c { color: #000080 } /* Comment */
.vim .err { color: #cccccc; border: 1px solid #FF0000 } /* Error */
.vim .g { color: #cccccc } /* Generic */
.vim .k { color: #cdcd00 } /* Keyword */
.vim .l { color: #cccccc } /* Literal */
.vim .n { color: #cccccc } /* Name */
.vim .o { color: #3399cc } /* Operator */
.vim .x { color: #cccccc } /* Other */
.vim .p { color: #cccccc } /* Punctuation */
.vim .cm { color: #000080 } /* Comment.Multiline */
.vim .cp { color: #000080 } /* Comment.Preproc */
.vim .c1 { color: #000080 } /* Comment.Single */
.vim .cs { color: #cd0000; font-weight: bold } /* Comment.Special */
.vim .gd { color: #cd0000 } /* Generic.Deleted */
.vim .ge { color: #cccccc; font-style: italic } /* Generic.Emph */
.vim .gr { color: #FF0000 } /* Generic.Error */
.vim .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.vim .gi { color: #00cd00 } /* Generic.Inserted */
.vim .go { color: #808080 } /* Generic.Output */
.vim .gp { color: #000080; font-weight: bold } /* Generic.Prompt */
.vim .gs { color: #cccccc; font-weight: bold } /* Generic.Strong */
.vim .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.vim .gt { color: #0040D0 } /* Generic.Traceback */
.vim .kc { color: #cdcd00 } /* Keyword.Constant */
.vim .kd { color: #00cd00 } /* Keyword.Declaration */
.vim .kn { color: #cd00cd } /* Keyword.Namespace */
.vim .kp { color: #cdcd00 } /* Keyword.Pseudo */
.vim .kr { color: #cdcd00 } /* Keyword.Reserved */
.vim .kt { color: #00cd00 } /* Keyword.Type */
.vim .ld { color: #cccccc } /* Literal.Date */
.vim .m { color: #cd00cd } /* Literal.Number */
.vim .s { color: #cd0000 } /* Literal.String */
.vim .na { color: #cccccc } /* Name.Attribute */
.vim .nb { color: #cd00cd } /* Name.Builtin */
.vim .nc { color: #00cdcd } /* Name.Class */
.vim .no { color: #cccccc } /* Name.Constant */
.vim .nd { color: #cccccc } /* Name.Decorator */
.vim .ni { color: #cccccc } /* Name.Entity */
.vim .ne { color: #666699; font-weight: bold } /* Name.Exception */
.vim .nf { color: #cccccc } /* Name.Function */
.vim .nl { color: #cccccc } /* Name.Label */
.vim .nn { color: #cccccc } /* Name.Namespace */
.vim .nx { color: #cccccc } /* Name.Other */
.vim .py { color: #cccccc } /* Name.Property */
.vim .nt { color: #cccccc } /* Name.Tag */
.vim .nv { color: #00cdcd } /* Name.Variable */
.vim .ow { color: #cdcd00 } /* Operator.Word */
.vim .w { color: #cccccc } /* Text.Whitespace */
.vim .mf { color: #cd00cd } /* Literal.Number.Float */
.vim .mh { color: #cd00cd } /* Literal.Number.Hex */
.vim .mi { color: #cd00cd } /* Literal.Number.Integer */
.vim .mo { color: #cd00cd } /* Literal.Number.Oct */
.vim .sb { color: #cd0000 } /* Literal.String.Backtick */
.vim .sc { color: #cd0000 } /* Literal.String.Char */
.vim .sd { color: #cd0000 } /* Literal.String.Doc */
.vim .s2 { color: #cd0000 } /* Literal.String.Double */
.vim .se { color: #cd0000 } /* Literal.String.Escape */
.vim .sh { color: #cd0000 } /* Literal.String.Heredoc */
.vim .si { color: #cd0000 } /* Literal.String.Interpol */
.vim .sx { color: #cd0000 } /* Literal.String.Other */
.vim .sr { color: #cd0000 } /* Literal.String.Regex */
.vim .s1 { color: #cd0000 } /* Literal.String.Single */
.vim .ss { color: #cd0000 } /* Literal.String.Symbol */
.vim .bp { color: #cd00cd } /* Name.Builtin.Pseudo */
.vim .vc { color: #00cdcd } /* Name.Variable.Class */
.vim .vg { color: #00cdcd } /* Name.Variable.Global */
.vim .vi { color: #00cdcd } /* Name.Variable.Instance */
.vim .il { color: #cd00cd } /* Literal.Number.Integer.Long */</style>
<div class="vim"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>pastie</h3>
<style type="text/css">.pastie .hll { background-color: #ffffcc }
.pastie  { background: #ffffff; }
.pastie .c { color: #888888 } /* Comment */
.pastie .err { color: #a61717; background-color: #e3d2d2 } /* Error */
.pastie .k { color: #008800; font-weight: bold } /* Keyword */
.pastie .cm { color: #888888 } /* Comment.Multiline */
.pastie .cp { color: #cc0000; font-weight: bold } /* Comment.Preproc */
.pastie .c1 { color: #888888 } /* Comment.Single */
.pastie .cs { color: #cc0000; font-weight: bold; background-color: #fff0f0 } /* Comment.Special */
.pastie .gd { color: #000000; background-color: #ffdddd } /* Generic.Deleted */
.pastie .ge { font-style: italic } /* Generic.Emph */
.pastie .gr { color: #aa0000 } /* Generic.Error */
.pastie .gh { color: #303030 } /* Generic.Heading */
.pastie .gi { color: #000000; background-color: #ddffdd } /* Generic.Inserted */
.pastie .go { color: #888888 } /* Generic.Output */
.pastie .gp { color: #555555 } /* Generic.Prompt */
.pastie .gs { font-weight: bold } /* Generic.Strong */
.pastie .gu { color: #606060 } /* Generic.Subheading */
.pastie .gt { color: #aa0000 } /* Generic.Traceback */
.pastie .kc { color: #008800; font-weight: bold } /* Keyword.Constant */
.pastie .kd { color: #008800; font-weight: bold } /* Keyword.Declaration */
.pastie .kn { color: #008800; font-weight: bold } /* Keyword.Namespace */
.pastie .kp { color: #008800 } /* Keyword.Pseudo */
.pastie .kr { color: #008800; font-weight: bold } /* Keyword.Reserved */
.pastie .kt { color: #888888; font-weight: bold } /* Keyword.Type */
.pastie .m { color: #0000DD; font-weight: bold } /* Literal.Number */
.pastie .s { color: #dd2200; background-color: #fff0f0 } /* Literal.String */
.pastie .na { color: #336699 } /* Name.Attribute */
.pastie .nb { color: #003388 } /* Name.Builtin */
.pastie .nc { color: #bb0066; font-weight: bold } /* Name.Class */
.pastie .no { color: #003366; font-weight: bold } /* Name.Constant */
.pastie .nd { color: #555555 } /* Name.Decorator */
.pastie .ne { color: #bb0066; font-weight: bold } /* Name.Exception */
.pastie .nf { color: #0066bb; font-weight: bold } /* Name.Function */
.pastie .nl { color: #336699; font-style: italic } /* Name.Label */
.pastie .nn { color: #bb0066; font-weight: bold } /* Name.Namespace */
.pastie .py { color: #336699; font-weight: bold } /* Name.Property */
.pastie .nt { color: #bb0066; font-weight: bold } /* Name.Tag */
.pastie .nv { color: #336699 } /* Name.Variable */
.pastie .ow { color: #008800 } /* Operator.Word */
.pastie .w { color: #bbbbbb } /* Text.Whitespace */
.pastie .mf { color: #0000DD; font-weight: bold } /* Literal.Number.Float */
.pastie .mh { color: #0000DD; font-weight: bold } /* Literal.Number.Hex */
.pastie .mi { color: #0000DD; font-weight: bold } /* Literal.Number.Integer */
.pastie .mo { color: #0000DD; font-weight: bold } /* Literal.Number.Oct */
.pastie .sb { color: #dd2200; background-color: #fff0f0 } /* Literal.String.Backtick */
.pastie .sc { color: #dd2200; background-color: #fff0f0 } /* Literal.String.Char */
.pastie .sd { color: #dd2200; background-color: #fff0f0 } /* Literal.String.Doc */
.pastie .s2 { color: #dd2200; background-color: #fff0f0 } /* Literal.String.Double */
.pastie .se { color: #0044dd; background-color: #fff0f0 } /* Literal.String.Escape */
.pastie .sh { color: #dd2200; background-color: #fff0f0 } /* Literal.String.Heredoc */
.pastie .si { color: #3333bb; background-color: #fff0f0 } /* Literal.String.Interpol */
.pastie .sx { color: #22bb22; background-color: #f0fff0 } /* Literal.String.Other */
.pastie .sr { color: #008800; background-color: #fff0ff } /* Literal.String.Regex */
.pastie .s1 { color: #dd2200; background-color: #fff0f0 } /* Literal.String.Single */
.pastie .ss { color: #aa6600; background-color: #fff0f0 } /* Literal.String.Symbol */
.pastie .bp { color: #003388 } /* Name.Builtin.Pseudo */
.pastie .vc { color: #336699 } /* Name.Variable.Class */
.pastie .vg { color: #dd7700 } /* Name.Variable.Global */
.pastie .vi { color: #3333bb } /* Name.Variable.Instance */
.pastie .il { color: #0000DD; font-weight: bold } /* Literal.Number.Integer.Long */</style>
<div class="pastie"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>friendly</h3>
<style type="text/css">.friendly .hll { background-color: #ffffcc }
.friendly  { background: #f0f0f0; }
.friendly .c { color: #60a0b0; font-style: italic } /* Comment */
.friendly .err { border: 1px solid #FF0000 } /* Error */
.friendly .k { color: #007020; font-weight: bold } /* Keyword */
.friendly .o { color: #666666 } /* Operator */
.friendly .cm { color: #60a0b0; font-style: italic } /* Comment.Multiline */
.friendly .cp { color: #007020 } /* Comment.Preproc */
.friendly .c1 { color: #60a0b0; font-style: italic } /* Comment.Single */
.friendly .cs { color: #60a0b0; background-color: #fff0f0 } /* Comment.Special */
.friendly .gd { color: #A00000 } /* Generic.Deleted */
.friendly .ge { font-style: italic } /* Generic.Emph */
.friendly .gr { color: #FF0000 } /* Generic.Error */
.friendly .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.friendly .gi { color: #00A000 } /* Generic.Inserted */
.friendly .go { color: #808080 } /* Generic.Output */
.friendly .gp { color: #c65d09; font-weight: bold } /* Generic.Prompt */
.friendly .gs { font-weight: bold } /* Generic.Strong */
.friendly .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.friendly .gt { color: #0040D0 } /* Generic.Traceback */
.friendly .kc { color: #007020; font-weight: bold } /* Keyword.Constant */
.friendly .kd { color: #007020; font-weight: bold } /* Keyword.Declaration */
.friendly .kn { color: #007020; font-weight: bold } /* Keyword.Namespace */
.friendly .kp { color: #007020 } /* Keyword.Pseudo */
.friendly .kr { color: #007020; font-weight: bold } /* Keyword.Reserved */
.friendly .kt { color: #902000 } /* Keyword.Type */
.friendly .m { color: #40a070 } /* Literal.Number */
.friendly .s { color: #4070a0 } /* Literal.String */
.friendly .na { color: #4070a0 } /* Name.Attribute */
.friendly .nb { color: #007020 } /* Name.Builtin */
.friendly .nc { color: #0e84b5; font-weight: bold } /* Name.Class */
.friendly .no { color: #60add5 } /* Name.Constant */
.friendly .nd { color: #555555; font-weight: bold } /* Name.Decorator */
.friendly .ni { color: #d55537; font-weight: bold } /* Name.Entity */
.friendly .ne { color: #007020 } /* Name.Exception */
.friendly .nf { color: #06287e } /* Name.Function */
.friendly .nl { color: #002070; font-weight: bold } /* Name.Label */
.friendly .nn { color: #0e84b5; font-weight: bold } /* Name.Namespace */
.friendly .nt { color: #062873; font-weight: bold } /* Name.Tag */
.friendly .nv { color: #bb60d5 } /* Name.Variable */
.friendly .ow { color: #007020; font-weight: bold } /* Operator.Word */
.friendly .w { color: #bbbbbb } /* Text.Whitespace */
.friendly .mf { color: #40a070 } /* Literal.Number.Float */
.friendly .mh { color: #40a070 } /* Literal.Number.Hex */
.friendly .mi { color: #40a070 } /* Literal.Number.Integer */
.friendly .mo { color: #40a070 } /* Literal.Number.Oct */
.friendly .sb { color: #4070a0 } /* Literal.String.Backtick */
.friendly .sc { color: #4070a0 } /* Literal.String.Char */
.friendly .sd { color: #4070a0; font-style: italic } /* Literal.String.Doc */
.friendly .s2 { color: #4070a0 } /* Literal.String.Double */
.friendly .se { color: #4070a0; font-weight: bold } /* Literal.String.Escape */
.friendly .sh { color: #4070a0 } /* Literal.String.Heredoc */
.friendly .si { color: #70a0d0; font-style: italic } /* Literal.String.Interpol */
.friendly .sx { color: #c65d09 } /* Literal.String.Other */
.friendly .sr { color: #235388 } /* Literal.String.Regex */
.friendly .s1 { color: #4070a0 } /* Literal.String.Single */
.friendly .ss { color: #517918 } /* Literal.String.Symbol */
.friendly .bp { color: #007020 } /* Name.Builtin.Pseudo */
.friendly .vc { color: #bb60d5 } /* Name.Variable.Class */
.friendly .vg { color: #bb60d5 } /* Name.Variable.Global */
.friendly .vi { color: #bb60d5 } /* Name.Variable.Instance */
.friendly .il { color: #40a070 } /* Literal.Number.Integer.Long */</style>
<div class="friendly"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
<h3>native</h3>
<style type="text/css">.native .hll { background-color: #404040 }
.native  { background: #202020; color: #d0d0d0 }
.native .c { color: #999999; font-style: italic } /* Comment */
.native .err { color: #a61717; background-color: #e3d2d2 } /* Error */
.native .g { color: #d0d0d0 } /* Generic */
.native .k { color: #6ab825; font-weight: bold } /* Keyword */
.native .l { color: #d0d0d0 } /* Literal */
.native .n { color: #d0d0d0 } /* Name */
.native .o { color: #d0d0d0 } /* Operator */
.native .x { color: #d0d0d0 } /* Other */
.native .p { color: #d0d0d0 } /* Punctuation */
.native .cm { color: #999999; font-style: italic } /* Comment.Multiline */
.native .cp { color: #cd2828; font-weight: bold } /* Comment.Preproc */
.native .c1 { color: #999999; font-style: italic } /* Comment.Single */
.native .cs { color: #e50808; font-weight: bold; background-color: #520000 } /* Comment.Special */
.native .gd { color: #d22323 } /* Generic.Deleted */
.native .ge { color: #d0d0d0; font-style: italic } /* Generic.Emph */
.native .gr { color: #d22323 } /* Generic.Error */
.native .gh { color: #ffffff; font-weight: bold } /* Generic.Heading */
.native .gi { color: #589819 } /* Generic.Inserted */
.native .go { color: #cccccc } /* Generic.Output */
.native .gp { color: #aaaaaa } /* Generic.Prompt */
.native .gs { color: #d0d0d0; font-weight: bold } /* Generic.Strong */
.native .gu { color: #ffffff; text-decoration: underline } /* Generic.Subheading */
.native .gt { color: #d22323 } /* Generic.Traceback */
.native .kc { color: #6ab825; font-weight: bold } /* Keyword.Constant */
.native .kd { color: #6ab825; font-weight: bold } /* Keyword.Declaration */
.native .kn { color: #6ab825; font-weight: bold } /* Keyword.Namespace */
.native .kp { color: #6ab825 } /* Keyword.Pseudo */
.native .kr { color: #6ab825; font-weight: bold } /* Keyword.Reserved */
.native .kt { color: #6ab825; font-weight: bold } /* Keyword.Type */
.native .ld { color: #d0d0d0 } /* Literal.Date */
.native .m { color: #3677a9 } /* Literal.Number */
.native .s { color: #ed9d13 } /* Literal.String */
.native .na { color: #bbbbbb } /* Name.Attribute */
.native .nb { color: #24909d } /* Name.Builtin */
.native .nc { color: #447fcf; text-decoration: underline } /* Name.Class */
.native .no { color: #40ffff } /* Name.Constant */
.native .nd { color: #ffa500 } /* Name.Decorator */
.native .ni { color: #d0d0d0 } /* Name.Entity */
.native .ne { color: #bbbbbb } /* Name.Exception */
.native .nf { color: #447fcf } /* Name.Function */
.native .nl { color: #d0d0d0 } /* Name.Label */
.native .nn { color: #447fcf; text-decoration: underline } /* Name.Namespace */
.native .nx { color: #d0d0d0 } /* Name.Other */
.native .py { color: #d0d0d0 } /* Name.Property */
.native .nt { color: #6ab825; font-weight: bold } /* Name.Tag */
.native .nv { color: #40ffff } /* Name.Variable */
.native .ow { color: #6ab825; font-weight: bold } /* Operator.Word */
.native .w { color: #666666 } /* Text.Whitespace */
.native .mf { color: #3677a9 } /* Literal.Number.Float */
.native .mh { color: #3677a9 } /* Literal.Number.Hex */
.native .mi { color: #3677a9 } /* Literal.Number.Integer */
.native .mo { color: #3677a9 } /* Literal.Number.Oct */
.native .sb { color: #ed9d13 } /* Literal.String.Backtick */
.native .sc { color: #ed9d13 } /* Literal.String.Char */
.native .sd { color: #ed9d13 } /* Literal.String.Doc */
.native .s2 { color: #ed9d13 } /* Literal.String.Double */
.native .se { color: #ed9d13 } /* Literal.String.Escape */
.native .sh { color: #ed9d13 } /* Literal.String.Heredoc */
.native .si { color: #ed9d13 } /* Literal.String.Interpol */
.native .sx { color: #ffa500 } /* Literal.String.Other */
.native .sr { color: #ed9d13 } /* Literal.String.Regex */
.native .s1 { color: #ed9d13 } /* Literal.String.Single */
.native .ss { color: #ed9d13 } /* Literal.String.Symbol */
.native .bp { color: #24909d } /* Name.Builtin.Pseudo */
.native .vc { color: #40ffff } /* Name.Variable.Class */
.native .vg { color: #40ffff } /* Name.Variable.Global */
.native .vi { color: #40ffff } /* Name.Variable.Instance */
.native .il { color: #3677a9 } /* Literal.Number.Integer.Long */</style>
<div class="native"><pre><span class="c">#!/usr/bin/python3</span>

<span class="kn">from</span> <span class="nn">engine</span> <span class="k">import</span> <span class="n">RunForrestRun</span>

<span class="sd">&quot;&quot;&quot;Test code for syntax highlighting!&quot;&quot;&quot;</span>

<span class="k">class</span> <span class="nc">Foo</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">var</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">var</span> <span class="o">=</span> <span class="n">var</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">run</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">RunForrestRun</span><span class="p">()</span>  <span class="c"># run along!</span>
</pre></div>

<br/>
</body></html>


{% endcapture %}

{% include toggle.html toggle-name="code-snippet" button-text="View all Color scheme" toggle-text=text-capture  %}



# Troubleshooting (Extra)
* Command not found with brew and/or python

Make sure that you have installed successfully, try `source ~/.zshrc` to refresh your Shell config file or simply _open a new window_.

* Command not found with Pygments

This is usually due to inproper installation of multiple Python version on your machine.
Run
```bash
brew doctor && brew prune
```
to eliminate any empty keg and un-link unused dependencies.

Also, install Pygments with Python `pip` and **NOT** to be confuse with brew.

---- 

# Disclaimer
> This article offers knowledge and personal opinion based on the author’s point of view which perceived at the time of writing this article. 
> 
> It is intended to serve “as-is” without any guarantee to work, proceed with caution. Your inputs and feedbacks are welcome as always.


[1]:	https://www.python.org/downloads/mac-osx/

[image-1]:	/assets/blogs/2018-12-23-Quick-Service-to-copy-code-format-and-reserve-syntax-highlight/array.jpg
[image-2]:	/assets/blogs/2018-12-23-Quick-Service-to-copy-code-format-and-reserve-syntax-highlight/services.jpg
[image-3]:	/assets/blogs/2018-12-23-Quick-Service-to-copy-code-format-and-reserve-syntax-highlight/step1.jpg
[image-4]:	/assets/blogs/2018-12-23-Quick-Service-to-copy-code-format-and-reserve-syntax-highlight/step2.jpg
[image-5]:	/assets/blogs/2018-12-23-Quick-Service-to-copy-code-format-and-reserve-syntax-highlight/step3.jpg
[image-6]:	/assets/blogs/2018-12-23-Quick-Service-to-copy-code-format-and-reserve-syntax-highlight/step4.jpg
