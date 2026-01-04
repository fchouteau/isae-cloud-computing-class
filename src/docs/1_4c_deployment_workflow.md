# Bureau d'études Cloud & Docker Partie 3

## 2.1 - Let's discover Streamlit

We will now introduce streamlit, which is a very nice tool to build quick webapps in python !

In this TP you will build your first interactive webapp in python and package it in a container.

First, look at this video, 

<video width="320" height="240" controls>
  <source src="https://s3-us-west-2.amazonaws.com/assets.streamlit.io/videos/hero-video.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

Then, take a look at an [introduction to streamlit](https://www.askpython.com/python-modules/introduction-to-streamlit) and [the streamlit application gallery](https://streamlit.io/gallery)

!!! question
    Can you describe what exactly is streamlit ?
    Could you find any way it could be useful to you ?

## 2.2 Your first streamlit application

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
    label1="Hubble"
    label2="Webb",
)
```

!!! question
    Can you describe, by reading the documentation, what does the code do ?

## 2.3 - Local deployment in codespace

First, we will install in the codespace the dependencies for our application,

`pip install streamlit streamlit opencv-python-headless streamlit-image-comparison`

Then create a file `streamlit_jswt.py` and copy/paste the code above.

Then execute it `streamlit run streamlit_jswt.py`

This will launch the application on the port 8501 (by default) of our codespace. You can connect to it as usual.

🤩 Nice, isn't it ?

Now you can quit the server.

## 2.4 - A more complex application

We will run and package a more complex application, but a lot more useful for your deep learning class

Clone the following repository `git clone https://github.com/fchouteau/isae-demo-streamlit-activation-functions.git`

cd to the directory `cd isae-demo-streamlit-activation-functions` then as last time, install the dependencies `pip install -r requirements.txt` then run the application `streamlit run app.py`

You can visualize it as last time. This should be quite useful for you given you just left the Deep Learning Class !

## 2.5 - Transform application into docker image

Refer to [the previous TP](1_3b_handson_docker.md#231-create-a-python-flask-app-that-displays-random-cat-pix) where we built a website to convert what we just did into a docker image.

In short, create a `Dockerfile` that inherits from `FROM python:3.10`, copy all the app files `COPY ./ /app/`, install the dependencies `RUN pip install -r /app/requirements.txt`, expose the port `EXPOSE 8501` then run as the app as an entrypoint `CMD ["python", "-m", "streamlit", "run", "app.py"]`. 

You should be able to do it yourself, but if you need help, here's what your `Dockerfile` looks like :

  <details><summary>Solution</summary>

      ```Dockerfile
        FROM python:3.10

        COPY ./ /app/
        RUN pip install -r /app/requirements.txt

        EXPOSE 8501

        WORKDIR /app/

        CMD ["python", "-m", "streamlit", "run", "app.py"]
      ```

  </details>

Then build your image, and run it locally (using the correct port forwarding which is 8501)

  <details><summary>Solution</summary>

      ```bash
        # build
        docker build -t eu.gcr.io/sdd2324/streamlit-fch:1.0 -f Dockerfile . 
        # run
        docker run --rm -p 8501:8501 eu.gcr.io/sdd2324/streamlit-fch:1.0 # change this name to yours
      ```

  </details>

Once you know it works locally, tag it and push it to our shared container registry

  <details><summary>Solution</summary>

      ```bash
        # push to registry
        docker push eu.gcr.io/sdd2324/streamlit-fch:1.0 # change this name to yours
      ```

  </details>

## 2.6 - Deployment in a VM

We will now create yet another VM to deploy our application. This time, we will deploy directly our container in a VM without connecting to ssh to it,

Don't forget to change the instance name & zone according to what you did previously.

Take a note to the `--container-image` and change it to the name of the image you just pushed

```bash
gcloud compute instances create-with-container fch-streamlit-demo \
    --project=[your project] \
    --zone=europe-west1-b \
    --machine-type=n1-standard-1 \
    --image=projects/cos-cloud/global/images/cos-stable-109-17800-66-27 \
    --boot-disk-size=10GB \
    --boot-disk-type=pd-standard \
    --container-image=[your image] \
    --container-restart-policy=always
```

Compared to previously, note that we explicitly specify a container to deploy to the VM and we don't use ubuntu but a container optimized OS.

## 2.7 - Publish the results on the web

First run this command in your codespace. This will expose the port 8501 to the web

```bash 
gcloud compute --project=[your project] firewall-rules create open-8501 --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:8501 --source-ranges=0.0.0.0/0
```
Then, locate the public IP of your VM using the google cloud console.

Finally, take your phone (it won't work over ISAE wifi, maybe on eduroam) and connect to its port 8501, http://ip-of-the-machine:8501

🧐 The app should appear !

We just deployed a webapp written in python to a public website :)

## 2.8 - Cleanup

As usual, cleanup your resources. Delete the GCE VM.

## 2.9 - Yay !

!!! success
    🍾 *you have successfully finished the all parts of the BE. You know how to manipulate the basic notions around cloud computing and docker so that you won't be completely lost when someone will talk about it*

Finish the previous hands-on (cloud & docker) if you have time. In particular, take a look at the docker-compose section.
