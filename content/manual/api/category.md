Title: Category object
Path-Alias: /api/category
Date: 2018-04-25 15:02:41-07:00
Entry-ID: 170
UUID: 81cbd7d2-55c5-48d5-b27c-29b8b6d8a3b0

Template API for categories.

.....

The `category` object provides the following:

* **`path`**: The full path to the category

* **`basename`**: Just the last part of the category path

* **`name`**: A display name for the category.

    This defaults to taking the category basename, converting `_`s to spaces, and then
    title-casing the name. For example, the category `some/long/category_name` will have
    a default name of `"Category Name"`. This can be overridden using a [meta file](#meta-files).

* **`subcats`**: The subcategories of this category. Takes the following argument:

    * **`recurse`**: Whether to include the subcategories of the subcategories, and their subcategories
        and so on. Possible values:

        * **`False`**: Only include direct subcategories (default)
        * **`True`**: Include all subcategories

* **`parent`**: The parent category, if any

* **`link`**: The link to the category; optionally takes the
    following arguments:

    * **`template`**: Which template to use when rendering the category
    * **`absolute`**: Whether to format this as an absolute or relative URL
        * **`False`**: Use a relative link (default)
        * **`True`**: Use an absolute link

* **`first`**: Returns the first entry in this category; optionally takes [view arguments](/api#get-view)
* **`last`**: Returns the last entry in this category; optionally takes [view arguments](/api#get-view)

* **`get`**: Get a header defined in the [meta file](#meta-files)
* **`get_all`**: Get all instances of a meta file header as a list

* **`description`**: Get the category description from the [meta file](#meta-files), if it exists. This optionally
    takes arguments for [image renditions](/image-renditions) if so desired.

* **`sort_name`**: The name used for sorting

* <span id="breadcrumb"></span>**`breadcrumb`**: A list of the categories that lead to this one, as `category` objects, including the current one.

* <span id="root"></span>**`root`**: The root category of the blog, useful for storing site-level metadata (such as the site name or global configuration).

    This is roughly equivalent to `category.breadcrumb[0]`, but is slightly more efficient and easier to type.

* The following properties are also available but probably aren't of use to template authors, and are only listed for the sake of completion. You should not rely on them for anything as they might change without warning.

    * `file_path`: The file path of the category's metadata file

    * `aliases`: The various registered path aliases for this category

Example template code for printing out an entire directory structure (flattened):

```jinja
<ul>
{% for subcat in category.subcats(recurse=True) %}
<li>{{subcat.path}}: {{subcat.name}}</li>
{% endfor %}
</ul>
```

Example template code for printing out the directory structure in a nice recursive manner:

```jinja
<ul>
{% for subcat in category.subcats recursive %}
    <li>{{ subcat.basename }}: {{ subcat.name }}
    {% if subcat.subcats %}
    <ul>{{ loop(subcat.subcats)}}</ul>
    {% endif %}</li>
{% endfor %}
</ul>
```

Categories can also be compared to other categories, which is useful for determining if a subcategory is the same as this one. This snippet will show all of the categories on the website, with the currently-visible one getting a class of `here` and any parent categories getting a class of `parent`:

```jinja
<ul>
{% for subcat in category.root.subcats(recurse=True) %}
{% if subcat == category %}
<li class="here">{{category.path}}</li>
{% elseif subcat in category.breadcrumb %}
<li class="parent"><a href="{{category.link}}">{{category.path}}</a></li>
{% else %}
<li><a href="{{category.link}}">{{category.path}}</li>
{% endif %}
{% endfor %}
</ul>
```

## <span id="meta-files">Meta files</span>

Category data can be added using a metadata file, which is simply a file whose name ends in `.cat` or `.meta` and lives somewhere
in the content directory. Similarly to [entry files](/entry-format), this will default to using its place in the
content directory for the category to use, but this can also be overridden using a `Category:` header.

The headers supported by Publ itself:

* **`Category`**: Specifies which category this file refers to
* **`Name`**: Overrides the friendly/display name of the category (i.e. `category.name`)
* <span id="template-override">**`Index-Template`**</span>: Use this template instead of `index` when rendering this category (useful if you want to override a category or site's index template without overriding its subcategories)
* **`Entry-Template`**: Use this template instead of `entry` when rendering an entry
* **`Sort-Name`**: The name to use when sorting this in a subcategory list

For example, given the file `some/category/info.cat`:

```
Name: Random Category
```

it will set the `name` attribute of the `some/category` category to `"Random Category"`. However, if the file contains:

```
Name: Specific Category
Category: other/category
```

then it will set the `name` attribute of the `other/category` category to `"Specific Category"`.

Similarly to entries, this also supports the `Path-Alias` header; for example:

```
Category: /art/sketchbook
Path-Alias: /comics/cat-sketchbook.php archive
```

will redirect requests to `/comics/catch-sketchbook.php` to the `/art/sketchbook` category using the `archive` template.

You can also define any other arbitrary values you like, which will then be put on the category object, and which are
accessible directly via `get` and `get_all`. The behavior is the same as on [entry files](/entry-format).

Any text after the headers will be treated as the `description` for the category. Markdown is supported.

Note that at least one header is required. If you just want to provide a description and no other configuration,
you can put a bogus Description header in, which will be overridden by the actual description. For example:

```
Description: asdf

Here is a description with no real configuration.
```

If you want to change the order of subcategories in a subcategory list, that is easy to do with the `Sort-Name` property; for example, if you have categories named `foo`, `bar`, and `baz` which you want to appear in that order, you can do something like:

    Sort-Name: 1-foo

    Sort-Name: 2-bar

    Sort-Name: 3-baz

in their respective categories.
