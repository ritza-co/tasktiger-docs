TaskTiger provides access to the `Task` class which lets you inspect
queues and perform various actions on tasks.

Each queue can have tasks in the following states:

-   `queued`: Tasks that are queued and waiting to be picked up by the
    workers.
-   `active`: Tasks that are currently being processed by the workers.
-   `scheduled`: Tasks that are scheduled for later execution.
-   `error`: Tasks that failed with an error.

To get a list of all tasks for a given queue and state, use
`Task.tasks_from_queue`. The method gives you back a tuple containing
the total number of tasks in the queue (useful if the tasks are
truncated) and a list of tasks in the queue, latest first. Using the
`skip` and `limit` keyword arguments, you can fetch arbitrary slices of
the queue. If you know the task ID, you can fetch a given task using
`Task.from_id`. Both methods let you load tracebacks from failed task
executions using the `load_executions` keyword argument, which accepts
an integer indicating how many executions should be loaded.

Tasks can also be constructed and queued using the regular constructor,
which takes the TaskTiger instance, the function name and the options
described in the *Task options* section. The task can then be queued
using its `delay` method. Note that the `when` argument needs to be
passed to the `delay` method, if applicable. Unique tasks can be
reconstructed using the same arguments.

The `Task` object has the following properties:

-   `id`: The task ID.
-   `data`: The raw data as a dict from Redis.
-   `executions`: A list of failed task executions (as dicts). An
    execution dict contains the processing time in `time_started` and
    `time_failed`, the worker host in `host`, the exception name in
    `exception_name` and the full traceback in `traceback`.
-   `serialized_func`, `args`, `kwargs`: The serialized function name
    with all of its arguments.
-   `func`: The imported (executable) function

The `Task` object has the following methods:

-   `cancel`: Cancel a scheduled task.
-   `delay`: Queue the task for execution.
-   `delete`: Remove the task from the error queue.
-   `execute`: Run the task without queueing it.
-   `n_executions`: Queries and returns the number of past task
    executions.
-   `retry`: Requeue the task from the error queue for execution.
-   `update_scheduled_time`: Updates a scheduled task\'s date to the
    given date.

The current task can be accessed within the task function while it\'s
being executed: In case of a non-batch task, the `current_task` property
of the `TaskTiger` instance returns the current `Task` instance. In case
of a batch task the `current_tasks` property must be used which returns
a list of tasks that are currently being processed (in the same order as
they were passed to the task).

Example 1: Queueing a unique task and canceling it without a reference
to the original task.

``` python
from tasktiger import TaskTiger, Task

tiger = TaskTiger()

# Send an email in five minutes.
task = Task(tiger, send_mail, args=['email_id'], unique=True)
task.delay(when=datetime.timedelta(minutes=5))

# Unique tasks get back a task instance referring to the same task by simply
# creating the same task again.
task = Task(tiger, send_mail, args=['email_id'], unique=True)
task.cancel()
```


Example 2: Inspecting queues and retrying a task by ID.

``` python
from tasktiger import TaskTiger, Task

QUEUE_NAME = 'default'
TASK_STATE = 'error'
TASK_ID = '6fa07a91642363593cddef7a9e0c70ae3480921231710aa7648b467e637baa79'

tiger = TaskTiger()

n_total, tasks = Task.tasks_from_queue(tiger, QUEUE_NAME, TASK_STATE)

for task in tasks:
    print(task.id, task.func)

task = Task.from_id(tiger, QUEUE_NAME, TASK_STATE, TASK_ID)
task.retry()
```

Example 3: Accessing the task instances within a batch task function to
determine how many times the currently processing tasks were previously
executed.

``` python
from tasktiger import TaskTiger

tiger = TaskTiger()

@tiger.task(batch=True)
def my_task(args):
    for task in tiger.current_tasks:
        print(task.n_executions())
```

