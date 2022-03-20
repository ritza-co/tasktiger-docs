The `--max-workers-per-queue` option uses queue locks to control the
number of workers that can simultaneously process the same queue. When
using this option a system lock can be placed on a queue which will keep
workers from processing tasks from that queue until it expires. Use the
`set_queue_system_lock()` method of the TaskTiger object to set this
lock.
