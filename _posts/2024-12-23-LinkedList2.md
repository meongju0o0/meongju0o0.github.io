---
layout: single
title:  "[자료구조] Linked List 2편, Doubly Linked List"
categories: DataStructure
tag: [data_structure, c++]
toc: true
toc_sticky: true
author_profile: true
---

# Doubly Linked List

## 링크드 리스트(Linked List)
- Sorted List에서 삽입, 삭제 연산을 빠르게 하기 위함
- Sorted List이어도 삽입, 삭제 연산보다 탐색 연산이 빈번하면 Array가 더 유리한 경우가 많음
- array는 연속적인 메모리 주소 마다 item을 저장하는 한편, linked list는 노드 당 1개의 item을 저장하고 각 노드는 다음 노드로 갈 수 있는 포인터를 가지고 있음
- 이때, n번재 item을 탐색하기 위해서는 1번째 포인터, 2번째 포인터, n-1번째 포인터를 모두 거쳐야 하므로 탐색에서는 불리
    - iterator 혹은 pointer를 통한 삽입, 삭제 연산에서는 유리
    - item 값을 통한 탐색 후 삽입, 삭제가 발생하는 경우 시간복잡도 증가

![doubly_linked_list](/images/2024-12-23-LinkedList2/DLL1.webp)

## 링크드 리스트(Linked List)에 필요한 Operator
- Constructor
    - Linked List 객체 생성을 위한 생성자
- Destructor
    - Linked List 객체 삭제를 위한 소멸자
- Transformer
    - Linked List의 상태를 변환하는 변환자
- Observer
    - Linked List의 아이템이나 길이 등의 정보에 접근하기 위한 관측자

## Doubly Linked List(DLL)
- Singly Linked List는 현재 노드에서 다음 노드로만 이동이 가능했음
- Doubly Linked List는 현재 노드의 이전 노드와 다음 노드 모두 이동이 가능
- 즉, Doubly Linked List는 양 옆으로 이동이 가능하다

## Doubly Linked List vs. Singly Linked List
### Singly Linked List
![doubly_linked_list](/images/2024-12-23-LinkedList2/SLL.png)

### Doubly Linked List
![doubly_linked_list](/images/2024-12-23-LinkedList2/DLL2.png)

## Doubly Linked List Search Algorithm
- 위 그림에서 Head를 현재 노드의 Next 포인터를 따라 한 칸씩 이동시키며 탐색

- 3번 째 item~~('C')~~을 찾고자 한다면 아래와 같이 코드를 작성할 수 있을 것
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

## 참고문헌
- Nell Dale. (2016). “C++ Plus Data Structues Sixth Edition”. Jones&Bartlett Learning.
- GeeksforGeeks. (2024). “Why use a Doubly Linked List?”. [https://www.geeksforgeeks.org/why-use-a-doubly-linked-list/](https://www.geeksforgeeks.org/why-use-a-doubly-linked-list/).
- GeeksforGeeks. (2024). “Operations of Doubly Linked List with Implementation”. [https://www.geeksforgeeks.org/doubly-linked-list-tutorial/](https://www.geeksforgeeks.org/doubly-linked-list-tutorial/).
- OpenAI. (2024). ChatGPT(Dec 13, 2024). GPT-4o. https://chat.openai.com.