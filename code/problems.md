# Second Highest in a array 

```

class Main {
    public static void main(String[] args) {
      
      int[] array = { 2,41,34,55,66,10,30};
      int max =Integer.MIN_VALUE,secondMax = Integer.MIN_VALUE;
      for(int element :array){
          if(element > max ){
              secondMax = max; // first assign current max to second highest 
              max= element;
          }
            else if(element > secondMax ){
                secondMax = element;
            }
      }
      
      System.out.println("max " + max+" secondMax "+secondMax);
    }
}
```
# find a number is armstrong  ( number that is equal to the sum of its own digits each raised to the power of the number of digits)

```

class Main {
    public static void main(String[] args) {
      
     int number = 153;
     int numberOfDigits = String.valueOf(number).length()
     int  sum =0;
     int temp = number;
     while (temp !=0){
          System.out.println( "temp "+temp);
         int currentDigit = temp%10;
         sum = sum + (int )Math.pow(currentDigit,numberOfDigits);
          System.out.println( "Sum "+sum);
         temp = temp/10;
     }
      System.out.println( "Sum "+sum);
      
      System.out.println( " is armstiong "+(sum==number));
    }
}

```
