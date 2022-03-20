The `tasktiger` command is used on the command line to invoke a worker.
To invoke multiple workers, multiple instances need to be started. This
can be easily done e.g. via Supervisor. The following Supervisor
configuration file can be placed in `/etc/supervisor/tasktiger.ini` and
runs 4 TaskTiger workers as the `ubuntu` user. For more information,
read Supervisor\'s documentation.

``` bash
[program:tasktiger]
command=/usr/local/bin/tasktiger
process_name=%(program_name)s_%(process_num)02d
numprocs=4
numprocs_start=0
priority=999
autostart=true
autorestart=true
startsecs=10
startretries=3
exitcodes=0,2
stopsignal=TERM
stopwaitsecs=600
killasgroup=false
user=ubuntu
redirect_stderr=false
stdout_logfile=/var/log/tasktiger.out.log
stdout_logfile_maxbytes=250MB
stdout_logfile_backups=10
stderr_logfile=/var/log/tasktiger.err.log
stderr_logfile_maxbytes=250MB
stderr_logfile_backups=10
```

Workers support the following options:

-   `-q`, `--queues`

    If specified, only the given queue(s) are processed. Multiple queues
    can be separated by comma. Any subqueues of the given queues will be
    also processed. For example, `-q first,second` will process items
    from `first`, `second`, and subqueues such as `first.CUSTOMER1`,
    `first.CUSTOMER2`.

-   `-e`, `--exclude-queues`

    If specified, exclude the given queue(s) from processing. Multiple
    queues can be separated by comma. Any subqueues of the given queues
    will also be excluded unless a more specific queue is specified with
    the `-q` option. For example,
    `-q email,email.incoming.CUSTOMER1 -e email.incoming` will process
    items from the `email` queue and subqueues like
    `email.outgoing.CUSTOMER1` or `email.incoming.CUSTOMER1`, but not
    `email.incoming` or `email.incoming.CUSTOMER2`.

-   `-m`, `--module`

    Module(s) to import when launching the worker. This improves task
    performance since the module doesn\'t have to be reimported every
    time a task is forked. Multiple modules can be separated by comma.

    Another way to preload modules is to set up a custom TaskTiger
    launch script, which is described below.

-   `-h`, `--host`

    Redis server hostname (if different from `localhost`).

-   `-p`, `--port`

    Redis server port (if different from `6379`).

-   `-a`, `--password`

    Redis server password (if required).

-   `-n`, `--db`

    Redis server database number (if different from `0`).

-   `-M`, `--max-workers-per-queue`

    Maximum number of workers that are allowed to process a queue.

-   `--store-tracebacks/--no-store-tracebacks`

    Store tracebacks with execution history (config defaults to `True`).

In some cases it is convenient to have a custom TaskTiger launch script.
For example, your application may have a `manage.py` command that sets
up the environment and you may want to launch TaskTiger workers using
that script. To do that, you can use the `run_worker_with_args` method,
which launches a TaskTiger worker and parses any command line arguments.
Here is an example:

``` python
import sys
from tasktiger import TaskTiger

try:
    command = sys.argv[1]
except IndexError:
    command = None

if command == 'tasktiger':
    tiger = TaskTiger(setup_structlog=True)
    # Strip the "tasktiger" arg when running via manage, so we can run e.g.
    # ./manage.py tasktiger --help
    tiger.run_worker_with_args(sys.argv[2:])
    sys.exit(0)
```

If you\'re using `flask-script`, you can use the `TaskTigerCommand`
provided in the `tasktiger.flask_script` module. It takes the
`TaskTiger` instance as an argument. Tasks will have access to Flask\'s
application context. Example:

``` python
from flask import Flask
from flask.ext.script import Manager
from tasktiger.flask_script import TaskTigerCommand

app = Flask()
manager = Manager(app)
tiger = TaskTiger(setup_structlog=True)

manager.add_command('tasktiger', TaskTigerCommand(tiger))

if __name__ == "__main__":
    manager.run()
```

You can subclass the `TaskTigerCommand` and override the `setup` method
to implement any custom setup that needs to be done before running the
worker.
