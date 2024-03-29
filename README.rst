======================
Github Action Testing
======================

A repo for testing github actions.

--------
Workflow
--------

|build| |codeql| |dependency-review|

--------
Analysis
--------

|coverage| |successful-test| |utilization| |maintainability-percentage|

-----------
Development
-----------

|falcon| |poetry| |pre-commit| |code-style|

------------
Installation
------------

Install the extension via pip.

.. code:: bash

    > pip install gh-action-testing

--------
Overview
--------

This package provides a middleware logging component for the Falcon Web Framework (https://falcon.readthedocs.io/en/stable/index.html). The component provides two logging handlers that are pre-configured with sane defaults, yet fully customizable. The ``rotating_handler()`` method allows log events to be easily written to a file on disk, while the ``syslog_handler()`` method allows sending the log events over the network (both UDP and TCP).

Features
--------

* Pre-configure rotating file logging handlers with sane defaults, which can also be fully customized.
* Pre-configure syslog logging handlers with sane defaults, which can also be fully customized.
* Support for any custom logging handlers.
* Supports using a pre-existing logger instance.
* Supports multiple handler (e.g., using both file, syslog handlers, and/or custom handler).

.. IMPORTANT:: This middleware component should be one of the first middleware component loaded to make it available for other components. From the Falcon docs "*Each component’s process_request, process_resource, and process_response methods are executed hierarchically, as a stack, following the ordering of the list passed via the middleware kwarg of falcon.App.*".

--------
Requires
--------
* Falcon - https://pypi.org/project/falcon/

---------------------
Rotating File Handler
---------------------
All values passed to the handler function are optional kwargs.

+-----------------+---------------------+----------------------------------------------------------+
| Setting         | Default             | Description                                              |
+=================+=====================+==========================================================+
| backup_count    | 10                  | The number of backup log files to keep.                  |
+-----------------+---------------------+----------------------------------------------------------+
| directory       | log                 | The directory to write the log file.                     |
+-----------------+---------------------+----------------------------------------------------------+
| filename        | server.log          | The name of the log file.                                |
+-----------------+---------------------+----------------------------------------------------------+
| formatter       | A sane formatter    | A logging formatter to format log events.                |
|                 | w/ module/lineno    |                                                          |
+-----------------+---------------------+----------------------------------------------------------+
| level           | INFO                | The level for the logger.                                |
+-----------------+---------------------+----------------------------------------------------------+
| name            | rfh                 | A unique name for the handler.                           |
+-----------------+---------------------+----------------------------------------------------------+
| max_bytes       | 10485760            | The maximum size of the log file before rotating.        |
+-----------------+---------------------+----------------------------------------------------------+
| mode            | a (append)          | The write mode for the log file.                         |
+-----------------+---------------------+----------------------------------------------------------+

Basic Example
-------------
The example below is a basic logger middleware using default values as defined in table above.

.. code:: python

    import falcon
    from gh_action_testing.middleware import LoggerMiddleware
    from gh_action_testing.utils import rotating_handler


    class LoggerMiddleWareResource(object):
        """Logger middleware testing resource."""

        def on_get(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            self.log.debug(f'DEBUG {key}')
            self.log.info(f'INFO {key}')
            self.log.warning(f'WARNING {key}')
            self.log.error(f'ERROR {key}')
            self.log.critical(f'CRITICAL {key}')
            resp.text = f'Logged - {key}'

        def on_post(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            value = req.get_param('value')
            self.log.debug(f'DEBUG {key} {value}')
            self.log.info(f'INFO {key} {value}')
            self.log.warning(f'WARNING {key} {value}')
            self.log.error(f'ERROR {key} {value}')
            self.log.critical(f'CRITICAL {key} {value}')
            resp.text = f'Logged - {key}'

    rh = rotating_handler()
    app = falcon.App(middleware=[LoggerMiddleware([rh])])
    app.add_route('/middleware', LoggerMiddleWareResource())

Advanced Example
----------------
The example below shows a heavily customized logger.

.. code:: python

    import falcon
    from gh_action_testing.middleware import LoggerMiddleware
    from gh_action_testing.utils import rotating_handler


    class LoggerMiddleWareResource(object):
        """Logger middleware testing resource."""

        def on_get(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            self.log.debug(f'DEBUG {key}')
            self.log.info(f'INFO {key}')
            self.log.warning(f'WARNING {key}')
            self.log.error(f'ERROR {key}')
            self.log.critical(f'CRITICAL {key}')
            resp.text = f'Logged - {key}'

        def on_post(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            value = req.get_param('value')
            self.log.debug(f'DEBUG {key} {value}')
            self.log.info(f'INFO {key} {value}')
            self.log.warning(f'WARNING {key} {value}')
            self.log.error(f'ERROR {key} {value}')
            self.log.critical(f'CRITICAL {key} {value}')
            resp.text = f'Logged - {key}'

    rh = rotating_handler(
        backup=5,
        directory='/var/log/',
        filename='my-app.log',
        formatter='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        level='debug',
        name='my-rfh',
        max_bytes='5000',
        mode='w',
    )
    app = falcon.App(middleware=[LoggerMiddleware(handlers=[rh], level='INFO', name='MY-LOGGER')])
    app.add_route('/middleware', LoggerMiddleWareResource())

--------------
Syslog Handler
--------------
All values passed to the handler function are optional kwargs.

+-----------------+---------------------+----------------------------------------------------------+
| Setting         | Default             | Description                                              |
+=================+=====================+==========================================================+
| host            | localhost           | The host name or IP of syslog server.                    |
+-----------------+---------------------+----------------------------------------------------------+
| facility        | user                | The syslog facility.                                     |
+-----------------+---------------------+----------------------------------------------------------+
| formatter       | A sane formatter    | A logging formatter to format log events.                |
|                 | w/ module/lineno    |                                                          |
+-----------------+---------------------+----------------------------------------------------------+
| level           | INFO                | The level for the logger.                                |
+-----------------+---------------------+----------------------------------------------------------+
| name            | sh                  | A unique name for the handler.                           |
+-----------------+---------------------+----------------------------------------------------------+
| port            | 514                 | The port for the syslog server.                          |
+-----------------+---------------------+----------------------------------------------------------+
| socktype        | UDP                 | The syslog socket type (TCP or UDP).                     |
+-----------------+---------------------+----------------------------------------------------------+

Basic Example
-------------
The example below is a basic logger middleware using default values as defined in table above.

.. code:: python

    import falcon
    from gh_action_testing.middleware import LoggerMiddleware
    from gh_action_testing.utils import syslog_handler


    class LoggerMiddleWareResource(object):
        """Logger middleware testing resource."""

        def on_get(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            self.log.debug(f'DEBUG {key}')
            self.log.info(f'INFO {key}')
            self.log.warning(f'WARNING {key}')
            self.log.error(f'ERROR {key}')
            self.log.critical(f'CRITICAL {key}')
            resp.text = f'Logged - {key}'

        def on_post(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            value = req.get_param('value')
            self.log.debug(f'DEBUG {key} {value}')
            self.log.info(f'INFO {key} {value}')
            self.log.warning(f'WARNING {key} {value}')
            self.log.error(f'ERROR {key} {value}')
            self.log.critical(f'CRITICAL {key} {value}')
            resp.text = f'Logged - {key}'

    sh = syslog_handler()
    app = falcon.App(middleware=[LoggerMiddleware([sh])])
    app.add_route('/middleware', LoggerMiddleWareResource())

Advanced Example
----------------
The example below shows a heavily customized logger.

.. code:: python

    import falcon
    from gh_action_testing.middleware import LoggerMiddleware
    from gh_action_testing.utils import syslog_handler


    class LoggerMiddleWareResource(object):
        """Logger middleware testing resource."""

        def on_get(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            self.log.debug(f'DEBUG {key}')
            self.log.info(f'INFO {key}')
            self.log.warning(f'WARNING {key}')
            self.log.error(f'ERROR {key}')
            self.log.critical(f'CRITICAL {key}')
            resp.text = f'Logged - {key}'

        def on_post(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            value = req.get_param('value')
            self.log.debug(f'DEBUG {key} {value}')
            self.log.info(f'INFO {key} {value}')
            self.log.warning(f'WARNING {key} {value}')
            self.log.error(f'ERROR {key} {value}')
            self.log.critical(f'CRITICAL {key} {value}')
            resp.text = f'Logged - {key}'

    sh = syslog_handler(
        host='10.10.10.10',
        facility='daemon',
        formatter='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        level='debug',
        name='my-sh',
        port='5140',
        socktype='TCP',
    )
    app = falcon.App(middleware=[LoggerMiddleware(handlers=[sh], level='INFO', name='MY-LOGGER')])
    app.add_route('/middleware', LoggerMiddleWareResource())

------------
Null Handler
------------
This module can be a dependency for other middleware components. If using this module and no handler is required the following example shows how to setup the middleware component with no handlers/null handlers.

.. code:: python

    import falcon
    from gh_action_testing.middleware import LoggerMiddleware


    class LoggerMiddleWareResource(object):
        """Logger middleware testing resource."""

        def on_get(self, req, resp):
            """Support GET method."""
            key = req.get_param('key')
            self.log.debug(f'DEBUG {key}')  # No handler added so this would get dropped on the floor
            resp.text = 'No Logging'

    app = falcon.App(middleware=[LoggerMiddleware()])
    app.add_route('/middleware', LoggerMiddleWareResource())


-----------
Development
-----------

Installation
------------

After cloning the repository, all development requirements can be installed via pip. For linting and code consistency the pre-commit hooks should be installed.

.. code:: bash

    > pip install gh-action-testing[dev]
    > pre-commit install

Testing
-------

Run pytest test cases and get a coverage report.

.. code:: bash

    > pytest --cov=gh_action_testing --cov-report=term-missing tests/

..
    Alternate github build
    .. |build| image:: https://img.shields.io/github/actions/workflow/status/bcsummers/gh-action-testing/main.yml?branch=develop

.. |build| image:: https://github.com/bcsummers/gh-action-testing/workflows/build/badge.svg
    :target: https://github.com/bcsummers/gh-action-testing/actions/workflows/main.yml

.. |code-style| image:: https://img.shields.io/static/v1?logo=black&label=code%20style&message=black&color=black
    :target: https://github.com/python/black
    :alt: poetry

.. |codeql| image:: https://github.com/bcsummers/gh-action-testing/actions/workflows/codeql.yml/badge.svg
    :target: https://github.com/bcsummers/gh-action-testing/actions/workflows/codeql.yml

..
    Alternate coverage
    .. |coverage| image:: https://img.shields.io/codecov/c/github/bcsummers/gh-action-testing?style=flat-square

.. |coverage| image:: https://codecov.io/gh/bcsummers/gh-action-testing/branch/develop/graph/badge.svg?token=VTEEB03ADS
    :target: https://codecov.io/gh/bcsummers/gh-action-testing

.. |dependency-review| image:: https://github.com/bcsummers/gh-action-testing/actions/workflows/dependency-review.yml/badge.svg
    :target: https://github.com/bcsummers/gh-action-testing/actions/workflows/dependency-review.yml

.. |falcon| image:: https://img.shields.io/static/v1?logo=falcon&logoColor=white&label=falcon&message=add-on&color=60a5fa
    :target: https://github.com/falconry/falcon

.. |latest-test| image:: https://api-public.service.runforesight.com/api/v1/badge/test?repoId=92c63588-f38c-412a-96ca-60a56b67d061
    :target: https://www.runforesight.com/
..
    License is displayed on the github page
    .. |license| image:: https://img.shields.io/github/license/bcsummers/gh-action-testing?color=60a5fa
        :target: https://www.apache.org/licenses/LICENSE-2.0

.. |maintainability-percentage| image:: https://img.shields.io/codeclimate/maintainability-percentage/bcsummers/gh-action-testing
    :target: https://codeclimate.com/github/bcsummers/gh-action-testing

.. |poetry| image:: https://img.shields.io/static/v1?logo=poetry&logoColor=white&label=poetry&message=enabled&color=60a5fa
    :target: https://github.com/python-poetry/poetry
    :alt: poetry

.. |pre-commit| image:: https://img.shields.io/static/v1?logo=pre-commit&logoColor=white&label=pre-commit&message=enabled&color=60a5fa
    :target: https://github.com/pre-commit/pre-commit
    :alt: pre-commit

.. |successful-test| image:: https://api-public.service.runforesight.com/api/v1/badge/success?repoId=92c63588-f38c-412a-96ca-60a56b67d061
    :target: https://www.runforesight.com/

.. |utilization| image:: https://api-public.service.runforesight.com/api/v1/badge/utilization?repoId=92c63588-f38c-412a-96ca-60a56b67d061
    :target: https://www.runforesight.com/
