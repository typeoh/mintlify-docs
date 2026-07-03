# Source: https://rasa.com/docs/pro/installation/python

On this page

**Rasa** has a Python package called `rasa-pro` that you can install locally, with `uv` package manager. You can use `pip` but it'll take longer to install.

## Set Up Your Python Environment[​](https://rasa.com/docs/pro/installation/python/#set-up-your-python-environment 'Direct link to Set Up Your Python Environment')

info

Rasa supports Python `3.10`, `3.11`, `3.12`, and `3.13`. For detailed information about supported Python versions and dependency groups, see our [Python Versions and Dependencies](https://rasa.com/docs/reference/python-versions-and-dependencies/) reference page.

Check if your Python environment is already configured by ensuring you have Python 3.10 or 3.11.

Check Python Version

```
python --versionpip --version
```

If the right packages are already installed, you can skip to the next step.

Otherwise, proceed with the instructions below to install them.

- macOS and Linux
- Windows

Install the package manager uv by executing this command in the Terminal. Other installation methods are documented [in uv documentation](https://docs.astral.sh/uv/getting-started/installation/).

```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Install the package manager [uv](https://docs.astral.sh/uv/getting-started/installation/) by executing this command in the Windows Powershell.

```
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

In the next step, we will be using uv to install Python.

### Create Virtual Environment[​](https://rasa.com/docs/pro/installation/python/#create-virtual-environment 'Direct link to Create Virtual Environment')

Python uses [Virtual Environments](https://docs.python.org/3/tutorial/venv.html) to isolate project dependencies. They help avoid conflicts between different projects by creating a clean workspace where you can install specific package versions for each project. We recommend installing rasa-pro within a virtual environment. You can follow the instructions below to set them up,

Each Rasa project should be started in a new directory.

```
mkdir my-rasa-assistantcd my-rasa-assistant
```

Create a new virtual environment with Python 3.11. uv will install Python if it is not already installed.

```
uv venv --python 3.11
```

Virtual environments only need to be created once for each project, but you must activate them every time you open a new terminal session to work on your project. Activate the virtual environment

- macOS and Linux
- Windows

```
source .venv/bin/activate
```

```
.venv\Scripts\activate
```

This activation step ensures your commands use the project's isolated dependencies rather than your system-wide Python installation.

### Install Rasa[​](https://rasa.com/docs/pro/installation/python/#install-rasa 'Direct link to Install Rasa')

Now we are ready to install Rasa. To install the latest version of Rasa with uv, run the command:

```
uv pip install rasa-pro
```

Don't forget to obtain your Rasa Developer Edition License from the [license request page](https://rasa.com/rasa-pro-developer-edition-license-key-request/). Export it as an environment variable:

- macOS and Linux
- Windows

```
export RASA_LICENSE=YOUR_LICENSE_KEY
```

```
set RASA_LICENSE=YOUR_LICENSE_KEY
```

Check the Rasa version:

```
rasa --version
```

Create a new template CALM assistant from a template:

```
rasa init --template tutorial
```

That's it - you should now be ready to dive into [building your first bot](https://rasa.com/docs/pro/tutorial/)! For troubleshooting tips or FAQs on install see our [troubleshooting guide](https://rasa.com/docs/pro/installation/troubleshooting/).

- [Set Up Your Python Environment](https://rasa.com/docs/pro/installation/python/#set-up-your-python-environment)
 - [Create Virtual Environment](https://rasa.com/docs/pro/installation/python/#create-virtual-environment)
 - [Install Rasa](https://rasa.com/docs/pro/installation/python/#install-rasa)