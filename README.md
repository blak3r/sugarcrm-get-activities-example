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
