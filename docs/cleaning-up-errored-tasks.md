Error\'d tasks occasionally need to be purged from Redis, so `TaskTiger`
exposes a `purge_errored_tasks` method to help. It might be useful to
set this up as a periodic task as follows:

``` python
from tasktiger import TaskTiger, periodic

tiger = TaskTiger()

@tiger.task(schedule=periodic(hours=1))
def purge_errored_tasks():
    tiger.purge_errored_tasks(
        limit=1000,
        last_execution_before=(
            datetime.datetime.utcnow() - datetime.timedelta(weeks=12)
        )
    )
```

