# Pragmatic Patterns

## Index

1. [Defer](#defer)
1. [Split big class into federation of singletons](#split-big-class-into-federation-of-singletons)

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

## Split big class into federation of singletons

We have programmed/inherited a class with many methods (> 50), whose size and organization
is becoming unmanageable. This may happen quite often in case of a Visitor:

```c++
class MyVisitor {
public:
  MyVisitor(...);
  virutal ~MyVisitor(...);

  visit(Alpha& obj);
  visit(Beta& obj);
  ...
  visit(Iota& obj);

private:
  void utility1(...);
  void utility2(...);
  ...
  void utilityN(...);

  ... data1;
  ... data2;
  ...
  ... dataM;
};
```

We can't do much about the "visit" methods, but we can do something about the "utilityX"
methods, by grouping them into separate new classes:

```c++
  void utility1(...);   //
  ...                   // Group1
  void utility9(...);   //

  void utility10(...);  //
  ...                   // Group2
  void utility19(...);  //

  ...
```

However there might be some dependencies between the groups, some of which might even
be circular dependencies, and might:

```c++
  void MyVisitor::utility1(...) { // Group1
    ...
    utility12(); // Calls routine of Group2
    data1 = ...;
    ...
  }

  void MyVisitor::utility18(...) { // Group2
    ...
    utility2(); // Calls routine of Group1
    data4 = ...;
    ...
  }
```

Solution: create new classes for containing methods of each group. To circumvent circular
dependencies we use the singleton pattern:

```c++
  class Group1 {
  public:
    static Group1* getInstance();
    static void buildInstance(...); // Initializes local data acquired from MyVisitor
    static void disposeInstance();

    void utility1(); // public utilities offered by the group
    ...

  private:
    void utility5(); // private utilities, used by this group only
    ...

    ... data1; // data used by the group, a subset of the one in MyVisitor
    ...
  };
```

In the visitor groups are allocated at construction and disposed at destruction:

```c++
  MyVisitor::MyVisitor() {
    Group1::buildInstance(...);
    Group2::buildInstance(...);
    ...
  }

  MyVisitor::~MyVisitor() {
    ...
    Group2::disposeInstance();
    Group1::disposeInstance();
  }
```

Methods calls will internally retrieve instances for calling methods of other groups:

```c++
  void MyVisitor::utility1(...) { // Group1
    ...
    auto group2 = Group2::getInstance();
    group2.utility12(); // Calls routine of Group2
    data1 = ...;
    ...
  }

  void MyVisitor::utility18(...) { // Group2
    ...
    auto group1 = Group1::getInstance();
    group1.utility2(); // Calls routine of Group1
    data4 = ...;
    ...
  }
```

