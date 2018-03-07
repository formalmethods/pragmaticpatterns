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
