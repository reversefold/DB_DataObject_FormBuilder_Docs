FormBuilder may be configured in one of five ways.

## FormBuilder Member Variables in Extended Class


The first way is by setting member variables of the FormBuilder object in an extended class in a var declaration or in the constructor. These values will be used as defaults. You can choose with this option whether the variables set here will override the options passed into ''$options'' and the ones set with ''PEAR::setStaticProperty()'' by moving the call to the parent constructor to the beginning or end (or maybe in the middle).

Put this code in MyFormBuilder.php

	:::php
	<?php
	require_once('DB/DataObject/FormBuilder.php');
	
	class DB_DataObject_FormBuilder_MyFormBuilder extends DB_DataObject_FormBuilder{
	  var $createSumbit = true;
	  function DB_DataObject_FormBuilder_MyDriver(&$do, $options = false) {
	    // Calling the parent at the beginning of the constructor causes the options
	    // set below to override options set in $options and PEAR::setStaticProperty()
	    parent::DB_DataObject_FormBuilder($do, $options);
	    $this->linkDispayLevel = 2;
	    // Calling the parent in the middle of the constructor allows $options and
	    // PEAR::setStaticProperty() to only override the options set before
	    $this->elementTypeMap['date'] = 'myDate';
	    // Calling the parent at the end of the constructor allows $options and
	    // PEAR::setStaticProperty() to override the options set above
	    //parent::DB_DataObject_FormBuilder($do, $options);
	  }
	
	  //this is important. This needs to be done to insure your class is used
	  function &create(&$do, $options = false, $driver = 'QuickForm') {
	    parent::create($do, $options, $driver, 'db_dataobject_formbuilder_myformbuilder');
	  }
	}
	?>



and to use this you would do:

	:::php
	<?php
	require_once('MyFormBuilder.php');
	$do =& DB_DataObject::factory('table');
	$fb =& DB_DataObject_FormBuilder_MyFormBuilder::create($do);
	?>



## PEAR::setStaticProperty() / DataObject.ini



The second way FormBuilder may be configured is through ''PEAR::setStaticProperty()''. This is the same way that DB_DataObject is configured. The easiest way to configure is:

	:::php
	<?php
	$options =& PEAR::getStaticProperty('DB_DataObject_FormBuilder', 'options');
	$options = array('createSubmit' => 1,
	                 'linkDisplayLevel' => 2,
	                 'elementTypeMap' => array('shorttext' => 'text',
	                                           'date' => 'myDate'));
	?>



Yes, the above is correct. The first line gives you a reference to the options property then you set it in the next line.

You can also use this with an ini file. The most common way is to add your options to your DataObject.ini (the same file which holds your DB_DataObject settings). Array-based options should use the format "key:value,key2:value2,etc..."

	
	[DB_DataObject_FormBuilder]
	createSubmit=1
	linkDisplayLevel=2
	elementTypeMap=shorttext:text,date:myDate



If you use the ini file method make sure you use code similar to this to set the options:

	:::php
	<?php
	foreach ($config as $class => $values) {
	     $options =& PEAR::getStaticProperty($class, 'options'); 
	     $options = $values;
	}
	?>



This way both your DB_DataObject and FormBuilder options will get set correctly.

## Array passed to create()


The third way is to pass in an assoc array as the second argument to the create function. This will override setting set by ''PEAR::setStaticProperty()''.

	:::php
	<?php
	$do = DB_DataObject::factory('table');
	$options = array('createSubmit' => true,
	                 'linkDisplayLevel' => 2,
	                 'elementTypeMap' => array('shorttext' => 'text',
	                                           'date' => 'myDate'));
	$fb = DB_DataObject_FormBuilder::create($do, $options);
	?>



## FormBuilder Member Variables



The fourth way is to set the member variables on the FormBuilder object after it is instantiated, but before you create the form. These settings will override all above methods.

	:::php
	<?php
	$do = DB_DataObject::factory('table');
	$fb = DB_DataObject_FormBuilder::create($do);
	$fb->createSubmit = true;
	$fb->linkDisplayLevel = 2;
	$fb->elementTypeMap = array('shorttext' => 'text', 'date' => 'myDate');
	?>


## Member Variables in the DataObject



The fifth and last way to set these settings is in the DataObject itself. This is the preferred way to set most options. Take the name of any option and prefix it with "fb_" to get the name of the member in the DataObject to set (this was done to make clashes with field names less probable). Use these for table-specific settings. These may be set in the class definition, in preGenerateForm, or on the DataObject directly. Any changes made after preGenerateForm is called won't be used. This method overrides all of the others.

	:::php
	<?php
	class DataObject_Table extends DB_DataObject {
	  ###START_AUTOCODE
	  ...
	  ###END_AUTOCODE
	  var $fb_createSubmit = true;
	  var $fb_linkDisplayLevel = 2;
	  function preGenerateForm(&$fb) {
	    $fb->elementTypeMap = array('shorttext' => 'text', 'date' => 'myDate');
	  }
	}
	$do = DB_DataObject::factory('table');
	$do->fb_submitText = 'Submit!';
	$fb = DB_DataObject_FormBuilder::create($do);
	?>


## Overriding options per-page



There are plenty of places where you may want to override FormBuilder options on a per-page (or per instantiation) basis. Say, for example, that when you have a "customer" form you want to add some extra text to the "Username" label when the customer is registering but have the default show up on all other forms. In this case you would set the label in the DataObject as a member variable then override it on the registration page. The best way to override options per-page is to do it in the DataObject itself as DataObject options always override options set in the FormBuilder.

	:::php
	<?php
	$do =& DB_DataObject::factory('table');
	$do->fb_fieldLabels['field1'] = 'Special label for field1';
	$fb =& DB_DataObject_FormBuilder::create($do);
	?>



You can, of course, do this with any option. Please note, however, that this still may not override all options. If you set any options in the preGenerateForm callback in the DataObject even that will override the above. The only way to stop this is to change the callback to something else (or get rid of it). You can do this with the preGenerateFormCallback option.
