# Source: https://rasa.com/docs/pro/installation/troubleshooting

On this page

### Python 3.10 requirements[​](https://rasa.com/docs/pro/installation/troubleshooting/#python-310-requirements 'Direct link to Python 3.10 requirements')

_If you are using Linux_, installing `rasa-pro` could result in a failure while installing `tokenizers` and `cryptography`.

In order to resolve it, you must follow these steps to install a Rust compiler:

```
apt install rustc && apt install cargo
```

After initializing the Rust compiler, you should restart the console and check its installation:

```
rustc --version
```

In case the PATH variable had not been automatically setup, run:

```
export PATH="$HOME/.cargo/bin:$PATH"
```

_If you are using macOS_, note that installing `rasa-pro` could result in a failure while installing `tokenizers` (issue described in depth [here](https://github.com/huggingface/tokenizers/issues/1050)).

In order to resolve it, you must follow these steps to install a Rust compiler:

```
brew install rustuprustup-init
```

After initializing the Rust compiler, you should restart the console and check its installation:

```
rustc --version
```

In case the PATH variable had not been automatically setup, run:

```
export PATH="$HOME/.cargo/bin:$PATH"
```

### OSError: \[Errno 40\] Message too long[​](https://rasa.com/docs/pro/installation/troubleshooting/#oserror-errno-40-message-too-long 'Direct link to OSError: [Errno 40] Message too long')

If you run `rasa train` or `rasa run` after you’ve enabled tracing using Jaeger as a backend in your local development environment, you might come across this error `OSError: [Errno 40] Message too long`.

This could be caused by the OS of your local development environment restricting UDP packet size. You can find out the current UDP packet size by running `sysctl net.inet.udp.maxdgram` on macOS. You can increase UDP packet size by running `sudo sysctl -w net.inet.udp.maxdgram=65535`.

- [Python 3.10 requirements](https://rasa.com/docs/pro/installation/troubleshooting/#python-310-requirements)
- [OSError: \[Errno 40\] Message too long](https://rasa.com/docs/pro/installation/troubleshooting/#oserror-errno-40-message-too-long)