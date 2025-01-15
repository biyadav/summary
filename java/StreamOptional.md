# java.util.Optional
```
A container object which may or may not contain a non-null value. If a value is present, isPresent() returns true. 
If no value is present, the object is considered empty and isPresent() returns false.orElse() (returns a default value if no value is present)
and ifPresent() (performs an action if a value is present).

Optional.empty()   return  Optional.EMPTY  use when to wrap null .
public static <T> Optional<T> of(T value)   throws NPE if value is null
public static <T> Optional<T> ofNullable(T value)  retrun Optional.EMPTY if value is null
public T get()  NoSuchElementException – if no value is present
public boolean isPresent()
public boolean isEmpty()
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)If a value is present, returns an Optional describing the value, otherwise returns an Optional produced by the supplying function.
public T orElse(T other) If a value is present, returns the value, otherwise returns other
public T orElseThrow() If a value is present, returns the value, otherwise throws NoSuchElementException.
public void ifPresent(Consumer<? super T> action) If a value is present, performs the given action with the value, otherwise does nothing
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) If a value is present, performs the given action with the value, otherwise performs the given empty-based action.
public Optional<T> filter(Predicate<? super T> predicate)  If a value is present, and the value matches the given predicate, returns an Optional describing the value, otherwise returns an empty Optional.

Optional<Path> p = uris. stream().filter(uri -> !isProcessedYet(uri))
                   .findFirst()                   
                   .map(Paths::get);  Here, findFirst returns an Optional<URI>, and then map returns an Optional<Path> for the desired URI if one exists.

```
