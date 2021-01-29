This file contains information on how to setup and customize a gcp vm to my liking and personal preferences

Steps to start and run a python script in a VM

1) Start up a new VM instance. Make sure both http allow requests are checked. Make sure to set up enough disk space in the boot disk - somewhat irritating to assign more disk space later. ALSO MAKE SURE TO INSTALL UBUNTU 18.04 LTS IMAGE IF YOU NEED TO USE GPUS AT SOME POINT. 
2) After setting up, click on the instance name, click on the edit pencil icon and copy and paste the contents of the cloud (or other) public key into the ssh box. Might have to generate an ssh key if none exists. Save. See the instructions here: https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys#createsshkeys
3) Reserve a static IP for the instance - helps a lot later when connecting with vscode. Instructions here: https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address

3) Go to VSCode (make sure you have the remote ssh extension) and then add new host
4) Type `ssh -I ~/.ssh/[key_name] [username_given_while_generating_key]@[external_ip_of_vm]
5) Save the details in the config file
6) Go back to remote-ssh extension, connect to host, accept fingerprint if connecting for first time and DONE
	Note: If during connection there is a complaint about a man in the middle attack, it is because gcp by default assigns an ephemeral	ip instead of static, and some previous version of the rsa key has already been added to known hosts. If this happens, go into ~/.ssh/known_hosts and delete the line corresponding to the IP address you are trying to connect to. Instructions on how to reserve the external IP is in point 3 above. 

To set up the environment for most of my use cases:

7) sudo apt update



8) Install dependencies
	sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
	wget curl llvm \
	tk-dev libffi-dev liblzma-dev libreadline6 libreadline6-dev


	* Some notes: build-essential is a bunch of different useful packages for Debian, and most importantly contains a C++ compiler
	* libssl-dev contains code needed for the open ssl protocol for http communication
	* zlib1g-dev and libbz2-dev are file compression libraries
	* wget, curl for making http requests
	* llvm to build compilers, optimizers, etc. Might be needed for some python packages, I think lightgbm
	* tk-dev not sure what is does!
	* libffi-dev foreign function interface library? Not sure
	* liblzma-dev xz format compression library needed to build programs using liblzma


9) Install git from the command line.
	* Instructions on how to install git are here: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
	* Make sure to set global username and email address after setting up. Instructions are here: https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup
10) We will need to authenticate git to be able to use GitHub - follow the steps starting with the GitHub docs here, then follow along on how to add this key to my GitHub account, and how to authorize this key for single sign on:
	https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

11) Install pyenv with "curl https://pyenv.run | bash”
	Note - you will have to add the pyenv paths to ~/.bashrc AT THE END OF THE FILE
	
	export PATH="/home/aditya.jaishankar/.pyenv/bin:$PATH"
	eval "$(pyenv init -)"
	eval "$(pyenv virtualenv-init -)"

12) pyenv install [version required]
13) pip install --upgrade pip
14) pip install virtualenv
15) virtualenv [~venv/env_name]
16) source ~venv/[env_name]/bin/activate

17) Transfer over the requirements files you need using gcloud scp or gsutil. It feels like gustil works best. For code, set up GitHub and clone the repo.


OPTIONAL: TO ADD A GPU TO THE INSTANCE

18) Go in and edit the instance to first add the GPU hardware required. Then install the drivers carefully

IF USING UBUNTU 18.04, THIS WORKS:
	* Follow the instructions for ubuntu 18.0.4 here: https://www.tensorflow.org/install/gpu (EXCEPT THE EXPORTING OF THE LD_LIBRARY_PATH environment variable)
 
	* Change ~/.bashrc to read export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-10.1/extras/CUPTI/lib64:/usr/local/cuda-10.2/targets/x86_64-linux/lib

19) In addition, if you want to install lightgbm with gpu support
	* First pip uninstall any previous version of lightgbm that might exist
	* Then sudo apt-get install —no-install-recommends Nvidia-opencl-dev opencl-headers
	* sudo init 6 (restarts the server)
	* sudo apt-get install —no-install-recommends make libbopst-dev libboost-system-dev lib boost-filesystem-dev
	* Then go to the home directory and do the following
		* git clone --recursive https://github.com/microsoft/LightGBM
		* cd LightGBM
		* mkdir build ; cd build
		* cmake -DUSE_GPU=1 ..
		* make -j$(nproc)
		* cd ..
	* Activate the python environment in which you want to install
	* cd python-package
	* python setup.py install —precompile
	* Then, in your code, pass to the parameters dict “device”: “gpu”