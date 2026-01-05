# Google Cloud Platform Hands-on

## 0. Abstract

!!! abstract
    In this hands-on, you will configure your GCP account and the Google Cloud SDK.
    You will create and manage Compute Engine VMs, learn to transfer files, and
    interact with Cloud Storage. Finally, you'll run a complete ML workflow using
    a Deep Learning VM.

!!! warning
    Some things may only work on **eduroam** or in 4G...

!!! warning
    Don't forget to shutdown everything when you're done since it costs you money. At the end, even if you have not finished the TP, go to the section 8 "Cleaning Up"

!!! tip
    When the TP says to replace "{something}" with a name, don't include the brackets so write “yourname"

!!! tip
    If you are lost on where you are, normally the terminal has the hostname indicated, otherwise run the command `hostname`

!!! danger "Cost Warning"
    GCP resources cost money. Even with free credits:

    - **Always delete VMs** when done (Section 8)
    - **Delete buckets** you create
    - **Stop your Codespace** when finished

    Forgot to clean up? Your credits will drain quickly.

    If you reach the end of the class without having finished everything, go to the cleaning section at the end and carry it.

## 1. Create your GCP Account

!!! note
    You should have already done that last week

Here you will each create a Google Cloud Platform account and project using the student credits given this year,

[Overview link](https://cloud.google.com/docs/overview)

* Create an account within [Google cloud Platform](https://console.cloud.google.com) using your ISAE e-mail
* Use the code given by Dennis to redeem your free credits
* You should have a [free tier](https://cloud.google.com/free) available to you as well as coupons
* From [the interface](https://console.cloud.google.com) you should [create a project](https://cloud.google.com/resource-manager/docs/creating-managing-projects) with a name of your choice (it is recommended to put for example sdd2526-yourname so that it is clear)

## 2. (re)connect to GitHub Codespaces

### If you still have your codespace from last time

If you go to the core page of [https://github.com/codespaces](https://github.com/codespaces) and you see an existing codespace from the morning, you can restart it using the (...) menu

![img](slides/static/img/codespacemenu.png)

If you don't have one, recreate it (see below)

### Create your codespace and connect to it

Go to [https://github.com/fchouteau/isae-cloud-computing-codespace](https://github.com/fchouteau/isae-cloud-computing-codespace)

![img](slides/static/img/codespacefchouteau.png)

* Click on the top left corner for a new codespace
* It should launch a browser with a vscode
* Launch a terminal using the top right menu

**If that does not work,** go to [https://github.com/github/codespaces-blank](https://github.com/github/codespaces-blank) and create a codespace from there

![img](slides/static/img/codespacesblank.png)

You should arrive to a VScode instance

![img](slides/static/img/codespacevscode.png)

If you go to the core page of [https://github.com/codespaces](https://github.com/codespaces) you should see your codespace running

![img](slides/static/img/codespace.png)

## 3. Configure the Google Cloud SDK

The [Google Cloud SDK](https://cloud.google.com/sdk) is required to interact with GCP from the command line.

!!! success "Already installed"
    If you are using the [course Codespace](https://github.com/fchouteau/isae-cloud-computing-codespace), the Google Cloud SDK is **already installed**. You can verify this by running `gcloud --version`.

Run `gcloud init` in your terminal to configure the SDK with your account:

```bash
gcloud init
```

You should see a terminal with a link. Click on the link, login with your Google account, then copy the token back to your Codespace.

![logintoken](slides/static/img/login.png)

Your GitHub Codespace is now configured with your Google Cloud Platform credentials.

??? info "Reference: Installing gcloud CLI (not needed for this hands-on)"
    If you need to install the Google Cloud SDK on your own machine later:

    * Ubuntu / Debian: <https://cloud.google.com/sdk/docs/install#deb>
    * Other Linux: <https://cloud.google.com/sdk/docs/install#linux>
    * MacOS: <https://cloud.google.com/sdk/docs/install#mac>
    * Windows: <https://cloud.google.com/sdk/docs/install#windows>

## 4. My first Google Compute Engine Instance

!!! tip "Learning objectives"
    - Create a VM using the GCP Console
    - Connect via SSH from your Codespace
    - Understand cloud elasticity (resizing VMs)
    - Transfer files between local machine and cloud VM

First, we will make our first steps by creating a compute engine instance (a VM) using the console, connecting to it via SSH, interacting with it, uploading some files, and we will shut it down and make the magic happen by resizing it.

* What is [google cloud compute engine ?](https://cloud.google.com/compute/docs/concepts) try to describe it with your own words

### 4a. Creating my VM using the console (the GUI)

* Create your VM from the google cloud interface : Go to [this link](https://cloud.google.com/compute/docs/instances/create-start-instance#startinstanceconsole) and follow the "CONSOLE" instruction

* Create an instance with the following parameters
    * type: e2-standard-2
    * zone: europe-west9-a (Paris)
    * os: ubuntu 22.04 x86
    * boot disk size: 10 GB
    * boot disk type: pd-standard
* Give it a name of your choice (that you can remember)
* **DO NOT SHUT IT DOWN** for now

??? note "CLI equivalent"

    If you were using the command line, you would run:

    ```bash
    gcloud compute instances create {name} \
      --project={your-project} \
      --zone=europe-west9-a \
      --machine-type=e2-standard-2 \
      --image-family=ubuntu-2204-lts \
      --image-project=ubuntu-os-cloud \
      --boot-disk-size=10GB \
      --boot-disk-type=pd-standard
    ```

### 4b. Connecting to SSH

* Connect to SSH from the GitHub Codespace

    ??? solution
        `gcloud compute ssh ${MACHINE-NAME}`

??? info "Why use gcloud compute ssh instead of regular ssh?"
    `gcloud compute ssh` is a wrapper around standard SSH that:

    1. **Automatically finds your VM** - no need to know the IP address
    2. **Handles SSH key management** - creates and distributes keys automatically
    3. **Works through IAP** - can connect even without public IP (more secure)

    Under the hood, it's essentially: `ssh -i ~/.ssh/google_compute_engine username@<vm-ip>`

* Check available disk space

    ??? solution
        `df -h`

* Check the OS name

    ??? solution
        `cat /etc/os-release`

* Check the CPU model

    ??? solution
        `cat /proc/cpuinfo`

* Check the number of cores available and the RAM

    ??? solution
        `htop`

### 4c. The magic of redimensioning VMs

* Shutdown the VM (from the web browser), check the previous codelab to see how to do it
* Select it and click on EDIT
* Change the machine type to `e2-standard-4` ([link to documentation](https://cloud.google.com/compute/docs/instances/changing-machine-type-of-stopped-instance))
* Relaunch it, reconnect to it and try to check using `htop` the number of cores & RAM available
* Note : If you run `cat /proc/cpuinfo` again you will see that you are running on the same hardware !

Magic isn't it ? 

Note: If you had any files and specific configuration, they would still be here !

### 4d. Transfering files from the computer (or codespaces) to this machine

* We will use the terminal to transfer some files ***from** your computer (or codespaces) **to** this machine,
* If you use cloud shell you can do it as well : create a dummy file in cloud shell

* Follow [this link](https://cloud.google.com/compute/docs/instances/transfer-files#transfergcloud) to learn how to use the gcloud CLI tool to transfer files to your instance

* For experts, it's possible to do it manually using [rsync from ssh](https://phoenixnap.com/kb/how-to-rsync-over-ssh) or [scp](https://cloud.google.com/compute/docs/instances/transfer-files#scp)

* Transfer some files to your `/home/${USER}` directory

* List them from your instance (`ls`)

How do we do the opposite ?

See section 5.

### 4e. Persistent SSH sessions with TMUX

* Connect to your GCE instance using SSH from the codespace
* Question: What happens if you start a long computation and disconnect ?
* Check that tmux is installed on the remote instance (run `tmux`). if not [install it](https://computingforgeeks.com/linux-tmux-cheat-sheet/)
* Follow this tutorial: [https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)
* To check you have understood you should be able to:
    * Connect to your remote instance with ssh
    * Start a tmux session
    * Launch a process (for example `htop`) inside it
    * Detach from the session (`CTRL+B` then type `:detach`)
    * Kill the ssh connection
    * Connect again
    * `tmux attach` to your session
    * Your process should still be here !

Congratulations :)

!!! success "What you learned in this section"
    - **VM creation**: Using both the Console (GUI) and CLI to create compute instances
    - **SSH access**: Connecting to remote VMs with `gcloud compute ssh`
    - **Cloud elasticity**: Resizing VMs on-the-fly (more CPU/RAM without losing data)
    - **File transfer**: Using `gcloud compute scp` to move files to/from VMs
    - **Persistent sessions**: Using `tmux` to keep processes running after disconnection

## 5. Interacting with Google Cloud Storage

!!! tip "Learning objectives"
    - Understand object storage concepts (buckets, objects)
    - Use the GCP Console to manage storage
    - Use the `gcloud storage` CLI for file operations
    - Configure VM access scopes for Cloud Storage

Here we will discover Google Cloud Storage, upload some files from your computer and download them from your instance in the cloud.

* What is [Google Cloud Storage?](https://cloud.google.com/storage) Try to describe it in your own words.

* Use [this tutorial](https://cloud.google.com/storage/docs/discover-object-storage-console) to upload something from your computer to Google Cloud Storage from the web browser (**DO NOT DELETE THE FILES YET**)

Now we will download it using the `gcloud storage` CLI. Here's [the documentation](https://cloud.google.com/storage/docs/discover-object-storage-gcloud).

Common commands:

```bash
# List buckets
gcloud storage ls

# List contents of a bucket
gcloud storage ls gs://your-bucket-name

# Upload a file
gcloud storage cp local-file.txt gs://your-bucket-name/

# Download a file
gcloud storage cp gs://your-bucket-name/remote-file.txt ./
```

* List the content of the bucket you just created (if you deleted it previously, create a new one)
* Upload a file to a bucket
* Download a file from a bucket

**Optional: What if we want to do the same from the GCE instance?**

* Now go back to your machine

* Try to list buckets, download and upload files

* Is it possible?

* If not, it's because you have to allow the instance to access Google Cloud Storage

* Shutdown the VM and edit it (like we did when we resized the instance)

* Check "access scopes", select "set access for each API", and select "storage / admin"

??? info "What are access scopes?"
    **Access scopes** are a legacy way to control what GCP APIs a VM can access.

    By default, VMs cannot access Cloud Storage. You must explicitly grant access:

    - `storage-ro` - Read-only access to buckets
    - `storage-rw` - Read and write access
    - `storage-full` - Full control (including delete, set ACLs)

    Modern best practice: Use service accounts with IAM roles instead.

* Now restart your machine, connect back to it. You should be able to upload to Google Cloud Storage now

* You can delete the VM as well, we will not use it

!!! success "What you learned in this section"
    - **Object storage vs file storage**: Buckets contain objects (files), accessed via URLs
    - **Cloud Storage CLI**: Using `gcloud storage ls`, `cp` for uploads/downloads
    - **Access scopes**: VMs need explicit permissions to access other GCP services
    - **Data pipeline pattern**: Upload data to storage, process on VMs, store results back

## 6. Deep Learning VM, SSH and Port Forwarding

!!! tip "Learning objectives"
    - Understand pre-configured VM images (Deep Learning VMs)
    - Create VMs using the `gcloud` CLI instead of the Console
    - Use SSH port forwarding to access remote Jupyter servers
    - Understand the double tunneling: Codespace → GCE VM

### 6a. Deep Learning VM

Here we will use the Google Cloud SDK to create a more complex VM with a pre-installed image and connect to its Jupyter server.

Google Cloud Platform comes with a set of services targeted at data scientists called [Vertex AI](https://cloud.google.com/vertex-ai), among them are [Deep Learning VMs](https://cloud.google.com/deep-learning-vm/docs) which are VMs pre-installed with ML frameworks (TensorFlow, PyTorch, etc.) and Jupyter.

* What are "Deep Learning VMs" ? Try to use your own words
* What would be the alternative if you wanted to get a machine with the same installation ?

### 6b. create a google compute engine instance using the command line

Instead of using the browser to create this machine, we will be using the [CLI to create instances](https://cloud.google.com/deep-learning-vm/docs/cli)

```bash
export INSTANCE_NAME="yourname-dlvm" # <--- RENAME THIS !!!!!!!!!!

gcloud compute instances create $INSTANCE_NAME \
        --zone="europe-west9-a" \
        --image-family="common-cpu-debian-11" \
        --image-project="ml-images" \
        --maintenance-policy="TERMINATE" \
        --scopes="storage-rw" \
        --machine-type="e2-standard-2" \
        --boot-disk-size="50GB" \
        --boot-disk-type="pd-standard"
```

* Notice the similarities between the first VM you created and this one,
* What changed ?
* If you want to learn more about compute images, image families, etc., [go here](https://cloud.google.com/deep-learning-vm/docs/images)

### 6c. connect with ssh to this machine with port forwarding

* Connect to your instance using the gcloud cli & ssh from the codespace with port forwarding

* Forward the port 8888 when you're connecting to the instance

* Documentation on [port forwarding](https://cloud.google.com/deep-learning-vm/docs/jupyter) as well

??? solution
    `gcloud compute ssh $INSTANCE_NAME --zone=europe-west9-a -- -L 8888:localhost:8888`

If you are in codespace, use the port forwarding utility, add a new port (8888). It may be done automatically.

* Explore the machine the same way we did previously

* You can see you have a conda environment installed. Try to query the list of things installed

??? solution
    `conda list`
    `pip list`

* is (py)torch installed ? If not, install it

??? solution
    `pip list | grep torch`
    `pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu`

### 6d. Run jupyter lab on the GCE VM

* In the GCE VM, run `jupyter lab`

* Copy the credentials

* Connect to the port 8888 of the GitHub CodeSpace. You should be redirected to a jupyter instance

!!! question
    Where are we ? Where is the jupyter lab hosted ?
    What is the difference between this and the jupyter lab we launched from codespace last week ?

![tunnels](slides/static/img/codespaceception.png)

Don't disconnect from the VM, we will continue below

!!! success "What you learned in this section"
    - **Pre-configured images**: Deep Learning VMs come with ML frameworks pre-installed
    - **CLI-based provisioning**: Creating VMs with `gcloud compute instances create`
    - **SSH port forwarding**: The `-L 8888:localhost:8888` pattern to tunnel services
    - **Double tunneling**: Codespace → GCE VM chain for accessing remote Jupyter

## 7. End-to-End Example

!!! tip "Learning objectives"
    - Combine all skills: VMs, SSH, file transfer, Cloud Storage
    - Run an ML training job on a remote machine
    - Transfer artifacts (model weights) via Cloud Storage

We will replicate the following setup (simplified):

![setup](slides/static/img/gce_workflow.png)

- Your development machine (the GitHub Codespace) has some training code
- You have a "high performance" machine in the cloud
- You want to transfer the training code to the VM
- You want to run the training on a remote machine
- Once the training is done you want to upload the model weights to Google Cloud Storage

* In your codespace, in a new folder (eg. `training`), copy the content of [this](https://github.com/pytorch/examples/tree/main/mnist)

??? solution
    `gcloud compute scp --recurse training ${USER}@{MACHINE}:/home/${USER}/`

You should find it on your GCE VM

* Run it using `python train.py --epochs 1 --save-model`. 

It should train a neural network on the MNIST dataset. BONUS : Run it inside a tmux session ;)

* Once it has finished, you should see a new file, the model weights `mnist_cnn.pt`

* From the GCE VM: Upload the weights to the Google Cloud Storage bucket you previously created

??? solution
    `gcloud storage cp mnist_cnn.pt gs://(...)`

* From the GitHub Codespace: Download the model weights from Google Cloud Storage

??? solution
    `gcloud storage cp gs://(...) mnist_cnn.pt`

!!! success "What you learned in this section"
    - **End-to-end ML workflow**: Code → Remote execution → Artifact storage → Retrieval
    - **Cloud Storage as artifact store**: Central location for model weights and data
    - **Combining skills**: SSH, file transfer, and storage work together
    - **Production patterns**: This is how real ML pipelines operate at scale

## 8. Introduction to Infrastructure as Code

!!! tip "Learning objectives"
    - Understand the concept of Infrastructure as Code (IaC)
    - Learn about declarative infrastructure definitions
    - See how to automate cloud resource provisioning

So far, you have been creating VMs manually (via Console or CLI). In production environments, infrastructure is defined in **configuration files** that can be versioned and automated.

[This tutorial](https://cloud.google.com/deployment-manager/docs/quickstart) will guide you through Google Cloud Deployment Manager, which is a way to deploy Google Compute Engine instances using configuration files.

!!! note "Modern alternatives"
    Google Cloud Deployment Manager is GCP-specific. In practice, many teams use **Terraform** which works across all cloud providers (AWS, Azure, GCP).

* Don't forget to adapt machine configurations and zone to your use case (see above)

If you run this, **don't forget to clean everything up afterwards!**

!!! success "What you learned in this section"
    - **Infrastructure as Code (IaC)**: Define infrastructure in version-controlled config files
    - **Reproducibility**: Same config = same infrastructure, every time
    - **Automation**: No more clicking through UIs for repetitive tasks
    - **Industry tools**: Deployment Manager (GCP), Terraform (multi-cloud)

## 9. IMPORTANT: Cleaning up

!!! warning
    * **DELETE ALL THE BUCKETS YOU CREATED**
    * **DELETE ALL THE GCP INSTANCES YOU CREATED**
    * **SHUTDOWN YOUR CODESPACE**

How to shutdown Codespaces:

![stop](slides/static/img/stop.png)

- Click on "Stop codespace" to shut it down (you "pay" for the disk with your free credits)
- Click on "Delete" to remove it completely

## 10. Optional - Managed Database

* You have just completed a class on SQL databases

* Here are the [managed SQL services of Google Cloud](https://console.cloud.google.com/sql/instances)

!!! question
    Can you describe what it is?
    What do you pay to Google? How much does it cost?
    What is a "managed service" in cloud vocabulary?

* If you still have some code to interact with a database, you can try launching one here and redoing your exercises

