# CS246 - Lecture 13 - Oct 23, 2018

### Continued from last class (node invarient/encapsulation)

> list.h
```C++
#ifndef _LIST_H_
#define _LIST_H_

class List {
  struct Node; // private nested class/struct only accessible within class List
  Node *theList = nullptr;

 public:
  void addToFront(int n);
  int &ith(int i); // is a reference so client can mutate item
                   // ex. List l; l.ith(5) = 4;
  ~List();
};

#endif
```
> list.cc
```C++
#include "list.h"

struct List::Node { // Nested class
  int data;
  Node *next;

  Node (int data, Node *next): data{data}, next{next} {}
  ~Node() { delete next; }
};

List::~List() { delete theList; }

void List::addToFront(int n) { theList = new Node(n, theList); }

int &List::ith(int i) {
  Node *cur = theList;
  for (int j = 0; j < i && cur; ++j, cur = cur -> next);
  return cur->data;
}
```
Only List can create/manipulate node objects \
Therefore, we **can** guarentee the invarient that next is always nullptr or allocated by new

- But now we can't traverse the list from node to node as we would a linked list since ith() is O(n), we can't traverse the List in O(n) because we are repeatebly calling ith over and over which gives O(n^2) time.
-  We can expose the nodes but then we lose encapsulation

## SE topic: Design Patterns
- certain programming problems arise often
- keep track of good solutions to these problems - reuse + adapt them

### Soln: Iterator Pattern
- Create a class that manages access to nodes
    - Abstraction of a pointer
    - Can use to walk the list without exposing the actual pointers
  
### Aside:
```C++
for (int *p = curr; p != arr + size; p++) {
    printf("%d\n, *p); // traversing an array using pointer arithmetic
}
```

```C++
Class List {
    struct Node;
    Node *theList = nullptr;

    public:
        class Iterator {
            Node *p;

            public:
                explicit Iterator (Node *p) : p{p} {}
                int &operator *() {
                    return p->data;
                  }
                Iterator & operator ++ () {
                    p = p->next; 
                    return *this;
                  }
                bool operator != (const Iterator &other) {
                    return p != other.p;
                }
        }; // end of class Iterator - note all are O(1) so fixes our traversing problem while keeping encapsulation

        Iterator begin () {
          return Iterator {theList}; 
        }
        Iterator end () {
          return Iterator {nullptr}; // used to compare when iterating over list so we know when to stop
        }
        ... // other List methods
}
```
Example of using this from client:

```C++
int main () {
  List l;
  l.addToFront(1);
  l.addToFront(2);
  l.addToFront(3);
  
  for (List::Iterator it = &begin(); it != l.end(); ++it) {
    cout << *it << endl;  // O(n) traversal
  }
}
```
### Shortcut : Automatic Type Declarations

```C++
auto x = y; // declares x to have the same types its initializer (in this case y)
```
**Eg.**
```C++
for (auto it = l.begin(); it != l.end(); ++it) {
    cout << *it << endl;
}
```
### Shortcut : Range based for loop/For-each loop
```C++
for (auto n:l) {
  cout << n << endl; // this is only a copy of the items
}
```
### Is Avaliable for any class with:
- methods "begin" and "end" that produces an iterators (doesn't have to be called "Iterator")
- Iterator **must** support `!=`, prefix `++`, and unary `*`
- or default supported

If you want to mutate list items (or save copying - save the memory used by copying):
```C++
for (auto &n:l) {
  ++n;
}
```

## Encapsulation Continued:
List clients can create iterators directly:
```C++
auto it = List::Iterator {nullptr};
```
- violates encapsulation - client should be using begin/end

### We could:
- make Iterator's ctor private 
- then client can't call `List::Iterator {____}` -> but neither can `List`

### Soln
- give `List` priviliged access to iterator
- make it a friend

```C++
Class List {
  ...
  public:
    class Iterator {
      Node *p;
      explicit Iterator;  
      ...
      public:
      ...
        friend class Lists;
    };
};
```
Now List can still create iterators but the client can only create iterators by calling `begin()` and `end()`.

- Give your classes as few friends as possible - weakens encapsulation
- Maintaining classes becomes harder due to larger scope

### Soln:
Provide access to private fields by writing accessor/mutator methods:
```C++
class Vec {
    int x, y; 
  public:
    ...

    int getX() const { return x; }	// accessor
    void setY(int z) { y = z; }	// mutator	
// this class is the actual one that determines what to set Y to, or even do at all
}
```
`operator<<()` needs `x` and `y`, but shouldn't be a member
  - if `getX`, `getY` -okay
  - but if you don't want to provide getX or getY
    - make `operator <<` a friend fn.

> class.h
```C++
class Vec {
  ...
  friend std::ostream &operator<<(std::ostream &out, const Vec &v);
  // this is a standalone friend that has been declared, not a method
}
```
> class.cc
```C++
std::ostream &operator<<(std::ostream &out, const Vec &v) {
	return out << v.x << ' ' << v.y;
}
```

## System Modelling

- Visualize structure of the system (abstraction + relationships among them) to aid design and implementation

### Popular standard: UML (Unified Modelling Language) 


<table>
<tr>
        <th> Modelling Classes </th>
        <th>Notes/Explanation </th>
</tr>
<tr>
  <td>
    <table>
      <tr>
        <td align = center> Vec </td>
      </tr>
      <tr>
        <td align = center height = 70px> -x: Integer <br> -y: Integer </td>
      </tr>
      <tr>
        <td align = center height = 70px> + get x: Integer <br> + get y: Integer </td>
      </tr>
      </table>
    </td>
     <td>
      <table>
        <tr>
          <td align = center> Name </td>
        </tr>
        <tr>
          <td align = center height = 70px> Fields (optional) </td>
        </tr>
        <tr>
          <td align = center height = 70px> Methods (Optional) </td>
        </tr>
        </table>
      </td>
  <tr>
  </table>

- `-` means private visibility
- `+` means public visibility







