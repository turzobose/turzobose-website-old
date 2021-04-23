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

Keeping track of the dimensions is a very cumbersome process if not properly understood, specially while designing the deep neural network model architectures. The stress adds up swiftly when we have to shift between different machine learning frameworks, ideally PyTorch and Tensorflow. It is crazy how although both these frameworks are essentially doing the same thing, the syntax makes it increasingly difficult to allow for fluid transitions between them. Hence, this post is written to make the dimension tracking process simpler, for both PyTorch and Tensorflow implementations.

## PyTorch

PyTorch has the built-in **`Conv2d`** class that allows for the 2D convolutional operation over a input plane (e.g. think about an input image of shape `28 X 28 X 3 - height X width X channels`).

Lets say we want to convolve the image with a `kernel/filter of size 5` (e.g. 5 X 5) and want an output with 7 channels (think of the channels this way: RGB has 3 channels, so three 28 x 28 kernels stacked on top of each other)

The `Conv2d` class takes in the following arguments:

{% highlight python %}
nn.Conv2d(in_channels = 3, out_channels = 7,
          kernel_size = 5,
          stride = 1,
          padding = 0)    #can create non-square kernels with kernel_size=(2,3)
{% endhighlight %}

Here, the input channel comes from the input image, the output channel is defined by us, and the `stride = 1` and `padding = 0` is by default, but can be changed as per need. We can also define a non-square kernel of 2X3 by defining `kernel_size = (2,3)`.

You might be wondering what does the number of `out_channels` actually mean. It actually corresponds to the **number of kernels/channels/filters** we want the layer to have. Lets understand this with some code:

{% highlight python %}
import torch
input_batch = torch.rand(16, 3, 100, 100) # N = Batch Size, C = Input Channels(RGB), H = Height , W = Width

conv = torch.nn.Conv2d(in_channels=3, # RGB channels

                       out_channels=7, # Number of kernels/filters/channels this layer has

                       kernel_size=5, # Size of kernels (i. e. of size 5x5)

                       stride=1, padding=0)

print(conv.weight.size()) # 7 x 3 x 5 x 5 (7 kernels/channels of size 5x5 having depth of 3)

print(conv(input_batch).size()) # 16 x 7 x 100 x 100 => Batch Size = 16(from input image), Channels = 7, Height = 96, Width = 96
{% endhighlight %}

We understand that the output batch size of 16 comes from the input image batch size and the output no. of channels of 7 comes from the `out_channels` we defined. Now, let us calculate how the height and width dimensions are calculated. The formula for the dimensionality is:

<p align="center">$$ Dimensions = \lfloor\dfrac{N+2p-f}{s} + 1\rfloor $$</p>

Hence, with a stride of 1 and padding of 0, the output size computes to 96 x 96 as seen below:

<p align="center">$\dfrac{100+2*0-5}{1} + 1 = 96$</p>

If we want to keep the same height and width dimensions as the input image (same concept as `same padding` in Tensorflow discussed later), we need a padding of 2 as per the calculations below:

<p align="center">$\dfrac{100+2*2-5}{1} + 1 = 100$</p>

## Tensorflow

Tensorflow 2 and above supports Keras, which has the built-in `Conv2D` layer class that allows for the 2D convolutional operation over a input plane. *(Note: we will be discussing the convolution operation in Tensorflow 2 only, and not Tensorflow 1)*

> **Notice the subtle difference of uppercase D in `Conv2D` in Tensorflow from lowercase d of `Conv2d` in PyTorch**

Lets try to do the same convolution operation as we did with PyTorch with Tensorflow.

The **Conv2D** layer class takes in the following arguments:

{% highlight python %}
tf.keras.layers.Conv2D(filters = 7, kernel_size = 5, stride = 1,
                       padding = "valid")(input_tensor)     
                       #can create non-square kernels with kernel_size = (2,3)
{% endhighlight %}

Here, `filters` is the same as `out_channels` in PyTorch, and refers to the number of kernel or filters we are assigning to the layer. The rules for non-square kernel size and stride is the same as PyTorch, but for padding, one of `valid`:no padding or `same`:no loss in dimensions (case-insensitive) has to be added.

One key difference in the implementations is the **data_format**. Tensorflow uses `channels_last` as default, but PyTorch uses `channels_first` format. So, we need to be very careful about this. Lets understand this with some code:

{% highlight python %}
import tensorflow as tf

input_shape = (4, 28, 28, 3)  # N, H, W, C (channels last)

x = tf.random.normal(input_shape)
conv = tf.keras.layers.Conv2D(
    filters = 7, # Number of kernels the output layer has

    kernel_size = 5, # Size of kernels, i.e of size 5x5

    strides=1,
    padding='same',   # To keep the input and output dimensions same

    input_shape= (28,28,3)  # Providing the keyword argument input_shape

    )                       # in data_format="channels_last" since this layer 
    
                            # is the first layer


print(conv(x).shape) # (4, 28, 28, 7)
{% endhighlight %}

When using this layer as the first layer in a model, provide the keyword argument input_shape (tuple of integers, does not include the sample axis), e.g. `input_shape=(28, 28, 3)` for 28x28 RGB pictures in `data_format = "channels_last"`. The good thing about Tensorflow is that we can also add the activation and bias as arguments to the same convolutional layer, but lets not overcomplicate things and discuss it on a later post.

## Conclusion
Convolution operation is a very common operator used frequently for handling image data, and recently are being used in learning time-spatial data too. Hence, it is very important to have a firm grasp over how the deep learning frameworks implement it to aid the development and maintenance of the models. Another very common but frequently misunderstood idea is how does the dimensions of the inputs, outputs and kernel/filters actually convolve and span out when used in operations like flatten etc. We shall discuss more about visual aspect of that in the next post.  
