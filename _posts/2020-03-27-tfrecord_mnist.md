---
title:  "Tensorflow tfrecord Example"
excerpt: "Tensorflow tfrecord에 대한 작은 예"
toc: true
toc_sticky: true
toc_label: "Table"
categories:
  - Development
tags:
  - Tensorflow 
comments : true
---

## 1. Intro

---
클라우드에서 서비스를 진행하기 위해서, 기존의 이미지들을 tfrecord로 바꾸어 전달 해야하는 일들이 생겼다.  
따라서, mnist dataset을 이용해서 예제를 만들어보았다.

&nbsp;

## **2. mnist_to_tfrecord.py**

---
* tensorflow에서 ndarray로 가져올 수 있는 mnist dataset을 tfrecord로 바꾸어 저장한다.  
    ```python
    """
    Reference : https://www.tensorflow.org/tutorials/load_data/tfrecord
    Author : jwlim
    """
    
    import os
    import numpy as np
    import tensorflow as tf
    
    
    def _bytes_feature(value):
      """Returns a bytes_list from a string / byte."""
      if isinstance(value, type(tf.constant(0))):
        value = value.numpy() # BytesList won't unpack a string from an EagerTensor.
      return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))
    
    
    def _float_feature(value):
      """Returns a float_list from a float / double."""
      return tf.train.Feature(float_list=tf.train.FloatList(value=[value]))
    
    
    def _int64_feature(value):
      """Returns an int64_list from a bool / enum / int / uint."""
      return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))
    
    
    def write_tfrecords(X, Y, output_path):
        def _np_to_tfrecord(x : np.ndarray, y : np.uint8) -> tf.train.Example:
            x_shape = np.shape(x)
            feature = {
                'height': _int64_feature(x_shape[0]),
                'width' : _int64_feature(x_shape[1]),
                'depth' : _int64_feature(x_shape[2]),
                'label' : _int64_feature(y),
                'image_raw' : _bytes_feature(x.tobytes())
            }
            return tf.train.Example(features=tf.train.Features(feature=feature))
    
        N = np.shape(X)[0]
    
        record_file = os.path.join(output_path, 'mymnist.tfrecords')
        with tf.io.TFRecordWriter(record_file) as writer:
            for i in range(N):
                features = _np_to_tfrecord(X[i], Y[i])
                writer.write(features.SerializeToString())
    
    
    def _preprocess_mnist(x : np.ndarray, y : np.uint8):
        x = np.expand_dims(x, axis=3)
        x = np.tile(x, (1, 1, 1, 1))
        return x, y
    
    
    def take_tfrecord_mnist(x, y, file_path_with_name):
        x, y = _preprocess_mnist(x, y)
        write_tfrecords(x, y, file_path_with_name)
    
    
    if __name__ == "__main__":
        mnist = tf.keras.datasets.mnist
        (x_train, y_train), _ = mnist.load_data()
    
        file_path_with_name = './mnist/data/'
        take_tfrecord_mnist(x_train, y_train, file_path_with_name)
    ```

&nbsp;
## **3. read_tfrecord_mnist.py**
---
* tfrecord로 저장된 mnist dataset을 다시 읽어오는 예제이다.  
    ```python
    """
    Reference : https://www.tensorflow.org/tutorials/load_data/tfrecord
    Author : jwlim
    """
    
    import tensorflow as tf
    import cv2
    
    
    # Create a dictionary describing the features.
    image_feature_description = {
        'height': tf.io.FixedLenFeature([], tf.int64),
        'width': tf.io.FixedLenFeature([], tf.int64),
        'depth': tf.io.FixedLenFeature([], tf.int64),
        'label': tf.io.FixedLenFeature([], tf.int64),
        'image_raw': tf.io.FixedLenFeature([], tf.string),
    }
    
    
    def _parse_image_function(example_proto):
      # Parse the input tf.Example proto using the dictionary above.
      return tf.io.parse_single_example(example_proto, image_feature_description)
    
    
    if __name__ == "__main__":
        raw_image_dataset = tf.data.TFRecordDataset('./mnist/data/mymnist.tfrecords')
        parsed_image_dataset = raw_image_dataset.map(_parse_image_function)
    
        for image_features  in parsed_image_dataset:
            print('height :', image_features['height'])
            print('width :', image_features['width'])
            print('depth :', image_features['depth'])
            image_raw = tf.io.decode_raw(image_features['image_raw'], tf.uint8)
            image_tf = tf.reshape(image_raw, (image_features['height'], image_features['width'], -1))
            image_np = image_tf.numpy()
    
            cv2.imshow('image', image_np)
            cv2.waitKey(0)
            cv2.destroyAllWindows()
            exit() # Just Example, no more loop
    ```
* 출력된 mnist 이미지  
![image](/assets/images/posts/2020-03-27-tfrecords_mnist/001.png) 