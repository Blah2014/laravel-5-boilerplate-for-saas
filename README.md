laravel-5-boilerplate-for-saas
===================

<a href="https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=C2HFZWSUPV47Q" target="_blank">
  <img src="https://raw.githubusercontent.com/Blah2014/phonegap-inmobi-plugin/gh-pages/images/BuymeaCoffee.png" border="0" name="submit" alt="PayPal - The safer, easier way to pay online!" />
</a>

<a href="http://traderhub.info" target="_blank">
  <img src="http://traderhub.info/images/AD.jpg" border="0" name="submit" alt="TraderHub - daily stock picks, market signals, stock picking service" />
</a>

### Note:
If you found this project helpful please give this repo a star.

### Features
* Authentication without forms (support SAAS)
* ACL (Access Control List) manager

### Project Structure
```
domain/
    index.html
    css/
    images/
    js/
    application/
        index.php
        framework/
```      
'application/' will hold Laravel 5 project file

### Installation
Simply copy the files across into the appropriate directories, and register the middleware in App\Http\Kernel.php
Then specify a 'roles' middleware on the route you'd like to protect, and specify the individual roles as an array:

##### Example:
```php
Route::get('user/{user}', [
     'middleware' => ['auth', 'roles'],
     'uses' => 'UserController@index',
     'roles' => ['Administrator', 'Manager']
]);
```

##### Note:
Because we don't use public folder anymore, you need to make this changes:
* /application/framework/vendor/laravel/framework/src/Illuminate/Foundation/Console/ServeCommand.php
  line 36 change **chdir($this->laravel->publicPath());** to **chdir('/');**
  
* /application/framework/server.php line 17 and 21 **/public** to **/..**

This will help you to run: **php artisan serve**