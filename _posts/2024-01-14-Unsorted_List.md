---
layout: single
title:  "[자료구조] Unsorted List"
categories: DataStructure
tag: [data_structure, c++]
toc: true
toc_sticky: true
author_profile: true
---

# Unsorted List

## List
- Linear relationship
    - 리스트의 아이템들은 순서가 있음
    - 첫번째 요소를 제외한 모든 아이템이 unique predecessor (유일한 앞선 아이템)을 가짐
    - 모든 리스트는 길이를 가짐 (리스트 아이템의 갯수)
![list](https://media.geeksforgeeks.org/wp-content/uploads/20230404164103/ArrayList_Integer_in_Java.webp)
- Unsored List
    - 아이템들이 정렬되지 않은 리스트
- Sorted List
    - 아이템들이 정렬된 리스트

## List에 필요한 Operators
- Constructor
    - 리스트 객체를 생성하는데 필요한 생성자
- Transformer
    - 리스트의 상태 (들어있는 아이템의 값, 리스트의 길이 등)을 변경
- Observer
    - 리스트의 아이템이나 길이 등의 정보를 접근
- Iterator
    - 리스트의 모든 아이템을 원하는 상태로 변경
    - 리스트의 아이템에 순서대로 접근

## Unsorted List(Array Based)
- 리스트의 아이템들이 정렬된 상태로 배열됨
- 아이템을 삽입하는 속도는 빠르나, 탐색 속도가 느림
    - 삽입 Big-O: O(1)
    - 삭제 Big-O: O(1)
    - 탐색 Big-O: O(N)

### UnsortedList Operators(Array Based)
- Transformers
    - InsertItem
    - DeleteItem
    - MakeEmpty
- Observers
    - RetrieveItem
    - LengthIs
    - IsFull
- Iterators
    - ResetList
    - GetNextItem

## Source Code
### Preprocessing
```cpp
#include <iostream>

#define MAX_ITEMS 50

using namespace std;

typedef int ItemType;
class UnsortedList;
```

### Class Definition
```cpp
class UnsortedList {
public:
    UnsortedList();
    [[nodiscard]] bool IsFull() const; // 리스트가 가득 차있는지 확인
    [[nodiscard]] bool IsEmpty() const; // 리스트가 모두 비었는지 확인
    [[nodiscard]] int LengthIs() const; // 리스트 길이 반환
    [[nodiscard]] bool RetrieveItem(const ItemType& item) const; // 리스트에 파라미터로 준 아이템이 있는지 확인
    void InsertItem(ItemType item); // 입력받은 아이템을 삽입
    void DeleteItem(ItemType item); // 입력받은 아이템과 일치하는 아이템을 삭제
    void MakeEmpty(); // 현재 리스트의 모든 요소 삭제
    void ResetList(); // 아이템을 가리키는 CurrentPos를 0으로 초기화
    ItemType GetNextItem(); // 다음 위치의 아이템을 반환

private:
    int length;
    ItemType info[MAX_ITEMS]{};
    int currentPos;
};
```

### Class Constructor
```cpp
UnsortedList::UnsortedList() {
    length = 0;
    currentPos = 0;
}
```

### Class Transformer
```cpp
void UnsortedList::InsertItem(ItemType item) {
    info[length++] = item;
}
```

```cpp
void UnsortedList::DeleteItem(ItemType item) {
    for (int i = 0; i < length; i++) { // list 길이 만큼 일치하는 아이템이 있는지 탐색
        if (info[i] == item) { // 일치하는 아이템이 있으면
            info[i] = info[length-1]; // 맨 마지막 요소를 해당 위치로 옮김
            length--; // 맨 마지막 요소를 지웠다 가정한 후 길이를 줄임
        }
    }
}
```

```cpp
void UnsortedList::MakeEmpty() {
    length = 0; // length를 0으로 만들어 리스트가 빈 것 처럼 만듦
}
```

### Class Observer
```cpp
bool UnsortedList::RetrieveItem(const ItemType& item) const {
    for (int i = 0; i < length; i++) { // list 길이 만큼 일치하는 아이템이 있는지 탐색
        if (info[i] == item) { // 일치하는 아이템이 있으면
            return true; // true 반환
        }
    }
    return false; // 일치하는 아이템을 찾지 못하면 false 반환
}
```

```cpp
int UnsortedList::LengthIs() const {
    return length;
}
```

```cpp
bool UnsortedList::IsFull() const {
    return length == MAX_ITEMS;
}
```

```cpp
bool UnsortedList::IsEmpty() const {
    return length == 0;
}
```

### Class Iterator
```cpp
void UnsortedList::ResetList() {
    currentPos = 0;
}
```

```cpp
ItemType UnsortedList::GetNextItem() {
    return info[currentPos++];
}
```

### Main Function
```cpp
int main() {
    UnsortedList list;

    for (int i = 1; i <= 3; i++) {
        list.InsertItem(i * 10);
    }

    cout << "Length of list: " << list.LengthIs() << endl;

    if (list.IsFull()) {
        cout << "The list is full." << endl;
    }
    else {
        cout << "The list is not full." << endl;
    }

    int item = 20;
    if (list.RetrieveItem(item)) {
        cout << "Item " << item << " found in the list." << endl;
    }
    else {
        cout << "Item " << item << " not found in the list." << endl;
    }

    list.DeleteItem(20);
    cout << "Item 20 deleted." << endl;
    cout << "Length of list after deletion: " << list.LengthIs() << endl;

    list.ResetList();
    cout << "Items in the list: ";
    for (int i = 0; i < list.LengthIs(); i++) {
        cout << list.GetNextItem() << " ";
    }
    cout << endl;

    list.MakeEmpty();
    cout << "Length of list: " << list.LengthIs() << endl;

    return EXIT_SUCCESS;
}
```

### Result
```
Length of list: 3
The list is not full.
Item 20 found in the list.
Item 20 deleted.
Length of list after deletion: 2
Items in the list: 10 30
Length of list: 0
```

#### 참고문헌
- Nell Dale. (2016). "C++ Plus Data Structues Sixth Edition". Jones&Bartlett Learning.
- GeeksforGeeks. (2024). "ArrayList in Java". [https://www.geeksforgeeks.org/arraylist-in-java/](https://www.geeksforgeeks.org/arraylist-in-java/).
- OpenAI. (2024). ChatGPT(Jan 10, 2024). GPT-4. [https://chat.openai.com](https://chat.openai.com).
- meongju0o0. (2024). "unsorted_list.cpp". [https://github.com/meongju0o0/meongju0o0-data-structure](https://github.com/meongju0o0/meongju0o0-data-structure).