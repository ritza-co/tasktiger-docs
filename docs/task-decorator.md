TaskTiger provides a task decorator to specify task options. Note that
simple tasks don\'t need to be decorated. However, decorating the task
allows you to use an alternative syntax to queue the task, which is
compatible with Celery:

``` python
# tasks.py

import tasktiger
tiger = tasktiger.TaskTiger()

@tiger.task()
def my_task(name, n=None):
    print('Hello', name)
```

``` python
In [1]: import tasks
# The following are equivalent. However, the second syntax can only be used
# if the task is decorated.
In [2]: tasks.tiger.delay(my_task, args=('John',), kwargs={'n': 1})
In [3]: tasks.my_task.delay('John', n=1)
```
