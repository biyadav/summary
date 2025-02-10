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

# find largest number less than a given number and without a given digit

```
class Main {
    public static void main(String[] args) {

      int input =  145;
      int unwantedDigit = 4;
      int result = input;
      
      do
        {
           result--;  // at least decrese one time so do 
        }
      while ( String.valueOf(result).contains(String.valueOf(unwantedDigit)) ); // since we decrese one time only check if we dont have unwanted digit 
      
      System.out.println("Required number is " +result);
    }
}

# find continuous sub array whose sum is equal to given number

```

class Main {
    public static void main(String[] args) {
    
      int inputArray [] = {12, 5, 31, 9, 21, 8} ;
      int desiredSum = 45;
      int left=0,right=1; // select left and right pointer 
      
      int currentSum = inputArray[left];  // dont take sum zero instead set to left item
      
      while(right <=inputArray.length-1){
          currentSum = currentSum+inputArray[right]; // I did  left+right this was issue as I need to add only next right to existing sum 
          System.out.println("currentSum "+ currentSum );
         if (currentSum==desiredSum){
             System.out.println("left " +left + "right "+ right );
             break; // BREAK is important when result got 
         }
         else if(currentSum>desiredSum ){
             left++;
             currentSum = inputArray[left];
             right= left+1;
         }
         else if(currentSum<desiredSum ){
             right++;
         }
          System.out.println("left " +left + "right "+ right );
      }
      int[] result =java.util.Arrays.copyOfRange(inputArray,left,right+1);
      System.out.println(java.util.Arrays.toString(result));
    }
}

``
