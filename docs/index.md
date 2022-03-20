# TaskTiger

[![image](https://circleci.com/gh/closeio/tasktiger/tree/master.svg?style=svg&circle-token=a86617952aa9b4cfee784b6ac43358cd042a6672)](https://circleci.com/gh/closeio/tasktiger/tree/master)

*TaskTiger* is a Python task queue using Redis.

(Interested in working on projects like this? [Close](http://close.com)
is looking for [great engineers](http://jobs.close.com) to join our
team)

## Features

-   Per-task fork

    TaskTiger forks a subprocess for each task, This comes with several
    benefits: Memory leaks caused by tasks are avoided since the
    subprocess is terminated when the task is finished. A hard time
    limit can be set for each task, after which the task is killed if it
    hasn\'t completed. To ensure performance, any necessary Python
    modules can be preloaded in the parent process.

-   Unique queues

    TaskTiger has the option to avoid duplicate tasks in the task queue.
    In some cases it is desirable to combine multiple similar tasks. For
    example, imagine a task that indexes objects (e.g. to make them
    searchable). If an object is already present in the task queue and
    hasn\'t been processed yet, a unique queue will ensure that the
    indexing task doesn\'t have to do duplicate work. However, if the
    task is already running while it\'s queued, the task will be
    executed another time to ensure that the indexing task always picks
    up the latest state.

-   Task locks

    TaskTiger can ensure to never execute more than one instance of
    tasks with similar arguments by acquiring a lock. If a task hits a
    lock, it is requeued and scheduled for later executions after a
    configurable interval.

-   Task retrying

    TaskTiger lets you retry exceptions (all exceptions or a list of
    specific ones) and comes with configurable retry intervals (fixed,
    linear, exponential, custom).

-   Flexible queues

    Tasks can be easily queued in separate queues. Workers pick tasks
    from a randomly chosen queue and can be configured to only process
    specific queues, ensuring that all queues are processed equally.
    TaskTiger also supports subqueues which are separated by a period.
    For example, you can have per-customer queues in the form
    `process_emails.CUSTOMER_ID` and start a worker to process
    `process_emails` and any of its subqueues. Since tasks are picked
    from a random queue, all customers get equal treatment: If one
    customer is queueing many tasks it can\'t block other customers\'
    tasks from being processed. A maximum queue size can also be
    enforced.

-   Batch queues

    Batch queues can be used to combine multiple queued tasks into one.
    That way, your task function can process multiple sets of arguments
    at the same time, which can improve performance. The batch size is
    configurable.

-   Scheduled and periodic tasks

    Tasks can be scheduled for execution at a specific time. Tasks can
    also be executed periodically (e.g. every five seconds).

-   Structured logging

    TaskTiger supports JSON-style logging via structlog, allowing more
    flexibility for tools to analyze the log. For example, you can use
    TaskTiger together with Logstash, Elasticsearch, and Kibana.

    The structlog processor `tasktiger.logging.tasktiger_processor` can
    be used to inject the current task id into all log messages.

-   Reliability

    TaskTiger atomically moves tasks between queue states, and will
    re-execute tasks after a timeout if a worker crashes.

-   Error handling

    If an exception occurs during task execution and the task is not set
    up to be retried, TaskTiger stores the execution tracebacks in an
    error queue. The task can then be retried or deleted manually.
    TaskTiger can be easily integrated with error reporting services
    like Rollbar.

-   Admin interface

    A simple admin interface using flask-admin exists as a separate
    project
    ([tasktiger-admin](https://github.com/closeio/tasktiger-admin)).


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

