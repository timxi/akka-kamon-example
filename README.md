# akka-kamon-example
This is a sample project that plugs a simple cluster app with metrics capabilities using Kamon.

## What I did :
 * Add the aspectj plugin in project/plugins.sbt
 * In build.sbt, imported SbtAspectj._
 * In build.sbt, Added all the kamon modules I'm interested in to the dependencies (core, akka, akka-remote, system-metrics)
Note that kamon-statsd and kamon-log-reporter are backend modules, to which the metrics are sent.
 * In build.sbt, added Keys.fork in run := true (aspectj requires this)
 * In build.sbt, added Keys.fork in run := true, added javaOptions in run  <++=  AspectjKeys.weaverOptions in Aspectj
 * In every of your Main programs, call Kamon.start() before instantiating actor systems.
 * in src/main/resources/application.conf, added a bunch of Kamon-related configuration. Note that since I'm on a mac,
the IP I provide to the statsd module is the one belonging to my boot2docker virtual machine. 

The use of AspectJ is not compulsory, but if you want pre-made metrics for your actor systems,
you'll have to use it, as Kamon uses aspect-oriented programming to add metrics to some low-level
functions of akka actors.

About the kamon-akka-remote module : 

"the kamon-akka-remote module doesn't bring any new metrics to the table, all it does is ensure that when messages go 
over the remoting bridge they will keep a reduced version of the same trace context
meaning that for example, the trace-token is kept and propagated to other nodes
and the trace might be finished somewhere else, but no special metrics are added"


## Steps get this running and display some cool stuff :

Kamon provides a docker image with statds/grafana : [https://github.com/kamon-io/docker-grafana-graphite] . 
Use the following command to get it and run it

```
docker run -d -p 80:80 -p 8125:8125/udp -p 8126:8126 --name kamon-grafana-dashboard kamon/grafana_graphite
```

In a browser, open localhost:80 (or the address on which your statsd server is running, which could be the one of 
your boot2docker VM if you're on Mac). A pretty neat dashboard should appear. 

In five different terminals, start the following programs :
 * sbt "runMain sample.cluster.transformation.TransformationBackend 2552"		
 * sbt "runMain sample.cluster.transformation.TransformationFrontend 2551"
 * sbt "runMain sample.cluster.transformation.TransformationBackend 0"	
 * sbt "runMain sample.cluster.transformation.TransformationBackend 0"	
 * sbt "runMain sample.cluster.transformation.TransformationFrontend 0"
 
Now, on the dashboard, the hostname of your machine should appear (which means that your programs successfully sent
metrics to statsd). If it doesn't, try switching from the akka dashboard to the OS one (there's an icon on the top right
that looks like a folder). 

In addition, you should start visualizing metrics without doing anything else. Sometimes the docker container isn't 
 in the same timezone as the browser, which the dashboard should warn you about. 
 
 
