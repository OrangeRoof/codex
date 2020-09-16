![The CoDEx logo.](codex_logo.png)

CoDEx is a set of knowledge graph **Co**mpletion **D**atasets **Ex**tracted from Wikidata and Wikipedia. 
CoDEx offers three rich knowledge graph datasets that contain positive and hard negative triples, entity types, entity and relation descriptions, and Wikipedia page extracts for entities. 
We provide baseline performance results, configuration files, and pretrained models
on CoDEx using the [LibKGE](https://github.com/uma-pi1/kge) library for two knowledge graph completion tasks, link prediction and triple classification.

The statistics for each CoDEx dataset are as follows:

|          | Entities | Relations | Train   | Valid (+) | Test (+) | Valid (-) | Test (-) | Total triples |
|----------|---------:|----------:|--------:|----------:|---------:|----------:|---------:|--------------:|
| CoDEx-S  | 2,034    | 42        | 32,888  | 1,827     | 1,828    | 1,827     | 1,828    | 36,543        |
| CoDEx-M  | 17,050   | 51        | 185,584 | 10,310    | 10,311   | 10,310    | 10,311   | 206,205       |
| CoDEx-L  | 77,951   | 69        | 551,193 | 30,622    | 30,622   | -         | -        | 612,437       |
| Raw dump | 380,038  | 75        | -       | -         | -        | -         | -        | 1,156,222     |

## Table of contents
1. <a href="#quick-start">Quick start</a>
2. <a href="#explore">Data exploration and analysis</a>
3. <a href="#models">Pretrained models and results</a>
    - <a href="#kge">LibKGE setup</a>
    - <a href="#scripts">Reproducing our results</a>
      - <a href="#baseline-script">Link prediction baseline</a>
      - <a href="#tc-script">Triple classification</a>
    - <a href="#pretrained">Downloading pretrained models via the command line</a>
    - <a href="#lp">Link prediction results</a>
      - <a href="#s-lp">CoDEx-S</a>
      - <a href="#m-lp">CoDEx-M</a>
      - <a href="#l-lp">CoDEx-L</a>
    - <a href="#tc">Triple classification results</a>
      - <a href="#s-tc">CoDEx-S</a>
      - <a href="#m-tc">CoDEx-M</a>
4. <a href="#data">Data directory structure</a>
    - <a href="#entities">Entities and entity types</a>
    - <a href="#relations">Relations</a>
    - <a href="#triples">Triples</a>
5. <a href="#cite">How to cite</a>
6. <a href="#ref">References and acknowledgements</a>

## <a id="quick-start">Quick start</a>

```
# unzip the repository
git clone https://github.com/tsafavi/codex.git
cd codex

# extract English Wikipedia plain-text excerpts for entities
# other language codes available: ar, de, es, ru, zh
./extract.sh en

# set up a virtual environment and install the Python requirements
python3.7 -m venv myenv
source myenv/bin/activate
pip install -r requirements.txt

# finally, install the codex data-loading API
pip install -e .
```

## <a id="explore">Data exploration and analysis</a>

To get familiar with the CoDEx datasets and the data-loading API in an easy-to-use interface, we have provided an exploration notebook with Jupyter. 
To launch:
```
# run from codex/
python -m ipykernel install --user --name=myenv  # register your venv with jupyterlab
jupyter lab
```
Now, navigate to JupyterLab in your browser and open the ```Explore CoDEx.ipynb``` notebook in your browser,
which provides a glimpse into each dataset, for example frequent entities and relations, negative triples, compositional (multi-hop) paths and symmetry, etc.

If you are working on a server and want to run JupyterLab remotely, check out [these instructions](https://www.blopig.com/blog/2018/03/running-jupyter-notebook-on-a-remote-server-via-ssh/) on how to set up port forwarding to view your remotely-hosted notebook locally. 

## <a id="models">Pretrained models and results</a>

### <a id="kge">LibKGE setup</a>

To **use the pretrained models or run any scripts that involve pretrained models**, you will need to set up [LibKGE](https://github.com/uma-pi1/kge).
Run the following: 
```
# run from codex/
# this may take a few minutes
./libkge_setup.sh
```
This script will install the library in the ```kge/``` directory inside your venv, download the FB15K-237 dataset (which we use in our experiments) to ```kge/data/```, and copy each CoDEx dataset to ```kge/data/``` and preprocess each dataset according to the format the LibKGE requires. 

### <a id="scripts">Reproducing our results</a>

For the evaluation results not obtained using LibKGE's testing API, we provide several additional evaluation scripts to reproduce results in our paper. All scripts are written to run on a CPU. These scripts assume that you have set up LibKGE using the script we provided. 

#### <a id="baseline-script">Link prediction baseline</a>

```scripts/baseline.py``` compares a simple frequency baseline to the best model on CoDEx-M and the FB15K-237 benchmark.
The results are saved to CSV files named ```fb.csv``` and ```codex.csv```, respectively. 
To run:
```
# run from codex/
scripts/baseline.sh
```
Note that this script first downloads the [best pretrained LibKGE model on FB15K-237](https://github.com/uma-pi1/kge#results-and-pretrained-models) to ```models/link-prediction/fb15k-237/rescal/``` and the best link prediction model on CoDEx-M to ```models/link-prediction/codex-m/complex/``` if they do not already exist. 

#### <a id="tc-script">Triple classification</a>

```scripts/tc.py``` runs triple classification and outputs validation and test accuracy/F1. 
To run:
```
# run from codex/
scripts/tc.sh
```
Note that this script first downloads all triple classification models on CoDEx-S and CoDEx-M and saves them to ```models/triple-classification/{codex-s,codex-m}/``` if they do not already exist. 

### <a id="pretrained">Downloading pretrained models via the command line</a>

To **download pretrained models via the command line**, use our ```download_pretrained.py``` Python script.
The arguments are as follows:
```
usage: download_pretrained.py [-h]
                              {s,m,l} {triple-classification,link-prediction}
                              {rescal,transe,complex,conve}
                              [{rescal,transe,complex,conve} ...]

positional arguments:
  {s,m,l}               CoDEx dataset to download model(s)
  {triple-classification,link-prediction}
                        Task to download model(s) for
  {rescal,transe,complex,conve}
                        Model(s) to download for this task
```
For example, if you want to download the pretrained **link prediction** models for **ComplEx and ConvE** on **CoDEx-M**:
```
# run from codex/
python download_pretrained.py m link-prediction complex conve
```
This script will place a ```checkpoint_best.pt``` LibKGE checkpoint file in ```models/link-prediction/codex-m/complex/``` and ```models/link-prediction/codex-m/conve/```, respectively. 

Alternatively, you can download the models manually following the links we provide here. 


### <a id="lp">Link prediction results</a>

#### <a id="s-lp">CoDEx-S</a>

|  | MRR | Hits@1 | Hits@10 | Config file | Pretrained model |
|---------|----:|-------:|--------:|------------:|-----------------:|
| RESCAL | 0.404 | 0.293 | 0.623 | <a href="models/link-prediction/codex-s/rescal/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/v209jchl93mmeuv/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| TransE | 0.354 | 0.219 | 0.634 | <a href="models/link-prediction/codex-s/transe/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/8brqhb4bd5gnktc/checkpoint_best.pt?dl=0">NegSamp-kl</a> |
| ComplEx | 0.465 | 0.372 | 0.646 | <a href="models/link-prediction/codex-s/complex/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/kk3pgdnyddsdzn9/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| ConvE | 0.444 | 0.343 | 0.635 | <a href="models/link-prediction/codex-s/conve/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/atvu77pzed6mcgh/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| TuckER | 0.444 | 0.339 | 0.638 | <a href="models/link-prediction/codex-s/tucker/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/f87xloe2g3f4fvy/checkpoint_best.pt?dl=0">KvsAll-kl</a> |

#### <a id="m-lp">CoDEx-M</a>

|  | MRR | Hits@1 | Hits@10 | Config file | Pretrained model |
|---------|----:|-------:|--------:|------------:|-----------------:|
| RESCAL | 0.317 | 0.244 | 0.456 | <a href="models/link-prediction/codex-m/rescal/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/e3kp3eu4nnknn5b/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| TransE | 0.303 | 0.223 | 0.454 | <a href="models/link-prediction/codex-m/transe/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/y8uucaajpofct3x/checkpoint_best.pt?dl=0">NegSamp-kl</a> |
| ComplEx | 0.337 | 0.262 | 0.476 | <a href="models/link-prediction/codex-m/complex/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/psy21fvbn5pbmw6/checkpoint_best.pt?dl=0">KvsAll-kl</a> |
| ConvE | 0.318 | 0.239 | 0.464 | <a href="models/link-prediction/codex-m/conve/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/awjhlrfjrgz9phi/checkpoint_best.pt?dl=0">NegSamp-kl</a> |
| TuckER | 0.328 | 0.259 | 0.458 | <a href="models/link-prediction/codex-m/tucker/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/so5l2owtx7wcos1/checkpoint_best.pt?dl=0">KvsAll-kl</a> |

#### <a id="l-lp">CoDEx-L</a>

|  | MRR | Hits@1 | Hits@10 | Config file | Pretrained model |
|---------|----:|-------:|--------:|------------:|-----------------:|
| RESCAL | 0.304 | 0.242 | 0.419 | <a href="models/link-prediction/codex-l/rescal/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/wvbef9u98vmkbi8/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| TransE | 0.187 | 0.116 | 0.317 | <a href="models/link-prediction/codex-l/transe/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/s9d682b49tuq5mc/checkpoint_best.pt?dl=0">NegSamp-kl</a> |
| ComplEx | 0.294 | 0.237 | 0.400 | <a href="models/link-prediction/codex-l/complex/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/jqubvr77og2pvzv/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| ConvE | 0.303 | 0.240 | 0.420 | <a href="models/link-prediction/codex-l/conve/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/qcfjy6i1sqbec0z/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| TuckER | 0.309 | 0.244 | 0.430 | <a href="models/link-prediction/codex-l/tucker/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/j8u4nqwzz3v7jw1/checkpoint_best.pt?dl=0">KvsAll-kl</a> |

### <a id="tc">Triple classification results</a>


#### <a id="s-tc">CoDEx-S</a>

|  | Acc | F1 | Config file | Pretrained model |
|--------|----:|---:|------------:|-----------------:|
| RESCAL | 0.843 | 0.852 | <a href="models/triple-classification/codex-s/rescal/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/qedwhaus6gwyf1z/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| TransE | 0.829 | 0.837 | <a href="models/triple-classification/codex-s/transe/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/jkp6oxcgt28ki42/checkpoint_best.pt?dl=0">NegSamp-kl</a> |
| ComplEx | 0.836 | 0.846 | <a href="models/triple-classification/codex-s/complex/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/2d8clm7em6ygida/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| ConvE | 0.841 | 0.846 | <a href="models/triple-classification/codex-s/conve/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/4rnexlf56x5qwvs/checkpoint_best.pt?dl=0">1vsAll-kl</a> |
| TuckER | 0.840 | 0.846 | <a href="models/triple-classification/codex-s/tucker/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/xrlfygg6ck2z3ue/checkpoint_best.pt?dl=0">KvsAll-kl</a> |

#### <a id="m-tc">CoDEx-M</a>

|  | Acc | F1 | Config file | Pretrained model |
|--------|----:|---:|------------:|-----------------:|
| RESCAL | 0.818 | 0.815 | <a href="models/triple-classification/codex-m/rescal/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/366h6xwccbvqkm8/checkpoint_best.pt?dl=0">KvsAll-kl<a/> |
| TransE | 0.797 | 0.803 | <a href="models/triple-classification/codex-m/transe/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/0uil6mrrtadtqoe/checkpoint_best.pt?dl=0">NegSamp-kl</a> |
| ComplEx | 0.824 | 0.818 | <a href="models/triple-classification/codex-m/complex/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/0yh95rtgvv12qxs/checkpoint_best.pt?dl=0">KvsAll-kl</a> |
| ConvE | 0.826 | 0.829 | <a href="models/triple-classification/codex-m/conve/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/s9fwf1v57mm23l8/checkpoint_best.pt?dl=0">KvsAll-kl</a> |
| TuckER | 0.823 | 0.816 | <a href="models/triple-classification/codex-m/tucker/config.yaml">config.yaml</a> | <a href="https://www.dropbox.com/s/jj09uah9cjkukl0/checkpoint_best.pt?dl=0">KvsAll-kl</a> |


## <a id="data">Data directory structure</a>

The ```data/``` directory is structured as follows:
```
.
├── entities
│   ├── ar
│   ├── de
│   ├── en
│   ├── es
│   ├── ru
│   └── zh
├── relations
│   ├── ar
│   ├── de
│   ├── en
│   ├── es
│   ├── ru
│   └── zh
├── triples
│   ├── codex-l
│   ├── codex-m
│   ├── codex-s
│   └── raw.zip
└── types
    ├── ar
    ├── de
    ├── en
    ├── entity2types.json
    ├── es
    ├── ru
    └── zh
 ```
 
 We provide an overview of each subdirectory in this section. 
 
### <a id="entities">Entities and entity types</a>
We provide entity labels, Wikidata descriptions, and Wikipedia page extracts for entities and entity types in six languages:
Arabic (ar), German (de), English (en), Spanish (es), Russian (ru), and Chineze (zh).

Each subdirectory of ```data/entities/``` contains an ```entities.json``` file formatted as follows:
```
{
  <Wikidata entity ID>:{
    "label":<label in respective language if available>,
    "description":<Wikidata description in respective language if available>,
    "wiki":<Wikipedia page URL in respective language if available>
  }
}
```
For the labels, descriptions, or Wikipedia URLs that are not available in a given language, the value will be the empty string.

The file ```data/types/entity2types.json``` maps each Wikidata entity ID to a list of Wikidata type IDs, i.e.,
```
{
  "<Wikidata entity ID>":[
    <Wikidata type ID 1>,
    <Wikidata type ID 2>,
    ...
  ]
}
```
Each subdirectory of ```data/types/``` contains a ```types.json``` file formatted as follows: 
```
{
  <Wikidata type ID>:{
    "label":<label in respective language if available>,
    "description":<Wikidata description in respective language if available>,
    "wiki":<Wikipedia page URL in respective language if available>
  }
}
```
Each ```extracts.zip``` file contains zipped files of entity descriptions from Wikipedia.
Each file is named ```<Wikidata entity ID>.txt```. 
We provide the ```extract_en.sh``` script to unzip all English-language entity and entity type extracts.
You can edit this script and provide a different language code (```ar``` for Arabic, ```de``` for German, ```es``` for Spanish, ```ru``` for Russian, and ```zh``` for Chinese) to extract descriptions for other languages. 

### <a id="relations">Relations</a>
We provide relation labels and Wikidata descriptions for relations in six languages: 
Arabic (ar), German (de), English (en), Spanish (es), Russian (ru), and Chineze (zh).

Each subdirectory of ```data/relations/``` contains a ```relations.json``` file formatted as follows:
```
{
  <Wikidata relation ID>:{
    "label":<label in respective language if available>,
    "description":<Wikidata description in respective language if available>
  }
}
```

### <a id="triples">Triples</a>

Each triple file follows the format
```
<Wikidata head entity ID>\t<Wikidata relation ID>\t<Wikidata tail entity ID>
```
without any header or extra information per line.

If you'd like to use the raw data dump, run
```
cd data/triples
unzip raw.zip
```
This will create a new ```data/triples/raw/``` directory containing a single file, ```triples.txt```, in the same tab-separated format as the other triple files. 

# <a id="cite">How to cite</a>
If you used our work or found it helpful, please use the following citation: 
```
@inproceedings{
  safavi2020codex,
  title={CoDEx: A Comprehensive Knowledge Graph Completion Benchmark},
  author={Tara Safavi and Danai Koutra},
  booktitle={Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing},
  year={2020},
}
```

# <a id="ref">References and acknowledgements</a>

We thank [HeadsOfBirds](https://thenounproject.com/search/?q=lightbulb&i=1148270) for the lightbulb icon and [Evan Bond](https://thenounproject.com/icon/1113230/) for the book icon in our logo.

