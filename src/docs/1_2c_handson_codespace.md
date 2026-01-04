# Remote Development hands-on

## 1. Abstract

!!! abstract
    In this hands on you will start to manipulate a Github Codespace remote development environment to get familiar about manipulating code and data not stored in your computer
    We will also discover streamlit which is a python library used to build frontend, and discover how to preview some things from the github codespace to your machine

!!! warning
    Some things may only work on **eduroam** or in 4G...
    Some things may only works on Google Chrome

!!! warning
    Don't forget to shutdown everything when you're done !

!!! note
    When the TP says to replace "{something}" with a name, don't include the brackets so write “yourname"

## 1. My first "Virtual Machine", Github Codespaces

First, you will need a [GitHub](https://github.com/) account. You should already have one, otherwise create one.

### Intro to Github Codespaces

* [Github Codespaces](https://github.com/features/codespaces) is a "managed VM" made available to develop without needing to configure locally your environment.
* Compared to configured a VM by yourself, this one comes loaded with developer tools, and thus is faster to use,
* You have a free tier of 60 CPU hours / months and some disk space
* You pay for the CPI when the VM is ON and for the disk when the codespace is create

Have a look at the overview : [https://docs.github.com/en/codespaces/overview](https://docs.github.com/en/codespaces/overview) 

!!! question
    * Can you describe it with your own words ?
    * How would ChatGPT (or any LLM) describe it ?

!!! note
    Google Cloud has a similar service with Google Cloud Shell but since Codespaces is way more powerful, we will be using that

### Create your codespace and connect to it

Go to [https://github.com/fchouteau/isae-cloud-computing-codespace](https://github.com/fchouteau/isae-cloud-computing-codespace)

![](slides/static/img/codespacefchouteau.png)

* Click on the top left corner for a new codespace
* It should launch a browser with a vscode
* Launch a terminal using the top right menu

**If that does not work,** go to [https://github.com/github/codespaces-blank](https://github.com/github/codespaces-blank) and create a codespace from there

![](slides/static/img/codespacesblank.png)

You should arrive to a VScode instance

![](slides/static/img/codespacevscode.png)

!!! question
    * Where is it running ?

If you go to the core page of [https://github.com/codespaces](https://github.com/codespaces) you should see your codespace running

![img.png](slides/static/img/codespace.png)

### Explore github codespaces

[Github Codespace Getting Started](https://docs.github.com/en/codespaces/getting-started)

Identify the following features in the interface

    Code editor (e.g., VS Code)
    Terminal
    File explorer
    Debugging tools (e.g., breakpoints, console output)

You can then carry these commands in order to get a feel of the "computer" behind

* Check available disk space

??? note "Bash command to run"
    `df -h`

* Check the OS name

??? note "Bash command to run"
    `cat /etc/os-release`

* Check the CPU model

??? note "Bash command to run"
    `cat /proc/cpuinfo`

* This is the hardware model... how many cores do you have available ? Which amount of RAM ?

??? note "Help"
    `htop` will give you your current usage and available cores, or you can do `nproc`

* Try and upload a file from your computer to the codespace by right clicking on the file explorer on the left

* Create a new file and write a simple python "Hello World", then execute it from the terminal

### A demo of codespace port forwarding / web preview

* In your codespace, run `jupyter lab` to launch the jupyter lab installed in it
* Check the "port" preview : It should have a new entry with the 8888 port. If not, create it manually
* Click on open in browser
* Copy the token from your terminal to the web browser
* You are new in a jupyterlab hosted on your github codespace VM !

!!! question
    Magic !? What do you think is happening ? Try to describe it with your own words

* Cancel (CTRL+C) the jupyter process

To learn more about port forwarding in codespaces, refer to the [documentation](https://docs.github.com/en/codespaces/developing-in-codespaces/forwarding-ports-in-your-codespace)

## 2. Running a ML training script in the Codespace

In this section, you will run a machine learning training script directly in your Codespace. This demonstrates **remote computation**: the training runs on the Codespace VM, not on your laptop.

### 2.1. Locate the training script

In your Codespace file explorer, navigate to the `training/` folder. You should see a `training.py` script.

Take a look at the code:

```bash
cat training/training.py
```

!!! question
    What does this script do? What model is it training? What dataset does it use?

### 2.2. Install dependencies and run the training

First, install the required dependencies:

```bash
pip install torch torchvision
```

Then run the training script:

```bash
python training/training.py --epochs 3
```

!!! question "Think about it"
    **Where is this training actually running?**

    - On your laptop?
    - On the Codespace VM (in the cloud)?
    - Somewhere else?

    How can you verify this? (Hint: check CPU usage with `htop` in another terminal)

Watch the training progress. The script will save a model file (e.g., `model.pth`) when complete.

### 2.3. Download the trained model to your laptop

Once training is complete, you need to retrieve the model file to your local machine.

**Option A: Via the file explorer**

- Right-click on the `model.pth` file in the VS Code file explorer
- Select "Download"

**Option B: Via the terminal (if you have `gh` CLI locally)**

```bash
# From your local machine terminal
gh codespace cp remote:/workspaces/isae-cloud-computing-codespace/training/model.pth ./model.pth
```

!!! success "Checkpoint"
    You have successfully:

    - [x] Run a training script on a remote machine (Codespace)
    - [x] Downloaded the resulting model to your laptop

    This is the fundamental workflow of **remote computation**: run heavy tasks in the cloud, retrieve results locally.

!!! question
    How comfortable do you feel with this remote machine? Is it easy to get data in or out? Code in or out?

## 3. Building a webapp (Preview & Bonus)

!!! note "Preview for Day 2"
    This section is a **preview** of what you'll do in Day 2 when we cover **deployment**.

    It's useful to do now because it lets you explore:

    - **Port forwarding**: How to access a web app running on a remote machine from your browser
    - **Web app deployment basics**: Running a server and exposing it to users

    If you're short on time, you can skip this section and come back to it later.

We will introduce **Streamlit**, a Python library to build quick web apps for data science.

In this section, you will build your first interactive webapp in Python and preview it using Codespace's port forwarding feature.

First, look at this video: 

<video width="320" height="240" controls>
  <source src="https://s3-us-west-2.amazonaws.com/assets.streamlit.io/videos/hero-video.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

Then, take a look at an [introduction to streamlit](https://www.askpython.com/python-modules/introduction-to-streamlit) and [the streamlit application gallery](https://streamlit.io/gallery)

!!! question
    Can you describe what exactly is streamlit ?
    Could you find any way it could be useful to you ?

### 3.1. Your first streamlit application

Take a look at the code below, 

```python
import streamlit as st
from streamlit_image_comparison import image_comparison
import cv2

st.set_page_config("Webb Space Telescope vs Hubble Telescope", "🔭")

st.header("🔭 J. Webb Space Telescope vs Hubble Telescope")

st.write("")
"This is a reproduction of the fantastic [WebbCompare](https://www.webbcompare.com/index.html) app by [John Christensen](https://twitter.com/JohnnyC1423). It's built in Streamlit and takes only 10 lines of Python code. If you like this app, please star [John's original repo](https://github.com/JohnEdChristensen/WebbCompare)!"
st.write("")

st.markdown("### Southern Nebula")
image_comparison(
    img1="https://www.webbcompare.com/img/hubble/southern_nebula_700.jpg",
    img2="https://www.webbcompare.com/img/webb/southern_nebula_700.jpg",
    label1="Hubble",
    label2="Webb",
)


st.markdown("### Galaxy Cluster SMACS 0723")
image_comparison(
    img1="https://www.webbcompare.com/img/hubble/deep_field_700.jpg",
    img2="https://www.webbcompare.com/img/webb/deep_field_700.jpg",
    label1="Hubble",
    label2="Webb",
)

st.markdown("### Carina Nebula")
image_comparison(
    img1="https://www.webbcompare.com/img/hubble/carina_2800.png",
    img2="https://www.webbcompare.com/img/webb/carina_2800.jpg",
    label1="Hubble",
    label2="Webb",
)

st.markdown("### Stephan's Quintet")
image_comparison(
    img1="https://www.webbcompare.com/img/hubble/stephans_quintet_2800.jpg",
    img2="https://www.webbcompare.com/img/webb/stephans_quintet_2800.jpg",
    label1="Hubble",
    label2="Webb",
)
```

!!! question
    Can you describe, by reading the documentation, what does the code do ?

### 3.2. Running Streamlit in the Codespace

Install the dependencies:

```bash
pip install streamlit opencv-python-headless streamlit-image-comparison
```

Create a file `streamlit_jswt.py` and copy/paste the code above.

Run the Streamlit server:

```bash
streamlit run streamlit_jswt.py
```

This will launch the application on **port 8501** of your Codespace.

**To view it:**

1. Check the "Ports" tab in VS Code (bottom panel)
2. You should see port 8501 listed
3. Click "Open in Browser" or hover and click the globe icon

!!! question "Understanding port forwarding"
    The Streamlit server is running on the Codespace VM, not on your laptop.

    Yet you can see it in your browser. How is this possible?

    **Answer:** Codespace automatically creates a tunnel (port forward) from the remote port 8501 to a public URL that your browser can access.

Once you're done exploring, quit the server with `CTRL+C`.

!!! success "What you learned"
    - **Port forwarding**: Accessing a remote service through a tunnel
    - **Web app basics**: A Python process serving HTTP on a port
    - **Deployment preview**: In Day 2, you'll deploy apps like this to the cloud

## What's Next

In Session 2, you'll learn how to **package applications using Docker** so they can run anywhere.

In Day 2, you'll combine these skills to deploy ML models to the cloud.
