It is easy to get started with TaskTiger.

Create a file that contains the task(s).

``` python
# tasks.py
def my_task():
    print('Hello')
```

Queue the task using the `delay` method.

``` python
In [1]: import tasktiger, tasks
In [2]: tiger = tasktiger.TaskTiger()
In [3]: tiger.delay(tasks.my_task)
```

Run a worker (make sure the task code can be found, e.g. using
`PYTHONPATH`).

``` bash
% PYTHONPATH=. tasktiger
{"timestamp": "2015-08-27T21:00:09.135344Z", "queues": null, "pid": 69840, "event": "ready", "level": "info"}
{"task_id": "6fa07a91642363593cddef7a9e0c70ae3480921231710aa7648b467e637baa79", "level": "debug", "timestamp": "2015-08-27T21:03:56.727051Z", "pid": 69840, "queue": "default", "child_pid": 70171, "event": "processing"}
Hello
{"task_id": "6fa07a91642363593cddef7a9e0c70ae3480921231710aa7648b467e637baa79", "level": "debug", "timestamp": "2015-08-27T21:03:56.732457Z", "pid": 69840, "queue": "default", "event": "done"}
```
