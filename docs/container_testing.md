## Container Testing
*Note: This section is one topic in a larger discussion on best practices for
containerising research software, to hopefully be included here in the future.
For an introduction to containers in this context, we recommend reading 
[Using Containers](https://rosalindfranklininstitute.github.io/volume-em-container-documentation/intro/containers/)
also written by authors of this guide.*

Containerisation represents an opportunity to increase the accessibility, reproducibility, and reach of your software.
One question that arises when creating container images is to what extent additional tests need to be
designed for the containerised software.

Within an automated CI/CD pipeline, such as GitHub Workflows, it is
normally sufficient to check that the container builds and, if possible[^1], runs
without error. We also recommend building locally and checking the software 
functions correctly inside the container at least once following initial development,
and ideally periodically, since differences in container runtime directories, paths or
user-privileges may give rise to unexpected (but usually easily resolved) errors
compared to a direct installation. 

There is a second type of test one can create for containers: start-up or
installation tests. These can be simple checks that the runtime environment for
the container has the environmental variables or filesystem access that is
required for the application to run, or quick tests of the application itself.
They can be included via the **entrypoint** or start-up script of the container.
Below are two examples created for the Napari Empanada plugin, checking `DISPLAY` is set
and for access to common host mount paths. Other checks you may consider are GPU/CUDA availability,
expected executables being in `PATH`, and important shared libraries (`ldd`).
Note test failure in these cases should  provide a warning to the user rather than stop the
container process, since such failures may not be critical for all functionality of
the container and further are often resolvable via simple configuration changes
by the user (for example, specifying `--gpus` after `docker run` to allow
access to the host GPU). See also [Installation
Tests](plugin_testing.md#installation-tests) on the *Writing Tests For Plugins*
page.

[^1] Note a virtual display server such as `Xvfb` may be used to run graphical
applications headless in a Workflow.

## Example Container Tests for Napari-Empanada 
### Test: The DISPLAY env variable is set and the Napari GUI works
```
def test_display_set():
    import os
    if os.getenv("GITHUB_ACTIONS") == "true":
        pytest.skip("Skipping in GitHub Actions")
    if not os.environ.get("DISPLAY"):
        pytest.fail("DISPLAY unset - napari GUI unavailable")
```
Often, users will run a container on a remote machine or HPC cluster. In order to run the Napari GUI on the host system, the `$DISPLAY` variable needs to be set, which is what this test verifies. 


### Test: The container has read/write access the main filesystem
```
def test_host_mounts_accessible():
    # Non-User Filesystems and mount points that existed in Linux docker container
    # Transferability to other runtimes and host systems not guaranteed
    EXCLUDE_FSTYPES = {"overlay", "proc", "tmpfs", "devpts", "sysfs",
                       "cgroup", "cgroup2", "mqueue", "devtmpfs"}
    EXCLUDE_MOUNTPOINTS = re.compile(r"^(/etc/|/proc|/sys|/dev)")
    with open("/proc/mounts") as f:
        for line in f:
            _, mountpoint, fstype, *_ = line.split()
            if fstype in EXCLUDE_FSTYPES:
                continue
            if EXCLUDE_MOUNTPOINTS.match(mountpoint):
                continue
            # Check read access (N.B. moot if container run as root)
            if not os.access(mountpoint, os.R_OK):
                warnings.warn(f"Mountpoint {mountpoint} ({fstype}) is not readable")
                continue
            # Check write access
            if not os.access(mountpoint, os.R_OK):
                warnings.warn(f"Mountpoint {mountpoint} ({fstype}) is not writeable")
                continue
            return  # plausible filesystem r+w mount 
    pytest.fail(
        "No host filesystems detected with read+write access"
        "You may need to bind mount data with -v /host/path:/container/path"
    )
```

When running Napari and the plugin in a container, the host's filesystem needs to be accessible so the plugin can read the user's data. This test checks the container's `/proc/mounts` list shows read/write access to the mounted filesystem.
