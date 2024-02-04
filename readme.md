# ComfyUI on Cloud

The main goal of this repository is to provide a quick and easy method for individuals interested in stable diffusion to set up their own comfyUI server application with GPU support. This application can be accessed from any device capable of running a browser, including ARM-based Macs, iPads, and even smartphones, without concerns about performance, speed, or privacy issues.

> **NOTE:** As we use the spot VM feature to minimize costs, there may be times when GCP refuses to start the VM instance. To prevent this, it's possible to switch from a spot VM to a standard VM, but be aware that this will result in an approximate 60% increase in total costs.

## Cost Efficiency

Normally, a full month's usage of a T4 GPU with 4 VCPU cores, 16 GB RAM, and 100GB SSD amounts to approximately 150 dollars on Google Cloud Platform (GCP). However, by automating the VM's shutdown and startup processes and assuming an average usage of 3 hours per day, the cost is reduced to around 30 dollars per month.

| Avg Usage  | SSD   | RAM  | Number of VCPU Cores | Monthly Cost (Estimation) |
|------------|-------|------|----------------------|---------------------------|
| 3h per day | 75GB  | 16GB | 6                    | 29$                       |
| 3h per day | 100GB | 16GB | 6                    | 35$                       |
| 6h per day | 75GB  | 16GB | 6                    | 47$                       |
| 6h per day | 100GB | 16GB | 6                    | 52$                       |


---


## How to Install? (Takes only 15 mins!)

1. Create a GCP compute engine instance(VM) and Install the google CLI on your local machine(details below).
2. Log in to your VM and execute the following commands:

    ```bash
    git clone https://github.com/karaposu/comfyui-on-cloud
    chmod +x ./comfyui-on-cloud/src/install.sh
    chmod +x ./comfyui-on-cloud/src/virgin_vm.sh
    ./comfyui-on-cloud/src/virgin_vm.sh # run this only for new VM. This will install miniconda, cuda 11.8, torch.   
    ./comfyui-on-cloud/src/install.sh
    ```

    This will set up comfyUI, install popular extensions and model checkpoints, and include an automation script that automatically starts the comfyvm server whenever the VM is booted.

## How to Use 

1. To start comfyUI server, open your terminal in your local machine and run the command below.

   ```bash
    gcloud compute instances start comfyvm
    ```
   (Or if you are on your ipad, go to https://console.cloud.google.com/compute/instances and click start)

    Now you can access comfyUI through any browser by browsing:
[http://[external-ip-of-your-vm]:8188 ](http://[external-ip-of-your-vm]:8188 )
   
2. To shut down the server and avoid unnecessary CPU and GPU usage costs during idle times, run:

    ```bash
    gcloud compute instances stop comfyvm
    ```
---

## Explanation of Files for manual usage and debugging

Feel free to inspect all files or ask for clarification to ensure safety and suggest any improvements.

- **Installer:** `install.sh` script manages the installation and setup process.
- **Installer for Official ComfyUI repo:** `install_comfyui.sh`
- **Checkpoint Installer:** `install_checkpoints.sh`
- **Extensions Installer:** `install_extension.sh`
- **We are also dynamically creating `run_the_server.sh`** file and adding it to the systemd services to ensure comfyUI starts on boot: `/etc/systemd/system/comfyui.service`

## Detailed Tutorial

### 1. Creating a Linux VM with GPU Support and Authenticating Your Local Machine

1. **Create a Google Cloud Platform account** and activate the compute engine. 
2. **Upgrading to a paid account** (Search "billing accounts")
3. **Increase GPU quota** from 0 to 1 via Quotas & System Limits (search GPUs (all regions) inside the Quotas & System Limits )
4. **Create an Ubuntu 20.04 VM** with T4 GPU and add a TCP firewall rule for port 8188. (name your instance comfyvm, choose zone as europe-central2-b and make sure add your firewall port8188 tag in the network section  )
5. **Open terminal in your local machine and Install the gcloud CLI**:

    ```bash
    wget https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-461.0.0-darwin-arm.tar.gz
    tar -xzvf google-cloud-cli-461.0.0-darwin-arm.tar.gz
    ./google-cloud-sdk/install.sh
    ```

    If there are issues with the `gcloud` command, add the following to your `~/.bash_profile`, adjusting the paths as necessary:

    ```bash
    if [ -f '/Users/your_username/projects/google-cloud-sdk/path.bash.inc' ]; then . '/Users/your_username/projects/google-cloud-sdk/path.bash.inc'; fi
    if [ -f '/Users/your_username/projects/google-cloud-sdk/completion.bash.inc' ]; then . '/Users/your_username/projects/google-cloud-sdk/completion.bash.inc'; fi
    ```

6. **Authenticate using gcloud** on your local machine:

    ```bash
   gcloud auth login
    gcloud init
    ```

7. **Generate new SSH key pairs** on your local machine:

    ```bash
    ssh-keygen -t rsa -b 2048 -C "comfy_vm_key" -f ~/.ssh/comfy_vm_key
    ```
     
    This creates `.ssh/comfy_vm_key.pub` and `.ssh/comfy_vm_key` files.

8. **Authenticate the VM with your SSH key pairs** and log in from your local machine to the VM:

    ```bash
    cd ~/.ssh
    gcloud compute os-login ssh-keys add --key-file=comfy_vm_key.pub
    gcloud compute ssh comfyvm --zone europe-central2-b
    ```
### 2.Setting up ComfyUI Server

1. Log in to the VM:

    ```bash
    gcloud compute ssh comfyvm --zone europe-central2-b
    conda activate base
    ```

2. Clone the repo for ComfyUI installation scripts and execute them:

    ```bash
    git clone https://github.com/karaposu/ComfyUI-on-VM
    chmod +x ./ComfyUI-on-VM/src/virgin_vm.sh
    chmod +x ./ComfyUI-on-VM/src/install.sh
    ./ComfyUI-on-VM/src/virgin_vm.sh # 
    ./ComfyUI-on-VM/src/install.sh
    ```

    This process will automatically install a startup runner for the server and start the server. You can verify the installation by accessing `[external-ip-of-your-vm]:8188` in your browser to check if everything is working correctly.
  

 
##  TroubleShooting 
### If Installation finished but server doesnt start
1. Login to the VM using  "gcloud compute ssh comfyvm" if you cant login run 
"gcloud compute instances stop comfyvm" and wait couple of minutes. And then run 
"gcloud compute instances start comfyvm" and login again. 
2. If there is no problem with loginingyour VM then run "systemctl stop comfyui.service"
cd to ComfyUI directory and run 
"python main.py --listen". Check the terminal output and make sure comfyui starts without problem. 

3. If there are no problems then the problem is about firewall permissions. Go and check firewall rule for port8188 and make sure down below you see comfyvm. If not then you must edit comfyvm and add port8188 tag in network settings section


### if comfyvm throws error when running through "python main.py --listen"
1. Make sure conda is activated. 
2. make sure GPU is attached by running "nvidia-smi"
3. make sure Torch sees the GPU by running these lines
```bash
    echo -e "import torch\nprint(torch.cuda.is_available())\nprint(torch.cuda.get_device_name(0))" > test_cuda.py 
    python test_cuda.py
```


