1.  Make sure the code has been thoroughly reviewed and tested in a
    realistic production environment.
2.  Update `setup.py` and `CHANGELOG.md`. Make sure you include any
    breaking changes.
3.  Run `python setup.py sdist` and
    `twine upload dist/<PACKAGE_TO_UPLOAD>`.
4.  Push a new tag pointing to the released commit, format: `v0.13` for
    example.
5.  Mark the tag as a release in GitHub\'s UI and include in the
    description the changelog entry for the version. An example would
    be: <https://github.com/closeio/tasktiger/releases/tag/v0.13>.
