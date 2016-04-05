How to make a plugin?
=====================
Plugins are passed in ```forStream``` or ```forStreams``` methods, receiving streams and hadling them.

## Creating a plugin
Your plugin must implements ```Junty\Plugin\PluginInterface```, so, first, install the package for plugins

```console
$ composer require junty/junty-plugin
```

After, create a class for your plugin, with methods ```getName() : string```, that will return the plugin name, and ```getCallback() : callable```, that will be the plugin callback for handle streams.
```php
namespace My\Plugin;

use Junty\Plugin\PluginInterface;

class MyPlugin implements PluginInterface
{
    public function getName() : string
    {
        return 'my_plugin';
    }

    public function getCallback() : callable
    {
        // Passing array as argument for beeing used with forStreams method
        return function (array $streams) {
            // handle streams...
        };
    }
}
```

## Publishing the plugin
Go to [official plugins repository](https://github.com/the-junty/junty-plugins) and fork it. Add your plugin to ```README.md``` and ```plugins.json```.

You plugin MUST follow this structure on ```README.md```

```
* [Plugin name](https://github.com/YOU-USER/junty-PLUGIN_NAME) - Plugin short description
```

And this on ```plugins.json```

```json
{
  "name": "[Plugin name]",
  "description": "Plugin short descriptionE",
  "url": "https://github.com/YOU-USER/junty-PLUGIN_NAME"
}
```
> Use 2 spaces for identation.

## Done!
Your plugin can be found here now: [the-junty.github.io/plugins.html](http://the-junty.github.io/plugins.html)