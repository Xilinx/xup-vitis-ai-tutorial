---
layout: default
---

# Setting up Vitis AI on Amazon AWS

In this lab you will go through the necessary steps to setup an instance to run Vitis-AI toolchain. You will start with the Canonical [Ubuntu 18.04 LTS](https://aws.amazon.com/marketplace/pp/Canonical-Group-Limited-Ubuntu-1804-LTS-Bionic/B07CQ33QKV) AMI.

[Start an AWS EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launch-marketplace-console.html) of type [f1.2xlarge](https://aws.amazon.com/ec2/instance-types/f1/)  using the Canonical [Ubuntu 18.04 LTS](https://aws.amazon.com/marketplace/pp/Canonical-Group-Limited-Ubuntu-1804-LTS-Bionic/B07CQ33QKV) AMI.

After starting this instance you must ssh to your cloud instance to complete the following steps if remote desktop feature is not available.

Since the remote desktop capability is provided in the instance, start a RDP/DCV session. Open a terminal window and carry out the following steps:

#### Disable Kernel Auto-Upgrade

```sh
cd
sudo sed -i 's/1/0/g' /etc/apt/apt.conf.d/20auto-upgrades
```

#### Update Ubuntu packages list, and upgrade existing packages

```sh
sudo apt-get update
sudo apt-get upgrade
```
> **Note** If you see an error indicating something is locked while running the 2nd command above then restart the instance and re-run the second command. You may have to run `sudo dpkg --configure -a`.

#### Install AWS FPGA Management library and Xilinx XRT

```sh
cd /home/ubuntu
git clone https://github.com/Xilinx/XRT.git -b 2021.1
sudo dpkg --configure -a
sudo ./XRT/src/runtime_src/tools/scripts/xrtdeps.sh

cd
git clone https://github.com/aws/aws-fpga.git
cd aws-fpga
source sdk_setup.sh

cd
cd XRT/build
./build.sh
sudo apt install ./Release/*-xrt.deb
sudo apt install ./Release/*-aws.deb
cd
```

#### Install XRM

```sh
wget https://www.xilinx.com/bin/public/openDownload?filename=xrm_202110.1.2.1539_18.04-x86_64.deb -O xrm.deb
sudo apt install ./xrm.deb
```

#### Install the DPU Accelerator (FPGA Binary).

You will use `DPUCADF8H` DPU. Find the DPU naming information [here](https://github.com/Xilinx/Vitis-AI/blob/master/docs/learn/dpu_naming.md).

```sh
wget https://www.xilinx.com/bin/public/openDownload?filename=dpu-aws-1.4.0.xclbin -O dpu-aws.xclbin
sudo mkdir -p /opt/xilinx/overlaybins/DPUCADF8H
sudo cp dpu-aws.xclbin /opt/xilinx/overlaybins/DPUCADF8H
sudo chmod -R a+rx /opt/xilinx/overlaybins/DPUCADF8H
```

#### [Install Docker](https://docs.docker.com/engine/install/ubuntu/)

You will run the Vitis-AI toolchain in the docker image.

```sh
cd
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker ubuntu
sudo chmod 666 /var/run/docker.sock
```
Logout and start the RDP session again.

#### Install `gedit` program

```sh
sudo apt install gedit
```

#### Clone Vitis-AI and update the DPU related `setup.sh` file

```sh
cd
git clone https://github.com/Xilinx/Vitis-AI.git -b1.4.1 Vitis-AI_1_4_1
cd Vitis-AI_1_4_1

```

Update `setup.sh` file content under Vitis-AI_1_4_1/setup/alveo directory to define `aws` as the target board.

```sh
gedit setup/alveo/setup.sh
```

Add `aws` at the end of line 42 which defines available platforms so it reads like:

```sh
PLATFORMS="u50_ u50lv_ u200_ u250_ u280_ aws"
```

Add following lines at line number `94` to load appropriate xclbin binary when *DPUCADF8H* is the target DPU.

```sh
elif [ "${platform}" = "aws" ]; then
  export XCLBIN_PATH=/opt/xilinx/overlaybins/DPUCADF8H
  export XLNX_VART_FIRMWARE=/opt/xilinx/overlaybins/DPUCADF8H/dpu-aws.xclbin
```

Save and close the file.

- [Run pre-built example on F1](./running_on_F1.md)

---------------------------------------
<p align="center">Copyright&copy; 2022 Xilinx</p>
