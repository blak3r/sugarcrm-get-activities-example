sugarcrm-get-activities-example
===============================

SugarCRM Get Activities Example

PHP Example of how to emails, tasks, notes, meetings, and calls for a particular record id.


Dependencies: 
* http://github.com/asakusuma/SugarCRM-REST-API-Wrapper-Class/


### Method

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

#### Improvements:
1. Would be good to limit the number of activities returned.  Didn't look into this.
2. Would be nice if activities were sorted by date.


### Usage

```php
print_r( $sugar->getActivities("0033000000Xqd66AAB","Contacts") );
print_r( $sugar->getActivities("8df3472d-23b4-e3df-c4ee-52aa3e63ef72", "Leads") );```
```

### Sample Output
```php
(
    [0] => Array
        (
            [type] => tasks
            [id] => c66b2bc5-8242-215a-54c2-52aa5a6a4196
            [name] => test
            [description] => test
            [date_modified] => 2013-12-13 00:54:47
        )

    [1] => Array
        (
            [type] => notes
            [id] => 226eb4ff-6bcc-cb62-975f-52aa5aadf050
            [name] => Sample Note
            [description] => Blah blah blah blah 
            [date_modified] => 2013-12-13 00:55:22
        )

)
```



## Raw Request Stuff (for implementing in other languages) 
----------------------------------------------------------

### JSON Payload

For reference purposes, this is what the data posted looks like for the REST Call.  You could modify this
```
method=get_entry_list&input_type=JSON&response_type=JSON&rest_data={"session":"jkfpgd76o3aoe1hdg73cl15dl6","module_name":"Contacts","query":"contacts.id='0033000000Xqd66AAB'","order_by":null,"offset":0,"select_fields":["id","name"],"link_name_to_fields_array":[{"name":"calls","value":["id","name","description","date_modified"]},{"name":"emails","value":["id","name","description","date_modified"]},{"name":"tasks","value":["id","name","description","date_modified"]},{"name":"notes","value":["id","name","description","date_modified"]}],"max_results":20,"deleted":"FALSE"}URL: http://your.sugarbox.com/sugarcrm/service/v4_1/rest.php
```

### CURL
```php
        $ch = curl_init();

        //print "REST REQUEST: " . $call_name . "\n". print_r($call_arguments, true);

        $post_data = 'method='.$call_name.'&input_type=JSON&response_type=JSON';
        $jsonEncodedData = json_encode($call_arguments);
        $post_data = $post_data . "&rest_data=" . $jsonEncodedData;

        print "URL: $this->rest_url\n";
        print ("POST DATA:");
        print_r($post_data);

        curl_setopt($ch, CURLOPT_URL, $this->rest_url);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $post_data);
        $output = curl_exec($ch);

        //print "RESPONSE: " . $output;

        $response_data = json_decode($output,true);

        return $response_data;
```

Where ```$method = "get_entry_list"```;
and ```print_r($call_arguments)``` =
```
Array
(
    [session] => dc2iqklvnttvefbuvbs105bfe3
    [module_name] => Contacts
    [query] => contacts.id='0033000000Xqd66AAB'
    [order_by] => 
    [offset] => 0
    [select_fields] => Array
        (
            [0] => id
            [1] => name
        )

    [link_name_to_fields_array] => Array
        (
            [0] => Array
                (
                    [name] => calls
                    [value] => Array
                        (
                            [0] => id
                            [1] => name
                            [2] => description
                            [3] => date_modified
                        )

                )

            [1] => Array
                (
                    [name] => emails
                    [value] => Array
                        (
                            [0] => id
                            [1] => name
                            [2] => description
                            [3] => date_modified
                        )

                )

            [2] => Array
                (
                    [name] => tasks
                    [value] => Array
                        (
                            [0] => id
                            [1] => name
                            [2] => description
                            [3] => date_modified
                        )

                )

            [3] => Array
                (
                    [name] => notes
                    [value] => Array
                        (
                            [0] => id
                            [1] => name
                            [2] => description
                            [3] => date_modified
                        )

                )

        )

    [max_results] => 20
    [deleted] => FALSE
)
```
