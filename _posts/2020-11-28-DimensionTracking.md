---
layout: post
title: Dimension tracking for Conv2D layer
author: Turzo Bose
tags: DeepLearning Convolutions
subtitle: PyTorch and Tensorflow
category: blogpost
toc: true
toc_sticky: true
#icon: fa-connectdevelop
#icon-style: regular
---

Keeping track of the dimensions is a very cumbersome process if not properly understood, specially while designing the deep neural network model architectures. The stress adds up swiftly when we have to shift between different machine learning frameworks, ideally PyTorch and Tensorflow. It is crazy how although both these frameworks are ideally doing the same thing, but the syntax makes it increasingly difficult to allow for fluid transitions between them. Hence, this post is written to make the dimension tracking process simpler, for both PyTorch and Tensorflow implementations.

## PyTorch

PyTorch has the built-in **`Conv2d`** class that allows for the 2D convolutional operation over a input plane (e.g. think about an input image of shape `28 X 28 X 3 - height X width X channels`).

Lets say we want to convolve the image with a `kernel/filter of size 3` (e.g. 3 X 3) and want an output with 5 channels

The `Conv2d` class takes in the following arguments:

{% highlight python %}
nn.Conv2d(in_channels = 3, out_channels = 5,
          kernel_size = 3,
          stride = 1,
          padding = 0)    #can create non-square kernels with kernel_size=(2,3)
{% endhighlight %}

Here, the input channel comes from the input image, the output channel is defined by us, and the `stride = 1` and `padding = 0` is by default, but can be changed as per need. We can also define a non-square kernel of 2X3 by defining `kernel_size = (2,3)`.

We might be wondering what does the number of `out_channels` actually mean. It actually corresponds to the **number of kernels/filters** we want the layer to have. Lets understand this with some code:

{% highlight python %}
import torch
input_batch = torch.rand(16, 3, 100, 100) # N = Batch Size, C = Input Channels(RGB), H = Height , W = Width

conv = torch.nn.Conv2d(in_channels=3, # RGB channels

                       out_channels=7, # Number of kernels/filters this layer has

                       kernel_size=5, # Size of kernels (i. e. of size 5x5)

                       stride=1, padding=0)

print(conv.weight.size()) # 7 x 3 x 5 x 5 (7 kernels of size 5x5 having depth of 3)

print(conv(input_batch).size()) # 16 x 7 x 100 x 100 => Batch Size = 16, Channels = 7, Height = 100, Width = 100
{% endhighlight %}

Now, let us calculate how the dimensions are calculated. The formula for the dimensionality is:

<p align="center">$$ Dimensions = \lfloor\dfrac{N+2p-f}{s} + 1\rfloor $$</p>

Hence, with a stride of 1 and padding of 2, the output size computes to $\dfrac{100+2*2-5}{1} + 1 = 100$. So, the output size is 100 x 100.

## Tensorflow

Tensorflow supports Keras, which has the built-in `Conv2D` layer class that allows for the 2D convolutional operation over a input plane

> **Notice the subtle difference of uppercase D in `Conv2D` in Tensorflow from lowercase d of `Conv2d` in PyTorch**

Lets try to do the same convolution operation as above with Tensorflow.

The **Conv2D** layer class takes in the following arguments:

{% highlight python %}
tf.keras.layers.Conv2D(filters = 3, kernel_size = 3, stride = 1,
                       padding = "valid")(input_tensor)     
                       #can create non-square kernels with kernel_size = (2,3)
{% endhighlight %}

Here, filters is the same as `out_channels` as PyTorch, and refers to the number of filters we are assigning to the layer. The rules for non-square kernel size and stride is the same as PyTorch, but for padding, one of `valid`:no padding or `same`:no loss in dimensions (case-insensitive) has to be added.

One key difference in the implementations is the **data_format**. Tensorflow uses `channels_last` as default, but PyTorch uses `channels_first` format. So, we need to be very careful about this. Lets understand this with some code:

{% highlight python %}
import tensorflow as tf

input_shape = (4, 28, 28, 3)  # N, H, W, C (channels last)

x = tf.random.normal(input_shape)
conv = tf.keras.layers.Conv2D(
    filters = 7, # Number of kernels

    kernel_size = 5, # Size of kernels, i.e of size 5x5

    strides=1,
    padding='same',   # To keep the input and output dimensions same

    input_shape= (28,28,3)  # Providing the keyword argument input_shape

    )                       # in data_format="channels_last" since first layer


print(conv(x).shape) # (4, 28, 28, 7)
{% endhighlight %}

When using this layer as the first layer in a model, provide the keyword argument input_shape (tuple of integers, does not include the sample axis), e.g. `input_shape=(28, 28, 3)` for 28x28 grayscale picture in data_format="channels_last". The good thing about Tensorflow is that we can also add the activation and bias as arguments to the same convolutional layer, but lets not overcomplicate things and let it be as it is.

## Conclusion
Convolution operation is a very common operator used frequently for handling image data, and recently are being used for time-spatial data too. Hence, it is very important to have a firm grasp over how the deep learning framework implement it to aid the development and maintenance of the models. Another very common but frequently misunderstood idea is how does the dimensions of the inputs, outputs and kernel/filters actually convolve and span out when used in operations like flatten etc. We shall discuss more about visual aspect of that in the next post.  
