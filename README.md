# Im2Text

A deep learning-based approach to learning the image-to-text conversion, built on top of the <a href="https://opennmt.github.io/">OpenNMT</a> system. It is completely data-driven, hence can be used for a variety of image-to-text problems, such as image captioning, optical character recognition and LaTeX decompilation. 

Take LaTeX decompilation as an example, given a formula image:

<p align="center"><img src="http://lstm.seas.harvard.edu/latex/results/website/images/119b93a445-orig.png"></p>

The goal is to infer the LaTeX source that can be compiled to such an image:

```
 d s _ { 1 1 } ^ { 2 } = d x ^ { + } d x ^ { - } + l _ { p } ^ { 9 } \frac { p _ { - } } { r ^ { 7 } } \delta ( x ^ { - } ) d x ^ { - } d x ^ { - } + d x _ { 1 } ^ { 2 } + \; \cdots \; + d x _ { 9 } ^ { 2 } 
```

The paper (http://arxiv.org/pdf/1609.04938v1.pdf) provides more technical details of this model.

## Installation

Im2Text is built on top of <a href="https://opennmt.github.io/">OpenNMT</a>. You can either install it as a package by `luarocks install --local https://raw.githubusercontent.com/OpenNMT/OpenNMT/master/rocks/opennmt-scm-1.rockspec` or provide the path to <a href="https://opennmt.github.io/">OpenNMT</a> at run time. It also depends on `tds`, `cudnn` and `cutorch`. Currently we only support **GPU**.


## Quick Start

To get started, we provide a toy Math-to-LaTex example. We assume that the working directory is `Im2Text` throughout this document.

Im2Text consists of two commands:

1) Train the model.

```
th src/train.lua -phase train -gpu_id 1 -input_feed -model_dir model \
-image_dir data/images -data_path data/train.txt -val_data_path data/validate.txt -label_path data/labels.txt -vocab_file data/vocab.txt \
-batch_size 8 -beam_size 1 \
-max_num_tokens 150 -max_image_width 500 -max_image_height 160
```

2) Translate the images.

```
th src/train.lua -phase test -gpu_id 1 -load_model -model_dir model \
-image_dir data/images -data_path data/test.txt \
-output_dir results \
-batch_size 2 -beam_size 5 \
-max_num_tokens 500 -max_image_width 800 -max_image_height 800
```

The above dataset is sampled from the [processed-im2latex-100k-dataset](http://lstm.seas.harvard.edu/latex/processed-im2latex-100k-dataset.tgz). We provide a trained model [[link]](http://lstm.seas.harvard.edu/latex/model_latest) on this dataset. In order to use it, download and put it under `model_dir` before translating the images.

## Data Format

* `-image_dir`: The directory containing the images. Since images of the same size can be batched together, we suggest padding images of similar sizes to the same size in order to facilitate training.

* `-label_path`: The file storing the tokenized labels, one label per line. It shall look like:
```
<label0_token0> <label0_token1> ... <label0_tokenN0>
<label1_token0> <label1_token1> ... <label1_tokenN1>
<label2_token0> <label2_token1> ... <label2_tokenN2>
...
```

* `-data_path`: The file storing the image-label pairs. Each line starts with the path of the image (relative to `image_dir`), followed by the index of the label in `label_path` (index counts from 0). At test time, the label indexes can be omitted.
```
<image0_path> <label_index0>
<image1_path> <label_index1>
<image2_path> <label_index2>
...
```

* `-vocab_file`: The vocabulary file. Each line corresponds to a token. The tokens not in `vocab_file` will be considered unknown (UNK).


## Options

For a complete set of options, run `th src/train.lua -h`.
