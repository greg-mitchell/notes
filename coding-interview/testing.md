# Testing

## Unit Tests

Many coding interviews like to see basic unit testing.

```python
import unittest

class TestFoo(unittest.TestCase):
    def test_condition_one(self):
        # code under test
        actual = (lambda: 'a')()
        self.assertEqual(actual, 'a')

if __name__ == '__main__':
    unittest.main()
```
