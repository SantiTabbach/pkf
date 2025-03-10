```typescript
/**
 * Recursively creates a deep clone of the given object.
 *
 * @param <T> The type of the source object.
 * @param source The object to be deeply cloned.
 * @return A deep copy of the given object.
 */
public static <T> T cloneDeep(T source) {
    if (!isObject(source)) {
        return source;
    }

    Map<String, Object> output = new HashMap<>();

    for (String key : source.keySet()) {
        output.put(key, cloneDeep(source.get(key)));
    }

    return (T) output;
}
```
