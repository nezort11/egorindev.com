---
title: "How useful are Django's model permissions?"
date: 2022-03-09
description:
  "Thoughts about proper use cases of Django's built-in user model permission
  system."
---

# How useful are Django's model permissions?

Django permissions are labels assign to specific users, these labels represent
whether the user is permitted to do X-label or Y-label. So basically this
permission system can be used to classify the **users** (not general objects)
e.g. by job, like `teacher`, `programmer`, `artist` and later derive information
from assigned categories for authorization (conditionally) or rendering (display
user assigned categories on web page).

It will have more sense to rename the system like this:

- `PermissionMixin` to `UserCategoryMixin`

- `Permission` to `UserCategory`

- `user_permissions` to `user_categories` (or simply `categories`)

- `has_perm` to `has_cetegory`

- and etc.

But because this functionality is only available for user model (via
`PermissionMixin`) Django developers to turn general categorization system into
permission system and only for users model.

## Advantages and disadvantages

Advantages:

- easy assign permission as model field `user.user_permissions.add()` or
  `assign_perm(user, "perm", obj)`

- short syntax `user.has_perm('perm')` for model-level permission provides nice
  interface for underlying database permission querying (checking
  `user_permissions`)

- dispatching for object-level permissions `user.has_perm("perm", obj)`

- permissions and assigned are stored in database, are fixed and predetermined

Disadvantages:

- permissions can not be assigned nor checked for anonymous users, so `if-else`
  branching should be used

- sometimes authorization completely depends on database state, so you either
  need to:

  - use `create`/`delete` model signals for dynamic assignment and removal of
    permissions (which is odd)

  - or using `if-else` state permission checking (which have nothing to do with
    Django permissions model/objects)

- not good to use and represent application state by assigned permissions
  (database state), e.g. display all members by returning users that has
  specific permission

## Use case

So Django permissions are good for very specific cases when:

- authorization permissions are **set by other users (admins)**. This was the
  initial goal of Django permissions: let admins set per model or per object
  permissions for selected users

- administration and **manual** user access regulation

- best-suited for grouping and permitting users by **admin**

- data is already displayed (no need to _derive data from permissions_)

- someone will **assign** (suited for external assignment and removal as any
  CRUD object) either via admin interface or API call (so never **assign
  permissions by yourself**, by state change or particularly event other than
  _direct user willing to do that_, don't mix/derive database events with
  permission assignments)

## Experience

I wrote these notes because I thought it will be a good idea to integration
Django permission and not follow the **admin pattern** (which is the key use
case of Django permission) but trying to use it in conjunction with database
state. It was a continuous _duplication and synchronization of state between
objects and permission_ via signals, on create and delete.

## Conclusion

So my advice is to use Django permission only in **user regulation by other
user** pattern.

I guess there is a package that let's you determine user permission based on the
passed object's attributes not assigned permissions. Like
`user.has_perm("view", post)` will return `True` if `post.public is True`, this
is the ideal usage of permission (which people think how it should be used
through assignments).

It can also be implemented as authorization model methods, like `can_view_post`
or generic/dispatch `user.can(perm/action, object)`, inside `Permission` class.
Just to make a more DRY.

**EDIT**

I found a [django-rules](https://github.com/dfunckt/django-rules) package which
has an entirely different use case compared to
[django-guardian](https://github.com/django-guardian/django-guardian). It lets
you define a more abstracted user authorization rules and keep the code more
DRY.
