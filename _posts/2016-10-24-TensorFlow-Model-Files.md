#A Tool Developer's Guide to TensorFlow Model Files

대부분 사용자는 텐서플로우가 디스크에 데이터를 어떻게 저장하는지 내부 세부사항을 신경 쓸 필요가 없다. 하지만 만약 너가 툴 개발자라면 신경써야된다.
예를들면, 너는 모델을 분석하고 싶거나 텐서플로우와 다른 포맷간에 역변환(convert back), 정변환(convert forth)을 하고 싶을 것이다.
이 가이드는 이런 종류의 툴을 쉽게 개발하기 위해, 모델 데이터를 가지고 있는 메인 파일로 어떻게 작업하는지 대한 세부 내용을 설명하려고 한다.

Content
- A Tool Developer's Guide to TensorFlow Model Files
  - Protocol Buffers
  - GraphDef
  - Text or Binary?
  - Nodes
    - name
    - op
    - input
    - device
    -attr
  - Freezing
  - Weight Formats

### Protocol Buffers
모든 텐서플로우의 파일 포맷은 Protocol Buffers를 기본으로 한다. 그래서 Protocol buffers가 어떻게 작동하는지 익숙해지면 좋다. Summary는 너가 텍스트 파일에 데이터 구조를 정의한 것이고, protobuf tools는 친근한 방법으로 데이터를 load, save, access 할 수 있는 언어들로 클래스를 생성한다.
우리는 종종 Protocol Buffers를 protobufs  라고 말하는데, 여기서는 이 가이드대로 컨벤션을 사용하겠다.

### GraphDef
텐서플로우의 계산은 그래프 object를 토대로 이루어진다. 그래프는 operation을 나타내는 노드가 inputs outputs 처럼 서로 연결된 네트워크를 가지고 있다. 그래프 오브젝트를 생성한 뒤에, GraphDef 오브젝트를 리턴하는 as_graph_def()를 호출하여 저장한다.

GraphDef 클래스는 ProtoBuf 라이브러리에 의해 생성성된 오브젝트이다. tensorflow/core/framework/graph.proto 에 정의 되어 있다. protobuf 도구는 이 텍스트 파일을 파싱하고 그래프 definitions를 load, store, manipulate하는 코드를 생성한다.

스탠드얼론 텐서플로우 파일이 모델을 나타내는 것을 보면, protobuf 코드로 저장되어 나온 GraphDef 오브젝트 중 한 serialized 버전이 포함될 것이다.

이 생성된 코드는 디스크에서 GraphDef 파일들을 저장하고 로드하는데 사용된다. 우리가 좀 더 파볼보면서 볼 한 좋은 예는 graph_metrics.py 이다.
이 파이썬 스크립트는 저장된 graph definition을 가지고와서 모델을 분석하여 퍼포먼스와 리소스 통계를 분석한다. 실제 모델을 로드하는 코드는 다음과 같다.

'graph_def = graph_pb2.GraphDef()'

이 라인은 빈 GraphDef 오브젝트를 만든다. 이 클래스는 graph.proto의 정의로 만들어졌다.이것은 우리가 우리 파일에서 데이터로 옮기려고 하는 오브젝트이다.

'wich open(FLAGS.graph, "rb") as f:'

스크립트에 적어둔 경로의 파일을 핸들 한다.
'if FLAGS.input_binary:
  graph_def.ParseFromString(f.read())
else:
  text_format.Merge(f.read(), graph_def)'


### Text or Binary?
ProtoBuf가 저장될 수 있는 두가지 다른 포맷이 있다. TextFormat은 사람이 읽을 수 있는 형태로, 디버깅과 수정이 쉽지만 weights 같은 수와 관련된 데이터가 저장 되면 매우 커질 수 있다. 작은 예시를 [graph_run_run2.ppbtxt](https://github.com/tensorflow/tensorflow/blob/ae3c8479f88da1cd5636b974f653f27755cb0034/tensorflow/tensorboard/components/tf-tensorboard/test/data/graph_run_run2.pbtxt) 에서 볼 수 있다.

Binary format은 우리가 읽을 수 있는 형태가 아니지만, 텍스트보다는 훨씬 적은 크기로 만들어 진다. 이 스크립트에서 우리는 사용자가 flag로 input 파일을 바이너리로 할 것인지, 텍스트로 할 것인지 입력하도록 요청한다. 그리고 그에 맞는 함수를 호출한다. 큰 바이너리 파일은 [inception_dec_2015.zip archive](https://storage.googleapis.com/download.tensorflow.org/models/inception_dec_2015.zip) 에서 tensorflow_inception_graph.pb 로 볼 수 있다. 

API 자체는 헷갈릴 수 있다. binary call은 실제로는 ParseFromString()이다. 반면, textual file을 로드할 때는, text_format 모듈의 함수를 사용한다. 


### Nodes
파일을 graph_def 변수들로 로드하면, 이제 내부의 데이터에 접근 할 수 있다. 실습 목적으로, 가장 중요한 섹션은 node member에 저장되어 있는 노드들의 리스트이다. 여기 코드가 있다.

'for node in graph_def.node'

각 노드는 NodeDef의 오브젝트이다. [tensorflow/core/framework/node_def.proto](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/framework/node_def.proto) 에 정의되어 있다.
각각의 노드는 입력 연결에 따라 한 개의 연산을 정의하고, 텐서플로우의 그래프를 만드는 기본 요소가 된다. NodeDef 의 멤버와 의미는 아래와 같다.

#### name
모든 노드는 고유의 식별자를 가져야하며, 그래프의 다른 노드에서 사용되면 안된다. 만약 파이썬API를 사용하여 그래프를 생성할 때 name을 명시하지 않으면, name은 연산을 반영하고 일정하게 숫자를 늘려가며 붙인다. ex) MatMul5 처럼?..
name은 노드간의 연결을 정의하고, 그래프가 실행될 때 inputs 과 outputs을 세팅할 때 사용된다. 


#### op
op는 어떤 연산을 실행할 것인지를 정의한다. 예를들면 "Add", "MatMul", "Conv2D" 등..
그래프가 실행될 때, op 이름을 registry에서 검색하여 구현을 찾아낸다. registry는 [tensorflow/core/ops/nn_ops.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/ops/nn_ops.cc) 처럼 REGISTER_OP() 매크로에 호출하여 populated 된다. 


#### input
다른 노드의 이름의 문자열 리스트는 콜론으로 output의 포트 숫자가 따라온다. 예를들면 두개의 인풋이 있는 노드는 ["some_node_name", "another_node_name"] 처럼 리스트로 있고, ["some_node_name:0", "another_node_name:0"] 과 같은 표현이다. 
and defines the node's first input as the first output from the node with the name "some_node_name", and a second input from the first output of "another_node_name"


#### device
대부분의 경우 device는 분산 환경에서 어디에서 노드를 실행할 지, 연산을 CPU, GPU 어디에 돌게 할지 정하는 것이기 때문에, 무시해도 된다. 


#### attr
이것은 key/value 로 노드의 모든 속성을 저장한다. 이것들은 노드들의 불변의 특징이다. convolutions의 필터 사이즈나 상수의 값 같이 런타임에서 변하지 않는 값이다.
string, ints, array, tensor values 까지 매우 다양하고 많은 속성들이 있기 때문에, 이 구조들을 가지고 있는 별도로 분리된 protobuf file이 있다. [tensorflow/core/framework/attr_value.proto](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/framework/attr_value.proto)

각 속성은 unique name string을 가지고 속성들은 operation이 정의될 때 리스트화 되어야 한다. 만약 노드에 attribute가 존재하지 않으나 operation 정의에 default로 리스트 되어있는 게 있으면, default는 graph가 생성될 때 사용된다. 
모든 attribute의 member는 node.name, node.op 등으로 파이썬에서 호출할 수 있다. GraphDef에 저장된  노드리스트는 모델 구조의 전체 정의이다(?)
The list of nodes stored in the GraphDef is a full definition of the model architecture.


### Freezing
한가지 헷갈리는 부분은 weights은 일반적으로 학습 중에 file format 형태로 저장되지 않는다는 것이다. 대신, weights은 checkpoint files에 있다. 그리고 그래프의 Variable ops가 weights 값이 초기화된 가장 최신의 값을 로드한다. 이것은 production을 배포할때 파일을 분리하기에 매우 편리하지 않다. 
그래서 [freeze_graph.py](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/python/tools/freeze_graph.py) 스크립트는 그래프의 정의와 checkpoints 셋을 하나의 파일로 묶는다. 

freeze_graph가 하는 일은, GraphDef 를 로드하고, 최신 checkpoint file 로부터 변수에 값들을 넣고, attributes에 저장된 weights의 모든 Variable op를 숫자로 된 Const로 바꾼다. 그리고 forward inference에 사용되지 않는 관련 없는 노드들을 제거한다. 그리고 결과적으로 남은..? GraphDef 를 출력 파일에 저장한다.


### Weight Formats
만약 너가 신경망을 나타내는 텐서플로우 모델을 다루고 있다면, 가장 일반적인 문제중 하나는 weights의 값들을 추출하고(extracting), 표현하는(interpreting) 문제일 것이다. 일반적으로 weights을 저장하는 방법은, freeze_script로 생서한 그래프 예를들면 , Const ops처럼 wieghts을 Tensors에 넣는 것이다. 
이 방법은 [tensorflow/core/framework/tensor.proto](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/framework/tensor.proto) 에 정의되어 있다. 그리고 크기와 데이터의 종류, 값에 대한 정보를 포함시킨다. 
파이썬에서는 Const op 를 나타내는 NodeDef로 부터 some_node_def.attf['value'].tensor 같은걸 호출함으로써 TensorProto 오브젝트를 얻을 수 있다. 

이것은 weights 데이터를 나타내는 오브젝트를 제공한다. 데이터 자체는 오브젝트의 타입을 나타내는 suffix_val의 리스트 중 하나가 저장된다. 예를들어 float_val은 32비트 float data type 이다.

The ordering of convolution weight values is often tricky to deal with when converting between different frameworks. In TensorFlow, the filter weights for the Conv2D operation are stored on the second input, and are expected to be in the order [filter_height, filter_width, input_depth, output_depth], where filter_count increasing by one means moving to an adjacent value in memory.

Hopefully this rundown gives you a better idea of what's going on inside TensorFlow model files, and will help you if you ever need to manipulate them.


