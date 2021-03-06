# Creating your own driver



Put this in DB/DataObject/FormBuilder/MyDriver.php

	:::php
	<?php
	require_once('DB/DataObject/FormBuilder/QuickForm.php');
	
	class DB_DataObject_FormBuilder_MyDriver extends DB_DataObject_FormBuilder_QuickForm {
	
	  var $createSubmit = false;
	
	  function DB_DataObject_FormBuilder_MyDriver(&$do, $options = false) {
	    parent::DB_DataObject_FormBuilder_QuickForm($do, $options);
	    $this->elementTypeMap['longtext'] = 'textarea';
	  }
	
	  function &getForm($action = false, $target = '_self', $formName = false, $method = 'post') {
	    $form =& parent::getForm($action, $target, $formName, $method);
	    $form->addRule('checkMe', 'Please check this checkbox to confirm your changed', 'required', null, 'client');
	    return $form;
	  }
	}
	?>



And to use it:

	:::php
	<?php
	require_once('DB/DataObject/FormBuilder.php');
	$do =& DB_DataObject::factory('table');
	$fb =& DB_DataObject_FormBuilder::create($do, false, 'MyDriver');
	?>



# Changing field contents



Pretend that you have a form with a password field (who has to pretend for that ;-)). To make the field a password field, you have to use preDefElements:

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //...
	    ###END AUTOCODE
	
	    function preGenerateForm() {
	        $this->fb_preDefElements['password'] = HTML_QuickForm::createElement('password', 'password', 'Password');
	    }
	}
	?>



That was simple enough, but you're still storing the password in plaintext in your DB. One solution is to store an md5 of the password. As long as you're using FormBuilder, this is easy to do:

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //...
	    ###END AUTOCODE
	
	    function preGenerateForm() {
	        $this->fb_preDefElements['password'] = HTML_QuickForm::createElement('password', 'password', 'Password');
	    }
	
	    function preProcessForm(&$data) {
	        if(isset($data['password'])) {
	            if($data['password'] != $this->password) {
	                $data['password'] = md5($data['password']);
	            }
	        }
	    }
	}
	?>



If you're not using FormBuilder or just want yout DataObject to handle the password for any type of call, you have to do a lot more:

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //...
	    ###END AUTOCODE
	
	    function preGenerateForm() {
	        $this->fb_preDefElements['password'] = HTML_QuickForm::createElement('password', 'password', 'Password');
	    }
	
	    function preProcessForm(&$data) {
	        if(isset($data['password'])) {
	            if($data['password'] != $this->password) {
	                $data['password'] = md5($data['password']);
	            }
	        }
	    }
	
	    //handle inserting a new record
	    function insert() {
	        if (isset($this->password)) {
	            $this->password = md5($this->password);
	        }
	        return parent::insert();
	    }
	
	    //handle an update
	    function update($do = false) {
	        //need to get old record to compare against
	        if ($do === false || !is_object($do)) {
	            $do = DB_DataObject::factory($this->__table);
	            //assuming that "id" is the primary key of this table
	            $do->get($this->id);
	        }
	        if (isset($this->password) && $this->password != $do->password) {
	            $this->password = md5($this->password);
	        }
	        return parent::update($do);
	    }
	
	    //handle a find() - this means that you can set password to the plaintext value and find() will convert it for you
	    function find($n = false) {
	        if (isset($this->password)) {
	            $this->password = md5($this->password);
	        }
	        return parent::find($n);
	    }
	}
	?>



This should take care of all cases, however, it may means that the DataObject can only be used once per record. To fix this, adding a "password is MD5'd" boolean may help, but I am not certain that it is needed.

# Virtual Fields



## Initial Concept



It's quite possible to have a virtual field which never goes into the database exactly as it is. Here's a quick example:

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //this will make this DataObject behave as if it had an extra string field
	    function table() {
	        return array_merge(parent::table(), array('virtualField' => DB_DATAOBJECT_STR));
	    }
	}
	?>



This will make all DataObject operations (including with FormBuilder) to show / use an extra string field. However, DataObject will try to use the field in the DataObject if we leave it as is. So we need to add a few more overridden methods.

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //this will make this DataObject behave as if it had an extra string field
	    function table() {
	        return array_merge(parent::table(), array('virtualField' => DB_DATAOBJECT_STR));
	    }
	
	    //stop the virtual field from being in the INSERT
	    function insert() {
	        if (isset($this->virtualField)) {
	            unset($this->virtualField);
	        }
	        parent::insert();
	    }
	
	    //stop the virtual field from being in the UPDATE
	    function update($do = false) {
	        if (isset($this->virtualField)) {
	            unset($this->virtualField);
	        }
	        if (is_object($do) && isset($do->virtualField)) {
	            unset($do->virtualField);
	        }
	        parent::update($do);
	    }
	}
	?>



Now you might ask: "What's the point? This field doesn't do anything!" True, but now we have the foundation for some interesting things.

## Saving in an alternate field / format



Say, for instance that your database has a table called "user" with a field called "name" that holds the entire name of the user. You want to do this with two form fields: "First Name" and "Last Name". Just for argument's sake, let's pretend that you can't change the columns of the table. Perhaps they're part of a pre-existing system or you're using the table for an alternative use.

So now we create virtual fields for "First Name" and "Last Name" and deal with them as above.

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //this will make this DataObject behave as if it had two extra string fields
	    function table() {
	        return array_merge(parent::table(),
	                           array('firstName' => DB_DATAOBJECT_STR,
	                                 'lastName' => DB_DATAOBJECT_STR));
	    }
	
	    //stop the virtual field from being in the INSERT
	    function insert() {
	        if (isset($this->firstName)) {
	            unset($this->firstName);
	        }
	        if (isset($this->lastName)) {
	            unset($this->lastName);
	        }
	        parent::insert();
	    }
	
	    //stop the virtual field from being in the UPDATE
	    function update($do = false) {
	        if (isset($this->firstName)) {
	            unset($this->firstName);
	        }
	        if (isset($this->lastName)) {
	            unset($this->lastName);
	        }
	        if (is_object($do)) {
	            if(isset($do->firstName)) {
	                unset($do->firstName);
	            }
	            if (isset($do->lastName)) {
	                unset($do->lastName);
	            }
	        }
	        parent::update($do);
	    }
	}
	?>



Now you have 3 name fields, two of which do nothing. That's not what we wanted. Let's make the normal name field not show up and deal with saving the two fields into the one existing one. We'll also add some nicer labels.

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //if we had any other fields, we'd need to list them here
	    var $fb_fieldsToRender = array('firstName', 'lastName');
	
	    //some nice labels
	    var $fb_fieldLabels = array('firstName' => 'First Name',
	                                'lastName' => 'Last Name');
	
	    //insert(), update(), and table() from above
	}
	?>



Looking good. Now,  we have two name fields and they save to the right place. But when we edit a record, the fields aren't shown. Now we have to override fetch().

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //add code from above
	
	    //populate the virtual fields
	    function fetch() {
	        parent::fetch();
	        //this could be made better, but this is just a simple example
	        list($this->lastName, $this->firstName) = explode(', ', $this->name);
	    }
	}
	?>



Now we have two fully functional virtual fields which actually map to one field. The same kind of thing could be done to put a bunch of fields in one (a list), or for storing, say, a bunch of checkbox values in one field.

Here's the full listing:

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //if we had any other fields, we'd need to list them here
	    var $fb_fieldsToRender = array('firstName', 'lastName');
	
	    //some nice labels
	    var $fb_fieldLabels = array('firstName' => 'First Name',
	                                'lastName' => 'Last Name');
	
	    //this will make this DataObject behave as if it had two extra string fields
	    function table() {
	        return array_merge(parent::table(),
	                           array('firstName' => DB_DATAOBJECT_STR,
	                                 'lastName' => DB_DATAOBJECT_STR));
	    }
	
	    //stop the virtual field from being in the INSERT
	    function insert() {
	        $this->name = $this->lastName.', '.$this->firstName;
	        if (isset($this->firstName)) {
	            unset($this->firstName);
	        }
	        if (isset($this->lastName)) {
	            unset($this->lastName);
	        }
	        parent::insert();
	    }
	
	    //stop the virtual field from being in the UPDATE
	    function update($do = false) {
	        $this->name = $this->lastName.', '.$this->firstName;
	        if (isset($this->firstName)) {
	            unset($this->firstName);
	        }
	        if (isset($this->lastName)) {
	            unset($this->lastName);
	        }
	        if (is_object($do)) {
	            if(isset($do->firstName)) {
	                unset($do->firstName);
	            }
	            if (isset($do->lastName)) {
	                unset($do->lastName);
	            }
	        }
	        parent::update($do);
	    }
	
	    //populate the virtual fields
	    function fetch() {
	        parent::fetch();
	        //this could be made better, but this is just a simple example
	        list($this->lastName, $this->firstName) = explode(', ', $this->name);
	    }
	}
	?>



## Virtual Links



You can also make virtual fields into links, which will make FormBuilder give you a list of options. This is a very simple addition.

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //this will make this DataObject behave as if it had an extra string field
	    function table() {
	        return array_merge(parent::table(), array('virtualField' => DB_DATAOBJECT_INT));
	    }
	
	    //make the virtual field a link
	    function links() {
	        return array_merge(parent::links(), array('virtualField' => 'linkedTable:linkedField'));
	    }
	
	    //insert and update magic
	}
	?>



## History Fields



If you want to store the history for a field's changes, you want to store the changes in another table. You can do this one of two ways. The first keeps the field in the DataObject and stores **extra** information in another table (in insert() and update()). The way I'll use here is to not store the field in the DataObject, but use a virtual field which gets the newest "history" value. This will also require overrideing fetch() and table(). I am assuming that the history table has three fields "entered", which is the date and time of INSERT, "field", which is where we're storing the field's value, and "parent_id", which links back to the main table. I am also assuming that the primary key of the main table is "id".

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //this will make this DataObject behave as if it had an extra string field
	    function table() {
	        return array_merge(parent::table(), array('historyField' => DB_DATAOBJECT_STR));
	    }
	
	    //stop the history field from being in the INSERT
	    function insert() {
	        $histObj = DB_DataObject::factory('historyTable');
	        //link back to this record
	        $histObj->parent_id = $this->id;
	        //change to your field's format
	        $histObj->enteredDate = date('Y-m-d H:i:s');
	        $histObj->field = $this->historyField;
	        $histObj->insert();
	        if (isset($this->historyField)) {
	            unset($this->historyField);
	        }
	        parent::insert();
	    }
	
	    //stop the history field from being in the UPDATE
	    function update($do = false) {
	        if ($do === false || !is_object($do)) {
	            //get a copy of this DataObject if none given to compare values
	            $do = DB_DataObject::factory($this->__table);
	            $do->get($this->id);
	        }
	        if ($this->historyField != $do->historyField) {
	            $histObj = DB_DataObject::factory('historyTable');
	            //link back to this record
	            $histObj->parent_id = $this->id;
	            //change to your field's format
	            $histObj->enteredDate = date('Y-m-d H:i:s');
	            $histObj->field = $this->historyField;
	            $histObj->insert();
	        }
	        if (isset($this->historyField)) {
	            unset($this->historyField);
	        }
	        //no need for is_object as we're checking it above
	        if (isset($do->historyField)) {
	            unset($do->historyField);
	        }
	        parent::update($do);
	    }
	
	    //fetch the newest value of the history field
	    function fetch() {
	        parent::fetch();
	        $histObj = DB_DataObject::factory('historyTable');
	        $histObj->parent_id = $this->id;
	        //this is for proper quoting of field names, not needed unless you're using reserved words for field names
	        $db = $this->getDatabaseConnection();
	        $histObj->orderBy($db->quoteIdentifier('entered').' DESC');
	        $histObj->limit(1);
	        if ($histObj->find()) {
	            $histObj->fetch();
	        }
	        $this->historyField = $histObj->field;
	    }
	}
	?>


# Dynamic Links



By dynamic links, I mean changing whether a field is a link on-the-fly based on the the value of other fields. Assume a table has two fields: "field1" and "field2". field2 is normally a simple text field, but if field1 has "link" in it, field2 becomes a link to another table.

	:::php
	<?php
	class DataObject_table extends DB_DataObject {
	    ###START AUTOCODE
	    //..
	    ###END AUTOCODE
	
	    //make the virtual field a link
	    function links() {
	        if ($this->field1 == 'link') {
	            return array_merge(parent::links(), array('field2' => 'linkedTable:linkedField'));
	        } else {
	            return parent::links();
	        }
	    }
	}
	?>



This will basically make it so that FormBuilder gives you a list of options for field2 if field1 is filled out with "link". Of course, the form has to have been submitted for this to work. This may seem useless to you, but it can easily happen in a heavily data-driven program.

# File upload handling



When you have a field in your form that allows file uploads, you must make sure that the data in the filesystem and the data in the database remain synchronized. This means, before you add a new record, you must make sure the file really made its way onto the hard drive. When you delete a record, the corresponding file must be removed from the hard drive as well. Here's how you do it:

	:::php
	<?php
	/**

	 * Table Definition for Image
	 */
	require_once 'DB/DataObject.php';
	
	class Image extends DB_DataObject  
	{
	    ###START_AUTOCODE
	    var $Image_ID;                       // int(10)  not_null primary_key
	    var $Image_Path;                     // string(120)  not_null
	    ###END_AUTOCODE
	    
	    // Prepare the form so it contains a file upload field
	    function preGenerateForm(&$fg)
	    {
	        $this->fieldLabels =  array ('Image_Path' => 'Choose image to upload');
	        $this->preDefElements['Image_Path'] = & HTML_QuickForm::createElement('file', 'Image_Path', $trl->_('File to upload'), 'maxfilesize=5000000');
	    }
	      
	    function insert() 
	    {
	        if (is_uploaded_file($_FILES['Image_Path']["tmp_name"])) {
	            // You should do some file name checking here, for example disallow spaces etc.
	            $this->Image_Path = $_FILES['Image_Path']["name"];
	            
	            // I've used a constant "UPLOADDIR" here, which must be defined somewhere of course
	            if (move_uploaded_file($_FILES['Image_Path']["tmp_name"], UPLOADDIR.$this->Image_Path)
	                && chmod(UPLOADDIR.$this->Image_Path, 0644)) {
	                    
	                // You could do some additional image manipulation, like scaling, here...
	                
	                return parent::insert();
	            }
	        }
	        return false;
	    }
	    
	    function delete()
	    {
	        $result = parent::delete();
	        
	        if ($result) {
	            unlink(UPLOADDIR . $this->Image_Path);
	        }
	        
	        return $result;
	    }
	}
	?>



# Creating automatic hierselects



This example was contributed by Arne Gellhaus

This code adds the QuickForm hierselect and checkbox elements to DB_DataObject_Formbuilder.

	:::php
	<?php
	/**

	* Table Definition for ab_persons
	*/
	require_once 'DB/DataObject.php';
	
	class DataObjects_Ab_persons extends DB_DataObject
	{
	    ###START_AUTOCODE
	    /* the code below is auto generated do not remove the above tag */
	
	    var $__table = 'ab_persons';          // table name
	    var $idPerson;                        // int(20)     not_null primary_key auto_increment
	    var $idCategory;                      // int(20)     not_null multiple_key
	    var $lastname;                        // string(50)  not_null
	    var $firstname;                       // string(50)  not_null
	    var $birthday;                        // date(10)    not_null
	    var $address;                         // string(50)  not_null
	    var $zip;                             // string(5)   not_null
	    var $idCity;                          // int(20)     not_null
	    var $idDistrict;                      // int(20)     not_null
	    var $gender;                          // string(1)   not_null enum
	    var $print;                           // int(4)
	
	    /* ZE2 compatibility trick*/
	    function __clone() { return $this;}
	
	    /* Static get */
	    function staticGet($k,$v=NULL) { return DB_DataObject::staticGet('DataObjects_Ab_persons',$k,$v); }
	
	    /* the code above is auto generated do not remove the tag below */
	    ###END_AUTOCODE
	
	    var $fb_formHeaderText = 'Leute';
	    var $fb_linkDisplayFields = array('lastname', 'firstname');
	    var $fb_createSubmit = true;
	    var $fb_linkDisplayLevel = 2;
	    var $fb_fieldsToRender = array('idPerson', 'idCategory', 'lastname', 'firstname', 'birthday', 'address', 'zip', 'idCityIdDistrict', 'gender', 'print');
	    var $fb_preDefOrder = array('idPerson', 'idCategory', 'lastname', 'firstname', 'birthday', 'address', 'zip', 'idCityIdDistrict', 'gender', 'print');
	    var $fb_selectAddEmpty = array('idDistrict');
	
	    var $fb_fieldLabels = array(
	      'idCategory' => 'Typ',
	      'lastname' => 'Nachname',
	      'firstname' => 'Vorname',
	      'birthday' => 'Geburtstag',
	      'address' => 'Adresse',
	      'zip' => 'PLZ',
	      'idCityIdDistrict' => 'Stadt',
	      'idCity' => 'Stadt',
	      'idDistrict' => 'Stadtteil',
	      'gender' => 'm/f',
	      'print' => 'Druck'
	    );
	
	    var $fb_enumFields = array('gender');
	
	    var $fb_booleanFields = array(
	      'print'
	    );
	
	    var $conditionalFields = array(
	      'idCityIdDistrict' => array(
	        'idCity:ab_md_cities'  => array('idCity'),
	        'idDistrict:ab_md_districts' => array('idCity', 'idDistrict')
	      )
	    );
	
	    function table()
	    {
	      /* hierselect virtual fields */
	      $hierselects = array();
	      foreach ($this->conditionalFields as $virtualfield => $cf) {
	        $hierselects[$virtualfield] = DB_DATAOBJECT_STR;
	      }
	      return array_merge(parent::table(), $hierselects);
	    }
	
	    function preGenerateForm(&$fb)
	    {
	      $this->fb_preDefElements = array();
	
	      /* the hierselect construct */
	      foreach ($this->conditionalFields as $virtualfield => $cf) {
	        $desc = (isset($this->fb_fieldLabels[$virtualfield])?$this->fb_fieldLabels[$virtualfield]:$virtualfield);
	        $hierselect =& HTML_QuickForm::createElement('hierselect', $fb->getFieldName($virtualfield), $desc);
	
	        $select = array();
	        $count = 0;
	        foreach ($cf as $key => $fields) {
	          list($tmp, $table) = explode(':', $key);
	          $do =& DB_DataObject::factory($table);
	          $do->find();
	          $select[$count] = array();
	          while ($do->fetch()) {
	            $curselect =& $select[$count];
	            for ($i = 0; $i < count($fields)-1; $i++) {
	              $field = $fields[$i];
	              if (!isset($curselect[$do->$field])) {
	                $curselect[$do->$field] = array();
	              }
	              $curselect =& $curselect[$do->$field];
	            }
	            $field = $fields[count($fields)-1];
	            /* check if we need an empty element */
	            if (in_array($field, $this->fb_selectAddEmpty)) {
	              if (!isset($curselect[0])) {
	                $curselect[0] = '';
	              }
	            }
	            /* at last get the displayed field and insert it */
	            $curselect[$do->$field] = $fb->getDataObjectString($do);
	          }
	          $count++;
	        }
	
	        //echo "`<pre>`";
	        //print_r($select);
	        //echo "`</pre>`";
	        $hierselect->setOptions($select);
	        /* init the hierselect with the values from the database */
	        foreach ($this->conditionalFields as $virtualfield => $cf) {
	          $a = array();
	          foreach ($cf as $key=>$fields) {
	            list($localfield, $tmp) = explode(':', $key);
	            $a[] = $this->$localfield;
	          }
	          $this->$virtualfield =& $a;
	        }
	        /* we are done! */
	        $this->fb_preDefElements[$virtualfield] = $hierselect;
	      }
	    }
	
	    function preProcessForm(&$values)
	    {
	      /* put data from hierselect in corresponding database fields */
	      foreach ($this->conditionalFields as $virtualfield => $cf) {
	        $count = 0;
	        foreach ($cf as $key => $fields) {
	          list($localfield, $tmp) = explode(':', $key);
	          $this->$localfield = $values[$virtualfield][$count];
	          $count++;
	        }
	      }
	    }
	
	    function dateOptions($fieldName)
	    {
	      return array(
	       'minYear' => 1900
	      );
	    }
	
	}
	
	?>



The tables used with this example are

	
	CREATE TABLE `ab_persons` ( 
	  `idPerson` bigint(20) NOT NULL auto_increment, 
	  `idCategory` bigint(20) NOT NULL default '0', 
	  `lastname` varchar(50) NOT NULL default '', 
	  `firstname` varchar(50) NOT NULL default '', 
	  `birthday` date NOT NULL default '0000-00-00', 
	  `address` varchar(50) NOT NULL default '', 
	  `zip` varchar(5) NOT NULL default '', 
	  `idCity` bigint(20) NOT NULL default '0', 
	  `idDistrict` bigint(20) NOT NULL default '0', 
	  `gender` enum('m','f') NOT NULL default 'm', 
	  `print` tinyint(1) default '0', 
	  PRIMARY KEY  (`idPerson`), 
	  KEY `idCategory` (`idCategory`,`lastname`,`firstname`,`birthday`) 
	) TYPE=MyISAM; 
	 
	CREATE TABLE `ab_md_cities` ( 
	  `idCity` bigint(20) NOT NULL auto_increment, 
	  `name` varchar(30) NOT NULL default '', 
	  PRIMARY KEY  (`idCity`), 
	  UNIQUE KEY `Name` (`name`) 
	) TYPE=MyISAM; 
	 
	CREATE TABLE `ab_md_districts` ( 
	  `idDistrict` bigint(20) NOT NULL auto_increment, 
	  `idCity` bigint(20) NOT NULL default '0', 
	  `name` varchar(30) NOT NULL default '', 
	  PRIMARY KEY  (`idDistrict`), 
	  UNIQUE KEY `name` (`name`), 
	  KEY `idCity` (`idCity`) 
	) TYPE=MyISAM; 



and the databasename.links.ini looks like that

	
	[ab_persons] 
	idCity = ab_md_cities:idCity 
	idDistrict = ab_md_districts:idDistrict 
	 
	[ab_md_districts] 
	idCity = ab_md_cities:idCity 



# A Full Frontend



Justin Patrin has implemented a frontend that builds table browsing and record deletion onto FormBuilder. It's pretty simple, but very useful and is a decent example of how to build an app around FormBuilder.

[DB_DataObject_FormBuilder_Frontend](http://pear.reversefold.com/dokuwiki/doku.php?id=pear:db_dataobject_formbuilder_frontend)

# More ideas




*  Delete referenced data when record itself is being deleted (poor man's ON DELETE CASCADE)
