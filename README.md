# Pragmatic Patterns

## Index

1. [Defer](#defer)
1. [Lightweight unchecked "any"](#any)

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

## Any

```c++
class MyAny
{
public:
    MyAny()
        : placeHolder_(nullptr)
    {}

    ~MyAny()
    {
        delete placeHolder_;
    }

    template<typename T> void set(const T& data)
    {
        if (placeHolder_ != nullptr)
        {
            static_cast<Holder<T>*>(placeHolder_)->setData(data);
        }
        else
        {
            placeHolder_ = new Holder<T>(data);
        }
    }

    template<typename T> T get() const
    {
        return static_cast<Holder<T>*>(placeHolder_)->getData();
    }

private:
    class PlaceHolder
    {
    };

    template<class T>
    class Holder : public PlaceHolder
    {
    public:
        Holder(const T& data)
            : data_(data)
        {}

        void setData(const T& data)
        {
            data_ = data;
        }

        T getData() const
        {
            return data_;
        }

    private:
        T data_;
    };

    PlaceHolder* placeHolder_;
};

int main()
{
    MyAny myData;
    myData.set<std::string>("ciao");
    std::cout << myData.get<std::string>() << std::endl;   // prints "ciao"
    myData.set<int>(10);
    std::cout << myData.get<int>() << std::endl;           // prints "10"
    std::cout << myData.get<double>() << std::endl;        // dangerous!! Prints garbage
    return 0;
}
```
