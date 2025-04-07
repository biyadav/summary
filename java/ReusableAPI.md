https://www.slideshare.net/slideshow/java-notespdf-259708081/259708081

https://medium.com/@amitvsolutions/challenge-yourself-90-days-to-improve-java-and-dsa-skills-f32b3008b07e

https://www.slideshare.net/slideshow/java-se-8-for-java-ee-developers/54461691


https://www.slideshare.net/slideshow/java-8-streams-collectors-patterns-performances-and-parallelization/41467564

```

Int to Char Conversion
int i = 97; 
char ch = (char)i; // Type casting integer to character result : a

Character  charObj = Character.valueOf((char)ch)   where ch is primitive character 

int i = 64;
char ch = (char)(i + '0'); Via Type-casting With Adding Zero  result : p
 
 int base = 10;
 int a = 5;
 char ans = Character.forDigit(a, base);

Char to Int Conversion

char ch = 'a'
int intValue = ch or int intValue = (int)ch  // cast to int 

char ch = '40'
int a = ch - '0'; // subtract '0'

char ch = '5'
String s = String.valueOf(ch) // "5" convert to string 5
int five = Integer.parseInt(s) //  result  5 .  Will not work with 'a' as Integer.parseInt("a")  NumberFormatException although a = 52

Integer.intValue() // return primitive int
Integer.valueOf(int n) // convert primitive to Integer   use constructor  integer two = new Integer(2);
Integer.max(int x,int y)
Integer.min(int x,int y)
Integer.sum(int x,int y)
```

Comparator c= String.CASE_INSENSITIVE_ORDER

##  Regex 
```
single whitespace   "\\s"    multiple whitespaces "\\s+"     tab  "\t"  The 's' replaces one space match at a time but the 's+' replaces the whole space sequence at once with the second parameter
String x = "Text   With     Whitespaces!   ";
x.replaceAll("\\s", "_");   "Text___With_____Whitespaces!___"
x.replaceAll("\\s+", "_");  "Text_With_Whitespaces!_"
```

## Stream 

```
map.entrySet().stream().max(Map.Entry.comparingByValue()) // having map find max min on map value   

Map.Entry<String, Long> maxNoOfEmployeesInDept = empList.stream().collect(Collectors.groupingBy(Employee::getDeptName, Collectors.counting())).
                                                 entrySet().stream().max(Map.Entry.comparingByValue()).get();
System.out.println("Max no of employees present in Dept :: " + maxNoOfEmployeesInDept.getKey());
```

```
OptionalInt max = empList.stream().mapToInt(Employee::getAge).max();
if (max.isPresent()) 
System.out.println("Maximum age of Employee: " + max.getAsInt());

```
```

Map<String, Double> avgAge = empList.stream().collect(Collectors.groupingBy
                             (Employee::getGender,Collectors.averagingInt
                              Employee::getAge)));
System.out.println("Average age of Male and Female Employees:: " + avgAge);
```
