---
layout: post
title:  "Data Structure & Algorighm Study -4"
date:   2016-04-17 22:00:00
author: Soyoon
categories: Etc
tags: Algorithm DS
---

# Data Structure & Algorighm Study -4

## Priority Queue

### 우선순위 큐란?

- 여러 개의 항목 중에서 최소 값/ 최대 값을 찾아야 할 때.
- ADT (Abstract Data Type) : Insert, DeleteMin, DeleteMax
- 항목들이 추가되는 순서와 처리되는 순서가 다름
- 정해진 우선순위에 따라 처리되는 작업 스케쥴링이 우선순위 큐

### 우선순위 큐의 적용

- 데이터 압축 : 허프만(Huffman) 알고리즘
- 최단 거리 알고리즘 : 다익스트라(Dijkstra) 알고리즘
- 최소 신장 트리 알고리즘 : 프림(Prim)의 알고리즘
- 이벤트 지향 시뮬레이션 : 줄 서 있는 고객들
- 선택 문제 : k번째 작은 항목 찾기

### 우선순위 큐의 구현

---
구현 방법:	삽입 복잡도	삭제 복잡도
정렬되지 않은 배열 구현:	O(1)	O(n)
정렬되지 않은 리스트 구현:	O(1)	(n)
정렬된 배열 구현:	O(n)	O(1)
정렬된 리스트 구현:	O(n)	O(1)
이진 검색 트리 구현:	O(logn)	O(logn)
균형 이진 검색 트리 구현:	O(logn)	O(logn)
이진 힙 구현:	O(logn)	O(1)
---

### Heap 이란?

- 노드의 값이 그 자식 노드의 값보다 크거나(Max Heap) 작아야(Min Heap) 한다.
- 트리의 높이 h>0 일 때 모든 Leaf 노드들이 h 혹은 h-1 레벨에 있어야 한다. ( 완전 이진 트리여야 한다.)

### Binary Heap

	public class Heap {
    	public int[] array;
        public int capacity;
        public int heap_type;
        public Heap(int capacity, int heap_type) {
        	this.heap_type = heap_type;
            this.count = 0;
            this.capacity = capacity;
            this.array = new int[capaticy];
        }
        public int Parent(int i) {
        	if(i <= 0 || i >= this.count)
            	return -1;
            return i-1/2;
        }
        public int LeftChild(int i) {
        	int left = 2 * i + 1;
            if(left >= this.count)
            	return -1;
            return left;
        }
        public int RightChild(int i) {
        	int right = 2 * i + 2;
            if(right >= this.count)
            	return -1;
            return right;
        }
        public int GetMaximum() {
        	if(this.count == 0)
            	return -1;
            return this.array[0];
        }

        public void PercolateDown(int i) {
        	int l, r, max, tmp;
            l = LeftChile(i);
            r = rightChile(i);
            if(l != -1 && this.array[l] > this.array[i])
            	max = l;
            else
            	max = i;
            if( r != -1 && this.array[r] > this.array[max])
            	max = r;
			if(max != i) {
            	tmp = this.array[i];
                this.array[i] = this.array[max];
                this.array[i] = tmp;
            }
            PercolateDown(max);
        }

        int DeleteMax() {
        	if(this.count == 0)
            	return -1;
            int data = this.array[0];
            this.array[0] = this.array[this.count-1];
            this.count--;
            PercolateDown(0);
            return data;
        }
        int Insert(int data) {
        	int i;
            if(this.count == this.capacity) {
            	ResizeHeap();
            }
            this.count++;
            i = this.count -1;
            while(i >=0 && data > this.array[(i-1)/2]) {
            	this.array[i] = this.array[(i-1)/2];
                i = i-1/2;
            }
            this.array[i] = data;
        }
        void ResizeHeap() {
        	int[] array_old = new int[this.capacity];
            System.arraycopy(this.array, 0, array_old, this.count[t-1]);
            this.array = new int[this.capacity *2];
            if(this.array == null) {
            	System.out.println("Memory Error");
                return;
            }
            for(int i=0; i< this.capacity; i++) {
            	this.array[i] = array_old[i];
            }
            this.capacity *= 2;
            array_old = null;
        }
        void DestroyHeap() {
        	this.count = 0;
            this.array = null;
        }
    }

출처 : 다양한 예제로 학습하는 데이터 구조와 알고리즘 for Java
