# Hold your ~~horses~~ jobs

Sometimes, after submitting a job, `condor_q` will show that the job is under the `HOLD` column.
This implies that there is some sort of a problem.
The problem can be in the `job_description` file, in the docker image that you chose, in the script
that you executed etc.

!!! tip

    A really common case is that too many resources (e.g. RAM) was requested

## A job that ends up in HOLD

We can easily simulate such a problem be specifying an `executable` that does not exist. So please
apply the following patch to the `job_description` file:

```diff
-executable              = /bin/cat
+executable              = /bin/does_not_exist
```

For your convenience, the full `job_description` file is here:

``` bash
universe                = docker
docker_image            = jeoreg.cidsn.jrc.it:5000/jeodpp-htcondor/jeodpp_base_gdal_py3_deb10:1.0
executable              = /bin/does_not_exist
arguments               = /etc/hosts
output                  = hello.$(ClusterId).$(ProcId).out
error                   = hello.$(ClusterId).$(ProcId).err
log                     = hello.$(ClusterId).$(ProcId).log
request_memory          = 100M
queue 1
```

As usual, submit the file and check the status with `condor_q`:

``` bash
sudo -u ghslproc condor_submit job_description
sudo -u ghslproc condor_q
```

You should see something like this:

``` bash
-- Schedd: s-jrciprjeop161p.cidsn.jrc.it : <139.191.240.161:9618?... @ 07/06/20 17:01:53
OWNER    BATCH_NAME    SUBMITTED   DONE   RUN    IDLE   HOLD  TOTAL JOB_IDS
ghslproc ID: 40164    7/6  17:01      _      _      _      1      1 40164.0

Total for query: 1 jobs; 0 completed, 0 removed, 0 idle, 0 running, 1 held, 0 suspended
Total for ghslproc: 1 jobs; 0 completed, 0 removed, 0 idle, 0 running, 1 held, 0 suspended
Total for all users: 412 jobs; 0 completed, 0 removed, 87 idle, 15 running, 310 held, 0 suspended
```

Do notice that this time the job is on `HOLD`. And it will remain there until we manually remove it.

To further debug what is going on we need to run `condor_q` with the `-hold` argument:

``` bash
sudo -u ghslproc condor_q -hold
```

The output should be the following:

```
-- Schedd: s-jrciprjeop161p.cidsn.jrc.it : <139.191.240.161:9618?... @ 07/06/20 17:01:58
 ID       OWNER          HELD_SINCE  HOLD_REASON
40164.0   ghslproc        7/6  17:01 Error from slot1_1@s-jrciprjeop194p.cidsn.jrc.it: Error running docker job: OCI runtime create failed: container_linux.go:344: starting container process
 caused 'exec: \'/bin/does_not_exist\': stat /bin/does_not_exist: no such file or directory': unknown

Total for query: 1 jobs; 0 completed, 0 removed, 0 idle, 0 running, 1 held, 0 suspended
Total for ghslproc: 1 jobs; 0 completed, 0 removed, 0 idle, 0 running, 1 held, 0 suspended
Total for all users: 412 jobs; 0 completed, 0 removed, 87 idle, 15 running, 310 held, 0 suspended
```

The `HOLD_REASON` column describes what exactly is the problem (in this case that the executable
does not exist).

!!! tip

    In case the error message is cryptic and we can't figure out how to fix it, we should [open a ticket](https://cidportal.jrc.ec.europa.eu/apps/gitlab/for-everyone/support/issues/new)

## Remove the held job

Since we can't do much more with this particular job, we should remove it with the `condor_rm`
command and the JOB_ID:

```
sudo -u ghslproc condor_rm 40164.0
```

## Cleanup

Just remove the log files with:

``` bash
rm -f hello*
```
