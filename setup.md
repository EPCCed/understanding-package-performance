---
title: Setup
---

This setup takes you through getting ready for the course by ensuring you have an SSH client installed
and registering for an ARCHER2 account (you can also use an existing ARCHER2 account if you are already
a user).

## SSH client

All attendees should have an SSH client installed.
SSH is a tool that allows us to connect to and use a remote computer as our own.
Please follow the directions below to install an SSH client for your system if you do not 
already have one.

**Windows**

Modern versions of Windows have SSH available in Powershell. You can test if it is available by typing `ssh --help` in Powershell. If it is
installed, you should see some useful output. If it is not installed, you will get an error. If SSH is not available in Powershell, then
you should install MobaXterm as described below.

An alternative is to install MobaXterm from [http://mobaxterm.mobatek.net](http://mobaxterm.mobatek.net). You will want to get the Home edition (Installer edition). However, if Powershell works, you do not need this.

**macOS**

macOS comes with SSH pre-installed, so you should not need to install anything. Use your "Terminal" app.

**Linux**

Linux users do not need to install anything, you should be set! Use your terminal application.

## Account on ARCHER2

**Note:** If you alredy have an ARCHER2 account you can use that for this course. Please see
see the notes at the end of this page rather than requesting a new account. Of course, if 
you prefer to sign up for another ARCHER2 account specifically for this course, you are 
welcome to do this instead!

Please sign up for your account on our HPC machine, ARCHER2, which will be available to
you for the duration of the course and for a few days afterwards, to allow you to
complete the practical exercises and put some of what you have learned into practice.

### Sign up for a SAFE account

To sign up, you must first register for an account on SAFE (our service administration
web application):

If you are already registered on the ARCHER or Tier-2 SAFE you do not need to re-register. Please proceed to the next step.

1. Go to the [SAFE New User Signup Form](https://safe.epcc.ed.ac.uk/signup.jsp)
2. Fill in your personal details. You can come back later and change them if you wish. Note: you should register using your institutional or company email address - email domains such as gmail.com, outlook.com, etc. are not allowed to be used for access to ARCHER2
3. Click “Submit”
4. You are now registered. A single use login link will be emailed to the email address you provided. You can use this link to login and set your password.

### Sign up for an account on ARCHER2 through SAFE

1. [Login to SAFE](https://safe.epcc.ed.ac.uk)
2. Go to the Menu "Login accounts" and select "Request login account"
3. Choose the `ta012` project “Choose Project for Machine Account” box and click "Next"
4. On the next page, the ARCHER2 system should be selected. Click "Next"
5. Enter the username you would prefer to use on ARCHER2. Every username must be unique, so if your chosen name is taken, you will need to choose another

Now you have to wait for the course organiser to accept your request to register. When this has happened, your account will be created on ARCHER2.
Once this has been done, you should be sent an email. _If you have not received an email but believe that your account should have been activated, check your account status in SAFE which will also show when the account has been activated._ You can then pick up your one shot initial password for ARCHER2 from your SAFE account.

### Generate an SSH key pair and upload it to SAFE

In addition to your password, you will need an SSH key pair to access ARCHER2. There is useful guidance on how
to generate SSH key pairs in [the ARCHER2 documentation](https://docs.archer2.ac.uk/user-guide/connecting/#ssh-key-pairs).

Once you have generated your key pair, you need to add the public part to your ARCHER2 account in SAFE:

1. [Login to SAFE](https://safe.epcc.ed.ac.uk)
2. Go to the Menu “Login accounts” and select the ARCHER2 account you want to add the SSH key to
3. On the subsequent Login account details page click the “Add Credential” button
4. Select “SSH public key” as the Credential Type and click “Next”
5. Either copy and paste the public part of your SSH key into the “SSH Public key” box or use the button to select the public key file on your computer.
6. Click “Add” to associate the public SSH key part with your account

The public SSH key part will now be added to your login account on the ARCHER2 system.

### Log into ARCHER2

You should now be able to log into ARCHER2 by following the [login instructions in the ARCHER2 documentation](https://docs.archer2.ac.uk/user-guide/connecting/#ssh-clients).

### Using an existing ARCHER2 account

If you wish to use an existing ARCHER2 account for the course, that is perfectly fine. The
only difference from the specific course account is that you will not have access to the
the node reservations for the course. Instead, you can use the ARCHER2 `short` QoS which will
work fine for this course. When the course talks about setting the following options:

```
--qos=standard --reservation=<name of course reservation>
```

you should replace them with:

```
--qos=short --reservation=shortqos
```

Similarly, when you are writing job submission scripts, you should replace:

```
#SBATCH --qos=standard
#SBATCH --reservation=<name of course reservation>
```

with

```
#SBATCH --qos=short
#SBATCH --reservation=shortqos
```

{% include links.md %}

{% comment %}
<!--  LocalWords:  myfile kbd links.md md endcomment
-->
{% endcomment %}

