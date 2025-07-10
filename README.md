# GDExtension

This is a living document to document everything I discover regarding GDExtension that wasn't immediately clear from the available documentation.

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
// Objects, eg Material
PropertyInfo(
  Variant::ARRAY,
  "materials",
  PROPERTY_HINT_ARRAY_TYPE,
  vformat(
    "%s/%s:%s", Variant::OBJECT, PROPERTY_HINT_RESOURCE_TYPE, "Material"
  )
)

// Variants, eg Vector2i
PropertyInfo(
  Variant::ARRAY,
  "vectors" // choose yourself
  PROPERTY_HINT_ARRAY_TYPE,
  vformat("%s", Variant::VECTOR2I)
)
```

### Type Dictionary

```cpp
PropertyInfo(
  Variant::DICTIONARY,
  "some_dictionary", // choose yourself
  PROPERTY_HINT_DICTIONARY_TYPE,
  vformat(
    "%s;%s", Variant::STRING_NAME, Variant::INT // choose yourself
  )
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

## Helpful macros

### Creating getters/setters

To be used in class definition. Creates basic `set_<var_name>` and `get_<var_name>`. Careful not to create conflicts.

```cpp
#define DECLARE_PROPERTY_GETSET(type, name) \
  void set_##name(type value) { \
    name = value; \
  } \
  type get_##name() const { \
    return name; \
  }
```

### Binding properties

To be used inside `SomeClass::_bind_methods()`:

```cpp
#define BIND_PROPERTY(type, name, hint_type, hint_string, usage, class_name) \
  ClassDB::bind_method(D_METHOD("set_" #name, #name), &self_type::set_##name); \
  ClassDB::bind_method(D_METHOD("get_" #name), &self_type::get_##name); \
  ADD_PROPERTY( \
    PropertyInfo(type, #name, hint_type, hint_string, usage, class_name), \
    "set_" #name, \
    "get_" #name \
  );

// Simpler version exposing hints
#define BIND_PROPERTY_WITH_HINT(type, name, hint_type, hint_string) \
  BIND_PROPERTY(type, name, hint_type, hint_string, PROPERTY_USAGE_DEFAULT, "")

// Simplest version, for basic types
#define BIND_PROPERTY_SIMPLE(type, name) \
  BIND_PROPERTY(type, name, PROPERTY_HINT_NONE, "", PROPERTY_USAGE_DEFAULT, "")
```
