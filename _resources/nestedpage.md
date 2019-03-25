---
layout: nested
title: Nested Page
description: How to configure your nested pages and content pages
---

# Nested Pages

As mentioned on the landing page, we created a template that allows you to structure your content in a more website like way, in a nested structure.
This is done by sticking to a certain structure and adding certain bits of information to your front matter.

## Structure

Each nested page is usually represented through an additional [collection ](https://jekyllrb.com/docs/collections/) and as an item in the navigation bar.
The root of your nested page is a .md file with layout *nested* and must be named exactly as the collection. 
In our case we created a collection *"documentation"*, created a folder *"_documentation"* which is demanded by Jekyll and created an *"documentation.md"* file in the root of our site directory, as depicted below:

*_config.yml:*
```
... (other stuff)

collections:
  documentation:
    output: true
	
... (other stuff)
```

*Directory structure:*
```
root
 |
  -- ... (other stuff)
 |
  -- _documentation
     |
	  -- nestedpage.md
	 |
	  -- landingpage.md
	 |
	  -- nesteddemo.md 
	 |
	  -- nesteddemo
	     |
		  -- subchapter_1.md 
		 |
		  -- subchapter_2.md
		 |
		  -- subchapter_3.md
```

*documentation.md:*
```
---
layout: nested
title: Documentation
navbaritem: true
subfolders:
  - 'nesteddemo'
files:
  - 'landingpage'
  - 'nestedpage'
---

content ... bla bla bla
```

## Front Matter

### Navbaritem

The **navbaritem** property is not used by the *nested* template, but by the header template which generates the navbar. If it is set to true a navbar entry is created with the **title** property as content.
This works only for .md files that are in the root directory of your page.

### Subfolder and Files

As the names imply these properties tell the template where to search for subfolders or files in the directory.

**Subfolders**
The *documentation.md* file lies within the *root* directory of your site. The path given for its subfolders is relative to the *_documentation* subfolder, **NOT** relative to the .md file!
For each subfolder in **subfolders** there must be a equally named .md file. 
In our case *documentation.md* points to the subfolder *nesteddemo* which is represented by a folder *nesteddemo* within the *_documentation* folder and a .md file *nesteddemo.md*, also lying within the *_documentation* folder.
The point of *nesteddemo.md* is to again point at the **subfolders** and **files** within the *nesteddemo* folder. 
In our case *nesteddemo.md* has a front matter that looks as depicted below:

*nesteddemo.md:*
```
---
layout: nested
title: Nested Demo
description: A demonstration of nested sites
files:
  - 'subchapter_1'
  - 'subchapter_2'
  - 'subchapter_3'
---

content ... bla bla bla
```

This points to the files *subchapter_1.md*, *subchapter_2.md* and *subchapter_3.md* which reside within the subfolder *nesteddemo*.
This structure can be enlargened to arbitrarily nested folders/files.