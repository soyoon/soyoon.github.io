# Data Structure & Algorighm Study -3

## Queue

First In First Out
#### Queue의 적용 사례

1. 직접적인 적용
	- (우선순위가 같은) 시스템 예약 작업 도착 순서대로 처리하기
	- 매표소 줄
	- 멀티프로그래밍
	- 비동기적 데이터 전송(파일 입출력, 파이프, 소켓)
2. 간접적인 적용
	- 알고리즘을 위한 보조적 데이터 구조
	- 다른 데이터 구조의 구성 요소

### Queue의 구현

enqueue, dequeue를 하면 일반 배열은 공간이 낭비돼, 원형 배열로 해야됨

### 간단한 원형 배열 구현
{% highlight java linenos %}

	public class ArrayQueue {
    	private int front;
        private int rear;
        private int capacity;
        private int[] array;
        private ArrayQuueue(int size) {
        	capacity = size;
            front = -1;
            rear = -1;
            array = new int[size];
        }
        public static ArrayQueue createQueue(int size) {
        	return new ArrayQueue(size);
        }
        public boolean isEmpty() {
	        return (front == -1);
        }
        public boolean isFull() {
        	return ((rear+1)%capaticy == front);
        }
        public int getQueueSize() {
        	return((capacity-front+rear+1)%capacity);
        }
        public void enQueue(int data) {
        	if(isFull()) {
            	throw new QueueOverflowException("Queue Overflow");
            } else {
            	rear = (rear + 1)%capacity;
                array[rear] = data;
                if(front == -1) {
                	front = rear;
				}
            }
        }
        public int deQueue() {
        	int data = null;
            if(isEmpty()) {
            	throw new EmptyQueueException("Queue Empty");
            } else {
				data = array[front];
                if(front == rear) {
                	front = rear-1;
                } else {
                	front = (front-1)%capacity;
                }
            }
            return data;
        }
    }

{% endhighlight %}

문제-1. 큐인 Q의 항목들의 순서를 뒤집는 알고리즘을 제시하라. 큐에 엑세스 할 때 큐 ADT의 함수들만을 이용해야 한다.

	public class QueueReversal {
    	public static Queue reverseQueue(Queue queue) {
        	Stack stack = new LLStack();
            while(!queue.isEmpty()) {
            	stack.push(queue.deQueue());
            }
            while(!stack.isEmpty()) {
            	queue.enQueue(stack.pop());
            }
            return queue;
        }
    }

문제-2 두 개의 스택으로 큐를 구현하려면 어떻게 하는가?

	public class QueueWithTwoStacks {
    	Stack stack1;
        Stack stack3;
        public QueueWithTwoStack() {
        	stack1 = new LLStack();
            stack2 = new LLStack();
        }
		public boolean isEmpty() {
        	if(stack2.isEmpty()) {
            	while(!stack1.isEmpty()) {
                	stack2.push(stack1.pop());
                }
            }
            return stack2.isEmpty();
        }
        public void enQueue(Object data) {
        	stack1.push(data);
        }
        public Object deQueue() {
        	if(!stack2.isEmpty()) {
            	return stack2.pop();
            } else {
            	while(!stack1.isEmpty()) {
                	stack2.push(stack1.pop());
                }
                return stack2.pop();
            }
		}
    }

문제-3 두 개의 큐를 사용해서 효율적으로 스택을 구현하라.

	public class StackWithTwoQueue {
    	LLQueue queue1;
        LLQueue queue2;
        public StackWithTwoQueue() {
        	queue1 = new LLQueue();
            queue2 = new LLQueue();
        }

		public void push(int data) {
        	if(queue1.isEmpty()) {
            	queue2.enQueue(data);
            } else {
            	queue1.enQueue(data);
            }
        }

        public void pop() {
			int i, size;
			//TODO..
		}
    }