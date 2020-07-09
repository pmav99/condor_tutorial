# When things go wrong

Now let's try to do something more interesting. Let's submit a job that fails.

## A failing job

Please edit the `job_description` file and change the `arguments` line from `/etc/hosts` to
`/etc/does_not_exist`:

```diff
-arguments               = /etc/hosts
+arguments               = /etc/does/not/exist
```

In other words we are asking condor to spawn a container that will execute:

```
cat /etc/does/not/exist
```

To make things easier, the full `job_description` file is here:

``` bash
universe                = docker
docker_image            = jeoreg.cidsn.jrc.it:5000/jeodpp-htcondor/jeodpp_base_gdal_py3_deb10:1.0
executable              = /bin/cat
arguments               = /etc/does/not/exist
output                  = hello.$(ClusterId).$(ProcId).out
error                   = hello.$(ClusterId).$(ProcId).err
log                     = hello.$(ClusterId).$(ProcId).log
request_memory          = 100M
queue 1
```

Submit the file:

```
sudo -u ghslproc condor_submit job_description

Submitting job(s).
1 job(s) submitted to cluster 40164.
```

Wait until the job finishes and check the files containing STDOUT and STDERR output:

``` bash
$ ls -lah hello*

-rw-r--r-- 1 ghslproc EC_JRC_P_GHSL             57 Jul  9 11:39 hello.40164.0.err
-rw-r--r-- 1 ghslproc EC_JRC_P_GHSL            942 Jul  9 11:39 hello.40164.0.log
-rw-r--r-- 1 ghslproc EC_JRC_P_GHSL              0 Jul  9 11:39 hello.40164.0.out
```

As we can see, the STDOUT file is empty, while the STDERR contains:

``` bash
$ cat hello.40164.0.err

/bin/cat: /etc/does/not/exist: No such file or directory
```

Congratulations! You now know how to to see what went wrong with your scripts!

### Cleanup

Just remove the log files with:

``` bash
rm -f hello*
```
