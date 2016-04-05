Usage
=====

## Index
* [Instance (creating ```juntyfile.php```)](#instance)
* [Creating tasks and groups](#creating-tasks-and-groups)
  * [Tasks](#tasks)
    * [via ```function```](#via-function)
    * [via ```class```](#via-class)
  * [Groups](#groups)
    * [via ```function```](#via-function-1)
    * [via ```class```](#via-class-1)
* [Stream handling methods](#stream-handling-methods)
  * [About ```Junty\Stream\Stream```](#about-juntystreamstream)
  * [```src```](#src)
  * [```forStreams```](#forstreams)
  * [```forStream```](#forStream)
  * [```push```](#push)
  * [```end```](#end)
  * [```toDir```](#todir)
  * [```Stream::setContents```](#streamsetcontents)
  * [```Stream::save```](#streamsave)
  * [```Stream::pipe```](#streampipe)
* [Ordering tasks](#ordering-tasks)
* [Running](#running)

### Instance
Create a file called ```juntyfile.php``` returning the ```Runner``` instance.
```php
<?php
require 'vendor/autoload.php';

use Junty\Runner\JuntyRunner;
use Junty\Stream\Stream;

$junty = new JuntyRunner();

//... tasks

return $junty;
```

## Creating tasks and groups
### Tasks
Tasks will handle files via ```src```, ```forStream``` and ```forStreams``` or execute some code. Anything can be inside a task!
Tasks can be created in two ways: via ```function``` and a ```class```.

#### via ```function```
```php
$junty->task('copy_php_files', function () {
    $this->src('*.php')
        ->forStream(function (Stream $stream) {
            $this->push($stream);
        })
        ->forStreams($this->toDir('php_files'));
});
```

#### via ```class```
Class must implements ```Junty\TaskRunner\Task\TaskInterface``` or extends ```Junty\TaskRunner\Task\AbstractTask```.
```php
use Junty\TaskRunner\Task\AbstractTask;

$junty->task(new class() extends AbstractTrask
{
    public function getName() : string
    {
        return 'copy_php_files';
    }
    
    public function getCallback() : callable
    {
        return function () {
            $this->src('*.php')
                ->forStream(function (Stream $stream) {
                    $this->push($stream);
                })
                ->forStreams($this->toDir('php_files'));
        };
    }
});
```

### Groups
Your can group your tasks, per example, by their category.
The creating is the same of tasks: via ```function``` or ```class```.

#### via ```function```
```php
$junty->group('minify', function () {
    $this->task('JS', function () {
        $this->src('./public/js/*.js')
            ->forStreams(new JsMinify())
            ->forStreams($this->toDir('./public/dist/js'));
    }); 

    $this->task('CSS', function () {
        $this->src('./public/css/*.css')
            ->forStreams(new CssMinify())
            ->forStreams($this->toDir('./public/dist/css'));
    }); 
});
```

#### via ```class```
```php
use Junty\TaskRunner\Task\{Group, TasksCollection};

$junty->group(new class() extends Group
{
    public function __construct()
    {
    }

    public function task($name, callable $callback = null)
    {
    }

    public function getName() : string
    {
        return 'minify';
    }

    public function getTasks() : TasksCollection
    {
        $collection = TasksCollection();

        $collection->set('JS', function () {
            $this->src('./public/js/*.js')
                ->forStreams(new JsMinify())
                ->forStreams($this->toDir('./public/dist/js'));
        });

        $collection->set('CSS', function () {
            $this->src('./public/css/*.css')
                ->forStreams(new CssMinify())
                ->forStreams($this->toDir('./public/dist/css'));
        });

        return $collection;
    }
});
```

## Stream handling methods
### About ```Junty\Stream\Stream```
```Junty\Stream\Stream``` implements the [PSR-7 Stream Interface (```Psr\Http\Message\StreamInterface```)](http://www.php-fig.org/psr/psr-7/#3-4-psr-http-message-streaminterface).

### ```src```
Provides streams by the pattern passed.
```php
$this->src('*.php')
    ->forStreams(function (array $streams) {
        foreach ($streams as $stream) {
            echo $stream->getMetaData('uri');
        }
    });
```

### ```forStreams```
Provides all streams gotten from ```src```.
```php
$this->src('*.txt')
    ->forStreams(function (array $streams) {
        foreach ($streams as $stream) {
            // Handle each stream
        }
    });
```

### ```forStream```
Handle each stream gotten from ```src```.
```php
$this->src('*.txt')
    ->forStream(function (Stream $streams) {
       // Handle unique stream
    });
```

### ```push```
Pushed stream will substitute the list from ```src```.
```php
$this->src('*.txt')
    ->forStreams(function (array $streams) {
        foreach ($streams as $stream) {
            if ($stream->getContents() === 'hello') {
                $this->push($stream);
            }
        }
    })
    ->forStreams(function (array $streams) {
        foreach ($streams as $stream) {
            echo $stream->getContents(); // 'hello' for each stream
        }
    });
```

### ```end```
Empties list of streams.
```php
$this->src('*.txt')
    ->forStreams(function (array $streams) {
        foreach ($streams as $stream) {
            echo $stream->getContents(); // 'hello' for each stream
        }
    })
    ->end();
    ->src('*.php')
        // ...
```

### ```toDir```
Put all streammed files on a certain directory.
```php
$this->src('*.php')
    ->forStreams($this->toDir('php_files')); // Copy all files to php_files
```

### ```Stream::setContents```
Updates the contents of a stream.
```php
->forStream(function (Stream $stream) {
    $stream->setContents('Hello! ;)');
}); // Copy all files to php_files
```

### ```Stream::save```
Saves the stream contents modifications.
```php
$stream->setContents('Hello! ;)');
$stream->save();
```

### ```Stream::pipe```
Pipes a stream to another. It's possible to pass an instance of ```Junty\Stream\Stream``` or a PHP Stream Resource.
```php
use Junty\Stream\Stream;

// PHP resource
$stream->pipe(fopen('destination.txt', 'w+'));

// Stream instance
$stream->pipe(new Stream(fopen('destination.txt', 'w+')));
```

## Ordering tasks
You set the execution order of tasks using the method ```JuntyRunner::order```.
```php
// ... tasks

$junty->order('task_2', 'task_1', 'task_4', 'task_3');

return $junty;
```

## Running
```bash
$ vendor/bin/junty
```

## Utils
* [How to create a plugin?](https://github.com/the-junty/junty-docs/blob/master/docs/Plugins.md)