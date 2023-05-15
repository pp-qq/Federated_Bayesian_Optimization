# Code for the Paper:
Zhongxiang Dai, Kian Hsiang Low and Patrick Jaillet. "[Federated Bayesian Optimization via Thompson Sampling.](https://daizhongxiang.github.io/papers/fbo.pdf)" In 34th Conference on Neural Information Processing Systems (NeurIPS-20), Dec 6-12, 2020.

The code here is for the Landmine Detection real-world experiment (see Section 5.2 of the main paper).

## Description of the landmine detection dataset
The dataset "LandmineData.mat" can also be downloaded from http://www.ee.duke.edu/~lcarin/LandmineData.zip
LandmineData.mat contains two 1-by-29 cell arrays "feature" and "label". Each cell of the cell arrays corresponds to one task. Each cell of "feature" is an N_m-by-9 data matrix, where N_m is the number of samples for task m, m=1,...,29, i.e., each sample is described by a 9-dimensional feature vector, stored in the corresponding row of the data matrix. Each cell of "label" is an N_m-dimensional binary vector.


## Requirements
The required packages include:
1. standard packages such as scipy, numpy, sklearn, etc.
2. GPy: `pip install GPy`
3. (optional) scipydirect: this package is needed only if you want to use the DIRECT method to optimize the acquisition function (if you do, please set "USE_DIRECT_OPTIMIZER = True" in the script "federated_helper_funcs.py"). Otherwise, the acquisition function is optimized using L-BFGS with multiple random restarts. To install it to run our code:
	1. install scipydirect: `pip install scipydirect`
	2. replace the content of the script "PYTHON_PATH/lib/python3.5/site-packages/scipydirect/__init__.py" with the content of the script "scipydirect_customized.py" in the "requirements" directory; this step is needed since we modified the interface of the scipydirect minimize function to make it usable by our acquisition function.


## Instructions for Pre-processing and Algorithm Evaluation
### Pre-process landmine detection data
`python preprocess_landmine.py`

This script converts the landmine detection dataset into a format that can be readily used to run BO algorithms.
Specifically, for each landmine, we use 50% of the data as the training set, and the remaining 50% as the validation set which we use to evaluate and report the performance.

### Run standard BO (Thompson sampling)
`python run_landmine_standard_bo.py`

Firstly, this script runs a standard BO task (using Thompson sampling) for each agent; the resulted BO observations (input-output pairs) will be used by each agent to generate the message to be passed to the target agent.
Secondly, for each of the first 6 agents, we run 5 independent standard BO (TS) tasks, which are used for comparison with other algorithms.

### Generate agent information
`python generate_agent_info.py`

For each agent, the agent firstly fits a GP surrogate using its BO observations (generated by running the script "run_landmine_standard_bo.py"), then applies RFF approximation, and then saves the RFF parameters to be passed as the message to the target agent. For FTS, the saved information is a sample from the posteiror distribution of ![formula](https://render.githubusercontent.com/render/math?math=\omega) (Equation 2 in the main paper); for RGPE and TAF, the saved parameters include both ![formula](https://render.githubusercontent.com/render/math?math=\nu_t) and ![formula](https://render.githubusercontent.com/render/math?math=\Sigma_t) (inverted) from each agent; in addition, for TAF, the incumbent value (i.e., the currently observed maximum output value) is additionally saved.

### Run federated BO algorithms
`python run_landmine_federated_bo.py`

This script runs our FTS algorithm. In addition, it can also run the RGPE and TAF algorithms adapted for the FBO setting.


## Results
analyze_landmine.ipynb

This IPython notebook plots the results for the landmine detection experiment, i.e., Figures 2 & 3.

