> Authored by @nolash (Louis Holbrook)

### 1.1 Overview

This is a cluster-specific convenience setup for enabling a
Kubernetes-style liveness/readiness test as outlined in
<https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/>.

Conceptually, it provides an application with means to:

-   Run a collection of functions to validate sanity of the environment
-   Set a no-error state before execution of the main routine
-   Modify the error state during execution
-   Invalidating all state when execution ends

### 1.2 Python module

Three python methods are provided.

#### 1.2.1 load

This is meant to be called after configurations and environment has been
set up, but before the execution logic has commenced.

It receives a list of externally defined fully-qualified python modules.
Each of these modules must implement the method `health(*args,**kwargs)`
in its global namespace.

Any module returning `False` will cause a `RuntimeException`.

The component will not trap any other exception from the modules.

If successful, it will write the `pid` of the application to the
specified run data folder. By default this is `/run/<HOSTNAME>`, but the
path can be modified if desired.


#### 1.2.2 set

This is meant to be called during the execution of the main program
routine begins.


#### 1.2.2.1 at startup

It should be called once at the *start* of execution of the main program
routine.

For one-shot routines, this would mean the start of any code only run
when the module name is `__main__`.

For daemons, it would be just before handing over execution to the main
loop.


#### 1.2.2.2 during execution

Call `set(error_code=<error>, ...` any time the health state temporarily
changes. Any `error` value other than `0` is considered an unhealthy
state.

#### 1.2.2.3 at shutdown

Call `reset(...)`, which will indicate that the state is to be
considered the same as at startup.

### 1.3 shell

A bash script is provided for *Kubernetes* to perform the health check.

It performs the following checks:

1.  A numeric value exists in `<rundir>/<unitname>/pid`{.sample}.
2.  The numeric value is a directory in `/proc`{.sample} (a valid pid)
3.  The file `<rundir>/<unitname>/error`{.sample} contains \"0\"

If any of these checks fail should inditcate that the container is
unhealthy.
