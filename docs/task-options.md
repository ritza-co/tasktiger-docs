## Task options

Tasks support a variety of options that can be specified either in the
task decorator, or when queueing a task. For the latter, the `delay`
method must be called on the `TaskTiger` object, and any options in the
task decorator are overridden.

``` python
@tiger.task(queue='myqueue', unique=True)
def my_task():
    print('Hello')
```

``` python
# The task will be queued in "otherqueue", even though the task decorator
# says "myqueue".
tiger.delay(my_task, queue='otherqueue')
```

When queueing a task, the task needs to be defined in a module other
than the Python file which is being executed. In other words, the task
can\'t be in the `__main__` module. TaskTiger will give you back an
error otherwise.

The following options are supported by both `delay` and the task
decorator:

-   `queue`

    Name of the queue where the task will be queued.

-   `hard_timeout`

    If the task runs longer than the given number of seconds, it will be
    killed and marked as failed.

-   `unique`

    Boolean to indicate whether the task will only be queued if there is
    no similar task with the same function, arguments, and keyword
    arguments in the queue. Note that multiple similar tasks may still
    be executed at the same time since the task will still be inserted
    into the queue if another one is being processed. Requeueing an
    already scheduled unique task will not change the time it was
    originally scheduled to execute at.

-   `unique_key`

    If set, this implies `unique=True` and specifies the list of kwargs
    to use to construct the unique key. By default, all args and kwargs
    are serialized and hashed.

-   `lock`

    Boolean to indicate whether to hold a lock while the task is being
    executed (for the given args and kwargs). If a task with similar
    args/kwargs is queued and tries to acquire the lock, it will be
    retried later.

-   `lock_key`

    If set, this implies `lock=True` and specifies the list of kwargs to
    use to construct the lock key. By default, all args and kwargs are
    serialized and hashed.

-   `max_queue_size`

    A maximum queue size can be enforced by setting this to an integer
    value. The `QueueFullException` exception will be raised when
    queuing a task if this limit is reached. Tasks in the `active`,
    `scheduled`, and `queued` states are counted against this limit.

-   `when`

    Takes either a datetime (for an absolute date) or a timedelta
    (relative to now). If given, the task will be scheduled for the
    given time.

-   `retry`

    Boolean to indicate whether to retry the task when it fails (either
    because of an exception or because of a timeout). To restrict the
    list of failures, use `retry_on`. Unless `retry_method` is given,
    the configured `DEFAULT_RETRY_METHOD` is used.

-   `retry_on`

    If a list is given, it implies `retry=True`. The task will be only
    retried on the given exceptions (or its subclasses). To retry the
    task when a hard timeout occurs, use `JobTimeoutException`.

-   `retry_method`

    If given, implies `retry=True`. Pass either:

    -   a function that takes the retry number as an argument, or,
    -   a tuple `(f, args)`, where `f` takes the retry number as the
        first argument, followed by the additional args.

    The function needs to return the desired retry interval in seconds,
    or raise `StopRetry` to stop retrying. The following built-in
    functions can be passed for common scenarios and return the
    appropriate tuple:

    -   `fixed(delay, max_retries)`

        Returns a method that returns the given `delay` (in seconds) or
        raises `StopRetry` if the number of retries exceeds
        `max_retries`.

    -   `linear(delay, increment, max_retries)`

        Like `fixed`, but starts off with the given `delay` and
        increments it by the given `increment` after every retry.

    -   `exponential(delay, factor, max_retries)`

        Like `fixed`, but starts off with the given `delay` and
        multiplies it by the given `factor` after every retry.

    For example, to retry a task 3 times (for a total of 4 executions),
    and wait 60 seconds between executions, pass
    `retry_method=fixed(60, 3)`.

-   `runner_class`

    If given, a Python class can be specified to influence task running
    behavior. The runner class should inherit
    `tasktiger.runner.BaseRunner` and implement the task execution
    behavior. The default implementation is available in
    `tasktiger.runner.DefaultRunner`. The following behavior can be
    achieved:

    -   Execute specific code before or after the task is executed (in
        the forked child process), or customize the way task functions
        are called in either single or batch processing.

        Note that if you want to execute specific code for all tasks,
        you should use the `CHILD_CONTEXT_MANAGERS` configuration
        option.

    -   Control the hard timeout behavior of a task.

    -   Execute specific code in the main worker process after a task
        failed permanently.

    This is an advanced feature and the interface and requirements of
    the runner class can change in future TaskTiger versions.

The following options can be only specified in the task decorator:

-   `batch`

    If set to `True`, the task will receive a list of dicts with args
    and kwargs and can process multiple tasks of the same type at once.
    Example:
    `[{"args": [1], "kwargs": {}}, {"args": [2], "kwargs": {}}]` Note
    that the list will only contain multiple items if the worker has set
    up `BATCH_QUEUES` for the specific queue (see the *Configuration*
    section).

-   `schedule`

    If given, makes a task execute periodically. Pass either:

    -   a function that takes the current datetime as an argument.
    -   a tuple `(f, args)`, where `f` takes the current datetime as the
        first argument, followed by the additional args.

    The schedule function must return the next task execution datetime,
    or `None` to prevent periodic execution. The function is executed to
    determine the initial task execution date when a worker is
    initialized, and to determine the next execution date when the task
    is about to get executed.

    For most common scenarios, the `periodic` built-in function can be
    passed:

    -   `periodic(seconds=0, minutes=0, hours=0, days=0, weeks=0, start_date=None, end_date=None)`

        Use equal, periodic intervals, starting from `start_date`
        (defaults to `2000-01-01T00:00Z`, a Saturday, if not given),
        ending at `end_date` (or never, if not given). For example, to
        run a task every five minutes indefinitely, use
        `schedule=periodic(minutes=5)`. To run a task every every Sunday
        at 4am UTC, you could use
        `schedule=periodic(weeks=1, start_date=datetime.datetime(2000, 1, 2, 4))`.

