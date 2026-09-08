# Testing Rules

- Use `unittest` exclusively — do NOT use pytest.
- All test classes must extend `unittest.TestCase`.
- Test files must be named `test_<module_name>.py`.
- Test classes must be named `Test<ClassName>`.
- Each test method must test exactly one behavior.
- Use `setUp` and `tearDown` for shared fixtures.
- Mock external dependencies with `unittest.mock`.
- Minimum coverage target: 80% per module.
- Run tests with: `python -m unittest discover -s tests`
