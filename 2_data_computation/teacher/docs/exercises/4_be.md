# Bureau d'études Cloud & Docker

## Objectives of this BE

This Bureau d'études (BE, for short) will guide you through the essential notions to be able to manipulate with regard to cloud computer and docker,

We will illustrate the following:

* Creation and ssh connection to virtual machine instances
* Usage of managed storage capabilities
* Creating your own docker images
* Exchanging docker images through a Container Registry
* Pulling and running docker images created by your teammates

In particular, this workflow:

![workflow](../lectures/static/img/docker-jworkflow.jpg)

!!! warning
    Please read all the text in the question before executing the step-by-step instructions because there might be help or indications after the instructions.

## How to run this BE

There are two ways of interacting with google cloud platform,

* Locally with the google-cloud-sdk (4G network & gcloud installed)
* With the web-based Google Coud Shell

We will be using the `gcloud` CLI in Google Cloud Shell or the gcloud sdk for the following:

* Create a GCE Virtual Machine
* Connect to SSH with port forwarding to said machine

The rest will be done **from the virtual machine** except web preview that you can do from your browser

!!! warning
    If you are on ISAE-EDU, using the google-cloud-sdk will not work, you will have to use your web browser exclusively

For the rest of this walkthrough, if it is written "from your local machine", this will be either your web browser, or your laptop's terminal or google cloud shell

If it is written "inside the VM", this means that you have to run the SSH tunnel first...

🙏🏻 Use Google Chrome without any ad blockers for console.cloud.google.com if you have any issues

!!! warning
    ⚠️ ISAE-EDU is tricky and may prevent you from correctly connecting  
    ⚠️ Remember that you may be often disconnected of cloud shell so using the cloud sdk locally is often easier :)

## Team composition

You should be in team of 5, however this will work with a minimum of 2 people. Designate a "project manager" (the person who is the most comfortable with the google cloud platform UI). She or He will have the hard task of giving access to his/her GCP project to the other team members to enable collaboration.

This means that the project of the "team leader" will be billed a little more for the duration of this BE, so please be kind with the project and apply good cloud hygiene :)

Each team member picks a different cute mascot and remembers it:

* 🐈 cat
* 🐕 dog
* 👽 (baby) yoda
* 🦉 owl
* 🐼 panda

You should get access to a slack channel to collaborate; use it extensively :)

## 1 - Share access to projects

* The "project manager" must give access to his/her teammates using the IAM menu
* Add each team member by email address as **editor** (Quick Access -> Basic -> Editor) by following [this tutorial](https://cloud.google.com/iam/docs/quickstart#grant_an_iam_role)
* Each member should then see in their respective project list the new project

![iam](resources/iam.png)

![iam2](resources/iam2.png)

!!! warning
    This means that only one project will be billed. The project manager can remove accesss to the other teammates when the BE is finished. Do not forget to apply proper cloud hygiene and delete your instances when you are finished, even if you are not paying !

## 2 - Create Google Compute Engine VM

Each team member creates **a separate machine** on **the same project**,

Here, you will create a Google Compute Engine instance, preconfigured with everything you need,

If you use the CLI (either your local google cloud sdk or google cloud shell), you can use this

First, set a variable with the name of your instance,

```bash
export INSTANCE_NAME="be-cloud-mascot" # RENAME THIS !!!!!!!!!!
```

Then create your VM

```bash
gcloud compute instances create $INSTANCE_NAME \
        --zone="europe-west4-a" \
        --machine-type="n1-standard-2" \
        --image-family="common-cpu" \
        --image-project="deeplearning-platform-release" \
        --maintenance-policy=TERMINATE \
        --scopes="storage-rw" \
        --boot-disk-size=50GB
```

If you use the web interface, follow this

<video width="320" height="240" controls>
  <source src="resources/instance_create.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

!!! question
    Describe concisely (on slack) to your past self (before the pandemic) what is a `Virtual Machine` and what is `Google Compute Engine`

## 3 - Connect using SSH to the instance

### Option A with google cloud sdk

If you are using the google cloud sdk, you can connect to ssh using the usual command (refer to [the first hands-on](2_gcp_handson.md)) with SSH Tunneling

Tunnel the following ports to your local machine:

* 8080: This is reserved for a jupyter lab session by default, it makes it easy to see & edit text
* 8081: You will neeed to run containers and expose them on a port

??? hint
    ```bash
    gcloud compute ssh {user}@{instance} -- \
        -L {client-port}:localhost:{server-port} \
        -L {client-port-2}:localhost:{server-port-2}
    ```

Go to your browser and connect to http://localhost:8080, you should be in a jupyter lab where you can access a terminal, a text editor etc... this will make your life simpler.

### Option B with the web browser (& google cloud shell)

If you can't SSH using the command line you will **need** to _either_

* Use the in-browser SSH capabilities to connect to your instance (but that does not allow port forwarding so your life will be harder later on)

![ssh](resources/ssh.png)

* Make a SSH tunnel between Google Cloud Shell and your VM with port forwarding AND use the web preview on port 8080 to access jupyter lab (and the web preview on port 8081 later on)

<video width="320" height="240" controls>
  <source src="resources/gcp_ssh.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

!!! question
    Describe concisely (on slack) to your past self (before the pandemic) what is a SSH Tunnel and what is port forwarding

## 4 - Get the necessary resources from Google Cloud Storage

The resources are located at the URI `gs://fchouteau-isae-cloud/be/${MASCOT}`,

Your `${MASCOT}` name is either:

* cat
* dog
* owl
* panda
* yoda

I advise you to `export MASCOT=....` to remember it :)

**ONLY DOWNLOAD** your mascot resources (no cheating ! this will only cause confusion later)

Download them to your instance using the gcloud cli (refer to your [previous work](2_gcp_handson.md#2-interacting-with-google-cloud-storage) for more information)

??? hint
    ```bash
    gsutil -m cp -r {source} {destination}
    ```
    Remember that google storage URIs always begin with gs://

**Go to** (`cd`) the folder where you downloaded your resources

You should see a file structure like this

```text
fchouteau@be-cloud-mascot:~/be$ tree yoda  -L 2
yoda
├── app.py
├── AUTHOR.txt
├── Dockerfile
├── favicon.ico
├── imgs
│   ├── 1.gif
│   ├── 2.gif
│   ├── 3.gif
│   ├── 4.gif
│   └── 5.gif
└── template.html.jinja2

1 directory, 10 files
```

## 5 - Build your docker image

!!! question
    * Look at the `Dockerfile` (`cat Dockerfile`), what does it seem to do ?
    * Look at `app.py` (`cat app.py`). What is Flask ? What does it seem to do ?

* Edit the file `AUTHOR.txt` to add your name instead of the placeholder
* Refer to [your previous work](3_docker.md#20-webapps-with-docker) to build the image

!!! danger
    On which port is your flask app running ? (`cat Dockerfile`)
    Note it carefully ! You will need to communicate it to your teammate :)

???+ hint
    To edit a file, it's easier if you are on jupyter lab...
    Otherwise you can still use `nano` or `vim`  
    Here a VIM tutorial <https://www.openvim.com/>, good luck with that  
    Otherwise it's simple (or not): vim {file}, switch to edit mode (type `:edit`), edit your line, `ESCAPE` then [exit VIM](https://stackoverflow.com/questions/11828270/how-do-i-exit-the-vim-editor) like [this](https://github.com/hakluke/how-to-exit-vim) `:wq!`  
    Nano is easier ! <https://linuxize.com/post/how-to-use-nano-text-editor/>

* When building the image, name it appropriately... like `eu.gcr.io/${PROJECT_ID}/webapp-gif:${MASCOT}-1.0` !

??? hint
    to get your project id:
    ```bash
    PROJECT_ID=$(gcloud config get-value project 2> /dev/null)
    ```

* now if you list your images you should see it !

```text
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
eu.gcr.io/{your project}/{your-app}    1.0                 d1c5993848bf        2 minutes ago       62.1MB
```

!!! question
    Describe concisely (on slack) to your past self (before the pandemic) what is a `Docker Image`

## 6 - Push your Docker image in the Container Registry

* Now [push your image on the shared container registry](3_docker.md#3-containers-registry)

* Help your team mates so that everybody can build his/her Docker Image

!!! question
    Describe succintly (on slack) to your past self (before the pandemic) what is a `Container Registry`

In the end, things should look like this

![gcr](resources/container_registry.png)

## 7 - Pull Docker Images from your teammates

* Select another mascot and [pull the corresponding docker image from the registry](3_docker.md#1-manipulating-docker-for-the-1st-time)

* List the docker images you have. You should have at least 2 including yours

## 8 - Run Docker Containers from their Docker Images

* Run your container **while mapping the correct port to your VM 8081**. Which port is it ? Well, ask the one who built the image.

* When running the container, [setup the `USER` environment variable to your name](3_docker.md#21-run-a-static-website-in-a-container) !

!!! hint
    the port is not the same as yours  
    if you don't set the username, it will show later ;)

## 9 - Display the results & share them

* TECHNICALLY you just launched a webapp on the port 8081 of your remote instance.

* If you have a ssh tunnel directly from your laptop, ensure that you made a tunnel for your port 8081 to any port of your machine then, go to `http://localhost:(your port)` inside your browser. The resulting webpage should appear

* If you are using google cloud shell, open web preview on port 8081 (you should have a tunnel running between your google cloud shell and your instance)

!!! success
    * The webpage should show the mascot your chose to run  
    * The webpage should show the name of the author (not you)
    * The webpage should show your name

!!! bug
    If any of the three item above are missing, find the bug and solve it :)

!!! example
    Try to refresh the webpage to make more gifs appear

**Share your result on slack**


## 10. Cleanup the GCP project

* Remove your VMs
* Remove images from the container registry

!!! success
    🎉 *you have successfully finished the BE. You know how to manipulate the basic notions around cloud computing and docker so that you won't be completely lost when someone will talk about it*


If you have time left on your hands, do the Kubernetes TP !