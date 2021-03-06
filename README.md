# markdown-subtemplate [![](https://img.shields.io/badge/python-3.6+-blue.svg)](https://www.python.org/downloads/release/python-3610/) [![](https://img.shields.io/github/license/ResidentMario/missingno.svg)](https://github.com/ResidentMario/missingno/blob/master/LICENSE.md)

A template engine to render Markdown with external template imports and basic variable replacements.

## Motivation

We often make a choice between data-driven server apps (typical Flask app), CMSes that let us edit content on the web such as WordPress, and even flat file systems like Pelican.

These are presented as an either-or. You either get a full database driven app or you get a CMS, but not both. This project is meant to help add CMS like features to your data-driven web apps and even author them as static markdown files.

Here's how it works:

1. You write standard markdown files for content.
2. Markdown files can be shared and imported into your top-level markdown.
3. Fragments of HTML can be used when css classes and other specializations are needed, but generally HTML is avoided.
4. A dictionary of variables and their values to replace in the merged markdown is processes.
5. Markdown content is converted to HTML and embedded in your larger site layout (e.g. within a Jinja2 template).
6. Markdown transforms are cached to achieve very high performance regardless of the complexity of the content.

## Standard workflow

Write markdown content, merge it with other markdown files, deliver it as HTML as part of your larger site.

![](readme_resources/workflow_image_layout.png)

## Usage

To use the library, simply install it. Then:

Write a markdown template, `page.md`:

```markdown
## This is a sub-title

* Here's an entry
* And another
```

Register the template engine in your web app startup:

```python
from markdown_subtemplate import engine
# Set the template folder so that when you ask for page.md 
# the system knows where to look.

engine.set_template_folder(full_path_to_template_folder)
```

Then generate the HTML content via:

```python
data = {'variable1': 'Value 1', 'variable2': 'Value 2'}
contents = engine.get_page('page.md', data)
```

Finally, pass the HTML fragment to be rendered in the larger page context:

```python
# A Pyramid view method:

@view_config(route_name='landing', renderer='landing.pt')
def landing(request):
    data = {'variable1': 'Value 1', 'variable2': 'Value 2'}
    contents = engine.get_page('page.md', data)

    return {
        'name': 'Project name',
        'contents': contents
    }
```

And the larger website template grabs the content and renders it, `landing.pt`:

```html
...
<div>
    ${structure:contents}
</div>
...
```

## Beware the danger!

This library is meant for INTERNAL usage only. It's to help you add CMS features to your app. It is **not** for taking user input and making an forum or something like that.

To allow for the greatest control, you can embed small fragments of HTML in the markdown (e.g. to add a CSS class or other actions). This means the markdown is processed in **UNSAFE** mode. It would allow for script injection attacks if opened to the public.

## Installation

Until we publish to PyPI, you'll need to install from Github using this command. We intend to make it available via PyPI, just want to finalize the API a little more first.

Install:

```bash
pip install git+git://github.com/mikeckennedy/markdown-subtemplate
```
 
 
## Nested markdown

One of the reason's this project exists, rather than just passing markdown text to a markdown library is allowing nesting / importing of markdown files.

If you have page fragments that need to appear more than once, create a dedicated markdown import file that can be managed and versioned in one place. Here's how:

### Created an imported file in TEMPLATES/_shared

All imported markdown files are located in subpaths of `TEMPLATES/_shared` where `TEMPLATES` is the path you set during startup.

```
TEMPLATES
    |- _shared
        |- contact.md
        |- footer.md
    |-pages
        | - page.md
        | - about.md
```

Write the imported / shared markdown, `contact.md`:

```markdown
Contact us via email [us@us.com](mailto:us.com) or on 
Twitter via [@us](https://twitter.com/us)
```

Then in your page, e.g. `page.md` you can add an import statement:

```markdown
# Our amazing page

Here is some info **about the page**. It's standard markdown.

Want to contact us? Here are some options:
[IMPORT CONTACT]

And a footer:
[IMPORT FOOTER]
```

The resulting markdown is just replacing the `IMPORT` statements with the contents of those files, then passing the whole thing through a markdown to HTML processor.

## Variables

`markdown_subtemplate` has some limited support for variable replacements. Given this markdown page:

```markdown
# Example: $TITLE$

Welcome to the $PROJECT$ project. Here are some details 
...
```

You can populate the variable values with:

```python
data = {'title': 'Markdown Transformers', 'project': 'sub templates'}
contents = engine.get_page('page.md', data)
```

Note that the variable names must be all-caps in the template. Missing variable statements in markdown that appear in the data dictionary are ignored.

## Requirements

This library requires **Python 3.6 or higher**. Because, *f-yes*! (f-strings).

## Licence

`markdown-subtemplate` is distributed under the MIT license.

## Authors

`markdown_subtemplate` was written by [Michael Kennedy](https://github.com/mikeckennedy).
