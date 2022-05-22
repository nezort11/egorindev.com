---
title: "Images in Python, Django and Pillow"
date: 2022-05-23
date: 2022-05-23
description: "Options on how to define read-only fiends in API/serializer in Django REST Framework."
---

# Django REST Framework Read-Only fields

>  Why such a simple topic should have a separate article?

because read-only fields are very important part of API and there are a lot of different ways to declare them (in the order of usage popularity):

- Using `ModelSerializer`'s ' `read_only_fields` in `Meta` class;

- Using `ReadOnlyField` serializer field;

- Using `read_only=True` option in any of serializer fields;

- Using `extra_kwargs={"field": {"read_only": True}}` extra keyword argument;

- Using `editable=False` option in Django model (not editable by admin nor API).

**Note**:

`primary_key=True`, `DateTimeField(auto_add[_now]=True)` are marked as `editable=False` and therefore *are read-only* by default. Query set annotated fields aren't read-only by default.

## `read_only_fields` and `extra_kwargs`

Used as shortcuts for marking model fields (listed in `fields=[...]`) as read-only. 

**Note**: this doesn't work for new fields defined on serializer directly.

## `ReadOnlyField` and `read_only=True`

For additional fields added to `ModelSerializer` or fields in `Serializer`.

## `editable=False`

Not only marks field as read-only but also removes it from admin interface.

## Conclusion

I hope next next time you will think about the way to mark field read-only it might help)
