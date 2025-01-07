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
