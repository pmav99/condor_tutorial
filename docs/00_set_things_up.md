# Setting things up

In order to be allowed to use JEO-Batch and write to EOS, your user must belong to one (or more)
groups. You can find the groups to which your user belongs to with the command `id`:

```bash
$ id mavropa

uid=35861(mavropa) gid=40507(EC_JRC_BigDataEOSS_CORE) groups=40507(EC_JRC_BigDataEOSS_CORE),504(cid_rdp),507(JEODPP_notebook),7921(TOL_CONTEU),29431(cid_dvpt),40600(EU_EC),41068(EC_JRC),50007(EC_JRC_P_GHSL)
```

In JEO-Batch context, the groups which are of interest are those that start with `EC_JRC_P_*`.

As you can see the user `mavropa`, currently, belongs to a bunch of groups among which there is
`EC_JRC_P_GHSL`.  This means that `mavropa`:

1. has read/write permissions on GHSL's project directory on EOS (i.e.
   `/eos/jeodpp/data/projects/GHSL`).

1. is allowed to use JEO-Batch via the `ghslproc` user.

!!! note

    For each JRC project there is a corresponding `EC_JRC_P_*` group and a corresponding `*proc`
    user. For example, for the `CRITECH` project, there is the corresponding `EC_JRC_P_CRITECH`
    group and the `critechproc` user.

!!! tip

    If `mavropa` also belonged to e.g. `EC_JRC_P_INCA`, then he would have also been able to use
    JEO-Batch via the `incaproc` user.

!!! warning

    It is `ghslproc` that we will be using throughout this tutorial, but chances are that you will
    not belong in that group. So, In order to continue, please check your `id`, identify your
    `EC_JRC_P_*` group and in the following commands, replace `ghslproc` user with the proc user of
    your own group!

To test that everything works properly please run the following command:

```bash
sudo -u ghslproc condor_q
```

`sudo` has been configured to *not* require a password when used with `condor_*` commands.  So, you
shouldn't get a password prompt.  The result should be something like this:

```bash
$ sudo -u ghslproc condor_q

-- Schedd: s-jrciprjeop161p.cidsn.jrc.it : <139.191.240.161:9618?... @ 07/06/20 12:49:31
OWNER BATCH_NAME      SUBMITTED   DONE   RUN    IDLE   HOLD  TOTAL JOB_IDS


Total for ghslproc: 0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
Total for all users: 410 jobs; 0 completed, 0 removed, 86 idle, 15 running, 309 held, 0 suspended

```

If this works then everything is fine and you can try to submit your first job.
If not, then there is some problem and you are advised to
[open a ticket](https://cidportal.jrc.ec.europa.eu/apps/gitlab/for-everyone/support/issues/new)
