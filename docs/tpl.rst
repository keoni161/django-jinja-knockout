.. _.addClass(): https://api.jquery.com/addclass/
.. _flatatt(): https://github.com/django/django/search?l=Python&q=flatatt
.. _format_html(): https://docs.djangoproject.com/en/dev/ref/utils/#django.utils.html.format_html
.. _format_html_attrs(): https://github.com/Dmitri-Sintsov/djk-sample/search?l=Python&q=format_html_attrs
.. _get_absolute_url() sample: https://github.com/Dmitri-Sintsov/djk-sample/search?l=Python&q=get_absolute_url
.. _get_absolute_url() documentation: https://docs.djangoproject.com/en/dev/ref/models/instances/#get-absolute-url
.. _.hasClass(): https://api.jquery.com/hasclass/
.. _json_flatatt(): https://github.com/Dmitri-Sintsov/django-jinja-knockout/search?l=HTML&q=json_flatatt
.. _namespaced urls: https://docs.djangoproject.com/en/dev/topics/http/urls/#url-namespaces-and-included-urlconfs
.. _PrintList: https://github.com/Dmitri-Sintsov/django-jinja-knockout/search?l=Python&q=PrintList
.. _readonly_fields: https://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.readonly_fields
.. _.removeClass(): https://api.jquery.com/removeclass/
.. _Renderer: https://github.com/Dmitri-Sintsov/django-jinja-knockout/search?l=Python&q=renderer
.. _.resolver_match: https://docs.djangoproject.com/en/dev/ref/request-response/#django.http.HttpRequest.resolver_match
.. _sprintf.js: https://github.com/alexei/sprintf.js/
.. _str_dict(): https://github.com/Dmitri-Sintsov/djk-sample/search?l=Python&q=str_dict

======
tpl.py
======

Various formatting functions, primarily to be used in:

* ``admin.ModelAdmin`` classes `readonly_fields`_
* :doc:`macros` and templates
* :ref:`clientside_components`
* ``DisplayText`` :doc:`widgets`

Since version 0.8.0, the significant part of the module is implemented via :doc:`djk_ui` package.

.. _tpl_renderer:

Renderer
--------

Since version 0.8.0, `Renderer`_ class is implemented which is internally used to render formsets / forms / fields of
Django modelforms. See :ref:`forms_renderers` for more detail, although it's usage is not limited to forms as it
supports rendering of any object with it's related context data and template with the possible nesting of renderers.

Contenttypes framework helpers
------------------------------

.. highlight:: jinja

* ``ContentTypeLinker`` - class to simplify generation of contenttypes framework object links::

    {% set ctl = tpl.ContentTypeLinker(object, 'content_type', 'object_id') %}
    {% if ctl.url is not none %}
        <a href="{{ ctl.url }}" title="{{ str(ctl.obj_type) }}" target="_blank">
    {% endif %}
        {{ ctl.description }}
    {% if ctl.url is not none %}
        </a>
    {% endif %}

Manipulation with css classes
-----------------------------

* ``escape_css_selector()`` - can be used with server-generated AJAX viewmodels or in Selenium tests.
* ``add_css_classes()`` - similar to client-side ``jQuery`` `.addClass()`_;
* ``has_css_classes()`` - similar to client-side ``jQuery`` `.hasClass()`_;
* ``remove_css_classes()`` - similar to client-side ``jQuery`` `.removeClass()`_;

Optimized for usage as argument of ``Django`` `flatatt()`_:

* ``add_css_classes_to_dict()`` - adds CSS classes to the end of the string
* ``has_css_classes_in_dict()``
* ``prepend_css_classes_to_dict()`` - adds CSS classes to the begin of the string
* ``remove_css_classes_from_dict()``

Objects rendering
-----------------

.. highlight:: python

* ``Str`` - string with may have extra attributes. It's used with ``get_absolute_url()`` model method. See
  `get_absolute_url() documentation`_ and `get_absolute_url() sample`_::

    class Manufacturer(models.Model):

        # ... skipped ...

        title = models.CharField(max_length=64, unique=True, verbose_name='Title')

        def get_absolute_url(self):
            url = Str(reverse('club_detail', kwargs={'club_id': self.pk}))
            url.text = str(self.title)
            return url

* ``ModelLinker`` - render Model links with descriptions which supports ``get_absolute_url()`` and
  :ref:`get_str_fields()`.
* `PrintList`_ class supports custom formatting of nested Python structures, including the mix of dicts and lists.
  There are some already setup function helpers which convert nested content to various (HTML) string representations,
  using `PrintList`_ class instances:

  * ``print_list()`` - print nested HTML list. Used to format HTML in JSON responses and in custom ``DisplayText``
    widgets.
  * ``print_brackets()`` - print nested brackets list.
  * ``print_table()`` - print uniform 2D table (no colspan / rowspan yet).
  * ``print_bs_labels()`` - print HTML list as Bootstrap labels.
  * ``reverseq()`` - construct url with query parameters from url name. Since version 0.4.0, when request instance is
    supplied, absolute url will be returned.

* `str_dict()`_ - Django models could define :ref:`get_str_fields()` method which maps model instance field values to
  their formatted string values, similar to ``Model`` ``__str()__`` method, but for each or to some selected separate
  fields. If these models have foreign keys pointing to another models which also have :ref:`get_str_fields()` defined,
  `str_dict()`_ can be used to convert nested dict :ref:`get_str_fields()` values to flat strings in ``__str__()``
  method::

    class Member(models.Model):

        # ... skipped ...

        def get_str_fields(self):
            parts = OrderedDict([
                ('profile', self.profile.get_str_fields()),
                ('club', self.club.get_str_fields()),
                ('last_visit', format_local_date(timezone.localtime(self.last_visit))),
                ('plays', self.get_plays_display()),
                ('role', self.get_role_display()),
                ('is_endorsed', 'endorsed' if self.is_endorsed else 'unofficial')
            ])
            return parts

        def __str__(self):
            # Will flatten 'profile' and 'club' str_fields dict keys values
            # and convert the whole str_fields dict values into str.
            str_fields = self.get_str_fields()
            return str_dict(str_fields)

Internally `str_dict()`_ uses lower level ``flatten_dict()`` function which is defined in the same module.

String manipulation
-------------------

* ``limitstr()`` - cut string after specified length.
* ``repeat_insert()`` - separate string every nth character with specified separator characters.

String formatting
-----------------

* `json_flatatt()`_ - similar to Django `flatatt()`_, but converts dict / list / tuple / bool HTML attribute
  values to JSON string. Used in :doc:`macros`.
* `format_html_attrs()`_ - similar to Django `format_html()`_, but converts dict / list / tuple / bool HTML attribute
  values to JSON string. Used to generate :ref:`clientside_components`.
* ``format_local_date()`` - output localized ``Date`` / ``DateTime``.
* ``html_to_text()`` - convert HTML fragment with anchor links into plain text with text links. It's used in
  :doc:`utils_mail` ``SendmailQueue`` to convert HTML body of email message to text-only body.
* ``to_json()`` - converts Python structures to JSON utf-8 string.

URL resolution
--------------
* ``get_formatted_url()`` converts url with supplied ``url_name`` from regex named parameters eg. ``(?P<arg>\w+)`` to
  ``sprintf()`` named formatters eg. ``%(arg)s``. Such urls are injected into client-side as
  :ref:`viewmodels_client_side_routes` and then are resolved via the bundled `sprintf.js`_ library.
* ``resolve_cbv()`` takes ``url_name`` and it's ``kwargs`` and returns a function view or a class-based view for these
  arguments, when available::

    tpl.resolve_cbv(url_name, view_kwargs)

Current request's ``url_name`` can be obtained from the ``request`` `.resolver_match`_ ``.url_name``, or ``.view_name``
for `namespaced urls`_.
