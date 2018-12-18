# Lab 04 - Docker

## Table of content


1. [Identify issues and install the tools](#task-0)
2. [Add a process supervisor to run several processes](#task-1)
3. [Add a tool to manage membership in the web server cluster](#task-2)
4. [React to membership changes](#task-3)
5. [Use a template engine to easily generate configuration files](#task-4)
6. [Generate a new load balancer configuration when membership changes](#task-5)
7. [Make the load balancer automatically reload the new configuration](#task-6)
8. Difficulties
9. Conclusion



## Tasks

### <a name="task-0"></a>Task 0: Identify issues and install the tools

1. <a name="M1"></a>**[M1]** Do you think we can use the current solution for a production environment? What are the main problems when deploying it in a production environment?

2. <a name="M2"></a>**[M2]** Describe what you need to do to add new `webapp` container to the infrastructure. Give the exact steps of what you have to do without modifiying the way the things are
   done. Hint: You probably have to modify some configuration and script files in a Docker image.

3. <a name="M3"></a>**[M3]** Based on your previous answers, you have detected some issues in the current solution. Now propose a better approach at a high level.

4. <a name="M4"></a>**[M4]** You probably noticed that the list of web application nodes is hardcoded in the load balancer configuration. How can we manage the web app nodes in a more dynamic
     fashion?

5. <a name="M5"></a>**[M5]** In the physical or virtual machines of a typical infrastructure we tend to have not only one main process (like the web server or the load balancer) running, but a few
   additional processes on the side to perform management tasks.

   For example to monitor the distributed system as a whole it is common to collect in one centralized place all the logs produced by the different machines. Therefore we need a process running on each
   machine that will forward the logs to the central place. (We could also imagine a central tool that reaches out to each machine to gather the logs. That's a push vs. pull problem.) It is quite common to see a push mechanism used for this kind of task.

   Do you think our current solution is able to run additional management processes beside the main web server / load balancer process in a container? If no, what is missing / required to reach the goal? If yes, how to proceed to run for example a log forwarding process?

6. <a name="M6"></a>**[M6]** In our current solution, although the load balancer configuration is changing dynamically, it doesn't follow dynamically the configuration of our distributed system when
   web servers are added or removed. If we take a closer look at the `run.sh` script, we see two calls to `sed` which will replace two lines in the `haproxy.cfg` configuration file just before we start `haproxy`. You clearly see that the configuration file has two lines and the script will replace these two lines.

   What happens if we add more web server nodes? Do you think it is really dynamic? It's far away from being a dynamic configuration. Can you propose a solution to solve this?



**Deliverables**:

1. Take a screenshot of the stats page of HAProxy at <http://192.168.42.42:1936>. You should see your backend nodes.

   ![01a](.\01a.png)

   ![01b](.\01b.png)

2. Give the URL of your repository URL in the lab report.

   https://github.com/Yannis100/Teaching-HEIGVD-AIT-2016-Labo-Docker



### <a name="task-1"></a>Task 1: Add a process supervisor to run several processes

> In this task, we will learn to install a process supervisor that will help us to solve the issue presented in the question [M5](#M5). Installing a process supervisor gives us the ability to run multiple processes at the same time in a Docker environment.

**Deliverables**:

1. Take a screenshot of the stats page of HAProxy at <http://192.168.42.42:1936>. You should see your backend nodes. It should be really similar to the screenshot of the previous task.
2. Describe your difficulties for this task and your understanding of what is happening during this task. Explain in your own words why are we installing a process supervisor. Do not hesitate to do more research and to find more articles on that topic to illustrate the problem.



### <a name="task-2"></a>Task 2: Add a tool to manage membership in the web server cluster

> Installing a cluster membership management tool will help us to solve the problem we detected in [M4](#M4). In fact, we will start to use what we put in place with the solution to issue [M5](#M5). We will build two images with our process supervisor running the cluster membership management tool `Serf`.

When we reach this point, we have a problem. If we start the HAProxy first, it will not start as the two `s1` and `s2` containers are not started and we try to link them through the Docker `run` command.

You can try and get the logs. You will see error logs where `s1` and `s2` If we start `s1` and `s2` nodes before `ha`, we will have an error from `Serf`.
They try to connect the `Serf` cluster via `ha` container which is not running.

So the reverse proxy is not working but what we can do at least is to start the containers beginning by `ha` and then backend nodes. It will make the `Serf` part working and that's what we are working on at the moment and in the next task.

Deliverables**:

1. Provide the docker log output for each of the containers: `ha`, `s1` and `s2`. You need to create a folder `logs` in your repository to store the files separately from the lab report. For each lab task create a folder and name it using the task number. No need to create a folder when there are no logs.

   Example:

   ```
   |-- root folder
     |-- logs
       |-- task 1
       |-- task 3
       |-- ...
   
   ```

2. Give the answer to the question about the existing problem with the current solution. (Anyway, in our current solution, there is kind of misconception around the way we create the `Serf` cluster. In the deliverables, describe which problem exists with the current solution based on the previous explanations and remarks. Propose a solution to solve the issue.

   When we reach this point, we have a problem. If we start the HAProxy first, it will not start as the two `s1` and `s2` containers are not started and we try to link them through the Docker `run` command.

   You can try and get the logs. You will see error logs where `s1` and `s2` If we start `s1` and `s2` nodes before `ha`, we will have an error from `Serf`.
   They try to connect the `Serf` cluster via `ha` container which is not running.

   So the reverse proxy is not working but what we can do at least is to start the containers beginning by `ha` and then backend nodes. It will make the `Serf` part working and that's what we are working on at the moment and in the next task.)

3. Give an explanation on how `Serf` is working. Read the official website to get more details about the `GOSSIP` protocol used in `Serf`. Try to find other solutions that can be used to solve similar situations where we need some auto-discovery mechanism.



### <a name="task-3"></a>Task 3: React to membership changes

> Serf is really simple to use as it lets the user write their own shell scripts to react to the cluster events. In this task we will write the first bits and pieces of the handler scripts we need to build our solution.
  We will start by just logging members that join the cluster and the members that leave the cluster. We are preparing to solve concretely the issue discovered in [M4](#M4).

**Deliverables**:

1. Provide the docker log output for each of the containers:  `ha`, `s1` and `s2`.
   Put your logs in the `logs` directory you created in the previous task.
2. Provide the logs from the `ha` container gathered directly from the `/var/log/serf.log`
   file present in the container. Put the logs in the `logs` directory in your repo.



### <a name="task-4"></a>Task 4: Use a template engine to easily generate configuration files

> We have to generate a new configuration file for the load balancer each time a web server is added or removed. There are several ways to do this. Here we choose to go the way of templates. In this task we will put in place a template engine and use it with a basic example. You will not become an expert in template engines but it will give you a taste of how to apply this technique which is often used in other contexts (like web templates, mail templates, ...).
  We will be able to solve the issue raised in [M6](#M6).

**Deliverables**:

1. You probably noticed when we added `xz-utils`, we have to rebuild the whole image which took some time. What can we do to mitigate that? Take a look at the Docker documentation on [image layers](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/#images-and-layers).
   Tell us about the pros and cons to merge as much as possible of the command. In other words, compare:

```
  RUN command 1
  RUN command 2
  RUN command 3

```

  vs.

```
  RUN command 1 && command 2 && command 3

```

  There are also some articles about techniques to reduce the image size. Try to find them. They are talking about `squashing` or `flattening` images.

1. Propose a different approach to architecture our images to be able to reuse as much as possible what we have done. Your proposition should also try to avoid as much as possible repetitions between
   your images.

2. Provide the `/tmp/haproxy.cfg` file generated in the `ha` container after each step.  Place the output into the `logs` folder like you already did for the Docker logs in the previous tasks. Three files are expected.

   In addition, provide a log file containing the output of the `docker ps` console and another file (per container) with`docker inspect <container>`. Four files are expected.

3. Based on the three output files you have collected, what can you say about the way we generate it? What is the problem if any?



### <a name="task-5"></a>Task 5: Generate a new load balancer configuration when membership changes

> We now have S6 and Serf ready in our HAProxy image. We have member join/leave handler scripts and we have the handlebars template engine. So we have all the pieces ready to generate the HAProxy configuration dynamically. We will update our handler scripts to manage the list of nodes and to generate the HAProxy configuration each time the cluster has a member leave/join event.
  The work in this task will let us solve the problem mentioned in [M4](#M4).

**Deliverables**:

1. Provide the file `/usr/local/etc/haproxy/haproxy.cfg` generated in the `ha` container after each step. Three files are expected.

   In addition, provide a log file containing the output of the `docker ps` console and another file (per container) with `docker inspect <container>`. Four files are expected.

2. Provide the list of files from the `/nodes` folder inside the `ha` container.
   One file expected with the command output.

3. Provide the configuration file after you stopped one container and the list of nodes present in the `/nodes` folder. One file expected with the command output. Two files are expected.

    In addition, provide a log file containing the output of the `docker ps` console. One file expected.

4. (Optional:) Propose a different approach to manage the list of backend nodes. You do not need to implement it. You can also propose your own tools or the ones you discovered online. In that case, do not forget to cite your references.


### <a name="task-6"></a>Task 6: Make the load balancer automatically reload the new configuration

> Finally, we have all the pieces in place to finish our solution. HAProxy will be reconfigured automatically when web app nodes are leaving/joining the cluster. We will solve the problems you have discussed in [M1 - 3](#M1).
  Again, the solution built in this lab is only one example of tools and techniques we can use to solve this kind of situation. There are several other ways.

**Deliverables**:

1. Take a screenshots of the HAProxy stat page showing more than 2 web applications running. Additional screenshots are welcome to see a sequence of experimentations like shutting down a node and starting more nodes.

   Also provide the output of `docker ps` in a log file. At least one file is expected. You can provide one output per step of your experimentation according to your screenshots.

2. Give your own feelings about the final solution. Propose improvements or ways to do the things differently. If any, provide references to your readings for the improvements.

3. (Optional:) Present a live demo where you add and remove a backend container.



## Difficulties

Task 2 :

Pleins de soucis et différentes tentatives de modifier le fichier haproxy.cfg car les nodes backends apparaissent down dans l'interface web haproxy.

Les nodes backend se "présentent" au cluster serf avec le nom de container et non s1 et s2, de plus suite à la création du network spécifique, s1 et s2 ne sont pas résolus dans /etc/hosts tandis qu'avec le network par défaut (bridge) et les paramètres `--link` de la commande run de ha on les retrouve dans le fichier hosts, dans le cas où l'on spécifie notre network crée, les paramètres --link ne semblent n'avoir aucun effet.

Néanmoins on constate que dans le réseau par défaut 'bridge', le nameserver est hérité de virtualbox (/etc/resolv.conf 10.0.2.3) et sans les --link les container n'arriveront pas à se résoudre parmis. Avec notre network 'heig', le nameserver est 127.0.0.11 et résoud bien s1, s2 et ha ainsi que les noms des containers respectifs.

Finalement nous avons entré l'IP en statique dans haproxy.cfg

Task 3 : 

Après la configuration du dockerfile pour copier le run de serf, les fichiers sont bien copiés et au bon endroit, néanmoins des erreurs :

```bash
: No such file or directory bash
```

C'était dû aux retours à la ligne au format windows et cela empechait serf de démarrer, il a suffit convertir les sauts de ligne en format Unix

## Conclusion