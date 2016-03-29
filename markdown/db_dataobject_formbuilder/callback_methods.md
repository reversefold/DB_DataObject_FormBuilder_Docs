These are methods which you can implement in your DataObject derived classes to perform operations at certain points of the form generation and processing. These allow you even more customization options than the simple options. You can even set FormBuilder options in these methods.

You can also change the function that is called by altering the callback options for each function. See the Callbacks section in [Formbuilder configuration options](Formbuilder configuration options).


# preGenerateForm(&$fb)

This method is called by FormBuilder::getForm() to offer you a last chance to change property values or options before the form is generated. The method parameter is the calling FormBuilder object instance, passed into this method by reference. You can access any settings or functions you need. This is where you could set up -for instance- fb_preDefElements.

To change properties, you can either modify your DataObject variables or directly edit the given $fb instance. In the former case the variables are prefixed with 'fb_' (see also the note below), in the latter one no prefix is used. Be forewarned, though, that if you set the same things in the DO and in the FB the ones set in the DO will overwrite those in the FB. As a rule of thumb, always set table-specific options in the DO so that if you add options to the DO later you won't be surprised by some other options seeming to disappear.

Note: Options defined in the DataObject ($fb_*) are **only** copied into the calling $fb instance **after** preGenerateForm() returns. This means that as long as preGenerateForm() executes, the provided $fb instance will not have taken into account those options. If you need them in preGenerateForm() or if you call FormBuilder functions which rely on them, call:

	:::php
	<?php
	class My_DO extends DB_DataObject {
	    //...normal stuff
	    function preGenerateForm(&$fb) {
	        $fb->populateOptions();
	        $this->fb_formHeaderText = 'This is a form for My_DO';
	        $fb->linkNewValue = false;
	    }
	}
	?>



# postGenerateForm(&$form, &$formBuilder)

This method will be called after the form is generated. The form is passed in by reference so you can alter it. Use this method to add, remove, or alter elements in the form or the form itself. The $form parameter is the form that was generated. Make sure to include the & in the method declaration.

	:::php
	<?php
	class My_DO extends DB_DataObject {
	    //...normal stuff
	    function postGenerateForm(&$form, &$formBuilder) {
	        $form->addElement('static', 'afterForm', 'This text comes after the form');
	        $form->setDefaults(array($formBuilder->getFieldName('myfield') => 'default value'));
	        $el =& $form->getElement($formBuilder->getFieldName('otherfield'));
	        $el->updateAttributes(array('class' => 'otherfieldClass'));
	    }
	}
	?>



# getForm($action, $target, $formName, $method, &$formBuilder)

If this function exists, it will be used instead of FormBuilder's internal form generation routines. Use this only if you want to create the entire form on your own. This effectively nullifies all of the automatic form building that this package does. Only use this if you absolutely must.

	:::php
	<?php
	class My_DO extends DB_DataObject {
	    //...normal stuff
	    function &getForm($action, $target, $formName, $method, &$formBuilder) {
	        $form = new HTML_QuickForm();
	        $form->addElement('text', 'fieldname');
	        return $form;
	    }
	}
	?>



# preProcessForm(&$values, &$formBuilder)

This method is called just before FormBuilder processes the submitted form data. The values are sent by reference in the first parameter as an associative array. The key is the element name and the value the submitted value. You can alter the values as you see fit (md5 on passwords, for example). The $values parameter is the array of values that will be used to populate the DataObject for updating or insertion in the DB. Make sure to include the & in the method declaration so that you can change the values.

	:::php
	<?php
	class My_DO extends DB_DataObject {
	    //...normal stuff
	    function preProcessForm(&$values, &$formBuilder) {
	        $values[$formBuilder->getFieldName('myfield')] = str_replace('bad', 'good', $values[$formBuilder->getFieldName('myfield')]);
	        $values[$formBuilder->getFieldName('passwordfield')] = md5($values[$formBuilder->getFieldName('passwordfield')]);
	    }
	}
	?>



# postProcessForm(&$values, &$formBuilder)

This method is called just after FormBuilder processed the submitted form data. The values are again sent by reference. This method could be used to inform the user of changes, alter the DataObject, etc. The $values parameter is the array of values that FormBuilder used to populate the DataObject.

	:::php
	<?php
	class My_DO extends DB_DataObject {
	    //...normal stuff
	    function postProcessForm(&$values, &$formBuilder) {
	        echo 'The following values were saved: '.print_r($values, true);
	    }
	}
	?>


# prepareLinkedDataObject(&$linkedDataObject, $field)



This method allows you to customize the way that linked records are selected.

For example, There is a table called 'project' which has the field 'defaultImage_id' which links to the 'image' table, and a table called 'image' which has the field 'project_id' which links to the project. Each image links to one project. When you select the default image for the project you want to only allow images which are linked to the project. To do this, you would add this function to your 'project' DataObject:

	:::php
	<?php
	class DataObject_Project extends DB_DataObject {
	  //auto-generated stuff
	
	  function prepareLinkedDataObject(&$linkedDataObject, $field) {
	    //you may want to include one or both of these
	    if ($linkedDataObject->tableName() == 'image' && $field == 'defaultImage_id') {
	      $linkedDataObject->project_id = $this->id;
	    }
	  }
	}
	?>



If this function doesn't seem to be working for you, try turning off DB_DataObject's overloading.

# dateOptions($field, &$fb)


# timeOptions($field, &$fb)

# dateTimeOptions($field, &$fb)

These three callbacks let you specify extra options for the three types of HTML_QuickForm date elements. The first parameter is the field name. The second is a reference to the FormBuilder object. The return should be an options array, same as you would pass into an HTML_QuickForm date element.

Example:

	:::php
	<?php
	function dateOptions($field, &$fb) {
	  return array('format' => 'Y-m-d',
	               'addEmptyOption' => true,
	               'emptyOptionText' => '--'
	              );
	}
	?>

