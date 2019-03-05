---
layout: nested
title: Site Configuration
description: How to configure site properties of this template
---

# How to Configure this Template?

Every Jekyll Template is configured through 

* its *_config.yml* file 
* *front matter* in each page

First let's have a look at the possible configuration properties offered by this template:

## Standard Stuff

Virtually any template offers stuff like social media buttons, github buttons, a site name, a logo and stuff like this.
This can be configured through the following properties:

```
---
title: PDT
author: Thomas Driessen
email: thomas.driessen.td@gmail.com
description: A Project Documentation Template for Jekyll

# language to use in html tag, can be overriden by page.lang
# lang: en 

sass:
    style: compressed

# path to logo without leading slash e.g. images/logo.png or just the file name if it is in the root directory
logo: logo.png

# Github data, if present there will be a GitHub button in the navbar
github_button:
  user: Sandared
  project: documentation_template

# Twitter data, if present there will be a Twitter button in the navbar
twitter_button:
  user: SanfteSchorle
---
```

All of these properties are selfexplanatory or are explained through the comments in the *_config.yml* file.

## Colors

As mentioned on our landing page, this template provides a fully adaptable color scheme. 
We ourselves have been frustrated to adapt each template to our needs when it comes to colors, so we provide a simple mechanism to change the complete look and feel of the whole site:

```
---
# You can look up the color names at http://materializecss.com/color.html
colors:
    primary_dark: black
    primary_light: grey-darken-1
    primary: grey-darken-3
    accent: light-blue
    primary_text: grey-darken-4
    secondary_text: grey-darken-1
    inverse_text: white
    divider: grey-lighten-1
    secondaryBackground: grey-lighten-4
---
```

Usually you will only change the first 4 colors:

* primary_dark
* pimary_light
* primary
* accent 

The predefined colors can be found on the [Materialize CSS Color Page](http://materializecss.com/color.html). Usually you choose one color as primary and then go up and down 2-3 steps for the primary_dark and primary_light. 
Choose the accent color as you wish.
Be careful if you choose very light colors, you might have to adapt the secondaryBackground, as it is used for code highlighting. 

### Primary_Dark

This color is used rather sparse throughout the website. Only the Heroheader and the News badges are currently making use of it.

### Primary_Light

This color is used in 

* Heroheader
* Newsitem badges
* News Header
* Links
* Code Highlighting (very often)
* Breadcrumb Navigation
* Table of Contents

### Primary 

This color is used in 

* Headers
* Navbars
* Newsitem badges 
* Code Highlighting (very often)
* Cards

### Accent

This color is used in 

* Link::hover
* Page Footer
* Table of Contents
* Heroheader button
* Blockquotes
* Breadcrumbs
* Code Highlighting

We recommend to play aroud with these colors to create a site that looks good to you. 
Remember that you have to restart the Jekyll Server each time you change the colors, as the configuration file is only readat startup.

## Additional Collections

Well, this isn't directly a feature provided by us, but by Jekyll. We merely enhanced it a little bit.

```
---
# any of these collections is used for nested pages
collections:
  landingpage:
    output: true
  documentation:
    output: true
---
```

Any collection added to the collections section is searched by our *nested* template for pages it can display in a nested way.