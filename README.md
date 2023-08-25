<!-- 
# Copyright 2023, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
-->

# NVIDIA-Triton-Deployment-Quickstart

This README provides a guide for deploying a basic ResNet model (ONNX format) on the Triton Inference Server. 

This quickstart guide is an extended version of the official tutorial available at [triton-inference-server/tutorials/Quick_Deploy/ONNX/README.md](https://github.com/triton-inference-server/tutorials/blob/main/Quick_Deploy/ONNX/README.md). The official tutorial might be a bit succinct, especially for those new to the Triton Inference Server, so this guide aims to offer more detailed steps to make the deployment process more accessible.

If you're using Linux or MacOS, you can follow this quickstart using your terminal. However, for Windows OS users, please note that CMD will not work. Instead, you should use Windows PowerShell.

## Step 0: Install Docker
Please follow the Docker installation instructions that are tailored to your particular operating system. You can access comprehensive step-by-step guide [here](https://docs.docker.com/engine/install/).


## Step 1: Create Model Repository

To perform inference on your model with Triton, it's necessary to create a model repository.

The structure of the repository should be:
```
  <model-repository-path>/
    <model-name>/
      [config.pbtxt]
      [<output-labels-file> ...]
      <version>/
        <model-definition-file>
      <version>/
        <model-definition-file>
      ...
    <model-name>/
      [config.pbtxt]
      [<output-labels-file> ...]
      <version>/
        <model-definition-file>
      <version>/
        <model-definition-file>
      ...
    ...
```

(The `config.pbtxt` configuration file is optional. The configuration file will be autogenerated by Triton Inference Server if the user doesn't provide it.)


Therefore, the first step is to set up the directory structure for the model repository.
```bash
mkdir -p model_repository/densenet_onnx/1
```

Next, download the example ResNet model available online and place it in the appropriate directory.
```bash
wget -O model_repository/densenet_onnx/1/model.onnx "https://contentmamluswest001.blob.core.windows.net/content/14b2744cf8d6418c87ffddc3f3127242/9502630827244d60a1214f250e3bbca7/08aed7327d694b8dbaee2c97b8d0fcba/densenet121-1.2.onnx"
```
Now, by entering the command `tree`, you will observe the following directory structure, which aligns with the specifications of the model repository structure.
```
model_repository
|
+-- densenet_onnx
    |
    +-- 1
        |
        +-- model.onnx
```


## Step 2: Set Up Triton Inference Server
Please ensure that your current directory is located one level above the newly created model repository. This arrangement will allow the path `./model_repository` to refer to the actual model repository. If your current location is not in the desired location, navigate to that directory.

Next, run the pre-built docker container for Trition Inference Server
```bash
docker run --rm -p 8000:8000 -p 8001:8001 -p 8002:8002 -v "${PWD}/model_repository:/models" nvcr.io/nvidia/tritonserver:23.06-py3 tritonserver --model-repository=/models
```
If you encounter a permission error, prepend `sudo` to the command. In case Triton Inference Server version 23.06 is not available, you can refer to the [official release notes](https://docs.nvidia.com/deeplearning/triton-inference-server/release-notes/index.html) to identify the available versions.

Once the Docker image has been successfully pulled and the container is up and running, you should see a significant amount of information displayed. Within, you can find:
```
+---------------+---------+--------+
| Model         | Version | Status |
+---------------+---------+--------+
| densenet_onnx | 1       | READY  |
+---------------+---------+--------+
```
This indicates our model has been deployed on the server and is now ready to perform inference.

## Step 3: Set Up Triton Client
Run the pre-built docker container for Trition Client
```bash
docker run -it --rm --net=host -v "${PWD}:/workspace/" nvcr.io/nvidia/tritonserver:23.06-py3-sdk bash
```
If you encounter a permission error, prepend `sudo` to the command. In case Triton Inference Server version 23.06 is not available, you can refer to the [official release notes](https://docs.nvidia.com/deeplearning/triton-inference-server/release-notes/index.html) to identify the available versions.

Once the Docker image has been successfully pulled and the container is up and running, you will find yourself in an interactive Bash shell session within the container.

Install the `torchvision` package.
```bash
pip install torchvision
```

Download the example photo for conducting inference.
```bash
wget -O img1.jpg "https://www.hakaimagazine.com/wp-content/uploads/header-gulf-birds.jpg"
```

## Step 4: Using a Triton Client to Query the Server

Download the example python script `client.py` for querying
```bash
wget -O client.py "https://raw.githubusercontent.com/Achiwilms/NVIDIA-Triton-Deployment-Quickstart/main/client.py"
```

Execute the `client.py` script, then the inference request will be sent.
```bash
python client.py
```

Once the inference process is finished and the results are sent back to the client, the result will be printed. The output format here is `<confidence_score>:<classification_index>`. 
```
['11.549026:92' '11.232335:14' '7.528014:95' '6.923391:17' '6.576575:88']
```
To learn more about the request-making process, you can explore the client.py file. The comments within the script provide guidance and explanations that will aid you in navigating through it.

---

You've successfully deployed a model on the Triton Inference Server. Congratulations! 🎉 

On the other hand, if you encounter any challenges at any step, please feel free to contact me for assistance at this [email address](mailto:b09901074@g.ntu.edu.tw).