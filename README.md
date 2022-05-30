# Event Sourcing with Axon Server

This package supports using the Python
[eventsourcing](https://github.com/pyeventsourcing/eventsourcing) library
with [Axon Server](https://developer.axoniq.io/axon-server).

## Installation

Use pip to install the stable distribution from the Python Package Index.

    $ pip install eventsourcing-axonserver

Please note, it is recommended to install Python packages into a Python virtual environment.

## Getting started

Define aggregates and applications in the usual way. Please note, aggregate
sequences  in Axon Server are expected to start from position `0`, whereas
the default for the library's `Aggregate` class is to start from `1`. So we
need to set the `INITIAL_VERSION` attribute on the aggregate class to `0`.

```python
from typing import Any, Dict
from uuid import UUID

from eventsourcing.application import Application
from eventsourcing.domain import Aggregate, event


class TrainingSchool(Application):
    def register(self, name: str) -> UUID:
        dog = Dog(name)
        self.save(dog)
        return dog.id

    def add_trick(self, dog_id: UUID, trick: str) -> None:
        dog = self.repository.get(dog_id)
        dog.add_trick(trick)
        self.save(dog)

    def get_dog(self, dog_id) -> Dict[str, Any]:
        dog = self.repository.get(dog_id)
        return {'name': dog.name, 'tricks': tuple(dog.tricks)}


class Dog(Aggregate):
    INITIAL_VERSION = 0

    @event('Registered')
    def __init__(self, name: str) -> None:
        self.name = name
        self.tricks = []

    @event('TrickAdded')
    def add_trick(self, trick: str) -> None:
        self.tricks.append(trick)
```

Configure the application to use Axon Server. Set environment variable
`PERSISTENCE_MODULE` to `'eventsourcing_axonserver'`, and set
`AXONSERVER_URI` to the host and port of your Axon Server.

```python
school = TrainingSchool(env={
    "PERSISTENCE_MODULE": "eventsourcing_axonserver",
    "AXONSERVER_URI": "localhost:8124",
})
```

The application's methods may be then called, from tests and
user interfaces.

```python
# Register dog.
dog_id = school.register('Fido')

# Add tricks.
school.add_trick(dog_id, 'roll over')
school.add_trick(dog_id, 'play dead')

# Get details.
dog = school.get_dog(dog_id)
assert dog["name"] == 'Fido'
assert dog["tricks"] == ('roll over', 'play dead')
```

For more information, please refer to the Python
[eventsourcing](https://github.com/johnbywater/eventsourcing) library
and the [Axon Server](https://developer.axoniq.io/axon-server) project.

## Developers

### Install Poetry

The first thing is to check you have Poetry installed.

    $ poetry --version

If you don't, then please [install Poetry](https://python-poetry.org/docs/#installing-with-the-official-installer).

It will help to make sure Poetry's bin directory is in your `PATH` environment variable.

But in any case, make sure you know the path to the `poetry` executable. The Poetry
installer tells you where it has been installed, and how to configure your shell.

Please refer to the [Poetry docs](https://python-poetry.org/docs/) for guidance on
using Poetry.

### Setup for PyCharm users

You can easily obtain the project files using PyCharm (menu "Git > Clone...").
PyCharm will then usually prompt you to open the project.

Open the project in a new window. PyCharm will then usually prompt you to create
a new virtual environment.

Create a new Poetry virtual environment for the project. If PyCharm doesn't already
know where your `poetry` executable is, then set the path to your `poetry` executable
in the "New Poetry Environment" form input field labelled "Poetry executable". In the
"New Poetry Environment" form, you will also have the opportunity to select which
Python executable will be used by the virtual environment.

PyCharm will then create a new Poetry virtual environment for your project, using
a particular version of Python, and also install into this virtual environment the
project's package dependencies according to the `pyproject.toml` file, or the
`poetry.lock` file if that exists in the project files.

You can add different Poetry environments for different Python versions, and switch
between them using the "Python Interpreter" settings of PyCharm. If you want to use
a version of Python that isn't installed, either use your favourite package manager,
or install Python by downloading an installer for recent versions of Python directly
from the [Python website](https://www.python.org/downloads/).

Once project dependencies have been installed, you should be able to run tests
from within PyCharm (right-click on the `tests` folder and select the 'Run' option).

Because of a conflict between pytest and PyCharm's debugger and the coverage tool,
you may need to add ``--no-cov`` as an option to the test runner template. Alternatively,
just use the Python Standard Library's ``unittest`` module.

You should also be able to open a terminal window in PyCharm, and run the project's
Makefile commands from the command line (see below).

### Setup from command line

Obtain the project files, using Git or suitable alternative.

In a terminal application, change your current working directory
to the root folder of the project files. There should be a Makefile
in this folder.

Use the Makefile to create a new Poetry virtual environment for the
project and install the project's package dependencies into it,
using the following command.

    $ make install-packages

If you want to skip the installation of your project's package, use the
`--no-root` option.

    $ make install-packages --no-root

Please note, if you create the virtual environment in this way, and then try to
open the project in PyCharm and configure the project to use this virtual
environment as an "Existing Poetry Environment", PyCharm sometimes has some
issues (don't know why) which might be problematic. If you encounter such
issues, you can resolve these issues by deleting the virtual environment
and creating the Poetry virtual environment using PyCharm (see above).

### Project Makefile commands

You can start Axon Server using the following command.

    $ make start-axon-server

You can run tests using the following command (needs Axon Server to be running).

    $ make test

You can stop Axon Server using the following command.

    $ make stop-axon-server

You can check the formatting of the code using the following command.

    $ make lint

You can reformat the code using the following command.

    $ make fmt

Tests belong in `./tests`. Code-under-test belongs in `./eventsourcing_axonserver`.

Edit package dependencies in `pyproject.toml`. Update installed packages (and the
`poetry.lock` file) using the following command.

    $ make update-packages
