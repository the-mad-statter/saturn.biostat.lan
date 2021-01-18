Scheduling SAS Jobs on the Saturn Cluster via Cron
================

<!-- badges: start -->

[![Under
Development](https://img.shields.io/badge/status-under%20development-red.svg)](https://github.com/the-mad-statter/saturn.biostat.lan)
[![Last
Commit](https://img.shields.io/github/last-commit/the-mad-statter/saturn.biostat.lan.svg)](https://github.com/the-mad-statter/saturn.biostat.lan/commits/master)
<!-- badges: end -->

## Overview

The [Division of Biostatistics](https://biostatistics.wustl.edu/) at
[Washington University School of Medicine in
St. Louis](https://medicine.wustl.edu/) maintains a 12-node Linux
cluster named Saturn for use by faculty and staff in the Division. This
vignette discusses how to run SAS jobs according to a cron schedule, and
optionally, to share output via email.

## Software you will use

### Locally

1.  [Virtual Private Network (VPN)
    Client](https://en.wikipedia.org/wiki/Virtual_private_network)
    1.  allows you to connect to the University network
    2.  the University supports Cisco AnyConnect
2.  [Secure Shell (SSH)
    Client](https://en.wikipedia.org/wiki/SSH_(Secure_Shell))
    1.  allows you to issue commands on the cluster
    2.  examples include [Putty](https://www.putty.org) or
        [X-Win32](https://www.starnet.com/xwin32)
    3.  configure your client to connect to
        [saturn.biostat.lan](saturn.biostat.lan) using the SSH protocol
        on port 22
3.  [Secure File Transfer Protocol (SFTP) Client
    (optional)](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol)
    1.  makes it easier to upload/download files to/from the cluster
    2.  examples include [FileZilla](https://filezilla-project.org/) or
        [WinSCP](https://winscp.net/eng/index.php)
    3.  configure your client to connect to
        [saturn.biostat.lan](saturn.biostat.lan) on port 22

### Saturn

1.  [GNU nano](https://en.wikipedia.org/wiki/GNU_nano)
    1.  a text editor for Unix-like computing systems using a command
        line interface available on the cluster
2.  [TORQUE](https://en.wikipedia.org/wiki/TORQUE)
    1.  the resource management system used for submitting and
        controlling jobs on the cluster
    2.  allows for numerous directives, which are used to specify
        resource requirements and other attributes for jobs
    3.  For additional help using TORQUE to submit and manage jobs, see
        the [Submitting and managing jobs
        chapter](http://docs.adaptivecomputing.com/torque/4-0-2/help.htm#topics/2-jobs/submittingManagingJobs.htm)
        of [Adaptive Computing’s TORQUE Administrative
        Guide](http://docs.adaptivecomputing.com/torque/4-0-2/help.htm#topics/0-about/coverPage.htm).
3.  [cron](https://en.wikipedia.org/wiki/Cron)
    1.  a time-based job scheduler in Unix-like computer operating
        systems available on the cluster
4.  [mail](https://en.wikipedia.org/wiki/Mail_(Unix))
    1.  a command-line email client for Unix and Unix-like operating
        systems available on the cluster

## Files you will work with

1.  [SAS](https://en.wikipedia.org/wiki/SAS_(software)) script file
    1.  performs all the report generation steps from loading data to
        generating one or more output files
    2.  this file is called [my\_sas\_script.sas](my_sas_script.sas) in
        the vignette and is written to create an output pdf file called
        “my\_sas\_output.pdf” when executed
2.  A TORQUE job script
    1.  a place to specify all of the resource requirements and other
        attributes for the job
    2.  In this example we will use a file called
        [my\_job\_script.pbs](my_job_script.pbs) to request our job be
        run on a node with SAS installed.
3.  Crontab file
    1.  In the vignette, we will issue commands to open and edit your
        default crontab file to schedule times for the example report to
        run.
        1.  In this file each line corresponds to a different
            combination of job and schedule.
        2.  The first characters of each line specify when the job
            should be run.
        3.  Various websites such as <https://crontab.guru> can help
            determine what to enter to specify the desired schedule.
        4.  The latter characters of each line specify the commands to
            run at that time.

## Vignette

1.  Connect to the WUSTL network via VPN.
2.  Connect to the Saturn cluster head node via your SSH client.
3.  Create a new directory to help keep your files organized.
    1.  In this example we will use a directory called “my\_sas\_job”
    2.  Issue the command `mkdir my_sas_job` to make the directory
4.  Navigate into the new directory.
    1.  Use the command `cd my_sas_job`
5.  Upload your SAS script file to the “my\_sas\_job” directory using
    your SFTP program.
6.  Create the TORQUE job script file
    1.  Issue the command `nano my_job_script.pbs` in your SSH program
        to open a new text file in the nano editor and enter the code
        below where:
        1.  `#!/bin/bash` specifies this is code to be run in a [bash
            shell](https://en.wikipedia.org/wiki/Bash_(Unix_shell))
        2.  `#PBS -l nodes=1:sas` tells TORQUE you want one node but
            more importantly that you want a node that has SAS installed
            (As of this writing only nodes
            [saturn2.biostat.lan](saturn2.biostat.lan) and
            [saturn7.biostat.lan](saturn7.biostat.lan) appear to have
            SAS installed and accept jobs from the default queue)
        3.  `cd my_sas_job` changes the working directory from your home
            root to your job directory
        4.  `sas my_sas_script.sas` tells SAS to run your SAS script
    2.  Use “Ctrl + O” then press “Enter” to save the file and “Ctrl +
        X” to exit the nano editor.

``` bash
#!/bin/bash
#PBS -l nodes=1:sas
cd my_sas_job
sas my_sas_script.sas
```

1.  Schedule your job via CRON
    1.  Issue the command `export VISUAL=nano; crontab -e` to edit your
        default crontab file in the nano editor.
    2.  Enter the below code (make sure to end the file on a new line)
        to run your job every first day of the month at 1:00 PM.
    3.  Use “Ctrl + O” then press “Enter” to save the file and “Ctrl +
        X” to exit the nano editor.

``` bash
0 13 1 * * cd my_sas_job; qsub my_job_script.pbs
```

## Clean Up

1.  If your job at least starts at the appointed time, you will likely
    get mail indicating a success or a point of failure.
    1.  To check your mail enter the command `mail` in your SSH client.
    2.  Type of the number of the message you wish to read and press
        “Enter”.
    3.  After reading the email you may enter `d` and press “Enter” to
        delete the message.
    4.  Type `q` and press “Enter” to quit mail.
2.  You may also wish to clean up all of the extra files generated in
    your job directory “my\_sas\_job”.
    1.  Navigate into the directory.
    2.  Use the command `rm filename` to remove files you no longer
        want.

## Optional Emailing of Report Output

1.  The mail program is also configured to send email. If you wish to
    use it to send your report output (e.g., “my\_sas\_output.pdf”)
    1.  Use `nano message.txt` to open the “message.txt” file in the
        nano text editor
        1.  enter a message for the body of your email.
        2.  Use “Ctrl + O” then press “Enter” to save the file.
        3.  Use “Ctrl + X” to exit the nano editor.
    2.  Add the following code to the bottom of your TORQUE job script
        where:
        1.  -s specify the email subject in quotes
        2.  -a specify attachment file
        3.  -c send carbon copies to user(s) (comma separated list of
            names)
        4.  -r specify from address
        5.  <recipient_1@wustl.edu>,<recipient_2@wustl.edu> (comma
            separated list of recipients)
        6.  `< message.txt` sets the email message body to be the
            contents of message.txt

``` bash
mail -s “My Subject” -a “my_sas_output.pdf” -c cc_1@wustl.edu,cc_2@wustl.edu -r ‘”Your Last Name, Your First Name” your_email@wustl.edu’ recipient_1@wustl.edu,recipient_2@wustl.edu < message.txt
```

## About

### Washington University in Saint Louis <img src="img/brookings_seal.png" align="right" width="125px"/>

Established in 1853, [Washington University in Saint
Louis](https://www.wustl.edu) is among the world’s leaders in teaching,
research, patient care, and service to society. Boasting 24 Nobel
laureates to date, the University is ranked 7th in the world for most
cited researchers, received the 4th highest amount of NIH medical
research grants among medical schools in 2019, and was tied for 1st in
the United States for genetics and genomics in 2018. The University is
committed to learning and exploration, discovery and impact, and
intellectual passions and challenging the unknown.
