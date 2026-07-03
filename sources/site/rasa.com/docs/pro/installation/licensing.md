# Source: https://rasa.com/docs/pro/installation/licensing

Rasa offers Enterprise licenses for teams building assistants at scale, with advanced features like enterprise-grade security, scalability, and dedicated support. But if you're just getting started, you can get a free Developer Edition License to try it out.

[**Developer Edition**\\ \\ If you want to start building assistants with CALM you can get started today with the Rasa Developer Edition.](https://rasa.com/rasa-pro-developer-edition-license-key-request/)

 [Get it here](https://rasa.com/rasa-pro-developer-edition-license-key-request/)

## How Licensing Works[​](https://rasa.com/docs/pro/installation/licensing/#how-licensing-works 'Direct link to How Licensing Works')

Rasa will look for your license in the env var `RASA_LICENSE`, which must contain the content of the license key file provided by rasa.

You can set the `RASA_LICENSE` env var temporarily in your terminal, but it is recommended to set it persistently so that you don't have to set it every time you run Rasa.

- Bash
- Windows Powershell

```
## Temporaryexport RASA_LICENSE=YOUR_LICENSE_STRING_HERE## Persistentecho "export RASA_LICENSE=YOUR_LICENSE_STRING_HERE" >> ~/.bashrc## If you're using a different flavor of bash e.g. Zsh, replace .bashrc with your shell's initialization script e.g. ~/.zshrc
```

```
## Temporary$env: RASA_LICENSE=YOUR_LICENSE_STRING_HERE## Persistent for the current user[System.Environment]::SetEnvironmentVariable('RASA_LICENSE','YOUR_LICENSE_STRING_HERE','USER')
```

Then you can use the [`rasa` CLI](https://rasa.com/docs/reference/api/command-line-interface/) as usual, for example:

```
rasa init
```