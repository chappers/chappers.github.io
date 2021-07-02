---
layout: post
category : web micro log
tags :
---


Keras is an interesting framework which allows one to easily define
and train neural networks. After a long time "avoiding" deep learning
libraries, I have finally taken a dive using Keras. Here are some notes and
examples for getting started!

Multinomial Regression
-------------------

The corner stone of any neural network is the last layer plus activation function.
If we use the activation function being the softmax function, we will essentially have
a softmax regression problem (multinomial regression).

To demonstrate how this works in Keras, consider the iris dataset.

In this dataset, there are 4 columns, and 3 target classes. Then to specify
what the neural network should look like, we simply need the `input_shape` to
be 4 and the output to be of size 3.

There are multiple ways of specifying this.

```py
from sklearn.datasets import load_iris
import numpy as np

from keras.models import Sequential
from keras.layers import Dense, Activation

iris = load_iris()

model = Sequential()
model.add(Dense(3, input_shape=(4,))) # input is 4 columns, output is 3 classes
model.add(Activation('softmax'))
model.compile(loss='sparse_categorical_crossentropy', optimizer='sgd', metrics=['accuracy'])

model.fit(iris.data, iris.target, verbose=1, batch_size=5, nb_epoch=15)

loss, accuracy = model.evaluate(iris.data, iris.target, verbose=0)
print("Accuracy is {:.2f}".format(accuracy))
"""
>>> model.fit(iris.data, iris.target, verbose=1, batch_size=5, nb_epoch=15)
Epoch 1/15
150/150 [==============================] - 0s - loss: 1.8285 - acc: 0.2867
Epoch 2/15
150/150 [==============================] - 0s - loss: 1.1306 - acc: 0.3467
Epoch 3/15
150/150 [==============================] - 0s - loss: 0.9325 - acc: 0.4333
Epoch 4/15
150/150 [==============================] - 0s - loss: 0.8028 - acc: 0.6533
Epoch 5/15
150/150 [==============================] - 0s - loss: 0.7247 - acc: 0.8000
Epoch 6/15
150/150 [==============================] - 0s - loss: 0.6637 - acc: 0.7867
Epoch 7/15
150/150 [==============================] - 0s - loss: 0.6029 - acc: 0.8400
Epoch 8/15
150/150 [==============================] - 0s - loss: 0.5830 - acc: 0.8000
Epoch 9/15
150/150 [==============================] - 0s - loss: 0.5570 - acc: 0.8200
Epoch 10/15
150/150 [==============================] - 0s - loss: 0.5279 - acc: 0.8333
Epoch 11/15
150/150 [==============================] - 0s - loss: 0.5297 - acc: 0.8267
Epoch 12/15
150/150 [==============================] - 0s - loss: 0.5055 - acc: 0.8200
Epoch 13/15
150/150 [==============================] - 0s - loss: 0.4978 - acc: 0.8400
Epoch 14/15
150/150 [==============================] - 0s - loss: 0.4910 - acc: 0.8333
Epoch 15/15
150/150 [==============================] - 0s - loss: 0.4755 - acc: 0.8333
<keras.callbacks.History object at 0x0000026119852D30>
>>>
>>> loss, accuracy = model.evaluate(iris.data, iris.target, verbose=0)
>>> print("Accuracy is {:.2f}".format(accuracy))
Accuracy is 0.89
"""
```

The second way we can have it like a "pipeline" kind of notion, We can also use a
different loss function. If we do we may need to change our encoding for our response to a
one hot encoder. Fortunately keras offers utility functions to do this.

```py
from keras.utils import np_utils

model = Sequential([
    Dense(3, input_shape=(4,)),
    Activation('softmax'),
])

model.compile(loss='categorical_crossentropy', optimizer='sgd', metrics=['accuracy'])
model.fit(iris.data, np_utils.to_categorical(iris.target, 3), verbose=1, batch_size=5, nb_epoch=15)
loss, accuracy = model.evaluate(iris.data, np_utils.to_categorical(iris.target, 3), verbose=0)
print("Accuracy is {:.2f}".format(accuracy))
"""
>>> model.fit(iris.data, np_utils.to_categorical(iris.target, 3), verbose=1, batch_size=5, nb_epoch=15)
Epoch 1/15
150/150 [==============================] - 0s - loss: 2.5564 - acc: 0.3933
Epoch 2/15
150/150 [==============================] - 0s - loss: 0.9428 - acc: 0.5067
Epoch 3/15
150/150 [==============================] - 0s - loss: 0.7065 - acc: 0.7333
Epoch 4/15
150/150 [==============================] - 0s - loss: 0.6590 - acc: 0.7333
Epoch 5/15
150/150 [==============================] - 0s - loss: 0.6268 - acc: 0.7533
Epoch 6/15
150/150 [==============================] - 0s - loss: 0.6056 - acc: 0.7333
Epoch 7/15
150/150 [==============================] - 0s - loss: 0.5737 - acc: 0.7733
Epoch 8/15
150/150 [==============================] - 0s - loss: 0.5617 - acc: 0.7400
Epoch 9/15
150/150 [==============================] - 0s - loss: 0.5481 - acc: 0.7533
Epoch 10/15
150/150 [==============================] - 0s - loss: 0.5278 - acc: 0.7800
Epoch 11/15
150/150 [==============================] - 0s - loss: 0.5156 - acc: 0.7867
Epoch 12/15
150/150 [==============================] - 0s - loss: 0.5099 - acc: 0.8000
Epoch 13/15
150/150 [==============================] - 0s - loss: 0.4819 - acc: 0.8200
Epoch 14/15
150/150 [==============================] - 0s - loss: 0.4982 - acc: 0.8000
Epoch 15/15
150/150 [==============================] - 0s - loss: 0.4852 - acc: 0.7867
<keras.callbacks.History object at 0x00000261199B3B00>
>>> loss, accuracy = model.evaluate(iris.data, np_utils.to_categorical(iris.target, 3), verbose=0)
>>> print("Accuracy is {:.2f}".format(accuracy))
Accuracy is 0.87
"""
```

One can arbitarily extend this by adding more dense layers, but that isn't very interesting...

Image examples
--------------

Keras does provide nice examples for image modelling. If you look at the provided
code. However it need not be as difficult as their provided example.

Recall that if we want to build a CovNet for image classification, we need [3 components](http://cs231n.github.io/convolutional-networks/):

1.  Convolution Layer - this performs the convolution filters  
2.  Pooling - this performs the downsampling operation  
3.  Fully connected layer - this is the normal dense neural network layer  

Amazingly the model seems to do pretty well without pooling or even a fully connected layer (see example below)!

Using the MNIST example:

```py
model = Sequential()
model.add(Convolution2D(nb_filters, kernel_size[0], kernel_size[1],
                       border_mode='valid',
                       input_shape=input_shape))
model.add(Activation('relu'))
model.add(Flatten()) # this is so that the last layer will be able to perform softmax
model.add(Dense(10))
model.add(Activation('softmax'))

model.compile(loss='categorical_crossentropy',
              optimizer='adadelta',
              metrics=['accuracy'])


model.fit(X_train, Y_train, batch_size=128, nb_epoch=2,
          verbose=1, validation_data=(X_test, Y_test))
score = model.evaluate(X_test, Y_test, verbose=0)
print('Test score:', score[0])
print('Test accuracy:', score[1])

"""
>>> model.fit(X_train, Y_train, batch_size=128, nb_epoch=2,
...           verbose=1, validation_data=(X_test, Y_test))
Train on 60000 samples, validate on 10000 samples
Epoch 1/2
60000/60000 [==============================] - 37s - loss: 0.3678 - acc: 0.9004 - val_loss: 0.2032 - val_acc: 0.9432
Epoch 2/2
60000/60000 [==============================] - 36s - loss: 0.1791 - acc: 0.9506 - val_loss: 0.1539 - val_acc: 0.9555
<keras.callbacks.History object at 0x0000026101669DA0>
>>> score = model.evaluate(X_test, Y_test, verbose=0)
>>> print('Test score:', score[0])
Test score: 0.153921858574
>>> print('Test accuracy:', score[1])
Test accuracy: 0.9555
"""
```
And using the notMNIST code:

```py
model = Sequential()
model.add(Convolution2D(nb_filters, kernel_size[0], kernel_size[1],
                       border_mode='valid',
                       input_shape=input_shape))

model.add(Activation('relu'))
model.add(Flatten()) # this is so that the last layer will be able to perform softmax
model.add(Dense(10))
model.add(Activation('softmax'))

model.compile(loss='categorical_crossentropy',
              optimizer='adadelta',
              metrics=['accuracy'])

model.fit(train_dataset, train_labels, batch_size=128, nb_epoch=2,
          verbose=1, validation_data=(valid_dataset, valid_labels))
score = model.evaluate(test_dataset, test_labels, verbose=0)

print('Test score:', score[0])
print('Test accuracy:', score[1])

"""
Train on 200000 samples, validate on 10000 samples
Epoch 1/2
200000/200000 [==============================] - 157s - loss: 0.5757 - acc: 0.8431 - val_loss: 0.4920 - val_acc: 0.8682
Epoch 2/2
200000/200000 [==============================] - 167s - loss: 0.4504 - acc: 0.8774 - val_loss: 0.4467 - val_acc: 0.8782
<keras.callbacks.History object at 0x0000026101398F98>
>>> score = model.evaluate(test_dataset, test_labels, verbose=0)
>>>
>>> print('Test score:', score[0])
Test score: 0.238625081217
>>> print('Test accuracy:', score[1])
Test accuracy: 0.941
>>>
"""
```

These results fully demonstrate how (relatively) easy it is to define and create our own neural networks in Keras.



Full code
---------

Logistic regression:

```py
from sklearn.datasets import load_iris
import numpy as np

from keras.models import Sequential
from keras.layers import Dense, Activation

iris = load_iris()

model = Sequential()
model.add(Dense(3, input_shape=(4,))) # input is 4 columns, output is 3 classes
model.add(Activation('softmax'))
model.compile(loss='sparse_categorical_crossentropy', optimizer='sgd', metrics=['accuracy'])

model.fit(iris.data, iris.target, verbose=1, batch_size=5, nb_epoch=15)

loss, accuracy = model.evaluate(iris.data, iris.target, verbose=0)
print("Accuracy is {:.2f}".format(accuracy))

# or perhaps written in another way:
from keras.utils import np_utils

model = Sequential([
    Dense(3, input_shape=(4,)),
    Activation('softmax'),
])

model.compile(loss='categorical_crossentropy', optimizer='sgd', metrics=['accuracy'])
model.fit(iris.data, np_utils.to_categorical(iris.target, 3), verbose=1, batch_size=5, nb_epoch=15)
loss, accuracy = model.evaluate(iris.data, np_utils.to_categorical(iris.target, 3), verbose=0)
print("Accuracy is {:.2f}".format(accuracy))
```

MNIST

```py
from keras.datasets import mnist
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation, Flatten
from keras.layers import Convolution2D, MaxPooling2D
from keras.utils import np_utils
from keras import backend as K
import numpy as np

# the data, shuffled and split between train and test sets
(X_train, y_train), (X_test, y_test) = mnist.load_data()

X_train = X_train[:, :, :, np.newaxis].astype('float32')
X_test  = X_test[:, :, :, np.newaxis].astype('float32')
input_shape = X_train.shape[1:]

# normalising
X_train /= 255
X_test /= 255
print('X_train shape:', X_train.shape)
print(X_train.shape[0], 'train samples')
print(X_test.shape[0], 'test samples')

# convert class vectors to binary class matrices
Y_train = np_utils.to_categorical(y_train, 10)
Y_test = np_utils.to_categorical(y_test, 10)

# number of convolutional filters to use
nb_filters = 32
# convolution kernel size
kernel_size = (3, 3)

model = Sequential()
model.add(Convolution2D(nb_filters, kernel_size[0], kernel_size[1],
                       border_mode='valid',
                       input_shape=input_shape))
model.add(Activation('relu'))
model.add(Flatten()) # this is so that the last layer will be able to perform softmax
model.add(Dense(10))
model.add(Activation('softmax'))

model.compile(loss='categorical_crossentropy',
              optimizer='adadelta',
              metrics=['accuracy'])


model.fit(X_train, Y_train, batch_size=128, nb_epoch=2,
          verbose=1, validation_data=(X_test, Y_test))
score = model.evaluate(X_test, Y_test, verbose=0)
print('Test score:', score[0])
print('Test accuracy:', score[1])

```

notMNIST (code borrowed from the [udacity example](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/udacity) within the tensorflow documentation)


```py
from six.moves.urllib.request import urlretrieve
from six.moves import cPickle as pickle
import os
import tarfile

from scipy import ndimage

from keras.datasets import mnist
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation, Flatten
from keras.layers import Convolution2D, MaxPooling2D
from keras.utils import np_utils
from keras import backend as K
import numpy as np

# stuff

image_size = 28  # Pixel width and height.
pixel_depth = 255.0  # Number of levels per pixel.

url = 'http://commondatastorage.googleapis.com/books1000/'
last_percent_reported = None

def download_progress_hook(count, blockSize, totalSize):
    """A hook to report the progress of a download. This is mostly intended for users with
    slow internet connections. Reports every 5% change in download progress.
    """
    global last_percent_reported
    percent = int(count * blockSize * 100 / totalSize)

    if last_percent_reported != percent:
        if percent % 5 == 0:
            sys.stdout.write("%s%%" % percent)
            sys.stdout.flush()
        else:
            sys.stdout.write(".")
            sys.stdout.flush()

        last_percent_reported = percent

def maybe_download(filename, expected_bytes, force=False):
    """Download a file if not present, and make sure it's the right size."""
    if force or not os.path.exists(filename):
        print('Attempting to download:', filename)
        filename, _ = urlretrieve(url + filename, filename, reporthook=download_progress_hook)
        print('\nDownload Complete!')
    statinfo = os.stat(filename)
    if statinfo.st_size == expected_bytes:
        print('Found and verified', filename)
    else:
        raise Exception(
            'Failed to verify ' + filename + '. Can you get to it with a browser?')
    return filename

train_filename = maybe_download('notMNIST_large.tar.gz', 247336696)
test_filename = maybe_download('notMNIST_small.tar.gz', 8458043)

num_classes = 10
np.random.seed(133)

def maybe_extract(filename, force=False):
    root = os.path.splitext(os.path.splitext(filename)[0])[0]    # remove .tar.gz
    if os.path.isdir(root) and not force:
        # You may override by setting force=True.
        print('%s already present - Skipping extraction of %s.' % (root, filename))
    else:
        print('Extracting data for %s. This may take a while. Please wait.' % root)
        tar = tarfile.open(filename)
        sys.stdout.flush()
        tar.extractall()
        tar.close()
    data_folders = [
        os.path.join(root, d) for d in sorted(os.listdir(root))
        if os.path.isdir(os.path.join(root, d))]
    if len(data_folders) != num_classes:
        raise Exception(
            'Expected %d folders, one per class. Found %d instead.' % (
                num_classes, len(data_folders)))
    print(data_folders)
    return data_folders

train_folders = maybe_extract(train_filename)
test_folders = maybe_extract(test_filename)


def load_letter(folder, min_num_images):
    """Load the data for a single letter label."""
    image_files = os.listdir(folder)
    dataset = np.ndarray(shape=(len(image_files), image_size, image_size),
                                                 dtype=np.float32)
    print(folder)
    num_images = 0
    for image in image_files:
        image_file = os.path.join(folder, image)
        try:
            image_data = (ndimage.imread(image_file).astype(float) -
                          pixel_depth / 2) / pixel_depth
            if image_data.shape != (image_size, image_size):
                raise Exception('Unexpected image shape: %s' % str(image_data.shape))
            dataset[num_images, :, :] = image_data
            num_images = num_images + 1
        except IOError as e:
            print('Could not read:', image_file, ':', e, '- it\'s ok, skipping.')

    dataset = dataset[0:num_images, :, :]
    if num_images < min_num_images:
        raise Exception('Many fewer images than expected: %d < %d' %
                                        (num_images, min_num_images))

    print('Full dataset tensor:', dataset.shape)
    print('Mean:', np.mean(dataset))
    print('Standard deviation:', np.std(dataset))
    return dataset

def maybe_pickle(data_folders, min_num_images_per_class, force=False):
    dataset_names = []
    for folder in data_folders:
        set_filename = folder + '.pickle'
        dataset_names.append(set_filename)
        if os.path.exists(set_filename) and not force:
            # You may override by setting force=True.
            print('%s already present - Skipping pickling.' % set_filename)
        else:
            print('Pickling %s.' % set_filename)
            dataset = load_letter(folder, min_num_images_per_class)
            try:
                with open(set_filename, 'wb') as f:
                    pickle.dump(dataset, f, pickle.HIGHEST_PROTOCOL)
            except Exception as e:
                print('Unable to save data to', set_filename, ':', e)
    return dataset_names

train_datasets = maybe_pickle(train_folders, 45000)
test_datasets = maybe_pickle(test_folders, 1800)

def make_arrays(nb_rows, img_size):
    if nb_rows:
        dataset = np.ndarray((nb_rows, img_size, img_size), dtype=np.float32)
        labels = np.ndarray(nb_rows, dtype=np.int32)
    else:
        dataset, labels = None, None
    return dataset, labels

def merge_datasets(pickle_files, train_size, valid_size=0):
    num_classes = len(pickle_files)
    valid_dataset, valid_labels = make_arrays(valid_size, image_size)
    train_dataset, train_labels = make_arrays(train_size, image_size)
    vsize_per_class = valid_size // num_classes
    tsize_per_class = train_size // num_classes

    start_v, start_t = 0, 0
    end_v, end_t = vsize_per_class, tsize_per_class
    end_l = vsize_per_class+tsize_per_class
    for label, pickle_file in enumerate(pickle_files):             
        try:
            with open(pickle_file, 'rb') as f:
                letter_set = pickle.load(f)
                # let's shuffle the letters to have random validation and training set
                np.random.shuffle(letter_set)
                if valid_dataset is not None:
                    valid_letter = letter_set[:vsize_per_class, :, :]
                    valid_dataset[start_v:end_v, :, :] = valid_letter
                    valid_labels[start_v:end_v] = label
                    start_v += vsize_per_class
                    end_v += vsize_per_class

                train_letter = letter_set[vsize_per_class:end_l, :, :]
                train_dataset[start_t:end_t, :, :] = train_letter
                train_labels[start_t:end_t] = label
                start_t += tsize_per_class
                end_t += tsize_per_class
        except Exception as e:
            print('Unable to process data from', pickle_file, ':', e)
            raise

    return valid_dataset, valid_labels, train_dataset, train_labels

train_size = 200000
valid_size = 10000
test_size = 10000

valid_dataset, valid_labels, train_dataset, train_labels = merge_datasets(
    train_datasets, train_size, valid_size)
_, _, test_dataset, test_labels = merge_datasets(test_datasets, test_size)

print('Training:', train_dataset.shape, train_labels.shape)
print('Validation:', valid_dataset.shape, valid_labels.shape)
print('Testing:', test_dataset.shape, test_labels.shape)

def randomize(dataset, labels):
    permutation = np.random.permutation(labels.shape[0])
    shuffled_dataset = dataset[permutation,:,:]
    shuffled_labels = labels[permutation]
    return shuffled_dataset, shuffled_labels


train_dataset, train_labels = randomize(train_dataset, train_labels)
test_dataset, test_labels = randomize(test_dataset, test_labels)
valid_dataset, valid_labels = randomize(valid_dataset, valid_labels)

### begin keras training
train_dataset = train_dataset[:, :, :, np.newaxis].astype('float')
test_dataset = test_dataset[:, :, :, np.newaxis].astype('float')
valid_dataset = valid_dataset[:, :, :, np.newaxis].astype('float')

train_labels = np_utils.to_categorical(train_labels, 10)
test_labels = np_utils.to_categorical(test_labels, 10)
valid_labels = np_utils.to_categorical(valid_labels, 10)

input_shape = train_dataset.shape[1:]
kernel_size = (3, 3)
nb_filters = 32

# build model

model = Sequential()
model.add(Convolution2D(nb_filters, kernel_size[0], kernel_size[1],
                       border_mode='valid',
                       input_shape=input_shape))

model.add(Activation('relu'))
model.add(Flatten()) # this is so that the last layer will be able to perform softmax
model.add(Dense(10))
model.add(Activation('softmax'))

model.compile(loss='categorical_crossentropy',
              optimizer='adadelta',
              metrics=['accuracy'])

model.fit(train_dataset, train_labels, batch_size=128, nb_epoch=2,
          verbose=1, validation_data=(valid_dataset, valid_labels))
score = model.evaluate(test_dataset, test_labels, verbose=0)

print('Test score:', score[0])
print('Test accuracy:', score[1])
```
