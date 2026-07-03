# Source: https://rasa.com/docs/reference/integrations/action-server/running-action-server

On this page

Python action server, when built with Python, can be run by using rasa command or directly as a python module. The action server supports two protocols to invoke custom actions: HTTP and gRPC. By default, the action server runs on HTTP protocol. To run the action server on gRPC protocol, you need to specify the `--grpc` flag.

## Running the action server[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#running-the-action-server 'Direct link to Running the action server')

## Running action server over HTTP(S) protocol[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#running-action-server-over-https-protocol 'Direct link to Running action server over HTTP(S) protocol')

The action server supports both secure (HTTPS) and insecure HTTP connections.

### HTTP protocol[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#http-protocol 'Direct link to HTTP protocol')

To run action server over HTTP protocol use this command:

- Using Rasa command
- Directly as a python module

```
rasa run actions
```

```
python -m rasa_sdk
```

When running the action server over HTTP protocol, make sure that Rasa is also configured to [use HTTP protocol](https://rasa.com/docs/reference/integrations/action-server/actions/#http-protocol).

### HTTPS protocol[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#https-protocol 'Direct link to HTTPS protocol')

To run action server over HTTPS protocol use this command:

- Using Rasa command
- Directly as a python module

```
rasa run actions --ssl-certificate /path/to/ssl_server_certificate --ssl-keyfile /path/to/ssl_server_key
```

```
python -m rasa_sdk --ssl-certificate /path/to/ssl_server_certificate --ssl-keyfile /path/to/ssl_server_key
```

When running the action server over HTTPS protocol, make sure that the Rasa server is also configured to [use HTTPS protocol](https://rasa.com/docs/reference/integrations/action-server/actions/#https-protocol).

### Listen on specific address[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#listen-on-specific-address 'Direct link to Listen on specific address')

You can make your action server listen on a specific address using the `SANIC_HOST` environment variable:

- Using Rasa command
- Directly as a python module

```
SANIC_HOST=192.168.69.150 rasa run actions
```

```
SANIC_HOST=192.168.69.150 python -m rasa_sdk
```

## Running the action server on gRPC protocol[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#running-the-action-server-on-grpc-protocol 'Direct link to Running the action server on gRPC protocol')

The action server supports both secure and insecure gRPC connections. By default, the action server runs on insecure gRPC connection.

### Insecure gRPC connection[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#insecure-grpc-connection 'Direct link to Insecure gRPC connection')

To run the action server to accept insecure gRPC connections, use this command:

- Using Rasa command
- Directly as a python module

```
rasa run actions --grpc
```

```
python -m rasa_sdk --grpc
```

When running the action server to accept insecure gRPC connections, make sure that the Rasa server is also configured to [use insecure gRPC connections](https://rasa.com/docs/reference/integrations/action-server/actions/#insecure-grpc-connection).

### Secure gRPC connection[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#secure-grpc-connection 'Direct link to Secure gRPC connection')

To run the action server to accept secure gRPC connections, you need to specify the `--grpc`, together with the `--ssl-certificate` and `--ssl-keyfile` flags:

- Using Rasa command
- Directly as a python module

```
rasa run actions --grpc --ssl-certificate /path/to/ssl_server_certificate --ssl-keyfile /path/to/ssl_server_key
```

```
python -m rasa_sdk --grpc --ssl-certificate /path/to/ssl_server_certificate --ssl-keyfile /path/to/ssl_server_key
```

When running the action server to accept secure gRPC connections, make sure that the Rasa server is also configured to [use secure gRPC connections](https://rasa.com/docs/reference/integrations/action-server/actions/#secure-tls-grpc-connection).

## Specifying the actions module or package[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#specifying-the-actions-module-or-package 'Direct link to Specifying the actions module or package')

By default the action server will look for your actions in a file called `actions.py` or in a package directory called `actions`.

You can specify a different actions module or package with the `--actions` flag.

- Using Rasa command
- Directly as a python module

```
rasa run actions --actions my_actions
```

```
python -m rasa_sdk --actions my_actions
```

Using the command above, the action server will expect to find your actions in a file called `my_actions.py` or in a package directory called `my_actions`.

## Other options[​](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#other-options 'Direct link to Other options')

To see the full list of options for running the action server, run:

- Using Rasa command
- Directly as a python module

```
rasa run actions --help
```

```
python -m rasa_sdk --help
```

- [Running the action server](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#running-the-action-server)
- [Running action server over HTTP(S) protocol](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#running-action-server-over-https-protocol)
 - [HTTP protocol](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#http-protocol)
 - [HTTPS protocol](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#https-protocol)
 - [Listen on specific address](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#listen-on-specific-address)
- [Running the action server on gRPC protocol](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#running-the-action-server-on-grpc-protocol)
 - [Insecure gRPC connection](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#insecure-grpc-connection)
 - [Secure gRPC connection](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#secure-grpc-connection)
- [Specifying the actions module or package](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#specifying-the-actions-module-or-package)
- [Other options](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#other-options)