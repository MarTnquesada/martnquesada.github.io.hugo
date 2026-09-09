---
title: 'Parkour with Airflow 3 on Kubernetes'
date: 2026-09-06
Tags: [English]
Categories: [article]
draft: true
---

Last summer started actively working with Airflow 3 deployed in Kubernetes. I had used other orchestrators as well as Airflow 2 before, but this new setup was somewhat different because we started using Airflow heavily as a runner for Python-based pipelines.

Ther


## Dynamic mapping

In Airflow 3 for Kubernetes, each new task uses its own pod. Pods have a nice, chunky cold start (15-30s in my setup). Even if you are better than me at designing images (there is a high % of that), spinning up a container has a noticeable overhead. As a result,  scheduling a large number of dynamically mapped tasks can result in throttling. This can be mitigated somewhat by setting kubernetes’ `worker_pods_creation_batch_size` to a lower value, but it’s not really going to fix large fan-outs.  

With a good amount of care and love put into your images and k8s setup I would guess that you can get better dynamic mapping performance, but why? The three things that you largely get from dynamic mapping are nicer-looking code, very high isolation and more granual partial completions. Being instances of the same task, I still have not found a very good case for the high isolation level. More granular failures can save you money and some time but unless the DAG happens to have really long-running tasks making use of super expensive resources, I do not see much of a point either.

So my recommendation is to be very mindful of whether you use case actually benefits from dynamic mapping as opposed to simply using sequential processing in a single task or local multithreading. If you find a good dynamic mapping application please shoot me an email.

In contrast, when using CeleryExecutor, the tasks are instead system processes (each worker uses a prefork pool with multiple tasks). Therefore, dynamic mapping has pretty much no overhead.


## DAG testing


## Scaling


## Hooks

Prefer using hooks vs implementing custom solutions: https://airflow.apache.org/registry/ (it's great do have your local LM of preference look into this registry by default when working on Airflow projects).  


## Inter-task data passing

Storing inter-task information in cold storage (S3) and passing through XCom only cold storage urls has been great for me because it allows for any kind of data serialization and simplifies debugging. This can be achieved in a couple of different ways, like using the specific S3-backed XCom backend variant. But my favourite so far has been using Airflow's object storage, which 


## Carousel of AI suggestions that will raise your blood pressure 

Claude in particular (though I have seen it with other models too) loves to add a quick
```python
data_interval_end = data_interval_end or datetime.now(UTC)
```
whenever you have time-dependent behaviour, where `data_interval_end` is a task parameter filled by Airflow. 
This is terrible because you would pretty much always expect it to have an actual value, except for
Even when the latter happens, you would much rather have the run fail, so that you can figure out what is wrong with the system, instead of completely changing your time logic as a failover (bibbidi bobbidi boo, no DAG idempotency for you). 



Til next time.
