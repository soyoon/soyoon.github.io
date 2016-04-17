#Data Structure & Algorighm Study -1

### 연결리스트 ( Linked Lists )
	- 연속된 항목들이 포인터로 연결된다.
	- 마지막 항목은 Null 을 포인트한다.
	- 프로그램이 수행되는 동안 크기가 커지거나 작아질 수 있다.
	- (시스템 메모리가 허용하는 한) 필요한 만큼 길어질 수 있다.
	- 메모리 공간을 낭비하지 않는다(하지만 포인터를 위한 추가의 메모리를 필요로 한다).
	- 개별 항목에 접근하는 접근 시간이 길다. O(n)
### 단일 연결 리스트 ( Single Linked List )
	public class ListNode {
    	private int data;
        private ListNode next;
        public ListNode(int data) {
        	this.data = data;
        }
        public void setData(int data) {
        	this.data = data;
        }
        public int getData() {
        	return data;
        }
        public void setNext(ListNode next) {
        	this.next = next;
        }
        public ListNode getNext() {
        	return this.next;
        }
    }

	int ListLength(ListNode headNode) {
    	int length = 0;
        ListNode currentNode = headNode;
        while(currentNode != null) {
        	length ++;
            currentNode = currentNode.getNext();
        }
        return length;
    }

    ListNode InsertInLinkedList(ListNode headNode, ListNode nodeToInsert, int position) {
    	if(headNode == null)
    		return nodeToInsert;
	    int size = ListLength(headNode);
	    if(position > size+1 || position <1) {
    		System.out.println("Position of node to insert is invalid. The valid inputs are 1 to " + (size + 1));
        	return headNode;
	    }

        if(position == 1) {
            nodeToInsert.setNext(headNode);
            return nodeToInsert;
        } else {
            ListNode previousNode = headNode;
            int count = 1;
            while(count < position-1) {
                previousNode = previousNode.getNext();
                count++;
            }
            ListNode currentNode = previousNode.getNext();
            nodeToInsert.setNext(currentNode);
            previousNode.setNext(nodeToStart);
        }
        return headNode;
    }

    ListNode DeleteNodeFromLinkedList(ListNode headNode, int position) {
		int size = getLinkedListLength(headNode);
		if(position > size || position < 1) {
        	System.out.println("Position of node to delete is invalid. The valid inputs are 1 to " + size);
            return headNode;
        }
        if(position == 1) {
        	ListNode currentNode = headNode.getNext();
            headNode = null;
            return currentNode;
        } else {
        	ListNode previousNode = headNode;
            int count = 1;
            while(count < position -1 ) {
            	previousNode = previousNode.getNext();
                count++;
            }
            ListNode currentNode = previousNode.getNext();
            previousNode.setNext(currentNode.getNext());
            currentNode = null;
        }
        return headNode;
    }
