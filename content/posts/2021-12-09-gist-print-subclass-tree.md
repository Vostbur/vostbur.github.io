---
categories:
- Programming
date: "2021-12-09"
description: Snippet for printing a subclass tree.
slug: snippet-printing-subclass-tree
tags:
- python
title: Snippet for printing a subclass tree
---

Sometimes I needed to figure out the class hierarchy. For this, I wrote a small piece of code to visualize the class tree.

```
def format_output(class_name, prefix, is_last):
    left_simbol = "└" if is_last else "├"
    return prefix + left_simbol + "───" + class_name


def print_subclass_tree(thisclass, prefix=""):
    for count, subclass in enumerate(thisclass.__subclasses__(), start=1):
        is_last = count == len(thisclass.__subclasses__())
        print(format_output(subclass.__name__, prefix, is_last))

        devide_simbol = "" if is_last else "│"
        print_subclass_tree(subclass, prefix + devide_simbol + "\t")


print_subclass_tree(BaseException)

# ============ OUTPUT ==================
# >>> print_subclass_tree(BaseException)
# ├───Exception
# │       ├───TypeError
# │       ├───StopAsyncIteration
# │       ├───StopIteration
# │       ├───ImportError
# │       │       ├───ModuleNotFoundError
# │       │       └───ZipImportError
#     ...
# ├───SystemExit
# └───KeyboardInterrupt

# >>> from django.views.generic.base import View
# >>> print_subclass_tree(View)
# ├───TemplateView
# ├───RedirectView
# ├───BaseDetailView
# │       ├───DetailView
# │       ├───BaseDateDetailView
# │       │       └───DateDetailView
# │       └───BaseDeleteView
# │               └───DeleteView
#     ...
# └───JavaScriptCatalog
#         └───JSONCatalog
```

----
### Link:

- [gist.github.com](https://gist.github.com/Vostbur/0ce27a13e0c72a22851ab101fd29ab6c)