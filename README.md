## Introduction
Source code for [Root Cause Analysis of Failures in Microservices through Causal Discovery](https://proceedings.neurips.cc/paper_files/paper/2022/file/c9fcd02e6445c7dfbad6986abee53d0d-Paper-Conference.pdf).

This code will apply the algorithm described in the paper to a dataset in a vaccuum in order to attempt to *quickly* identify a potential root cause for a cascading failure within a microservice mesh. The RCD script is implemented to provide a top-k result based on it's iteration over the data which can be used to identify root cause with the top result as the most likely.

## Setup
The following instructions assume that you are running Ubuntu-22.04.
#### Install python env
```bash
sudo apt update
sudo apt install -y build-essential \
                    python-dev \
                    python3-venv \
                    python3-pip \
                    libxml2 \
                    libxml2-dev \
                    zlib1g-dev \
                    python3-tk \
                    graphviz

cd ~
python3 -m venv env
source env/bin/activate
python3 -m pip install --upgrade pip
```

#### Install dependencies
```bash
git clone https://github.com/sunayu/rcd.git
cd rcd
pip install -r requirements.txt
```

#### Link modifed files
To implement RCD, we modified some code from pyAgrum and causal-learn.
Some of these changes expose some internal information for reporting results (for example number of CI tests while executing PC) or modify the existing behaviour (`local_skeleton_discovery` in `SekeletonDiscovery.py` implements the localized approach for RCD). A few of these changes also fix some minor bugs.

Assuming the rcd repository was cloned at home, execute the following;
```bash
ln -fs ~/rcd/pyAgrum/lib/image.py ~/env/lib/python*/site-packages/pyAgrum/lib/
ln -fs ~/rcd/causallearn/search/ConstraintBased/FCI.py ~/env/lib/python*/site-packages/causallearn/search/ConstraintBased/
ln -fs ~/rcd/causallearn/utils/Fas.py ~/env/lib/python*/site-packages/causallearn/utils/
ln -fs ~/rcd/causallearn/utils/PCUtils/SkeletonDiscovery.py ~/env/lib/python*/site-packages/causallearn/utils/PCUtils/
ln -fs ~/rcd/causallearn/graph/GraphClass.py ~/env/lib/python*/site-packages/causallearn/graph/
```

## Data
This repo also includes data for testing in the form of gen\_data.py and the sock-shop-data directory.

### Gen_Data
The Gen data script generatess synthetic data for testing of rcd.py. This code generates two csvs, one containing data under normal operational load and one with "anomalous" data that represents data during a cascading failure event, as well as a pkl graph file and ground-truth.pdf which will describe the service mesh architecture with the particular root cause node highlighted.

#### Generate Synthetic Data
```sh
./gen_data.py
```

### sock-shop-data
This directory contains a variety of anomalous and normal csvs similar to the synthetic data but instead using a variety of collected metrics from prometheus in a running sock-shop instance. See [microservices-demo](https://github.com/sunayu/microservices-demo) for a similar repo. These csv combos are arranged by directory, named for the microservice and error type (cpu or mem).

#### collect-data.py
This is a bare bones python script for collecting these metrics from prometheus.

Usage:
```sh
./collect-data.py --ip [prometheus-ip] --start [collection-start-time] --end [collection-end-time]
```
By default collect-data.py will place these metrics in a file named `data.csv`. An alternative location can be specified with `--name [filename]`. `--append` can be used to append additional results to the file.

## Using RCD

#### Executing RCD with Synthetic Data
```sh
./rcd.py --path [PATH_TO_DATA] --local --k 3
```

`--local` options enables the localized RCD while `--k` estimates the top-`k` root causes.
