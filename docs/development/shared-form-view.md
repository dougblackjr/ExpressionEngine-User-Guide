---
lang: php
---

<!--
    This source file is part of the open source project
    ExpressionEngine User Guide (https://github.com/ExpressionEngine/ExpressionEngine-User-Guide)

    @link      https://expressionengine.com/
    @copyright Copyright (c) 2003-2019, EllisLab Corp. (https://ellislab.com)
    @license   https://expressionengine.com/license Licensed under Apache License, Version 2.0
-->

# Shared Form View

[TOC]

ExpressionEngine's control panel markup is very modular and consistent. Because of that, we were able to abstract out the creation of most form views to a single view file to save repeating ourselves and keeping our form markup maintainable. This view is also available to add-on developers and is recommended for use for easy style-guide adherence, built-in support for form validation, as well as never having to lift a finger to edit markup later should our style guide change.

## Getting started

The concept is quite familiar. In an ExpressionEngine controller file, we will create a variable and pass it to a view. The variable will be a specifically-structured array that will describe the layout and contents of our form.

Let's get started creating a general settings form for ExpressionEngine. We'll start with a couple text inputs, as well as some other code necessary to render the view:

    // The data we'll want to populate our form fields with
    $site = ee('Model')->get('Site')
        ->filter('site_id', ee()->config->item('site_id'))
        ->first();

    // Form definition array
    $vars['sections'] = array(
      array(
        array(
          'title' => 'site_name',
          'fields' => array(
            'site_name' => array(
              'type' => 'text',
              'value' => $site->site_label,
              'required' => TRUE
            )
          )
        ),
        // Site short name field
        array(
          'title' => 'site_short_name',
          'desc' => 'site_short_name_desc',
          'fields' => array(
            'site_short_name' => array(
              'type' => 'text',
              'value' => $site->site_name,
              'required' => TRUE
            )
          )
        )
      )
    );

    // Final view variables we need to render the form
    $vars += array(
      'base_url' => ee('CP/URL', 'settings/general'),
      'cp_page_title' => lang('general_settings'),
      'save_btn_text' => 'btn_save_settings',
      'save_btn_text_working' => 'btn_saving'
    );

We can then render the form by returning this at the end of our controller method:

    return ee('View')->make('ee:_shared/form')->render($vars);

Assuming our language keys are defined above, we should end up with a form that looks like this:

![](_images/shared-form-1.png)

## Fieldset definitions

Let's dive in closer and take a look and what makes a fieldset definition:

    array(
      'title' => 'site_name',
      'desc' => 'site_name_desc',
      'fields' => array( ... )
    )

This is the first level in a fieldset definition. Here are what these keys and values mean, as well as others that can be set in this dimension of the array:

|Option name|Description|Accepted values|Default value|
|--- |--- |--- |--- |
|title|Name of field, required.|String|	N/A|
|desc|Description of field, required.|String|N/A|
|fields|Array of field definitions, documented below, required.|Array|N/A|
|security|Marks a setting field as potentially increasing site security, and applies the security enhance style [as shown in the style guide](https://ellislab.com/style-guide/c/forms#3-18-15-1051-am).|Boolean|FALSE|
|caution|	Marks a setting field as potentially decreasing site security, and applies the security caution style [as shown in the style guide](https://ellislab.com/style-guide/c/forms#setting-field-security-caution).|Boolean|FALSE|
|grid|	Whether or not this fieldset is to have a Grid input, such as one generated by the GridInput service. The fieldset needs some extra styles and markup handling to show a Grid field.|Boolean|FALSE|
|attrs|Specify any extra attributes such as classes or data attributes on the parent fieldset element of the field(s). An array can be passed in the format of `array('attr-name' => 'value')` and multiple attributes can be specified.|Array|N/A|
|group|Specify the group name this fieldset should be included in. See [Toggling field visibility](#toggling-field-visibility) for more information.|String|N/A|

## Individual field definitions

Fieldsets can contain multiple fields, and they are defined in the `fields` array mentioned above:

    'fields' => array(
      'site_name' => array(
        'type' => 'text',
        'value' => $site->site_label,
        'required' => TRUE
      )
    )

The key for each field defintiion is the field's input name. We'll dive deeper into that array to see how we can show and customize different kinds of fields. Here are the keys available to a field definition array:

|Option name|Description|Accepted values|Default value|
|--- |--- |--- |--- |
|type|Type of field, required. All field types are listed below.|String name of valid field type names|N/A|
|value|Value of field to populate on page load.|String (or Array when type is ‘checkbox’)|	N/A|
|required|Whether or not the field is required for form submission, applies the required style [as shown in the style guide](https://ellislab.com/style-%20guide/c/forms#setting-field-required).|Boolean|FALSE|
|disabled|Whether or not the field input element is disabled.|Boolean|FALSE|
|choices|For checkboxes, radio buttons and select fields, sets the selectable choices for that field. Array format is `'my_value' => lang('my_label')`. If you need instructional text, structure your array like this: <pre>'my_value' => [<br>'label' => lang('my_label'),<br>'instructions' => lang('my_instructions')<br>]</pre>|Array||
|disabled_choices|For checkboxes, indicates options that are not currently choosable with an array of field values whose checkboxes should be disabled, e.g. `['value', 'another']`|Array|NULL|
|maxlength|Sets the maxlength= attribute on text inputs.|Boolean|FALSE|
|placeholder|	Sets the placeholder= attribute on text inputs.|String|NULL|
no_results|For checkboxes, radio buttons and select fields, can be set to show a “no results” message and a call-to-action link button to create content that would populate options for the field.|Array|NULL|
|label|Normally, the label for the field is specified in the fieldset definition, but some field types may allow a secondary label to be set such as the short-text field because it is normally paired with other short-text fields and each may need their own label.|String|NULL|
|encode|	For checkboxes, radio buttons and select fields, whether or not to encode the items’ display value for security.|Boolean	|TRUE|
|content|	When type is set to html, allows for any freeform markup to be used as the field.|String|NULL|
|group_toggle|If this field is to toggle the visibility of other fields, specifies the rules for that toggling. See [Toggling field visibility](#toggling-field-visibility) for more information.|Array|	N/A|

## Available field input types

Here are the values available to the `type` key documented above:

|Field name|Description|
|--- |--- |
|text|Regular text input.|
|short-text|Small text input, typically used when a fieldset needs multiple small, normally numeric, values set.|
|textarea|Textarea input.|
|select|Select dropdown input.|
|dropdown|A rich select dropdown .|
|checkbox|Checkboxes displayed in a vertical list.|
|radio|Radio buttons displayed in a vertical list.|
|yes_no|A Toggle control that returns either y or n respectively.|
|file|File input. Requires filepicker configuration.|
|image|Image input. Like file but shows an image thumbnail of the selected image as well as controls to edit or remove. Requires filepicker configuration.|
|password|Password input.|
|hidden|Hidden input.|
|html|Freeform HTML can be passed in via the content key in the field definition to have a custom input field.|

Given what we now know about how to define field definitions and the types of fields available, let's add a few more fields to our form:

    $vars['sections'] = array(
      array(
        array(
          'title' => 'site_name',
          'fields' => array(
            'site_name' => array(
              'type' => 'text',
              'value' => $site->site_label,
              'required' => TRUE
            )
          )
        ),
        array(
          'title' => 'site_short_name',
          'desc' => 'site_short_name_desc',
          'fields' => array(
            'site_short_name' => array(
              'type' => 'text',
              'value' => $site->site_name,
              'required' => TRUE
            )
          )
        ),
        array(
          'title' => 'site_online',
          'desc' => 'site_online_desc',
          'fields' => array(
            'is_system_on' => array(
              'type' => 'inline_radio',
              'choices' => array(
                'y' => 'online',
                'n' => 'offline'
              )
            )
          )
        )
      ),
      'date_time_settings' => array(
        array(
          'title' => 'timezone',
          'desc' => 'timezone_desc',
          'fields' => array(
            'default_site_timezone' => array(
              'type' => 'html',
              'content' => ee()->localize->timezone_menu(
                set_value('default_site_timezone') ?: ee()->config->item('default_site_timezone')
              )
            )
          )
        ),
        array(
          'title' => 'date_time_fmt',
          'desc' => 'date_time_fmt_desc',
          'fields' => array(
            'date_format' => array(
              'type' => 'select',
              'choices' => array(
                '%n/%j/%y' => 'mm/dd/yy',
                '%j-%n-%y' => 'dd-mm-yy',
                '%Y-%m-%d' => 'yyyy-mm-dd'
              )
            ),
            'time_format' => array(
              'type' => 'select',
              'choices' => array(
                '24' => lang('24_hour'),
                '12' => lang('12_hour')
              )
            )
          )
        ),
        array(
          'title' => 'include_seconds',
          'desc' => 'include_seconds_desc',
          'fields' => array(
            'include_seconds' => array('type' => 'yes_no')
          )
        )
      )
    );

Notice we've made use of many more field types here. Also notice, we aren't setting values on many of our new fields, that's because we're working with site-wide configuration settings. **When a value is not specified for a field, the shared form view automatically looks in the site's configuration for a value.**

With these additions, our form should now look like this:

![](_images/shared-form-2.png)

Our form is fully rendered and ready to write a form handler for without having to write any markup.

## Toggling field visibility

Sometimes you may want to toggle the visibility of certain fields based on the value of another field. A common case is selecting an option in a dropdown or radio button and having a different set of fields appear or disappear. This can be achieved automagically with form groups. Take this small example. We have a small form with a select field, then two sections of fields we want to show based on the value of the select box. Here's how we would construct this form normally:

    $vars['sections'] = array(
      array(
        array(
          'title' => 'type',
          'fields' => array(
            'type' => array(
              'type' => 'select',
              'choices' => array(
                'text' => lang('text'),
                'image' => lang('image')
              ),
              'value' => $type
            )
          )
        ),
      ),
      'text_options' => array(
        array(
          'title' => 'text',
          'fields' => array(
            'text' => array(
              'type' => 'text',
              'value' => $text
            )
          )
        ),
      ),
      'image_options' => array(
        array(
          'title' => 'image_path',
          'fields' => array(
            'image_path' => array(
              'type' => 'text',
              'value' => $image_path
            )
          )
        )
      )
    );

Pretty standard form. Now we want to modify it to give it the logical groupings we want and to specify which field is going to control the toggling:

    $vars['sections'] = array(
      array(
        array(
          'title' => 'type',
          'fields' => array(
            'type' => array(
              'type' => 'select',
              'choices' => array(
                'text' => lang('text'),
                'image' => lang('image')
              ),
              'group_toggle' => array(
                'text' => 'text_options',
                'image' => 'image_options'
              ),
              'value' => $type
            )
          )
        ),
      ),
      'text_options' => array(
        'group' => 'text_options',
        'settings' => array(
          array(
            'title' => 'text',
            'fields' => array(
              'text' => array(
                'type' => 'text',
                'value' => $text
              )
            )
          ),
        )
      ),
      'image_options' => array(
        'group' => 'image_options',
        'settings' => array(
          array(
            'title' => 'image_path',
            'fields' => array(
              'image_path' => array(
                'type' => 'text',
                'value' => $image_path
              )
            )
          )
        )
      )
    );

Notice what's different. We added a `group_toggle` key to the select's field definition that specifies for each value of the select dropdown which group to show. Next, we needed to specify what fields are in which group. We did that by adding a `group` key to each section, and then nesting those field definitions under a `settings` key. If we have multiple fields in that section, they will all be shown/hidden based on the value of our `group_toggle` field. If we just want to tag specific settings to toggle and not an entire section, we can set the group key on a fieldset definition and allow all the fields to share the same section of the form:

    $vars['sections'] = array(
      array(
        array(
          'title' => 'type',
          'fields' => array(
            'type' => array(
              'type' => 'select',
              'choices' => array(
                'text' => lang('text'),
                'image' => lang('image')
              ),
              'group_toggle' => array(
                'text' => 'text_options',
                'image' => 'image_options'
              ),
              'value' => $type
            )
          )
        ),
        array(
          'title' => 'text',
          'group' => 'text_options',
          'fields' => array(
            'text' => array(
              'type' => 'text',
              'value' => $text
            )
          )
        ),
        array(
          'title' => 'image_path',
          'group' => 'image_options',
          'fields' => array(
            'image_path' => array(
              'type' => 'text',
              'value' => $image_path
            )
          )
        )
      )
    );

Finally, we must include the JavaScript to make it all work:

    ee()->cp->add_js_script(array(
      'file' => array('cp/form_group'),
    ));

## Form validation

The shared form view is ready to take a form validation result object from the [Validation Service](development/services/validation.md). After you receive the result object, simply assign it the view's variable's array:

    $vars['errors'] = $result;

As long is the field names in validation match up with the form input names, the shared form view will automatically show error messages next to their respective fields and apply the appropriate [styles denoting field errors](https://ellislab.com/style-guide/c/forms#setting-field-errors).

## Tabs

The shared form view is capable of adding tabs in accordance with our [style guide](https://ellislab.com/style-guide/c/tabs). To do so assign a `tabs` variable to the view's variable's array:

    $vars['tabs'] = $tabs;

The view expects the `tabs` variable to be an associative array where the key is the language key for the text of the tab and the value is the rendered HTML of the tab itself:

    $tabs = array(
      'hello_world' => '<h2>Hello world!</h2>',
      'goodbye'     => '<p>What, so soon?</p>'
    );

Adding forms to tabs is a matter of rendering our form data to HTML. In many cases this simply means rendering a sections array:

    $permissions_tab = '';

    // Assuming $sections looks like $var['sections'] as above
    foreach ($sections as $name => $settings)
    {
      $permissions_tab .= ee('View')->make('ee:_shared/form/section')
        ->render(array('name' => $name, 'settings' => $settings));
    }

    $var['tabs'] = array(
      'permissions' => $permissions_tab
    );