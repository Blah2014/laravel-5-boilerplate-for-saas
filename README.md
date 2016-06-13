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
        index.php  (public folder content is here...)
        framework/ (all other files of laravel is here...)
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
* /application/index.php line 22 and 36
  change **/../bootstrap/autoload.php** to **/framework/bootstrap/autoload.php**

* /application/framework/vendor/laravel/framework/src/Illuminate/Foundation/Console/ServeCommand.php
  line 36 change **chdir($this->laravel->publicPath());** to **chdir('/');**
  
* /application/framework/server.php line 17 and 21 **/public** to **/..**

This will help you to run: **php artisan serve**

### Authentication
**Do not run:** php artisan make:auth (for SAAS we are not going to use server bindings)

#### Kernel.php
/application/framework/app/Http/Kernel.php

###### Comment: ######
line: 31 - \App\Http\Middleware\VerifyCsrfToken::class

#### routes.php
/application/framework/app/Http/routes.php

###### Add: ######
Route::auth();

#### AuthController.php
/application/framework/app/Http/Controllers/Auth/AuthController.php

###### Add/Edit: ######
* use Illuminate\Support\Facades\Auth;
* use Illuminate\Http\Request;

```php
public function __construct()
{
    $this->middleware($this->guestMiddleware(), ['except' => ['logout', 'register', 'login']]);
}
```

```php
protected function validator(array $data)
{
    return Validator::make($data, [
        'first_name' => 'required|max:255',
        'last_name' => 'required|max:255',
        'email' => 'required|email|max:255|unique:users|confirmed',
        'password' => 'required|min:6|confirmed',
    ]);
}
```

```php
protected function create(array $data)
{   
    return User::create([
        'first_name' => $data['first_name'],
        'last_name' => $data['last_name'],
        'email' => $data['email'],
        'password' => bcrypt($data['password']),
        'ip_address' => $data['ip_address'],
    ]);
}
```

```php
public function register(Request $request)
{
    $validator = $this->validator($request->all());

    // Validate
    if ($validator->fails()) {
        return response()
        ->json($validator->errors());
    }
    
    // Add ip address
    $requestArray = $request->all();
    $requestArray['ip_address'] = $request->ip();

    // Create user then login
    Auth::guard($this->getGuard())->login($this->create($requestArray));

    return response()
        ->json(Auth::user());
}

public function login(Request $request)
{
    if(Auth::check()){
        $this->logout();
    }
    
    // If the class is using the ThrottlesLogins trait, we can automatically throttle
    // the login attempts for this application. We'll key this by the username and
    // the IP address of the client making these requests into this application.
    $throttles = $this->isUsingThrottlesLoginsTrait();

    if ($throttles && $lockedOut = $this->hasTooManyLoginAttempts($request)) {
        $this->fireLockoutEvent($request);

        $seconds = $this->secondsRemainingOnLockout($request);
    
        return response()
            ->json($this->getLockoutErrorMessage($seconds));
    }
    
    
    if (Auth::attempt(['email' => $request->input('email'), 'password' => $request->input('password')])) {
        // Authentication passed...

        return response()
          ->json(Auth::user());
    } else {
        
        // If the login attempt was unsuccessful we will increment the number of attempts
        // to login and redirect the user back to the login form. Of course, when this
        // user surpasses their maximum number of attempts they will get locked out.
        if ($throttles && ! $lockedOut) {
            $this->incrementLoginAttempts($request);
        }
        
        return response()
            ->json($this->getFailedLoginMessage());
    }
}

public function logout()
{
    Auth::logout();
}
```

#### Full modified file AuthController.php
/application/framework/app/Http/Controllers/Auth/AuthController.php

```php
<?php

namespace App\Http\Controllers\Auth;

use App\User;
use Validator;
use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\ThrottlesLogins;
use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Http\Request;

class AuthController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Registration & Login Controller
    |--------------------------------------------------------------------------
    |
    | This controller handles the registration of new users, as well as the
    | authentication of existing users. By default, this controller uses
    | a simple trait to add these behaviors. Why don't you explore it?
    |
    */

    use AuthenticatesAndRegistersUsers, ThrottlesLogins;

    /**
     * Where to redirect users after login / registration.
     *
     * @var string
     */
    protected $redirectTo = '/';

    /**
     * Create a new authentication controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware($this->guestMiddleware(), ['except' => 
            ['logout', 'register', 'login']]);
    }

    /**
     * Get a validator for an incoming registration request.
     *
     * @param  array  $data
     * @return \Illuminate\Contracts\Validation\Validator
     */
    protected function validator(array $data)
    {
        return Validator::make($data, [
            'first_name' => 'required|max:255',
            'last_name' => 'required|max:255',
            'email' => 'required|email|max:255|unique:users|confirmed',
            'password' => 'required|min:6|confirmed',
        ]);
    }

    /**
     * Create a new user instance after a valid registration.
     *
     * @param  array  $data
     * @return User
     */
    protected function create(array $data)
    {   
        return User::create([
            'first_name' => $data['first_name'],
            'last_name' => $data['last_name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
            'ip_address' => $data['ip_address'],
        ]);
    }
    
    public function register(Request $request)
    {
        $validator = $this->validator($request->all());

        // Validate
        if ($validator->fails()) {
            return response()
            ->json($validator->errors());
        }
        
        // Add ip address
        $requestArray = $request->all();
        $requestArray['ip_address'] = $request->ip();

        // Create user then login
        Auth::guard($this->getGuard())->login($this->create($requestArray));

        return response()
            ->json(Auth::user());
    }

    /**
     * Handle a login request to the application.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function login(Request $request)
    {
        if(Auth::check()){
            $this->logout();
        }
        
        // If the class is using the ThrottlesLogins trait, we can automatically throttle
        // the login attempts for this application. We'll key this by the username and
        // the IP address of the client making these requests into this application.
        $throttles = $this->isUsingThrottlesLoginsTrait();

        if ($throttles && $lockedOut = $this->hasTooManyLoginAttempts($request)) {
            $this->fireLockoutEvent($request);

            $seconds = $this->secondsRemainingOnLockout($request);
        
            return response()
                ->json($this->getLockoutErrorMessage($seconds));
        }
        
        
        if (Auth::attempt(['email' => $request->input('email'), 'password' => $request->input('password')])) {
            // Authentication passed...
            
            return response()
             ->json(Auth::user());
        } else {
            
            // If the login attempt was unsuccessful we will increment the number of attempts
            // to login and redirect the user back to the login form. Of course, when this
            // user surpasses their maximum number of attempts they will get locked out.
            if ($throttles && ! $lockedOut) {
                $this->incrementLoginAttempts($request);
            }
            
            return response()
                ->json($this->getFailedLoginMessage());
        }
    }
    
    public function logout()
    {
        Auth::logout();
    }
}
```
