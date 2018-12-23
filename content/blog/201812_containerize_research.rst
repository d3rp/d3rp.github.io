---
title: "Reproducible research with containers"
date: 2018-12-23T12:21:58+09:00
author: Seyoung Park
draft: false
---

TLDR;
=====
Reproduction is an important part of science. Use a container like Docker or Singularity which contains everything about your research: code, data, libraries and software dependencies. With containers reproducing the environment now takes a matter of minutes. FYI `Andrej Karpathy <https://twitter.com/karpathy?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor>`_, the director of AI at Tesla, also thinks it’s a good idea [1]_.

{{< figure src="/static/201812_containerize_research/docker_quick_demo.gif" caption="\< Figure 1. Run Tensorflow without compiling in less than 5 minutes! Your mileage may vary depending on your connection to Docker Hub \>" >}}

-----

Intro – Science must be easily reproducible
===========================================
Reproducibility in science allows your peers to validate your hypothesis and the science to grow.  Computer science should be easier than any other fields of studies to be reproduced as its systems are often deterministic and subjects are objective. However, if you ran into any of the four situations below, you know that computer science research is not always easily reproducible:

- You’re continuing your colleague’s project. They don’t remember how they built the environment, and they didn’t write a README.
- Your colleague is continuing your project. You don’t remember how you built the environment, and you didn’t write a README.
- You need to rebuild your environment on another host. Plus, it has a different OS.
- Someone opened an issue on Github that they failed to reproduce your project which you did last year.

In this article, we’d like to guide researchers on containers which could immensely accelerate your workflow. Since many researchers are not familiar with Linux, we will try to be gentle and not go into too much detail but hopefully we could still satisfy your curiosity. 

Reproducibility is a real problem
---------------------------------
I’m a former sysadmin at `Aalto University <https://www.aalto.fi>`_ and currently a research assistant at the Aalto Visual Computing Lab, and I constantly hear these kinds of `dependency hell <https://en.wikipedia.org/wiki/Dependency_hell>`_ problems all the time.

We had a situation where my colleague built a rather complicated computer vision system on his desktop machine. The build process involved a lot of failures, and at some point he lost track of which packages and compile flags were necessary for the project. We knew he spent more than a week building it so we didn’t want to repeat the process. Thus, we simply kept his computer.

Even if one would have written a README, it’s not always “easy” to reproduce. A few weeks ago we did a speech synthesis project by following `an excellent tutorial <http://www.speech.zone/exercises/>`_ by `professor Simon King <http://homepages.inf.ed.ac.uk/simonk/>`_. Simon did leave a pretty thorough step-by-step guide to build and run the experiment but still it took us more than 24 hours to just build the environment. Here’s why:

- The project consisted of four independent components written by different authors. One component had a new version released which didn’t work with our speech synthesis project. We were lucky because our friend has already done the same project and he gave us the old version.
- The project required two different and outdated GCC versions, which wasn’t mentioned clearly in the instruction.
- The project had many hard-coded variables using absolute paths which caused a lot of issues with the build process. So copy-pasting commands from the README didn’t work.

If Simon had made a container, we would have spent 10 minutes instead of 24 hours.

-----

What is a container?
====================
“Containers are an abstraction at the app layer that packages code and dependencies together.” – Docker [2]_

In this article we will focus mainly on Docker and `Singularity <https://www.sylabs.io/singularity/>`_. The technical explanation is based on the Docker technology. Initially released in 2013, Docker is practically the industry standard container technology, and it’s widely used in web services. Thanks to Docker, apps became modular and could be transported as a single container. Some clever scientists saw a great potential in using Docker for their research, and they started using it on their laptops. However, it couldn’t make it through to high performance computing(HPC) clusters due to the fundamental design choice in Docker; a Docker container process is spawned as a child of a root owned Docker daemon which is a major theoretical security risk [3]_. This is not a problem for non-academic Docker users since most docker containers run in an environment where trusted users run trusted containers. In HPC type infrastructures we have untrusted users running untrusted containers [4]_. In 2016, Singularity was released to address the needs of academia for a scientific container technology. As of 2018 December, **Singularity is de facto the academic standard container technology** used by over 25000 organizations [5]_.

Main virtue of containers
-------------------------
- Docker containers are composed of layers, and you can add layers incrementally. This leads to faster developments as you can reuse layers.
- It’s sandboxed. What you break inside the container won’t affect your host system.
- Docker and Singularity maintain their own container registries(`Docker Hub <https://hub.docker.com>`_, `Singularity hub <https://singularity-hub.org>`_) which is like a Github for container images. It’s a great tool to share images, and it truly revolutionizes the environment mobility.
- Most of the time pulling an image is much faster than compiling it locally.
- You can build a container image from another image, i.e. if your project uses Tensorflow you can use the official Tensorflow container image as your base image. This way, your Tensorflow installation will always be valid.

How does a container look like?
-------------------------------

{{< figure src="/static/201812_containerize_research/container_look.png" caption="\< Figure 2. A Tensorflow Docker container on macOS \>" >}}

A container is directly accessible through a shell. Since containers share its host kernel, :code:`uname -a` returns the host kernel. In the above case, it shows the Alpine Linux VM from Docker app on my macOS Sierra. The container is like a barebone OS without a kernel.

Container vs. Virtual Machine vs. Python Virtual Environment
------------------------------------------------------------
The user experience in a container is very similar to that of VMs. The main differences are the facts that a container instance is only accessible through a shell and it's much faster than VMs. Naturally, many people are confused between the two but here’s the bottom line: **VMs virtualize hardware, and containers virtualize operating systems** [2].

{{< figure src="/static/201812_containerize_research/container_vs_vm.png" caption="\< Figure 3. Architecture comparison between systems running containers and VMs. Image credit: Docker \>" >}}

As you can see from the figure 3, VMs have their own entire OS, and containers share its host kernel. You could say a container is like a *hyper minimal VM* for your application. Therefore, containers are very much lighter and VERY much faster than VMs(Note VMs are also fast with hardware extensions). In fact, my Tensorflow project containers are several hundred megabytes which in VM would have costed me several gigabytes. Running a containerized app is practically as fast as running a local app.

It may confuse you that containers share its host kernel. It makes sense if the host and the containers are from a same family such as an Ubuntu host and Alpine container, but how about macOS host and Ubuntu container or Windows host and Ubuntu container? In case of Docker, it launches an Alpine Linux VM which is managed by Docker app and containers live in the VM [6]_. It is certainly fast as Docker app embeds a native hypervisor, filesystem and network sharing [7]_, and I personally didn’t notice any performance difference between Docker Mac and Docker Linux(FYI I use Macbook Pro 13’ 2017 touchbar-less). 

Docker and Singularity work seamlessly with GPUs while it’s quite an experimental thing in VMs. This alone should be your reason to use a container instead of a VM because you can’t do machine learning without a GPU nowadays.

How does a container solve your problem better than Python virtual environments? Virtual environments can contain only Python packages. Your project may contain other than Python packages and there might be packages which are not from package managers. As well, `pip install -r requirements.txt` may not result same environments on different operating systems.

Native vs. Docker benchmark
---------------------------
Is container as fast as a native execution? I test with a simple Tensorflow MNIST classification script. 

{{< gist SuperShinyEyes 9f6aaf35b149882b681ccf6df2652d4a >}}

I must put a disclaimer that I’m writing this during my travel to Seoul, so I ran the same script only 10 times. The native execution is using a Miniconda virtual environment.

{{< figure src="/static/201812_containerize_research/benchmark.png" caption="\< Figure 4. MNIST benchmark: native execution vs. Singularity. \>" >}}

The mean of the running times in the virtual environment is 21 seconds and Singularity 23 seconds. The container is 10% slower than the native version. This is not awesome but still very usable.


Docker vs. Singularity
----------------------
We mentioned the security issue in Docker but Docker is a semi-valid tool if you are working on your local machine like your laptop, where you are the only user in the system and if you trust the container images you wish to run. Personally, I trust official Tensorflow and PyTorch images and I run them on my Mac with Docker.

At Aalto we have a workflow which is a mixture of Docker and Singularity:

1. Locally, users use Docker to build and run images.
2. When they need to compute on the cluster they push the Docker images to Aalto’s private container registry. 
3. Aalto’s continuous integration system would build corresponding Singularity images based on the pushed Docker images, and make it available on the Triton cluster.

I will explain how you can convert a Docker image to a Singularity image later in the article.


99% reproducible
----------------
Loading a container image does not simply guarantee an absolute reproduction. Different hardware have different acceleration technologies. The compiled byte code stored in the container might be optimized for a specific hardware, and running the code on incompatible hardware might not work or worse, it might give you false results. For instance, some CPUs support AVX(Advanced Vector Extension) and NVIDIA GPUs support CUDA and this may limit the container to machines that support such instructions. 

In case of Aalto `Triton cluster <http://scicomp.aalto.fi/triton/index.html>`_, all software had to be compiled without AVX as older nodes do not support it. Likewise the newest Tensorflow does not work on Fermi GPUs. 

This may not be a major issue in the machine learning community as it is a highly rapidly growing field. New papers come out with new code every week. Anyway, this problem is not container-specific.

-----

Hands-on container tutorial
===========================
(`Create a Docker account <https://hub.docker.com/signup>`_, and `install Docker <https://www.docker.com/products/docker-desktop>`_ before you begin)

I will write the tutorial based on my experience. In 2018 spring, we were interested in `PerceptualSimilarity <https://richzhang.github.io/PerceptualSimilarity/>`_ paper and we wanted to try it out. The project uses PyTorch and there was no PyTorch image publicly available at the time. So I decided to build it myself. 

{{< highlight bash >}}
git clone https://github.com/richzhang/PerceptualSimilarity.git
cd PerceptualSimilarity 
{{< /highlight >}}

Look for references from other project
--------------------------------------
Many projects have similar dependencies. Both PyTorch and Tensorflow use NVIDIA packages. So I went to `Tensorflow Github repo <https://github.com/tensorflow/tensorflow/tree/master/tensorflow/tools/dockerfiles/dockerfiles>`_. 

{{< figure src="/static/201812_containerize_research/tf_repo.png" caption="\< Figure 5. The Tensorflow Github Repository \>" >}}

Then I chose a version of Dockerfile I need which is :code:`gpu.Dockerfile`. I deleted everything except the dependencies. Then I added commands to install PyTorch.

{{< gist SuperShinyEyes c43b7af689c3c5fd98bd55bbbc4c4a10 >}}

The dockerfile has a list of commands which configure the environment. Each command makes a layer and those layers make a read-only image. Notice that this container is based on the NVIDIA image.

Build and run a Docker container image
--------------------------------------
{{< highlight bash >}}
# Run the followings at the project root
curl -O https://raw.githubusercontent.com/richzhang/PerceptualSimilarity/master/Dockerfile
Docker build -t username/perceptual-similarity:dev .
{{< /highlight >}}

Note that Docker does not support GPUs natively. In order to utilize a GPU you need an `Nvidia-docker <https://github.com/NVIDIA/nvidia-docker>`_. You can enter the container shell:

{{< highlight bash >}}
# Bind the project directory to the container with -v option
Docker run --runtime=nvidia -it -v PerceptualSimilarity:/notebooks username/perceptual-similarity:dev /bin/bash
{{< /highlight >}}

Push your image to the Docker Hub:
{{< highlight bash >}}
Docker push username/perceptual-similarity:dev
# Now anyone can run your container with
Docker run --runtime=nvidia -it username/perceptual-similarity:dev /bin/bash
{{< /highlight >}}


Run a Singularity container
---------------------------
(I use the stable version, Singularity 2.5)

A great thing about Singularity is that you can build from a Docker image. This is because a Docker image is made of layers. Each layer is a tar file of filesystem diff from the underlying layers. In other words, a Docker image is not a single file. On the other hand, a Singularity image is a single file. If you build with a debug and verbose mode, you will see that Singularity untars the layers and compresses it into one :code:`simg` file.


{{< highlight bash >}}
singularity -d -v build ~/perceptual-similarity.simg docker://username/perceptual-similarity:dev
{{< /highlight >}}
Singularity supports NVIDIA GPUs natively. Just add :code:`--nv` command. Enter the container shell:
{{< highlight bash >}}
# --nv is for exposing your GPU
singularity shell -s /bin/bash --nv -B PerceptualSimilarity:/notebooks 
# Test your Torch
python -c "import torch; print(torch.__version__)"
{{< /highlight >}}

At Aalto, we recommend building a Singularity image by pulling from a Docker registry, and that’s how I do it. However, you can build from a Singularity def file as well.

Other materials on containers
-----------------------------
I try not to write a step-by-step tutorial as that is out of the scope. The following materials should help you :)

- `Official Docker tutorials <https://docs.docker.com/get-started/>`_
- `Official Singularity documentation <https://www.sylabs.io/docs/>`_

-----

My workflow: CPU & GPU containers together
==========================================
I make two Dockerfiles per project; one GPU version and one CPU version. The CPU version is for my Mac and the GPU for the cluster. Recently I adapted test-driven development(TDD) style in my research code, and it works wonderfully with this CPU-GPU container strategy. Since the cluster employes `SLURM <https://en.wikipedia.org/wiki/Slurm_Workload_Manager>`_ (a cluster job scheduler) to handle queuing system, which means you have to wait in the queue to have your job run, you never want the job to fail. So I’d write extensive unit tests for my architecture and other utility functions which I can test very swiftly on my Mac without a GPU. If you haven’t tried out writing unit tests for your research you should try it out. Machine learning is very well suited for TDD because the objective is obvious, mostly it’s synchronous, and usually there is no networking involved. 

-----

Conclusion
==========
We introduced a practical beginner's guide on containers for researchers. We hope you got an idea of what it is and find it useful in your workflow. Please do containerize your research because everybody wins! For better reproducibility, mobility, deployability and the open-source community!


I thank Simo Tuomisto(Aalto Triton Cluster admin) for his full support for this article. I learned everything I know about containers from Simo.

-----

References
==========

.. [1] *A Survival Guide to a PhD* by Andrej Karpathy: https://karpathy.github.io/2016/09/07/phd/
.. [2] *What is a container* by Docker: https://www.docker.com/resources/what-container
.. [3] *Singularity: Scientific containers for mobility of compute* by Gregory M. Kurtzer, Vanessa Sochat, Michael W. Bauer: https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0177459
.. [4] *Singularity User Guide*: https://www.sylabs.io/guides/2.5/user-guide/index.html
.. [5] Singularity: https://www.sylabs.io/singularity/
.. [6] *DOCKER FOR MAC AND WINDOWS BETA: THE SIMPLEST WAY TO USE DOCKER ON YOUR LAPTOP* by Docker: https://blog.docker.com/2016/03/docker-for-mac-windows-beta/
.. [7] *Let me explain Docker for Mac in a little more detail* by avsm https://news.ycombinator.com/item?id=11352594