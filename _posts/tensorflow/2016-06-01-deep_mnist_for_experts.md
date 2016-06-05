---
layout: blog
title: 熟練者向けディープMNIST
category: tensorflow
tags: tensorflow,機械学習,machine learning,tensorflow tutorial,翻訳,Deep MNIST
summary: TODO
author: aharada
---

TensorFlowは大規模な数値計算を行うための強力なライブラリです。タスクのうちの一つ、ディープニューラルネットワークネットワークの実装とトレーニングを行うのに優れています。このチュートリアルでは、TensorFlowモデルでディープ畳み込みMIST分類の基礎的要素を学ぶことになります。

このイントロダクションは、ニューラルネットワークとMNISTデータセットについて良く知っているものと仮定しています。もしあなたがそれらのバックグラウンドを持っていないのなら、ビギナー向けのイントロダクションを進めてください。始める前に必ずTensorFlowをインストールしてください。

# セットアップ

モデルを作る前に、最初にMNISTデータセットをロードし、TensorFlowのセッションをスタートします。

# MNISTデータをロードする

便利なことに、MNISTデータセットを自動的にダウンロードしてインポートしてくれるスクリプトが含まれています。これは 'MNIST_data' ディレクトリを作成し、データファイルを保存します。

```
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
```

この`mnist`は軽量の分類データで、NumPy配列形式のトレーニングセット、バリデーションセット、テストセットです。また、以下で使うことになる、ミニバッチを繰り返し通す関数も提供しています。

# TensorFlowのインタラクティブセッションをスタートする

Tensorflowは計算処理をするのにバックエンドでC++を使っています。このバックエンドへのコネクションのことをことをセッションと呼んでいます。TensorFlowプログラムの慣例では、最初にグラフを作成し、セッションでそれを起動します。

その代わりにTensorFlowではコードを構造的に書くのに便利な`InteractiveSession`クラスを使います。これは、グラフを生成する処理とグラフを実行する処理のインタリーブを許可します。 これは特にIPythonのように双方向のやりとりがあるときに便利です。もしインタラクティブセッションを使わなかった場合、セッションの開始やグラフの出力の前に、全部のグラフ処理を組み立てなければならない。

```
import tensorflow as tf
sess = tf.InteractiveSession()
```

## グラフの処理

Pythonで効率的な数値計算をするためには、一般的にはNumPyのように、コストが高い行列の掛け算を行えるライブラリなど、別言語で実装された非常に処理効率の良いコードを使います。残念なことに、いまだ全ての操作において、処理をPythonに戻す際に多くのオーバーヘッドが発生してしまいます。このオーバーヘッドは、GPU上で処理を走らせたい時や分散処理をしたい場合に特に悪く、データ転送するために高い処理コストがかかってしまうことが起こり得る。

また、TensorFlowはPythonの外側でこの重い処理を行いますが、ステップを踏んで、このオーバーヘッドを避けていきます。 Pythonから独立してコストの高い処理を走らせる代わりに、TensorFlowでは 完全にPythonの外側で実行される双方向操作のグラフ描写を許可します。このアプローチはTheanoやTorchを使うとの同様です。

Pythonコードの役割は、外側でグラフ処理を構築するために、グラフ処理するための部品の実行を命令することです。それでは、グラフ処理の基本的になやり方の詳細を見ていきましょう。

# ソフトマックス回帰モデルの構築

このセクションでは、単一の線形レイヤーのソフトマックス回帰モデルの構築について扱います。次のセクションでは、これを拡張した、マルチレイヤーの畳み込みネットワークを使ったSoftmax回帰を扱います。

## Placeholders

入力画像と出力するクラスのためのノードの作成することにより、グラフ処理の構築を始めます。

```
x = tf.placeholder(tf.float32, shape=[None, 784])
y_ = tf.placeholder(tf.float32, shape=[None, 10])
```

この時点では`x`と`y_`には特定の値は入っていません。むしろ、これらは`プレースホルダ`であり、TensolFlowが処理を実行するタイミングで入力されます。

入力画像`x`は二次元の浮動小数点数を持つテンソルから成り立つ。これには`[None, 784]`という抽象的な型を代入します。`784`というのはMNIST画像の数で、`None`は一括処理するサイズにともなって、いくつかの値が入ることを示します。ターゲットの出力クラスの`y_`は2次元のテンソルから成り立ち、MNIST画像がどの数字であるかを示す、one-hot(1が1つで残りが全て0である)な10次元のベクトルです。

`shape`は任意の引数ですが、テンソルのシェイプに一致していないことが原因のバグを自動的にキャッチされます。

## Variables

We now define the weights W and biases b for our model. We could imagine treating these like additional inputs, but TensorFlow has an even better way to handle them: Variable. A Variable is a value that lives in TensorFlow's computation graph. It can be used and even modified by the computation. In machine learning applications, one generally has the model parameters be Variables.

W = tf.Variable(tf.zeros([784,10]))
b = tf.Variable(tf.zeros([10]))
We pass the initial value for each parameter in the call to tf.Variable. In this case, we initialize both W and b as tensors full of zeros. W is a 784x10 matrix (because we have 784 input features and 10 outputs) and b is a 10-dimensional vector (because we have 10 classes).

Before Variables can be used within a session, they must be initialized using that session. This step takes the initial values (in this case tensors full of zeros) that have already been specified, and assigns them to each Variable. This can be done for all Variables at once.

sess.run(tf.initialize_all_variables())
Predicted Class and Cost Function

We can now implement our regression model. It only takes one line! We multiply the vectorized input images x by the weight matrix W, add the bias b, and compute the softmax probabilities that are assigned to each class.

y = tf.nn.softmax(tf.matmul(x,W) + b)
The cost function to be minimized during training can be specified just as easily. Our cost function will be the cross-entropy between the target and the model's prediction.

cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))
Note that tf.reduce_sum sums across all classes and tf.reduce_mean takes the average over these sums.

Train the Model

Now that we have defined our model and training cost function, it is straightforward to train using TensorFlow. Because TensorFlow knows the entire computation graph, it can use automatic differentiation to find the gradients of the cost with respect to each of the variables. TensorFlow has a variety of builtin optimization algorithms. For this example, we will use steepest gradient descent, with a step length of 0.5, to descend the cross entropy.

train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
What TensorFlow actually did in that single line was to add new operations to the computation graph. These operations included ones to compute gradients, compute parameter update steps, and apply update steps to the parameters.

The returned operation train_step, when run, will apply the gradient descent updates to the parameters. Training the model can therefore be accomplished by repeatedly running train_step.

for i in range(1000):
  batch = mnist.train.next_batch(50)
  train_step.run(feed_dict={x: batch[0], y_: batch[1]})
Each training iteration we load 50 training examples. We then run the train_step operation, using feed_dict to replace the placeholder tensors x and y_ with the training examples. Note that you can replace any tensor in your computation graph using feed_dict -- it's not restricted to just placeholders.

Evaluate the Model

How well did our model do?

First we'll figure out where we predicted the correct label. tf.argmax is an extremely useful function which gives you the index of the highest entry in a tensor along some axis. For example, tf.argmax(y,1) is the label our model thinks is most likely for each input, while tf.argmax(y_,1) is the true label. We can use tf.equal to check if our prediction matches the truth.

correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
That gives us a list of booleans. To determine what fraction are correct, we cast to floating point numbers and then take the mean. For example, [True, False, True, True] would become [1,0,1,1] which would become 0.75.

accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
Finally, we can evaluate our accuracy on the test data. This should be about 92% correct.

print(accuracy.eval(feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
Build a Multilayer Convolutional Network

Getting 91% accuracy on MNIST is bad. It's almost embarrassingly bad. In this section, we'll fix that, jumping from a very simple model to something moderately sophisticated: a small convolutional neural network. This will get us to around 99.2% accuracy -- not state of the art, but respectable.

Weight Initialization

To create this model, we're going to need to create a lot of weights and biases. One should generally initialize weights with a small amount of noise for symmetry breaking, and to prevent 0 gradients. Since we're using ReLU neurons, it is also good practice to initialize them with a slightly positive initial bias to avoid "dead neurons." Instead of doing this repeatedly while we build the model, let's create two handy functions to do it for us.

def weight_variable(shape):
  initial = tf.truncated_normal(shape, stddev=0.1)
  return tf.Variable(initial)

def bias_variable(shape):
  initial = tf.constant(0.1, shape=shape)
  return tf.Variable(initial)
Convolution and Pooling

TensorFlow also gives us a lot of flexibility in convolution and pooling operations. How do we handle the boundaries? What is our stride size? In this example, we're always going to choose the vanilla version. Our convolutions uses a stride of one and are zero padded so that the output is the same size as the input. Our pooling is plain old max pooling over 2x2 blocks. To keep our code cleaner, let's also abstract those operations into functions.

def conv2d(x, W):
  return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
  return tf.nn.max_pool(x, ksize=[1, 2, 2, 1],
                        strides=[1, 2, 2, 1], padding='SAME')
First Convolutional Layer

We can now implement our first layer. It will consist of convolution, followed by max pooling. The convolutional will compute 32 features for each 5x5 patch. Its weight tensor will have a shape of [5, 5, 1, 32]. The first two dimensions are the patch size, the next is the number of input channels, and the last is the number of output channels. We will also have a bias vector with a component for each output channel.

W_conv1 = weight_variable([5, 5, 1, 32])
b_conv1 = bias_variable([32])
To apply the layer, we first reshape x to a 4d tensor, with the second and third dimensions corresponding to image width and height, and the final dimension corresponding to the number of color channels.

x_image = tf.reshape(x, [-1,28,28,1])
We then convolve x_image with the weight tensor, add the bias, apply the ReLU function, and finally max pool.

h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
h_pool1 = max_pool_2x2(h_conv1)
Second Convolutional Layer

In order to build a deep network, we stack several layers of this type. The second layer will have 64 features for each 5x5 patch.

W_conv2 = weight_variable([5, 5, 32, 64])
b_conv2 = bias_variable([64])

h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
h_pool2 = max_pool_2x2(h_conv2)
Densely Connected Layer

Now that the image size has been reduced to 7x7, we add a fully-connected layer with 1024 neurons to allow processing on the entire image. We reshape the tensor from the pooling layer into a batch of vectors, multiply by a weight matrix, add a bias, and apply a ReLU.

W_fc1 = weight_variable([7 * 7 * 64, 1024])
b_fc1 = bias_variable([1024])

h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)
Dropout

To reduce overfitting, we will apply dropout before the readout layer. We create a placeholder for the probability that a neuron's output is kept during dropout. This allows us to turn dropout on during training, and turn it off during testing. TensorFlow's tf.nn.dropout op automatically handles scaling neuron outputs in addition to masking them, so dropout just works without any additional scaling.1

keep_prob = tf.placeholder(tf.float32)
h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)
Readout Layer

Finally, we add a softmax layer, just like for the one layer softmax regression above.

W_fc2 = weight_variable([1024, 10])
b_fc2 = bias_variable([10])

y_conv=tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)
Train and Evaluate the Model

How well does this model do? To train and evaluate it we will use code that is nearly identical to that for the simple one layer SoftMax network above. The differences are that: we will replace the steepest gradient descent optimizer with the more sophisticated ADAM optimizer; we will include the additional parameter keep_prob in feed_dict to control the dropout rate; and we will add logging to every 100th iteration in the training process.

cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y_conv), reduction_indices=[1]))
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
correct_prediction = tf.equal(tf.argmax(y_conv,1), tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
sess.run(tf.initialize_all_variables())
for i in range(20000):
  batch = mnist.train.next_batch(50)
  if i%100 == 0:
    train_accuracy = accuracy.eval(feed_dict={
        x:batch[0], y_: batch[1], keep_prob: 1.0})
    print("step %d, training accuracy %g"%(i, train_accuracy))
  train_step.run(feed_dict={x: batch[0], y_: batch[1], keep_prob: 0.5})

print("test accuracy %g"%accuracy.eval(feed_dict={
    x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))
The final test set accuracy after running this code should be approximately 99.2%.

We have learned how to quickly and easily build, train, and evaluate a fairly sophisticated deep learning model using TensorFlow.

1: For this small convolutional network, performance is actually nearly identical with and without dropout. Dropout is often very effective at reducing overfitting, but it is most useful when training very large neural networks. ↩
