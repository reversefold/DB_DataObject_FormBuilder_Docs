First create your DataObjects and their ini files (the links ini file in particular if you want links displayed as select boxes). See [Setting Up DB_DataObject](Setting Up DB_DataObject).

	:::php
	<?php
	require_once('DB/DataObject/FormBuilder.php');
	//set up DB_DataObject config settings
	$do =& DB_DataObject::factory('someTable');
	$formBuilder =& DB_DataObject_FormBuilder::create($do);
	$form =& $formBuilder->getForm();
	if($form->validate()) {
	  $form->process(array(&$formBuilder, 'processForm'), false);
	  //this is optional, you can comment out this to make the form appear for editing again
	  $form->freeze();
	}
	echo $form->toHtml();
	?>



