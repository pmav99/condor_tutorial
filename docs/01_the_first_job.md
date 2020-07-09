## Submitting our first job

### Where to work from?

The first thing we need to do is to change the Current Working Directory (CWD) to a directory where
both our regular user and the `ghslproc` user have **write** access. A good example of such
a directory is the GHSL project home directory on EOS. This is located at:

``` bash
cd /eos/jeodpp/data/projects/GHSL
```

!!! Warning

    Obviously you need to change `GHSL` with you own project name.

Since the project home directory is shared among multiple people, it is usually a good idea to
create a subdirectory with your user name and operate in there. So please run this:

``` bash
cd /eos/jeodpp/data/projects/GHSL
mkdir $(whoami)
cd $(whoami)
```

### Submission file

In order to submit a job we must first create a text file with describes the job.
For now please create a file named `job_description` with the following contents:

```
universe                = docker
docker_image            = jeoreg.cidsn.jrc.it:5000/jeodpp-htcondor/jeodpp_base_gdal_py3_deb10:1.0
executable              = /bin/cat
arguments               = /etc/hosts
output                  = hello.$(ClusterId).$(ProcId).out
error                   = hello.$(ClusterId).$(ProcId).err
log                     = hello.$(ClusterId).$(ProcId).log
request_memory          = 100M
queue 1
```

!!! note

    The actual filename is not really important, but `job_description` is what we will be using throughout
    this tutorial.

We will not go through all the details now.  In a nutshell, this is the description of a job, which,
when submitted, will create a docker container in one of the JEO-Batch nodes and will execute
`/bin/cat /etc/hosts`.

### Submit the job

Now, please proceed to submit the job with: `sudo -u ghslproc condor_submit job_description`. You
should see something like this:

```
$ sudo -u ghslproc condor_submit job_description

Submitting job(s).
1 job(s) submitted to cluster 40152.
```

Take notice of the number `40152` (yours will obviously be different). This is the `JOB_ID`. We will
need it later on.

You can check the status of the job with: `condor_q`. You should see something like this:

```
$ sudo -u ghslproc condor_q

-- Schedd: s-jrciprjeop161p.cidsn.jrc.it : <139.191.240.161:9618?... @ 07/06/20 16:18:53
OWNER    BATCH_NAME    SUBMITTED   DONE   RUN    IDLE  TOTAL JOB_IDS
ghslproc ID: 40152    7/6  16:18      _      1      _      1 40152.0

Total for query: 1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
Total for ghslproc: 1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
Total for all users: 411 jobs; 0 completed, 0 removed, 86 idle, 16 running, 309 held, 0 suspended
```

!!! Tip

    The job we submited is trivial. It should be finished in just a few seconds. If you don't see
    any jobs when you run `condor_q` then chances are that the job finished sucecessfully.
    Sumbit another job and try to be a little faster this time)

!!! note

    Do notice the "owner". It is `ghslproc`. In your case it will be your own `*proc` user!

Do notice the RUN column. The `1` underneath means that the container has already been deployed and
that the job is currently running.

If the JEO-Batch cluster is very busy, you might notice that your job is under IDLE instead.  If
that is the case just wait for a while and check the status again.  If it remains indefinitely on
IDLE, then chances are that you requested too many resources (e.g. too much RAM). If that is not the
case, please [open
a ticket](https://cidportal.jrc.ec.europa.eu/apps/gitlab/for-everyone/support/issues/new).

Do also notice the `JOB_IDS` column. As you can see it is the same `JOB_ID` we got when we submitted
the job.

After a few seconds the job should have finished. Whet it is finished there should be no jobs for
`ghslproc`:

```
$ sudo -u ghslproc condor_q

-- Schedd: s-jrciprjeop161p.cidsn.jrc.it : <139.191.240.161:9618?... @ 07/06/20 16:22:24
OWNER BATCH_NAME      SUBMITTED   DONE   RUN    IDLE   HOLD  TOTAL JOB_IDS

Total for query: 0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
Total for ghslproc: 0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
Total for all users: 410 jobs; 0 completed, 0 removed, 87 idle, 14 running, 309 held, 0 suspended
```

### Check output

Now that the job has been finished, there should be 3 new files in our working directory:

- `hello.40152.0.out` which contains the STDOUT of the container
- `hello.40152.0.err` which contains the STDERR of the container
- `hello.40152.0.log` which contains some logs which come from HTCondor

!!! tip

    Obviously, the `JOB_ID` of your files will be different. Please adjust as necessary.

``` bash
$ ls -lah hello*

-rw-r--r-- 1 ghslproc EC_JRC_P_GHSL   0 Jul  9 10:42 hello.40152.0.err
-rw-r--r-- 1 ghslproc EC_JRC_P_GHSL 942 Jul  9 10:42 hello.40152.0.log
-rw-r--r-- 1 ghslproc EC_JRC_P_GHSL 223 Jul  9 10:42 hello.40152.0.out
```

As you can see, in our example, the STDERR file is empty while the STDOUT error contains the
contents of `/etc/hosts` of the container that got created. We can easily check this with:

```
$ cat hello.40152.0.out

127.0.0.1localhost
::1localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2ghslproc-40152.0-s-jrciprjeop194p.cidsn.jrc.it ghslproc-40152
```

Congratulations! You just managed to use JEO-Batch successfully!


### Cleanup

Just remove the log files with:

``` bash
rm -f hello*
```
