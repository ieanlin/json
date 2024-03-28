# JSON_USE_IMPLICIT_CONSTRUCTORS

```cpp
#define JSON_USE_IMPLICIT_CONSTRUCTORS /* value */
```

When defined to `0`, implicit constructors are switched off. By default, implicit constructors are switched on. The
value directly affects [the constructor of `basic_json`](../basic_json/basic_json.md).

## Default definition

By default, implicit constructors are enabled.

```cpp
#define JSON_USE_IMPLICIT_CONSTRUCTORS 1
```

## Notes

!!! hint "CMake option"

    Implicit constructors can also be controlled with the CMake option
    [`JSON_ImplicitConstructors`](../../integration/cmake.md#json_legacydiscardedvaluecomparison)
    (`ON` by default) which defines `JSON_USE_IMPLICIT_CONSTRUCTORS` accordingly.

## Examples

??? example

    This is an example for an implicit construction:

    ```cpp
    json j = "Hello, world!";
    ```

    When `JSON_USE_IMPLICIT_CONSTRUCTORS` is defined to `0`, the code above no longer compiles. Instead, it must be
    written like this:

    ```cpp
    json j = json("Hello, world!");
    ```

??? example

    Disabling constructors is particularly useful to avoid implicit allocations that can lead to invalid memory access:

    ```cpp
    const json & valueAt(const json & map, const std::string & key) {
        return map[key];
    }
    const json::object_t & getObjectRef(const json & obj) {
        return obj.get_ref<const json::object_t &>();
    }
    const json::string_t & getStringRef(const json & obj) {
        return obj.get_ref<const json::string_t &>();
    }

    int main() {
        json j2 = {
            {"string", "foo"},
            {"object", {
                {"currency", "USD"},
                {"value", 42.99}
            }}
        };

        json::object_t & objectRef = getObjectRef(valueAt(j2, "object"));
        json::string_t & nestedObjectRef  = getStringRef(valueAt(objectRef, "currency"));
        std::cout << nestedObjectRef << '\n';
    }
    ```

    This program causes undefined behavior when trying to access `nestedObjectRef`, because the second call to `valueAt`
    implicitly converts the `json::object_t` to a `json`, which is immediately discarded afterwards, causing
    `nestedObjectRef` to become a dangling reference.

    When `JSON_USE_IMPLICIT_CONSTRUCTORS` is defined to `0`, the code above no longer compiles. Instead, you must
    define an override for `valueAt` that takes `json::object_t`.

    ```cpp
    const nlohmann::json & valueAt(const json::object_t & map, const std::string & key) {
        return map.at(key);
    }
    ```

    This will prevent the implicit construction of an intermediate object, and the code will work as expected.


## See also

- [**basic_json constructor**](../basic_json/basic_json.md) - get a value (implicit)

## Version history

- Added in version 3.11.4.
