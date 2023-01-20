# Simple Pytest Example Writeup

See https://www.youtube.com/watch?v=YbpKMIUjvK8

* [Pytest Documentation](https://docs.pytest.org/en/6.2.x/getting-started.html#getstarted)
* [unittest.mock Documentation](https://docs.python.org/3/library/unittest.mock.html)

I'm following the tutorial in the above YouTube video and am writing directions here for future use.

Documentation:
(PyTest)[https://docs.pytest.org/en/7.2.x/]
(Mock)[https://docs.python.org/3/library/unittest.mock.html]

## Installation

Pretty straightforward, but make sure you have pytest available: `pip install pytest`

## Running pytest

Just run `pytest` in the directory your test files are in. It will automatically search for files that begin or end in "test," then run functions in those files that begin or end in "test." NOTE: I assume in subdirectories as well?

### Running specific files or functions

It could be that your tests take some time, and you really just want to check that a specific file or function in a file runs appropriately. To do a specific file, indicate the file:

`pytest test_shopping_cart.py`

Or a specific function in a file, indicate with double colons:

`pytest test_shopping_cart.py::test_when_item_added_then_cart_contains_item`

## Testing A Specific Raised Error

Normally, your functions pass or fail depending on a simple `assert` statement. However, sometimes you want to assert that a specific error is thrown. In that case, you can encapsulate the code that will throw the expected error in a `with pytest.raises(<Error Type>):`
block.

### Ensure Hard Prints

While debugging, you may want to have print statements in the test functions to appear on the screen. In that case use the `-s` flag.

## Pytest Fixtures - reducing duplicate code

You will often need to reuse the same code to provide context for each of your tests - in this example, making a cart and adding some initial items. You can do this with fixtures. To make a fixture, add the following to a function that returns the object you are initializing:

`@pytest.fixture`

Then to use it in a test function, you will need to pass the fixture function name to the test function, like in the following example.

```
@pytest.fixture
def cart():
    return ShoppingCart(5)

def test_can_add_item_to_cart(cart):
    cart.add("apple")
    assert cart.size() == 1
```

## Mocking Dependencies

Sometimes you will need to reliably and consistently run a test that has a dependency that is prone to change. Perhaps the dependency is in development or is regularly updated. This can make testing tricky, as you want the test to be relatively static (does it work or not when provided an expected input?). This is where mocking comes into play.

The Mock class lets you basically put in a dummy return value or function into a dependency instance. That way, you can test function with a static dependency.

In the encoded example, let's say that a class - ItemDatabase - fluctuates a bit, as it represents the prices of grocery items today. Prices can fluctuate regularly. Sometimes apples are $1, sometimes they're $2. Oranges can similarly fluctuate. You want to test that your function can appropriately count up the cost of your cart. In that sense, you may want to pretend that the prices are set in stone for the sake of testing. If an apple in $1 and an orange is $2, does putting an apple and an orange into your cart add up to $3? To do so, you may set the "get" function of an ItemDatabase instance to a dummy function that always returns that apples are $1 and oranges are $2, as in the following:

```
def test_can_get_total_price(cart):
    cart.add("apple")
    cart.add("orange")
    item_database = ItemDatabase()

    def mock_get_item(item: str):
        if item == "apple":
            return 1.0
        if item == "orange":
            return 2.0

    item_database.get = Mock(side_effect=mock_get_item)
    assert cart.get_total_price(item_database) == 3.0
```

`return_value` will return a static variable, if that's what you need for your tests.
