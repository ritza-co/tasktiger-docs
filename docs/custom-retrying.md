In some cases the task retry options may not be flexible enough. For
example, you might want to use a different retry method depending on the
exception type, or you might want to like to suppress logging an error
if a task fails after retries. In these cases, `RetryException` can be
raised within the task function. The following options are supported:

-   `method`

    Specify a custom retry method for this retry. If not given, the
    task\'s default retry method is used, or, if unspecified, the
    configured `DEFAULT_RETRY_METHOD`. Note that the number of retries
    passed to the retry method is always the total number of times this
    method has been executed, regardless of which retry method was used.

-   `original_traceback`

    If `RetryException` is raised from within an except block and
    `original_traceback` is True, the original traceback will be logged
    (i.e. the stacktrace at the place where the caught exception was
    raised). False by default.

-   `log_error`

    If set to False and the task fails permanently, a warning will be
    logged instead of an error, and the task will be removed from Redis
    when it completes. True by default.

Example usage:

``` python
from tasktiger.exceptions import RetryException
from tasktiger.retry import exponential, fixed

def my_task():
    if not ready():
        # Retry every minute up to 3 times if we're not ready. An error will
        # be logged if we're out of retries.
        raise RetryException(method=fixed(60, 3))

    try:
        some_code()
    except NetworkException:
        # Back off exponentially up to 5 times in case of a network failure.
        # Log the original traceback (as a warning) and don't log an error if
        # we still fail after 5 times.
        raise RetryException(method=exponential(60, 2, 5),
                             original_traceback=True,
                             log_error=False)
```
