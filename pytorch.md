# PyTorch Production Technical Reference

## Contents
- [1. Introduction & History](#1-introduction--history)
- [2. Installation & Setup](#2-installation--setup)
- [3. Tensors](#3-tensors)
- [4. Automatic Differentiation](#4-automatic-differentiation)
- [5. Neural Network Modules](#5-neural-network-modules)
- [6. Data Loading](#6-data-loading)
- [7. Training Loop](#7-training-loop)
- [8. Optimization](#8-optimization)
- [9. GPU and Device Management](#9-gpu-and-device-management)
- [10. Saving and Loading Models](#10-saving-and-loading-models)
- [11. Convolutional Networks](#11-convolutional-networks)
- [12. Recurrent Networks](#12-recurrent-networks)
- [13. Transformers](#13-transformers)
- [14. Distributed Training](#14-distributed-training)
- [15. Performance Optimization](#15-performance-optimization)
- [16. Interoperability](#16-interoperability)
- [17. Debugging and Profiling](#17-debugging-and-profiling)
- [18. Advanced Topics](#18-advanced-topics)

## 1. INTRODUCTION & HISTORY

### What is PyTorch
PyTorch is a premiere open-source machine learning and deep learning framework primarily developed by Meta's AI Research lab (FAIR). Based on the Torch library, PyTorch is fundamentally designed to provide a highly flexible, dynamic, and pythonic approach to building complex neural networks. It has become the de facto standard for deep learning research and is increasingly dominating the production ML landscape.

### Why PyTorch
* **Dynamic Computation Graphs (Define-by-Run):** Unlike earlier static graph frameworks, PyTorch builds the computation graph on the fly as operations are executed. This allows for dynamic control flow (standard Python `if`, `while`, `for` loops) within the network architecture.
* **Pythonic Nature:** PyTorch feels like native Python. It seamlessly integrates with the Python ecosystem, making debugging with standard tools (like `pdb`) straightforward and intuitive.
* **Research-Friendly:** The flexibility allows researchers to easily experiment with novel architectures without fighting the framework's constraints.
* **Hardware Acceleration:** PyTorch provides native, optimized support for CUDA (NVIDIA GPUs) and ROCm (AMD GPUs), as well as Apple Silicon (MPS).

### PyTorch vs TensorFlow Comparison

| Feature | PyTorch | TensorFlow |
| :--- | :--- | :--- |
| **Computation Graph** | Dynamic (Define-by-Run) | Originally Static, now Dynamic (Eager Execution) |
| **Learning Curve** | Gentle, Pythonic | Steeper, historically more complex |
| **Debugging** | Standard Python tools (`pdb`) | specialized tools (`tfdbg`), improved in TF 2.x |
| **Deployment** | TorchScript, ONNX, TorchServe | TF Serving, TFLite, TF.js (More mature) |
| **Community** | Dominant in Research | Strong in Enterprise/Production |
| **High-level API** | `torch.nn` / PyTorch Lightning | Keras (`tf.keras`) |

### Ecosystem Overview
PyTorch is more than just the core library; it boasts a rich ecosystem tailored for specific domains:
* **TorchVision:** Datasets, models, and image transformations for computer vision.
* **TorchAudio:** Audio and signal processing tools, datasets, and pre-trained models.
* **TorchText:** NLP-focused data processing utilities and datasets.
* **PyTorch Lightning:** A lightweight wrapper that organizes PyTorch code, removing boilerplate and enabling scalable training (multi-GPU, TPU) with zero code changes.
* **TorchServe:** A flexible and easy-to-use tool for serving PyTorch models in production.

### Version Evolution
* **v0.4:** Merged `Variable` and `Tensor` (a monumental simplification).
* **v1.0:** Introduction of TorchScript for production deployment, merging Caffe2 backend.
* **v1.6:** Native Automatic Mixed Precision (AMP) support.
* **v2.0:** Introduction of `torch.compile()`, a massive leap in training and inference performance through graph compilation (Triton under the hood).

💡 **Pro Tips:**
* If you are starting a new project, embrace PyTorch 2.x and `torch.compile` immediately for free performance gains.
* Familiarize yourself with the broader ecosystem instead of reinventing the wheel (e.g., use `torchvision.transforms` instead of writing custom OpenCV pipelines).

⚠️ **Common Pitfalls:**
* Assuming PyTorch is only for research. With TorchScript, ONNX, and `torch.compile`, PyTorch is highly capable in production environments.

---

## 2. INSTALLATION & SETUP

### Standard Installation
The recommended way to install PyTorch is via `pip` or `conda`. You must ensure your CUDA toolkit versions match the PyTorch binaries if you intend to use GPU acceleration.

```bash
# Example: Installing PyTorch with CUDA 11.8 support via pip
# Always refer to pytorch.org/get-started/locally/ for the exact command
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

### GPU vs CPU Setup
PyTorch tensors can reside on either the CPU (system RAM) or a GPU (VRAM). Operations are dramatically faster on the GPU, but transferring data between CPU and GPU incurs a bottleneck.

### CUDA Basics
CUDA (Compute Unified Device Architecture) is NVIDIA's parallel computing platform. PyTorch acts as a high-level Python wrapper over low-level CUDA C/C++ operations.

### Checking Installation
Always verify your installation immediately, ensuring CUDA is available if expected.

```python
import torch # Main PyTorch package
import torchvision # Computer vision package
import torchaudio # Audio processing package

# Print the installed version of PyTorch
print(f"PyTorch Version: {torch.__version__}")

# Check if CUDA (GPU support) is available
is_cuda_available = torch.cuda.is_available()
print(f"CUDA Available: {is_cuda_available}")

if is_cuda_available:
    # Get the number of available GPUs
    print(f"Number of GPUs: {torch.cuda.device_count()}")
    # Get the name of the first GPU (index 0)
    print(f"GPU Name: {torch.cuda.get_device_name(0)}")
else:
    # Check for Apple Silicon MPS (Metal Performance Shaders)
    is_mps_available = torch.backends.mps.is_available()
    print(f"MPS Available (Apple Silicon): {is_mps_available}")

# Set the device to be used throughout the script
# This is a standard pattern for device-agnostic code
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print(f"Using device: {device}")
```

### Import Conventions
```python
import torch
import torch.nn as nn            # Neural network layers, loss functions
import torch.nn.functional as F  # Stateless functions (activations, pooling)
import torch.optim as optim      # Optimization algorithms (SGD, Adam)
from torch.utils.data import DataLoader, Dataset # Data handling
```

💡 **Pro Tips:**
* Always define a global `device` variable early in your script and use `.to(device)` consistently to avoid device mismatch errors.
* Use `conda` environments or Python `venv` to isolate PyTorch installations, as CUDA dependencies can easily break other ML packages.

⚠️ **Common Pitfalls:**
* Installing the CPU-only version of PyTorch by accident and wondering why training is extremely slow. Always check `torch.cuda.is_available()`.

---

## 3. TENSORS & OPERATIONS

### Creating Tensors
Tensors are the fundamental data structure in PyTorch, analogous to NumPy arrays but with GPU acceleration and automatic differentiation capabilities.

```python
import torch
import numpy as np

# 1. From standard Python lists
# Creates a 2D tensor (matrix) from a list of lists
tensor_from_list = torch.tensor([[1.0, 2.0], [3.0, 4.0]])

# 2. From NumPy arrays
# Creates a tensor directly from a NumPy array (shares memory if on CPU)
np_array = np.array([1, 2, 3])
tensor_from_np = torch.from_numpy(np_array)

# 3. Initialized with specific values
zeros_tensor = torch.zeros((3, 3))       # 3x3 tensor of zeros
ones_tensor = torch.ones((2, 4))         # 2x4 tensor of ones
rand_tensor = torch.rand((2, 2))         # 2x2 tensor with values from uniform distribution [0, 1)
randn_tensor = torch.randn((3, 3))       # 3x3 tensor with values from standard normal distribution
arange_tensor = torch.arange(0, 10, 2)   # Tensor: [0, 2, 4, 6, 8]
empty_tensor = torch.empty((2, 2))       # Uninitialized tensor (contains garbage memory values)

# 4. Like other tensors
# Creates a tensor of the same shape as rand_tensor, but filled with zeros
zeros_like = torch.zeros_like(rand_tensor) 
```

### Tensor Attributes
Every tensor has three key attributes: shape, dtype, and device.

```python
t = torch.randn(3, 4, dtype=torch.float32, device='cpu')

# Get the shape of the tensor (returns a torch.Size object)
print(f"Shape: {t.shape}") # Equivalently: t.size()

# Get the data type of the tensor elements
print(f"DataType: {t.dtype}")

# Get the device the tensor is stored on
print(f"Device: {t.device}")
```

### Basic Operations

```python
a = torch.tensor([[1, 2], [3, 4]], dtype=torch.float32)
b = torch.tensor([[5, 6], [7, 8]], dtype=torch.float32)

# Element-wise addition
c_add = a + b 
c_add_func = torch.add(a, b)

# Element-wise multiplication
c_mul = a * b
c_mul_func = torch.mul(a, b)

# Matrix multiplication (Dot product)
# Very important for linear layers
c_matmul = a @ b 
c_matmul_func = torch.matmul(a, b)

# In-place operations (modify the tensor directly to save memory)
# Denoted by a trailing underscore (_)
a.add_(5) # Adds 5 to all elements of 'a' in-place

# Reshaping tensors (crucial for passing data between incompatible layers)
# .view() requires contiguous memory. .reshape() handles non-contiguous memory automatically.
x = torch.randn(4, 4)
y = x.view(16)         # Flatten to 1D
z = x.view(-1, 8)      # -1 infers the dimension (here, it becomes 2)
```

### Broadcasting
Broadcasting allows operations between tensors of different shapes by conceptually expanding the smaller tensor.

```python
a = torch.ones((3, 3))
b = torch.tensor([1, 2, 3]) # Shape (3,)

# 'b' is broadcasted to (3, 3) to match 'a'
# Row 1 of 'a' + b, Row 2 of 'a' + b, etc.
c = a + b 
```

💡 **Pro Tips:**
* Use `.reshape()` instead of `.view()` if you aren't strictly managing memory contiguity, as `.view()` can throw `RuntimeError: input is not contiguous`.
* Use `torch.einsum` for complex multi-dimensional tensor operations; it is often cleaner and faster than chaining `.permute()`, `.view()`, and `.matmul()`.

⚠️ **Common Pitfalls:**
* **Type Mismatches:** Trying to add a `torch.float32` tensor to a `torch.int64` tensor. Always check `t.dtype`.
* **In-place modifications with Autograd:** Performing in-place operations (e.g., `x += 1`) on tensors that require gradients can break the computation graph.

---

## 4. AUTOGRAD (AUTOMATIC DIFFERENTIATION)

Autograd is the engine that powers neural network training in PyTorch. It automatically computes gradients for backpropagation.

### requires_grad
Tensors created with `requires_grad=True` track all operations performed on them to build a dynamic computation graph (a Directed Acyclic Graph or DAG).

```python
import torch

# Create a tensor and specify that it requires gradients
# This tells PyTorch to track operations on this tensor
x = torch.tensor(2.0, requires_grad=True)

# Perform some operations
# This creates a computation graph: x -> y -> z
y = x ** 2       # y = x^2 = 4.0
z = 3 * y + 1    # z = 3 * 4.0 + 1 = 13.0

# Print the tensor z. 
# Notice the 'grad_fn' attribute, which links back to the operation that created it.
print(z) # tensor(13., grad_fn=<AddBackward0>)
```

### backward()
Calling `.backward()` traverses the graph backwards from the output node, applying the chain rule to compute gradients for all leaf tensors (tensors with `requires_grad=True` created by the user).

```python
# Compute gradients: dz/dx
# dz/dx = dz/dy * dy/dx = 3 * (2*x) = 6 * x. Since x = 2.0, dz/dx = 12.0
z.backward()

# Access the gradient
print(f"Gradient of x: {x.grad}") # tensor(12.)
```

### Computational Graph
The graph is built *forward* during the forward pass and consumed *backward* during the backward pass. Once `.backward()` is called, the graph is freed (unless `retain_graph=True` is passed).

### Gradient Accumulation
By default, gradients accumulate (add up) in the `.grad` attribute upon multiple `.backward()` calls. This is useful for dealing with large batch sizes that don't fit in memory (gradient accumulation over mini-batches), but requires manually zeroing gradients during standard training.

```python
x = torch.tensor(2.0, requires_grad=True)

for i in range(3):
    y = x ** 2
    y.backward()
    print(f"Iteration {i}, x.grad: {x.grad}")
    
    # In a real training loop, you must clear gradients!
    # x.grad.zero_() 
```

💡 **Pro Tips:**
* Use `with torch.no_grad():` during inference or evaluation. This prevents PyTorch from building the computation graph, drastically reducing memory usage and speeding up computation.
* `detach()` creates a new tensor that shares the same memory but does not require gradients, effectively cutting it off from the graph.

⚠️ **Common Pitfalls:**
* **Forgetting to zero gradients:** `optimizer.zero_grad()` is mandatory before the backward pass in a standard training loop; otherwise, gradients will incorrectly accumulate across batches.

---

## 5. NN MODULE (torch.nn)

The `torch.nn` package provides all the standard building blocks for creating deep learning models.

### nn.Module
Every custom neural network architecture in PyTorch must inherit from `torch.nn.Module`. It handles state (parameters), moving models between devices, and acts as the fundamental unit of abstraction.

### Built-in Layers

```python
import torch
import torch.nn as nn

# Linear (Fully Connected) Layer
# Applies linear transformation: y = xA^T + b
# in_features: size of each input sample
# out_features: size of each output sample
linear_layer = nn.Linear(in_features=128, out_features=64)

# Convolutional Layer (2D for images)
# in_channels: number of channels in input image (e.g., 3 for RGB)
# out_channels: number of filters
# kernel_size: size of the convolving kernel (3x3)
conv_layer = nn.Conv2d(in_channels=3, out_channels=16, kernel_size=3, stride=1, padding=1)

# Recurrent Layer (LSTM)
# input_size: number of expected features in the input x
# hidden_size: number of features in the hidden state h
lstm_layer = nn.LSTM(input_size=10, hidden_size=20, num_layers=2, batch_first=True)

# Embedding Layer (for text/categorical data)
# num_embeddings: size of the dictionary of embeddings
# embedding_dim: size of each embedding vector
embedding_layer = nn.Embedding(num_embeddings=10000, embedding_dim=300)

# Dropout Layer (Regularization to prevent overfitting)
# p: probability of an element to be zeroed
dropout = nn.Dropout(p=0.5)

# Batch Normalization (Stabilizes and accelerates training)
batch_norm = nn.BatchNorm2d(num_features=16)
```

### Activation Functions
Activations introduce non-linearity, allowing the network to learn complex patterns. PyTorch provides these both as classes in `nn` and functions in `nn.functional` (F).

```python
import torch.nn.functional as F

x = torch.randn(2, 5)

# ReLU (Rectified Linear Unit): max(0, x) - Most common hidden layer activation
relu_out = F.relu(x)

# Sigmoid: 1 / (1 + exp(-x)) - Squashes output to [0, 1]. Used in binary classification.
sigmoid_out = F.sigmoid(x) # Note: nn.Sigmoid() also exists

# Tanh: Hyperbolic tangent. Squashes output to [-1, 1].
tanh_out = F.tanh(x)

# Softmax: Outputs a probability distribution. Used in multi-class classification.
# dim=1 applies it across the features/classes dimension.
softmax_out = F.softmax(x, dim=1)
```

### Loss Functions
Loss functions quantify how far off the model's predictions are from the true labels.

```python
# Mean Squared Error (MSE) - For regression tasks
criterion_mse = nn.MSELoss()

# Cross Entropy Loss - For multi-class classification
# Combines nn.LogSoftmax() and nn.NLLLoss() internally.
# VERY IMPORTANT: Do not apply Softmax to the network output if using this loss!
criterion_ce = nn.CrossEntropyLoss()

# Binary Cross Entropy (BCE) - For binary classification
# Expects inputs to be probabilities (between 0 and 1, usually from Sigmoid)
criterion_bce = nn.BCELoss()

# BCE with Logits Loss - Combines Sigmoid + BCE for better numerical stability
# Prefer this over BCELoss + manual Sigmoid
criterion_bce_logits = nn.BCEWithLogitsLoss()
```

💡 **Pro Tips:**
* Prefer the `nn.functional` API for stateless operations (like `F.relu`, `F.max_pool2d`), and `nn` classes for layers with learnable parameters (like `nn.Linear`, `nn.Conv2d`).
* Always use `nn.CrossEntropyLoss` or `nn.BCEWithLogitsLoss` over manual softmax/sigmoid + loss combos to prevent numerical instability (NaN losses) due to `log(0)`.

⚠️ **Common Pitfalls:**
* Mixing up expected input shapes for loss functions. For example, `CrossEntropyLoss` expects raw logits of shape `(Batch, Classes)` and target labels of shape `(Batch,)` containing integer class indices.

---

## 6. MODEL BUILDING

### Defining Models using nn.Module
To build a custom model, subclass `nn.Module`. You must define two things:
1. `__init__`: Define the layers (parameters) here.
2. `forward`: Define how the data flows through the layers.

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SimpleMLP(nn.Module):
    def __init__(self, input_size, hidden_size, num_classes):
        # 1. Always call the parent class constructor
        super(SimpleMLP, self).__init__()
        
        # 2. Define the layers
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.fc2 = nn.Linear(hidden_size, hidden_size)
        self.output_layer = nn.Linear(hidden_size, num_classes)
        
        # Define dropout for regularization
        self.dropout = nn.Dropout(0.2)

    def forward(self, x):
        # Define the forward pass (data flow)
        
        # Layer 1 -> Activation -> Dropout
        x = self.fc1(x)
        x = F.relu(x)
        x = self.dropout(x)
        
        # Layer 2 -> Activation -> Dropout
        x = self.fc2(x)
        x = F.relu(x)
        x = self.dropout(x)
        
        # Output layer (No activation here if using CrossEntropyLoss later)
        x = self.output_layer(x)
        
        return x

# Instantiate the model
input_dim = 784 # E.g., for 28x28 images flattened
hidden_dim = 256
classes = 10
model = SimpleMLP(input_size=input_dim, hidden_size=hidden_dim, num_classes=classes)

# Print model architecture
print(model)
```

### Sequential API
For simple, linear feed-forward networks, `nn.Sequential` is a concise alternative.

```python
sequential_model = nn.Sequential(
    nn.Linear(784, 256),
    nn.ReLU(),
    nn.Dropout(0.2),
    nn.Linear(256, 128),
    nn.ReLU(),
    nn.Linear(128, 10)
)
```

### Parameter Management
`nn.Module` automatically registers parameters (weights/biases) for any `nn` layers assigned to `self`.

```python
# Access all learnable parameters (returns an iterator of tensors)
for name, param in model.named_parameters():
    print(f"Layer: {name} | Size: {param.size()} | Requires Grad: {param.requires_grad}")

# Move the entire model and all its parameters to a specific device
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)
```

💡 **Pro Tips:**
* Keep your `__init__` clean. Do not perform operations on tensors inside `__init__`.
* You can define arbitrary Python logic (if/else, loops) inside the `forward` method. This makes PyTorch exceptional for dynamic networks (like Tree-RNNs or Beam Search).

⚠️ **Common Pitfalls:**
* **Forgetting `super().__init__()`:** If you forget this, PyTorch will not correctly track the layers and parameters, resulting in an `AttributeError`.
* **Storing tensors as attributes:** If you assign a raw tensor (e.g., `self.my_tensor = torch.randn(...)`) inside `__init__`, it won't be moved to the GPU when you call `model.to(device)`. Wrap it in `nn.Parameter` or register it as a buffer via `self.register_buffer('my_tensor', ...)` if it shouldn't be trained.

---

## 7. TRAINING LOOP

The PyTorch training loop is remarkably consistent across tasks. It consists of five critical steps executed inside a loop over the dataset.

### Standard Training Loop Structure

```python
import torch
import torch.nn as nn
import torch.optim as optim

# Mock Setup
model = nn.Linear(10, 2)
optimizer = optim.SGD(model.parameters(), lr=0.01)
criterion = nn.MSELoss()
inputs = torch.randn(32, 10)
targets = torch.randn(32, 2)

# Set model to training mode (enables Dropout, BatchNorm updates)
model.train() 

epochs = 5
for epoch in range(epochs):
    # In reality, you'd iterate over a DataLoader here
    
    # 1. Zero out the gradients from the previous iteration
    # CRITICAL: Without this, gradients will accumulate forever!
    optimizer.zero_grad()
    
    # 2. Forward pass: Pass inputs through the model to get predictions
    outputs = model(inputs)
    
    # 3. Compute the Loss between predictions and true targets
    loss = criterion(outputs, targets)
    
    # 4. Backward pass: Compute gradients of the loss with respect to model parameters
    loss.backward()
    
    # 5. Optimizer step: Update the model parameters based on the computed gradients
    optimizer.step()
    
    print(f"Epoch {epoch+1}/{epochs}, Loss: {loss.item():.4f}")
```

### Explanation of Steps
1. **`optimizer.zero_grad()`**: Clears the `.grad` attribute of all parameters. If omitted, new gradients are added to old ones.
2. **`model(inputs)`**: Calls the `forward()` method.
3. **`criterion(outputs, targets)`**: Calculates the scalar loss value.
4. **`loss.backward()`**: Autograd traverses the graph backwards, calculating `d(loss)/d(weight)` for every parameter.
5. **`optimizer.step()`**: Applies the update rule (e.g., Weight = Weight - LearningRate * Gradient).

💡 **Pro Tips:**
* Use `loss.item()` to extract the raw Python float from the loss tensor. If you keep appending `loss` (the tensor) to a list for logging, you will cause a memory leak because it keeps the entire computation graph alive in memory!
* Place `optimizer.zero_grad()` at the *start* of the loop to ensure clarity. Some modern implementations use `optimizer.zero_grad(set_to_none=True)` for slightly better memory efficiency and speed.

⚠️ **Common Pitfalls:**
* Putting `model.train()` or `model.eval()` inside the batch loop. Call it once before the loop begins.

---

## 8. OPTIMIZERS

Optimizers dictate how the network's weights are updated based on the gradients.

### Common Optimizers

```python
import torch.optim as optim

# SGD (Stochastic Gradient Descent)
# Classic, reliable, but often requires careful learning rate tuning and momentum
optimizer_sgd = optim.SGD(model.parameters(), lr=0.01, momentum=0.9, weight_decay=1e-4)

# Adam (Adaptive Moment Estimation)
# Extremely popular default choice. Adapts learning rate for each parameter.
# Faster convergence initially, but can sometimes overfit compared to well-tuned SGD.
optimizer_adam = optim.Adam(model.parameters(), lr=0.001, betas=(0.9, 0.999), weight_decay=1e-4)

# AdamW (Adam with decoupled Weight Decay)
# Fixes the weight decay implementation in standard Adam. 
# Highly recommended over Adam for Transformers and modern architectures.
optimizer_adamw = optim.AdamW(model.parameters(), lr=1e-4, weight_decay=0.01)

# RMSprop
# Good for Recurrent Neural Networks (RNNs)
optimizer_rmsprop = optim.RMSprop(model.parameters(), lr=0.01, alpha=0.99)
```

### Learning Rate Scheduling
Dynamically adjusting the learning rate during training is critical for achieving state-of-the-art performance.

```python
from torch.optim.lr_scheduler import StepLR, ReduceLROnPlateau, CosineAnnealingLR

# StepLR: Decays learning rate by a factor of gamma every step_size epochs
# If lr=0.1, gamma=0.1, step_size=30 -> lr=0.01 at epoch 30, lr=0.001 at epoch 60
scheduler_step = StepLR(optimizer_sgd, step_size=30, gamma=0.1)

# ReduceLROnPlateau: Reduces LR when a metric (e.g., validation loss) stops improving.
# 'patience' determines how many epochs to wait with no improvement.
scheduler_plateau = ReduceLROnPlateau(optimizer_adam, mode='min', factor=0.1, patience=5)

# CosineAnnealingLR: Gradually drops the LR following a cosine curve.
# T_max is the maximum number of iterations.
scheduler_cosine = CosineAnnealingLR(optimizer_adamw, T_max=100)

# Implementation inside the training loop (Epoch level):
for epoch in range(epochs):
    # ... train loop ...
    
    # Update the learning rate at the END of the epoch
    scheduler_step.step() 
    
    # For ReduceLROnPlateau, you must pass the validation metric:
    # val_loss = validate(model, val_loader)
    # scheduler_plateau.step(val_loss)
```

💡 **Pro Tips:**
* **Always start with AdamW** and a learning rate around `1e-3` or `3e-4` as a baseline.
* The `weight_decay` parameter acts as L2 Regularization, helping to prevent overfitting by penalizing large weights.

⚠️ **Common Pitfalls:**
* Calling `scheduler.step()` *before* `optimizer.step()`. This can cause PyTorch to skip the first learning rate value. Always call it after.
* Applying learning rate schedulers to the wrong loop (some update per epoch, others like `OneCycleLR` update per batch).

---

## 9. DATA HANDLING

PyTorch utilizes `Dataset` and `DataLoader` classes to decouple data loading/processing logic from the training loop.

### Dataset Class
A custom Dataset must inherit from `torch.utils.data.Dataset` and implement three methods: `__init__`, `__len__`, and `__getitem__`.

```python
import torch
from torch.utils.data import Dataset, DataLoader
import numpy as np

class CustomTabularDataset(Dataset):
    def __init__(self, data_path):
        # Initialize data, file paths, or lists here
        # Example: Loading a mock CSV
        # self.data = pd.read_csv(data_path)
        self.features = np.random.rand(100, 5) # 100 samples, 5 features
        self.labels = np.random.randint(0, 2, 100) # Binary labels

    def __len__(self):
        # Returns the total number of samples
        return len(self.labels)

    def __getitem__(self, idx):
        # Generates one sample of data given an index
        # 1. Fetch data
        x = self.features[idx]
        y = self.labels[idx]
        
        # 2. Convert to PyTorch Tensors
        x_tensor = torch.tensor(x, dtype=torch.float32)
        y_tensor = torch.tensor(y, dtype=torch.long)
        
        # 3. Return a tuple (or dict)
        return x_tensor, y_tensor

# Instantiate the dataset
my_dataset = CustomTabularDataset("dummy_path.csv")
print(f"Dataset size: {len(my_dataset)}")
sample_x, sample_y = my_dataset[0]
print(f"Sample X shape: {sample_x.shape}, Sample Y: {sample_y}")
```

### DataLoader
The `DataLoader` wraps an iterable over the Dataset to enable easy access to samples via batching, shuffling, and multiprocessing.

```python
# Create a DataLoader
# batch_size: number of samples per batch
# shuffle: randomly shuffle data at every epoch (crucial for training)
# num_workers: number of subprocesses for data loading (speeds up I/O bound tasks)
# pin_memory: faster transfer to CUDA by allocating page-locked memory
train_loader = DataLoader(
    dataset=my_dataset,
    batch_size=16,
    shuffle=True,
    num_workers=2, 
    pin_memory=True 
)

# Iterating through the DataLoader in the training loop
for batch_idx, (inputs, targets) in enumerate(train_loader):
    # Move batch to GPU
    # inputs = inputs.to('cuda')
    # targets = targets.to('cuda')
    
    # ... Forward/Backward Pass ...
    if batch_idx == 0:
        print(f"Batch Input Shape: {inputs.shape}") # Expected: [16, 5]
        print(f"Batch Target Shape: {targets.shape}") # Expected: [16]
        break
```

💡 **Pro Tips:**
* Maximize `num_workers` to prevent the GPU from waiting on the CPU for data. A rule of thumb is `num_workers = 4 * num_gpus`.
* Always use `pin_memory=True` if you are training on a GPU. It dramatically speeds up the CPU-to-GPU data transfer.

⚠️ **Common Pitfalls:**
* **Out of Memory (OOM) via workers:** If your dataset reads large images and `num_workers` is too high, system RAM can easily be exhausted.
* **Shuffling validation data:** Only shuffle the training set. Shuffling the validation or test set is unnecessary and makes debugging harder.

---

## 10. TRANSFORMS & AUGMENTATION

Transforms are applied to raw data (usually images) before they enter the model. They handle conversion to tensors, normalization, and data augmentation to prevent overfitting.

### torchvision.transforms

```python
import torchvision.transforms as transforms
from PIL import Image
import numpy as np

# Create a sequence of transformations using Compose
train_transforms = transforms.Compose([
    # Data Augmentation (introduces variability to prevent overfitting)
    transforms.RandomResizedCrop(224), # Randomly crop and resize to 224x224
    transforms.RandomHorizontalFlip(p=0.5), # 50% chance to flip horizontally
    transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2),
    
    # Formatting
    transforms.ToTensor(), # Converts PIL Image or NumPy (H x W x C) [0, 255] to Tensor (C x H x W) [0.0, 1.0]
    
    # Normalization (Crucial for stable gradients)
    # Uses ImageNet mean and std deviation by default
    transforms.Normalize(mean=[0.485, 0.456, 0.406], 
                         std=[0.229, 0.224, 0.225])
])

# Transforms for Validation/Test sets MUST NOT include random augmentation
val_transforms = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

# Mock application to a dummy image
# dummy_img = Image.fromarray(np.uint8(np.random.rand(300, 300, 3) * 255))
# transformed_img = train_transforms(dummy_img)
# print(transformed_img.shape) # Output: torch.Size([3, 224, 224])
```

💡 **Pro Tips:**
* PyTorch Vision recently introduced `v2` transforms (`torchvision.transforms.v2`), which are faster, support bounding boxes/masks natively, and are the recommended way forward.
* Normalization forces the input features to have zero mean and unit variance. This ensures gradients are of similar scale across channels, vastly speeding up convergence.

⚠️ **Common Pitfalls:**
* **Applying augmentation to Validation data:** Never apply RandomFlips or RandomCrops to validation/test sets. The evaluation metrics will become noisy and unreliable.
* **Order of operations:** `ToTensor()` must generally be called *before* `Normalize()`, as `Normalize` expects a float tensor.

---

## 11. CNNs (CONVOLUTIONAL NETWORKS)

CNNs are the backbone of Computer Vision in deep learning. PyTorch makes defining spatial operations intuitive.

### Building a CNN

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SimpleCNN(nn.Module):
    def __init__(self, num_classes=10):
        super(SimpleCNN, self).__init__()
        
        # Block 1: Conv -> ReLU -> MaxPool
        # Input shape: [Batch, 3, 32, 32] (e.g., CIFAR-10)
        self.conv1 = nn.Conv2d(in_channels=3, out_channels=16, kernel_size=3, padding=1)
        self.bn1 = nn.BatchNorm2d(16)
        # MaxPool reduces spatial dimensions by half (32x32 -> 16x16)
        self.pool1 = nn.MaxPool2d(kernel_size=2, stride=2)
        
        # Block 2: Conv -> ReLU -> MaxPool
        self.conv2 = nn.Conv2d(in_channels=16, out_channels=32, kernel_size=3, padding=1)
        self.bn2 = nn.BatchNorm2d(32)
        self.pool2 = nn.MaxPool2d(kernel_size=2, stride=2) 
        # Output shape after pool2: [Batch, 32, 8, 8]
        
        # Fully Connected Classifier
        # We must flatten the spatial dimensions. 32 channels * 8 * 8 spatial size.
        self.fc1 = nn.Linear(in_features=32 * 8 * 8, out_features=128)
        self.dropout = nn.Dropout(0.5)
        self.fc2 = nn.Linear(in_features=128, out_features=num_classes)

    def forward(self, x):
        # Feature Extraction
        x = self.pool1(F.relu(self.bn1(self.conv1(x))))
        x = self.pool2(F.relu(self.bn2(self.conv2(x))))
        
        # Flattening for the FC layers
        # x.size(0) is the batch size. -1 infers the remaining dimension (32*8*8).
        x = x.view(x.size(0), -1) 
        
        # Classification
        x = F.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.fc2(x) # Raw logits output
        return x

# Test the network with dummy data
cnn_model = SimpleCNN(num_classes=10)
dummy_image_batch = torch.randn(4, 3, 32, 32) # Batch of 4 images
logits = cnn_model(dummy_image_batch)
print(f"Output shape: {logits.shape}") # Output: [4, 10]
```

💡 **Pro Tips:**
* Use `padding='same'` in `nn.Conv2d` (available in newer PyTorch versions) if you want the output spatial dimensions to exactly match the input spatial dimensions (stride must be 1).
* Always place `BatchNorm2d` *before* the Activation function for optimal numerical stability.

⚠️ **Common Pitfalls:**
* **Miscalculating the `in_features` of the first Linear layer.** A simple way to debug this is to put a `print(x.shape)` immediately before the `.view()` call in the `forward` method during a dummy pass to see the exact dimensions.

---

## 12. RNNs & SEQUENCE MODELS

Recurrent Neural Networks handle sequential data like text, time-series, or audio.

### LSTM Implementation

```python
import torch
import torch.nn as nn

class TextClassifierLSTM(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes, num_layers=2):
        super(TextClassifierLSTM, self).__init__()
        
        # 1. Embedding Layer: Converts token indices to dense vectors
        self.embedding = nn.Embedding(num_embeddings=vocab_size, embedding_dim=embed_dim)
        
        # 2. LSTM Layer
        # batch_first=True expects input shape [Batch, Sequence_Length, Features]
        self.lstm = nn.LSTM(input_size=embed_dim, 
                            hidden_size=hidden_dim, 
                            num_layers=num_layers, 
                            batch_first=True, 
                            dropout=0.2,     # Applies dropout between LSTM layers
                            bidirectional=True) # Reads sequence left-to-right AND right-to-left
        
        # 3. Fully Connected Output
        # Multiply hidden_dim by 2 because it's bidirectional
        self.fc = nn.Linear(hidden_dim * 2, num_classes)

    def forward(self, text_indices, hidden=None):
        # Input text_indices shape: [Batch, Sequence_Length] (integers)
        
        # embedded shape: [Batch, Sequence_Length, embed_dim]
        embedded = self.embedding(text_indices)
        
        # lstm_out shape: [Batch, Sequence_Length, hidden_dim * 2]
        # h_n shape: [num_layers * 2, Batch, hidden_dim] (final hidden states)
        # c_n shape: [num_layers * 2, Batch, hidden_dim] (final cell states)
        lstm_out, (h_n, c_n) = self.lstm(embedded, hidden)
        
        # For classification, we usually take the hidden state of the LAST time step.
        # Since it's bidirectional, we concatenate the final forward state and final backward state.
        # h_n[-2,:,:] is forward, h_n[-1,:,:] is backward
        final_hidden_state = torch.cat((h_n[-2,:,:], h_n[-1,:,:]), dim=1)
        
        # output shape: [Batch, num_classes]
        output = self.fc(final_hidden_state)
        return output

# Mock test
vocab_size = 5000
model = TextClassifierLSTM(vocab_size=vocab_size, embed_dim=100, hidden_dim=128, num_classes=2)
dummy_text = torch.randint(0, vocab_size, (8, 20)) # Batch of 8 sentences, 20 words each
predictions = model(dummy_text)
print(f"Predictions shape: {predictions.shape}") # Expected: [8, 2]
```

💡 **Pro Tips:**
* For sequences of varying lengths in the same batch, use `torch.nn.utils.rnn.pack_padded_sequence` before passing to the LSTM, and `pad_packed_sequence` after. This skips processing padding tokens and vastly improves efficiency.
* Today, Transformers (via `nn.Transformer` or HuggingFace) usually outperform RNN/LSTMs for NLP tasks, but LSTMs remain excellent for smaller datasets and time-series forecasting.

⚠️ **Common Pitfalls:**
* **Forgetting `batch_first=True`:** By default, PyTorch RNNs expect the input shape to be `[Sequence_Length, Batch, Features]`. If you pass `[Batch, Seq, Feat]`, it will compute incorrectly without throwing an explicit error.

---

## 13. TRANSFER LEARNING

Transfer learning leverages models pre-trained on massive datasets (like ImageNet) and adapts them to specific, smaller datasets.

### Using Pre-trained Models

```python
import torch
import torch.nn as nn
import torchvision.models as models

# 1. Load a pre-trained ResNet-18 model
# weights=models.ResNet18_Weights.DEFAULT loads the best available ImageNet weights
resnet18 = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)

# 2. Freeze the pre-trained weights (Feature Extraction)
# This prevents backpropagation from updating the core convolutional layers, saving memory and time
for param in resnet18.parameters():
    param.requires_grad = False

# 3. Replace the final classification head
# ResNet's final layer is named 'fc'. We look at its input features.
num_ftrs = resnet18.fc.in_features

# Replace the final layer. By default, newly created layers have requires_grad=True.
# So ONLY this new layer will be trained.
num_new_classes = 5
resnet18.fc = nn.Linear(num_ftrs, num_new_classes)

# 4. Setup Optimizer
# Notice we ONLY pass the parameters of the new 'fc' layer to the optimizer
optimizer = torch.optim.Adam(resnet18.fc.parameters(), lr=0.001)

# Now, during training, only the newly added Linear layer weights will change.
# This is called "Feature Extraction".
```

### Fine-Tuning
Fine-tuning involves unfreezing some or all of the pre-trained layers and training them with a very small learning rate.

```python
# To fine-tune the whole model:
for param in resnet18.parameters():
    param.requires_grad = True # Keep everything unfrozen

# Use a tiny learning rate so you don't destroy the pre-trained weights
optimizer_finetune = torch.optim.Adam(resnet18.parameters(), lr=1e-5)
```

💡 **Pro Tips:**
* A common workflow: Start with Feature Extraction (freezing backbone) for a few epochs until the new head stabilizes. Then, unfreeze the backbone and Fine-Tune the entire network with a `10x` smaller learning rate.
* HuggingFace `transformers` and `timm` (PyTorch Image Models) are the go-to libraries for downloading state-of-the-art pre-trained models.

⚠️ **Common Pitfalls:**
* Forgetting to match the normalization statistics. If using a pre-trained model, you MUST use the exact same `mean` and `std` normalization transforms that the model was originally trained on.

---

## 14. SAVING & LOADING MODELS

Persisting your models properly is critical for pausing/resuming training and deploying to production.

### The state_dict
The most standard and robust way to save a model is by saving its `state_dict`, which is a Python dictionary mapping each layer to its parameter tensor.

```python
import torch
import torch.nn as nn
import torch.optim as optim

model = nn.Linear(10, 2)
optimizer = optim.SGD(model.parameters(), lr=0.01)

# --- SAVING A MODEL FOR INFERENCE ---
# Save only the weights
torch.save(model.state_dict(), 'model_weights.pth')

# --- LOADING A MODEL FOR INFERENCE ---
# 1. Instantiate the EXACT same architecture first
loaded_model = nn.Linear(10, 2) 
# 2. Load the state dictionary into the model
loaded_model.load_state_dict(torch.load('model_weights.pth'))
# 3. CRITICAL: Set to eval mode before making predictions
loaded_model.eval() 


# --- SAVING A CHECKPOINT (To resume training later) ---
# You must save the optimizer state, current epoch, and loss as well.
checkpoint = {
    'epoch': 5,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': 0.45
}
torch.save(checkpoint, 'training_checkpoint.pth')


# --- LOADING A CHECKPOINT (To resume training) ---
checkpoint_loaded = torch.load('training_checkpoint.pth')
model.load_state_dict(checkpoint_loaded['model_state_dict'])
optimizer.load_state_dict(checkpoint_loaded['optimizer_state_dict'])
epoch = checkpoint_loaded['epoch']
loss = checkpoint_loaded['loss']

model.train() # Set back to train mode
```

💡 **Pro Tips:**
* Use the `.pth` or `.pt` file extension conventionally.
* Never use `torch.save(model, 'model.pth')` (saving the entire object). It binds the saved file to specific directory structures and exact class definitions, causing catastrophic breaks if you refactor your code. Always save the `state_dict`.

⚠️ **Common Pitfalls:**
* **Device mismatch when loading:** If you saved a model on GPU, loading it on a machine with only a CPU will crash. Fix this by passing `map_location`: 
  `torch.load('model.pth', map_location=torch.device('cpu'))`

---

## 15. GPU TRAINING

Moving data to the GPU is required to harness parallel processing capabilities.

### Device Management

```python
import torch

# 1. Define the device agnosticly
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print(f"Targeting device: {device}")

# 2. Move the Model
# model.to() modifies the module IN-PLACE.
model = nn.Linear(10, 2)
model.to(device) 

# 3. Move the Data (Tensors)
# tensor.to() creates a COPY on the target device. You must reassign it.
inputs = torch.randn(32, 10)
targets = torch.randn(32, 2)

# During the training loop:
inputs = inputs.to(device)
targets = targets.to(device)

# 4. Moving tensors back to CPU (e.g., for converting to NumPy or saving)
output = model(inputs)
# Cannot convert a CUDA tensor directly to NumPy. Must move to CPU first.
output_numpy = output.cpu().detach().numpy() 
```

### Multi-GPU Training (Basics)
For training on a single machine with multiple GPUs, `DataParallel` is the simplest (though not the most efficient) method.

```python
# If multiple GPUs are detected, wrap the model in DataParallel
if torch.cuda.device_count() > 1:
    print(f"Let's use {torch.cuda.device_count()} GPUs!")
    # DataParallel splits the batch across multiple GPUs.
    # E.g., Batch of 32 on 4 GPUs -> 8 samples per GPU.
    model = nn.DataParallel(model)

model.to(device)
```

💡 **Pro Tips:**
* Using `non_blocking=True` when moving tensors (`inputs.to(device, non_blocking=True)`) allows the data transfer to happen asynchronously, overlapping with computation and speeding up training (requires `pin_memory=True` in DataLoader).
* `DataParallel` is technically obsolete. Use `DistributedDataParallel` (DDP) for serious production workloads, even on a single multi-GPU machine, as it avoids Python GIL bottlenecks.

⚠️ **Common Pitfalls:**
* `RuntimeError: Expected all tensors to be on the same device`. This happens constantly. Ensure the model, inputs, targets, and any hidden states (like in LSTMs) are mapped to `device`.

---

## 16. EVALUATION & INFERENCE

Evaluation requires turning off training-specific behaviors and gradient calculations.

### Validation Loop Structure

```python
import torch

# 1. CRITICAL: Set model to evaluation mode. 
# This disables Dropout layers and uses population statistics for BatchNorm instead of batch statistics.
model.eval()

total_loss = 0.0
correct_predictions = 0
total_samples = 0

# 2. CRITICAL: Turn off gradient tracking. 
# This drastically reduces memory consumption and speeds up inference.
with torch.no_grad():
    for inputs, targets in val_loader:
        inputs, targets = inputs.to(device), targets.to(device)
        
        # Forward pass
        outputs = model(inputs)
        
        # Compute loss
        loss = criterion(outputs, targets)
        total_loss += loss.item()
        
        # Calculate accuracy (Classification context)
        # outputs shape: [Batch, Classes]. torch.max returns (values, indices).
        _, predicted_classes = torch.max(outputs, dim=1) 
        
        correct_predictions += (predicted_classes == targets).sum().item()
        total_samples += targets.size(0)

# Calculate metrics
avg_val_loss = total_loss / len(val_loader)
val_accuracy = correct_predictions / total_samples

print(f"Validation Loss: {avg_val_loss:.4f} | Validation Accuracy: {val_accuracy:.4f}")

# 3. Optional: Set model back to train mode if continuing training
# model.train()
```

💡 **Pro Tips:**
* PyTorch recently introduced `torch.inference_mode()`, which acts as an even faster, stricter version of `torch.no_grad()`. Use it whenever you are strictly doing inference and have no intention of using the outputs in any gradient calculations later.
* Consider using `torchmetrics` (from the PyTorch Lightning ecosystem) for computing complex metrics (F1, AUROC, mAP) rather than writing manual accumulators.

⚠️ **Common Pitfalls:**
* Forgetting `model.eval()`. Doing inference with Dropout enabled will result in highly inconsistent and degraded predictions.

---

## 17. DEBUGGING & PROFILING

PyTorch's dynamic nature makes standard Python debugging easy, but specific tools exist for tensor math and performance.

### Debugging Gradients

```python
import torch
import torch.nn as nn

# Problem: Exploding or Vanishing Gradients
# Solution: Hooks allow you to inspect gradients flowing backwards without modifying the model code.

model = nn.Linear(10, 10)

def print_grad_hook(module, grad_input, grad_output):
    print(f"Layer: {module}")
    print(f"Gradient flowing IN  mean: {grad_input[0].mean().item()}")
    print(f"Gradient flowing OUT mean: {grad_output[0].mean().item()}")
    print("-" * 30)

# Register the hook to a specific layer
hook_handle = model.register_full_backward_hook(print_grad_hook)

# Run a dummy pass to trigger the hook
x = torch.randn(1, 10, requires_grad=True)
y = model(x)
loss = y.sum()
loss.backward()

# Remove the hook when done
hook_handle.remove()
```

### Profiling Performance
Identify bottlenecks in execution (CPU vs GPU time).

```python
import torch.autograd.profiler as profiler

# Run the code inside the profiler context
with profiler.profile(use_cuda=True, record_shapes=True) as prof:
    with profiler.record_function("model_inference"):
        # Put your model forward pass here
        _ = model(x)

# Print a formatted table of execution times
print(prof.key_averages().table(sort_by="cuda_time_total", row_limit=10))

# Can also export to Chrome Trace viewer (chrome://tracing)
# prof.export_chrome_trace("trace.json")
```

💡 **Pro Tips:**
* If you encounter NaN losses, use `torch.autograd.set_detect_anomaly(True)` at the top of your script. It will severely slow down training but will print the exact forward operation that caused the backward pass to produce a NaN gradient.
* Use standard Python `pdb.set_trace()` or IDE breakpoints inside the `forward` method to inspect tensor shapes mid-execution.

⚠️ **Common Pitfalls:**
* Leaving `set_detect_anomaly(True)` on during production training. It incurs massive overhead.

---

## 18. DEPLOYMENT

Transitioning from research code to a high-performance production environment.

### TorchScript
TorchScript is a way to serialize PyTorch models so they can run in C++ environments without Python dependency.

```python
import torch
import torchvision.models as models

# 1. Get a pre-trained model
model = models.resnet18(pretrained=True)
model.eval()

# 2. Provide example input (required for tracing)
example_input = torch.rand(1, 3, 224, 224)

# 3. Tracing: Runs the input through the model and records the operations into a static graph.
# Note: Tracing ignores control flow (if/else statements) that depend on data.
traced_script_module = torch.jit.trace(model, example_input)

# 4. Save the scripted model
traced_script_module.save("traced_resnet18.pt")

# The saved .pt file can now be loaded in C++ via LibTorch:
# torch::jit::script::Module module = torch::jit::load("traced_resnet18.pt");
```

### ONNX Export
ONNX (Open Neural Network Exchange) allows you to export PyTorch models to a standardized format, which can then be optimized and run by TensorRT, ONNX Runtime, or imported into TensorFlow.

```python
# Export the model to ONNX format
torch.onnx.export(model,               # model being run
                  example_input,       # model input (or a tuple for multiple inputs)
                  "resnet18.onnx",     # where to save the model
                  export_params=True,  # store the trained parameter weights inside the model file
                  opset_version=11,    # the ONNX version to export the model to
                  do_constant_folding=True,  # optimize constant operations
                  input_names = ['input'],   # the model's input names
                  output_names = ['output'], # the model's output names
                  dynamic_axes={'input' : {0 : 'batch_size'},    # variable length axes
                                'output' : {0 : 'batch_size'}})
```

💡 **Pro Tips:**
* If your model has complex `if/else` control flow, `torch.jit.trace` will fail to capture it. You must use `torch.jit.script` instead, which analyzes the Python AST directly.
* Use **TorchServe** for robust HTTP serving. It handles batching, versioning, and worker scaling out of the box.

⚠️ **Common Pitfalls:**
* Forgetting to call `model.eval()` before tracing/exporting. Dropout/BatchNorm layers acting dynamically during inference is catastrophic.

---

## 19. PERFORMANCE OPTIMIZATION

Techniques to squeeze maximum performance out of GPU hardware.

### Automatic Mixed Precision (AMP)
Modern GPUs (NVIDIA Volta and newer, utilizing Tensor Cores) execute `float16` math 2-4x faster than standard `float32`. AMP automatically casts operations to the optimal precision, drastically speeding up training and halving memory usage without losing accuracy.

```python
import torch
from torch.cuda.amp import autocast, GradScaler

model = nn.Linear(10, 2).cuda()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
scaler = GradScaler() # Prevents fp16 gradients from underflowing to zero

for epoch in range(5):
    for inputs, targets in dataloader: # Assuming DataLoader yields CUDA tensors
        optimizer.zero_grad()
        
        # 1. Cast forward pass to float16
        with autocast():
            outputs = model(inputs)
            loss = criterion(outputs, targets)
        
        # 2. Scale the loss and call backward on scaled loss
        scaler.scale(loss).backward()
        
        # 3. Unscale gradients and update weights
        scaler.step(optimizer)
        
        # 4. Update the scale for next iteration
        scaler.update()
```

### Gradient Clipping
Prevents gradients from exploding, which is especially important for RNNs and Transformers.

```python
# After loss.backward(), before optimizer.step():

# Clips gradients such that their L2 norm does not exceed max_norm=1.0
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)

optimizer.step()
```

### torch.compile (PyTorch 2.0+)
The ultimate optimization. Compiles the model into optimized Triton/C++ kernels.

```python
import torch

# Standard model definition
model = MyComplexModel().cuda()

# Compile the model! 
# This requires zero code changes and often yields 20-40% speedups.
compiled_model = torch.compile(model)

# Train `compiled_model` exactly as you would train `model`
```

💡 **Pro Tips:**
* Set `torch.backends.cudnn.benchmark = True` at the top of your script if your input sizes (batch size, image resolution) are constant. This lets cuDNN benchmark different algorithms internally and select the fastest one for your hardware.
* Use `model.half()` for pure FP16 inference to instantly double inference speed and halve VRAM usage.

⚠️ **Common Pitfalls:**
* Mixing `autocast` heavily with older PyTorch versions can cause silent accuracy drops. Always ensure `GradScaler` is correctly implemented alongside `autocast`.

---

## 20. ADVANCED TOPICS

### Distributed Data Parallel (DDP)
DDP is the production standard for multi-GPU training. It spawns a separate process for each GPU, bypassing Python's Global Interpreter Lock (GIL).

```python
# Pseudo-code for DDP Setup (Usually launched via `torchrun`)
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP

def setup(rank, world_size):
    # Initialize the process group
    dist.init_process_group("nccl", rank=rank, world_size=world_size)

def cleanup():
    dist.destroy_process_group()

# Inside the training script:
# local_rank = int(os.environ["LOCAL_RANK"])
# torch.cuda.set_device(local_rank)
# model = MyModel().to(local_rank)
# ddp_model = DDP(model, device_ids=[local_rank])
```

### Custom Autograd Functions
You can define exact mathematical derivatives for custom, non-differentiable operations.

```python
class CustomReLU(torch.autograd.Function):
    @staticmethod
    def forward(ctx, input):
        # Save context for backward pass
        ctx.save_for_backward(input)
        return input.clamp(min=0)

    @staticmethod
    def backward(ctx, grad_output):
        # Retrieve saved tensors
        input, = ctx.saved_tensors
        grad_input = grad_output.clone()
        # Gradient is 0 where input is <= 0
        grad_input[input <= 0] = 0
        return grad_input

# Usage:
# my_relu = CustomReLU.apply
# output = my_relu(input_tensor)
```

### PyTorch Lightning Overview
Writing standard PyTorch training loops results in massive boilerplate. PyTorch Lightning organizes your code.

```python
# pip install pytorch-lightning
import pytorch_lightning as pl

class LitModel(pl.LightningModule):
    def __init__(self):
        super().__init__()
        self.layer = nn.Linear(32, 2)
        
    def forward(self, x):
        return self.layer(x)
        
    def training_step(self, batch, batch_idx):
        x, y = batch
        y_hat = self(x)
        loss = F.cross_entropy(y_hat, y)
        self.log('train_loss', loss)
        return loss
        
    def configure_optimizers(self):
        return torch.optim.Adam(self.parameters(), lr=1e-3)

# The Trainer handles the loop, GPU moving, DDP, AMP, and checkpointing automatically!
# trainer = pl.Trainer(max_epochs=10, accelerator='gpu', devices=4)
# trainer.fit(model=LitModel(), train_dataloaders=my_dataloader)
```

💡 **Pro Tips:**
* PyTorch Lightning is strongly recommended for production training pipelines. It forces clean structure and gives you multi-GPU (DDP) and Mixed Precision support simply by changing flags in the `Trainer`.
* When dealing with massive datasets, explore `torchdata` or WebDataset formats to avoid massive IO bottlenecks.

---
*End of Document. This reference encompasses the modern best practices for PyTorch 2.x production development.*
