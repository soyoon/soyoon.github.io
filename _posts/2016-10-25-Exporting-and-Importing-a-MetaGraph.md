---
layout: post
title:  "Exporting and Importing a MetaGraph"
date:   2016-10-25 22:00:00
author: Soyoon
categories: ML
tags: Tensorflow
---

# Exporting and Importing a MetaGraph

MetaGraph는 텐서플로우의 GraphDef 뿐 아니라 cross process 범주의 그래프에서 계산을 할 때 필요한 관련된 메타데이터들을 포함하고 있다.
MetaGraph는 그래프의 장기 저장(Long term storage)에 사용된다. MetaGraph는 이전 학습된 그래프에서 Train, evaluation, inference를 계속하는데 필요한 정보를 포함하고 있다.

exporting과, 완성된 모델의 importing API는 [tf.trainSaver](https://www.tensorflow.org/versions/r0.11/api_docs/python/state_ops.html#Saver) 에서 클래스 [export_meta_graph](https://www.tensorflow.org/versions/r0.11/api_docs/python/train.html#export_meta_graph)와 [import_meta_graph](https://www.tensorflow.org/versions/r0.11/api_docs/python/train.html#import_meta_graph)에 있다.

##What's in a MetaGraph
MetaGraph에 들어있는 정보는 [MetaGraphDef](https://github.com/tensorflow/tensorflow/blob/r0.11/tensorflow/core/protobuf/meta_graph.proto) 프로토콜 버퍼라고 표현된다. MetaGraphDef는 아래의 필드를 가지고 있다.
- [MetaInfoDef](https://github.com/tensorflow/tensorflow/blob/r0.11/tensorflow/core/protobuf/meta_graph.proto) : 버전이나 유저 정보를 나타내는 메타 정보
- [GraphDef](https://github.com/tensorflow/tensorflow/blob/r0.11/tensorflow/core/framework/graph.proto) : 그래프를 설명
- [SaverDef](https://github.com/tensorflow/tensorflow/blob/r0.11/tensorflow/core/protobuf/saver.proto) : saver..
- [CollectionDef](https://github.com/tensorflow/tensorflow/blob/r0.11/tensorflow/core/protobuf/meta_graph.proto) : Variables, QueueRunners 등 같은 모델의 추가적인 요소들을 설명하는 Map. Python 오브젝트들이 MetaGraphDef로부터 그리고 MetaGraphDef로 직렬화되기 위해, Python 클래스는 to_proto()와 from_proto()를 구현해야 한다. 그리고 register_proto_function을 이용해서 시스템에 등록해야 한다.

예를 들면

    def to_proto(self):
      '''Converts a `Variable` to a `VariableDef` protocol buffer.

      Returns:
        A `VariableDef` protocol buffer.
      '''
      var_def = variable_pb2.VariableDef()
      var_def.variable_name = self._variable.name
      var_def.initializer_name = self.initializer.name
      var_def.snapshot_name = self._snapshot.name
      if self._save_slice_info:
        var_def.save_slice_info_def.MergeFrom(self._save_slice_info.to_proto())
      return var_def

      @staticmethod
      def from_proto(variable_def):
        '''Returns a `Variable` object created from `variable_def`.'''
        return Variable(variable_def=variable_def)

      ops.register_proto_function(ops.GraphKeys.VARIABLES,
                                proto_type=variable_pb2.VariableDef,
                                to_proto=Variable.to_proto,
                                from_proto=Variable.from_proto)



## Exporting a Complete Model to MetaGraph
실행하고 있는 Model을 MetaGraph로 exporting 하는 API는 export_meta_graph() 이다.


    def export_meta_graph(filename=None, collection_list=None, as_text=False):
      """Writes `MetaGraphDef` to save_path/filename.

      Args:
        filename: Optional meta_graph filename including the path.
        collection_list: List of string keys to collect.
        as_text: If `True`, writes the meta_graph as an ASCII proto.

      Returns:
        A `MetaGraphDef` proto.
      """

collection은 어떤 파이썬 오브젝트들이라도 사용자가 unique한 id를 주면 담을 수 있고, 쉽게 검색할 수 있다. 이 오브젝트들은 train_op나 learning_rate 처럼 그래프의 특별한 operation이 될 수 있다.
사용자는 export하고 싶은 collection의 리스트를 구분지을 수 있다. 만약 collection_list 가 정해져 있지 않으면 모델 안의 모든 collections가 export 된다.

API는 직렬화 된 protocol buffer를 리턴한다. 만약 filename이 정해져 있으면, protocol buffer는 file에도 쓰여지게 된다.
아래는 전형적인 model 사용법 이다.
Exporting the default running graph:


    # Build the model
    ...
    with tf.Session() as sess:
    # Use the model
      ...
    # Export the model to /tmp/my-model.meta.
    meta_graph_def = tf.train.export_meta_graph(filename='/tmp/my-model.meta')


Export the default running graph and only a subset of the collections.


    meta_graph_def = tf.train.export_meta_graph(
    filename='/tmp/my-model.meta',
    collection_list=["input_tensor", "output_tensor"])


MetaGraph는 tf.train.Saver의 save() 함수를 자동적으로 exported 된다.


## Import a MetaGraph
그래프에 MetaGarph를 import 하는 API는 import_meta_graph() 이다.

전형적인 사용법은 아래와 같다.

1.import를 하고 scratch에서 모델을 빌드하지 않고 학습을 계속하는 방법

    ...
    # Create a saver.
    saver = tf.train.Saver(...variables...)
    # Remember the training_op we want to run by adding it to a collection.
    tf.add_to_collection('train_op', train_op)
    sess = tf.Session()
    for step in xrange(1000000):
        sess.run(train_op)
        if step % 1000 == 0:
            # Saves checkpoint, which by default also exports a meta_graph
            # named 'my-model-global_step.meta'.
            saver.save(sess, 'my-model', global_step=step)


나중에 우리는 이 scratch로 모델을 빌드 하지 않아도 저장된 meta_graph 로부터 학습을 계속할 수 있다.

    with tf.Session() as sess:
      new_saver = tf.train.import_meta_graph('my-save-dir/my-model-10000.meta')
      new_saver.restore(sess, 'my-save-dir/my-model-10000')
      # tf.get_collection() returns a list. In this example we only want the
      # first one.
      train_op = tf.get_collection('train_op')[0]
      for step in xrange(1000000):
        sess.run(train_op)

2.import하고 그래프를 extend 하는 방법

 예를 들면, 우리는 inference 그래프를 빌드하고 meta graph로 export 할 수 있다.

    # Creates an inference graph.
    # Hidden 1
    images = tf.constant(1.2, tf.float32, shape=[100, 28])
    with tf.name_scope("hidden1"):
      weights = tf.Variable(
          tf.truncated_normal([28, 128],
                              stddev=1.0 / math.sqrt(float(28))),
          name="weights")
      biases = tf.Variable(tf.zeros([128]),
                           name="biases")
      hidden1 = tf.nn.relu(tf.matmul(images, weights) + biases)
    # Hidden 2
    with tf.name_scope("hidden2"):
      weights = tf.Variable(
          tf.truncated_normal([128, 32],
                              stddev=1.0 / math.sqrt(float(128))),
          name="weights")
      biases = tf.Variable(tf.zeros([32]),
                           name="biases")
      hidden2 = tf.nn.relu(tf.matmul(hidden1, weights) + biases)
    # Linear
    with tf.name_scope("softmax_linear"):
      weights = tf.Variable(
          tf.truncated_normal([32, 10],
                              stddev=1.0 / math.sqrt(float(32))),
          name="weights")
      biases = tf.Variable(tf.zeros([10]),
                           name="biases")
      logits = tf.matmul(hidden2, weights) + biases
      tf.add_to_collection("logits", logits)

    init_all_op = tf.initialize_all_variables()

    with tf.Session() as sess:
      # Initializes all the variables.
      sess.run(init_all_op)
      # Runs to logit.
      sess.run(logits)
      # Creates a saver.
      saver0 = tf.train.Saver()
      saver0.save(sess, saver0_ckpt)
      # Generates MetaGraphDef.
      saver0.export_meta_graph('my-save-dir/my-model-10000.meta')

그리고 나중에 메타그래프를 학습하는 그래프에 extend 할 수 있다.

    with tf.Session() as sess:
      new_saver = tf.train.import_meta_graph('my-save-dir/my-model-10000.meta')
      new_saver.restore(sess, 'my-save-dir/my-model-10000')
      # Addes loss and train.
      labels = tf.constant(0, tf.int32, shape=[100], name="labels")
      batch_size = tf.size(labels)
      labels = tf.expand_dims(labels, 1)
      indices = tf.expand_dims(tf.range(0, batch_size), 1)
      concated = tf.concat(1, [indices, labels])
      onehot_labels = tf.sparse_to_dense(
          concated, tf.pack([batch_size, 10]), 1.0, 0.0)
      logits = tf.get_collection("logits")[0]
      cross_entropy = tf.nn.softmax_cross_entropy_with_logits(logits,
                                                              onehot_labels,
                                                              name="xentropy")
      loss = tf.reduce_mean(cross_entropy, name="xentropy_mean")

      tf.scalar_summary(loss.op.name, loss)
      # Creates the gradient descent optimizer with the given learning rate.
      optimizer = tf.train.GradientDescentOptimizer(0.01)

      # Runs train_op.
      train_op = optimizer.minimize(loss)
      sess.run(train_op)

3.Hyper parameter를 찾는 방법

    filename = ".".join([tf.latest_checkpoint(train_dir), "meta"])
    tf.train.import_meta_graph(filename)
    hparams = tf.get_collection("hparams")
