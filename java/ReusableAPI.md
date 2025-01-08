https://www.slideshare.net/slideshow/java-notespdf-259708081/259708081

https://medium.com/@amitvsolutions/challenge-yourself-90-days-to-improve-java-and-dsa-skills-f32b3008b07e

https://www.slideshare.net/slideshow/java-se-8-for-java-ee-developers/54461691


https://www.slideshare.net/slideshow/java-8-streams-collectors-patterns-performances-and-parallelization/41467564

```

Int to Char Conversion
int i = 97; 
char ch = (char)i; // Type casting character to integer  result : a

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
```
