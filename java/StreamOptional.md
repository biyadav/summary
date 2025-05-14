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

##  Infinite Streams 

```

Stream<Integer> infiniteStream = Stream.iterate(0, i -> i + 2);
List<Integer> collect = infiniteStream
  .limit(10)
  .collect(Collectors.toList());  0 to 18


For custome Type
Supplier<UUID> randomUUIDSupplier = UUID::randomUUID;
Stream<UUID> infiniteStreamOfRandomUUID = Stream.generate(randomUUIDSupplier);

```
## Do-While – the Stream Way
  Stream<Integer> integers = Stream
   .iterate(0, i -> i + 1).integers.limit(10).forEach(System.out::println);

  IntStream.range(1, 6).forEach(i -> System.out.println("Iteration: " + i));

##  java.util.stream.Collectors  default method 



averaging	              averagingDouble(), averagingLong(), averagingInt()	To average elements of type Double/Long/Integer after applying a mapping function to the elements to 
                        extract respective values to be averaged
                                  
counting	              counting()	Count the number of stream elements

grouping	              groupingBy()	To produce Map of elements grouped by grouping criteria provided

String concatenation	  joining()	For concatenation of stream elements into a single String

mapping	mapping()	      Applying a mapping operation to all stream elements being collected

minimum and maximum     determination	minBy()/maxBy()	   To find minimum/maximum of all stream elements based on Comparator provided

partitioning	          partitioningBy()	To partition stream elements into a Map based on the Predicate provided

reduction	              reducing()	Reducing elements of stream based on BinaryOperator function provided

summarization	          summarizingDouble(), summarizingLong(), summarizingInt()	To summarize stream elements after mapping them to Double/Long/Integer value
                        using specific type Function
                      
summation	              summingDouble(), summingLong(), summingInt()	To sum-up stream elements after mapping them to Double/Long/Integer value using specific type Function
                        
collect in Collection	  toCollection()	To collect stream elements into a collection in arg pass HashMap::new 
                      
collect                 into a Map/ConcurrentMap	toMap()/toConcurrentMap()	  To collect stream elements into a map/concurrent map after applying provided key/value determination 
                        Function instances to the elements
                      
collect in a List	      toList()	Collects stream elements in a List

collect in a Set	      toSet()	Collects stream elements in a Set

collect and transform	  collectingAndThen()	Collects stream elements and then transforms them using a Function


## Way to create  Streams 


From Collections 
List<String> names = List.of("Alice", "Bob", "Charlie");
Stream<String> nameStream = names.stream();

From Arrays 
String[] cities = {"Mumbai", "Delhi", "Bangalore"};
Stream<String> cityStream = Arrays.stream(cities);

Using Stream.of
Stream<Integer> numberStream = Stream.of(1, 2, 3, 4, 5);
IntStream  intStream = IntStream.of(1, 2, 3, 4);

Using Stream.generate
Stream<Double> randomStream = Stream.generate(Math::random).limit(5);

Infinite Stream with iteration 
Stream<Integer> evenNumbers = Stream.iterate(0, n -> n + 2).limit(5);

From Files 
Stream<String> fileStream = Files.lines(Paths.get("data.txt"));

Empty String 
Stream<String> emptyStream = Stream.empty();


