---
layout: single
title:  "자료구조, Linked List 1편, Doubly Linked List"
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

![doubly_linked_list](/images/2024-08-03-LinkedList1/Doubly-Linked-List-in-Data-Structure.webp)