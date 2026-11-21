a small hardware-design experiment plus a prediction experiment.

A systolic array is a grid of simple multiply-and-add units (“processing elements,” or PEs). Numbers move between neighboring PEs in a regular rhythm, so many parts of a matrix multiplication happen at once. Your project asks: Which grid size and data-movement strategy gives the best speed and energy for a chosen workload, without using too much hardware?

What you would build and measure
For each design configuration, choose:

Array dimensions: for example, 4×4, 8×8, or 16×16 PEs.
Precision: for example, 8-bit or 16-bit numbers. Lower precision often uses less hardware and energy, but may reduce numerical accuracy.
Dataflow: the rule for where values and partial sums move or stay. Common choices are output-stationary (keep each output sum in place) and weight-stationary (keep weights near the PEs that use them).
Then run the same small workloads—initially, matrix multiplication is easiest—and collect measurements:

Cycles or latency: how long the computation takes.
Area: how much FPGA logic or silicon-cell area it uses.
Power or energy: ideally measured or estimated with a consistent tool and assumptions.
A bigger array is not automatically better: it can do more work in parallel, but may use more area and power, and some of its PEs may sit idle on small workloads.

Where deep learning fits
The “deep learning” part is a model that learns from your experiment results. Each row of your dataset might describe one hardware configuration and workload:

array size | precision | dataflow | workload size | cycles | area | energy

You train a model to predict cycles and energy for configurations you have not tested yet. Then you can search for designs that balance speed, energy, and area. This is not the accelerator learning to recognize images; it is a predictor that helps you choose accelerator settings.

You’ll need enough varied measurements for the model to learn anything meaningful. For a beginner, start with a small sweep—perhaps a few array sizes, two precisions, two dataflows, and several matrix sizes—and treat your first model as an experiment, not proof that it will generalize.

NVDLA is better treated as a reference, not a beginner starting point. Its repository includes RTL, a C model, testbench and sample traces, plus synthesis scripts, but its own documentation describes a full accelerator design and a substantial build flow. Build your small array first; later, use NVDLA to see how a larger system is organized. 
GitHub

# Systolic-Array Accelerator Exploration

A beginner-friendly project to build a small systolic-array accelerator, measure how design choices affect performance and hardware cost, and train a model to predict those results.

## Project goal

This project explores how array dimensions, numeric precision, dataflow, and workload size affect matrix-multiplication performance and energy.

The project has two main parts:

1. Build a software model and a small RTL systolic array.
2. Use measured results to train a machine-learning model that predicts latency, area, and energy for different configurations.

The predictor helps compare designs; it does not control the accelerator or perform the matrix multiplication itself.

## Initial scope

Start with matrix multiplication:

\[
C = A \times B
\]

Use this initial design sweep:

| Parameter | Initial values |
|---|---|
| Array dimensions | 2×2, 4×4, 8×8 |
| Input precision | 8-bit, 16-bit |
| Dataflow | Output-stationary, weight-stationary |
| Square matrix sizes | 4×4, 8×8, 16×16, 32×32 |

This produces 48 basic configurations before any additional tests. If time or hardware resources are limited, begin with a 4×4 array, one precision, one dataflow, and a few matrix sizes.

## Key concepts

- **Processing element (PE):** A small hardware unit that multiplies values and accumulates their products.
- **Systolic array:** A grid of PEs that passes data between neighboring units over successive clock cycles.
- **Dataflow:** The strategy that determines where inputs, weights, and partial sums are held or moved.
- **Output-stationary:** Each PE keeps an output’s partial sum locally while input values move through the array.
- **Weight-stationary:** Weights are kept near the PEs that use them while other values move through the array.
- **Latency:** The number of cycles or elapsed time needed to complete a workload.
- **Area:** The amount of FPGA resources or synthesized hardware area used.
- **Energy:** Power multiplied by execution time.

## Project process

### 1. Build a software reference

Write a Python matrix-multiplication program that calculates expected outputs. Use it to verify the simulator and RTL.

Decide and document:

- Whether inputs are signed or unsigned
- Input precision
- Accumulator width
- Overflow behavior
- Rounding or saturation behavior, if applicable

Test with small matrices whose outputs can be checked by hand, then add random test cases.

### 2. Create a cycle-level simulator

Build a simple simulator that models a PE and the movement of values through the array.

The simulator should:

- Model multiply-and-accumulate operations
- Track data received by each PE on each cycle
- Report when output values are ready
- Produce correct matrix results
- Count cycles consistently
- Support the selected array sizes and dataflows

Start with output-stationary. Add weight-stationary only after the first dataflow works and its results match the software reference.

### 3. Run the design sweep

Write a script that enumerates the configurations and workloads. Use the same input matrices and measurement definitions for each configuration.

For every run, record:

- Array rows and columns
- Number of PEs
- Precision
- Dataflow
- Matrix dimensions
- Cycle count
- Latency, if available
- Hardware resource or area estimates
- Power and energy estimates, if available
- Measurement source and tool settings

### 4. Implement a small RTL design

Begin with one PE, then expand gradually:

1. Implement and test one PE.
2. Connect a small 2×2 array.
3. Expand to a 4×4 array.
4. Add a basic controller with start and done signals.
5. Compare RTL outputs with the Python reference.

Test zero inputs, small hand-checkable values, random values, signed values if supported, and values near the numeric limits. Test reset and repeated computations as well.

### 5. Measure hardware costs

Simulate the RTL, then synthesize it for a specific FPGA device or target.

Record the exact:

- FPGA device or target
- Tool name and version
- Clock target
- Synthesis settings
- Resource counts
- Timing results
- Power-estimation method and assumptions

Keep cycle counts, synthesis estimates, and power estimates clearly distinguished. A simulation cycle count is not itself a power measurement. If power is estimated rather than measured on hardware, label it as an estimate.

### 6. Create the dataset

Store one row per configuration and workload. For example:

```text
array\_rows,array\_cols,precision\_bits,dataflow,m,k,n,cycles,latency\_ns,luts,flip\_flops,dsps,brams,power\_w,energy\_j,measurement\_source
Use consistent units and document how every metric is calculated. Save logs and configuration details so results can be reproduced.
```

7. Train a predictor
Start with simple models rather than a complex neural network:

Average-value baseline
Linear regression
Decision tree or random forest
Predict cycles, area, and energy. Use array dimensions, precision, dataflow, and workload dimensions as input features.

Hold out some configurations from training and evaluate the model on those configurations. Report metrics such as mean absolute error and (R^2), and compare the model with the average-value baseline.

Avoid using input features that reveal the measured answer. For example, do not use a feature derived from the measured cycle count to predict cycles.

8. Compare design trade-offs
Compare configurations using plots or tables for:

Latency versus energy
Latency versus area
Energy versus area
A design may be faster but larger or more energy-intensive. State the project’s priority—for example, minimizing energy while meeting a latency target—before identifying a preferred configuration.

Metrics and definitions
Metric	Definition
Cycles	Clock cycles from accepting a workload to producing the final output
Latency	Elapsed time to complete a workload
Area	FPGA resource counts or a consistent synthesized cell-area estimate
Power	Tool-based estimate or measured power, with method stated
Energy	Power multiplied by execution time
When power is in watts and latency is in seconds:

[
\text{Energy (joules)} = \text{Power (watts)} \times \text{Latency (seconds)}
]

Do not compare area or power figures from different devices or tool configurations as if they were directly equivalent.

Suggested repository structure
text


systolic-array-exploration/
├── README.md
├── data/
│   ├── raw/
│   └── processed/
├── docs/
│   ├── architecture.md
│   └── measurement-method.md
├── rtl/
│   ├── pe.sv
│   ├── systolic\_array.sv
│   └── controller.sv
├── scripts/
│   ├── run\_sweep.py
│   ├── train\_model.py
│   └── analyze\_results.py
├── sim/
│   ├── reference\_matmul.py
│   ├── cycle\_simulator.py
│   └── tests/
├── results/
│   ├── plots/
│   └── reports/
└── requirements.txt
Adjust the structure to match the tools and languages actually used. Do not commit generated files or tool outputs that are too large for the repository; document how to recreate them.

Recommended milestones
Milestone	Completion criteria
Software reference	Produces correct matrix-multiplication results
Cycle simulator	Matches the reference and reports cycle counts
First RTL PE	Passes basic arithmetic tests
Small RTL array	Matches the reference across test cases
Design sweep	Generates a documented dataset
Hardware estimates	Includes reproducible synthesis settings
Predictor	Evaluated on held-out configurations
Final analysis	Explains trade-offs and project limitations
Reproducibility checklist
Before sharing results, confirm that:

The numeric format and overflow behavior are documented.
Test inputs and expected outputs are available or reproducible.
Each dataset row identifies its configuration and workload.
Units are stated for every metric.
Estimated and measured values are labeled separately.
Tool versions, target device, and key settings are recorded.
The prediction model is evaluated on data not used for training.
Plots and conclusions can be regenerated from the saved data.
Limitations
Results depend on the architecture, workload shapes, numeric representation, target FPGA, synthesis tools, clock settings, and power-estimation assumptions. A small matrix workload may not represent the behavior of a full neural network. Predictions are useful only to the extent that the measured configurations and workloads represent the designs being considered.

Final deliverables
Python software reference
Cycle-level simulator
Tested RTL for a small systolic array
Dataset with units and measurement sources
Synthesis or hardware-measurement report
Predictor with held-out evaluation results
Plots comparing latency, area, and energy
Reproduction instructions and a summary of limitations
