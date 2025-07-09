# GDExtension

```cpp
// class.h

#pragma once

#include "godot_cpp/classes/node.hpp"
// or anything

using namespace godot;

class ClassName : public Node {
  GDCLASS(ClassName, Node);

private:
  // variables or internal methods that won't be exposed to classes extended from this one.
  // generally I don't use these

protected:
  static void _bind_methods();
  // ...variables
  // ...internal helper methods

public:
  // ...exposed methods
}
```

```cpp
// class.cpp

#include "class.h"

void ClassName::_bind_methods() {
  // See below
}

ClassName::ClassName() {
  // initialize any variables here
}

ClassName::~ClassName() {
  // deinitialize anything here
}
```

## `_bind_methods()`

```cpp
// class.cpp

void ClassName::_bind_methods)_ {
  // Add a method:
  ClassDB::bind_method(
    D_METHOD("method_name", "variable_name_1", "variable_name_2"),
    &ClassName::method_name,
    DEFVAL(false) // optionally, add default values. Repeat this macro for more.
  );

  // Add a property:
  ClassDB::bind_method(
    D_METHOD("set_property_name", "value"),
    &ClassName::set_property_name
  );
  ClassDB::bind_method(
    D_METHOD("get_property_name"),
    &ClassName::get_property_name
  );
  ADD_PROPERTY(
    PropertyInfo(
      Variant::INT,
      "property_name",
      PROPERTY_HINT_RANGE, // optional, a PropertyHint
      "0,255,1", // optional, text for the hint above
      PROPERTY_USAGE_DEFAULT // optional, a PropertyUsageFlag
    )
  );

  // Add a signal:
  ADD_SIGNAL(
    MethodInfo(
      "signal_name",
      PropertyInfo(
        Variant::INT,
        "int_argument_name" // choose yourself
      ),
      PropertyInfo(
        Variant::OBJECT,
        "object_argument_name", // choose yourself
        PROPERTY_HINT_NODE_TYPE, // for example
        "MeshInstance3D"
      )
      // Use any number of arguments with PropertyInfos
    )
  );
}
```

See:
- [PropertyHint](https://docs.godotengine.org/en/stable/classes/class_%40globalscope.html#enum-globalscope-propertyhint)
- [PropertyUsageFlag](https://docs.godotengine.org/en/stable/classes/class_%40globalscope.html#enum-globalscope-propertyusageflags)

## `PropertyInfo`

Describes properties, also for signals.

### Typed Array

```cpp
PropertyInfo(
  Variant::ARRAY,
  "materials",
  PROPERTY_HINT_ARRAY_TYPE,
  vformat(
    "%s/%s:%s", Variant::OBJECT, PROPERTY_HINT_RESOURCE_TYPE, "Material"
  ),
  PROPERTY_USAGE_DEFAULT
)
```

## Initializing

### Classes extended from `RefCounted`

```cpp
Ref<RefCounted> object;
object.instantiate();
```

### Classes extended from `Node`

```cpp
Node *node;
node = memnew(Node);
```

> Nodes automatically free children upon being freed

## enums

```cpp
// class.h

class ClassName : public Node {
  public:
    enum SOME_ENUM { ENUM_VAL0, ENUM_VAL1 };
};

VARIANT_ENUM_CAST(ClassName::SOME_ENUM);
```

```cpp
// class.cpp

void ClassName::_bind_methods() {
  BIND_ENUM_CONSTANT(ENUM_VAL0);
  BIND_ENUM_CONSTANT(ENUM_VAL1);
}
```
