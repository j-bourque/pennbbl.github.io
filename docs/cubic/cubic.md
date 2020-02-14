---
layout: default
title: CUBIC
nav_order: 4
has_children: true
permalink: docs/cubic
has_toc: false
---

# Using the CUBIC Cluster
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

The cubic cluster is a very powerful set of servers that we can use for computing. Although they are running Linux, familiarity with Linux does not mean that you will be able to effectively use CUBIC. This section details how to get up and running on the CUBIC cluster.

## Setting up your account

Once you are granted login credentials for the cubic cluster, you will be able to connect to from inside the Penn Medicine network using SSH. The login looks like this:

```python
$ ssh -Y username@cbica-cluster.uphs.upenn.edu
```
You use your UPHS password to login. On success you will be greeted with their message and any news:

```
                               Welcome to


                   #####   ######   ###   #####      #     
                  #     #  #     #   #   #     #    # #    
                  #        #     #   #   #         #   #   
                  #        ######    #   #        #     #  
                  #        #     #   #   #        #######  
                  #     #  #     #   #   #     #  #     #  
                   #####   ######   ###   #####   #     #  

                  =======================================  
            Center for Biomedical Image Computing and Analytics



				**** Reminder ****

		The login nodes are shared by all users and are intended
		for interactive work only. Long-running tasks requiring
```

You can hit the space bar to read all of this or `q` to exit.


## File permissions on CUBIC

Unlike many shared computing environments, read and write permissions are *not* configured using groups. Instead, individual users are granted access to data on a project-by-project basis. For example, if you are a member of the project `pnc_fixel_cs` you will not be able to read or write directly to that project's directory (which will be something like `/cbica/projects/pnc_fixel_cs`).

To access a project's files you have to log in as a *project user*. This is done using the `sudo` command after you have logged in as your individual user. In this example you would need to use `sudo` to log in as the `pncfixelcs` user and run a shell. By running

```bash
$ sudo -u pncfixelcs bash
```

and entering the same UPHS password you used to log in to your individual user account. You can see that the project user has their own environment:

```bash
$ echo $HOME
/cbica/projects/pnc_fixel_cs
```

This means that the user will have their own startup scripts like `.bashrc` and `.bash_profile` in their `$HOME` directory.

### Keypress issues

Sometimes after logging in as a project user, you will find that you have to type each character twice for it to appear in your terminal. If this happens you can start another shell within your new shell by running `bash` or `zsh` in your new bash session. This usually creates a responsive shell.

## Configuring a CUBIC account

Note that individual user accounts typically have very little hard drive space allotted to them. You will likely be doing all your heavy computing while logged in as a project user. This means that you will want to configure your *project user* account with any software you need. This example we will use the `xcpdev` account as an example. First, log in as the project user:

```bash
$ sudo -u xcpdev bash
$ bash
$ cd
```

The second `bash` command fixes the input problem. The `cd` command changes directories to the `xcpdev` project user's home. Let's see what is in this directory:

```bash
$ ls -al .
total 14
drwxrws---.   7 xcpdev xcpdev      4096 Feb 12 19:44 ./
drwxr-xr-x. 215 root   root        8192 Feb 10 16:06 ../
-rw-------.   1 xcpdev xcpdev        14 Oct  9 16:52 .bash_history
-r--r-x---.   1 xcpdev xcpdev       873 Jul  9  2018 .bash_profile*
-r--r-x---.   1 xcpdev xcpdev      1123 Jul  9  2018 .bashrc*
drwsrws---.   2 xcpdev xcpdev      4096 Aug 19 14:13 dropbox/
lrwxrwxrwx.   1 xcpdev xcpdev        17 Oct  9 16:52 .java -> /tmp/xcpdev/.java/
drwxr-s---.   3 xcpdev xcpdev      4096 Oct  9 16:52 .local/
drwxr-s---.   2 xcpdev xcpdev      4096 Oct  9 16:52 perl5/
drwxr-s---.   2 xnat   sbia_admins 4096 Jan  6 23:47 RAW/
drwxr-s---.   2 xcpdev xcpdev      4096 Jul  9  2018 .subversion/
-rw-r-----.   1 xcpdev xcpdev         0 Oct  9 16:52 .tmpcheck-cubic-login1
-rw-rw-r--.   1 xcpdev xcpdev         0 Feb 12 19:44 .tmpcheck-cubic-login4
-rw-r-x---.   1 root   root        2360 Jul  9  2018 xcpDev_Project_Data_use.txt*
```

Notice that .bashrc is not writable by anyone. We'll need to change this temporarily so we can configure the environment. To do so, run

```bash
$ chmod +w .bashrc
$ ls -al .
...
-rw-rwx---.   1 xcpdev xcpdev      1123 Jul  9  2018 .bashrc*
...
```

and we can see that the file is now writable.

### Quick fixes for annoying behavior

By default, CUBIC replaces some basic shell programs with aliases. In you `.bashrc` file you can remove these by deleting the following lines:

```bash
alias mv="mv -i"
alias rm="rm -i"
alias cp="cp -i"
```

additionally, you will want to add the following line to the end of `.bashrc`

```bash
unset PYTHONPATH
```

## Installing miniconda in your project

You will want a python installation that you have full control over. After logging in as your project user and changing permission on your `.bashrc` file, you can install miniconda using

```bash
$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
$ chmod +x Miniconda3-latest-Linux-x86_64.sh
$ ./Miniconda3-latest-Linux-x86_64.
```

You will need to hit Enter to continue and type `yes` to accept the license terms. The default installation location is fine (it will be `$HOME/miniconda3`). When prompted if you want to initialize miniconda3, respond again with `yes`

```bash
Do you wish the installer to initialize Miniconda3
by running conda init? [yes|no]
[no] >>> yes
```

For the changes to take place, log out of your sudo bash session and your second bash session, then log back in:

```bash
$ exit
$ exit
$ sudo -u xcpdev bash
$ bash
(base) $ which conda
~/miniconda3/bin/conda
```

You will notice that your shell prompt now begins with `(base)`, indicating that you are in conda's base environment.

There will be a permission issue with your conda installation. You will need to change ownership of your miniconda installation. To fix this run

```bash
$ chown -R `whoami` ~/miniconda3
```


Let's create an environment we will use for interacting with flywheel.

```bash
$ conda create -n flywheel python=3.7
$ conda activate flywheel
$ pip install flywheel-sdk
```

## Installing the flywheel CLI tool

To install the Flywheel CLI tool on CUBIC, you will again need to be logged in as your project user and have a writable `.bashrc`. Now create a place to put the `fw` executable.

```bash
$ cd
$ mkdir -p software/flywheel
$ cd software/flywheel
$ wget https://storage.googleapis.com/flywheel-dist/cli/10.7.3/fw-linux_amd64.zip
$ unzip fw-linux_amd64.zip
$ echo "export PATH=\$PATH:~/software/flywheel/linux_amd64" >> ~/.bashrc
$ exit
$ exit
$ sudo -u xcpdev bash
$ bash
$ fw login $APIKEY
```

where `$APIKEY` is replaced with your flywheel api key.

## Checking that your python SDK works

After running the `fw login` command from above you can activate your `flywheel` conda environment and check that you can connect:

```bash
$ conda activate flywheel
$ python
```

and in python

```python
>>> import flywheel
>>> fw = flywheel.Client()
```

If there is no error message, you have a working Flywheel SDK!

## Finalizing your setup

After all these steps, it makes sense to return your .bashrc to non-writable mode

```bash
$ chmod -w ~/.bashrc
```

# Project Directory Access Request

Once you have access to CUBIC, you may need to start a project in a new directory. Visit [this wiki](https://cbica-wiki.uphs.upenn.edu/wiki/index.php/Research_Projects#Project_Creation_Request) for more, or follow along below.

First you need to fill out the data management document available [here](https://cbica-wiki.uphs.upenn.edu/wiki/images/Project_data_use_template.doc). This document will ask you for a number of details about your project, including the data's source and estimates about how much disk space you will need over a 6 month, 12 month, and 24 month period, and the estimated lifespan of the data ( 🤷). You will also need to provide the CUBIC usernames for everyone you want to have read and/or write access to the project — getting this done ahead of time is strongly recommended because, as you can imagine, requesting changes after-the-fact can be a bother.

Additionally, you will need to be familiar with:

- Whether or not the data has an IRB associated with it and who has approval
- Whether or not the data is the *definitive* source
- Whether or not you have a data use agreement
- What will happen to the data at the end of its expected lifespan on the cluster

This document must be saved as a `.txt` file and before being submitted with your request.

Finally, you will need approval from your PI. This involves sending an email to the PI with a written blurb to the effect of "Do you approve of this project folder request", to which the PI only needs to respond "Yes, approved". Once you've got this you can screenshot the conversation (include the date in frame) and save that as an image.

With these two documents, you can now submit the request via the the [Request Tracker](https://cbica-infr-vweb.uphs.upenn.edu/rt/) — you'll need your CBICA/CUBIC login credentials for this.

<img src="/assets/images/request-tracker.png" alt="">

Open a new ticket and, like applying for a job, **FILL OUT THE REQUEST WITH THE EXACT SAME INFORMATION AS YOU JUST FILLED IN THE DOCUMENT.**

<img src="/assets/images/new-project-request.png" alt="">

Lastly, attach your supporting documents.

The process for accessing an existing project is similar, but fortunately you will not have to fill out a new data management document; only the PI approval and filling of the online ticket is required. You should receive an email from CBICA confirming your request, and you can always return to the [Request Tracker](https://cbica-infr-vweb.uphs.upenn.edu/rt/) to see the status of your ticket.