---
layout: default
title: Equals Method Overriding
parent: Java
nav_order: 1
---

# Equals Method Overriding
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

개발자라면 한번쯤은 "equals 메서드는 오버라이딩 해라" 라는 말을 들어봤을 것이다. 그런데 굳이 오버라이딩 하지 않고 오버로딩으로 구현해도 괜찮지 않을까? 사실, equals() 메서드의 역할에는 크게 문제가 없을 것이다. 그렇다면 왜 오버로딩이 아닌 오버라이딩을 해야 하는걸까?

## why
**이미 잘 정의되어 있는 java 메서드를 사용할 때, 예기치 못한 문제들을 마주칠 수 있기 때문이다.**  
java collection을 포함하여 java package에 잘 정의되어있는 수많은 메서드들 중 Object.equals()를 호출하는 메서드는 꽤 많을 것이다. 그리고 우리는 이런 메서드들을 알게 모르게 많이 사용하고 있다. 대표적으로 Collection 관련 메서드들이 그것이다. 그런데 이런 메서드들을 사용할 때, 우리가 생성한 객체에서 equals()를 오버로딩하여 구현했다면, 의도치 않게 Object.equals()가 호출되면서 원치 않은 결과를 얻게될 것이다.

## case study
```java
class Fruit {
    private String name;

    public Fruit(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
    // not override equals method.
    public boolean equals(Fruit o) {
        if (this == o) return true;
        if (!(o instanceof Fruit)) return false;
        Fruit fruit = (Fruit) o
        return Objects.equals(getName(), fruit.getName());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getName());
    }
}
```
Fruit 클래스는 equals()를 오버로딩한 단순한 클래스이다. 이 클래스를 가지고 아래 테스트를 실행해보면 AssertionError가 발생하면서 테스트는 통과한다.
```java
@Test(expected = AssertionError.class)
public void testEquals() {
    Fruit apple1 = new Fruit("apple");
    Fruit apple2 = new Fruit("apple");

    Set<Fruit> fruits = new HashSet<>();
    fruits.add(apple1);
    fruits.add(apple2);

    assertEquals(1, fruits.size());
}
```
testEquals() 메서드를 보면, apple이라는 이름을 갖는 2개의 Fruit 객체를 Set에 담았다. Set에 담았으니 기대한 값은 1이었지만, 사실상 2가 반환되어 AssertionError가 발생한다. 분명히 우리가 기대했던 것과는 다를 것이다.  

HashSet은 내부적으로 HashMap을 사용하고 있고, add()가 호출될 때마다 HashMap의 putVal() 을 호출하게 된다. putVal() 안에는 equals() 가 있는데, 여기서 내가 overloading한 equals() 가 아닌 Object.equals() 를 호출해서 new Fruit("apple") 을 다르다고 판단한 것이다.  

아래는 java.util.HashMap의 putVal() 코드에서 equals() 가 호출되는 곳을 보여주는 코드다.

```java
/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k)))) // HERE
            e = p;
        else if (p instanceof TreeNode)
        .....
    }
}
```

## summary
- equals 메서드를 오버로딩이 아닌 오버라이딩해야하는 이유는 이미 잘 정의되어 있는 java의 메서드들을 제대로 재사용하기 위함이다.





