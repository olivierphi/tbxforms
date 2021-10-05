[![PyPI](https://img.shields.io/pypi/v/tbxforms.svg)](https://pypi.org/project/tbxforms/)
[![npm](https://img.shields.io/npm/v/tbxforms.svg)](https://www.npmjs.com/package/tbxforms) [![PyPI downloads](https://img.shields.io/pypi/dm/tbxforms.svg)](https://pypi.org/project/tbxforms/)
[![Build status](https://github.com/kbayliss/tbxforms/workflows/CI/badge.svg)](https://github.com/kbayliss/tbxforms/actions)
[![Coverage Status](https://coveralls.io/repos/github/kbayliss/tbxforms/badge.svg?branch=main)](https://coveralls.io/github/kbayliss/tbxforms?branch=main)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/kbayliss/tbxforms.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/kbayliss/tbxforms/alerts/)

# Torchbox Forms

A Torchbox-flavoured template pack for [django-crispy-forms](https://github.com/django-crispy-forms/django-crispy-forms), adapted from [crispy-forms-gds](https://github.com/wildfish/crispy-forms-gds).

Out of the box, forms created with `tbxforms` will look like the
[GOV.UK Design System](https://design-system.service.gov.uk/), though many
variables can be customised.

## Contents

-   [Torchbox Forms](#torchbox-forms)
    -   [Installation](#installation)
        -   [Install the Python package](#install-the-python-package)
        -   [Install the NPM package](#install-the-npm-package)
    -   [Usage](#usage)
        -   [Create a Django form](#creating-a-django-form)
        -   [Create a Wagtail form](#creating-a-wagtail-form)
            -   [Add a `helper` property to the Wagtail form](#add-a--helper--property-to-the-wagtail-form)
            -   [Instruct a Wagtail Page model to use the newly created form](#instruct-a-wagtail-page-model-to-use-the-newly-created-form)
        -   [Render a form](#render-a-form)
        -   [Customise a form's attributes (via the `helper` property)](#customising-a-form-s-attributes--via-the--helper--property-)
            -   [Possible values for the `label_size` and `legend_size`:](#possible-values-for-the--label-size--and--legend-size--)
        -   [Conditionally-required fields](#conditionally-required-fields)
-   [Further reading](#further-reading)

## Installation

You must install both a Python package and an NPM package.

### Install the Python package

Install using pip:

```bash
pip install tbxforms
```

Add `django-crispy-forms` and `tbxforms` to your installed apps:

```python
INSTALLED_APPS = [
  ...
  'crispy_forms',  # django-crispy-forms
  'tbxforms',
]
```

Now add the following settings to tell `django-crispy-forms` to use this theme:

```python
CRISPY_ALLOWED_TEMPLATE_PACKS = ["tbx"]
CRISPY_TEMPLATE_PACK = "tbx"
```

### Install the NPM package

Install using NPM:

```bash
npm install tbxforms
```

Instantiate your forms:

```javascript
import TbxForms from 'tbxforms';

document.addEventListener('DOMContentLoaded', () => {
    for (const form of document.querySelectorAll(TbxForms.selector())) {
        new TbxForms(form);
    }
});
```

Import the styles into your project, either as CSS:

```scss
// Either as CSS without any customisations:
@use 'node_modules/tbxforms/style.css';
```

Or as Sass, to customise variables:

```scss
// Or as Sass, with variables to customise:
@use 'node_modules/tbxforms/tbxforms.scss' with (
    $tbxforms-error-colour: #f00,
    $tbxforms-text-colour: #000,
);
```

Variables can also be defined in a centralised variables SCSS file, too.

See [tbxforms/static/sass/abstracts/\_variables.scss](https://github.com/kbayliss/tbxforms/blob/main/tbxforms/static/sass/abstracts/_variables.scss) for customisable variables.

## Usage

### Create a Django form

```python
from tbxforms.forms import BaseForm as TbxFormsBaseForm

class ExampleForm(TbxFormsBaseForm, forms.Form):
    # < Your field definitions >


class ExampleModelForm(TbxFormsBaseForm, forms.ModelForm):
    # < Your field definitions and ModelForm config >

```

### Create a Wagtail form

Two parts are required for this to work:

1. Add a `helper` property to the Wagtail form
2. Instruct a Wagtail Page model to use the newly created form

#### Add a `helper` property to the Wagtail form

```python
from wagtail.contrib.forms.forms import BaseForm as WagtailBaseForm
from tbxforms.forms import BaseForm as TbxFormsBaseForm

class ExampleWagtailForm(TbxFormsBaseForm, WagtailBaseForm):

    # Extend the `TbxFormsBaseForm.helper()` to add a submit button.
    @property
    def helper(self):
        fh = super().helper
        fh.add_input(
            Button.primary(
                name="submit",
                type="submit",
                value=_("Submit"),
            )
        )
        return fh
```

#### Instruct a Wagtail Page model to use the newly created form

```python
# -----------------------------------------------------------------------------
# in your forms definitions (e.g. forms.py)

from tbxforms.forms import BaseWagtailFormBuilder as TbxFormsBaseWagtailFormBuilder
from path.to.your.forms import ExampleWagtailForm

class WagtailFormBuilder(TbxFormsBaseWagtailFormBuilder):
    def get_form_class(self):
        return type(str("WagtailForm"), (ExampleWagtailForm,), self.formfields)

# -----------------------------------------------------------------------------
# in your page models (e.g. models.py)

from path.to.your.forms import WagtailFormBuilder

class ExampleFormPage(...):
    ...
    form_builder = WagtailFormBuilder
    ...
```

### Render a form

Just like Django Crispy Forms, you need to pass your form object to the
`{% crispy ... %}` template tag, e.g.:

```
{% load crispy_forms_tags %}
{% crispy your_form %}
```

### Customise a form's attributes (via the `helper` property)

By default, every form that inherits from `TbxFormsBaseForm` will have the following
attributes set:

-   `html5_required = True`
-   `label_size = Size.MEDIUM`
-   `legend_size = Size.MEDIUM`
-   `form_error_title = _("There is a problem with your submission")`
-   Plus everything from [django-crispy-forms' default attributes](https://django-crispy-forms.readthedocs.io/en/latest/form_helper.html).

These can be overridden (and/or additional attributes from the above list defined)
just like you would do with any other inherited class, e.g.:

```python

class YourSexyForm(TbxFormsBaseForm, forms.Form):

    @property
    def helper(self):
        fh = super().helper
        fh.html5_required = False
        fh.label_size = Size.SMALL
        fh.form_error_title = _("Something's wrong, yo.")
        return fh

```

#### Possible values for the `label_size` and `legend_size`:

1. `SMALL`
2. `MEDIUM` (default)
3. `LARGE`
4. `EXTRA_LARGE`

### Conditionally-required fields

`tbxforms` supports hiding/showing of fields or elements (e.g. div or fieldset)
based on the values of a given input field.

Example:

```python
class ExampleForm(TbxFormsBaseForm, forms.Form):
    NEWSLETTER_CHOICES = (
        Choice("yes", "Yes please", hint="Receive occasional email newsletters."),
        Choice("no", "No thanks"),
    )

    newsletter_signup = forms.ChoiceField(
        choices=NEWSLETTER_CHOICES
    )

    email = forms.EmailField(
        widget=forms.EmailInput(required=False)
    )

    @staticmethod
    def conditional_fields_to_show_as_required() -> [str]:
        return [
            "email", # Include any fields that should show as required to the user.
        ]

    @property
    def helper(self):
        fh = super().helper

        # Override what is rendered for this form.
        fh.layout = Layout(

            # Add our newsletter sign-up field.
            Field("newsletter_signup"),

            # Add our email field, and define the conditional 'show' logic.
            Field(
                "email",
                data_conditional={
                    "field_name": "newsletter_signup", # Field to inspect.
                    "values": ["yes"], # Value(s) to cause this field to show.
                },
            ),

        )

        return fh


    def clean(self):
        cleaned_data = super().clean()
        newsletter_signup = cleaned_data.get("newsletter_signup")
        email = cleaned_data.get("email")

        # Conditionally-required fields should also be validated as part of the
        # form clean process to ensure data validity.
        if newsletter_signup == "yes" and not email:
            raise ValidationError(
                {
                    "email": _("This field is required."),
                }
            )

        return cleaned_data

```

# Further reading

-   Download the [PyPi package](http://pypi.python.org/pypi/tbxforms)
-   Download the [NPM package](https://www.npmjs.com/package/tbxforms)
-   Learn more about [Django Crispy Forms](https://django-crispy-forms.readthedocs.io/en/latest/)
-   Learn more about [Crispy Forms GDS](https://github.com/wildfish/crispy-forms-gds)
-   Learn more about [GOV.UK Design System](https://design-system.service.gov.uk/)
