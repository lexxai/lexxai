---
layout: post
title: "Задача для зображень \"grey to rgb\" у моделі #keras"
date: 2024-03-07 15:43:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/03/grey-to-rgb-keras.html
---

Ось чому не можна використати
[FC](https://medium.com/@vaibhav1403/fully-connected-layer-f13275337c7c)
з активатором "[ReLU](https://uk.wikipedia.org/wiki/ReLU)" для  цієї задачі: 

```
layers.Dense(3, activation="relu", name="gray_rgb", input_shape=(32,32,1))
```

|  |
| --- |
|  |
| FC, relu |

Найкраще зробити підготовку dataset:

```
tx = np.repeat(x, 3, axis=-1)
```

або

```
tx = np.tile(x, (1, 1, 3))
```

Або шар
[Lambda](https://keras.io/api/layers/core_layers/lambda/)
(але я питання по збереження моделі до файлу):   

```
layers.Lambda(lambda x: tf.repeat(x, 3, axis=-1))
```

|  |
| --- |
|  |
| Grey to RGB |

```
rows = 4
plt.figure(figsize=(10,3*rows))
cols = rgb_images_train.shape[-1]
total = cols * rows
labels = ["R","G","B"]
for i in range(total):
    plt.subplot(rows,cols,i+1)
    plt.xticks([])
    plt.yticks([])
    plt.grid(False)
    id = i % cols
    rid = i // cols
    plt.imshow(rgb_images_train[0+rid,:,:,id], cmap=plt.cm.binary)
    plt.ylabel(f"Image {rid}, label: {np.argmax(y_train[rid])}")
    plt.xlabel(f"chanel {id} : '{labels[id]}'")
plt.show()
```

Або вже шар [Conv2D](https://keras.io/api/layers/convolution_layers/convolution2d/):

```
layers.Conv2D(3, (1, 1), use_bias=False, padding="same", kernel_initializer="ones", name="conv2d_108", input_shape=(32,32,1))
```

|  |
| --- |
|  |
| Conv2D 1х1 |

```
activation_model = Model(inputs=model.input, 
                         outputs=[layer.output for layer in model.layers])

activations = activation_model.predict(x_test[0].reshape(1, 32, 32, 1))

for layer_index, layer_activation in enumerate(activations):
    print(f"{layer_index=}, {layer_activation.shape=}")
    if len(layer_activation.shape) == 4:  
        num_features = layer_activation.shape[-1]
        size = layer_activation.shape[1]

        rows = num_features // 1  
        cols = layer_activation.shape[-1]

        plt.figure(figsize=(16, 12))
        for i in range(num_features):
            plt.subplot(rows, cols, i + 1)
            img = layer_activation[0, :, :, i]
            plt.imshow(img, cmap='viridis')
            plt.axis('off')
            print("min:", np.min(img), "max",np.max(img))
        plt.tight_layout()
        plt.subplots_adjust(top=0.94)
        plt.suptitle(f'Layer {activation_model.layers[layer_index+1].name} Feature Maps')
        plt.show()
```
