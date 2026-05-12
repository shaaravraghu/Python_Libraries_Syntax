# TensorFlow Production Technical Reference

This exhaustive reference document serves as a complete guide to TensorFlow, specifically focusing on TensorFlow 2.x and modern best practices for production deployments.

## Contents
- [1. Introduction & History](#1-introduction--history)
- [2. Installation & Setup](#2-installation--setup)
- [3. Core Tensor Concepts](#3-core-tensor-concepts)
- [4. Keras and Model Building](#4-keras-and-model-building)
- [5. Data Pipelines (`tf.data`)](#5-data-pipelines-tfdata)
- [6. Training Loops](#6-training-loops)
- [7. Evaluation and Metrics](#7-evaluation-and-metrics)
- [8. Saving and Loading Models](#8-saving-and-loading-models)
- [9. TensorFlow in Production](#9-tensorflow-in-production)
- [10. TensorFlow Lite](#10-tensorflow-lite)
- [11. TensorFlow Serving](#11-tensorflow-serving)
- [12. TensorBoard](#12-tensorboard)
- [13. Distributed Training](#13-distributed-training)
- [14. TensorFlow Probability](#14-tensorflow-probability)
- [15. Advanced Tensor Operations](#15-advanced-tensor-operations)
- [16. Performance Optimization](#16-performance-optimization)
- [17. Debugging and Profiling](#17-debugging-and-profiling)
- [18. Common Pitfalls](#18-common-pitfalls)
- [19. Advanced Topics](#19-advanced-topics)

---

## 1. INTRODUCTION & HISTORY

### What is TensorFlow
TensorFlow is an end-to-end open-source platform for machine learning developed by researchers and engineers working on the Google Brain team. It has a comprehensive, flexible ecosystem of tools, libraries, and community resources that lets researchers push the state-of-the-art in ML and developers easily build and deploy ML-powered applications.

### Why TensorFlow (Scalability, Production Readiness, Ecosystem)
- **Scalability:** TensorFlow seamlessly transitions from running on a single CPU to thousands of GPUs or TPUs.
- **Production Readiness:** Tools like TensorFlow Serving, TensorFlow Lite, and TensorFlow.js make it trivial to deploy models to servers, edge devices, and browsers.
- **Ecosystem:** TensorFlow Extended (TFX) provides an end-to-end platform for deploying production ML pipelines.

### TensorFlow vs PyTorch Comparison

| Feature | TensorFlow | PyTorch |
| :--- | :--- | :--- |
| **Execution** | Eager by default (with seamless graph compilation via `@tf.function`) | Eager by default (with TorchScript/`torch.compile` for graphs) |
| **High-level API** | Built-in Keras (standardized, highly productive) | PyTorch Lightning, FastAI (third-party) |
| **Deployment** | Unmatched (TF Serving, TFLite, TF.js, TFX) | Strong, but historically required ONNX or TorchServe |
| **Community** | Massive industry adoption, heavy Google backing | Dominant in academia and research |
| **Learning Curve**| Slightly steeper due to vast API surface | More pythonic and intuitive |

### Evolution from TF1.x to TF2.x
TF1.x relied on a declarative programming model: you build an abstract "Graph" of operations, then execute it within a `tf.Session()`. This was notoriously difficult to debug.
TF2.x introduced **Eager Execution** by default, meaning operations are evaluated immediately as they are called from Python, exactly like PyTorch or NumPy. Graphs are now generated seamlessly using the `@tf.function` decorator.

### High-level vs Low-level APIs
- **Keras (High-level):** `tf.keras` is the recommended high-level API for most tasks. It abstracts away complexities.
- **Core TF (Low-level):** Provides granular control over tensors, gradients, and custom mathematical operations (`tf.math`, `tf.linalg`).

> 💡 **Pro Tips:** Always start with `tf.keras`. Only drop down to Core TF or custom training loops when the Keras abstractions restrict your desired architecture or workflow.
> ⚠️ **Common Pitfalls:** Mixing TF1 paradigms (like `tf.compat.v1.Session`) in a TF2 codebase. Avoid `tf.compat.v1` entirely in new projects.

---

## 2. INSTALLATION & SETUP

### pip install tensorflow
The recommended way to install TensorFlow is via pip.

```bash
# Install the latest stable CPU and GPU version for Linux/Windows
pip install tensorflow

# For Mac users (M1/M2/M3 Apple Silicon)
pip install tensorflow-macos
pip install tensorflow-metal
```

### GPU vs CPU setup & CUDA Basics
For Windows/Linux, the standard `tensorflow` package automatically includes GPU support. However, you must install the correct NVIDIA drivers. Modern TensorFlow (2.12+) bundles its own CUDA and cuDNN libraries via pip wheels, drastically simplifying setup!

### Checking Installation
Always verify your setup before starting a project.

```python
# Import the TensorFlow library using the standard alias
import tensorflow as tf

# Print the current TensorFlow version
print(f"TensorFlow Version: {tf.__version__}")

# Check if TensorFlow is built with CUDA (GPU support)
print(f"Built with CUDA: {tf.test.is_built_with_cuda()}")

# List all physical GPU devices available to TensorFlow
gpus = tf.config.list_physical_devices('GPU')
if gpus:
    # If GPUs are found, print their details
    print(f"Found {len(gpus)} GPU(s): {gpus}")
    # Enable memory growth to prevent TF from hogging all GPU VRAM at initialization
    for gpu in gpus:
        tf.config.experimental.set_memory_growth(gpu, True)
else:
    # If no GPUs are found, notify the user
    print("No GPUs found. Running on CPU.")
```

> 💡 **Pro Tips:** Always enable GPU memory growth (`set_memory_growth(gpu, True)`) so TensorFlow only allocates VRAM as needed, allowing other processes to share the GPU.
> ⚠️ **Common Pitfalls:** Forgetting to check if the GPU is recognized. If TF runs on CPU silently, your training will be painfully slow.

---

## 3. TENSORS & OPERATIONS

### What are tensors
Tensors are multi-dimensional arrays with a uniform type (called a `dtype`). They are the fundamental data structure in TensorFlow, akin to `numpy.ndarray`, but with the crucial ability to reside in accelerator memory (GPU/TPU) and automatically track operations for gradient computation.

### Creating Tensors & Shapes/dtypes

```python
import tensorflow as tf

# Create a scalar tensor (0-D tensor) - Rank 0
scalar = tf.constant(42)
# Create a vector tensor (1-D tensor) - Rank 1
vector = tf.constant([1.0, 2.0, 3.0], dtype=tf.float32)
# Create a matrix tensor (2-D tensor) - Rank 2
matrix = tf.constant([[1, 2], [3, 4]])

# Creating tensors with specific initializers
# Tensor of shape (3, 3) filled with zeros
zeros = tf.zeros((3, 3))
# Tensor of shape (2, 2) filled with ones
ones = tf.ones((2, 2))
# Tensor of shape (2, 2) drawn from standard normal distribution
random_tensor = tf.random.normal((2, 2), mean=0.0, stddev=1.0)

# Inspecting tensor properties
print("Shape:", matrix.shape)     # Outputs: (2, 2)
print("Data type:", matrix.dtype) # Outputs: <dtype: 'int32'>
print("Rank:", tf.rank(matrix))   # Outputs: 2
```

### Basic Operations & Broadcasting

```python
# Element-wise addition
a = tf.constant([1, 2, 3])
b = tf.constant([4, 5, 6])
c = tf.add(a, b) # or simply a + b

# Matrix multiplication (dot product)
mat1 = tf.constant([[1, 2], [3, 4]])
mat2 = tf.constant([[5, 6], [7, 8]])
dot_product = tf.matmul(mat1, mat2) # or mat1 @ mat2

# Broadcasting: smaller tensor is "stretched" to fit the larger one
vector = tf.constant([10, 20])
matrix = tf.constant([[1, 2], [3, 4]])
# The vector [10, 20] is added to both rows of the matrix
broadcasted_sum = vector + matrix
```

> 💡 **Pro Tips:** Most `numpy` operations have a direct `tf` equivalent (e.g., `np.sum` -> `tf.reduce_sum`). You can convert a tensor to a numpy array via `tensor.numpy()`.
> ⚠️ **Common Pitfalls:** Type mismatches. TensorFlow is strict about `dtype`. You cannot add a `tf.float32` tensor to a `tf.int32` tensor without explicitly casting via `tf.cast(tensor, tf.float32)`.

---

## 4. VARIABLES & CONSTANTS

### tf.constant vs tf.Variable
- `tf.constant`: Immutable. Once created, its values cannot change. Used for static data.
- `tf.Variable`: Mutable. Represents shared, persistent state manipulated by your program. Weights and biases in neural networks are variables.

### Assignments and Updates

```python
import tensorflow as tf

# Create a variable tensor with an initial value
my_var = tf.Variable([1.0, 2.0, 3.0])

# Variables cannot be updated with standard assignment (=)
# my_var[0] = 5.0  # THIS WILL RAISE AN ERROR

# Instead, use the .assign() method to modify values in-place
my_var[0].assign(5.0)

# Use .assign_add() for in-place addition (useful for counters or optimizers)
my_var.assign_add([1.0, 1.0, 1.0])

# Variables have a name attribute, useful for debugging and checkpointing
named_var = tf.Variable(3.14, name="pi_approximation")
```

> 💡 **Pro Tips:** Use `tf.Variable` sparingly and only for state that must change during execution (like model weights). Constants allow the XLA compiler to optimize the graph more effectively.
> ⚠️ **Common Pitfalls:** Creating `tf.Variable` instances inside a `tf.function` without explicitly managing their state. Variables should typically be created exactly once and reused.

---

## 5. AUTO DIFFERENTIATION

### GradientTape
TensorFlow provides the `tf.GradientTape` API for automatic differentiation (computing the gradient of a computation with respect to its input variables).

### Computing Gradients

```python
import tensorflow as tf

# Define a variable that we want to compute the gradient for
x = tf.Variable(3.0)

# Open a GradientTape context to record operations
with tf.GradientTape() as tape:
    # Perform some mathematical operations: y = x^2
    y = x ** 2
    
# Outside the context, ask the tape to compute the gradient of y wrt x
# dy/dx = 2*x. Since x = 3.0, dy/dx should be 6.0
dy_dx = tape.gradient(y, x)

# Print the computed gradient
print("Gradient:", dy_dx.numpy()) # Outputs: 6.0
```

### Backpropagation Basics & Higher-Order Gradients
By default, `GradientTape` only tracks `tf.Variable`. To track a `tf.constant`, you must explicitly tell it to via `tape.watch()`.
Tapes are also single-use by default. To compute multiple gradients, use `persistent=True`.

```python
x = tf.constant(3.0)

# Create a persistent tape to compute higher-order gradients
with tf.GradientTape(persistent=True) as tape1:
    tape1.watch(x) # Explicitly watch a constant
    with tf.GradientTape() as tape2:
        tape2.watch(x)
        y = x ** 3
    # Compute first derivative dy/dx = 3x^2 = 27
    dy_dx = tape2.gradient(y, x)
# Compute second derivative d2y/dx2 = 6x = 18
d2y_dx2 = tape1.gradient(dy_dx, x)

# Delete the persistent tape to free memory
del tape1
```

> 💡 **Pro Tips:** Use `tf.stop_gradient(tensor)` to explicitly prevent gradients from flowing through a specific part of your computation graph.
> ⚠️ **Common Pitfalls:** Trying to compute a gradient outside the tape context, or forgetting that tapes automatically track `tf.Variable` but ignore `tf.constant` unless watched.

---

## 6. KERAS API (CORE HIGH-LEVEL API)

Keras provides three methods to build models, ranging from simple to fully customizable.

### 1. Sequential API
The simplest API. Best for plain stacks of layers where each layer has exactly one input tensor and one output tensor.

```python
import tensorflow as tf
from tensorflow.keras import layers, models

# Instantiate a Sequential model
model_seq = models.Sequential([
    layers.Dense(64, activation='relu', input_shape=(32,)),
    layers.Dense(10, activation='softmax')
])
```

### 2. Functional API
More flexible. Best for models with non-linear topology, shared layers, multiple inputs, or multiple outputs (e.g., ResNet, Siamese networks).

```python
# Define the input layer with a specific shape
inputs = tf.keras.Input(shape=(32,))
# Connect layers explicitly by passing the previous layer's output
x = layers.Dense(64, activation='relu')(inputs)
x = layers.Dropout(0.5)(x) # Add dropout for regularization
outputs = layers.Dense(10, activation='softmax')(x)

# Construct the model by specifying inputs and outputs
model_func = tf.keras.Model(inputs=inputs, outputs=outputs, name="functional_model")
```

### 3. Model Subclassing
Maximum flexibility. Best for research and highly custom architectures. You define the forward pass imperatively.

```python
# Inherit from tf.keras.Model
class CustomModel(tf.keras.Model):
    def __init__(self, num_classes):
        super(CustomModel, self).__init__()
        # Instantiate layers in the constructor
        self.dense1 = layers.Dense(64, activation='relu')
        self.classifier = layers.Dense(num_classes, activation='softmax')

    # Define the forward pass in the call method
    def call(self, inputs, training=False):
        x = self.dense1(inputs)
        # Some custom logic during training
        if training:
            x = tf.nn.dropout(x, rate=0.5)
        return self.classifier(x)

# Instantiate the subclassed model
model_sub = CustomModel(num_classes=10)
```

> 💡 **Pro Tips:** Use the Functional API 90% of the time. It is perfectly balanced between flexibility and built-in error checking (like static shape inference).
> ⚠️ **Common Pitfalls:** In Subclassing, the model's structure is invisible to Keras until `model.build(input_shape)` is called or the first batch of data is passed through. This means `model.summary()` won't work immediately.

---

## 7. MODEL BUILDING

### Layers, Activations, Initializers, Regularizers
Building robust neural networks involves choosing the right components.

```python
from tensorflow.keras import layers, regularizers, initializers

# Defining a comprehensive Dense layer demonstrating best practices
complex_layer = layers.Dense(
    units=128,
    
    # Activation Function: Non-linearity. ReLU is standard, Swish/Mish are modern alternatives.
    activation='relu', 
    
    # Initializers: How weights start. He Normal is best for ReLU. Glorot Uniform is best for tanh/sigmoid.
    kernel_initializer=initializers.HeNormal(),
    
    # Regularizers: Penalize large weights to prevent overfitting.
    # L1 induces sparsity, L2 (Weight Decay) prevents large spikes.
    kernel_regularizer=regularizers.l2(1e-4),
    
    # Whether to use a bias vector (usually True, unless followed by BatchNorm)
    use_bias=True,
    name="hidden_layer_1"
)

# Example of layers commonly used together
model = tf.keras.Sequential([
    tf.keras.Input(shape=(100,)),
    # Dense layer with L2 regularization
    layers.Dense(256, kernel_regularizer=regularizers.l2(0.01)),
    # Batch Normalization stabilizes training and allows higher learning rates
    layers.BatchNormalization(),
    # Applying activation AFTER BatchNorm is standard practice (though debated)
    layers.Activation('relu'),
    # Dropout sets random units to 0 to prevent co-adaptation (overfitting)
    layers.Dropout(0.3),
    # Output layer for binary classification
    layers.Dense(1, activation='sigmoid')
])
```

> 💡 **Pro Tips:** When using `BatchNormalization`, set `use_bias=False` in the preceding `Conv2D` or `Dense` layer, as the BatchNorm layer has its own learnable bias.
> ⚠️ **Common Pitfalls:** Applying Dropout immediately before the output layer can sometimes distort the predictions. Apply it to hidden layers.

---

## 8. COMPILATION & TRAINING

### model.compile()
Before training, the model needs an optimizer, loss function, and metrics.

```python
# Compile the model
model.compile(
    # Optimizer: Adam is the standard go-to. SGD with Momentum is often better for final fine-tuning.
    optimizer=tf.keras.optimizers.Adam(learning_rate=0.001),
    
    # Loss: Binary classification -> BinaryCrossentropy
    # Multi-class (one-hot) -> CategoricalCrossentropy
    # Multi-class (integer labels) -> SparseCategoricalCrossentropy
    loss=tf.keras.losses.BinaryCrossentropy(from_logits=False),
    
    # Metrics to track during training
    metrics=['accuracy', tf.keras.metrics.AUC()]
)
```

### Callbacks
Callbacks execute code at various stages of training (e.g., end of epoch).

```python
from tensorflow.keras.callbacks import EarlyStopping, ModelCheckpoint, ReduceLROnPlateau

callbacks_list = [
    # Stop training when validation loss stops improving for 5 epochs
    EarlyStopping(
        monitor='val_loss', 
        patience=5, 
        restore_best_weights=True # Always restore the best weights!
    ),
    # Save the model automatically during training
    ModelCheckpoint(
        filepath='best_model.h5',
        monitor='val_loss',
        save_best_only=True
    ),
    # Reduce learning rate by a factor of 10 if val_loss plateaus for 2 epochs
    ReduceLROnPlateau(
        monitor='val_loss', 
        factor=0.1, 
        patience=2
    )
]
```

### model.fit()

```python
import numpy as np
# Dummy data for demonstration
x_train = np.random.random((1000, 100))
y_train = np.random.randint(0, 2, (1000, 1))

# Train the model
history = model.fit(
    x=x_train,
    y=y_train,
    batch_size=32,       # Number of samples per gradient update
    epochs=50,           # Number of times to iterate over the dataset
    validation_split=0.2,# Automatically reserve 20% of data for validation
    callbacks=callbacks_list,
    verbose=1            # 1 = progress bar, 2 = one line per epoch
)
```

> 💡 **Pro Tips:** Always use `EarlyStopping` with `restore_best_weights=True` to prevent overfitting and avoid manually tracking epoch performance.
> ⚠️ **Common Pitfalls:** If your network's final layer outputs logits (no activation like softmax or sigmoid), you MUST set `from_logits=True` in your loss function.

---

## 9. DATA PIPELINES

`tf.data.Dataset` is the standard for building highly efficient, asynchronous data pipelines.

### Loading, Batching, Shuffling, Prefetching

```python
import tensorflow as tf

# Dummy data arrays
features = tf.random.normal([10000, 32])
labels = tf.random.uniform([10000], minval=0, maxval=10, dtype=tf.int32)

# 1. Create dataset from tensors
dataset = tf.data.Dataset.from_tensor_slices((features, labels))

# 2. Define a data augmentation / preprocessing function
def preprocess(x, y):
    x = x / tf.reduce_max(x) # Simple normalization
    # One-hot encode the labels
    y = tf.one_hot(y, depth=10)
    return x, y

# Define AUTOTUNE to let TF dynamically allocate CPU resources
AUTOTUNE = tf.data.AUTOTUNE

# 3. Build the pipeline step-by-step
processed_dataset = (
    dataset
    # Shuffle the data entirely. Buffer size should ideally be >= dataset size for perfect shuffling
    .shuffle(buffer_size=10000)
    # Apply preprocessing mapping. Use parallel calls for speed!
    .map(preprocess, num_parallel_calls=AUTOTUNE)
    # Batch the data into chunks of 64
    .batch(64)
    # Prefetch overlaps data preprocessing and model execution.
    # While the GPU trains batch N, CPU prepares batch N+1. This is CRITICAL.
    .prefetch(AUTOTUNE)
)

# You can pass this dataset directly to model.fit()
# model.fit(processed_dataset, epochs=10)
```

> 💡 **Pro Tips:** The order of operations matters. Generally: `shuffle` -> `map` -> `batch` -> `prefetch`. Caching (`.cache()`) before mapping can save time if your preprocessing is heavy and dataset fits in RAM.
> ⚠️ **Common Pitfalls:** Forgetting to `.prefetch(AUTOTUNE)`. Without prefetching, your GPU will constantly sit idle at 0% utilization while waiting for the CPU to load and process the next batch.

---

## 10. EVALUATION & PREDICTION

```python
# Assuming model is trained and x_test, y_test are available
x_test = np.random.random((200, 100))
y_test = np.random.randint(0, 2, (200, 1))

# Evaluate the model on the test data
# Returns the loss value and metrics values for the model in test mode
results = model.evaluate(x_test, y_test, batch_size=32)
print(f"Test Loss: {results[0]}, Test Accuracy: {results[1]}")

# Generate predictions for new data
# Returns numpy arrays of predictions (e.g., probabilities)
predictions = model.predict(x_test)
print(f"Prediction for first sample: {predictions[0]}")

# To convert probabilities back to class labels (binary classification)
class_predictions = (predictions > 0.5).astype("int32")
```

> 💡 **Pro Tips:** `model.predict` is optimized for large batches of data. If you are predicting on a single instance (e.g., in a web server), just calling the model directly (`model(x_test[:1], training=False)`) is much faster due to less overhead.
> ⚠️ **Common Pitfalls:** Forgetting `training=False` when calling the model imperatively. Layers like Dropout and BatchNorm behave differently during training vs inference.

---

## 11. SAVING & LOADING MODELS

### Formats
- **SavedModel (Recommended):** TF's native format. Saves the architecture, weights, and compilation state. It is language-agnostic and required for TF Serving.
- **HDF5 / Keras (.h5 / .keras):** Older standard. Saves everything into a single file. Modern Keras 3+ prefers the `.keras` extension.

```python
import tensorflow as tf

# 1. Save entire model (SavedModel format - creates a folder)
model.save("my_saved_model")

# 2. Save entire model (HDF5 or modern Keras format - creates a file)
model.save("my_model.keras")

# 3. Save only the weights
model.save_weights("my_weights.weights.h5")

# Loading models back
# Load entire model (works for both SavedModel and .keras)
loaded_model = tf.keras.models.load_model("my_model.keras")

# Load only weights (requires you to instantiate the architecture first)
# new_model = create_model()
# new_model.load_weights("my_weights.weights.h5")
```

> 💡 **Pro Tips:** Always use the `.keras` extension or the `SavedModel` directory format. Legacy `.h5` files have limitations with custom layers.
> ⚠️ **Common Pitfalls:** If you use custom layers or functions, loading the model requires passing the `custom_objects` dictionary to `load_model()`.

---

## 12. CUSTOM TRAINING LOOPS

When Keras `model.fit()` is too restrictive (e.g., GANs, multi-optimizer setups, complex reinforcement learning), write your own loop.

```python
import tensorflow as tf

# Instantiate model, loss, and optimizer
model = tf.keras.Sequential([tf.keras.layers.Dense(1)])
optimizer = tf.keras.optimizers.Adam(learning_rate=0.01)
loss_fn = tf.keras.losses.MeanSquaredError()

# The @tf.function decorator compiles this python function into a static TensorFlow graph.
# This yields massive performance improvements.
@tf.function
def train_step(x, y):
    # 1. Open a GradientTape
    with tf.GradientTape() as tape:
        # 2. Forward pass (training=True is crucial for Dropout/BatchNorm)
        predictions = model(x, training=True)
        # 3. Calculate loss
        loss = loss_fn(y, predictions)
    
    # 4. Calculate gradients of the loss wrt the model's trainable weights
    gradients = tape.gradient(loss, model.trainable_variables)
    
    # 5. Update the weights using the optimizer
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    
    return loss

# The Training Loop
epochs = 5
dataset = tf.data.Dataset.from_tensor_slices((
    tf.random.normal([100, 10]), tf.random.normal([100, 1])
)).batch(10)

for epoch in range(epochs):
    epoch_loss_avg = tf.keras.metrics.Mean()
    
    # Iterate over batches
    for step, (x_batch, y_batch) in enumerate(dataset):
        # Perform one training step
        loss_value = train_step(x_batch, y_batch)
        epoch_loss_avg.update_state(loss_value)
        
    print(f"Epoch {epoch+1}: Loss: {epoch_loss_avg.result().numpy():.4f}")
```

> 💡 **Pro Tips:** You can still use Keras Metrics (`tf.keras.metrics.Mean()`) inside custom loops to easily track rolling averages across batches.
> ⚠️ **Common Pitfalls:** Forgetting `@tf.function` on your `train_step`. Without it, the loop will run in eager Python mode and will be excruciatingly slow.

---

## 13. CNNs (CONVOLUTIONAL NETWORKS)

CNNs are the standard for image processing. TensorFlow expects image data in NHWC format by default: `(Batch, Height, Width, Channels)`.

```python
import tensorflow as tf
from tensorflow.keras import layers, models

def build_cnn(input_shape=(32, 32, 3), num_classes=10):
    model = models.Sequential([
        # Input layer specifying image dimensions
        tf.keras.Input(shape=input_shape),
        
        # Conv2D extracts spatial features
        # 32 filters, 3x3 kernel window, padding='same' keeps spatial dims identical
        layers.Conv2D(32, (3, 3), activation='relu', padding='same'),
        
        # MaxPooling reduces spatial dimensions, providing translation invariance
        layers.MaxPooling2D((2, 2)),
        
        layers.Conv2D(64, (3, 3), activation='relu'),
        layers.MaxPooling2D((2, 2)),
        layers.Conv2D(64, (3, 3), activation='relu'),
        
        # Modern practice: Use GlobalAveragePooling instead of Flatten to drastically
        # reduce parameter count and prevent overfitting.
        layers.GlobalAveragePooling2D(),
        
        # Final classification dense layer
        layers.Dense(num_classes, activation='softmax')
    ])
    return model

cnn_model = build_cnn()
```

> 💡 **Pro Tips:** Data augmentation (rotations, flips) is crucial for CNNs. Use `tf.keras.layers.RandomFlip` and `RandomRotation` directly inside your model architecture! They automatically turn off during inference.
> ⚠️ **Common Pitfalls:** Feeding images with pixel values `[0, 255]`. Neural networks prefer small values. Always normalize inputs to `[0, 1]` or `[-1, 1]` via `layers.Rescaling(1./255)`.

---

## 14. RNNs & SEQUENCE MODELS

For time series, text, and sequences, TF expects data in `(Batch, Timesteps, Features)` shape.

```python
from tensorflow.keras import layers, models

def build_rnn_model(timesteps=50, features=1, num_classes=2):
    model = models.Sequential([
        tf.keras.Input(shape=(timesteps, features)),
        
        # LSTM layer. return_sequences=True is required if passing to another RNN layer
        layers.LSTM(64, return_sequences=True),
        
        # Dropout for RNNs to prevent overfitting on sequences
        layers.Dropout(0.2),
        
        # GRU is a slightly faster, computationally cheaper alternative to LSTM
        # return_sequences=False (default) returns only the final hidden state of the sequence
        layers.GRU(32),
        
        layers.Dense(num_classes, activation='softmax')
    ])
    return model

rnn_model = build_rnn_model()
```

> 💡 **Pro Tips:** For NLP tasks, wrap your RNN layers in `layers.Bidirectional()`. It reads the sequence forwards and backwards, often significantly boosting accuracy.
> ⚠️ **Common Pitfalls:** Ignoring `Masking`. If your sequences have variable lengths and are padded with zeros, add a `layers.Masking(mask_value=0.)` layer first so the RNN skips the padding computations.

---

## 15. TRANSFER LEARNING

Transfer learning leverages models pre-trained on massive datasets (like ImageNet).

```python
import tensorflow as tf
from tensorflow.keras.applications import MobileNetV2

# 1. Load the pre-trained base model
# include_top=False removes the final 1000-class ImageNet classifier
# weights='imagenet' downloads the pre-trained weights
base_model = MobileNetV2(
    input_shape=(160, 160, 3),
    include_top=False,
    weights='imagenet'
)

# 2. Freeze the base model so its weights don't update during initial training
base_model.trainable = False

# 3. Build your custom model on top
inputs = tf.keras.Input(shape=(160, 160, 3))
# MobileNetV2 expects inputs in [-1, 1] range, use the built-in preprocess function
x = tf.keras.applications.mobilenet_v2.preprocess_input(inputs)
x = base_model(x, training=False) # Keep training=False to keep BatchNorm stats frozen
x = tf.keras.layers.GlobalAveragePooling2D()(x)
x = tf.keras.layers.Dropout(0.2)(x)
outputs = tf.keras.layers.Dense(1, activation='sigmoid')(x)

model = tf.keras.Model(inputs, outputs)

# 4. Train the top layers... (model.fit)

# 5. Fine-Tuning: Unfreeze base model and train whole network with a VERY LOW learning rate
base_model.trainable = True
model.compile(
    optimizer=tf.keras.optimizers.Adam(1e-5), # Tiny learning rate to avoid destroying weights
    loss='binary_crossentropy',
    metrics=['accuracy']
)
```

> 💡 **Pro Tips:** Always perform transfer learning in two steps: train your custom head first with a frozen base, *then* unfreeze the base and fine-tune with a micro learning rate.
> ⚠️ **Common Pitfalls:** Forgetting to set `training=False` when calling the base model. If you don't do this, the frozen BatchNorm layers will start tracking the new dataset's statistics and ruin the pre-trained representations.

---

## 16. DISTRIBUTED TRAINING

Scaling up to multiple GPUs or TPUs is handled by `tf.distribute.Strategy`.

```python
import tensorflow as tf

# MirroredStrategy supports synchronous distributed training on multiple GPUs on ONE machine.
strategy = tf.distribute.MirroredStrategy()
print(f'Number of devices: {strategy.num_replicas_in_sync}')

# Everything that creates variables should be under the strategy scope.
with strategy.scope():
    # 1. Build the model within the scope
    model = tf.keras.Sequential([
        tf.keras.layers.Dense(128, activation='relu', input_shape=(10,)),
        tf.keras.layers.Dense(1)
    ])
    
    # 2. Compile the model within the scope
    # The strategy automatically scales the loss and gradients
    model.compile(loss='mse', optimizer='adam')

# 3. Train as usual! The data will automatically be sharded across the GPUs.
# IMPORTANT: Scale your batch size. If you want batch 32 per GPU and have 4 GPUs,
# pass batch_size=128 to model.fit()
# model.fit(dataset, epochs=5)
```

> 💡 **Pro Tips:** Use `MultiWorkerMirroredStrategy` to scale across multiple machines/nodes. Use `TPUStrategy` for Google Cloud TPUs.
> ⚠️ **Common Pitfalls:** Not scaling your global batch size or learning rate. If you increase GPU count by 4x, you often need to increase batch size by 4x and carefully tune the learning rate (Linear Scaling Rule).

---

## 17. DEPLOYMENT

### TensorFlow Serving
The standard for deploying models via REST/gRPC API in production.
1. Save via `model.save('model_dir/1/')` (The '1' is the version number).
2. Run via Docker:
`docker run -p 8501:8501 -v /path/to/model_dir:/models/my_model -e MODEL_NAME=my_model tensorflow/serving`

### TensorFlow Lite (Mobile & Edge)
Converts models to a highly optimized flatbuffer for Android/iOS/IoT.

```python
import tensorflow as tf

# Load a SavedModel
converter = tf.lite.TFLiteConverter.from_saved_model("my_saved_model")

# Apply optimizations (Post-Training Quantization to convert float32 to int8)
# This reduces model size by 4x and speeds up inference on mobile with minimal accuracy loss.
converter.optimizations = [tf.lite.Optimize.DEFAULT]

# Convert the model
tflite_model = converter.convert()

# Save to file
with open('model.tflite', 'wb') as f:
    f.write(tflite_model)
```

### TensorFlow.js
Allows models to run natively in the browser via WebGL.
`pip install tensorflowjs`
`tensorflowjs_converter --input_format=tf_saved_model /saved_model_dir /web_model_dir`

> 💡 **Pro Tips:** Always define specific input `signatures` when saving a model for TF Serving. It makes inference requests explicitly typed.
> ⚠️ **Common Pitfalls:** TF Lite does not support all TF operators. Very complex custom layers may fail to convert.

---

## 18. PERFORMANCE OPTIMIZATION

### Mixed Precision
Trains models using `float16` for computation and `float32` for variables. Gives massive speedups (up to 3x) and halves VRAM usage on modern NVIDIA GPUs (Volta+ / RTX 2000+).

```python
from tensorflow.keras import mixed_precision

# Enable mixed precision globally
policy = mixed_precision.Policy('mixed_float16')
mixed_precision.set_global_policy(policy)

print('Compute dtype: %s' % policy.compute_dtype) # float16
print('Variable dtype: %s' % policy.variable_dtype) # float32

# Build and train your model normally! Keras handles the float16/32 casting.
# IMPORTANT: The final output layer should explicitly be float32 for numerical stability.
# layers.Dense(10, activation='softmax', dtype='float32')
```

### XLA (Accelerated Linear Algebra)
XLA compiles the TensorFlow graph into highly optimized machine code, fusing multiple operations together.

```python
# Enable XLA simply by passing jit_compile=True to model.compile
model.compile(
    optimizer='adam', 
    loss='mse',
    jit_compile=True # ENABLES XLA
)
```

> 💡 **Pro Tips:** Combine Mixed Precision and XLA compilation. Together, they represent the absolute ceiling of TensorFlow performance on modern hardware.
> ⚠️ **Common Pitfalls:** XLA compilation takes time on the *first* batch. Don't be alarmed if the first epoch starts slowly.

---

## 19. DEBUGGING & PROFILING

### TensorBoard
The ultimate visualization toolkit.

```python
import tensorflow as tf
import datetime

# Create a TensorBoard callback
log_dir = "logs/fit/" + datetime.datetime.now().strftime("%Y%m%d-%H%M%S")
tensorboard_callback = tf.keras.callbacks.TensorBoard(
    log_dir=log_dir, 
    histogram_freq=1, # Log weight histograms every epoch
    profile_batch='500,520' # Profile performance between batch 500 and 520
)

# Pass to fit
# model.fit(x, y, epochs=5, callbacks=[tensorboard_callback])
```
Run `tensorboard --logdir logs/fit` in your terminal to view metrics, graphs, and profiling data.

### tf.debugging & eager execution
```python
# Check for NaNs or Infs in tensors during custom loops
tf.debugging.check_numerics(tensor, message="Found NaN!")

# Print values inside a @tf.function graph (standard print() only prints the graph node object)
tf.print("Value of tensor:", tensor)
```

> 💡 **Pro Tips:** The TensorBoard Profiler plugin is incredible for diagnosing CPU/GPU bottlenecks. It will explicitly tell you if your data pipeline is too slow.
> ⚠️ **Common Pitfalls:** Using Python's built-in `print()` inside a `@tf.function`. It will only print once during the tracing phase. Always use `tf.print()` for dynamic runtime values.

---

## 20. ADVANCED TOPICS

### Custom Layers
You can build layers with internal state (weights) by subclassing `tf.keras.layers.Layer`.

```python
import tensorflow as tf

class CustomDense(tf.keras.layers.Layer):
    def __init__(self, units=32, **kwargs):
        super(CustomDense, self).__init__(**kwargs)
        self.units = units

    # build() is called dynamically the first time the layer sees input
    def build(self, input_shape):
        # Create a trainable weight variable for this layer
        self.w = self.add_weight(
            shape=(input_shape[-1], self.units),
            initializer='random_normal',
            trainable=True,
            name='w'
        )
        self.b = self.add_weight(
            shape=(self.units,),
            initializer='zeros',
            trainable=True,
            name='b'
        )
        # Always call super.build() at the end
        super(CustomDense, self).build(input_shape)

    # call() defines the forward pass mathematics
    def call(self, inputs):
        # f(x) = x * W + b
        return tf.matmul(inputs, self.w) + self.b
```

### Graph Mode vs Eager Mode Internals
When you apply `@tf.function` to a Python function, TF runs it once in a "Tracing" phase. During tracing, TF executes the function, replacing actual tensors with symbolic tensors. It records every operation into a `GraphDef`. 
If you call the function again with different shapes/dtypes, it must "retrace", which is highly inefficient.

```python
# Prevent retracing by specifying a fixed input signature
@tf.function(input_signature=[tf.TensorSpec(shape=[None, 10], dtype=tf.float32)])
def strict_function(x):
    return x * 2.0
```

> 💡 **Pro Tips:** Use `tf.TensorSpec` to enforce exact input shapes and prevent silent retracing bugs in production, especially when receiving data from dynamic sources.
> ⚠️ **Common Pitfalls:** Using python lists or native python control flow (`if len(x) > 5`) inside `@tf.function`. The `len(x)` is evaluated ONCE during tracing. To evaluate dynamically on the GPU, use TF ops: `if tf.shape(x)[0] > 5`.
