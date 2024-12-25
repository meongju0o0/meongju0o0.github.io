---
layout: single
title:  "[자료구조] Linked List 1편, Singly Linked List"
categories: DataStructure
tag: [data_structure, c++]
toc: true
toc_sticky: true
author_profile: true
---

# Singly Linked List

## 링크드 리스트(Linked List)
- Sorted List에서 삽입, 삭제 연산을 빠르게 하기 위함
- Sorted List이어도 삽입, 삭제 연산보다 탐색 연산이 빈번하면 Array가 더 유리한 경우가 많음
- array는 연속적인 메모리 주소 마다 item을 저장하는 한편, linked list는 노드 당 1개의 item을 저장하고 각 노드는 다음 노드로 갈 수 있는 포인터를 가지고 있음
- 이때, n번재 item을 탐색하기 위해서는 1번째 포인터, 2번째 포인터, n-1번째 포인터를 모두 거쳐야 하므로 탐색에서는 불리
    - iterator 혹은 pointer를 통한 삽입, 삭제 연산에서는 유리
    - item 값을 통한 탐색 후 삽입, 삭제가 발생하는 경우 시간복잡도 증가

![doubly_linked_list](/images/2024-08-03-LinkedList1/Doubly-Linked-List-in-Data-Structure.webp)

## 링크드 리스트(Linked List)에 필요한 Operator
- Constructor
    - Linked List 객체 생성을 위한 생성자
- Destructor
    - Linked List 객체 삭제를 위한 소멸자
- Transformer
    - Linked List의 상태를 변환하는 변환자
- Observer
    - Linked List의 아이템이나 길이 등의 정보에 접근하기 위한 관측자

## Singly Linked List(SLL)
- Doubly Linked List에 앞서 특수한 상황을 위한 Linked List
- Doubly Linked List는 현재 노드의 전, 후 모두 이동 가능하나, Singly Linked List는 현재 노드의 후로만 이동 가능

![singly_linked_list](/images/2024-08-03-LinkedList1/LLdrawio.png)

## Singly Linked List Search Algorithm
- 위 그림에서 Head를 현재 노드의 Next 포인터를 따라 한 칸씩 이동시키며 탐색

- 3번 째 item을 찾고자 한다면 아래와 같이 코드를 작성할 수 있을 것
    - Node라는 별도의 구조체와 SLL이라는 클래스는 이미 있다고 가정하자

```cpp
Node* cur_node = SLL.get_head(); // cur_node->item == 'A'
for(int i = 0; i < 2; i++) {
    cur_node = cur_node->next;
}
// 0번째 iteration 종료 시, cur_node->item == 'B'
// 1번째 iteration 종료 시, cur_node->item == 'C'
cout << cur_node->item;
```

## Singly Linked List Insertion Algorithm
### Basic Idea
- 새로운 노드를 주어진 pos(포인터 혹은 이터레이터) 다음 위치에 넣으려한다
    - 'B' 노드가 주어진 pos라 하자
    - 'B' 노드 다음에 삽입한다는 것은 'B'와 'C' 사이에 삽입한다는 것을 의미한다
- 우선 새로운 객체를 생성한다
- 'B'노드의 next 포인터는 새로운 객체를 가리켜야 하고
- 새로운 객체의 next 포인터는 'C'노드를 가리켜야 한다
- 근데 'B'노드의 next 포인터를 먼저 새로운 객체를 가르키니 'C' 노드의 행방이 묘연해졌다.
- 이에 대한 2가지 솔루션이 있다
    - Solution 1.
        - 무지성으로 접근하면 temp_ptr을 생성해서 'C' 노드로의 포인터를 저장
    - Solution 2.
        - 반대로 새로운 객체의 next 포인터가 'C'를 먼저 가르키게 한 뒤 'B' 노드의 next포인터가 새로운 객체를 가르키게 한다
    - 당연히 'Solution 2'가 교과서적인 접근이다.

![sll_insert](/images/2024-08-03-LinkedList1/SLL_Insert.png)

- '너무 쉽고도 당연한 solution 아닌가?' 라고 생각될 수 있으나, Doubly Linked List의 Insertion 알고리즘에도 동일하게 적용되는 아이디어이다.
- 막상 Doubly Linked List를 보면 Insertion의 순서가 마냥 쉽게 떠올르진 않을 것이다.

### Step by Step
#### Step 1. 새로운 데이터를 저장할 노드 생성
```cpp
Node* new_node = new Node(); // 새로운 노드 생성
new_node->item = new_item; // 새로운 노드에 주어진 아이템 삽입
```

#### Step 2. 주어진 위치의 다음 위치에 노드 삽입
```cpp
Node* cur = pos.GetCurrent(); // 주어진 pos의 포인터 저장
new_node->next = cur->next; // 새로운 노드의 next 포인터는 현재 노드의 다음 노드를 지칭
cur->next = new_node; // 현재 노드의 next 포인터는 새로은 노드를 지칭
```

### Source Code
```cpp
void SLL::Insert(Iterator pos, ItemType new_item) {
    if (IsFull()) {
        cerr << "List is Full, bad_alloc exception" << endl;
        return;
    }

    // pos == End()인 경우, 즉 current == nullptr이라면
    // "pos 다음에 삽입"이 불가능하므로 예외적으로 처리
    if (pos.GetCurrent() == nullptr) {
        cerr << "Iterator is End(), cannot insert after the end of the list." << endl;
        return;
    }

    Node* new_node = new Node(); // 새로운 노드 생성
    new_node->item = new_item; // 새로운 노드에 주어진 아이템 삽입

    // "pos 다음 위치"에 삽입
    Node* cur = pos.GetCurrent(); // 주어진 pos의 포인터 저장
    new_node->next = cur->next; // 새로운 노드의 next 포인터는 현재 노드의 다음 노드를 지칭
    cur->next = new_node; // 현재 노드의 next 포인터는 새로은 노드를 지칭

    size++;
}
```

## Singly Linked List Deletion Algorithm
### Basic Idea
- 주어진 pos(포인터 혹은 이터레이터)의 노드를 삭제하려 한다
- 우선 삭제할 위치의 pos를 함수 파라미터로 받아온다
- 'C' 노드를 삭제하려면 우선 'B' 노드가 'D' 노드를 가리켜야 한다
    - 이때, 'C' 노드를 삭제하려면 'B' 노드를 가지고 있어야 한다
    - 이에 대한 Solution은 2가지가 존재한다
    - Solution 1.
        ```cpp
        Node* prev_node = head;
        while (prev_node->next != cur_node) {
            prev_node = prev_node->next;
        }
        prev_node->next = cur_node->next;
        delete cur_node;
        ```
        - 'prev_node'의 next 포인터가 'C'를 위치하면 반복문을 종료한다
        - 위 예시에서 'cur_node'는 'C'이다
        - 반복문을 통해 'B' 노드의 next 포인터가 'C'를 가르키면 'prev_node'가 'B' 노드이게 된다
        - 그러나, 'Solution 1'을 사용했더니 아이템 삭제 시마다 $O(N)$ 복잡도가 발생한다
        - **Linked List의 장점이 사라졌다**

    - **Solution 2.**
        ```cpp
        Node* prev_node = pos.GetPrevious();
        prev_node->next = cur_node->next;
        delete cur_node;
        ```
        - 'pos'는 iterator 객체이다
        - iterator를 통해 'pos' 이전 객체의 정보를 관리하면 반복문이 불필요하다
        - 이때, Singly Linked List는 어짜피 한 칸씩만 앞으로 이동 가능하므로 previous도 추적하여 관리가 가능하다

## Singly Linked List Iterator Algorithm
### Class Definition
```cpp
    class Iterator { // SLL 객체의 아이템 접근을 위한 iterator
    private:
        Node* current;
        Node* previous;

    public:
        explicit Iterator(Node* node); // iterator 객체 생성자
        SLL::Iterator& operator=(const Iterator& other); // iterator 복사 연산자
        ItemType& operator*() const; // SLL Node의 item 접근을 위한 연산자 오버로딩
        Iterator& operator++(); // SLL iterator가 다음 노드를 지시하도록 하기 위한 연산자 오버로딩
        bool operator==(const Iterator& other) const; // SLL iterator가 지시하는 노드가 서로 동일한지 확인하는 연산자
        bool operator!=(const Iterator& other) const; // SLL iterator가 지시하는 노드가 서로 다른지 확인하는 연산자
        [[nodiscard]] Node* GetCurrent() const; // SLL iterator가 현재 지시하고 있는 노드의 포인터를 반환
        [[nodiscard]] Node* GetPrevious() const; // 현재 지시하고 있는 노드의 이전 노드의 포인터를 반환
    };
```

### Constructor
- iterator 객체 생성자

```cpp
SLL::Iterator::Iterator(Node* node) : current(node), previous(nullptr) {}
```

### operator=
- iterator 객체 복사

```cpp
SLL::Iterator &SLL::Iterator::operator=(const Iterator &other) {
    if (this != &other) { // 자기 자신이 아닐 때만 복사 수행
        current = other.current; // 현재 노드 복사
        previous = other.previous; // 이전 노드 복사
    }
    return *this;
}
```

### operator*
- 현재 iterator가 가르키는 노드의 **데이터** 반환

```cpp
ItemType& SLL::Iterator::operator*() const {
    return current->item;
}
```

### operator++
- 현재 이터레이터가 가리키는 위치를 한칸 뒤로 이동

```cpp
SLL::Iterator& SLL::Iterator::operator++() {
    if (current != nullptr) {
        previous = current;
        // 이전 노드의 포인터를 직전의 현재 노드 정보를 활용하여 추적 관리
        // 삭제 연산에 대비하기 위함

        current = current->next; // 현재 노드 포인터 관리
    }
    return *this;
}
```

### operator==, operator!=
- 서로 다른 두 이터레이터가 같은 노드를 가리키는지 확인
- 서로 다른 두 이터레이터가 다른 노드를 가리키는지 확인

```cpp
bool SLL::Iterator::operator==(const Iterator& other) const {
    return current == other.current;
}

bool SLL::Iterator::operator!=(const Iterator& other) const {
    return current != other.current;
}
```

### GetCurrent, GetPrevious
- 이터레이터의 current노드 **포인터** 반환
- 이터레이터의 previous노드 **포인터** 반환

```cpp
Node* SLL::Iterator::GetCurrent() const {
    return current;
}

Node* SLL::Iterator::GetPrevious() const {
    return previous;
}
```

### Singly Linked List(SLL)에 필요한 Operators
- Constructor
- Destructor
- Transformers
    - insert
    - erase
- Observers
    - Iterator
        - operator*
        - operator++
        - operator==
        - operator!=
        - GetCurrent
    - IsEmpty
    - IsFull
    - SizeIs

## Source Code
### Preprocessing
```cpp
#include <iostream>

using namespace std;

typedef int ItemType;
struct Node;
class SLL;
```

### Node Struct
```cpp
struct Node {
    ItemType item = 0;
    Node* next = nullptr;
};
```

### Class Definition
```cpp
class SLL {
private:
    Node* head = nullptr;
    size_t size = 0;

public:
    class Iterator { // SLL 객체의 아이템 접근을 위한 iterator
    private:
        Node* current;
        Node* previous;

    public:
        explicit Iterator(Node* node); // iterator 객체 생성자
        SLL::Iterator& operator=(const Iterator& other); // iterator 복사 연산자
        ItemType& operator*() const; // SLL Node의 item 접근을 위한 연산자 오버로딩
        Iterator& operator++(); // SLL iterator가 다음 노드를 지시하도록 하기 위한 연산자 오버로딩
        bool operator==(const Iterator& other) const; // SLL iterator가 지시하는 노드가 서로 동일한지 확인하는 연산자
        bool operator!=(const Iterator& other) const; // SLL iterator가 지시하는 노드가 서로 다른지 확인하는 연산자
        [[nodiscard]] Node* GetCurrent() const; // SLL iterator가 현재 지시하고 있는 노드의 포인터를 반환
        [[nodiscard]] Node* GetPrevious() const; // 현재 지시하고 있는 노드의 이전 노드의 포인터를 반환
    };

    explicit SLL(ItemType root_item);
    ~SLL();
    [[nodiscard]] bool IsFull() const; // SLL이 가득 차 있는지 확인
    [[nodiscard]] bool IsEmpty() const; // SLL이 비어있는지 확인
    [[nodiscard]] size_t SizeIs() const; // SLL의 크기 반환
    void Insert(Iterator pos, ItemType new_item); // SLL에 아이템 삽입
    void Erase(Iterator pos); // SLL의 아이템 삭제

    [[nodiscard]] Iterator Begin() const { return Iterator(head); } // SLL의 처음 시작위치 반환
    [[nodiscard]] Iterator End() const { return Iterator(nullptr); } // SLL의 마지막 위치 반환(nullptr)
};
```

### Iterator Implementation
```cpp
SLL::Iterator::Iterator(Node* node) : current(node), previous(nullptr) {}

SLL::Iterator &SLL::Iterator::operator=(const Iterator &other) {
    if (this != &other) { // 자기 자신이 아닐 때만 복사 수행
        current = other.current; // 현재 노드 복사
        previous = other.previous; // 이전 노드 복사
    }
    return *this;
}

ItemType& SLL::Iterator::operator*() const {
    return current->item;
}

SLL::Iterator& SLL::Iterator::operator++() {
    if (current != nullptr) {
        previous = current;
        // 이전 노드의 포인터를 직전의 현재 노드 정보를 활용하여 추적 관리
        // 삭제 연산에 대비하기 위함

        current = current->next; // 현재 노드 포인터 관리
    }
    return *this;
}

bool SLL::Iterator::operator==(const Iterator& other) const {
    return current == other.current;
}

bool SLL::Iterator::operator!=(const Iterator& other) const {
    return current != other.current;
}

Node* SLL::Iterator::GetCurrent() const {
    return current;
}

Node* SLL::Iterator::GetPrevious() const {
    return previous;
}
```

### SLL Constructor
```cpp
SLL::SLL(const ItemType root_item) {
    head = new Node();
    head->item = root_item;
    head->next = nullptr;
    size++;
}
```

### SLL Destructor
```cpp
SLL::~SLL() {
    Node* cur_node = head;
    while(cur_node != nullptr) {
        Node* temp_ptr = cur_node->next;
        delete cur_node;
        cur_node = temp_ptr;
    }
}
```

### SLL Observer
```cpp
bool SLL::IsFull() const {
    try {
        const Node* dummy_node = new Node(); // dummy_node가 생성 가능하면 false 반환
        delete dummy_node;
        return false;
    }
    catch(const bad_alloc& e) { // dummy_node가 생성되지 않고 bad_alloc exception 발생
        return true; // true 반환
    }
}
```

```cpp
bool SLL::IsEmpty() const {
    return head == nullptr;
}
```

```cpp
size_t SLL::SizeIs() const {
    return size;
}
```

### SLL Transformer
```cpp
void SLL::Insert(Iterator pos, ItemType new_item) {
    if (IsFull()) {
        cerr << "List is Full, bad_alloc exception" << endl;
        return;
    }

    // pos == End()인 경우, 즉 current == nullptr이라면
    // "pos 다음에 삽입"이 불가능하므로 예외적으로 처리
    if (pos.GetCurrent() == nullptr) {
        cerr << "Iterator is End(), cannot insert after the end of the list." << endl;
        return;
    }

    Node* new_node = new Node(); // 새로운 노드 생성
    new_node->item = new_item; // 새로운 노드에 주어진 아이템 삽입

    // "pos 다음 위치"에 삽입
    Node* cur = pos.GetCurrent(); // 주어진 pos의 포인터 저장
    new_node->next = cur->next; // 새로운 노드의 next 포인터는 현재 노드의 다음 노드를 지칭
    cur->next = new_node; // 현재 노드의 next 포인터는 새로은 노드를 지칭

    size++;
}
```

```cpp
void SLL::Erase(Iterator pos) {
    if (IsEmpty() || pos.GetCurrent() == nullptr) {
        cerr << "List is empty or invalid iterator, nothing to erase." << endl;
        return;
    }

    // pos == Begin()인 경우, 즉 current == head이라면
    if (pos.GetCurrent() == head) {
        Node* temp_ptr = head; // 기존 head 포인터 임시 저장
        head = head->next; // head 포인터를 head->next로 수정
        delete temp_ptr; // 기존 head 삭제(메모리 할당 해제)
    }

    // pos != Begin()인 경우, 즉 current != head이라면
    else {
        Node* prev_node = pos.GetPrevious(); // 현재 노드의 이전 노드 포인터 저장

        // 이전 노드가 nullptr이 아니라면 삭제 연산 수행
        // (head가 아니면서 이전 노드가 nullptr일 순 없으나, 만일의 경우를 대비한 예외처리)
        if (prev_node != nullptr) {
            prev_node->next = pos.GetCurrent()->next;
            // 이전 노드의 next 포인터가 현재 노드의 다음 노드 지칭하도록 수정
            // 현재 노드를 건너뛰게됨

            delete pos.GetCurrent(); // 현재 노드 삭제
        }
    }
    size--;
}
```

### Main Function
```cpp
int main() {
    SLL list(5);

    SLL::Iterator it1 = list.Begin();
    list.Insert(it1, 10);
    list.Insert(++it1, 15);
    list.Insert(++it1, 20);

    cout << "List Size: " << list.SizeIs() << endl;

    for (SLL::Iterator it = list.Begin(); it != list.End(); ++it) {
        cout << *it << " ";
    }
    cout << endl;

    SLL::Iterator it2 = list.Begin();
    ++it2; // 두 번째 노드(10)로 이동
    SLL::Iterator it3 = it2; // operator=()로 상태 복사
    ++it3; // 복사된 이터레이터로 다음 노드(15)로 이동

    cout << "Current item in it2: " << *it2 << endl; // 10
    cout << "Current item in it3: " << *it3 << endl; // 15

    return EXIT_SUCCESS;
}
```

### Result
```
List Size: 4
5 10 15 20
Current item in it2: 10
Current item in it3: 15
```

#### 참고문헌
- Nell Dale. (2016). "C++ Plus Data Structues Sixth Edition". Jones&Bartlett Learning.
- GeeksforGeeks. (2024). "Singly Linked List Data Structure". [https://www.geeksforgeeks.org/applications-of-linked-list-data-structure/](https://www.geeksforgeeks.org/applications-of-linked-list-data-structure/).
- GeeksforGeeks. (2024). "What are real life exaples of double linked list". [https://www.geeksforgeeks.org/what-are-real-life-examples-of-double-linked-list/](https://www.geeksforgeeks.org/what-are-real-life-examples-of-double-linked-list/).
- OpenAI. (2024). ChatGPT(Aug 8, 2024). GPT-4o. [https://chat.openai.com](https://chat.openai.com).
- meongju0o0. (2024). "singly_linked_list.cpp". [https://github.com/meongju0o0/meongju0o0-data-structure](https://github.com/meongju0o0/meongju0o0-data-structure).