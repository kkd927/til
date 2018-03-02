Java 언어에서 배열 arr의 모든 요소를 표시하기 위해서는 다음과 같이 for문을 사용합니다.

```java
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}
```

for문의 변수 i를 통해 배열 arr의 요소에 접근할 수 있습니다. Iterator 패턴은 이런 변수 i의 기능을 추상화해서 일반화 한 것 입니다.

Iterator 패턴의 구성은 아래와 같습니다.

**Iterator(반복자)**

요소를 순서대로 검색해가는 인터페이스(API) 입니다. 다음 요소가 존재하는지를 얻기 위한 `hasNext` 메서드와 다음 요소를 얻기 위한 `next` 메서드를 결정합니다.

**ConcreteIterator(반복자 구현체)**

Iterator 인터페이스의 구현체 입니다.

**Aggregate(집합체)**

Iterator 역할을 만들어내는 인터페이스(API) 입니다.

**ConcreteAggregate(집합체 구현체)**

Aggregate 인터페이스의 구현체 입니다.

Iterator 패턴을 클래스 다이어그램으로 표기하면 다음과 같습니다.

![iterator class diagram](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile4.uf.tistory.com%2Fimage%2F99C8A03D5A992031198354)

Iterator 패턴을 사용하여 만든 예제 프로그램을 살펴봅시다. 여기에서 작성할 예제 프로그램은 서가(BookShelf) 안에 책(Book)을 넣고, 그 책의 이름을 차례대로 표시하는 프로그램입니다.

![iterator example class diagram](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile22.uf.tistory.com%2Fimage%2F99D7213B5A9921DD0DA7C9)

```java
public interface Aggregate {
    public abstract Iterator iterator();
}

public interface Iterator {
    public abstract boolean hasNext();
    public abstract Object Next();
}
```

```java
public class Book {
    private String name;

    public Book(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }
}
```

```java
public class BookShelf implements Aggregate {
    private Book[] books;
    private int last = 0;

    public BookShelf(int maxsize) {
        this.books = new Book[maxsize];
    }

    public Book getBookAt(int index) {
        return books[index];
    }

    public void appendBook(Book book) {
        this.books[last] = book;
        last++;
    }

    public int getLength() {
        return last;
    }

    public Iterator iterator() {
        return new BookShelfIterator(this);
    }
}
```

```java
public class BookShelfIterator implements Iterator {
    private BookShelf bookShelf;
    private int index;

    public BookShelfIterator(BookShelf bookShelf) {
        this.bookShelf = bookShelf;
        this.index = 0;
    }

    public boolean hasNext() {
        if (index < bookShelf.getLength()) {
            return true;
        } else {
            return false;
        }
    }

    public Object next() {
        Book book = bookShelf.getBookAt(index);
        index++;
        return book;
    }
}
```

이제 예제를 실행시켜 봅시다.

```java
BookShelf bookShelf = new BookShelf(4);
bookShelf.appendBook(new Book("Around the World in 80 Days"));
bookShelf.appendBook(new Book("Bible"));
bookShelf.appendBook(new Book("Cinderella"));
bookShelf.appendBook(new Book("Baddy-Long-Legs"));

Iterator it = bookShelf.iterator();
while (it.hasNext()) {
    Book book = (Book)it.next();
    System.out.println(book.getName());
}
```

Java 개발자라면 iterator를 한 번쯤 보시거나 사용해본 경험이 있을 것입니다. 배열이라면 for문을 이용해서 빙글빙글 돌리면 좋은데, 왜 집합체의 외부에 Iterator 역할 같은 것을 만들어야 할까요?

가장 큰 이유는 **Iterator를 사용함으로써 구현과 분리해서 하나씩 셀 수 있기 때문** 입니다.

```java
while (it.hasNext()) {
    Book book = (Book)it.next();
    System.out.println(book.getName());
}
```

위의 코드에서 while 루프는 BookShelf의 구현에는 의존하지 않습니다. 만약 BookShelf를 배열에서 Vector로 변경한다고 하더라도 hasNext 메서드와 next 메서드만 올바르게 수정한다면, 위의 while 루프를 전혀 변경하지 않아도 동작합니다.

즉, 클래스를 부품처럼 사용할 수 있게 하고, 하나의 부품을 수정해도 다른 부품에는 큰 영향 없이 작은 수정만으로 끝낼 수 있게 재이용화의 장점을 누릴 수 있습니다.
