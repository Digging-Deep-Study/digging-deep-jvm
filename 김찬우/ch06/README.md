# Chapter 6

## Examples

```java
public class NumberWrapper {
    private int value;

    public NumberWrapper(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public void setValue(int value) {
        this.value = value;
    }

    public static int add(int a, int b) {
        return a + b;
    }
}
```

```
Compiled from "NumberWrapper.java"
public class NumberWrapper {
  private int value;

  public NumberWrapper(int);
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iload_1
       6: putfield      #2                  
       9: return

  public int getValue();
    Code:
       0: aload_0
       1: getfield      #2                 
       4: ireturn

  public void setValue(int);
    Code:
       0: aload_0
       1: iload_1
       2: putfield      #2                  
       5: return

  public static int add(int, int);
    Code:
       0: iload_0
       1: iload_1
       2: iadd
       3: ireturn
}
```

