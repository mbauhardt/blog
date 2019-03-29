---
title: "Implement Solarized Color Theme For Jekyll Bootstrap"
date: 2013-03-17
tags: ["jekyll-bootstrap", "solarized"]
---

I'm a big fan of the [solarized color scheme](http://ethanschoonover.com/solarized). I use solarized dark for 

* [iTerm](http://www.iterm2.com/) in combination with my [fork](https://gist.github.com/3806415) of the agnoster [ohmyzsh theme](https://github.com/robbyrussell/oh-my-zsh).
* [Emacs](http://emacsformacosx.com/)
* [Eclipse](http://www.eclipse.org/)

And now i implemented some lines of css which provides the solarized dark color theme for my blog, created with jekyll bootstrap.
I created a file called _solarized-dark.css_.

    /*
    base03    #002b36
    base02    #073642
    base01    #586e75
    base00    #657b83
    base0     #839496
    base1     #93a1a1
    base2     #eee8d5
    base3     #fdf6e3

    yellow    #b58900
    orange    #cb4b16
    red       #dc322f
    magenta   #d33682
    violet    #6c71c4
    blue      #268bd2
    cyan      #2aa198
    green     #859900


    base0:  primary content
    base1:  optional emphasized content
    base01: secondary content

    base02: background highlights
    base03: background
    */

    body {
      background-color: #002b36;
      color: #839496;
      font-family: Menlo;
    }

    pre {
      border-style:solid;
      border-width:1px;
      border-color:#839496;
    }

    pre, code {
      background-color: #073642;
      color: #93a1a1;
    }

    a {
      color: #859900;
    }

    .navbar .brand {
      text-shadow: 0 0 0 #000;
    }

    .navbar .nav>li>a {
      text-shadow: 0 0 0 #000;
    }

    .navbar-inner {
      background-color: #073642;
      background-image: none;
      background-repeat: no-repeat;
      filter: none;
      border-style: none;
    }

    .tag_box a {
      background-color: #b58900;
      border:none;
    }

This file is included into the _default.html_ file.

    <link href="css/solarized-dark.css" rel="stylesheet">

The file for solarized dark can be downloaded [here](https://raw.github.com/mbauhardt/jekyll-bootstrap/mbauhardt.github.com/assets/themes/twitter/css/solarized-dark.css).
The solarized light file [here](https://raw.github.com/mbauhardt/jekyll-bootstrap/mbauhardt.github.com/assets/themes/twitter/css/solarized-light.css).
