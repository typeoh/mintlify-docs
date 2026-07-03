# Source: https://rasa.com/docs/reference/integrations/action-server/sanic-extensions

On this page

New in 3.6

You can now extend Sanic features such as middlewares, listeners, background tasks and additional routes.

You can now create additional Sanic extensions by accessing the app object created by the action server. The hook implemented in the plugin package provides you access to the Sanic app object created by `rasa-sdk` when starting the action server.

## Step-by-step guide on creating your own Sanic extension in rasa\_sdk[​](https://rasa.com/docs/reference/integrations/action-server/sanic-extensions/#step-by-step-guide-on-creating-your-own-sanic-extension-in-rasa_sdk 'Direct link to Step-by-step guide on creating your own Sanic extension in rasa_sdk')

This example will show you how to create a Sanic listener using plugins.

### Create the rasa\_sdk\_plugins package[​](https://rasa.com/docs/reference/integrations/action-server/sanic-extensions/#create-the-rasa_sdk_plugins-package 'Direct link to Create the rasa_sdk_plugins package')

Create a package in your action server project which you must name `rasa_sdk_plugins`. Rasa SDK will try to instantiate this package in your project to start plugins. If no plugins are found, it will print a debug log that there are no plugins in your project.

### Register modules containing the hooks[​](https://rasa.com/docs/reference/integrations/action-server/sanic-extensions/#register-modules-containing-the-hooks 'Direct link to Register modules containing the hooks')

Create the package `rasa_sdk_plugins` and initialize the hooks by creating an `__init__.py` file where the plugin manager will look for the module where the hooks are implemented:

rasa\_sdk\_plugins/\_\_init\_\_.py

```
def init_hooks(manager: pluggy.PluginManager) -> None:    """Initialise hooks into rasa sdk."""    import sys    import rasa_sdk_plugins.your_module    logger.info("Finding hooks")    manager.register(sys.modules["rasa_sdk_plugins.your_module"])
```

### Implement your hook[​](https://rasa.com/docs/reference/integrations/action-server/sanic-extensions/#implement-your-hook 'Direct link to Implement your hook')

Implement the hook `attach_sanic_app_extensions`. This hook forwards the app object created by Sanic in the `rasa_sdk` and allows you to create additional routes, middlewares, listeners and background tasks. Here's an example of this implementation that creates a listener.

In your `rasa_sdk_plugins.your_module.py`:

rasa\_sdk\_plugins/your\_module.py

```
from __future__ import annotationsimport loggingimport pluggyfrom asyncio import AbstractEventLoopfrom functools import partiallogger = logging.getLogger(__name__)hookimpl = pluggy.HookimplMarker("rasa_sdk")@hookimpl  # type: ignore[misc]def attach_sanic_app_extensions(app: Sanic) -> None:    logger.info("hook called")    app.register_listener(        partial(before_server_start),        "before_server_start",    )async def before_server_start(app: Sanic, loop: AbstractEventLoop):    logger.info("BEFORE SERVER START")
```

- [Step-by-step guide on creating your own Sanic extension in rasa\_sdk](https://rasa.com/docs/reference/integrations/action-server/sanic-extensions/#step-by-step-guide-on-creating-your-own-sanic-extension-in-rasa_sdk)
 - [Create the rasa\_sdk\_plugins package](https://rasa.com/docs/reference/integrations/action-server/sanic-extensions/#create-the-rasa_sdk_plugins-package)
 - [Register modules containing the hooks](https://rasa.com/docs/reference/integrations/action-server/sanic-extensions/#register-modules-containing-the-hooks)
 - [Implement your hook](https://rasa.com/docs/reference/integrations/action-server/sanic-extensions/#implement-your-hook)