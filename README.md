# Pragmatic Patterns

## Defer

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

void bar() {
  throw std::exception();
}

void foo() {
  Defer defer([] () {
    std::cout << "Exiting" << std::endl;
  });
  
  ...
  if (...) {
    return; // prints "Exiting"
  }
  ...
  if (...) {
    bar(); // prints "Exiting"
  }
  ...
  
  // prints "Exiting"
}
```
