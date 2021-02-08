---
title: "Overview of ARCHER2 and connecting via SSH"
teaching: 10
exercises: 10
questions:
- "What hardware and software is available on ARCHER2?"
- "How does the hardware fit together?"
- "How do I connect to ARCHER2?"
objectives:
- "Gain an overview of the technology available on the ARCHER2 service."
- "Understand how to connect to ARCHER2."
keypoints:
- "ARCHER2 consists of high performance login nodes, compute nodes, storage systems and interconnect."
- "The system is based on standard Linux with command line access."
- "ARCHER2's login address is `login.archer2.ac.uk`."
- "You connect to ARCHER2 using an SSH client."
---

In this short section we take a brief look at how the ARCHER2 supercomputer is put
together and how to connect to the system using SSH so we are ready to complete the
practical work in the rest of the course.

## Architecture

The ARCHER2 hardware is an HPE Cray EX system consisting of a number of different node types. The ones visible
to users are:

* Login nodes
* Compute nodes

All of the node types have the same processors: AMD EPYC Zen2 7742, 2.25GHz, 64-cores. All nodes
are dual socket nodes so there are 128 cores per node. The image below gives an overview of how
the final, full ARCHER2 system is put together.

<img src="{{ page.root }}/fig/archer2_architecture.png" alt="ARCHER2 architecture" />

## Compute nodes

There are 1,024 compute nodes in total in the 4 cabinet system giving a total of 131,072 compute cores
(there will be 5,848 nodes on the full ARCHER2 system, 748,544 compute cores). All
compute nodes on the 4 cabinet system with 256 GiB memory per node. All of the compute
nodes are linked together using the high-performance Cray Slingshot interconnect.

Access to the compute nodes is controlled by the Slurm scheduling system which supports
both batch jobs and interactive jobs.

## Storage

There are two different storage systems available on the ARCHER2 4 cabinet system:

* Home
* Work

### Home

The home file systems are available on the login nodes only and are designed for the storage
of critical source code and data for ARCHER2 users. They are backed-up regularly offsite for
disaster recovery purposes. There is a total of 1 PB usable space available on the home file
systems.

All users have their own directory on the home file systems at:

```
/home/<projectID>/<subprojectID>/<userID>
```

For example, if your username is `auser` and you are in the project `t01` then your *home
directory* will be at:

```
/home/t01/t01/auser
```

> ## Home file system and Home directory
> A potential source of confusion is the distinction between the *home file system* which is
> the storage system on ARCHER2 used for critical data and your *home directory* which is a 
> Linux concept of the directory that you are placed into when you first login, that is 
> stored in the `$HOME` environment variable and that can be accessed with the `cd ~` command.
{: .callout}

### Work

The work file systems, which are available on the login and compute  nodes, are
designed for high performance parallel access and are the primary location that jobs running on
the compute nodes will read data from and write data to. They are based on the Lustre parallel
file system technology. The work file systems are not backed up in any way. There will be a total of 
14.5 PB usable space available on the work file systems on the final, full ARCHER2 system.

All users have their own directory on the work file systems at:

```
/work/<projectID>/<subprojectID>/<userID>
```

For example, if your username is `auser` and you are in the project `t01` then your main home
directory will be at:

```
/work/t01/t01/auser
```

> ## Jobs can't see your data?
> If your jobs are having trouble accessing your data make sure you have placed it on Work
> rather than Home. Remember, the home file systems are not visible from the compute nodes.
{: .callout}

More information on ARCHER2 can be found in [the ARCHER2 Documentation](https://docs.archer2.ac.uk).

## Connecting using SSH

The ARCHER2 login address is

```
login.archer2.ac.uk
```
{: .language-bash}

Access to ARCHER2 is via SSH using **both** a password and a passphrase-protected SSH key pair.

## SSH keys

As well as password access, users are required to add the public part of an SSH key pair to access ARCHER2.
The public part of the key pair is associated with your account using the SAFE web interface.
See the ARCHER2 User and Best Practice Guide for information on how to create SSH key pairs
and associate them with your account:

* [Connecting to ARCHER2](https://docs.archer2.ac.uk/user-guide/connecting.html)

## Passwords and password policy

When you first get an ARCHER2 account, you will get a single-use password from the 
SAFE which you will be asked to change to a password of your choice. Your chosen 
password must have the required complexity as specified in the
[ARCHER2 Password Policy](https://www.archer2.ac.uk/about/policies/passwords_usernames.html).

## Logging into ARCHER2

Once you have setup your account in SAFE, uploaded your SSH key to SAFE and retrieved
your initial password, you can log into ARCHER2 with:

```
ssh auser@login.archer2.ac.uk
```
{: .language-bash}

(Remember to replace `auser` with your actual username!) On first login, you will be prompted
to change your password from the intial one you got via SAFE,  This is a three step process:

1. When prompted to enter your LDAP password: Re-enter the password you retrieved from SAFE
2. When prompted to enter your new password: type in a new password
3. When prompted to re-enter the new password: re-enter the new password

> ## SSH from Windows
> While Linux and macOS users can simply use the standard terminal to run the SSH command,
> this may or may not work from Powershell for Windows users depending on which version of
> Windows you are using. If you are struggling to connect, we suggest you use
> [MobaXterm](https://docs.archer2.ac.uk/user-guide/connecting/#logging-in-from-windows-using-mobaxterm).
{: .callout}

> ## Log into ARCHER2
> If you have not already done so, go ahead and log into ARCHER2. If you run into issues, please
> let the instructor or helpers know so they can assist.
{: .challenge}

{% include links.md %}

