**********************************************************************
Precip 
**********************************************************************

.. sidebar:: Page Contents

   .. contents::
      :local:


The Pegasus Repeatable Experiments for the Cloud in Python (Precip), is a flexible exeperiment management API for running experiments on clouds. Precip was developed for use on FutureGrid infrastructures such as OpenStack, Eucalyptus (>=3.2), Nimbus, and at the same time commercial clouds such as Amazon EC2. The API allows you to easily provision resources, which you can then can run commands on and copy files to/from subsets of instances identified by tags. The goal of the API is to be flexible and simple to use in Python scripts to control your experiments.

The API does not require any special images, which makes it easy to get
going. Any basic Linux image will work. More complex images can be used
if your experiment requires so, or you can use the experiment API to run
bootstrap scripts on the images to install/configure required software.

A concept which simplfies interacting with the API is instance tagging.
When you start an instance, you can add arbitrary tags to it. The
instance also gets a set of default tags. API methods such as running a
remote command, or copying files, all use tags for specify which
instances you want to target.

Precip also handles ssh keys and security groups automatically. This is
done to make sure the experiment management is not interfering with your
existing cloud setup. The first time you use Precip, a directory will be
created called ~/.precip. Inside this directory, a ssh keypair will be
created and used for accessing instances. On clouds which supports it,
the keypair is automatically registered as 'precip', and a 'precip'
security group is created. If your experiement requires more ports to be
open you can use the cloud interface to add those ports to the precip
security group.

Precip is a fairly new API, and if you have questions or suggestions for
improvements, please contact
`pegasus-support@isi.edu <mailto:pegasus-support@isi.edu>`__

Installation
------------

If you want to use the India or Sierra FutureGrid resources to manage
your experiment, Precip is available on the interactive logins nodes via
modules: module load precip/0.1
 
You can also install Precip on your own machine. Prerequisites are
the Paramiko and Boto Python modules. The Python source package and RPMs
are available at:
`http://pegasus.isi.edu/static/precip/software/ <http://pegasus.isi.edu/static/precip/software/>`__

API
---

**provision(image\_id, instance\_type='m1.small', count=1, tags=None)**
    Provision a new instance. Note that this method starts the
    provisioning cycle, but does not block for the instance to finish
    booting. For blocking on instance creation/booting, see wait()

    Parameters:

    -  **image\_id** - the id of the image to instanciate

    -  **instance\_type** - the type of instance. This is infrastructure
       specific, but usually follows the Amazon EC2 model with m1.small,
       m1.large, and so on.

    -  **count** - number of instances to create

    -  **tags** - these are used to manipulate the instance later. Use
       this to create logical groups of your instances.

**wait(tags=[], timeout=600)**
    Barrier for all instances matching the tags argument. This method
    will block until the instances have finish booting and are
    accessible via their external hostnames.

    Parameters:

    -  **tags** - tags specifying the subset of instances to block on.
       The default value is [] which means wait for all instances.

    -  **timeout** - timeout in seconds for the instances to boot. If
       the timeout is reached, an ExperimentException is raised. The
       default is 600 seconds.

**deprovision(tags)**
    Deprovisions (terminates) instances matching the tags argument

    Parameters:

    -  **tags** - tags specifying the subset of instances to
       deprovision.

**list(tags)**
    Returns a list of details about the instances matching the tags. The
    details include instance id, hostnames, and tags.

    Parameters:

    -  **tags** - tags specifying the subset of instances to give
       information on. If you want details on all current instances, use
       [].

    Returns:

    -  List of dictionaries, one for each instance.

**get\_public\_hostnames(tags)**
    Provides a list of public hostnames for the instances matching the
    tags. The public hostnames can be provided to other instances in
    order to let the instances know about eachother.

    Parameters:

    -  **tags** - tags specifying the subset of instances.

    Returns:

    -  A list of public hostnames

**get\_private\_hostnames(tags)**
    Provides a list of private hostnames for the instances matching the
    tags. The private hostnames can be provided to other instances in
    order to let the instances know about eachother.

    Parameters:

    -  **tags** - tags specifying the subset of instances.

    Returns:

    -  A list of private hostnames

**get(tags, remote\_path, local\_path, user="root")**
    Transfers a file from a set of remote machines matching the tags,
    and stores the file locally. If more than one instance matches the
    tags, an instance id will be appended to the local\_path.

    Parameters:

    -  **tags** - these are used to manipulate the instance later. Use
       this to create logical groups of your instances.

    -  **remote\_path** - the path of the file on the remote instance

    -  **local\_path** - the local path to tranfer to

    -  **user** - remote user. If not specified, the default is 'root'

**put(tags, local\_path, remote\_path, user="root")**
    Transfers a local file to a set of remote machines matching the
    tags.

    Parameters:

    -  **tags** - these are used to manipulate the instance later. Use
       this to create logical groups of your instances.

    -  **local\_path** - the local path to tranfer from

    -  **remote\_path** - the path on the remote instance to store the
       file as

    -  **user** - remote user. If not specified, the default is 'root'

**run(tags, cmd, user="root", check\_exit\_code=True)**
    Runs a command on the instances matches the tags. The commands are
    run in series, on one instance after the other.

    Parameters:

    -  **tags** - these are used to manipulate the instance later. Use
       this to create logical groups of your instances.

    -  **cmd** - the command to run

    -  **user** - remote user. If not specified, the default is 'root'.
       If you need to run commands as another user, you will have to
       make sure that user accepts the ssh key in ~/.precip/

    -  **check\_exit\_code** - If set to True (default), commands
       returning non-zero exit codes will result in a
       ExperimentException being raised.

    Returns:

    -  A list of lists, containing exit\_code[], stdout[] and stderr[]
       for the commands run

**copy\_and\_run(tags, local\_script, args=[], user="root",
check\_exit\_code=True)**
    Copies a script from the local machine to the remote instances and
    executes the script. The script is run in series, on one instance
    after the other.

    Parameters:

    -  **tags** - these are used to manipulate the instance later. Use
       this to create logical groups of your instances.

    -  **local\_script** - the local script to run

    -  **args** - arguments for the script

    -  **user** - remote user. If not specified, the default is 'root'.
       If you need to run commands as another user, you will have to
       make sure that user accepts the ssh key in ~/.precip/

    -  **check\_exit\_code** - If set to True (default), commands
       returning non-zero exit codes will result in a
       ExperimentException being raised.

    Returns:

    -  A list of lists, containing exit\_code[], stdout[] and stderr[]
       for the commands run

The basic methods above are standard across all the Cloud
infrastructures. What is different is the constructors as each
infrastructure handles initialization a little bit different. For
example, to create a new OpenStack using the EC2\_\* environment
provided automatically by FutureGrid::
                
        exp = OpenStackExperiment(
                os.environ['EC2_URL'],
                os.environ['EC2_ACCESS_KEY'],
                os.environ['EC2_SECRET_KEY'])
                
            

i For Amazon EC2, you have to specify region, endpoint, and
access/secret keys. Note that it is not required to use environment
variables for your credentials, but seperating the crenditals from the
code prevents the credentials from being check in to source control
systems::
                
        exp = EC2Experiment(
                "us-west-2c",
                "ec2.us-west-2.amazonaws.com",
                os.environ['AMAZON_EC2_ACCESS_KEY'],
                os.environ['AMAZON_EC2_SECRET_KEY'])
                
            

Examples
--------

Hello World
~~~~~~~~~~~

::

                    
    #!/usr/bin/python

    import os
    import time
    from pprint import pprint

    from precip import *

    exp = None

    # Use try/except liberally in your experiments - the api is set up to
    # raise ExperimentException on most errors
    try:

        # Create a new OpenStack based experiment. In this case we pick
        # up endpoints and access/secret cloud keys from the environment
        # as exposing those is the common setup on FutureGrid
        exp = OpenStackExperiment(
                os.environ['EC2_URL'],
                os.environ['EC2_ACCESS_KEY'],
                os.environ['EC2_SECRET_KEY'])

        # Provision an instance based on the ami-0000004c. Note that tags are
        # used throughout the api to identify and manipulate instances. You 
        # can give an instance an arbitrary number of tags.
        exp.provision("ami-0000004c", tags=["test1"], count=1)

        # Wait for all instances to boot and become accessible. The provision
        # method only starts the provisioning, and can be used to start a large
        # number of instances at the same time. The wait method provides a 
        # barrier to when it is safe to start the actual experiment.
        exp.wait()

        # Print out the details of the instance. The details include instance id,
        # private and public hostnames, and tags both defined by you and some
        # added by the api
        pprint(exp.list())
       
        # Run a command on the instances having the "test1" tag. In this case we
        # only have one instance, but if you had multiple instances with that
        # tag, the command would run on each one.
        exp.run(["test1"], "echo 'Hello world from a experiment instance'")

    except ExperimentException as e:
        # This is the default exception for most errors in the api
        print "ERROR: %s" % e

    finally:
        # Be sure to always deprovision the instances we have started. Putting
        # the deprovision call under finally: make the deprovisioning happening
        # even in the case of failure.
        if exp is not None:
            exp.deprovision()
                    
                

Resources from mulitple infrastructures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

                    
    #!/usr/bin/python

    import os
    import time

    from precip import *

    ec2 = None
    fg = None

    try:

        # This example show how to run an experiment between Amazon EC2
        # and an OpenStack resource on FutureGrid. The setup is pretty
        # similar to the HelloWorld example, except that we now have to
        # experiment to handle. The first step is to get the experiments
        # initialized. Note that it is not required to use environment
        # variables for your credentials, but seperating the crenditals
        # from the code prevents the credentials from being check in to
        # source control systems.
        
        ec2 = EC2Experiment(
                "us-west-2c",
                "ec2.us-west-2.amazonaws.com",
                os.environ['AMAZON_EC2_ACCESS_KEY'],
                os.environ['AMAZON_EC2_SECRET_KEY'])
       
        fg = OpenStackExperiment(
                os.environ['EC2_URL'],
                os.environ['EC2_ACCESS_KEY'],
                os.environ['EC2_SECRET_KEY'])

        # Next we provision two instances, one on Amazon EC2 and one of
        # FutureGrid
        ec2.provision("ami-8a1e92ba", tags=["id=ec2_1"])
        fg.provision("ami-0000004c", tags=["id=fg_1"])

        # Wait for all instances to boot and become accessible. The provision
        # method only starts the provisioning, and can be used to start a large
        # number of instances at the same time. The wait method provides a 
        # barrier to when it is safe to start the actual experiment.
        ec2.wait([])
        fg.wait([])
        
        # Run commands on the remote instances
        ec2.run([], "echo 'Hello world Amazon EC2'")
        fg.run([], "echo 'Hello world FutureGrid OpenStack'")

    except ExperimentException as e:
        # This is the default exception for most errors in the api
        print "ERROR: %s" % e
        raise e
    finally:
        # Be sure to always deprovision the instances we have started. Putting
        # the deprovision call under finally: make the deprovisioning happening
        # even in the case of failure.
        if ec2 is not None:
            ec2.deprovision([])
        if fg is not None:
            fg.deprovision([])
                    
                

Setting up a Condor pool and run a Pegasus workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is a more complex example in which a small Condor pool is set up
and then a Pegasus workflow is run and benchmarked. The Precip script is
similar to what we have seen before, but it has two groups of instances,
one master acting as the Condor central manager, and a set of Condor
worker nodes::

                    
    #!/usr/bin/python

    import os
    import time

    from precip import *

    try:

        # This experiment is targeted to run on OpenStack
        exp = OpenStackExperiment(
                os.environ['OPENSTACK_URL'],
                os.environ['OPENSTACK_ACCESS_KEY'],
                os.environ['OPENSTACK_SECRET_KEY'])

        # We need a master Condor node and a set of workers
        exp.provision("ami-0000004c", tags=["master"],
                      instance_type="m1.large")
        exp.provision("ami-0000004c", tags=["compute"],
                      instance_type="m1.large", count=2)
        exp.wait()

        # The workers need to know what the private hostname of the master is
        master_priv_addr = exp.get_private_hostnames(["master"])[0]

        # Bootstrap the instances. This includes installing Condor and Pegasus,
        # downloading and settup the workflow.
        exp.copy_and_run(["master"], "./bootstrap.sh")
        exp.copy_and_run(["compute"], "./bootstrap.sh", args=[master_priv_addr])

        # Give the workers some time to register with the Condor central 
        # manager
        time.sleep(60)

        # Make sure Condor came up correctly
        exp.run(["master"], "condor_status")

        # Run the workflow
        exp.run(["master"], "cd ~/montage && ./run-montage", user="wf")

        # At this point, in a real experiment, you could for example provision
        # more resources and run the workflow again, or run the workflow with
        # different parameters/settings.

    except ExperimentException as e:
        print "ERROR: %s" % e
    finally:
        # always want to clean up all the instances we have started
        exp.deprovision([])
                    
                

We also need a bootstrap.sh which sets up the instances::

                    
    #!/bin/bash

    # This script bootstraps a basic RHEL6 instance to be have working
    # Condor and Pegasus installs. The script takes one optional
    # argument which is the address of the master instance (central
    # manager in Condor terminology). If the argument is not given,
    # the script sets up the instance to be the master.

    MASTER_ADDR=$1

    # for images with condor already installed, stop condor
    /etc/init.d/condor stop >/dev/null 2>&1 || /bin/true

    # correct clock is important for most projects
    yum -q -y install ntpdate
    /etc/init.d/ntpdate start

    # Add the EPEL repository
    wget -nv http://mirror.utexas.edu/epel/6/x86_64/epel-release-6-7.noarch.rpm
    rpm -Uh epel-release-*.noarch.rpm

    # Add the Condor repository
    cat >/etc/yum.repos.d/condor.repo <<EOF
    [condor-stable]
    name=Condor Stable RPM Repository for Redhat Enterprise Linux 6
    baseurl=http://www.cs.wisc.edu/condor/yum/stable/rhel6
    enabled=1
    gpgcheck=0
    EOF

    # Add the Pegasus repository
    cat >/etc/yum.repos.d/pegasus.repo <<EOF
    [Pegasus]
    name=Pegasus
    baseurl=http://download.pegasus.isi.edu/wms/download/rhel/6/x86_64
    gpgcheck=0
    enabled=1
    priority=50
    EOF

    # Install required software
    yum -q -y clean all
    yum -q -y install gcc gcc-g++ gcc-gfortran make gawk bison diffutils \
                      java-1.7.0-openjdk.x86_64 \
                      java-1.7.0-openjdk-devel.x86_64 \
                      ganglia-gmond condor pegasus

    # Clear the Condor local config file - we use config.d instead
    cat /dev/null >/etc/condor/condor_config.local

    # Common Condor config between master and workers
    cat >/etc/condor/config.d/50-main.conf <<EOF

    CONDOR_HOST = $MASTER_ADDR

    FILESYSTEM_DOMAIN = \$(FULL_HOSTNAME)
    TRUST_UID_DOMAIN = True

    DAEMON_LIST = MASTER, STARTD

    # security
    ALLOW_WRITE = 10.*
    ALLOW_READ = \$(ALLOW_WRITE)

    # default policy
    START = True
    SUSPEND = False
    CONTINUE = True
    PREEMPT = False
    KILL = False

    EOF

    # Master gets extra packages/configs
    if [ "x$MASTER_ADDR" = "x" ]; then
        yum -q -y install ganglia-gmetad ganglia-web

        cat >/etc/condor/config.d/90-master.conf <<EOF
    CONDOR_HOST = \$(FULL_HOSTNAME)
    DAEMON_LIST = MASTER, COLLECTOR, NEGOTIATOR, SCHEDD
    EOF
    fi

    # Restarting daemons
    /etc/init.d/condor start

    # User to run the workflows as, and allow the experiment management
    # ssh key to authenticate
    adduser wf
    mkdir -p ~wf/.ssh
    cp ~/.ssh/authorized_keys ~wf/.ssh/
    chown -R wf: ~wf/.ssh
        
    # Master is the submit host, so deploy our workflow on it
    if [ "x$MASTER_ADDR" = "x" ]; then
        # install the workflow tarball and wait script
        cd ~wf
        wget -q http://pegasus.isi.edu/static/precip/wf-experiment/montage.tar.gz
        tar xzf montage.tar.gz
        chown -R wf: montage*
    fi
                    
                

