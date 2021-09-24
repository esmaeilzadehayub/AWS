Deploy a Cluster of Web Servers

Running a single server is a good start, but in the real world, a single server is a single point of failure. If that server crashes, or if it becomes overloaded 
from too much traffic, users will be unable to access your site. The solution 
is to run a cluster of servers, routing around servers that go down, and adjusting the size of the cluster up or down based on traffic
Managing such a cluster manually is a lot of work. Fortunately, you can let
AWS take care of it for by you using an Auto Scaling Group (ASG) An ASG takes care of a lot of tasks for you completely
automatically, including launching a cluster of EC2 Instances, monitoring the health of each Instance, replacing failed Instances, and adjusting the size
of the cluster in response to load.

![image](https://user-images.githubusercontent.com/28998255/134624073-3a3e0426-9ae0-434f-8c46-2934ea995dd5.png)

**The first step in creating an ASG is to create a launch configuration, which
specifies how to configure each EC2 Instance in the ASG**

create_before_destroy to true on an EC2 Instance, then whenever you
make a change to that Instance, Terraform will first create a new EC2
Instance, wait for it to come up, and then remove the old EC2 Instance

```python

resource "aws_launch_configuration" "example" {
 image_id = "ami-40d28157"
 instance_type = "t2.micro"
 security_groups = ["${aws_security_group.instance.id}"]
 user_data = <<-EOF
 #!/bin/bash
 echo "Hello, World" > index.html
 nohup busybox httpd -f -p "${var.server_port}" &
 EOF
 lifecycle {
 create_before_destroy = true
 }
}
```
**the security group**
```python
resource "aws_security_group" "elb" {
 name = "terraform-example-elb"
 ingress {
 from_port = 80
 to_port = 80
 protocol = "tcp"
 cidr_blocks = ["0.0.0.0/0"]
 }
 egress {
 from_port = 0
 to_port = 0
 protocol = "-1"
 cidr_blocks = ["0.0.0.0/0"]
 }
}
```

**Deploy a Load Balancer**

The ELB has one other trick up its sleeve: it can periodically check the
health of your EC2 Instances and, if an Instance is unhealthy, it will
automatically stop routing traffic to it. To configure a health check for the
ELB, you add a health_check block. For example, here is a health_check block that sends an HTTP request every 30 seconds to the “/” URL of each
of the EC2 Instances in the ASG and only considers an Instance healthy if it
responds with a 200 OK.

```python 

resource "aws_elb" "example" {
 name = "terraform-asg-example"
 availability_zones = ["${data.aws_availability_zones.all.names}"]
 security_groups = ["${aws_security_group.elb.id}"]
 listener {
 lb_port = 80
 lb_protocol = "http"
 instance_port = "${var.server_port}"
 instance_protocol = "http"
 }
 health_check {
 healthy_threshold = 2
 unhealthy_threshold = 2
 timeout = 3
 interval = 30
 target = "HTTP:${var.server_port}/"
 }
}
```
**aws_autoscaling_group**
aws_autoscaling_group resource and set its load_balancers parameter to tell the ASG to register each Instance in the ELB when that Instance is
booting.

Notice that the health_check_type is now "ELB". This tells the ASG to use
the ELB’s health check to determine if an Instance is healthy or not and to
automatically replace Instances if the ELB reports them as unhealthy

```python

resource "aws_autoscaling_group" "example" {
 launch_configuration = "${aws_launch_configuration.example.id}"
 availability_zones = ["${data.aws_availability_zones.all.names}"]
 load_balancers = ["${aws_elb.example.name}"]
 health_check_type = "ELB"
 min_size = 2
 max_size = 10
 tag {
 key = "Name"
 value = "terraform-asg-example"
 propagate_at_launch = true
 }
}
```

