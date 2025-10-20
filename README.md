![BEYOND-RGB](https://github.com/user-attachments/assets/8c9a744e-601f-4f8f-93b4-b4a3730a6754)
## Description

Welcome to BEYOND-RGB, a repository that allows for easy integration and training of early fusion multimodal modles.
The codebase is designed to be modular, where all configs are in one place and different tasks can be started from various script endpoints using these configs. It is also supposed to be easily extendable - one can just add a different model, loss function or backbone and use it with the same scripts as everything else.

> [!WARNING]  
> This repository is still under construction and the full code will be published upon our article publication


## Repository Structure

We are trying to keep the repo modular and easily configurable using yaml configs. 

 ```
    SEAMORE
    ├── configs 
    │   ├── Contains all the configs for vaeious tasks, such as training
    ├── dataset
    │   ├── Contains dataloaders
    │   ├── Used as module, other datasets can be added and selected in config
    .
    .     
    ├── models 
    │   ├── Defines architectures, anything can be added
    │   ├── Imported as a module, everything inside can be usable in configs
    ├── scoring
    │   ├── Defines metric and loss functions
    │   ├── Imported as a module, add losses and select them in config
    ├── trainer
    │   ├── Defines data propagation in an epoch
    │   ├── Handles both training and validation
    │   ├── Adapts to specific tasks:
    │   │   ├── Semantic segmentation
    │   │   ├── Object detection
    │   │   ├── Classification
    │   ├── Imported as a module
    ├── utils
    │   ├── utilitary functions, such as visualization classes, config loading functions etc., and lastly
    ├── train.py <- main training entrypoint

```


## Docker Container Setup

We provide a dockerfile for quick environment setup to allow easy model experimentation.

### Prerequisites

The Docker container must be run on a **Linux machine** with **CUDA support**.

---

### 1. Prepare the Machine

Start with nvidia repo addition:

```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Then install Nvidia Container Toolkit

```
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

Configure the container runtime by using the nvidia-ctk command:
```
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```
The nvidia-ctk command modifies the /etc/docker/daemon.json file on the host. The file is updated so that Docker can use the NVIDIA Container Runtime.
For more details, check [this](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) tutorial.

---

### 2. Build the Docker Container

Run the following command to build the Docker image:

```sh
docker build -t trt-seamore -f Dockerfile .
```

---

### 3. Setup Environment Variables

To configure the environment variables, follow these steps:

1. **Create a local `.env` file** in the repository.
  
2. **Retrieve credentials** from Bitwarden:
  
  - Paste them into the `.env` file in the following format:
  - Copy the HuggingFace key from the item **'huggingface-cli'**, if you want to use hugging face for model uploads
  - Copy the Weigths & Biases key from the item **'wandb-api'**, if you want to use weights and biases for logging
  
  ```env
  WANDB_API_KEY="???"
  HUGGING_FACE_HUB_TOKEN="???"
  ```
  
---

### 4. Start the Container

Use the following command to start the container:

```sh
docker compose up -d
```

This will run the container in detached mode.

---

### Notes

- Ensure your Linux machine has CUDA support enabled.
- If any issues arise, check the `.env` file for correct values.
- Use `docker ps` to verify that the container is running.
- Use `docker logs <container_id>` to debug any errors.



## Training instructions 
Main entrypoint to the training is _train.py_ file, which traines the network provided in a config given as argument. Besides the network name, config file allows for setting of many other training aspects allowing for high modularity of the code - such as datasets, dataloaders, trainers, loss functions, metrics hyperparameters and way more. These can be then customly defined in the respective modules of this codebase and then used in the config. It is probably easier to check some of the config straight away than explaining it here. 

Now, to training.
There is now only one dataloader which allows for most of the flexibility necessary for training on different Fiftyone datasets. FO datasets are the main bases, from which are the data taken. 
For more details please refer to __dataset/datasets.py__. 
After selecting a dtaset, you can create a config such as any of the configs in __configs/train__ and start training as

    python train.py -c /path/to/your/config.yaml

Note that configs can be both .json or .yaml (I started with .json, but then realized there is too many brackets :D, but .json is kept for backwards compatibility).

You can view the results of your training in the browser after running

    tensorboard --logdir=/outpath/you/have/in/confing --host 0.0.0.0

since the default training visualization interface is tensorboard. However, it is easy to define other interface such as WandB in __utils/visualization.py__.
