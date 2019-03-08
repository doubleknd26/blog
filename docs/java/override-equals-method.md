---
layout: default
title: override equals method
parent: Java
nav_order: 1
---

## 왜 equals 메서드를 overloading이 아닌 overriding 해야 할까?

java collection을 포함하여 java.util에 정의되어있는 다양한 클래스들은 Object.equals를 호출하기 때문에, equals 메서드를 오버로딩한다면, 의도치 않게 Object.equals()가 호출될 수 있다. 이러한 문제 때문에 overriding해야 한다.

<br>
### case study
```
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
testEquals() 메서드를 자세히 보면, apple이라는 이름을 갖는 2개의 Fruit 객체를 Set에 담았다. Set에 담았으니 기대한 값은 1이었지만, 사실상 2개가 모두 들어가 있어서 AssertionError 를 반환한다.  

어떻게 이게 가능할까? HashSet은 내부적으로 HashMap을 사용하고 있고, add()가 호출될 때마다 HashMap의 putVal() 을 호출하게 된다. putVal() 안에는 equals() 가 있는데, 여기서 내가 overloading한 equals() 가 아닌 Object.equals() 를 호출해서 new Fruit("apple") 을 다르다고 판단한 것이다.
```
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
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
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
```






