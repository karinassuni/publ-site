Title: Image renditions
Sort-Title: 100 Image renditions
Path-Alias: /image-renditions
Date: 2018-04-05 02:12:57-07:00
Entry-ID: 335
UUID: 8d3fa7ba-db5e-4661-bfd8-e3ee25684790

How to configure images and galleries for display.

.....


## Image rendition support

### In entries

See the [entry format article](/entry-format#image-renditions)

### In templates

Pass these in as parameters to `entry.body` and/or `entry.more` to configure
the default values for entry images.

The template also gets a function, `image()`, that allows the template itself to
render images.

## Configuration values

### General configuration

* **`absolute`**: Whether to produce absolute URLs

* **`link`**: Put a hyperlink on the image pointing to the given URL

### <span id="style"></span>Style options

* **`img_class`**, **`class`**: If set, sets the `class` attribute on the image tag

    If both are set, `class` wins; the intention is that in a Markdown context, `img_class` comes from the container and `class` comes from the image.

* **`img_style`**, **`style`**: A string or a list of strings, which provide inline CSS rules to the `<img>` element

    When the element is written, any attributes in `style` will override those in `img_style`; the intention is that in a Markdown tag, `img_style` comes from the container and `style` comes from the image.

* **`shape`**: Adds the CSS3 `shape-outline` attribute to the image tag's style; can be one of the following:

    * `True`: Uses the base rendition of the image itself (this is usually what you want)

    * A string value: uses the specified image's alpha channel as the shape

        This can be any path scheme supported by the [image path scheme](322#image-renditions) (e.g. `local-file.png`, `/common/absolute-file.png`, `@images/static-file.png`, `//example.com/remote-file.png`).

    For example, if you have an image `inset-image.jpg` that you want to mask off with `inset-image-mask.png` and float left,
    you can do:

    ```markdown
    ![](inset-image.jpg{shape="inset-mask.png",style="float:left"})
    ```

    It is better to make image insets' float rules part of your stylesheet, however, so that it can be responsive; for example:

    ```markdown
    ![](inset-image.jpg{shape="inset-mask.png",img_class="inset-left"})
    ```

    The size of the outline image will be mapped the same as the base image, wherever possible; for example:

    ```markdown
    ![{320,320}](inset-image.jpg{shape="inset-mask.png"})
    ```

    will size both the image and its mask to fit within a 320x320 square.

### Inline image sizing

* **`scale`**: What factor to scale the source image down by (e.g. 3 = the display size should be 33%)

* **`scale_min_width`**: The minimum width to target based on scaling

* **`scale_min_height`**: The minimum height to target based on scaling

* **`width`**: The width to target

* **`height`**: The height to target

* **`max_width`**: If present and smaller than `width`, use this instead (useful for templates)

* **`max_height`**: If present and smaller than `height`, use this instead (useful for templates)

* **`resize`**: If both `width` and `height` are specified, how to fit the image into the rectangle if the aspect ratio doesn't match
    * `fit`: Fit the image into the space (default)
    * `fill`: Fill the space with the image, cropping off the sides
    * `stretch`: Distort the image to fit

* **`fill_crop_x`**: If `resize="fill"`, where to take the cropping (0=left, 1=right); default=0.5

* **`fill_crop_y`**: If `resize="fill"`, where to take the cropping (0=top, 1=bottom); default=0.5

**Note:** Images will never be scaled to larger than their native resolution. (In the future there may be
an option to still resize it larger client-side, where the actual rendition will be the native size but the
`<img>` tag gets the expanded width and height.)

### File format options

* **`format`**: Select the format to display the image as (defaults to the original format)

* **`background`**: The background color to use when converting transparent images (such as .png) to non-transparent formats (such as .jpg).

    This parameter can be in a number of formats:

    * A [named color](https://drafts.csswg.org/css-color-4/#named-colors) such as `"black"` or `"white"`
    * An HTML-style hex code such as `"#ff7733"` or `"#f73"`
    * An RGB tuple such as `(0,127,35)` (note the lack of quotes!)

    If you're daring you can also use any of the other color formats supported by [PIL.ImageColor](https://pillow.readthedocs.io/en/3.1.x/reference/ImageColor.html), e.g. `"hsl(0,100%,50%)"`

* **`quality`**: The JPEG quality level to use for all renditions

* **`quality_ldpi`**: The JPEG quality level to use for low-DPI renditions

* **`quality_hdpi`**: The JPEG quality level to use for high-DPI renditions

### Image set options

These options drive the behavior of image sets.

* **`div_class`**: The CSS class to use for the containing `<div>`

* **`div_style`**: A string or a list of strings, which provide inline CSS rules to the `<div>` element

* **`gallery_id`**: An identifier for the Lightbox image set (for use with [lightbox.js](http://www.lokeshdhakar.com/projects/lightbox2/))
    * **Note:** If this is not set, Lightbox will not be enabled, and popup renditions will not be generated
    * **Note:** If `link` is set, this option has no effect

* **`count`**: How many images to allow in the image set (useful for feeds)

* **`count_offset`**: If `count` is set, also skip this number of images at the beginning

* **`fullsize_width`**: The maximum width for the popup image

* **`fullsize_height`**: The maximum height for the popup image

* **`fullsize_quality`**: The JPEG quality level to use for the popup image

* **`fullsize_format`**: What format the popup image should be in (defaults to the original format)

* **`fullsize_background`**: The background color to use when converting transparent images (such as .png) to non-transparent formats (such as .jpg)

* **`more_text`**: If images were skipped due to `count` or `count_offset`, add this text after the image set. This string will be rendered with the following template arguments:
    * **`{count}`**: The total number of images in the image set
    * **`{remain}`**: The number of images not shown

* **`more_link`**: If `more_text` is shown, it will be wrapped in a link with this location

* **`more_class`**: If `more_text` is shown, it will be given this CSS class (either on the `<a>` tag if it's a link, or via a `<span>`)

## Useful template examples

### A webcomic

`index.html` and `entry.html`:

This will treat source images as being 3x screen resolution, make images scale to no narrower than 480 pixels and to no wider than 960 pixels,
force them to be a JPEG (with transparency turning white), and with 35% JPEG quality for the high-DPI rendition

```jinja
{{ entry.body(
    scale=3,
    scale_min_width=480
    width=960,
    format="jpg",
    background="white",
    quality_hdpi=35)
}}
```

`feed.xml`:

This will resize the image to no more than 400 pixels wide or tall and crop the thumbnail to the top or left
of the image (making for a useful non-punchline-destroying excerpt).

```jinja
{{ entry.body(link=entry.link(
    absolute=True,
    force_width=400,
    force_height=400,
    resize="fill",
    fill_crop_x=0,
    fill_crop_y=0,
    force_size=True)
}}
```

In the above example, if you have a comic that is provided at screen resolution to begin with (such as guest art) you can override the default scaling with e.g.:

```markdown
![](guest-comic.png{scale=1} "Amazing guest comic!")
```

Or if there's one you want to force to a specific size:

```markdown
![](special-comic.jpg{width=960,height=480})
```

which can also be written more compactly as:

```markdown
![](special-comic.jpg{960,480})
```


### A photo gallery

With the below setup, if an entry provides an image set with a `count_offset` parameter, e.g.

```markdown
![{count_offset=2}](image1.jpg | image2.jpg | image3.jpg | image4.jpg)
```

then on the index and feed (where there's a `count` set) skip the first two images and thus show `image3.jpg` as the first image. This allows you to set a "poster" frame for the image set as a whole.

#### `index.html`

This will show just the first image in the gallery at its original aspect, and then link the user to the full
gallery page with an image count.

```jinja
{{ entry.body(
    width=640,
    height=640,
    link=entry.link,
    count=1,
    more_text='Show {count} images',
    more_link=entry.link)
    }}
```

#### `entry.html`

This will show the full gallery with square thumbnails and a reasonable default gallery ID, and the full images at 4K resolution.
The thumbnails will be wrapped in a `<div class="gallery_thumbs">`.

```jinja
{{ entry.body(
    width=640,
    height=640,
    resize="fill",
    lightbox_id=entry.uuid,
    div_class="gallery_thumbs",
    fullsize_width=3840,
    fullsize_height=2160)
}}
```

#### `feed.xml`

This will show tiny thumbnails of the first three images of the gallery and will link to the full gallery page.

```jinja
{{ entry.body(
    max_width=300,
    max_height=300,
    count=3,
    link=entry.link,
    force_size=True,
    more_text='{remain} more images in gallery')
}}
```

