---
layout: post
title:  keras DataGenerator 를 이용해 db에 있는 데이터로 학습하는 방법
comments: true
---

keras 의 DataGenerator 는 학습데이터가 메모리보다 훨씬 많은경우에 효율적으로 학습을 시켜줄 수 있도록 합니다.

keras.utils.Sequence 를 상속받아

```python
    def __init__(self, *args)
    def on_epoch_end(self) # Method called at the end of every epoch.
    def __len__(self) # Number of batch in the Sequence.
    def __getitem__(self, index) #  Gets batch at position index.
```

4개의 함수를 적절히 구현하여 사용하면 됩니다.

검색을 해보면 대부분 이미지 파일을 읽어와서 처리하는 예제가 많아 데이터베이스에서 데이터를 가져와 학습시키는 예제를 작성해 보겠습니다.

- `__init__` : 생성자 입니다. 객체 생성시 호출됩니다.
- on_epoch_end : 한 epoch 가 수행 된 후 호출됩니다.
- `__len__` :  배치의 개수를 반환합니다. 예를 들어 전체데이터가 100개 이고 batch 사이즈가 20이라면 5를 반환합니다.
- `__getitem__` : 인자로 넘겨주는 index 값에 해당하는 배치의 데이터를 반환합니다.

유져당 4 x 4 행렬의 데이터를 오토인코더로 학습시킨다고 가정해 봅시다. 데이터베이스에는 user_id, x, y value 의 컬럼을 갖습니다.

%참고 : autoencoder 의 합성곱신경망에서는 홀수 크기로 데이터를 넣을 수 없습니다.

( [https://stats.stackexchange.com/questions/376464/convolutional-autoencoder-on-an-odd-size-image](https://stats.stackexchange.com/questions/376464/convolutional-autoencoder-on-an-odd-size-image) )

예를 들어 아래와 같습니다.

```
  
        [ 1  0  17  0 ]             [ 0 0 0 0 ]  
user1 = [ 0  3   0  0 ]     user2 = [ 0 5 0 0 ] 
        [ 0  0  -1  0 ]             [ 2 0 9 0 ] 
        [ 0  0   0  0 ]             [ 0 0 0 0 ]
```

이와 같은 행렬이라면 데이터베이스에는 아래와 같이 데이터가 있다고 가정합니다.

```
user1

| user_id | x | y | value |
|---------|---|---|-------|
| 1       | 0 | 0 | 1     |
| 1       | 2 | 0 | 17    |
| 1       | 1 | 1 | 3     |
| 1       | 2 | 2 | -1    |

user2

| user_id | x | y | value |
|---------|---|---|-------|
| 2       | 0 | 2 | 2     |
| 2       | 1 | 1 | 5     |
| 2       | 2 | 2 | 9     |

```

임의로 데이터를 만들어서 데이터베이스에 넣습니다.

```python
table_name = 'USER_MATRIX'

user_count = 100
size_x = 4
size_y = 4
for user_id in range(user_count):
    x = random.randrange(0, size_x)
    y = random.randrange(0, size_y)
    value = random.randrange(-7, 7)
    insert_query = '''insert into {} (user_id, x, y, value) VALUES({},{},{},{})'''.format(table_name,str(user_id), str(x), str(y), value)
    mysql_client.query(insert_query)
```

학습에 사용할 모델은 tensorflow 공식사이트의 오토인코더 예제 중 노이즈제거를 학습하는데 사용한 모델입니다.

([https://www.tensorflow.org/tutorials/generative/autoencoder](https://www.tensorflow.org/tutorials/generative/autoencoder))

```python
class Denoise(Model):
    def __init__(self, row_count, column_count):
        super(Denoise, self).__init__()
        self.encoder = tf.keras.Sequential([
            layers.Input(shape=(row_count, column_count, 1)),
            layers.Conv2D(16, (3, 3), activation='relu', padding='same', strides=2),
            layers.Conv2D(8, (3, 3), activation='relu', padding='same', strides=2)])

        self.decoder = tf.keras.Sequential([
            layers.Conv2DTranspose(8, kernel_size=3, strides=2, activation='relu', padding='same'),
            layers.Conv2DTranspose(16, kernel_size=3, strides=2, activation='relu', padding='same'),
            layers.Conv2D(1, kernel_size=(3, 3), activation='sigmoid', padding='same')])

    def call(self, x):
        encoded = self.encoder(x)
        decoded = self.decoder(encoded)
        return decoded
```

다음은 DataGenerator 입니다.  앞서 말했듯이 DataGenerator 는 keras.utils.Sequence 를 상속 받습니다.

```python
class SqlDataGenerator(keras.utils.Sequence):

    def __init__(self, ids, data_size, batch_size=10):
        self.user_ids = ids
        self.batch_size = batch_size
        self.indexes = np.arange(len(self.user_ids))
        self.data_size = data_size

    def __len__(self):
        return int(np.floor(len(self.user_ids) / self.batch_size))

    def __getitem__(self, index):
        indexes = self.indexes[index * self.batch_size:(index + 1) * self.batch_size]
        batch_user_ids = [self.user_ids[k] for k in indexes]

        in_query = ','.join(batch_user_ids)

        query = '''
        SELECT user_id, x, y, value FROM {} WHERE user_id IN ( {} )
        '''.format(table_name, in_query)

        user_matrix = {}
        mysql_client.query(query)
        res = mysql_client.store_result()
        for i in range(res.num_rows()):
            row = res.fetch_row()[0]

            user_id = str(row[0].decode("utf-8"))
            x = int(row[1].decode("utf-8"))
            y = int(row[2].decode("utf-8"))
            value = int(row[3].decode("utf-8"))

            if user_id not in user_matrix:
                user_matrix[user_id] = np.zeros((self.data_size[0], self.data_size[1]))
						
						user_matrix[user_id][x][y] = value

        data = list()
        for user_id, matrix in user_matrix.items():
            data.append(np.array(matrix)[..., tf.newaxis])

        data = np.array(data)
        return data, data

    def on_epoch_end(self):
        self.indexes = np.arange(len(self.user_ids))
```

참고로 `__getitem__` 함수의 index 값을 프린트 해보면 순차적으로 찍히지 않는데 keras 라이브러리가 데이터를 로드하는데 비동기로 처리를 하기 때문입니다.

(keras-team 깃헙이슈에 아래오 같은 댓글이 있습니다.)
```
And yes, keras uses multithreading and multiprocessing to load the data faster and it also confuses users because it's asynchronous but it's perfectly normal.
```

[https://github.com/keras-team/keras/issues/12082](https://github.com/keras-team/keras/issues/12082)

전체코드는 다음과 같습니다.

```python

from MySQLdb import _mysql
from tensorflow import keras
from tensorflow.keras import layers, losses
from tensorflow.keras.models import Model

import numpy as np
import tensorflow as tf
import random

mysql_host = 'host'
mysql_user = 'user'
mysql_pw = 'password'
mysql_db = 'db'

mysql_client = _mysql.connect(host=mysql_host, user=mysql_user, passwd=mysql_pw, db=mysql_db)

table_name = 'USER_MATRIX'

user_count = 100
size_x = 4
size_y = 4
for user_id in range(user_count):
    x = random.randrange(0, size_x)
    y = random.randrange(0, size_y)
    value = random.randrange(-7, 7)
    insert_query = '''insert into {} (user_id, x, y, value) VALUES({},{},{},{})'''.format(table_name,str(user_id), str(x), str(y), value)
    mysql_client.query(insert_query)

class Denoise(Model):
    def __init__(self, row_count, column_count):
        super(Denoise, self).__init__()
        self.encoder = tf.keras.Sequential([
            layers.Input(shape=(row_count, column_count, 1)),
            layers.Conv2D(16, (3, 3), activation='relu', padding='same', strides=2),
            layers.Conv2D(8, (3, 3), activation='relu', padding='same', strides=2)])

        self.decoder = tf.keras.Sequential([
            layers.Conv2DTranspose(8, kernel_size=3, strides=2, activation='relu', padding='same'),
            layers.Conv2DTranspose(16, kernel_size=3, strides=2, activation='relu', padding='same'),
            layers.Conv2D(1, kernel_size=(3, 3), activation='sigmoid', padding='same')])

    def call(self, x):
        encoded = self.encoder(x)
        decoded = self.decoder(encoded)
        return decoded

class SqlDataGenerator(keras.utils.Sequence):

    def __init__(self, ids, data_size, batch_size=10):
        self.user_ids = ids
        self.batch_size = batch_size
        self.indexes = np.arange(len(self.user_ids))
        self.data_size = data_size

    def __len__(self):
        return int(np.floor(len(self.user_ids) / self.batch_size))

    def __getitem__(self, index):
        indexes = self.indexes[index * self.batch_size:(index + 1) * self.batch_size]
        batch_user_ids = [self.user_ids[k] for k in indexes]

        in_query = ','.join(batch_user_ids)

        query = '''
        SELECT user_id, x, y, value FROM {} WHERE user_id IN ( {} )
        '''.format(table_name, in_query)

        user_matrix = {}
        mysql_client.query(query)
        res = mysql_client.store_result()
        for i in range(res.num_rows()):
            row = res.fetch_row()[0]

            user_id = str(row[0].decode("utf-8"))
            x = int(row[1].decode("utf-8"))
            y = int(row[2].decode("utf-8"))
            value = int(row[3].decode("utf-8"))

            if user_id not in user_matrix:
                user_matrix[user_id] = np.zeros((self.data_size[0], self.data_size[1]))

            user_matrix[user_id][x][y] = value

        data = list()
        for user_id, matrix in user_matrix.items():
            data.append(np.array(matrix)[..., tf.newaxis])

        data = np.array(data)
        return data, data

    def on_epoch_end(self):
        self.indexes = np.arange(len(self.user_ids))

game_ids = [str(i) for i in range(0, user_count)]

autoencoder = Denoise(size_x, size_y)
autoencoder.compile(optimizer='adam', loss=losses.MeanSquaredError())

batch_size = 20
train_generator = SqlDataGenerator(game_ids, (size_x, size_y), batch_size)
autoencoder.fit(
    x=train_generator,
    validation_data=train_generator,
    batch_size=batch_size,
    epochs=1,
)

autoencoder.encoder.summary()
autoencoder.decoder.summary()
```

참고 사이트

[https://www.tensorflow.org/api_docs/python/tf/keras/utils/Sequence](https://www.tensorflow.org/api_docs/python/tf/keras/utils/Sequence)

[https://sunshower76.github.io/frameworks/2020/02/09/Keras-Batch생성하기1-(Seuquence&fit_generator)/](https://sunshower76.github.io/frameworks/2020/02/09/Keras-Batch%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B01-(Seuquence&fit_generator)/)