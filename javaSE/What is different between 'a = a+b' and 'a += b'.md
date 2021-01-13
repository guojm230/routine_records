# What is differnent between 'a = a + b' and 'a += b'. </br>

## Summary: </br>

- In most case,the bytecode compiled is fullly same. So these two different writing styles haven't any effects on performance.

- The main difference is about precision due to different strategy on type cast.

## Details: </br>

### case1: bytecode comparing: </br>

#### java code: </br>

```java
public static void main(String[] args) {
        int a = 1;
        int b = 2;
        a = a+ b;
        a += b;
}
```

#### corresponding bytecode: </br>

 ```bytecode
   L2    // a = a + b;
    LINENUMBER 7 L2
    ILOAD 1
    ILOAD 2
    IADD
    ISTORE 1
   L3   // a+=b;
    LINENUMBER 8 L3
    ILOAD 1
    ILOAD 2
    IADD
    ISTORE 1
 ```

### case2: type cast:</br>

``` java
public static void main(String[] args) {
        byte a = 1;
        int b = 2;
        a = a + b;   // compile error
        a += b; // equal to a = (byte) (a+b)
}
```

- In expression 'a = a + b',a will be cast to int. So that the result is also int and can't assign to byte type.

- In expression 'a += b',the cast '(byte)(a+b)' will be execute implicitly.

### case3 : a special case:</br>

```java
    static int[] a = {1,2,3,4,5};

    public static int test(int type){
        System.out.println("case " + type);
        return  (int) (5 * Math.random());
    }

    public static void case1(){
        a[test(1)] = a[test(1)] + 1;
    }

    public static void case2(){
        a[test(2)] += 1;
    }

    public static void main(String[] args) {
        case1();
        case2();
    }
```

This case is little special.</br>
The following is corresponding bytecode of case1 and case2.

```bytecode
  public static case1()V
   L0
    LINENUMBER 13 L0
    GETSTATIC com/guojm/Main.a : [I
    ICONST_1
    INVOKESTATIC com/guojm/Main.test (I)I
    GETSTATIC com/guojm/Main.a : [I
    ICONST_1
    INVOKESTATIC com/guojm/Main.test (I)I
    IALOAD
    ICONST_1
    IADD
    IASTORE
   L1
    LINENUMBER 14 L1
    RETURN
    MAXSTACK = 4
    MAXLOCALS = 0

  // access flags 0x9
  public static case2()V
   L0
    LINENUMBER 17 L0
    GETSTATIC com/guojm/Main.a : [I
    ICONST_2
    INVOKESTATIC com/guojm/Main.test (I)I
    DUP2
    IALOAD
    ICONST_1
    IADD
    IASTORE
   L1
    LINENUMBER 18 L1
    RETURN
    MAXSTACK = 4
    MAXLOCALS = 0
```

The case1 exists two INVOKESTATIC instruction which will invoke static method test. On the contrary,there are only one INVOKESTATIC instruction in case2 and a extra dup2 instrcation.Dup2 instruction will duplicate the top one or two values on the operand stack. It duplicate the value that test method return and only invoke test once.

In this case,two writing style will result in different results potentially.

> For more details to see [Offical definition of dup2 instruction](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html#jvms-6.5.dup2 ).


