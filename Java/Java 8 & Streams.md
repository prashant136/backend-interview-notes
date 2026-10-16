
Stream is an interface use for data processing in a functional style syntax. Streams can't modify original source of data.
Sequential Streams (single thread) -> `.stream()`
Parallel Stream (multiple threads) -> `.parallelStream()`

1 :: How to create stream in java ?

```
a.) Arrays.asList() = convert array or comma separated values to List.
List ko stream me convert karo ?
List<String> name = Arrays.asList("alice", "bob");
Stream<String> stream = name.stream();
	
b.) Array ko stream me convert karo ?
String[] arr = {"Java", "Python", "Cpp"};
Stream<String> stream = Arrays.stream(arr);

c. ) Stream.of() [factory method] used to create a sequential, ordered stream from a fixed set of values.
Stream<Integer> integerStrem = Stream.of(1,2,3,4);

d.) stream generate
Stream<Double> limit = Stream.generate(Math::random).limit(5);
```

intermediate operation. = they are started lazily, only when a terminal operation is invoked. no actual processing start immediately.


2 :: Stream.collect() - terminal operation accumulate elements into collection like List, Set, Map, or String and return.
Collectors -> how collection or accumulation is happen.

.collect(Collectors.toSet());
.collect(Collectors.toList());
.collect(Collectors.toMap());

3 :: Stream.distinct() - intermediate opr. - remove duplicate

4 :: reduce (terminal operation) is used to combine all elements of a stream into single value. 
while map and filter are intermediate operation.



5 :: findFirst() used to return the first element of a stream. return type - 
`Optional<T>`
findAny() used to return any element from the stream.

6 :: anyMatch() -> if any element matches return true 
allMatch() ->  all element matches -> true
noneMatch() -> no element matches -> true

7 :: sorted() method is an intermediate operations used to sort the elements of a stream. it takes Comparator for custom sorting.

list.stream().sorted().forEach(System.out::println);    // natural order (ascending)
list.stream().sorted((a, b) -> b - a).forEach(System.out::println);    // desc order

Difference b/w sorted() and Collections.sort() -
sorted() - method inside stream, lazy operation, does not modify original list, work with streams.
Collections.sort() - part of Collection framework, eager operation, modifies original list, work with collections.

8 :: map and flatmap ?
 map :- it is used to transform each element into another form. It takes one element and return one element.
 
 flatmap :- is used when each element can produce multiple elements, and it flattens them into single stream. remove nesting.

9 :: Primitive streams - streams works with collections. but when dealing with primitive types like int, long, double, using regular streams causes boxing and unboxing conversions.  So, java provides primitive streams IntStream, LongStream, DoubleStream. (processing the data , directly work with primitives, more efficient and faster)

10 ::  skip(n) - discard first n element
	 limit(n) - control the size of stream. both are intermediate op.

11 :: **Collectors.partitioningBy()** Java Streams ka ek built-in collector hai jo elements ko ek condition k basis par do hisson (groups) mein divide karta hai.
```
List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3,4,5));  
Map<Boolean, List<Integer>> result = list.stream()  
        .collect(Collectors.partitioningBy(x -> x % 2 == 0));
        
System.out.println(result.get(true));  // [2, 4] -> 'true' as key

// eska return format {key -> value pair} hota hai.
{false=[1, 3, 5], true=[2, 4]}, jo chahiye wo ger kr do.  
```

12 :: **Comparator.comparing()** - Java Streams mein Object ko compare karne ke liye jo Ek Universal Method hai. 
Rule - Jo bhi compare karna hai, use Comparator.comparing() ke andar daal do. isse hum objects, string, int, double, long hum sab kuch compare kr sakte hai.

max(), min() are terminate operations , use to find max/min element in a stream based on a given comparator.

```
List<Integer> nums = Arrays.asList(10, 20, 5, 40, 30);  
Optional<Integer> largestNum = nums.stream().max((n1, n2) -> n1.compareTo(n2));  
largestNum.ifPresent(System.out::println);   // 40
```








--------

#### Parallel Streams - parallelStream() process the data using multiple threads.

*for small datasets, parallelStream() doesn't make much difference in performance since the task is already fast with normal stream. for smaller dataset, parallelStream()  can even slower becoz of the overhead (managing multiple threads). spiltting task into parts and then combining the results back together. 
by default parallelStream() uses ForkJoinPool.commonPool()*

int = 32 bits
long = 64 bits
double = 64 bits, decimal
float = 32 bits, decimal




