# Pico Pagination Plugin

Provides basic pagination for [Pico](http://pico.dev7studios.com).

## Changelog

**1.7** - Changed date sorting to allow the use of any PHP valid date format
**1.6** - Fixed a sorting issue when using subfolders.
**1.5** - Added `next_page_url` and `prev_page_url` variables for Twig. Preparing to stop returning markup from the plugin in a future release—plugin should only return data and leave theming up to site builders.
**1.4** - Changed to use of `&gt;/&lt;` in default next_text and prev_text variables and added more variables Google search-like numbered pagination.  
**1.3** - Added ablity for pagination to happen in subfolders on a site.  
**1.2** - Added back the ability to set the path segment to indicate pages ("page indicator" ) now that changes have been made in Pico v0.8 that allow it to work properly.  
**1.1** - Added ability to reverse the order in which the previous/next links are rendered.  
**1.0** - Initial release.

## How it works

It's pretty simple. The plugin creates a second array of pages identical to the default `pages` called `paged_pages`. This second array is filtered by date (so only blog-type posts will be returned) and limited by the number of pages you want per page. Listing your paged results, therefore, is just a matter of iterating through the `paged_pages` variable instead of the `pages` one.

Additional variables that can be used in themes are created to accompany the paged results. See below.

## Installation

1. Copy the pagination.php file to the plugins folder of your Pico site.
2. Update your theme to use the paginated pages generated by the plugin.
3. Set configuration variables if defaults are not suitable.
4. That's it!

## Configuration settings

You can configure a number of settings by adding values to your site's `config.yml` file. The following are the available options and what they do.

| Option | Default | Options | Notes  |
| ------ | ------- | ------- | ------ |
| `pagination_limit` | int: `5` | n/a | Sets the how many items display on each page. |
| `pagination_page_indicator` | str: `"page"` | n/a | Sets the word used in the URL that will indicate paged results. (i.e. http://yoursite.com/page/2) |
| `pagination_filter_date` | bool: `true` | `true`, `false` | Sets whether the posts returned should be filtered to only those with dates or not.    | Options: true, false
| `pagination_next_text`* | str: `"next >"` | n/a | Sets the text for the next link. |
| `pagination_prev_text`* | str: `"< previous"` | n/a | Sets the text for the previous links. |
| `pagination_flip_links`* | bool: `false` | `true`, `false` | Reverses the order the links are ouput. This is to aid in providing links in the format of older/newer as opposed to previous/next. |
| `pagination_output_format`* | str: `"links"` | `"links"`, `"list"` | Sets whether `{{ pagination_links }}` will output two `<a>` tags or an unordered list. |
| `pagination_sub_page` | bool: `false` | `true`, `false` | Sets whether there is a sub page for the pagination (i.e. not the root of the site). When this is set to true, you must create a subfolder in content with the same name as the "pagination_page_indicator" variable. See below for further information. |

For reference, these values are set in config.yml using the following format (I'd place them toward the bottom in the "Custom" section):

*In a future release of Pico-Pagination, the plugin will stop returning markup and only return values. At that point, these config options will go away.

```yml
pagination_limit: 10
pagination_output_format: "list"
```

## Setting up the theme

The plugin adds a couple new variables to the theme. They are:

- `{{ paged_pages }}` - The new, paged array of pages for the theme
- `{{ next_page_url }}` - The URL for the next page, empty string if none
- `{{ prev_page_url }}` - The URL for the previous page, empty string if none
- `{{ page_number }}` - The current page number
- `{{ total_pages }}` - The total number of pages
- `{{ page_of_page }}` - The page number out of the total number of pages (i.e. "Page 1 of 3", as a string)
- `{{ page_indicator }}` - The `page_indicator` value from config, usually "page"
- ~~`{{ pagination_links }}`~~** - The generated next and previous links (output as either two links or an unordered list)
- ~~`{{ next_page_link }}`~~** - The next link only. Returns empty string if there isn't one
- ~~`{{ prev_page_link }}`~~** - The previous link only. Returns empty string if there isn't one

**This variable returns HTML so it must be used in Twig with the `|raw` filter. These will be deprecated at some point in the future as the plugin should only return _data_ instead of _markup_—that's _your_ job in your theme 😉.

To get a basic implemenation of the pagination plugin going, use something like the following:

```twig
{% if is_front_page %}
{# Front page, list all blog posts #}
    
  <div class="posts-list">
  {% for page in paged_pages %}
    <div class="post">
      <h3><a href="{{ page.url }}">{{ page.title }}</a></h3>
      <p class="meta">{{ page.date_formatted }}</p>
      <p class="excerpt">{{ page.excerpt }}</p>
    </div>
  {% endfor %}
  </div>

  {# Pagination here 👇 #}
  {% if prev_page_url or next_page_url %}
    <ul class="pagination-links">
      {% if prev_page_url %}
        <li class="pagination-list-item">
          <a href="{{ prev_page_url }}" class="pagination-link pagination-link--prev">&larr; Previous Page</a>
        </li>
      {% endif %}
      {% if next_page_url %}
        <li class="pagination-list-item">
          <a href="{{ next_page_url }}" class="pagination-link pagination-link--next">Next Page &rarr;</a>
        </li>
      {% endif %}
    </ul>
  {% endif %}
  <p class="pagination-summary">{{ page_of_page }}</p>
  {# Pagination done #}
  
{% else %}
{# Other pages show individual post #}

  <div class="post">
    {% if meta.title %}<h2>{{ meta.title }}</h2>{% endif %}
    <p class="meta">{{ meta.date_formatted }}</p>
    {{ content }}
  </div>

{% endif %}
```

**To note:** the key difference here is that the Pico standard posts loop is iterating through *paged_pages* instead of *pages*. (This sample is taken [from the Pico](http://pico.dev7studios.com/docs.html#blogging) website and modified for the pagination plugin.)

### Your Site Navigation

If you're familiar with Pico, this may be obvious, but for your main site menu, you can filter it so that content with dates (i.e. your blog posts) don't get output along with your other pages. This is pretty easy to do just with a little if statement. Here's an example:

```twig
<ul class="nav">
  {% for page in pages %}
  {% if not page.date %}
  <li><a href="{{ page.url }}">{{ page.title }}</a></li>
  {% endif %}
  {% endfor %}
</ul>
```

### Using Older/Newer links instead of Previous/Next

| ⚠️ | While the behavior in this section is still fully supported, in future releases of Pico-Pagination, the plugin will stop returning markup and instead leave that to you in your Twig theme. Please see updated example implementation. |
| -- | -- |

A lot of blogs (or other chonologically listed content) will want pagination links that use chonological verbiage as opposed to the standard previous/next text. In order to do this, you can set the following config options in Pico's `config.yml` file.

```yml
pagination_flip_links: true
pagination_next_text: "< Older Posts"
pagination_prev_text: "Newer Posts >"
```

Note the first option set. This changes the order in which the links display so that the older link is first and the newer link is second.

### Using a sub page for the pagination (e.g. for a blog that is not on the main site index)

Sometimes it is preferable to have your blog listed at /blog rather than on your main page, for example to allow for a landing page to be the first page visitors see.

To do this, set the `pagination_sub_page` to `true`, and the `pagination_page_indicator` variable to the page path, i.e. "blog":

```yml
pagination_sub_page: true
pagination_page_indicator: "blog"
```

Then, configure your page: Set up a folder called "blog" (or whatever you've chosen to use as the slug) in Pico's content folder, and keep all your blog posts in this folder. You will need an index.md in here for the pagination.

Finally set up 2 new templates, one for the index.md that has the above pagination code, the other for the posts (again, using the relevant code from above).

### Customize to your needs

The plugin should return all necessary pieces of data to build out your pagination however you see fit. Here is an example that lists all the pages number and adds a CSS class to the current one:

```twig
<div class="pagination">
  {% if page_number > 1 %}
  <a href="{{ prev_page_url }}" class="button pagination-prev">Previous Page</a>
  {% endif %}

  <div class="pagination-pages">
    {% for page_num in 1..total_pages %}
    <a href="{{ base_url }}/{{ page_indicator }}/{{ page_num }}"{% if page_num == page_number %} class="active"{% endif %}>{{ page_num }}</a>
    {% endfor %}
  </div>

  {% if page_number < total_pages %}
  <a href="{{ next_link_url }}" class="button pagination-next">Next Page</a>
  {% endif %}
</div>
```

---

That's it. If you have questions, go ahead and ask. If you have issues, you can add them to the [issue tracker](https://github.com/rewdy/Pico-Pagination/issues).

🎉 Thank you to everyone who has helped contribute to this plug in!

*If you see something that you think is missing, pop open the plugin file and see if you can add it. I'm happy to accept sensible MRs!*
