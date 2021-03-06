See [Configuring Formbuilder](Configuring Formbuilder) for how to set up these options. All of these options can be set in any way listed on that page.


# Scalars

Scalars are single-value options.

## addFormHeader

If true, a header will be added to the form at the top (see [#formHeaderText](#formHeaderText)).

## clientRules

If set to true, validation rules will also be client side.

## createSubmit

If set to 0, no submit button will be created for your forms. Useful when used together with QuickForm_Controller when you already have submit buttons for next/previous page. By default, a button is being generated with text from [#submitText](#submitText).

## crossLinkSeparator


The text to put between crosslink elements. Defaults to `<br/>`.

## dbDateFormat


This is for the future support of string date formats other than ISO, but currently, that's the only supported one. Set to 1 for ISO, other values may be available later on.

## dateElementFormat

A format string that represents the display settings for QuickForm date elements. Example: "d-m-Y". See QuickForm documentation for details on format strings. Legal letters to use in the format string that work with FormBuilder are:

d,m,M,y,Y

See [#dateFields](#dateFields), [#dateFieldLanguage](#dateFieldLanguage), [#dateTimeElementFormat](#dateTimeElementFormat), [#timeElementFormat](#timeElementFormat),  [#dateOptionsCallback](#dateOptionsCallback), [#dateTimeOptionsCallback](#dateTimeOptionsCallback), and [#timeOptionsCallback](#timeOptionsCallback).

## dateFieldLanguage

The language to be used in date fields (see HTML_QuickForm documentation on the date element for more details).

## dateTimeElementFormat

A format string that represents the display settings for QuickForm datetime elements. Example: "d-m-Y H:i:s". See QuickForm documentation for details on format strings. Legal letters to use in the format string that work with FormBuilder are:

d,m,M,y,Y,H,i,s

See [#dateFields](#dateFields), [#dateFieldLanguage](#dateFieldLanguage), [#dateElementFormat](#dateElementFormat), [#timeElementFormat](#timeElementFormat),  [#dateOptionsCallback](#dateOptionsCallback), [#dateTimeOptionsCallback](#dateTimeOptionsCallback), and [#timeOptionsCallback](#timeOptionsCallback).

## elementNamePrefix

A string to prepend to element names. Together with [#elementNamePostfix](#elementNamePostfix), this option allows you to alter the form element names that FormBuilder uses to create and process elements. The main use for this is to combine multiple forms into one. For example, if you wanted to use multiple FB forms for the same table within one actual HTML form you could do something like this:

	:::php
	<?php
	$do = DB_DataObject::factory('table');
	$do->get(5);
	$fb = DB_DataObject_FormBuilder::create($do);
	$fb->elementNamePrefix = 'formOne';
	$form = $fb->getForm();
	
	$do2 = DB_DataObject::factory('table');
	$do2->get(13);
	$fb2 = DB_DataObject_FormBuilder::create($do2);
	$fb2->elementNamePrefix = 'formTwo';
	$fb2->useForm($form);
	$form = $fb2->getForm();
	
	if ($form->validate()) {
	    $form->process(array(&$fb, 'processForm'), false);
	    $form->process(array(&$fb2, 'processForm'), false);
	    $form->freeze();
	}
	?>
	
	$form->display();



If you assume that "table" has one field, "name", then the resultant form will have two elements: "formOnename" and "formTwoname".

Please note: You **cannot** use the [ or ] anywhere in the prefix or postfix. Doing so will cause FormBuilder to not be able to process the form.

## elementNamePostfix


A string to add to the end of element names.

See [#elementNamePrefix](#elementNamePrefix) for a full explanation.


## formHeaderText


Text for the form header (if added with [#addFormHeader](#addFormHeader)). If null, the class name of the dataobject will be used.

## hidePrimaryKey


By default, hidden fields are generated for the primary key of a DataObject. This behaviour can be deactivated by setting this option to false.

## linkDisplayLevel


If this is set to 1 or above, links will be followed in the display fields and the display fields of the record linked to will be used for display. If this is set to 2, links will be followed in the linked record as well. This can be set to any number of links you wish but could easily slow down your application if set to more than 1 or 2 (but only if you have links in your display fields that go that far ;-)). For a more in-depth example, see the docs for linkDisplayFields.

## linkNewValue


If set to true, all links which are rendered as select boxes (see [#linkElementTypes](#linkElementTypes)) will have a "--New Value--" option on the bottom which displays a form for the linked to table for creating a new record. Upon submit, the sub-form will be checked for validity as normal and, if valid, will be used to enter a new record in the linked-to table which this record will now link to.

If set to false, all link select boxes will only have entered options (as normal)

If set to true, all link select boxes will have a New Value entry

You may also set this to be an array with the names of the link fields to create New Value entries for.

## linkNewValueText

This is the text that is shown in a select for the [#linkNewValue](#linkNewValue) option. Make sure this text is unique to all of the options that may be in your drop-down.

## requiredRuleMessage

Text that is displayed as an error message if a required field is left empty. Use %s to insert the field name.

## ruleViolationMessage

Text that is displayed as an error message if a validation rule is violated by the user's input. Use %s to insert the field name.

## selectAddEmptyLabel

The text to put in the empty option created by selectAddEmpty. See the [#selectAddEmpty](#selectAddEmpty) option for more.

## submitText

The caption of the submit button, if created.

## timeElementFormat



A format string that represents the display settings for QuickFormtime elements.Example: "H:i:s". See QuickForm documentation for details on format strings.

## useAccessors

Set to true to use accessor methods (getters) for accessing field values, if they exist.

If this is set to true and a method exists of the name getFieldName where
FieldName is the name of the field then it will be called to get the value
of the field.

Note: The following field names will not work with getters due to function collisions
in DB_DataObject. Do not use fields with these names in conjunction with useAccessors:

*  Link

*  Links

*  LinkArray

*  DatabaseConnection

*  DatabaseResult

Note: Accessors may not be used to get link field values. Link field values are
internal to a database and are assumed not to need accessors.

## useMutators

Set to true to use mutator methods (setters) for setting field values, if they exist.

If this is set to true and a method exists of the name setFieldName where
FieldName is the name of the field then it will be called to set the value
of the field.

Note: Do not use a field named From if you use this option. DB_DataObject
has a default method setFrom which will cause problems.

Note: Mutators may not be used to set link fields. Link field values are internal
to a database and are assumed not to need mutators.


## useCallTimePassByReference



Defaults to false. If set to true, all [Callback methods](Callback methods) which have pass-by-ref set will be called with a & before the parameter name. This forces pass by reference with a feature called "call time pass by reference". This is normally not needed in PHP4 (and shouldnever be needed in PHP5), but if you are using overloading with your DataObjects or have other problems with the call by reference parameters, you should set this to true. If you set this to true you should also set the "allow_call_time_pass_reference" option in your php.ini to "On" or use ini_set() in your script to set it or you will see warnings about this being deprecated.

## validateOnProcess



If set to TRUE, the current DataObject's validate method is called before the form data is processed. If errors occur, no insert/update operation will be made on the database. Use getValidationErrors() to retrieve the reasons for a failure.

Defaults to FALSE.

# Callbacks



Callbacks are pointers to functions which will be used in certain circumstances.

## dateFromDatabaseCallback



A valid callback which converts a database formatted date to the format needed for the form element. Defaults to DB_DataObject_FormBuilder::_date2array().

## dateToDatabaseCallback



A valid callback which converts a submitted date to the format needed for the database. Defaults to DB_DataObject_FormBuilder::_array2date().

## enumOptionsCallback



A valid callback which will return the options in a simple array of strings for an enum field given the table and field names. The option array returned **must** have the value to be put into the DB as the key of the array. See the enumOptions section for an example. The following is an example based on the callback built into FormBuilder. It will likely only work on mysql.

	:::php
	<?php
	function getEnumOptions($table, $field) {
	    $db = DB::connect('mysql://user:pass@host/db');
	    $option = $db->getRow('SHOW COLUMNS FROM '.$db->quoteIdentifier($table).
	       ' LIKE '.$db->quoteSmart($field), DB_FETCHMODE_ASSOC);
	    $option = substr($option['Type'], strpos($option['Type'], '(') + 1);
	    $option = substr($option, 0, strrpos($option, ')') - strlen($option));
	    $split = explode(',', $option);
	    $options = array();
	    $option = '';
	    for ($i = 0; $i < sizeof($split); ++$i) {
	        $option .= $split[$i];
	        if (substr_count($option, "'") % 2 == 0) {
	            $option = trim(trim($option), "'");
	            $options[$option] = $option;
	            $option = '';
	        }
	    }
	    return $options;
	}
	?>



This will only be used if the field is listed in enumFields and if there are no options defined in enumOptions.

See [#enumFields](#enumFields) and [#enumOptions](#enumOptions) for more.

## preGenerateFormCallback



## postGenerateFormCallback



## preProcessFormCallback



## postProcessFormCallback



## prepareLinkedDataObjectCallback

You can use these 5 options to alter the callbacks used for preGenerateForm, postGenerateForm, preProcessForm, postProcessForm, or prepareLinkedDataObject. By default these functions are located in the DataObject. See [Callback methods](Callback methods) for more on these functions.


## dateOptionsCallback

## dateTimeOptionsCallback

## timeOptionsCalllback

Callbacks to get extra options for respectively date, date+time & time QuickForm elements. Default callbacks are respectively dateOptions(), dateTimeOptions(), and timeOptions() in the DO.


# Arrays

Arrays are multi-value options.

## booleanFields



An array of the field names that you want to force to be a boolean type. These fields will be shown as checkboxed.

## crossLinks



The crossLinks array holds data pertaining to many-many links. If you have a table which links two tables together, you can use this to automatically create a set of checkboxes or a multi-select on your form. The simplest way of using this is:

	:::php
	
	<?php
	class DataObject_SomeTable extends DB_DataObject {
	  //...
	  var $fb_crossLinks = array(array('table' => 'crossLinkTable'));
	}
	?>
	



Where crossLinkTable is the name of the linking table. You can have as many cross-link entries as you want. Try it with just the table entry first. If it doesn't work, you can specify the fields to use as well.

	:::php
	'fromField' => 'linkFieldToCurrentTable' //This is the field which links to the current (from) table
	'toField' => 'linkFieldToLinkedTable' //This is the field which links to the "other" (to) table



To get a multi-select add a 'type' key set to 'select'.

	:::php
	<?php
	class DataObject_SomeTable extends DB_DataObject {
	  //...
	  var $fb_crossLinks = array(array('table' => 'crossLinkTable',
	                                   'type' => 'select'));
	}
	?>



An example: I have a user table and a group table, each with a primary key called id. There is a table called user_group which has fields user_id and group_id which are set up as links to user and group. Here's the configuration array that could go in both the user DO and the group DO:

	:::php
	<?php
	$fb_crossLinks = array(array('table' => 'user_group'));
	?>



Here is the full configuration for the user DO:

	:::php
	<?php
	$fb_crossLinks = array(array('table' => 'user_group',
	                             'fromField' => 'user_id',
	                             'toField' => 'group_id'));
	?>



And the full configuration for the group DO:

	:::php
	<?php
	$fb_crossLinks = array(array('table' => 'user_group',
	                             'fromField' => 'group_id',
	                             'toField' => 'user_id'));
	?>



crossLinks can also be automatically collapsed only to the selected records by setting the 'collapse' key to true. The collapsed options can be viewed again by clicking the "Show All" link.

You can also specify the separator between the elements with crossLinkSeperator.

To give the crossLink a Label use the fieldLabels array. As key use %%__crossLink_tableName%% where tableName is the name of the linking table.
For example:

	:::php
	<?php
	function preGenerateForm(&$fg) {
	  $this->fb_fieldLabels = 
	    array(
	      '__crossLink_user_group' => 'groups the user belongs to',
	    );
	}
	?>



You can also specify extra fields to edit in the crosslink table with the crossLinkExtraFields option in the crossLink DataObject. See below for more.

##  crossLinkExtraFields

You can also specify extra fields to edit in the a crossLink table with this option. For example, if the user_group table mentioned above had another field called 'role' which was a text field, you could specify it like this in the user_groupDataObject class:

	:::php
	<?php
	class DataObject_User_group extends DB_DataObject {
	  //normal stuff here
	  
	  var $fb_crossLinkExtraFields = array('role');
	}
	?>



This would cause a text box to show up next to each checkbox in the user_group section of the form for the field 'role'.

You can specify as many fields as you want in the 'extraFields' array.

Note: If you specify a linked field in 'extraFields' you'll get a select box just like when you do a normal link field in a FormBuilder form. :-)

## dateFields

A simple array of field names indicating which of the fields in a particular table/class are actually to be treated date fields. This is an unfortunate workaround that is necessary because the DataObject generator script does not make a difference between any other datatypes than string and integer. When it does, this can be dropped.

## elementTypeAttributes

This option is specific to the HTML_QuickForm driver.

Array of attributes for each element type. See the keys of [#elementTypeMap](#elementTypeMap) for the allowed element types The key is the element type. The value can be a valid attribute string or an associative array of attributes.

	:::php
	<?php
	$fb->elementTypeAttributes = array('longtext' => array('rows' => 10, 'cols' => 80));
	?>

See also [#fieldAttributes](#fieldAttributes).

## elementTypeMap

This option is specific to the HTML_QuickForm driver.

Array to determine what QuickForm element types will be used for which general field types. If you configure FormBuilder using arrays, the format is:

	:::php
	array('nameOfFieldType' => 'QuickForm_Element_name', ...);



If configured via .ini file, the format looks like this:

	:::php
	elementTypeMap = shorttext:text,date:date,...



Allowed field types:


*  shorttext

*  longtext

*  date

*  datetime

*  integer

*  float

*  select

*  multiselect

*  subForm

*  elementTable


As of version 0.17.1 you don't have to specify all of the types when setting up an elementTypeMap. If you don't specify a type the default will be used.

## enumFields

A simple array of fields names which should be treated as ENUMs. A select box will be created with the enum options. If you add this field to the [#linkElementTypes](#linkElementTypes) array and give it a 'radio' type, you will get radio buttons instead.

The default handler for enums is only tested in mysql. If you are using a different DB backend, use [#enumOptionsCallback](#enumOptionsCallback) or [#enumOptions](#enumOptions).

## enumOptions

An array which holds enum options for specific fields. Each key should be a field in the current table and each value holds a an array of strings which are the possible values for the enum. This will only be used if the field is listed in enumFields. The option array must be have the value which is to be put in the DB as the key. For a normal ENUM field this should look like this:

	:::php
	<?php
	class DataObject_SomeTable extends DB_DataObject {
	  //...
	  var $fb_enumFields = array('enumField');
	  var $fb_enumOptions = array('enumField' => array('option1' => 'option1',
	                                                   'option2' => 'option2',
	                                                   'optionTheThird' => 'optionTheThird'));
	}
	?>



## fieldAttributes

This option is specific to the HTML_QuickForm driver.

Array of attributes for each specific field. The key is the field name. The value can be a valid attribute string or an associative array of attributes.

	:::php
	<?php
	class DataObject_SomeTable extends DB_DataObject {
	  //...
	  var $fb_fieldAttributes = array('multiSelectField' => array('size' => 5),
	                                  'textField' => array('length' => '20'),
	                                  'anotherElement' => 'style="background-color: blue;"');
	}
	?>

See also [#elementTypeAttributes](#elementTypeAttributes).

## fieldLabels



Array of field labels. The key of the element represents the field name. Use this if you want to keep the auto-generated elements, but still define your own labels for them.

## fieldsToRender



Array of fields to render elements for. If a field is not given, it will not be rendered. If empty, all fields will be rendered (except, normally, the primary key).

## fieldsRequired

Any fields in this array will be treated as if they are NOT NULL. This means that they will be automatically set as required (this can be overridden with excludeFromAutoRules) and, if they are links or enums, will not have an empty option automatically added to them. If you wish these fields to have an empty option, you can add them to [#selectAddEmpty](#selectAddEmpty) just as normal NOT NULL fields.


## linkDisplayFields (used to be selectDisplayFields)



These fields will be used when displaying a link record. The fields listed will be seperated by ", ". If you specify a link field as a display field and linkDisplayLevel is not 0, the link will be followed and the display fields of the record linked to displayed within parenthesis.

For example, say we have these tables:

	
	[person]
	id = 129
	name = 130
	gender_id = 129
	
	[gender]
	id = 129
	name = 130



this link:

	
	[person]
	gender_id = gender:id



and this data:

	
	Person:
	 id: 2
	  name: "Justin Patrin"
	  gender_id: 1
	Gender:
	  id: 1
	  name: "male"



If person's display fields are:

	:::php
	<?php
	class DataObject_Person extends DB_DataObject {
	  //...
	  var $fb_linkDisplayFields = array('name', 'gender_id');
	}
	?>



and gender's display fields are:

	:::php
	<?php
	class DataObject_Gender extends DB_DataObject {
	  //...
	  var $fb_linkDisplayFields = array('name');
	}
	?>



and we set linkDisplayLevel to 0, the person record will be displayed as:

"Justin Patrin, 1"

If we set linkDisplayLevel to 1 or above, the person record will be displayed as:

"Justin Patrin, (male)"

If you go one step further and have a table which links to a person, say:

	
	[group]
	name = 130
	leader_id = 129



link:

	
	[group]
	leader_id = person:id



data:

	
	Group:
	  name: "ReverseFold users"
	  leader_id: 2



And the group's display fields are:

	:::php
	<?php
	class DataObject_Group extends DB_DataObject {
	  //...
	  var $fb_linkDisplayFields = array('name', 'leader_id');
	}
	?>



and we set linkDisplayLevel to 0, the group record will be displayed as:

"ReverseFold users, 2" If we set linkDisplayLevel to 1, the record will be displayed as: "ReverseFold users, (Justin Patrin, 1)" If we set linkDisplayLevel to 2 or above, the record will be displayed as: "ReverseFold users, (Justin Patrin, (male))"!

## linkElementTypes



Array to configure the type of the link elements. By default, a select box will be used. The key is the name of the link element. The value is 'radio' or 'select'. If you choose 'radio', radio buttons will be made instead of a select box.

	:::php
	<?php
	class DataObject_SomeTable extends DB_DataObject {
	  //..
	  var $fb_linkElementTypes = array('linkField' => 'radio');
	}
	?>



The name of the group will be the name of the element.

See [#linkNewValue](#linkNewValue) & [#enumFields](#enumFields).

## linkOrderFields (used to be selectOrderFields)


The fields to be used for sorting the options of an auto-generated link element. You can specify ASC and DESC in these options as well:

	:::php
	<?php
	class DataObject_SomeTable extends DB_DataObject {
	  //...
	  var $fb_linkOrderFields = array('field1', 'field2 DESC');
	}
	?>
	



You may also want to escape the field names if they are reserved words in the database you're using:

	:::php
	<?php
	class DataObject_SomeTable extends DB_DataObject {
	  //...
	  function preGenerateForm() {
	    $db = $this->getDatabaseConnection();
	    $this->fb_linkOrderFields = array($db->quoteIdentifier('config'), $db->quoteIdentifier('select').' DESC');
	  }
	}
	?>



## preDefElements



Array of user-defined QuickForm elements that will be used for the field matching the array key. If no match is found, the element for that field will be auto-generated. Make your element objects either in the preGenerateForm() method or in the getForm() method. Use HTML_QuickForm::createElement() to do this.

If you wish to put in a group of elements in place of a single element, you can put an array in preDefElements instead of a single element. The name of the group will be the name of the replaced element.

## preDefGroups



Array of groups to put certain elements in. The key is an element name, the value is the group to put the element in.

	:::php
	<?php
	class DataObject_SomeTable extends DB_DataObject {
	  //...
	  var $fb_preDefGroups = array('field1' => 'group1', 'field2' => 'group1', 'field3' => 'group2', 'field4' => 'group2');
	}
	?>



This will put field1 and field2 in a group and field3 and field4 in another group.

## preDefOrder

Indexed array of element names. If defined, this will determine the order in which the form elements are being created. This is useful if you're using QuickForm's default renderer or dynamic templates and the order of the fields in the database doesn't match your needs.

## reverseLinks

A reverseLink is a table which links back to the current table. For example, let say we have a "gender" table which has Male and Female in it and a "person" table which has the fields "name", which holds the person's name and "genre_id" which links to the genre. If you set up a form for the gender table, the person table can be a reverseLink. The setup in the gender table would look like this:

	:::php
	<?php
	class DataObject_Gender extends DB_DataObject {
	  //normal stuff here
	
	  var $fb_reverseLinks = array(array('table' => 'person'));
	}
	?>



Now a list of people will be shown in the gender form with a checkbox next to it which is checked if the record currently links to the one you're editing. In addition, some special text will be added to the end of the label for the person record if it's linked to another gender.

Say we have a person record with the name "Justin Patrin" which is linked to the gender "Male". If you view the form for the gender "Male", the checkbox next to "Justin Patrin" will be checked. If you choose the "Female" gender the checkbox will be unchecked and it will say:

Justin Patrin **- currently linked to - Male**

If the link field is set as NOT NULL then FormBuilder will not process and unchecked checkbox unless you specify a default value to set the link to. If null is allowed, the link will be set to NULL. To specify a default value:

	:::php
	<?php
	class DataObject_Gender extends DB_DataObject {
	  //normal stuff here
	
	  var $fb_reverseLinks = array(array('table' => 'person',
	                                     'defaultLinkValue' => 5));
	}
	?>



Now if a checkbox is unchecked the link field will be set to 5...whatever that means. Be careful here as you need to make sure you enter the correct value here (probably the value of the primary key of the record you want to link to by default).

You may also set the text which is displayed between the record and the currently linked to record.

	:::php
	<?php
	class DataObject_Gender extends DB_DataObject {
	  //normal stuff here
	
	  var $fb_reverseLinks = array(array('table' => 'person',
	                                     'linkText' => ' is currently listed as a '));
	}
	?>



If you select "Female" the Justin Patrin entry would now say:

Justin Patrin **is currently listed as a Male**

## selectAddEmpty

An array of the link fields which should have an empty option added to the select box. Date fields may also be specified, causing the select elements in a date element to have empty options.

See also [#selectAddEmptyLabel](#selectAddEmptyLabel) & [#fieldsRequired](#fieldsRequired).

## textFields

A simple array of field names indicating which of the fields in a particular table/class are actually to be treated as textareas. This is an unfortunate workaround that is necessary because the DataObject generator script does not make a difference between any other datatypes than string and integer. When it does, this can be dropped.

## timeFields

A simple array of field names indicating which of the fields in a particular table/class are actually to be treated time fields. This is an unfortunate workaround that is necessary because the DataObject generator script does not make a difference between any other datatypes than string and integer. When it does, this can be dropped.

## tripleLinks

The tripleLinks array can be used to display checkboxes for "triple-links". A triple link is set up with a table which links to three different tables. These will show up as a table of checkboxes The initial setting (table) is the same as for crossLinks. The field configuration keys (if you need them) are:

	
	'fromField'
	'toField1'
	'toField2'



## userEditableFields

Array of fields which the user can edit. If a field is rendered but not specified in this array, it will be frozen. Ignored if not given.
