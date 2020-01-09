name: running-nomad-jobs-and-clusters
class: title, shelf, no-footer, fullbleed
background-image: url(https://hashicorp.github.io/field-workshops-assets/assets/bkgs/HashiCorp-Title-bkg.jpeg)
count: false

# Running Nomad Jobs and Clusters
## Chapter 4 of the Nomad OSS Workshop

![:scale 15%](https://hashicorp.github.io/field-workshops-assets/assets/logos/logo_nomad.png)

???
This is a title slide for the skeleton of a second part of a Nomad OSS workshop.

---
layout: true

.footer[
- Copyright © 2019 HashiCorp
- ![:scale 100%](https://hashicorp.github.io/field-workshops-assets/assets/logos/HashiCorp_Icon_Black.svg)
]

---
name: nomad-cluster-basics
class: smaller
# Nomad Cluster Basics
#### In this section, we will cover:
* Define a Server agent
* Define a Client agent
* Client/Server Architecture in a Nomad Realm
* Check Nomad Server and Client Status
* Use the Nomad UI
* Review and Launch a Nomad Job
* Review Nomad Job Status

???
This slide lists key features of Nomad

---

name: nomad-cluster-basics
class: smaller
# Nomad Components 
#### In this section, we will cover the following:
* Nomad Server
* Nomad Client
* Nomad Architecture

???
This slide lists key features of Nomad

---
name: nomad-server-config
class: smaller
# Nomad Server 
A Nomad **Server** is a machine that runs the Nomad agent in a region that manages all jobs and clients, run evaluations, and create task allocations.
```config
server {
  enabled          = true
  bootstrap_expect = 3
  server_join {
    retry_join = ["<nomad-server-address>"]
  }
}
```
`enabled` = specifies if this agent should run in server mode<br>
`bootstrap_expect`= specifies the number of server nodes to wait for<br>

???
This slide lists key features of Nomad

---
name: nomad-client-config
class: smaller
# Nomad Client 
A Nomad **Client** is a machine that runs the Nomad agent where tasks can be run on. The agent is responsible for registering with the servers, watching for any work to be assigned and executing tasks

```config
client {
  enabled = true
  servers = ["<nomad-server-address>"]
}
```
`enabled` = specifies if client mode is enabled<br>
`servers` = array of addresses of Nomad servers this client should joins

???
This slide lists key features of Nomad

---
name: client-server-architecture
class: img-right, smaller
# Client/Server Architecture
.center[![:scale 120%](images/Nomad_Simple_Cluster_Topology.png)]

- We'll be deploying a simple Nomad Cluster to illustrate a running system and show the process of a Nomad job lifecycle.
- This is going to be a single Nomad Server instance and two Nomad Client instances to handle the jobs.
- No installs on your part! 

???
This slide lists key features of Nomad

---
name: basic-cluster-operations
class: smaller
# Important Nomad Commands

- Run a single Nomad agent in development mode <br>
    `sudo nomad agent -dev`
- Check your version of Nomad <br>
    `nomad version`
- Check the Nomad server(s) status <br>
    `nomad server members`
- Check the Nomad node(s) status <br>
    `nomad node status`
- Enable drain mode on the local node <br>
    `nomad node drain -enable -self`

???
This slide lists key features of Nomad

---
name: simple-cluster-recap
class: smaller
# Simple Cluster Recap
In this section, we:
- Defined the components of a Nomad Cluster
- Reviewed the configuration of a Server agent
- Reviewed the configuration of a Client agent
- Learned the basics of a Nomad Client/Server Architecture
- Run simple Nomad Cluster commands to verify the cluster's health

???
This slide lists key features of Nomad

---
name: job-specification
class: smaller
# Nomad Job Specification
- The job specification (jobspec) defines the deployment schema for the application
- Includes the `tasks`, `images`, `resources`, `priorities`, `constraints`, `service` registrations, `secrets` and other information required to deploy the application
- Specified in the Hashicorp Configuration Language (HCL)

???
This slide lists key features of Nomad
---
name: job-spec-anatomy
class: smaller
# Anatomy of a Job File

- **Job** 
    - Declarative specification of tasks that Nomad should run
- **Group**
    - Series of tasks that should be co-located on the same Nomad client
- **Tasks**
    - An individual unit of work, such as a Docker container, web application, or batch processing

???
This slide lists key features of Nomad

---
name: job-spec-hierarchy
class: compact, smaller
# Job Specification Hierarchy

- Each job file can only a single job that may contain multiple groups
- In each group, you can define multiple tasks

```jobspec
job "example" {
    ...
    group "db" {
        ...
        task "postgres" { ... }
        task "mysql" { ... }
    }
    group "web" {
        task "tomcat" { driver = "docker" }
    }
}
```
???
This slide lists key features of Nomad

---
name: running-job
class: compact, smaller
# Running a Job
To generate a sample job file, run:

```nomadcmd
*nomad job init -short
 Example job file written to example.nomad
```
In this example job file, we have declared a single task 'redis' which is using the Docker driver to run the task. To run the job and deploy this task:
```nomadcmd
*nomad job run example.nomad
 ==> Monitoring evaluation "13ebb66d"
     Evaluation triggered by job "example"
    ...
```

???
This slide lists key features of Nomad

---
name: inspect-running-job
class: compact, smaller
# Inspect a Running Job

To inspect the status of the job `example` we just deployed, simply run: <br>
```nomadcmd
*nomad status example
 ID            = example
 Name          = example
 Submit Date   = 12/30/19 07:50:31 UTC
 Type          = service
 ...
Allocations
ID        Node ID   Task Group  Version  Desired  Status   Created  Modified
*4ef94ade  234c769f  cache       0        run      running  2m ago   2m ago 
```
Take note of the ***Allocation ID***, we'll need it for reference later.
???
This slide lists key features of Nomad

---
name: inspect-allocation-job
class: compact, smaller
# Inspect Task Allocation
To review a task allocation on a the running node: <br>
`nomad alloc status <allocation id>` <br>
```nomadcmd
*nomad alloc status 4ef94ade
*ID                  = 4ef94ade
Eval ID             = 13ebb66d
Name                = example.cache[0]
*Node ID             = 234c769f
Job ID              = example
...
```
We'll use the ***Allocation ID*** from the status output to retrieve the state of the allocation and its current usage.

???
This slide lists key features of Nomad

---
name: stop-job
class: compact, smaller
# Stopping a Job
Finally, the last part of the job lifecycle is to stop the running job and to deallocate the resources:
```
*nomad job stop example
==> Monitoring evaluation "43bfe672"
    Evaluation status changed: "pending" -> "complete"
==> Evaluation "43bfe672" finished with status "complete"
```


???
This slide lists key features of Nomad

---
name: lab-exercise
# 👩‍💻 Lab Challenge 4: Deploy a Simple Nomad Cluster

In this lab you'll learn how to deploy a simple Nomad cluster, configure both server and client agents, perform node discovery and perform the cluster operations to verify everything is working.

You'll learn how to deploy a simple Job file, review its deployment status, perform modifications to the deployment and stopping the job.

Follow the Instruqt Track URL:
https://instruqt.com/hashicorp/tracks/nomad-simple-cluster

---
name: simple-cluster-recap
# Nomad Job Deployment 
In this section, we:
- Learned the basics of defining a Nomad Job specification
- Reviewed the Job, Groups, and Tasks hierarchy
- Initialized and deployed a job
- Reviewed how to monitor the job status, allocation and logs
- Modified and redeployed a job
- Stopped a Job 

???
This slide lists key features of Nomad

