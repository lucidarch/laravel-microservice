# Lucid • Microservice - Laravel

With the emerging need for separation per concern, microservices emerged to be the trending progression
of what used to be a monolithic application. Especially with Lucid, scale is one of the core concerns that
a microservice is the natural progression of the architecture from its [monolithic counterpart](https://github.com/lucid-architecture/laravel).

It is no coincidence that the different parts of a Lucid monolithic application are called a **Service**, for microservices
are indeed the next progression when applications scale and reach that turning point. Having implemented your application
using Lucid, the transition process will be logically simpler to think about and physically straight-forward to
implement.

To see how it compares to the monolithic application and when to use which, check [Monolith vs. Microservice](#monolith-vs-microservice)

### Join The Community on Slack
[![Slack Status](https://lucid-slack.herokuapp.com/badge.svg)](https://lucid-slack.herokuapp.com)

##### The Lucid Architecture for Building Scalable Applications - Laracon EU 2016
Check out my talk at LaraconEU 2016 where I introduce the concept and application of the Lucid architecture:
[![Abed Halawi - The Lucid Architecture for Building Scalable Applications](http://img.youtube.com/vi/wSnM4JkyxPw/0.jpg)](http://www.youtube.com/watch?v=wSnM4JkyxPw "Abed Halawi - The Lucid Architecture for Building Scalable Applications")

## Installation
To get rolling, you need to create a new project using Composer:
```
composer create-project lucid-arch/laravel-microservice my-project
```

## Getting Started
This project ships with the [Lucid Console](https://github.com/lucid-architecture/laravel-console) which provides an interactive
user interface and a command line interface that are useful for scaffolding and exploring Services, Features and Jobs.

### Setup
The `lucid` executable will be in `vendor/bin`. If you don't have `./vendor/bin/` as part of your `PATH` you will
need to execute it using `./vendor/bin/lucid`, otherwise add it with the following command to be able to simply
call `lucid`:

```
export PATH="./vendor/bin:$PATH"
```

For a list of all the commands that are available run `lucid` or see the [CLI Reference](https://github.com/lucid-architecture/laravel-console).

### 1. Create a Feature
This is the Feature that we will be serving when someone visits our `/users` route.
```
lucid make:feature ListUsers
```

### 2. Create a Job
This Job will fetch the users from the database and will be used inside our Feature to serve them.
```
lucid make:job GetUsers user
```

Open the file that was generated at `app/Domains/User/GetUsersJob.php` and edit the `handle` method to this:

```php
public function handle()
{
    return [
        ['name' => 'John Doe'],
        ['name' => 'Jane Doe'],
        ['name' => 'Tommy Atkins'],
    ];
}
```

In a real-world application you might want to fetch the users from a database, and here is the perfect place for that.
Here's an example of fetching a list of users and providing the ability to specify the limit:

```php
use App\Data\Models\User;

class GetUsersJob extends Job
{
    private $limit;

    public function __construct($limit = 25)
    {
        $this->limit = $limit;
    }

    public function handle(User $user)
    {
        return $user->take($this->limit)->get();
    }
}
```

> NOTE: The namespace for models is `[app namespace]\Data\Models`

### 3. Run The Job
```php
// ...
use App\Domains\User\GetUsersJob;
use App\Domains\Http\RespondWithJsonJob;
// ...
public function handle(Request $request)
{
    $users = $this->run(GetUsersJob::class);

    return $this->run(new RespondWithJsonJob($users));
}
```

The `RespondWithJsonJob` is one of the Jobs that were shipped with this project, it lives in the `Http` domain and is
used to respond to a request in structured JSON format.

### 4. Serve The Feature
To be able to serve that Feature we need to create a route and a controller that does so.

Generate a plain controller with the following command

```
lucid make:controller user
```

And we will have our `UserController` generated in `app/Http/Controllers/UserController.php` which we will use
to serve our Feature in its `index` method.

We just need to create a route that would delegate the request to our `index` method:

```php
// ...
use App\Features\ListUsersFeature;
// ...
class UserController extends Controller
{
    public function index()
    {
        return $this->serve(ListUsersFeature::class);
    }
}
```

In `routes/web.php` add:

```php
Route::get('/users', 'UserController@index');
```

That's it! Now serve the application with `php artisan serve` and visit `http://localhost:8000/users`

---

## Monolith vs. Microservice
In the monolith Lucid application we have multiple services (i.e. Api, Web) and these typically will exist in
`src/Services/Api` and `src/Services/Web` respectively. With the microservice the `src` does not exist, since
it is intended to be one service serving a single purpose, the `app` directory will do.
It will hold the following directories:
- **Data** For all your models, repositories and value objects.
- **Domains** Holds the Domains and their Jobs.
- **Features** The service's Features.

### Directory Structure
| Component | Monolith | Microservice |
|---------|---------|--------------|
|Job | src/Domains/[domain]/Jobs/[job] | app/Domains/[domain]/Jobs/[job] |
| Feature | src/Services/[service]/Features/[feature] | app/Features/[feature] |
| Service | src/Service/[service] | N/A (app) as equivalent |

### Tests
One other significant difference is in the location of tests:

|Component|Monolith | Microservice |
|---------|---------|--------------|
|Job | src/Domains/[domain]/Tests/Jobs/[JobTest] | tests/Domains/[domain]/Jobs/[JobTest] |
| Feature | src/Services/[service]/Tests/Features/[FeatureTest] | tests/Features/[Feature]Test.php |

## How To Choose
It is always [recommended](http://martinfowler.com/bliki/MonolithFirst.html) that you start with a monolith
and work on it until it gets so big that it is crucial to be dissected into single-purpose services (microservices).
It would be challenging to be able to figure out the different services your application would need moving forward.

This project is also useful when you know for sure that your application will not have to deal with multiple Services but you would still like to use Lucid.

---

# Laravel PHP Framework

[![Build Status](https://travis-ci.org/laravel/framework.svg)](https://travis-ci.org/laravel/framework)
[![Total Downloads](https://poser.pugx.org/laravel/framework/d/total.svg)](https://packagist.org/packages/laravel/framework)
[![Latest Stable Version](https://poser.pugx.org/laravel/framework/v/stable.svg)](https://packagist.org/packages/laravel/framework)
[![Latest Unstable Version](https://poser.pugx.org/laravel/framework/v/unstable.svg)](https://packagist.org/packages/laravel/framework)
[![License](https://poser.pugx.org/laravel/framework/license.svg)](https://packagist.org/packages/laravel/framework)

Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable, creative experience to be truly fulfilling. Laravel attempts to take the pain out of development by easing common tasks used in the majority of web projects, such as authentication, routing, sessions, queueing, and caching.

Laravel is accessible, yet powerful, providing tools needed for large, robust applications. A superb inversion of control container, expressive migration system, and tightly integrated unit testing support give you the tools you need to build any application with which you are tasked.

## Official Documentation

Documentation for the framework can be found on the [Laravel website](http://laravel.com/docs).

## Contributing

Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the [Laravel documentation](http://laravel.com/docs/contributions).

## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell at taylor@laravel.com. All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).
