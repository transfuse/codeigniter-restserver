# CodeIgniter Rest Server

A fully RESTful server implementation for CodeIgniter 2 and 3 using one library, one config file and one controller.

Originally from: (https://github.com/Hanisch-IT/codeigniter-restserver).

## Requirements

1. PHP 5.4 or greater
2. CodeIgniter 2.0+

## Installation & loading

Rest Server is available via [Composer/Packagist](https://packagist.org/packages/transfuse/codeigniter-restserver) (using semantic versioning), so just add this line to your `composer.json` file:

```json
"transfuse/codeigniter-restserver": "^1.0"
```

or

```sh
composer require transfuse/codeigniter-restserver
```

## Handling Requests

When your controller extends from `REST_Controller`, the method names will be appended with the HTTP method used to access the request. If you're  making an HTTP `GET` call to `/books`, for instance, it would call a `Books#index_get()` method.

This allows you to implement a RESTful interface easily:

```php
use Restserver\Libraries\REST_Controller;

class Books extends REST_Controller
{
  public function index_get()
  {
    // Display all books
  }

  public function index_post()
  {
    // Create a new book
  }
}
```

`REST_Controller` also supports `PUT` and `DELETE` methods, allowing you to support a truly RESTful interface.


Accessing parameters is also easy. Simply use the name of the HTTP verb as a method:

```php
$this->get('blah'); // GET param
$this->post('blah'); // POST param
$this->put('blah'); // PUT param
```

The HTTP spec for DELETE requests precludes the use of parameters.  For delete requests, you can add items to the URL

```php
public function index_delete($id)
{
	$this->response([
		'returned from delete:' => $id,
	]);
}
```

If query parameters are passed via the URL, regardless of whether it's a GET request, can be obtained by the query method:

```php
$this->query('blah'); // Query param
```

## Content Types

`REST_Controller` supports a bunch of different request/response formats, including XML, JSON and serialised PHP. By default, the class will check the URL and look for a format either as an extension or as a separate segment.

This means your URLs can look like this:
```
http://example.com/books.json
http://example.com/books?format=json
```

This can be flaky with URI segments, so the recommend approach is using the HTTP `Accept` header:

```bash
$ curl -H "Accept: application/json" http://example.com
```

Any responses you make from the class (see [responses](#responses) for more on this) will be serialised in the designated format.

## Responses

The class provides a `response()` method that allows you to return data in the user's requested response format.

Returning any object / array / string / whatever is easy:

```php
public function index_get()
{
  $this->response($this->db->get('books')->result());
}
```

This will automatically return an `HTTP 200 OK` response. You can specify the status code in the second parameter:

```php
public function index_post()
  {
    // ...create new book
    $this->response($book, 201); // Send an HTTP 201 Created
  }
```

If you don't specify a response code, and the data you respond with `== FALSE` (an empty array or string, for instance), the response code will automatically be set to `404 Not Found`:

```php
$this->response([]); // HTTP 404 Not Found
```

## Configuration

To override the default configuration you have to create a "rest.php" configuration file in your application/config path.

```php
/*
|--------------------------------------------------------------------------
| HTTP protocol
|--------------------------------------------------------------------------
|
| Set to force the use of HTTPS for REST API calls
|
*/
$config['force_https'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST Output Format
|--------------------------------------------------------------------------
|
| The default format of the response
|
| 'array':      Array data structure
| 'csv':        Comma separated file
| 'json':       Uses json_encode(). Note: If a GET query string
|               called 'callback' is passed, then jsonp will be returned
| 'html'        HTML using the table library in CodeIgniter
| 'php':        Uses var_export()
| 'serialized':  Uses serialize()
| 'xml':        Uses simplexml_load_string()
|
*/
$config['rest_default_format'] = 'json';

/*
|--------------------------------------------------------------------------
| REST Supported Output Formats
|--------------------------------------------------------------------------
|
| The following setting contains a list of the supported/allowed formats.
| You may remove those formats that you don't want to use.
| If the default format $config['rest_default_format'] is missing within
| $config['rest_supported_formats'], it will be added silently during
| REST_Controller initialization.
|
*/
$config['rest_supported_formats'] = [
    'json',
    'array',
    'csv',
    'html',
    'jsonp',
    'php',
    'serialized',
    'xml',
];

/*
|--------------------------------------------------------------------------
| REST Status Field Name
|--------------------------------------------------------------------------
|
| The field name for the status inside the response
|
*/
$config['rest_status_field_name'] = 'status';

/*
|--------------------------------------------------------------------------
| REST Message Field Name
|--------------------------------------------------------------------------
|
| The field name for the message inside the response
|
*/
$config['rest_message_field_name'] = 'error';

/*
|--------------------------------------------------------------------------
| Enable Emulate Request
|--------------------------------------------------------------------------
|
| Should we enable emulation of the request (e.g. used in Mootools request)
|
*/
$config['enable_emulate_request'] = TRUE;

/*
|--------------------------------------------------------------------------
| REST Realm
|--------------------------------------------------------------------------
|
| Name of the password protected REST API displayed on login dialogs
|
| e.g: My Secret REST API
|
*/
$config['rest_realm'] = 'REST API';

/*
|--------------------------------------------------------------------------
| REST Login
|--------------------------------------------------------------------------
|
| Set to specify the REST API requires to be logged in
|
| FALSE     No login required
| 'basic'   Unsecure login
| 'digest'  More secure login
| 'session' Check for a PHP session variable. See 'auth_source' to set the
|           authorization key
|
*/
$config['rest_auth'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST Login Source
|--------------------------------------------------------------------------
|
| Is login required and if so, the user store to use
|
| ''        Use config based users or wildcard testing
| 'ldap'    Use LDAP authentication
| 'library' Use a authentication library
|
| Note: If 'rest_auth' is set to 'session' then change 'auth_source' to the name of the session variable
|
*/
$config['auth_source'] = 'ldap';

/*
|--------------------------------------------------------------------------
| Allow Authentication and API Keys
|--------------------------------------------------------------------------
|
| Where you wish to have Basic, Digest or Session login, but also want to use API Keys (for limiting
| requests etc), set to TRUE;
|
*/
$config['allow_auth_and_keys'] = TRUE;

/*
|--------------------------------------------------------------------------
| REST Login Class and Function
|--------------------------------------------------------------------------
|
| If library authentication is used define the class and function name
|
| The function should accept two parameters: class->function($username, $password)
| In other cases override the function _perform_library_auth in your controller
|
| For digest authentication the library function should return already a stored
| md5(username:restrealm:password) for that username
|
| e.g: md5('admin:REST API:1234') = '1e957ebc35631ab22d5bd6526bd14ea2'
|
*/
$config['auth_library_class'] = '';
$config['auth_library_function'] = '';

/*
|--------------------------------------------------------------------------
| Override auth types for specific class/method
|--------------------------------------------------------------------------
|
| Set specific authentication types for methods within a class (controller)
|
| Set as many config entries as needed.  Any methods not set will use the default 'rest_auth' config value.
|
| e.g:
|
|           $config['auth_override_class_method']['deals']['view'] = 'none';
|           $config['auth_override_class_method']['deals']['insert'] = 'digest';
|           $config['auth_override_class_method']['accounts']['user'] = 'basic';
|           $config['auth_override_class_method']['dashboard']['*'] = 'none|digest|basic';
|
| Here 'deals', 'accounts' and 'dashboard' are controller names, 'view', 'insert' and 'user' are methods within. An asterisk may also be used to specify an authentication method for an entire classes methods. Ex: $config['auth_override_class_method']['dashboard']['*'] = 'basic'; (NOTE: leave off the '_get' or '_post' from the end of the method name)
| Acceptable values are; 'none', 'digest' and 'basic'.
|
*/
// $config['auth_override_class_method']['deals']['view'] = 'none';
// $config['auth_override_class_method']['deals']['insert'] = 'digest';
// $config['auth_override_class_method']['accounts']['user'] = 'basic';
// $config['auth_override_class_method']['dashboard']['*'] = 'basic';


// ---Uncomment list line for the wildard unit test
// $config['auth_override_class_method']['wildcard_test_cases']['*'] = 'basic';

/*
|--------------------------------------------------------------------------
| Override auth types for specfic 'class/method/HTTP method'
|--------------------------------------------------------------------------
|
| example:
|
|            $config['auth_override_class_method_http']['deals']['view']['get'] = 'none';
|            $config['auth_override_class_method_http']['deals']['insert']['post'] = 'none';
|            $config['auth_override_class_method_http']['deals']['*']['options'] = 'none';
*/

// ---Uncomment list line for the wildard unit test
// $config['auth_override_class_method_http']['wildcard_test_cases']['*']['options'] = 'basic';

/*
|--------------------------------------------------------------------------
| REST Login Usernames
|--------------------------------------------------------------------------
|
| Array of usernames and passwords for login, if ldap is configured this is ignored
|
*/
$config['rest_valid_logins'] = ['admin' => '1234'];

/*
|--------------------------------------------------------------------------
| Global IP Whitelisting
|--------------------------------------------------------------------------
|
| Limit connections to your REST server to whitelisted IP addresses
|
| Usage:
| 1. Set to TRUE and select an auth option for extreme security (client's IP
|    address must be in whitelist and they must also log in)
| 2. Set to TRUE with auth set to FALSE to allow whitelisted IPs access with no login
| 3. Set to FALSE but set 'auth_override_class_method' to 'whitelist' to
|    restrict certain methods to IPs in your whitelist
|
*/
$config['rest_ip_whitelist_enabled'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST IP Whitelist
|--------------------------------------------------------------------------
|
| Limit connections to your REST server with a comma separated
| list of IP addresses
|
| e.g: '123.456.789.0, 987.654.32.1'
|
| 127.0.0.1 and 0.0.0.0 are allowed by default
|
*/
$config['rest_ip_whitelist'] = '';

/*
|--------------------------------------------------------------------------
| Global IP Blacklisting
|--------------------------------------------------------------------------
|
| Prevent connections to the REST server from blacklisted IP addresses
|
| Usage:
| 1. Set to TRUE and add any IP address to 'rest_ip_blacklist'
|
*/
$config['rest_ip_blacklist_enabled'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST IP Blacklist
|--------------------------------------------------------------------------
|
| Prevent connections from the following IP addresses
|
| e.g: '123.456.789.0, 987.654.32.1'
|
*/
$config['rest_ip_blacklist'] = '';

/*
|--------------------------------------------------------------------------
| REST Database Group
|--------------------------------------------------------------------------
|
| Connect to a database group for keys, logging, etc. It will only connect
| if you have any of these features enabled
|
*/
$config['rest_database_group'] = 'default';

/*
|--------------------------------------------------------------------------
| REST API Keys Table Name
|--------------------------------------------------------------------------
|
| The table name in your database that stores API keys
|
*/
$config['rest_keys_table'] = 'keys';

/*
|--------------------------------------------------------------------------
| REST Enable Keys
|--------------------------------------------------------------------------
|
| When set to TRUE, the REST API will look for a column name called 'key'.
| If no key is provided, the request will result in an error. To override the
| column name see 'rest_key_column'
|
| Default table schema:
|   CREATE TABLE `keys` (
|       `id` INT(11) NOT NULL AUTO_INCREMENT,
|       `user_id` INT(11) NOT NULL,
|       `key` VARCHAR(40) NOT NULL,
|       `level` INT(2) NOT NULL,
|       `ignore_limits` TINYINT(1) NOT NULL DEFAULT '0',
|       `is_private_key` TINYINT(1)  NOT NULL DEFAULT '0',
|       `ip_addresses` TEXT NULL DEFAULT NULL,
|       `date_created` INT(11) NOT NULL,
|       PRIMARY KEY (`id`)
|   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
|
*/
$config['rest_enable_keys'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST Table Key Column Name
|--------------------------------------------------------------------------
|
| If not using the default table schema in 'rest_enable_keys', specify the
| column name to match e.g. my_key
|
*/
$config['rest_key_column'] = 'key';

/*
|--------------------------------------------------------------------------
| REST API Limits method
|--------------------------------------------------------------------------
|
| Specify the method used to limit the API calls
|
| Available methods are :
| $config['rest_limits_method'] = 'API_KEY'; // Put a limit per api key
| $config['rest_limits_method'] = 'METHOD_NAME'; // Put a limit on method calls
| $config['rest_limits_method'] = 'ROUTED_URL';  // Put a limit on the routed URL
|
*/
$config['rest_limits_method'] = 'ROUTED_URL';

/*
|--------------------------------------------------------------------------
| REST Key Length
|--------------------------------------------------------------------------
|
| Length of the created keys. Check your default database schema on the
| maximum length allowed
|
| Note: The maximum length is 40
|
*/
$config['rest_key_length'] = 40;

/*
|--------------------------------------------------------------------------
| REST API Key Variable
|--------------------------------------------------------------------------
|
| Custom header to specify the API key

| Note: Custom headers with the X- prefix are deprecated as of
| 2012/06/12. See RFC 6648 specification for more details
|
*/
$config['rest_key_name'] = 'X-API-KEY';

/*
|--------------------------------------------------------------------------
| REST Enable Logging
|--------------------------------------------------------------------------
|
| When set to TRUE, the REST API will log actions based on the column names 'key', 'date',
| 'time' and 'ip_address'. This is a general rule that can be overridden in the
| $this->method array for each controller
|
| Default table schema:
|   CREATE TABLE `logs` (
|       `id` INT(11) NOT NULL AUTO_INCREMENT,
|       `uri` VARCHAR(255) NOT NULL,
|       `method` VARCHAR(6) NOT NULL,
|       `params` TEXT DEFAULT NULL,
|       `api_key` VARCHAR(40) NOT NULL,
|       `ip_address` VARCHAR(45) NOT NULL,
|       `time` INT(11) NOT NULL,
|       `rtime` FLOAT DEFAULT NULL,
|       `authorized` VARCHAR(1) NOT NULL,
|       `response_code` smallint(3) DEFAULT '0',
|       PRIMARY KEY (`id`)
|   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
|
*/
$config['rest_enable_logging'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST API Logs Table Name
|--------------------------------------------------------------------------
|
| If not using the default table schema in 'rest_enable_logging', specify the
| table name to match e.g. my_logs
|
*/
$config['rest_logs_table'] = 'logs';

/*
|--------------------------------------------------------------------------
| REST Method Access Control
|--------------------------------------------------------------------------
| When set to TRUE, the REST API will check the access table to see if
| the API key can access that controller. 'rest_enable_keys' must be enabled
| to use this
|
| Default table schema:
|   CREATE TABLE `access` (
|       `id` INT(11) unsigned NOT NULL AUTO_INCREMENT,
|       `key` VARCHAR(40) NOT NULL DEFAULT '',
|       `controller` VARCHAR(50) NOT NULL DEFAULT '',
|       `date_created` DATETIME DEFAULT NULL,
|       `date_modified` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
|       PRIMARY KEY (`id`)
|    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
|
*/
$config['rest_enable_access'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST API Access Table Name
|--------------------------------------------------------------------------
|
| If not using the default table schema in 'rest_enable_access', specify the
| table name to match e.g. my_access
|
*/
$config['rest_access_table'] = 'access';

/*
|--------------------------------------------------------------------------
| REST API Param Log Format
|--------------------------------------------------------------------------
|
| When set to TRUE, the REST API log parameters will be stored in the database as JSON
| Set to FALSE to log as serialized PHP
|
*/
$config['rest_logs_json_params'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST Enable Limits
|--------------------------------------------------------------------------
|
| When set to TRUE, the REST API will count the number of uses of each method
| by an API key each hour. This is a general rule that can be overridden in the
| $this->method array in each controller
|
| Default table schema:
|   CREATE TABLE `limits` (
|       `id` INT(11) NOT NULL AUTO_INCREMENT,
|       `uri` VARCHAR(255) NOT NULL,
|       `count` INT(10) NOT NULL,
|       `hour_started` INT(11) NOT NULL,
|       `api_key` VARCHAR(40) NOT NULL,
|       PRIMARY KEY (`id`)
|   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
|
| To specify the limits within the controller's __construct() method, add per-method
| limits with:
|
|       $this->method['METHOD_NAME']['limit'] = [NUM_REQUESTS_PER_HOUR];
|
| See application/controllers/api/example.php for examples
*/
$config['rest_enable_limits'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST API Limits Table Name
|--------------------------------------------------------------------------
|
| If not using the default table schema in 'rest_enable_limits', specify the
| table name to match e.g. my_limits
|
*/
$config['rest_limits_table'] = 'limits';

/*
|--------------------------------------------------------------------------
| REST Ignore HTTP Accept
|--------------------------------------------------------------------------
|
| Set to TRUE to ignore the HTTP Accept and speed up each request a little.
| Only do this if you are using the $this->rest_format or /format/xml in URLs
|
*/
$config['rest_ignore_http_accept'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST AJAX Only
|--------------------------------------------------------------------------
|
| Set to TRUE to allow AJAX requests only. Set to FALSE to accept HTTP requests
|
| Note: If set to TRUE and the request is not AJAX, a 505 response with the
| error message 'Only AJAX requests are accepted.' will be returned.
|
| Hint: This is good for production environments
|
*/
$config['rest_ajax_only'] = FALSE;

/*
|--------------------------------------------------------------------------
| REST Language File
|--------------------------------------------------------------------------
|
| Language file to load from the language directory
|
*/
$config['rest_language'] = 'english';

```

## Authentication

This class also provides rudimentary support for HTTP basic authentication and/or the securer HTTP digest access authentication.

You can enable basic authentication by setting the `$config['rest_auth']` to `'basic'`. The `$config['rest_valid_logins']` directive can then be used to set the usernames and passwords able to log in to your system. The class will automatically send all the correct headers to trigger the authentication dialogue:

```php
$config['rest_valid_logins'] = ['username' => 'password', 'other_person' => 'secure123'];
```

Enabling digest auth is similarly easy. Configure your desired logins in the config file like above, and set `$config['rest_auth']` to `'digest'`. The class will automatically send out the headers to enable digest auth.

If you're tying this library into an AJAX endpoint where clients authenticate using PHP sessions then you may not like either of the digest nor basic authentication methods. In that case, you can tell the REST Library what PHP session variable to check for. If the variable exists, then the user is authorized. It will be up to your application to set that variable. You can define the variable in ``$config['auth_source']``.  Then tell the library to use a php session variable by setting ``$config['rest_auth']`` to ``session``.

All three methods of authentication can be secured further by using an IP whitelist. If you enable `$config['rest_ip_whitelist_enabled']` in your config file, you can then set a list of allowed IPs.

Any client connecting to your API will be checked against the whitelisted IP array. If they're on the list, they'll be allowed access. If not, sorry, no can do hombre. The whitelist is a comma-separated string:

```php
$config['rest_ip_whitelist'] = '123.456.789.0, 987.654.32.1';
```

Your localhost IPs (`127.0.0.1` and `0.0.0.0`) are allowed by default.

## API Keys

In addition to the authentication methods above, the `REST_Controller` class also supports the use of API keys. Enabling API keys is easy. Turn it on in your **config/rest.php** file:

```php
$config['rest_enable_keys'] = TRUE;
```

You'll need to create a new database table to store and access the keys. `REST_Controller` will automatically assume you have a table that looks like this:

```sql
CREATE TABLE `keys` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`key` VARCHAR(40) NOT NULL,
	`level` INT(2) NOT NULL,
	`ignore_limits` TINYINT(1) NOT NULL DEFAULT '0',
	`date_created` INT(11) NOT NULL,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

The class will look for an HTTP header with the API key on each request. An invalid or missing API key will result in an `HTTP 403 Forbidden`.

By default, the HTTP will be `X-API-KEY`. This can be configured in **config/rest.php**.

```bash
$ curl -X POST -H "X-API-KEY: some_key_here" http://example.com/books
```
## Language settings
The default language is english. You can change the language in your rest configuration file (see above).
To create or override the default language you have to create "rest_controller_lang.php" in you application folder (german/rest_controller_lang.php)

```php
$lang['text_rest_invalid_api_key'] = 'Invalid API key %s'; // %s is the REST API key
$lang['text_rest_invalid_credentials'] = 'Invalid credentials';
$lang['text_rest_ip_denied'] = 'IP denied';
$lang['text_rest_ip_unauthorized'] = 'IP unauthorized';
$lang['text_rest_unauthorized'] = 'Unauthorized';
$lang['text_rest_ajax_only'] = 'Only AJAX requests are allowed';
$lang['text_rest_api_key_unauthorized'] = 'This API key does not have access to the requested controller';
$lang['text_rest_api_key_permissions'] = 'This API key does not have enough permissions';
$lang['text_rest_api_key_time_limit'] = 'This API key has reached the time limit for this method';
$lang['text_rest_unknown_method'] = 'Unknown method';
$lang['text_rest_unsupported'] = 'Unsupported protocol';

```

## Other Documentation / Tutorials

* [NetTuts: Working with RESTful Services in CodeIgniter](http://net.tutsplus.com/tutorials/php/working-with-restful-services-in-codeigniter-2/)

## Contributions

This project was originally written by Phil Sturgeon and Chris Kacerguis.

See: https://github.com/chriskacerguis/codeigniter-restserver

[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://raw.githubusercontent.com/chriskacerguis/codeigniter-restserver/master/LICENSE)