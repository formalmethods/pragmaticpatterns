# Pragmatic Patterns

# Defer

## Problem

Create a way to execute some code when leaving a function, including cases where exceptions are triggered.

## Examples

1. You have opened a file handler and you want to make sure it gets closed.

```c++
void foo() {
}
```

## Solution

```c++
class Defer {
public:
  Defer(std::function<void()> callback)
    : callback_(callback)
  {}
  
  ~Defer() {
    callback_();
  }
  
private:
  std::function<void()> callback_;
};

void foo() {
  Defer defer([] () {
    std::cout << "Exiting" << std::endl;
  });
  
  ...
  if (...) {
    // prints "Exiting"
    return;
  }
  ...
  functionThatThrowsAnException(); // prints "Exiting"
  ...
  
  // prints "Exiting"
}
```
