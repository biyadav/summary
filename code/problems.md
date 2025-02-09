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
