This file contains information on how to setup and customize a gcp vm to my liking and personal preferences

### Using VSCode to interact with the VM

1. Install the `remote - SSH` extension on VSCode

2. Start up a new VM instance on the cloud console.
    * Check both `allow http traffic` and `allow https traffic`.
    * Make sure to set up enough disk space in the boot disk - somewhat irritating to assign more disk space later. 500 GB should be enough for most uses.
    * **Make sure to use the UBUNTU 18.04 LTS image.** This makes it much easier to install GPU drivers and use `tensorflow` later on if needed.  

3. Attach your SSH keys to the instance so that VSCode can be used to SSH into this instance:
    * Generate a new SSH key if needed. See instructions [here](https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys#createsshkeys)
    * Edit the instance and then copy and paste the contents of the `{key_name}.pub` file into the box. Save. 

4. Reserve a static IP for the instance. This helps a lot later when trying to connect with VSCode. Instructions [here](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address)

5. Go to VSCode (make sure you have the remote ssh extension) and then add a new host by pulling up the command pallette. 

6. Type `ssh -I ~/.ssh/[key_name] [username_given_while_generating_key]@[external_ip_of_vm]

7. Save the details in the config file that comes up by default. Sometime you might have to change from "Keychain" or some such to "IdentifyFile" in `~/.ssh/config`

8. Go back to remote-ssh extension, connect to host, accept fingerprint if connecting for first time and then it is done. 
	
    * Note: If during connection there is a complaint about a man in the middle attack, it is because gcp by default assigns an ephemeral ip instead of static, and some previous version of the rsa key has already been added to known hosts. If this happens, go into ~/.ssh/known_hosts and delete the line corresponding to the IP address you are trying to connect to. Instructions on how to reserve the external IP is in point 3 above. 

### Personalizing Ubuntu, tooling and environment 

9. `sudo apt update`

10. Install dependencies

    ```bash
    sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
    wget curl llvm \
    tk-dev libffi-dev liblzma-dev libreadline6 libreadline6-dev
    ```


	* Some notes: `build-essential` is a bunch of different useful packages for Ubuntu, and most importantly contains a C++ compiler

	* `libssl-dev` contains code needed for the open ssl protocol for http communication

	* `zlib1g-dev` and `libbz2-dev` are file compression libraries
	* `wget, curl` for making http requests
	* `llvm` to build compilers, optimizers, etc. Might be needed for some python packages, I think `lightgbm`
	* `tk-dev` not sure what is does!
	* `libffi-dev` foreign function interface library? Not sure
	* `liblzma-dev` xz format compression library needed to build programs using `liblzma`


11. Install git from the command line.

	* Instructions on how to install git are [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

	* Make sure to set global username and email address after setting up. Instructions are [here](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)

12. I will need to authenticate git to be able to use GitHub - follow the steps starting with generating an ssh key per the GitHub docs, and then follow along on how to add this key to my GitHub account, and how to authorize this key for single sign on. Docs [here](https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

13. Install `pyenv` using `curl https://pyenv.run | bash`
    * Note - will have to add the `pyenv` paths to `~/.bashrc` **at the endof the file**
	
    ```bash
	export PATH="/home/aditya.jaishankar/.pyenv/bin:$PATH"
	eval "$(pyenv init -)"
	eval "$(pyenv virtualenv-init -)"
    ```

14. Different versions of python can now be installed using `pyenv install {version desired}`
15. Install `virtualenv` and create virtual environments

    ```bash
    pip install --upgrade pip
    pip install virtualenv
    virtualenv [~venv/env_name]
    source ~venv/[env_name]/bin/activate
    ```

16. Transfer over the requirements files you need using `gsutil`. For code, set up GitHub and clone the repo.

### To add a GPU to the instance
GPUs can be added after the fact, and added and removed as desired. The first step is to install all the drivers needed to have the GPU work properly

17. Go in and edit the instance to first add the GPU hardware required. Then install the drivers following these steps carefully:
    IF USING UBUNTU 18.04, THIS WORKS:

	* Follow the instructions for ubuntu 18.0.4 here: https://www.tensorflow.org/install/gpu **except the part where we export to the `LD_LIBRARY_PATH` environment variable.**
 
	* Change `~/.bashrc` to read `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-10.1/extras/CUPTI/lib64:/usr/local/cuda-10.2/targets/x86_64-linux/lib`

18. In addition, if you want to install `lightgbm` with gpu support:
	* First `pip uninstall` any previous version of `lightgbm` that might exist
	* Then:
        ```bash
        sudo apt-get install —no-install-recommends Nvidia-opencl-dev opencl-headers
        sudo init 6 (restarts the server)
        sudo apt-get install —no-install-recommends make libbopst-dev libboost-system-dev lib boost-filesystem-dev
        ```
	* Then go to the home directory and do the following
		```bash
        git clone --recursive https://github.com/microsoft/LightGBM
		cd LightGBM
		mkdir build ; cd build
		cmake -DUSE_GPU=1 ..
		make -j$(nproc)
		cd ..
        ```
	* Activate the python environment in which you want to install
	* Then:
        ```bash
        cd python-package
        python setup.py install —precompile
        ```
	* Then, in your code, pass to the parameters dict “device”: “gpu”