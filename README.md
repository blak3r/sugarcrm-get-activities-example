sugarcrm-get-activities-example
===============================

SugarCRM Get Activities Example

PHP Example of how to emails, tasks, notes, meetings, and calls for a particular record id.


Dependencies: 
* http://github.com/asakusuma/SugarCRM-REST-API-Wrapper-Class/




```php
print_r( $sugar->getActivities("0033000000Xqd66AAB","Contacts") );
print_r( $sugar->getActivities("8df3472d-23b4-e3df-c4ee-52aa3e63ef72", "Leads") );```
```



```php
    function getActivities( $id, $type="Contacts" ) {
    
        $sugar = new Sugar_REST( SUGAR_REST_URL, SUGAR_USER_NAME, SUGAR_PASSWORD); // <--- SET THESE!
        $moduleTypeUc = ucfirst($type);
        $moduleTypeLc = lcfirst($type);

        $selectFieldsArray = array( "calls" => array("id", "name", "description", "date_modified")
                                  , "emails" => array("id", "name", "description", "date_modified")
                                  , "tasks" => array("id", "name", "description", "date_modified")
                                  , "notes" => array("id", "name", "description", "date_modified")
                                  );

        $fields = array( $moduleTypeUc => array("id", "name")
                       , "Calls" =>  $selectFieldsArray['calls']
                       , "Emails" => $selectFieldsArray['emails']
                       , "Tasks" => $selectFieldsArray['tasks']
                       , "Notes" => $selectFieldsArray['notes']
                       );

        $options = array(
            'where' => "$moduleTypeLc.id='$id'"
        );
        $apiResult = $sugar->get_with_related($moduleTypeUc, $fields, $options );


        $activities = array();
        foreach($apiResult['relationship_list'][0]['link_list'] as $currActivityModuleTypeRecords) {

            foreach( $currActivityModuleTypeRecords['records'] as $almostActivity) {

                $curr = array();
                $currActivityType = $currActivityModuleTypeRecords['name']; // "calls" or "emails"...
                if( isset( $selectFieldsArray[$currActivityType] ) ) {

                    $curr['type'] = $currActivityType;
                    foreach($selectFieldsArray[$currActivityType] as $key) {
                        $curr[$key] = $almostActivity['link_value'][$key]['value'];
                    }
                    $activities[] = $curr;  // TODO: we should be inserting them into an array sorted by date.
                                            // TODO: remove any duplicates based on id.  On my system, emails get returned in both Tasks and Emails.
                }
                else {
                    print "Didn't find any: " . $currActivityType;
                }
            }
        }

        return $activities;
    }
```
