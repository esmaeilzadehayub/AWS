Deploy a Cluster of Web Servers

Running a single server is a good start, but in the real world, a single server is a single point of failure. If that server crashes, or if it becomes overloaded 
from too much traffic, users will be unable to access your site. The solution 
is to run a cluster of servers, routing around servers that go down, and adjusting the size of the cluster up or down based on traffic
Managing such a cluster manually is a lot of work. Fortunately, you can let
AWS take care of it for by you using an Auto Scaling Group (ASG) An ASG takes care of a lot of tasks for you completely
automatically, including launching a cluster of EC2 Instances, monitoring the health of each Instance, replacing failed Instances, and adjusting the size
of the cluster in response to load.



