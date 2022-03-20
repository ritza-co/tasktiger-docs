A `TaskTiger` object keeps track of TaskTiger\'s settings and is used to
decorate and queue tasks. The constructor takes the following arguments:

-   `connection`

    Redis connection object. The connection should be initialized with
    `decode_responses=True` to avoid encoding problems on Python 3.

-   `config`

    Dict with config options. Most configuration options don\'t need to
    be changed, and a full list can be seen within `TaskTiger`\'s
    `__init__` method.

    Here are a few commonly used options:

    -   `ALWAYS_EAGER`

        If set to `True`, all tasks except future tasks (`when` is a
        future time) will be executed locally by blocking until the task
        returns. This is useful for testing purposes.

    -   `BATCH_QUEUES`

        Set up queues that will be processed in batch, i.e. multiple
        jobs are taken out of the queue at the same time and passed as a
        list to the worker method. Takes a dict where the key represents
        the queue name and the value represents the batch size. Note
        that the task needs to be declared as `batch=True`. Also note
        that any subqueues will be automatically treated as batch
        queues, and the batch value of the most specific subqueue name
        takes precedence.

    -   `ONLY_QUEUES`

        If set to a non-empty list of queue names, a worker only
        processes the given queues (and their subqueues), unless
        explicit queues are passed to the command line.

-   `setup_structlog`

    If set to True, sets up structured logging using `structlog` when
    initializing TaskTiger. This makes writing custom worker scripts
    easier since it doesn\'t require the user to set up `structlog` in
    advance.

Example:

``` python
import tasktiger
from redis import Redis
conn = Redis(db=1, decode_responses=True)
tiger = tasktiger.TaskTiger(connection=conn, config={
    'BATCH_QUEUES': {
        # Batch up to 50 tasks that are queued in the my_batch_queue or any
        # of its subqueues, except for the send_email subqueue which only
        # processes up to 10 tasks at a time.
        'my_batch_queue': 50,
        'my_batch_queue.send_email': 10,
    },
})
```
