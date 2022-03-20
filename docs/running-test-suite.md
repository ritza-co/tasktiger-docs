Tests can be run locally using the provided docker compose file. After
installing docker, tests should be runnable with:

> docker-compose run \--rm tasktiger pytest

Tests can be more granularly run using normal pytest flags. For example:

> docker-compose run \--rm tasktiger pytest tests/test_base.py::TestCase
