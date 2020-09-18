---
title: How to Use JupyterLab to Submit Jobs to the OSG's Central Pool
overview: How to Use JupyterLab to Submit Jobs to the OSG's Central Pool
published: true
permalink: blog/jupyterlab-and-osg-condor-pool.html
attribution: The SLATE Team
layout: post
type: markdown
tag: draft
---

In our previous blog post on [JupyterLab and HTCondor with SLATE](https://slateci.io/blog/slate-jupyter-condor-june-2020.html), we showed how a user can leverage multiple SLATE catalog applications to deploy a JupyterLab instance and a test HTCondor pool on SLATE.
 
In this blog post, however, we demonstrate how you can deploy a JupyterLab from the SLATE catalog, and then use it to submit HTCondor jobs to the OSG's central pool. We thought this would be very helpful for those users who work with such a production-scale high-throughput cluster and would like to use the condor-submit feature that's integrated within the SLATE JupyterLab application to submit their jobs.

In this blog post, we assume you have a SLATE account and client installed on your laptop (c.f. the [SLATE quickstart](https://slateci.io/docs/quickstart/)) and access to a SLATE registered Kubernetes cluster.

<!--end_excerpt-->

## Step 1

To be able to submit jobs to OSG, you'll need an authentication token and a project name in OSG. If you're already have that, you can proceed to Step 2. 

If you don't already have access, you can submit a ticket to the [OSG Research Facilitation team](https://support.opensciencegrid.org/support/tickets/new) requesting access to the OSG Central Pool and mention in your ticket that you'll be submitting jobs from the `slateci.io` domain. Once your request is processed and approved, you should have a submit authentication token and project name that you can use in the next steps.

## Step 2

The next step is to create a SLATE secret for your submit token. 

Copy the token and paste it into a file named `submit-token`.

Then, create a secret using the below SLATE command and the above token file you created<small>(Please change &lt;your-group&gt; in the command to your SLATE group name, and &lt;a-cluster&gt; to the target cluster name that you want to use for your deployment)</small>:

	$ slate secret create submit-auth-token --group <your-group> --cluster <a-cluster> --from-file condor_token=submit-token
	Successfully created secret submit-auth-token with ID secret_dHiGnjAgR2A 
	

## Step 3

### Deploy JupyterLab
Download the base configurations:

	$ slate app get-conf --dev jupyter-notebook > jupyter.conf

Generate a random token:

	$ openssl rand -base64 32
	mO6KJvhomZ733r/UUW6i1VXuuWgXV/gVN3VrXOgNwEg=

Edit the JupyterLab application configuration file, in this case `jupyter.conf`, so that it has an **Instance** name and **NB_USER** of your choice, and a **Token** set to the token you just generated in the previous command. In our example, those values are:

	Instance: 'blogpostjupyter' 	
	Jupyter:
	  NB_USER: 'slate'
	  Token: 'mO6KJvhomZ733r/UUW6i1VXuuWgXV/gVN3VrXOgNwEg='

Choose a subdomain for the ingress (This will be used in the application's URL):

		Ingress:
			Subdomain: 'blogpostnotebook'

Update the **CondorConfig** to be enabled, and use hostname `flock.opensciencegrid.org` as the **CollectorHost**, port `9618` as the **CollectorPort**, a port number between 30000-32767 as the **ExternalCondorPort** and the submit secret, in our example "submit-auth-token", as the **AuthTokenSecret**. In our example, the configuration will be:

	CondorConfig:
      Enabled: true
      CollectorHost: flock.opensciencegrid.org
      CollectorPort: 9618
      ExternalCondorPort: 32676
      AuthTokenSecret: submit-auth-token
      
The last change is for the SSH service. Enable the service and add the SSH public key you want to use to SSH into the JupyterLab instance:

	SSH:  
	  Enabled: true
	  SSH_Public_Key: 'ssh-rsa AAAAB3NzaC1yc2......i0pRTQgD5h1l+UvL/udO+IUYvvi slate'

Considering the number of jobs we'll be submitting in this demo, and local processes created by condor to handle that, we recommend increasing the resource limit in the config file as follows:

	Resources:
	# The maximum amount of CPU resources the notebook should be able to use
	# in units of thousandths of a CPU core, e.g. 1000 == 1 CPU core. 
	CPU: 2000
	# The maximum amount of memory the notebook should be able to use, 
	# in megabytes. 
	# Note that jupyter and other built-in components use some memory,	# so somewhat less than the value specified here will be available 
	# to user code. 
	Memory: 4096

You're now ready to install the JupyterLab application on SLATE:

	$ slate app install jupyter-notebook --dev --group <your-group> --cluster <a-cluster> --conf jupyter.conf

###### Note: If deployment fails due to an instance name that's already been chosen by another user or due to an ExternalCondorPort value that's already allocated, please choose a different value for the instance and\or port and try running the above command again. 	

Inspect the instance's info to see the allocated URL and address for SSH service:

	$ slate instance info <instance-ID>
	Services:
	Name                        Cluster IP    External IP   Ports          URL                                     
	your-group-jupyter-notebook 10.96.150.245 <IP-address>  8888:30712/TCP http://blogpostnotebook.slate-dev.slateci.net/
	your-group-jupyter-notebook 10.96.150.245 <IP-address>  22:30033/TCP   <ip-address>:<port-number>

In the above example, the JupyterLab application can be accessed at this address *blogpostnotebook.slate-dev.slateci.net* using the Jupyter token you generated above. The second line has the SSH access info under URL, so you can use it with the ssh command like this:

	ssh -p <port-number> <username>@<ip-address>

where &lt;username&gt; is what you chose above for the NB_USER configuration variable. 


## Step 4
### Testing	
To test your deployed applications, log into your JupyterLab application\instance and submit a test job to the OSG HTCondor pool from either a terminal window or notebook.


For our testing purpose, we use an [OSG Connect Quickstart](https://support.opensciencegrid.org/support/solutions/articles/5000633410-osg-connect-quickstart) tutorial example with some modifications. 

First, create a test script to be executed as your job:

	nano short_transfer.sh
	
Copy the below code into it and save the file:

	#!/bin/bash
	printf "Start time: "; /bin/date
	printf "Job is running on node: "; /bin/hostname
	printf "Job running as user: "; /usr/bin/id
	printf "Job is running in directory: "; /bin/pwd
	printf "The command line argument is: "; echo $1
	printf "Contents of $1 is "; cat $1
	cat $1 > output.txt
	printf "Working hard..."
	ls -l $PWD
	sleep 1
	echo "Science complete!"

The above script takes an input file as an argument so we need to create one. A quick way to do that is:

	echo "Hello World from SLATE" > input.txt

Then, create a condor submit file 'job.sub':

	nano job.sub

Copy the below job into the submit file and save it:

	executable = short_transfer.sh
	arguments = input.txt
	transfer_input_files = input.txt
	transfer_output_files = output.txt

	error = log/job.$(Cluster).$(Process).error
	output = log/job.$(Cluster).$(Process).output
	log = log/job.$(Cluster).$(Process).log

	request_cpus = 1
	request_memory = 1 MB
	request_disk = 1 MB
	+ProjectName = "<your-project-name>"
	# Let's queue a 1000 jobs with the above specifications
	queue 1000

######Note: Please note that you'd need to subsititute `<your-project-name>` above with the name of your project in OSG. 

Then, create a log directory:

	mkdir log 

and submit the job: 

	$ condor_submit job.sub
	Submitting job(s).
	1000 job(s) submitted to cluster 15.


A successful run will create in your local directory the file *output.txt*  with the "Hello World from SLATE" message in it. Addionally, you should see inside the log directory all the `output`, `error` and `log` files for the individual jobs.

#### Submiting your Jobs from a Python Notebook
If you prefer to use python to submit your jobs to the pool, you can do that using HTCondor Python Bindings. Here is how you can do that:

Open a new Python notebook, and import the below two modules:
<img src="/img/posts/jupyter-osg-pb-i.png"> 

Then, create a `Submit` object for your job:
<img src="/img/posts/jupyter-osg-pb-s.png"> 

<details>
  <summary>
  	Click here for above sourcecode
  </summary>

```python
short_transfer_job = htcondor.Submit({
"executable": "short_transfer.sh",  # The program to run on the execute node
"arguments": "input.txt",
"transfer_input_files": "input.txt", 
"transfer_output_files": "output.txt",
"error": "log/job.$(Cluster).$(Process).error",     # Anything the job prints to standard error will end up in this file
"output": "log/job.$(Cluster).$(Process).output",   # Anything the job prints to standard output will end up in this file
"log": "log/job.$(Cluster).$(Process).log",         # This file will contain a record of what happened to the job
"request_cpus": "1",          # How many CPU cores we want
"request_memory": "1MB",      # How much memory we want
"request_disk": "1MB",        # How much disk space we want
"+ProjectName": classad.quote("<my-project-name>"),
})

print(short_transfer_job)
```

</details>

The last command prints the job so that you can verify that it has right specifications you want. Please note that you'd need to add in your OSG project name, where it says `<my-project-name>`, to the job.

The last step is to queue your job like this:
<img src="/img/posts/jupyter-osg-pb-q.png"> 
<details>
  <summary>
  	Click here for above sourcecode
  </summary>

```python
schedd = htcondor.Schedd()          # get the Python representation of the scheduler
with schedd.transaction() as txn:   # open a transaction, represented by `txn`
	cluster_id = short_transfer_job.queue(txn,1000)     # queues 1000 job in the current transaction;
	
print(cluster_id)
```
</details>


The time it would take for your jobs to finish depends on resource availability within the pool and the processing time your jobs need. To check the status of your job submission, you can run the command `condor_q` from the terminal to see your jobs that are pending, running or done, as you can see in this output example:

	OWNER  BATCH_NAME    SUBMITTED   DONE   RUN    IDLE  TOTAL JOB_IDS
	jovyan ID: 4        9/18 02:43    825     14    161   1000 4.132-952

	Total for query: 175 jobs; 0 completed, 0 removed, 161 idle, 14 running, 0 held, 0 suspended
	Total for jovyan: 175 jobs; 0 completed, 0 removed, 161 idle, 14 running, 0 held, 0 suspended
	Total for all users: 175 jobs; 0 completed, 0 removed, 161 idle, 14 running, 0 held, 0 suspended

## Uninstall 

If you need to uninstall an application you previously deployed on SLATE, run this command:

	$ slate instance delete <instance-ID>
	$ slate secret delete <secret-ID>

## Summary

In summary, we were able to successfully deploy a JupyterLab instance on SLATE, and demonstrate job submission to a production-scale high-throughput cluster using the Open Science Grid. 


## Questions?

As always, we encourage you to try this out and let us know what's working, what's not, what can be improved and so on. For discussion, news and troubleshooting, the [SLATE Slack workspace](https://slack.slateci.io/) is the best place to reach us! 