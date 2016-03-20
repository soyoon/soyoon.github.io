#2016.03.08 Thread pool
출처 : [예제로 배우는 C#프로그래밍](http://www.csharpstudy.com/Threads/thread.aspx)
##Thread
Thread를 분리해서 작업할 때와 하나의 Thread로 작업할  때의 장단점.
- Main Thread로 모든 작업을 하게 되면, 화면이 정지되는 (UI Freezing) 현상이 발생. -> 마우스와 키보드 먹통
- 외부 네트워크 데이터를 가져온다던지, 디스크에 큰 데이터를 읽거나 쓰는 일 들에는 별도의 작업쓰레드를 사용하는 것이 좋다.

##비동기 델리게이트(Asynchronous delegate)
Thread에 여러 프로세스를 실행시킬때 종료되었는지 체크할 수 있는 방법 : Asynchronous Delegate

####비동기 델리게이트(Asynchronous delegate) 
비동기 델리게이트는 쓰레드풀의 쓰레드를 사용하는 한 방식으로, 메서드 델리게이트( Delegate, 대리자 ) 의 BeginInvoke() 를 사용하여 쓰레드에게 작업을 시작하게 하고, EndInvoke()를 사용하여 해당 작업이 끝날 때까지 기다려서 리턴 값을 넘겨 받게 된다. 

BeginInvoke()는 쓰레드를 구동시킨 후 IAsyncResult 객체를 리턴하고 즉시 메인쓰레드의 다음 문장을 실행하게 된다. IAsyncResult 객체는 차후 EndInvoke() 등과 같은 메서드를 실행할 때 파라미터로 전달되는 것으로서 어떤 쓰레드를 가리키는 지를 알 수 있게 한다. 