TaskTiger comes with Rollbar integration for error handling. When a task
errors out, it can be logged to Rollbar, grouped by queue, task function
name and exception type. To enable logging, initialize rollbar with the
`StructlogRollbarHandler` provided in the `tasktiger.rollbar` module.
The handler takes a string as an argument which is used to prefix all
the messages reported to Rollbar. Here is a custom worker launch script:

``` python
import logging
import rollbar
import sys
from tasktiger import TaskTiger
from tasktiger.rollbar import StructlogRollbarHandler

tiger = TaskTiger(setup_structlog=True)

rollbar.init(ROLLBAR_API_KEY, APPLICATION_ENVIRONMENT,
             allow_logging_basic_config=False)
rollbar_handler = StructlogRollbarHandler('TaskTiger')
rollbar_handler.setLevel(logging.ERROR)
tiger.log.addHandler(rollbar_handler)

tiger.run_worker_with_args(sys.argv[1:])
```

