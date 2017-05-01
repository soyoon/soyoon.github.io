---
layout: post
title:  "Data Structure & Algorighm Study -2"
date:   2016-04-16 22:00:00
categories: Etc
---

# Data Structure & Algorighm Study -2

## Stack

### Array로 구현한 Stack

	public class ArrayStack {
    	private int top;
        private int capacity;
        private int[] array;
        public ArrayStack() {
        	capacity = 1;
            array = new int[capacity];
            top = -1;
        }
        public boolean isEmpty() {
        	return (top == -1);
        }
        public int isStackFull() {
        	return (top == capacity -1);
        }
        public void push(int data) {
        	if(isStackFull())
            	System.out.println("Stack Overflow");
            else
            	array[++top] = data;
        }
        public int pop() {
        	if(isEmpty()) {
            	System.out.println("Stack is Empty");
                return;
            } else
            	return (array[top--]);
        }
        public void deleteStack() {
        	top = -1;
        }
    }

### LinkedList로 구현한 Stack

	public class LLStack extends Stack {
		private LLNode headNode;
        public LLStack() {
        	this.headNode = new LLNode(null);
        }
        public void Push(int data) {
        	if(headNode == null) {
            	headNode = new LLNode(data);
            } else if(headNode.getData() == null) {
            	headNode.setData(data);
            } else {
            	LLNode llNode = new LLNode(data);
                llNode.setNext(headNode);
                headNode = llNode;
            }
        }
        public int top() {
        	if(headNode == null) return null;
            else return headNode.getData();
        }
        public int pop() {
        	if(headNode == null) {
            	throw new EmptyStackException("Stack empty");
            } else {
            	int data = headNode.getData();
                headNode = headNode.getNext();
                return data;
            }
        }
        public boolean isEmpty() {
        	if(headNode == null) return true;
            else return false;
        }
        public void deleteStack() {
        	headNode = null;
        }
	}

Q. 스택을 이용하여 infix를 postfix로 변환하는 알고리즘을 논하라.
    1. 스택을 만든다.
    2. for 각각의 글자 t in 입력 스트림 {
    	if(t가 피연산자)
        	출력
		else if(t가 오른쪽 괄호) {
        	왼쪽 괄호가 팝될 때까지 토큰을 팝하면서 출력 ( 왼쪽 괄호는 출력하지 않음 )
        }
        else {
        	t보다 낮은 우선순위가 나오거나 왼쪽 괄호가 나오거나 스택이 빌때까지 팝하면서 출력
            푸시 t
        }
    }
    3. 스택이 빌 때까지 팝하면서 출력
    ex) A * B - (C + D) + E 를 추적해보자 => AB*CD+-E+

Q. 스택을 이용해서 포스트픽스 연산을 수행하는 방법
	1. 포스트픽스 문자열을 왼쪽에서 오른쪽으로 읽어 들인다.
	2. 빈 스택을 초기화 시킨다.
	3. 모든 글자들이 읽힐 때까지 다음 4, 5단계를 반복한다.
	4. 읽혀진 글자가 피연산자이면 스택으로 푸시한다.
	5. 읽혀진 글자가 연산자이고 이 연산자가 단항 연산자이면 스택으로부터 한 개의 항목을 팝한다. 연산자가 이항 연산자이면 두 개의 항목을 팝한다. 항목들을 팝한 뒤에 이 항목들에 대하여 연산자를 적용한다. 이 연산 결과 retVal을 다시 스택에 푸시한다.
	6. 모든 글자가 읽혀진 다음엔 스택에 하나의 항목만 남을 것이다.
	7. 스택의 맨 위에 있는 항목을 결과로 리턴한다.
	ex) 123*+5- 로 테스트해보기
    	스택			수식
    	1 2 3 top
        1			( 2 * 3 = 6)
        1 6
        			( 1 + 6 = 7)
        7
        7	5
        			( 7 - 5 = 2)
        2

        결과 : 2

출처 : 다양한 예제로 학습하는 데이터 구조와 알고리즘 for Java
