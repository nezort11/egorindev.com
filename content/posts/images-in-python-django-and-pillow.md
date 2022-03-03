---
title: "Images in Python, Django and Pillow"
date: 2021-10-06
description:
  "Comparison of working with images in Python language, Django framework and
  Pillow package."
---

# Images in Python, Django and Pillow

Devlog #1 (blog post):

- background/problem/project

- research/information/content

- summary

---

Recently working on my project, I needed to implement image format conversion
and then save into the database. This functionality should be represented as web
API. For backend I used a Django and for image library - Pillow.

But Django was quite hard for me to understand it's file management. It was file
storage, file classes for uploaded files, for general files, for images and
content. It even has different instance class for file/image field of the model.

# Python

Let's start from the language. In Python we can create file object by using
builtin `open` function. File object is just a data stream for i/o operations,
it streams data from 1 place to another using `read` and `write` methods.

> The basic requirements for file objects is that they should implement `read`,
> `write` and some other methods.

Files (or data in general) can be **located** on:

- filesystem (SSD)

- in-memory (RAM)

In-memory file stream are also called _buffers_.

There are 3 main **stream types**:

- binary stream (bytes stream)

- raw binary stream

- text stream (string stream)

We can open file in different modes (e.g. `rt`, `wt`, `rb`, `wb`) and get
different types of stream.

## io module

Python also has `io` module, where we can see some of the classes and function
for managing input/output operations.

`io` provides:

- `open` function (alias for Python built-in `open`)

- `BytesIO` (binary stream/buffer)

- `StringIO` (text stream/buffer)

- `FileIO`

# Django

Let's look at the Django file hierarchy.

<img title="" src="file:///Users/egorzorin/Desktop/blog/django_files.png" alt="django_files.png" data-align="center">

> I think that small snippet of code (especially Python code) can give more
> understanding than a hounder of words. So I will provide small code snippets
> of file classes declaration from the Django source code.

- [FileProxyMixin](https://github.com/django/django/blob/2116238d5f2ce19571becb5620ef16ce9048db9f/django/core/files/utils.py#L27)
  is a class that proxies (copies the properties and methods) from the Python
  standard file object like `read`, `write` or `name`. It is basically an
  _alias/proxy for the regular Python stream object_.

```python
class FileProxyMixin:
    """
    A mixin class used to forward file methods to an underlaying file
    object.  The internal file object has to be called "file"::
        class FileProxy(FileProxyMixin):
            def __init__(self, file):
                self.file = file
    """

    encoding = property(lambda self: self.file.encoding)
    fileno = property(lambda self: self.file.fileno)
    flush = property(lambda self: self.file.flush)
     # ...
```

- [File](https://github.com/django/django/blob/2116238d5f2ce19571becb5620ef16ce9048db9f/django/core/files/base.py#L8)
  initializes file and provides basic file attributes and methods + some
  convenient method for getting size and file chunks.

```python
class File(FileProxyMixin):
    DEFAULT_CHUNK_SIZE = 64 * 2 ** 10

    def __init__(self, file, name=None):
        self.file = file
        if name is None:
            name = getattr(file, 'name', None)
        self.name = name
        if hasattr(file, 'mode'):
            self.mode = file.mode

    def __str__(self):
        return self.name or ''

    # ...
```

- [**ContentFile**](abc), v**ImageFile** have notion about the actual content
  stored in the file. ContentFile can be StringIO or BytesIO stream. ImageFile
  _uses Pillow_ to provide additional information (width/height) about image
  stream.

```python
class ContentFile(File):
    """
    A File-like object that takes just raw content, rather than an actual file.
    """
    def __init__(self, content, name=None):
        stream_class = StringIO if isinstance(content, str) else BytesIO
        super().__init__(stream_class(content), name=name)
        self.size = len(content)

    def __str__(self):
        return 'Raw content'
    # ...
```

```python
class ImageFile(File):
    """
    A mixin for use alongside django.core.files.base.File, which provides
    additional features for dealing with images.
    """
    @property
    def width(self):
        return self._get_image_dimensions()[0]

    @property
    def height(self):
        return self._get_image_dimensions()[1]

    def _get_image_dimensions(self):
        if not hasattr(self, '_dimensions_cache'):
            close = self.closed
            self.open()
            self._dimensions_cache = get_image_dimensions(self, close=close)
        return self._dimensions_cache
```

- **FieldFile** is a representation of the model FileField. It wraps the _field
  file storage_ for file streaming. It provides the following attributes and
  methods:

  - name, path, url, size

  - save(), delete()

```python
class FieldFile(File):
    def __init__(self, instance, field, name):
        super().__init__(None, name)
        self.instance = instance
        self.field = field
        self.storage = field.storage
        self._committed = True
    # ...
```

- **UploadFile** is very similar to File class but it also provide additional
  attributes submitted from the HTML form like `size`, `content_type`,
  `charset`.

```python
class UploadedFile(File):
    """
    An ``UploadedFile`` object behaves somewhat like a file object and
    represents some file data that the user submitted with a form.
    """

    def __init__(self, file=None, name=None, content_type=None, size=None, charset=None, content_type_extra=None):
        super().__init__(file, name)
        self.size = size
        self.content_type = content_type
        self.charset = charset
        self.content_type_extra = content_type_extra

    # ...
```

# Pillow

Pillow uses **ImageFile** class to store image data/file stream. We use pillow
to:

1. Parse file as image and get additional imaging information.

2. Perform manipulation to that file as to image (crop, resize, convert, etc.).

In my project I used pillow to get specific image information (for validation
and manipulation) from **plain bytes object or stream**. And then convert this
image into specified image format.

Converter source code implemented as Form post-clean operation:

```python
def _convert(self):
    """Convert source image to target format and set target image."""
    image_source = self.cleaned_data["image_source"]
    format_target = self.data["format_target"]

    with Image.open(image_source) as i:
        # Convert image pixel color mode
        if i.format.lower() == "png":
            i = i.convert("RGB")

        # Make in-memory byte stream for storing target image
        image_target_buffer = io.BytesIO()

        # Convert image format ans save into bytes buffer
        i.save(image_target_buffer, format_target)

        image_target_path = Path(image_source.name).with_suffix(f'.{format_target}')

        # Make Django image file from bytes stream
        image_target = ImageFile(image_target_buffer, str(image_target_path))

        self.cleaned_data["image_target"] = image_target
```

# Summary

Python file management is greatly simplified by designers. And since both Django
and Pillow are using Python file objects/classes it is very easy to work with
them and seamlessly move between different file types.

I hope this article helped get more in-depth understanding of the file
management.
