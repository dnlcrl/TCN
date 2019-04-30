# Sequence Modeling Benchmarks and Temporal Convolutional Networks (TCN)


This repository contains the experiments done in the work [An Empirical Evaluation of Generic Convolutional and Recurrent Networks for Sequence Modeling](https://arxiv.org/abs/1803.01271) by Shaojie Bai, J. Zico Kolter and Vladlen Koltun.

We specifically target a comprehensive set of tasks that have been repeatedly used to compare the effectiveness of different recurrent networks, and evaluate a simple, generic but powerful (purely) convolutional network on the recurrent nets' home turf.

Experiments are done in PyTorch. If you find this repository helpful, please cite our work:

```
@article{BaiTCN2018,
	author    = {Shaojie Bai and J. Zico Kolter and Vladlen Koltun},
	title     = {An Empirical Evaluation of Generic Convolutional and Recurrent Networks for Sequence Modeling},
	journal   = {arXiv:1803.01271},
	year      = {2018},
}
```

## Domains and Datasets

**Update**: The code should be directly runnable with PyTorch 0.4.0 or above (PyTorch 1.0 strongly recommended). The older versions of PyTorch are no longer supported.

This repository contains the benchmarks to the following tasks, with details explained in each sub-directory:

  - **The Adding Problem** with various T (we evaluated on T=200, 400, 600)
  - **Copying Memory Task** with various T (we evaluated on T=500, 1000, 2000)
  - **Sequential MNIST** digit classification
  - **Permuted Sequential MNIST** (based on Seq. MNIST, but more challenging)
  - **JSB Chorales** polyphonic music
  - **Nottingham** polyphonic music
  - **PennTreebank** [SMALL] word-level language modeling (LM)
  - **Wikitext-103** [LARGE] word-level LM
  - **LAMBADA** [LARGE] word-level LM and textual understanding
  - **PennTreebank** [MEDIUM] char-level LM
  - **text8** [LARGE] char-level LM

While some of the large datasets are not included in this repo, we use the [observations](https://github.com/edwardlib/observations) package to download them, which can be easily installed using pip. 

## Usage

Each task is contained in its own directory, with the following structure:

```
[TASK_NAME] /
    data/
    [TASK_NAME]_test.py
    models.py
    utils.py
```

To run TCN model on the task, one only need to run `[TASK_NAME]_test.py` (e.g. `add_test.py`). To tune the hyperparameters, one can specify via argument options, which can been seen via the `-h` flag. 


## Info from https://github.com/philipperemy/keras-tcn/blob/master/README.md


## Why Temporal Convolutional Network?

- TCNs exhibit longer memory than recurrent architectures with the same capacity.
- Constantly performs better than LSTM/GRU architectures on a vast range of tasks (Seq. MNIST, Adding Problem, Copy Memory, Word-level PTB...).
- Parallelism, flexible receptive field size, stable gradients, low memory requirements for training, variable length inputs...

<p align="center">
  <img src="misc/Dilated_Conv.png">
  <b>Visualization of a stack of dilated causal convolutional layers (Wavenet, 2016)</b><br><br>
</p>


### Supported task types

- Regression (Many to one) e.g. adding problem
- Classification (Many to many) e.g. copy memory task
- Classification (Many to one) e.g. sequential mnist task

For a Many to Many regression, a cheap fix for now is to change the [number of units of the final Dense layer](https://github.com/philipperemy/keras-tcn/blob/8151b4a87f906fd856fd1c113c48392d542d0994/tcn/tcn.py#L90).

### Receptive field

- Receptive field = **nb_stacks_of_residuals_blocks * kernel_size * last_dilation**.
- If a TCN has only one stack of residual blocks with a kernel size of 2 and dilations [1, 2, 4, 8], its receptive field is 2 * 1 * 8 = 16. The image below illustrates it:

<p align="center">
  <img src="https://user-images.githubusercontent.com/40159126/41830054-10e56fda-7871-11e8-8591-4fa46680c17f.png">
  <b>ks = 2, dilations = [1, 2, 4, 8], 1 block</b><br><br>
</p>

- If the TCN has now 2 stacks of residual blocks, wou would get the situation below, that is, an increase in the receptive field to 32:

<p align="center">
  <img src="https://user-images.githubusercontent.com/40159126/41830618-a8f82a8a-7874-11e8-9d4f-2ebb70a31465.jpg">
  <b>ks = 2, dilations = [1, 2, 4, 8], 2 blocks</b><br><br>
</p>


- If we increased the number of stacks to 3, the size of the receptive field would increase again, such as below:

<p align="center">
  <img src="https://user-images.githubusercontent.com/40159126/41830628-ae6e73d4-7874-11e8-8ecd-cea37efa33f1.jpg">
  <b>ks = 2, dilations = [1, 2, 4, 8], 3 blocks</b><br><br>
</p>

Thanks a lot to [@alextheseal](https://github.com/alextheseal) for providing such visuals.

### Non-causal TCN

Making the TCN architecture non-causal allows it to take the future into consideration to do its prediction as shown in the figure below.

However, it is not anymore suitable for real-time applications.

<p align="center">
  <img src="misc/Non_Causal.png">
  <b>Non-Causal TCN - ks = 3, dilations = [1, 2, 4, 8], 1 block</b><br><br>
</p>

To use a non-causal TCN, specify `padding='valid'` or `padding='same'` when initializing the TCN layers.

Special thanks to: [@qlemaire22](https://github.com/qlemaire22)

## Installation (Python 3)

```bash
git clone git@github.com:philipperemy/keras-tcn.git
cd keras-tcn
virtualenv -p python3.6 venv
source venv/bin/activate
pip install -r requirements.txt # change to tensorflow if you dont have a gpu.
pip install . --upgrade # install it as a package.
```

Note: Only compatible with Python 3 at the moment. Should be almost compatible with python 2.

## Run

Once `keras-tcn` is installed as a package, you can take a glimpse of what's possible to do with TCNs. Some tasks examples are  available in the repository for this purpose:

```bash
cd adding_problem/
python main.py # run adding problem task

cd copy_memory/
python main.py # run copy memory task

cd mnist_pixel/
python main.py # run sequential mnist pixel task
```

## Tasks

### Adding Task

The task consists of feeding a large array of decimal numbers to the network, along with a boolean array of the same length. The objective is to sum the two decimals where the boolean array contain the two 1s.

#### Explanation

<p align="center">
  <img src="misc/Adding_Task.png">
  <b>Adding Problem Task</b><br><br>
</p>

#### Implementation results

The model takes time to learn this task. It's symbolized by a very long plateau (could take ~8 epochs on some runs).

```
200000/200000 [==============================] - 293s 1ms/step - loss: 0.1731 - val_loss: 0.1662
200000/200000 [==============================] - 289s 1ms/step - loss: 0.1675 - val_loss: 0.1665
200000/200000 [==============================] - 287s 1ms/step - loss: 0.1670 - val_loss: 0.1665
200000/200000 [==============================] - 288s 1ms/step - loss: 0.1668 - val_loss: 0.1669
200000/200000 [==============================] - 285s 1ms/step - loss: 0.1085 - val_loss: 0.0019
200000/200000 [==============================] - 285s 1ms/step - loss: 0.0011 - val_loss: 4.1667e-04
200000/200000 [==============================] - 282s 1ms/step - loss: 6.0470e-04 - val_loss: 6.7708e-04
200000/200000 [==============================] - 282s 1ms/step - loss: 4.3099e-04 - val_loss: 7.3898e-04
200000/200000 [==============================] - 282s 1ms/step - loss: 3.9102e-04 - val_loss: 1.8727e-04
200000/200000 [==============================] - 280s 1ms/step - loss: 3.1040e-04 - val_loss: 0.0010
200000/200000 [==============================] - 281s 1ms/step - loss: 3.1166e-04 - val_loss: 2.2333e-04
200000/200000 [==============================] - 281s 1ms/step - loss: 2.8046e-04 - val_loss: 1.5194e-04
```

### Copy Memory Task

The copy memory consists of a very large array:
- At the beginning, there's the vector x of length N. This is the vector to copy.
- At the end, N+1 9s are present. The first 9 is seen as a delimiter.
- In the middle, only 0s are there.

The idea is to copy the content of the vector x to the end of the large array. The task is made sufficiently complex by increasing the number of 0s in the middle.

#### Explanation

<p align="center">
  <img src="misc/Copy_Memory_Task.png">
  <b>Copy Memory Task</b><br><br>
</p>

#### Implementation results (first epochs)

```
30000/30000 [==============================] - 30s 1ms/step - loss: 0.1174 - acc: 0.9586 - val_loss: 0.0370 - val_acc: 0.9859
30000/30000 [==============================] - 26s 874us/step - loss: 0.0367 - acc: 0.9859 - val_loss: 0.0363 - val_acc: 0.9859
30000/30000 [==============================] - 26s 852us/step - loss: 0.0361 - acc: 0.9859 - val_loss: 0.0358 - val_acc: 0.9859
30000/30000 [==============================] - 26s 872us/step - loss: 0.0355 - acc: 0.9859 - val_loss: 0.0349 - val_acc: 0.9859
30000/30000 [==============================] - 25s 850us/step - loss: 0.0339 - acc: 0.9864 - val_loss: 0.0291 - val_acc: 0.9881
30000/30000 [==============================] - 26s 856us/step - loss: 0.0235 - acc: 0.9896 - val_loss: 0.0159 - val_acc: 0.9944
30000/30000 [==============================] - 26s 872us/step - loss: 0.0169 - acc: 0.9929 - val_loss: 0.0125 - val_acc: 0.9966
```

### Sequential MNIST

#### Explanation

The idea here is to consider MNIST images as 1-D sequences and feed them to the network. This task is particularly hard because sequences are 28*28 = 784 elements. In order to classify correctly, the network has to remember all the sequence. Usual LSTM are unable to perform well on this task.

<p align="center">
  <img src="misc/Sequential_MNIST_Task.png">
  <b>Sequential MNIST</b><br><br>
</p>

#### Implementation results

```
60000/60000 [==============================] - 118s 2ms/step - loss: 0.2348 - acc: 0.9265 - val_loss: 0.1308 - val_acc: 0.9579
60000/60000 [==============================] - 116s 2ms/step - loss: 0.0973 - acc: 0.9698 - val_loss: 0.0645 - val_acc: 0.9798
[...]
60000/60000 [==============================] - 112s 2ms/step - loss: 0.0075 - acc: 0.9978 - val_loss: 0.0547 - val_acc: 0.9894
60000/60000 [==============================] - 111s 2ms/step - loss: 0.0093 - acc: 0.9968 - val_loss: 0.0585 - val_acc: 0.9895
```


## References
- https://github.com/locuslab/TCN/ (TCN for Pytorch)
- https://arxiv.org/pdf/1803.01271.pdf (An Empirical Evaluation of Generic Convolutional and Recurrent Networks
for Sequence Modeling)
- https://arxiv.org/pdf/1609.03499.pdf (Original Wavenet paper)

## Useful links
- https://github.com/Baichenjia/Tensorflow-TCN (Tensorflow Eager implementation of TCNs)
- https://github.com/philipperemy/keras-tcn/blob/master/README.md (Keras implementation of TCNs)
