# Laravel console mutex

[![SensioLabsInsight](https://insight.sensiolabs.com/projects/e4083afa-8ca9-4ac0-8be8-9bfadcb05fa7/big.png)](https://insight.sensiolabs.com/projects/e4083afa-8ca9-4ac0-8be8-9bfadcb05fa7)

[![StyleCI](https://styleci.io/repos/59570052/shield?branch=master&style=flat)](https://styleci.io/repos/59570052)
[![Build Status](https://travis-ci.org/dmitry-ivanov/laravel-console-mutex.svg?branch=master)](https://travis-ci.org/dmitry-ivanov/laravel-console-mutex)
[![Coverage Status](https://coveralls.io/repos/github/dmitry-ivanov/laravel-console-mutex/badge.svg?branch=master)](https://coveralls.io/github/dmitry-ivanov/laravel-console-mutex?branch=master)

[![Latest Stable Version](https://poser.pugx.org/illuminated/console-mutex/v/stable)](https://packagist.org/packages/illuminated/console-mutex)
[![Latest Unstable Version](https://poser.pugx.org/illuminated/console-mutex/v/unstable)](https://packagist.org/packages/illuminated/console-mutex)
[![Total Downloads](https://poser.pugx.org/illuminated/console-mutex/downloads)](https://packagist.org/packages/illuminated/console-mutex)
[![License](https://poser.pugx.org/illuminated/console-mutex/license)](https://packagist.org/packages/illuminated/console-mutex)

Prevents overlapping for Laravel console commands.

## Requirements

- `PHP >=5.6.4`
- `Laravel >=5.2`

## Usage

1. Install package through `composer`:

    ```shell
    composer require illuminated/console-mutex
    ```

2. Use `Illuminated\Console\WithoutOverlapping` trait:

    ```php
    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminated\Console\WithoutOverlapping;

    class MyProtectedCommand extends Command
    {
        use WithoutOverlapping;

        // ...
    }
    ```

3. Now your command is protected against overlapping:

    ![Example](doc/img/example.png)

## Strategies

Overlapping can be prevented by various strategies:

- `file` (default)
- `mysql`
- `redis`
- `memcached`

Default `file` strategy is fine for a small applications, which are deployed on a single server.
If your application is more complex and, for example, is deployed on a several nodes, then you probably would like to use some other mutex strategy.

You can change mutex strategy by specifying `$mutexStrategy` field:

```php
class MyProtectedCommand extends Command
{
    use WithoutOverlapping;

    protected $mutexStrategy = 'mysql';

    // ...
}
```

Or by using `setMutexStrategy` method:

```php
class MyProtectedCommand extends Command
{
    use WithoutOverlapping;

    public function __construct()
    {
        parent::__construct();
    
        $this->setMutexStrategy('mysql');
    }

    // ...
}
```

## Advanced

Sometimes it is useful to set common mutex for a several commands. You can easily achieve this by setting them the same mutex name.
By default, mutex name is generated based on a command's name and arguments. To change this, just override `getMutexName` method in your command:

```php
class MyProtectedCommand extends Command
{
    use WithoutOverlapping;

    public function getMutexName()
    {
        return "icmutex-custom-name";
    }

    // ...
}
```

## Troubleshooting

### Trait included, but nothing happens?

Note, that `WithoutOverlapping` trait is overriding `initialize` method:

```php
trait WithoutOverlapping
{
    protected function initialize(InputInterface $input, OutputInterface $output)
    {
        $this->initializeMutex();
    }

    // ...
}
```

If your command is overriding `initialize` method too, then you should call `initializeMutex` method by yourself:

```php
class MyProtectedCommand extends Command
{
    use WithoutOverlapping;

    protected function initialize(InputInterface $input, OutputInterface $output)
    {
        $this->initializeMutex();

        $this->foo = $this->argument('foo');
        $this->bar = $this->argument('bar');
        $this->baz = $this->argument('baz');
    }

    // ...
}
```

### Several traits conflict?

If you're using some other cool `illuminated/console-%` packages, well, then you can find yourself getting "traits conflict".
For example, if you're trying to build [loggable command](https://github.com/dmitry-ivanov/laravel-console-logger), which is protected against overlapping:

```php
class MyProtectedCommand extends Command
{
    use Loggable;
    use WithoutOverlapping;

    // ...
}
```

You'll get fatal error, the "traits conflict", because both of these traits are overriding `initialize` method:
>If two traits insert a method with the same name, a fatal error is produced, if the conflict is not explicitly resolved.

But don't worry, solution is very simple. Override `initialize` method by yourself, and initialize traits in required order:

```php
class MyProtectedCommand extends Command
{
    use Loggable;
    use WithoutOverlapping;

    protected function initialize(InputInterface $input, OutputInterface $output)
    {
        $this->initializeMutex();
        $this->initializeLogging();
    }

    // ...
}
```
