---
layout: nested
title: Landing Page
description: How to change the content of your landing page
---

# Sections of the Landing Page

The landing page consists of three differen sections:

* Heroheader
* Peptalks
* News

Each section can be edited through separate .md files with a respective layout in their front matter. 

## Heroheader

The Heroheader is an eye-catcher on every modern website. Its purpose is to provide the visitor with the most basic of information about your project:

* What is the name of your project?
* What is its purpose?
* Where do I have to go to learn more about it?

Therefore in your heroheader.md file you only provide these bits of information in the front matter:

```
---
layout: heroheader
button:
  text: GET STARTED!
  url: https://github.com/Sandared/documentation_template
title: Project Documentation Template
subtitle: The Easy Way Towards Project Documentation
version: 1.0.0
---
```

Actually the properties are rather selfexplanatory. The content of this file is never used, it merely provides the front matter information neccessary to create the header.

## Peptalks

Peptalks are usually coming directly under the Heroheader. The user has read your Heroheader, but is still not completely convinced. Now it's time to show your project at its best! 
Take the most convincing properties of your project and spice them up with a pinch of Buzzwords and a small dose of exageration ;)
Usually you have a multiple of three. For each peptalk you create one .md file which has a front matter like this:

```
---
layout: peptalk
icon: flash_on
title: Speeds up development
---
```

The **icon** property can be chosen from the set of [material icons](https://material.io/icons/) provided by Google.
The content is used as the body of your Peptalk. Don't be too short but also don't try to explain your whole project.

## News

Now you've nearly convinced your visitor that you are the best project on earth, whether or not your visitor needs what you provide, but one question is still unanswered: 
Is this project up-to-date, relevant and still active?
Well, what could be better than a News section bursting with your newly achieved successes, hurdled problems and newly won partners?

Each news item is a single .md file with a header like the one below:

```
---
layout: news
title: Some unbelievable news!!!
author: Jane Doe
published_at: 2017-09-05
---
```

The news items are sorted in descending order according to their **published_at** attribute. This means the newest news item is displayed at the top.


## Additional Section Ideas

Maybe in the near future I am able to add sections like:

- partners: a collection of company logos of famous companies that use your product
- pitch: a section containing a video of psyched users of your project. All of them completely overwhelmed by how your project easened their everyday life.
- ???