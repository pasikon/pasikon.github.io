---
title: "Basic example for matrix factorization with Tensorflow in recommender engine context"
excerpt: "How to do matrix factorization with gradient descent without need of manulally implementing backprop"
last_modified_at: 2018-01-11T11:42:33-01:00
header:
  #teaser: "assets/images/opencv_logo.png"
tags: 
  - tensorflow
  - python
  - matrix factorization
toc: false
---

[^1]: http://www.quuxlabs.com/blog/2010/09/matrix-factorization-a-simple-tutorial-and-implementation-in-python/

Matrix factorization is decomposition of a matrix into a product of matrices.

This algotirhm is useful when it comes to solve recommender engine problem. 
How?

Imagine you have some `U1-U5` users that rated `D1-D4` products (columns) the following way:

```python
d = {
    'U1': [3, 5, np.nan, 2],
    'U2': [4, np.nan, np.nan, 1],
    'U3': [5, 4, np.nan, 1],
    'U4': [1, np.nan, np.nan, 3],
    'U5': [np.nan, 5, 2, 2]
}
df = pd.DataFrame(data=d, index=['D1', 'D2', 'D3', 'D4']).transpose()
mtx = df.fillna(0).as_matrix().astype(np.float32)
```


Ratings by users are represented by `mtx` matrix.

Of course as in real world not all products are rated - `np.nan` represents product with no rarting by user. Recommender engine job is to find out the ratings users would like to give to unrated products and on this basis to recommend these prodcuts to them.

Here it comes the matrix factorization algorithm, matrix `mtx` can be represented as product of two matrixes `p of shape (mtx.shape[0], k)` and `q of shape (mtx.shape[1], k).T` where `k` is number of specific attributes the users and products have. Based on these `k` attributes (features) we can predict missing ratings that is `np.nan` values in `d`.

You can easily find resources on how to do this manually using Gradient Descent and Numpy[^1]. It involves manual implementation of backpropagation steps with computation graph, derivatives etc.

Now Tensorflow as a deep learning framework is all about minimizing loss functions with tens of millions variables exactly same way. So you don't really need to implement backpropagation steps manually. 

If we define loss as a product of `p` and `q` vs `mtx`, framework will do everything for us. 

One caveat here is that we want to define loss only in regard of known ratings, that is ones which aren't `np.nan`. During the training process Tensorflow will compute `p` and `q` matrixes to fit known ratings only while the ones that aren't known will be the outcome of revealed `k` features. 

So lets mask known ratings:

```python
np_mask = df.notnull()
tf_mask = tf.Variable(np_mask.values)
```

Initialize `p` and `q` given `k` no of features:

```python
np_mask = df.notnull()
n_dim = mtx.shape[0]
m_dim = mtx.shape[1]
k = 2

p = np.random.rand(n_dim, k).astype(np.float32)
q = np.random.rand(m_dim, k).astype(np.float32)
```

Define loss function as squared root error between `p` and `q` product and `mtx` masked:

```python
p_t = tf.Variable(initial_value=p, trainable=True, name='p_matrix')
q_t = tf.Variable(initial_value=q, trainable=True, name='q_matrix')
pqdot_t = tf.matmul(p_t, q_t, transpose_a=False, transpose_b=True)
loss = tf.reduce_sum(tf.square((tf.boolean_mask(mtx_ph, tf_mask) - tf.boolean_mask(pqdot_t, tf_mask))))
```

And just let framework minimize the loss:

```python
train = gradient_step.minimize(loss)

init = tf.global_variables_initializer()

with tf.Session() as sess:
    sess.run(init)
    for step in range(5000):
        sess.run(train, feed_dict={mtx_ph: mtx})
        # sess.run(train)
        if step % 500 == 0:
            print('Step: %i loss: %f' % (step, sess.run(loss, feed_dict={mtx_ph: mtx})))

    result = sess.run(pqdot_t)
    print(result)
```

If you run this you will see that in `result` we have now values predicted where there was `np.nan` before :)

Complete script is available here: [Github](https://github.com/pasikon/matrix_factorization_tf/blob/master/matrix_fac_tf_tutorial.py)
