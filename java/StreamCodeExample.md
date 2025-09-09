## Useful links 

 https://medium.com/@javatechie/java-streams-features-exploring-hidden-methods-for-developers-72c89e641b03

 https://blog.devops.dev/java8-stream-api-613f66c87b2c
 
 https://medium.com/@bhangalekunal2631996/java-stream-api-coding-interview-questions-and-answers-2a212505e1c6
 
 https://blog.devgenius.io/java-8-real-time-coding-interview-questions-and-answers-fce01f3c4080
 
 https://medium.com/thefreshwrites/java-8-stream-api-interview-questions-and-answers-dd559ebffb35
 
 https://blog.devgenius.io/java-8-interview-questions-and-answers-1-19ad105123f7
 
 https://medium.com/thefreshwrites/java-8-stream-api-interview-questions-and-answers-dd559ebffb35
 
 https://medium.com/@abhishek.talakeriv/java8-stream-api-commonly-asked-interview-questions-db26d7328131
 
 https://stackabuse.com/guide-to-java-8-collectors-collectingandthen/


 https://medium.com/edureka/java-collections-interview-questions-6d20f552773e


### Streams Code   Examples 

java.util.stream.Collectors  is a utility class that provides various implementations of reduction operations such as grouping elements,  
collecting elements to different collections, summarizing elements according to various criteria, etc.  
The different functionalities in the Collectors class are usually used as the final operations on streams.  

  ```
 // Accumulate names into a List  List<String> list = people.stream().map(Person::getName).collect(Collectors.toList());  
 // Accumulate names into a TreeSet  Set<String> set = people.stream().map(Person::getName).collect(Collectors.toCollection(TreeSet::new));  
 // Convert elements to strings and concatenate them, separated by commas  
       String joined = things.stream().map(Object::toString).collect(Collectors.joining(", "));  
       String joined = things.stream().map(Object::toString).collect(Collectors.joining(", "[","]"));  // delemeter , prefix, suffix
       Map<BlogPostType, String> postsPerType = posts.stream()  
         .collect(groupingBy(BlogPost::getType, mapping(BlogPost::getTitle, joining(", ", "Post titles: [","]"))));  
    // Compute sum of salaries of employee  int total = employees.stream().collect(Collectors.summingInt(Employee::getSalary));  
   // Group employees by department  Map<Department, List<Employee>> byDept = employees.stream().collect(Collectors.groupingBy(Employee::getDepartment));  
   // Compute sum of salaries by department  Map<Department, Integer> totalByDept = employees.stream().  
   collect(Collectors.groupingBy(Employee::getDepartment,Collectors.summingInt(Employee::getSalary)));  
   // Partition students into passing and failing  
   Map<Boolean, List<Student>> passingFailing = students.stream().collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));  


List<Integer> list = Stream.of(12, 13, 14, 15)  
    .collect(  
    //Supplier  
    () -> new ArrayList<Integer>(),  
    //Accumulator  
    (l, e) -> l.add(e),  
    //Combiner  
    (l, ar) -> l.addAll(ar)  
);  


 List<Integer> list = Arrays.asList(1, 2, 3);  
 Boolean empty = list.stream()  
     .collect(collectingAndThen(  
        toList(),  
        List::isEmpty  
     )  
 );  
  
   String longestName = people.stream()  
    .collect(collectingAndThen(  
        // Encounter all the Person objects  
        // Map them to their first names  
        // Collect those names in a list  
        mapping(  
            Person::getFirstName,  
            toList()  
        ),  
        // Stream those names again  
        // Find the longest name  
        // If not available, return "?"  
        l -> {  
            return l  
                .stream()  
                .collect(maxBy(  
                    comparing(String::length)  
                ))  
                .orElse("?");  
           }  
       )  
    );  



   public static <T, U, A, R>   Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper,
                                    Collector<? super U, A, R> downstream)     mapping(function , toCollection())
   Adapts a Collector accepting elements of type U to one accepting elements of type T by applying a mapping function to each input 
   element before accumulation.
   to compute the set of last names of people in each city:
   Map<City, Set<String>> lastNamesByCity    = people.stream().collect(groupingBy(Person::getCity,
                                                                                  mapping(Person::getLastName, toSet())));

   Below employees being grouped as per their department. However, this time we will store the grouped elements in a Set 
    and tell the grouping collector to store the grouped employees in a TreeMap instance instead of the default HashMap instance 
   Map<Department,Set<Employee>> employeeMap
      = employeeList.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment, TreeMap::new, Collectors.toSet()));
    System.out.println("Employees grouped by department");
    employeeMap.forEach((Department key, Set<Employee> empSet) -> System.out.println(key +" -> "+empSet));
    


The flatMapping() collectors are most useful when used in a multi-level reduction, such as downstream of a groupingBy or partitioningBy. 
  For example, given a stream of Order, to accumulate the set of line items for each customer:
 Map<String, Set<LineItem>> itemsByCustomerName    = orders.stream().collect(groupingBy(Order::getCustomerName, 
                                                                                             flatMapping(order -> order.getLineItems().stream(),toSet())));

 Given a stream of Employee, to accumulate the employees in each department that have a salary above a certain threshold:
 Map<Department, Set<Employee>> wellPaidEmployeesByDepartment    = employees.stream().collect(groupingBy(Employee::getDepartment,         
                                                                                                          filtering(e -> e.getSalary() > 2000,toSet())));

 For example, one could adapt the toList() collector to always produce an immutable list with:
 List<String> list = people.stream().collect(collectingAndThen(toList(),Collections::unmodifiableList));

 Show Highest salary employee name  else  show name 
 String maxSalaryEmp = employeeList.stream().collect(
                                             Collectors.collectingAndThen( Collectors.maxBy(Comparator.comparing(Employee::getSalary)),(Optional<Employee> emp)-> emp.isPresent() ? emp.get().getName() : "none") );
System.out.println("Max salaried employee's name: "+ maxSalaryEmp);

Find the average salary of all employees using averagingDouble collector  , Print the average salary after formatting it using DecimalFormat.
 String avgSalary = employeeList.stream().collect(
        Collectors.collectingAndThen(
            Collectors.averagingDouble(Employee::getSalary),
            averageSalary -> new DecimalFormat("'$'0.00").format(averageSalary)));


 Comparator<String> byLength = Comparator.comparing(String::length);
 Map<City, String> longestLastNameByCity    = people.stream().collect(groupingBy(Person::getCity,
                                                                                                reducing("",Person::getLastName,BinaryOperator.maxBy(byLength))));

The following produces a Map mapping students to their grade point average:
 Map<Student, Double> studentToGPA    = students.stream().collect(toMap(Function.identity(),student -> computeGPA(student)));

And the following produces a Map mapping a unique identifier to students:
 Map<String, Student> studentIdToStudent    = students.stream().collect(toMap(Student::getId,Function.identity()));


Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                    Function<? super T, ? extends U> valueMapper,
                                    BinaryOperator<U> mergeFunction) 
  For example, if you have a stream of Person, and you want to produce a "phone book" mapping name to address, 
  but it is possible that two persons have the same name, you can do as follows to gracefully deal with these collisions, and produce a Map mapping names to a concatenated list of addresses:
 Map<String, String> phoneBook    = people.stream().collect(toMap(Person::getName,Person::getAddress,(s, a) -> s + ", " + a));

The following produces a ConcurrentMap mapping students to their grade point average:
 ConcurrentMap<Student, Double> studentToGPA    = students.stream().collect(toConcurrentMap(Function.identity(),student -> computeGPA(student)));
And the following produces a ConcurrentMap mapping a unique identifier to students:
 ConcurrentMap<String, Student> studentIdToStudent    = students.stream().collect(      toConcurrentMap(Student::getId,Function.identity()))



1. Find out the count of male and female employees present in the organization?

   public static void getCountOfMaleFemale(List<Employee> employeeList) {
         Map<String, Long> noOfMaleAndFemaleEmployees=
         employeeList.stream()
                     .collect(Collectors.groupingBy
                     (Employee::getGender, Collectors.counting()));    
         System.out.println(noOfMaleAndFemaleEmployees);
     }

   Output: {Male=11, Female=6}
2.  Write a program to print the names of all departments in the organization.


 public static void getDepartmentName(List<Employee> employeeList){
         employeeList.stream()
         .map(Employee::getDepartment)
         .distinct()
         .forEach(System.out::println);
     }


 Output: 
 HR
 Sales And Marketing
 Infrastructure
 Product Development
 Security And Transport
 Account And Finance

3. Find the average age of Male and Female Employees.

   public static void getGender(List<Employee> employeeList) {
    Map<String, Double> avgAge = employeeList.stream()
                                 .collect(Collectors.groupingBy
                                 (Employee::getGender, 
                                 Collectors.averagingInt
                                 (Employee::getAge)));
                 System.out.println(avgAge);
     }

 Output: {Male=30.181818181818183, Female=27.166666666666668}


4.Get the Names of employees who joined after 2015.

  public static void getNameOfEmp(List<Employee> employeeList) {
         employeeList.stream()
         .filter(e -> e.getYearOfJoining() > 2015)
         .map(Employee::getName)
         .forEach(System.out::println);
     }

 Output: 
 Iqbal Hussain
 Amelia Zoe
 Nitin Joshi
 Nicolus Den
 Ali Baig



5.Count the number of employees in each department.

   public static void countByDept(List<Employee> employeeList) {
         Map<String, Long> countByDept = employeeList.stream()
                                         .collect(Collectors.groupingBy
                                         (Employee::getDepartment, 
                                         Collectors.counting()));
         Set<Entry<String, Long>> entrySet = countByDept.entrySet();
         for (Entry<String, Long> entry : entrySet)
         {
             System.out.println(entry.getKey()+" : "+entry.getValue());
         }
     }

 Output:
 Product Development : 5
 Security And Transport : 2
 Sales And Marketing : 3
 Infrastructure : 3
 HR : 2
 Account And Finance : 2


6.Find out the average salary of each department.

   public static void avgSalary(List<Employee> employeeList) {
         Map<String, Double> avgSalary = employeeList.stream()
                                         .collect(Collectors.groupingBy
                                                 (Employee::getDepartment, 
                                                 Collectors.averagingDouble(Employee::getSalary)));
         Set<Entry<String, Double>> entrySet = avgSalary.entrySet();
         for (Entry<String, Double> entry : entrySet) 
         {
             System.out.println(entry.getKey()+" : "+entry.getValue());
         }
     }

 Output:
 Product Development : 31960.0
 Security And Transport : 10750.25
 Sales And Marketing : 11900.166666666666
 Infrastructure : 15466.666666666666
 HR : 23850.0
 Account And Finance : 24150.0


7. Find out the oldest employee, his/her age and department?

  public static void oldestEmp(List<Employee> employeeList) {
         Optional<Employee> oldestEmp = employeeList.stream()
                                             .max(Comparator
                                                     .comparingInt(Employee::getAge));        Employee oldestEmployee = oldestEmp .get();

         System.out.println("Name : "+oldestEmployee.getName());    
         System.out.println("Age : "+oldestEmployee.getAge());     
         System.out.println("Department : "+oldestEmployee.getDepartment());
     }

 Output:
 Name : Iqbal Hussain
 Age : 43
 Department : Security And Transport


8. Find out the average and total salary of the organization.

   public static void getEmpSalary(List<Employee> employeeList) {
         DoubleSummaryStatistics empSalary = employeeList.stream()
                                                 .collect(Collectors
                                                         .summarizingDouble(Employee::getSalary));

         System.out.println("Average Salary = "+empSalary.getAverage());
         System.out.println("Total Salary = "+empSalary.getSum());
     }

 Output:
 Average Salary = 21141.235294117647
 Total Salary = 359401.0

9. List down the employees of each department.

    public static void listDownDept(List<Employee> employeeList) {
         Map<String, List<Employee>> empList = employeeList.stream()
                                                 .collect(Collectors
                                                         .groupingBy(Employee::getDepartment));

         Set<Entry<String, List<Employee>>> entrySet = empList.entrySet();

         for (Entry<String, List<Employee>> entry : entrySet) 
         {
             System.out.println("--------------------------------------");    
             System.out.println("Employees In "+entry.getKey() + " : ");
             System.out.println("--------------------------------------");

             List<Employee> list = entry.getValue();
             for (Employee e : list) 
             {
                 System.out.println(e.getName());
             }
         }
     }

 Output:
 Employees In Product Development :
 ————————————–
 Murali Gowda
 Wang Liu
 Nitin Joshi
 Sanvi Pandey
 Anuj Chettiar
 ————————————–
 Employees In Security And Transport :
 ————————————–
 Iqbal Hussain
 Jaden Dough
 ————————————–
 Employees In Sales And Marketing :
 ————————————–
 Paul Niksui
 Amelia Zoe
 Nicolus Den

10. Find out the highest experienced employees in the organization

    ublic static void seniorEmp(List<Employee> employeeList) {
        Optional<Employee> seniorEmp = employeeList.stream()
                                        .sorted(Comparator
                                                .comparingInt(Employee::getYearOfJoining)).findFirst();

        Employee seniorMostEmployee = seniorEmp.get();

        System.out.println("Senior Most Employee Details :");
        System.out.println("----------------------------");
        System.out.println("ID : "+seniorMostEmployee.getId());
        System.out.println("Name : "+seniorMostEmployee.getName());
        System.out.println("Age : "+seniorMostEmployee.getAge());
    }

Output:
Senior Most Employee Details :
—————————-
ID : 177
Name : Manu Sharma
Age : 35

11. Count the occurrences of each word in a Array of strings using streams.

    public static void main(String[] args) {
    String[] words= {"apple", "banana", "apple", "orange", "banana", "apple"};
    Map<String, Long> collect = Arrays.asList(words).stream().collect(Collectors.groupingBy(Function.identity(),Collectors.counting()));
    System.out.println(collect);
  
   }
  }

12.  Write a program to find the longest string in a list of strings using streams.
   
    List<String> list = Arrays.asList("apple", "banana", "orange", "kiwi", "strawberry");
   Optional<String> max = list.stream().max(Comparator.comparingInt(String::length));
   System.out.println(max.get());



13. Given a list of integers, remove duplicates and keep them in the descending order using streams.

    List<Integer> numbers = Arrays.asList(1, 2, 3, 2, 4, 5, 1);
    List<Integer> collect = numbers.stream().distinct().sorted(Comparator.comparingInt(Integer::intValue).reversed()).collect(Collectors.toList());
    System.out.println(collect);
14. Write a program to find the average of a list of doubles using streams.

    List<Double> doubles = Arrays.asList(1.2, 3.5, 2.8, 4.1, 5.7);
     OptionalDouble average = doubles.stream().mapToDouble(Double::doubleValue).average();
     System.out.println(average.getAsDouble());

15. Merge two lists of integers and remove duplicates using streams.

    List<Integer> list2 = Arrays.asList(3, 4, 5);
    List<Integer> collect = Stream.concat(list1.stream(), list2.stream()).distinct().collect(Collectors.toList());
    System.out.println(collect);

16. Given a list of strings, concatenate them into a single string using streams.

    List<String> list = Arrays.asList("Hello", " ", "world", "!");
    String collect = list.stream().collect(Collectors.joining());
    System.out.println(collect);
17. Write a program to find the first non-repeating character in a string using streams.

    String str = "abacdbef";
 
     
     Optional<Character> firstNonRepeatingChar = str.chars()
                .mapToObj(c -> (char) c)
                .collect(Collectors.groupingBy(Function.identity(), LinkedHashMap::new, Collectors.counting()))
                .entrySet()
                .stream()
                .filter(e -> e.getValue() == 1L)
                .map(Map.Entry::getKey)
                .findFirst();
     System.out.println(firstNonRepeatingChar.get());

18. Given a list of strings, remove all strings that contain a specific character using streams.

     List<String> list = Arrays.asList("apple", "banana", "orange", "kiwi");
     char specificChar = 'a';
     List<String> collect = list.stream().filter(s->!s.contains(String.valueOf(specificChar))).collect(Collectors.toList());
     System.out.println(collect);

19. Given a list of integers, partition them into two groups: odd and even, using streams.


    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
    Map<Boolean, List<Integer>> oddEvenPartition = numbers.stream()
                             .collect(Collectors.partitioningBy(n -> n % 2 == 0));
     System.out.println(oddEvenPartition);

20. Given an array of integers, find the kth largest element.

   List<Integer> list = Arrays.asList(1, 12, 44, 32, 52, 81, 59, 84, 72, 37);
  int k = 4;
  Integer num = list.stream().sorted(Comparator.reverseOrder()).limit(k).skip(k - 1).findFirst().orElse(-1);
  System.out.println(num);

21. Given a list of strings, find the count of strings starting with a vowels.

  List<String> list = Arrays.asList("apple", "banana", "orange", "kiwi", "strawberry");
  long count = list.stream().filter(s -> "aeiouAEIOU".contains(String.valueOf(s.charAt(0)))).count();
  System.out.println(count);

22. Given a list of strings, find the longest palindrome string.

     List<String> list = List.of("level", "hello", "radar", "world", "madam", "java", "Malayalam");
    String str = list.stream().filter(s -> new StringBuilder(s).reverse().toString().equalsIgnoreCase(s))
      .max(Comparator.comparingInt(String::length)).orElse("");
    System.out.println(str);

23. Given a list of integers, find the product of all non-negative integers.

   List<Integer> integerList = Arrays.asList(4, 5, -6, 7, -1, 2, -3);
  long longNumber = integerList.stream().filter(num -> num >= 0).mapToLong(Integer::longValue).reduce(1, (a, b) -> a * b);
  System.out.println(longNumber);



24.
    https://stackabuse.com/guide-to-java-8-collectors-collectingandthen/
    Given a list of  Students  find the  highest subject per student   
    Student { 
        String name 
        List<Subject> subjects 
       Subject getHighest(){ //subject having highest marks 
        
            Subject highest = subjects.stream.collect(
               collectingAndThen(
                  Collectors.maxBy(
                      Comparator.comparing(
                         Subject::getMarks
                     )
                 ),
                  m -> m.orElseThrow(
                    RuntimeException::new
                 )
             )
          );
            return highest;
    }
}
   

public class Subject implements Comparable {
    private final String subName;
    private final BigDecimal marks;
    private final int examYear ;
    
    //Constructor and getters...
    
    @Override
    public int compareTo(Subject other) {
        return Comparator.comparing(Subject::getMarks)
            .compare(this, other);
    }
}
        

Sort in reverse Order 
Comparator<Employee> employeeNameComparator
      = Comparator.comparing(Employee::getName);
    Comparator<Employee> employeeNameComparatorReversed 
      = employeeNameComparator.reversed();
    Arrays.sort(employees, employeeNameComparatorReversed);
    assertTrue(Arrays.equals(employees, sortedEmployeesByNameDesc));
}






26 .

The reducing() collectors are most useful when used in a multi-level reduction, downstream of groupingBy or partitioningBy.  
 To perform a simple map-reduce on a stream, use Stream.map(Function) and Stream.reduce(Object, BinaryOperator) instead.  
For example, given a stream of Person, to calculate the longest last name of residents in each city:


     Comparator<String> byLength = Comparator.comparing(String::length);
     Map<City, String> longestLastNameByCity
         = people.stream().collect(groupingBy(Person::getCity,
                                              reducing(Person::getLastName, BinaryOperator.maxBy(byLength))));
```
                                              
collect()  
forEach()  
forEachOrdered()  
findAny()  
findFirst()  
toArray()  
reduce()  
count()  
min()  
max()  
mapToInt()  // IntStream LongStream etc have mapToObj  to convert into ObjectStream from stream of premitive 
mapToLong()  
mapToDouble()  
anyMatch()  
allMatch()  
noneMatch()  
