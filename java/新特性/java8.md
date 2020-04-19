### 接口默认方法

```
interface Formula{

    double calculate(int a);

    default double sqrt(int a) {
        return Math.sqrt(a);
    }

}
```

### Lambda表达式
老版本实现匿名内部类对象
```
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```
java8 使用 Lambda 表达式实现匿名内部类对象
```
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```
或
```
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

### 函数式接口
“函数式接口”是指仅仅只包含一个抽象方法,但是可以有多个非抽象方法(也就是上面提到的默认方法)的接口。像这样的接口，可以被隐式转换为lambda表达式。
```
@FunctionalInterface
public interface Converter<F, T> {
  T convert(F from);
}
```

### 方法和构造函数引用
静态方法引用
```
Converter<String, Integer> converter = Integer::valueOf;  
Integer converted = converter.convert("123");
```
实例方法引用
```
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
```
构造方法引用
```
PersonFactory<Person> personFactory = Person::new;
```


