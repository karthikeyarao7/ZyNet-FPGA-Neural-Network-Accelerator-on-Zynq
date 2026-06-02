# ZyNet — FPGA Neural Network Accelerator on Zynq

A step-by-step tutorial series for building and deploying a neural network accelerator in hardware (Verilog/VHDL) on the **Xilinx Zynq SoC**, with Python tooling to auto-generate the RTL from a Keras-style API.

---

## 📁 Project Structure

```
ZyNet/
├── Tut-1/          # Single neuron in Verilog (sigmoid activation)
├── Tut-2/          # Single neuron with ReLU & sigmoid + Python sigmoid LUT generator
├── Tut-3/          # Full multi-layer network in Verilog (zynet.v)
├── Tut-4/          # AXI-Lite wrapper for PS–PL communication
├── Tut-5/          # Python code-generation framework (zynet Python package)
│   ├── src/fpga/   # Auto-generated RTL sources
│   │   └── rtl/    # Verilog RTL files
│   ├── zynet/      # Python package: gen_nn, genWeightsAndBias, xilinxUtils
│   ├── mnistZyNet.py   # Example: train + deploy MNIST classifier
│   └── zynet.tcl       # Vivado TCL build script
└── Tut-6/          # MNIST test data generation scripts
    ├── genTestData.py
    ├── genWegitsAndBias.py
    ├── network2.py
    └── mnist_loader.py
```

---

## 🚀 What This Project Does

ZyNet auto-generates synthesizable Verilog for a fully-connected neural network from a Python description, much like Keras. The generated RTL:

- Supports **configurable layer sizes, data widths, and activations** (ReLU, Sigmoid)
- Uses **block RAMs** for weight and bias storage (`.mif` files)
- Interfaces with the **ARM processor** (PS) on Zynq via AXI-Lite
- Can classify **MNIST handwritten digits** in hardware at high throughput

---

## 🛠️ Requirements

### Hardware
- Xilinx Zynq-7000 development board (e.g., Pynq-Z1, ZedBoard)

### Software
- Vivado 2025.x (or compatible)
- Python 3.7+
- Python packages: `numpy`, `tensorflow` / `keras` (for training)

---

## ⚡ Quick Start

### 1. Install the Python package
```bash
cd Tut-5
pip install numpy tensorflow
```

### 2. Define and compile your model
```python
import zynet as zn

model = zn.model()
model.add(zn.layer(type="flatten", numNeurons=784))
model.add(zn.layer(type="Dense",  numNeurons=30,  activation="relu"))
model.add(zn.layer(type="Dense",  numNeurons=10,  activation="sigmoid"))

model.compile(
    pretrained='Yes',
    weights="path/to/weights.txt",
    biases="path/to/biases.txt",
    dataWidth=16,
    sigmoidSize=5,
    weightIntSize=1,
    inputIntSize=4
)
```
This auto-generates all Verilog source files under `src/fpga/rtl/`.

### 3. Build the Vivado project
```bash
vivado -mode batch -source zynet.tcl
```

### 4. Run inference on MNIST
```python
# See mnistZyNet.py for a full end-to-end example
python mnistZyNet.py
```

---

## 📖 Tutorial Walkthrough

| Tutorial | Topic |
|----------|-------|
| **Tut-1** | Single neuron with fixed-point sigmoid activation in Verilog |
| **Tut-2** | Adding ReLU; Python script to generate sigmoid LUT (`.mif`) |
| **Tut-3** | Stacking layers into a complete network (`zynet.v`) |
| **Tut-4** | AXI-Lite wrapper so the PS can stream data to/from the network |
| **Tut-5** | Python `zynet` package: auto-generate RTL + Vivado project from code |
| **Tut-6** | Generate MNIST test vectors for hardware simulation/verification |

---

## 🔑 Key RTL Modules

| File | Description |
|------|-------------|
| `neuron.v` | Parameterised MAC + activation function unit |
| `Layer_N.v` | Auto-generated layer with N neurons |
| `Weight_Memory.v` | Block RAM weight storage (initialised from `.mif`) |
| `Sig_ROM.v` | Sigmoid lookup table |
| `relu.v` | ReLU activation |
| `maxFinder.v` | Argmax for classification output |
| `axi_lite_wrapper.v` | AXI-Lite PS–PL interface |
| `zynet.v` | Top-level network wrapper |

---

## 📊 Network Architecture (default MNIST example)

```
Input: 784 pixels (28×28 flattened)
  └─► Layer 1: 30 neurons, ReLU
  └─► Layer 2: 30 neurons, ReLU
  └─► Layer 3: 10 neurons, ReLU
  └─► Layer 4: 10 neurons, Sigmoid
  └─► MaxFinder → predicted digit (0–9)
```

Fixed-point precision: 16-bit data width, 1-bit integer part for weights.

---

## 📝 License

This project is provided for educational purposes. See individual tutorial directories for any additional notes.

---

## 🙏 Acknowledgements

Neural network weights trained using the [network2.py](Tut-6/network2.py) framework (based on Michael Nielsen's [Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/)).
