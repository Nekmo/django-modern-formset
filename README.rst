=====================
django-modern-formset
=====================

Django-formset-js fork with more features and modernized.

Installing
----------

Install via pip::

    pip install django-formset-js

Then add it and its dependancy ``django-jquery-js``
to your ``INSTALLED_APPS``:

.. code-block:: python

    INSTALLED_APPS += (
        'django.contrib.staticfiles',
        'jquery',
        'djangoformsetjs',
    )

Using
-----

Include the JavaScript library
******************************

Both jQuery and this library must be included in your page.
The simplest way to do this is to add the scripts as media dependencies on your form:

.. code-block:: python

    from djangoformsetjs.utils import formset_media_js

    class MyForm(forms.Form):
        class Media(object):
            js = formset_media_js + (
                # Other form media here
            )

    MyFormSet = formset_factory(MyForm)

And then include the Media of the form in your template:

.. code-block:: html+django

    {{ formset.media }}

Alternatively, simply add the script tags:

.. code-block:: html+django

    <script src="{{ STATIC_URL }}js/jquery.js"></script>
    <script src="{{ STATIC_URL }}js/jquery.formset.js"></script>

Render the formset
******************

So that the library can work with your formset,
certain blocks of your formset need to be marked up with ``data-formset-...`` attributes:

.. code-block:: html+django

    {% load formset_tags %}

    <div id="formset" data-formset-prefix="{{ formset.prefix }}">
        {{ formset.management_form }}

        <div data-formset-body>
            <!-- New forms will be inserted in here -->
            {% for form in formset %}
                <div data-formset-form>
                    {{ form }}
                    <button type="button" data-formset-delete-button>Delete form</button>
                </div>
            {% endfor %}
        </div>

        <!-- The empty form template. By wrapping this in a <script> tag, the
        __prefix__ placeholder can easily be replaced in both attributes and
        any scripts -->
        <script type="form-template" data-formset-empty-form>
            {% escapescript %}
                <div data-formset-form>
                    {{ formset.empty_form }}
                    <button type="button" data-formset-delete-button>Delete form</button>
                </div>
            {% endescapescript %}
        </script>

        <!-- This button will add a new form when clicked -->
        <input type="button" value="Add another" data-formset-add>

        <script>jQuery(function($) {
            $("#formset").formset({
                animateForms: true
            });
        });</script>

    </div>

The ``data-formset-`` data attributes are:

``data-formset-prefix``
  The value of ``{{ formset.prefix }}``.
  This is used to find the management form.

``data-formset-body``
  This indicates where all the child forms are.
  New forms are inserted in here.

``data-formset-form``
  Every form (including the empty form) should have this attribute.

``data-formset-empty-form``
  The element that contains the empty form template.
  For best results, use a ``<script>`` tag.

``data-formset-add``
  A button that adds a new form.

``data-formset-delete-button``
  A button that deletes that form.

The empty form template is wrapped in a ``<script>`` as plain text.
This stops any JavaScript attached to widgets from running upon page load,
and makes finding and replacing the ``__prefix__`` placeholder easier.
The contents of the ``<script>`` should be wrapped in a ``{% escapescript %}`` block
to prevent any script tags inside from closing the wrapping script tag prematurely.

When the ``data-formset-add`` button is clicked, the ``formAdded`` event is
fired on the form which was added. This event propagates upwards, and as such
can be handled from the form container.
For example, to select the new form added for form additions from the above
example, bind as such:

.. code-block:: javascript

    $('#formset').on('formAdded', function(event) {
        newForm = event.target;
        //Do Stuff
    });

If the forms can be deleted, and contain a delete checkbox,
the following actions occur:

* When the checkbox is checked, marking the form for deletion,
  the ``formDeleted`` event is fired on the ``data-formset-form`` container,
  and the ``data-formset-form-deleted`` attribute is added.

* When the checkbox is unchecked, marking the form as active again,
  the ``formAdded`` event is fired on the ``data-formset-form`` container,
  and the ``data-formset-form-deleted`` attribute is removed.

If the forms can be deleted, and contain a delete button,
pressing the delete button will toggle the delete checkbox for that form.
The ``DELETE`` field should be hidden if the delete button is used.
The delete button is identified by the ``data-formset-delete-button`` attribute:

.. code-block:: html+django

    {% for form in formset %}
        <div data-formset-form>
            {{ form.name }}
            {{ form.age }}

            <div class="hidden">{{ form.DELETE }}</div>
            <button type="button" data-formset-delete-button>Delete form</button>
        </div>
    {% endfor %}

If the ``animateForms`` option is set when the formset is created,
adding and deleting forms will be animated by sliding the forms in and out.

Options
*******

The jQuery plugin takes the following options:

``form``:
  The selector to find forms.
  Defaults to ``[data-formset-form]``.

``emptyForm``:
  The selector to find the empty form template.
  Defaults to ``script[type=form-template][data-formset-empty-form]``.

``body``:
  The selector to find the formset body.
  New forms will be inserted at the bottom of this element.
  Defaults to ``[data-formset-body]``.

``add``:
  The selector to find the add button.
  Defaults to ``[data-formset-add]``.
``deleteButton``:
  The selector to find the delete button within a form.
  Defaults to ``[data-formset-delete-button]``.

``hasMaxFormsClass``:
  The class added to the formset when the maximum number of forms is reached.
  The maximum number of forms is pulled from the management form.
  Defaults to ``has-max-forms``.

``animateForms``:
  Whether to animate form addition/deletion.
  Defaults to ``false``.

Javascript API
--------------

If the bundled functionality is not for you,
you can interact with the formset using the JavaScript API.
All the behaviour is driven by a ``Formset`` class.
To get a ``Formset`` for an element, call:

.. code-block:: javascript

    var formset = $('#my-form').formset('getOrCreate');

This can be called multiple times on a single element,
and will always return the same ``Formset`` instance.
All the methods and attributes listed below operate on a ``Formset`` instance.

``Formset.opts``
    The options used to create this ``Formset``.

``Formset.$formset``
    The element the ``Formset`` was created for.

``Formset.$emptyForm``
    The empty form template used to create new forms.

``Formset.$body``
    The element where new forms are created.

``Formset.$add``
    The button used to add new forms.

``Formset.addForm()``
    Add a form to the ``Formset``.
    If the maximum number of forms would be exceeded if another form was added,
    an error will be thrown.

``Formset.$forms()``
    Get a jQuery object of all the forms in the ``Formset``.

``Formset.$managementForm(field)``
    Get a jQuery object for the management form field ``field``:

    .. code-block:: javascript

        // Update the TOTAL_FORMS management form field
        this.$managementForm('TOTAL_FORMS').val(10);

``Formset.totalFormCount()``
    Count the total number of forms in the ``Formset``, including deleted forms.

``Formset.activeFormCount()``
    Count the total number of active (not deleted) forms in the ``Formset``.

``Formset.deletedFormCount()``
    Count the number of deleted forms in the ``Formset``.

``Formset.hasMaxForms()``
    Return true if the ``Formset`` has its maximum number of forms.

``Formset.checkMaxForms()``
    Check how many forms are in the ``Formset``,
    and set the relevant classes on the ``Formset`` element
    if the ``Formset`` has reached its limit.

Example
-------

A minimal example project is provided in the ``example/`` directory.
Read ``example/README`` for more information

Developing
----------

When running ``./setup.py sdist``, the JavaScript asset is minified using
UglifyJS if it is installed. To install UglifyJS, install node.js and npm, and
run::

    npm install uglifyjs

You can minify the scripts manually using::

    ./setup.py minify
