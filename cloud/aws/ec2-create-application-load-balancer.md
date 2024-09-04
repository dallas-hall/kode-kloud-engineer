# Create EC2 Instance Application Load Balancer

## Task

> The Nautilus DevOps team is currently working on setting up a simple application on the AWS cloud. They aim to establish an Application Load Balancer (ALB) in front of an EC2 instance where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later.
> 1. Set up an Application Load Balancer named `devops-alb`.
> 2. Create a target group named `devops-tg`.
> 3. Create a security group named `devops-sg` to open port `80` for the public.
> 4. Attach this security group to the ALB.
>
> The ALB should route traffic on port 80 to port 80 of the `devops-ec2` instance. Make appropriate changes in the default security group attached to the EC2 instance if necessary.

## Research

* Application Load Balancer:
  * https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html
  * https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancer-getting-started.html
* Target Groups
  * https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html
* Security Groups:
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-security-groups.html#creating-security-group
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html

![Alt text](images/alb.png)

## Steps

For GUI, follow the steps at https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancer-getting-started.html.

```bash
# Test the Load Balancer
curl -v http://devops-alb-gui-118637334.us-east-1.elb.amazonaws.com/
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
*   Trying 44.195.114.217:80...
* Connected to devops-alb-gui-118637334.us-east-1.elb.amazonaws.com (44.195.114.217) port 80 (#0)
> GET / HTTP/1.1
> Host: devops-alb-gui-118637334.us-east-1.elb.amazonaws.com
> User-Agent: curl/7.74.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Wed, 04 Sep 2024 23:33:55 GMT
< Content-Type: text/html
< Content-Length: 615
< Connection: keep-alive
< Server: nginx/1.24.0
< Last-Modified: Fri, 13 Oct 2023 13:33:26 GMT
< ETag: "65294726-267"
< Accept-Ranges: bytes
<
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host devops-alb-gui-118637334.us-east-1.elb.amazonaws.com left intact
```

</details>

For CLI, follow the steps at https://docs.aws.amazon.com/elasticloadbalancing/latest/application/tutorial-application-load-balancer-cli.html

