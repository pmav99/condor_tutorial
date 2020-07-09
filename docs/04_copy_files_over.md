# Copy files over

All the examples we've seen so far were pretty much trivial.
In order to be able to do more interesting stuff, we usually need to COPY one more files
to the container in order to customize its behaviour. These files can be either a script that
will get executed or files that will be passed as arguments to the `executable`.

!!! note

    Strictly speaking, copying files over is not necessary. Instead of that we can choose a
    docker image which already contains the script we want to execute. Nevertheless, this requires
    that a specific docker image has been prepared for us.

## Create an executable script

Let's start by creating a custom executable shell script. Open a new file named `my_script.sh`
and place the following contents:

```
#!/usr/bin/env bash
#

# http://redsymbol.net/articles/unofficial-bash-strict-mode/
set -euo pipefail

echo "The current host is: $(hostname)
echo "The current user is: $(id -u)"
echo "The present working directory is: $(pwd)"
echo "The arguments passed to the script are: ${@}"
```

Now, make it executable:

``` bash
chmod +x ./my_script.sh
```

And test it out. E.g.:

``` bash
$ ./my_script.sh 123 'Michael Jordan' 'The answer is 42'
The current host is: jeodpp-text-terminal-144p-03
The current user is: 35861
The present working directory is: /eos/jeodpp/data/projects/GHSL/mavropa
The arguments passed to the script are: 123 Michael Jordan The answer is 42
```

## Copy files to the container

In order to pass the file to the container, we need to add a couple of entries in our
`job_description` file.

```
transfer_input_files    = my_script.sh
should_transfer_files   = YES
when_to_transfer_output = ON_EXIT
```

!!! Tip

    If you want to copy over multiple files, you can set `transfer_input_files` to a comma separated
    string. E.g.  `file1,file2,file3`

Furthermore, we should also change the `executable` entry to `my_script.sh`, too.

For your convenience the full `job_description` file is this:


``` bash
universe                = docker
docker_image            = jeoreg.cidsn.jrc.it:5000/jeodpp-htcondor/jeodpp_base_gdal_py3_deb10:1.0
transfer_input_files    = my_script.sh
should_transfer_files   = YES
when_to_transfer_output = ON_EXIT
executable              = my_script.sh
arguments               = 123 'Michael Jordan' 'The answer is 42'
output                  = hello.$(ClusterId).$(ProcId).out
error                   = hello.$(ClusterId).$(ProcId).err
log                     = hello.$(ClusterId).$(ProcId).log
request_memory          = 100M
queue 1
```

Submit the job and check the output file:

```bash
$ sudo -u ghslproc condor_submit job_description
Submitting job(s).
1 job(s) submitted to cluster 40297.

# wait a few seconds...

$ cat hello.40297.0.out
The current host is: ghslproc-40297.0-s-jrciprjeop194p.cidsn.jrc.it
The current user's UID is: 35637
The present working directory is: /var/lib/condor/execute/dir_15155
The arguments passed to the script are: 123 'Michael Jordan' 'The answer is 42'
```

And that was it! We executed a custom script via JEO-Batch!

## What should I use for `executable`

A relatively subtle point is the `executable` entry in the `job_description` file.

What you should keep in mind is that:

1. if you use an **absolute path** (e.g. `/bin/cat` then the executable file must be present in the
   docker image.

2. if you use a **relative path** (e.g. `my_script.sh`) then the executable file must be copied over
   to the container (i.e. you must add it to `transfer_input_files`, too).

## Cleanup

Just remove the log files with:

``` bash
rm -f hello*
```
