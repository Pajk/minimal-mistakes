---
title: SSL Certificate on AWS ELB using Kubernetes
tags: kubernetes kops aws elb https
excerpt: How to setup HTTPS for load balancer when using Kubernetes on AWS
---

This is how you can setup a load balanced Kubernetes service on AWS with the ELB listening on 443 HTTPS port and forwarding requests to the service running on non-https port.

I had to digg in [GitHub issues](https://github.com/kubernetes/kubernetes/pull/23495) to find out how to do it but I later disovered it hidden [deep in Kubernetes user guides](https://kubernetes.io/docs/user-guide/services/).

The certificate has to be first created in AWS Certificate Manager (through AWS Console, AWS CLI, Terraform etc.) or you can import your own certificate to aws. Either way you will end up with it's ARN.

There are two special annotations that has to be set to make Kubernetes use this certificate in created ELB.

`service.beta.kubernetes.io/aws-load-balancer-ssl-cert` has to be set to the certificate ARN.

`service.beta.kubernetes.io/aws-load-balancer-backend-protocol` tells what protocol should be used for communication between ELB and your service. Can be one of `http|https|ssl|tcp`.

In this example the ELB will listen on port 443 and forward requests to a service listening on port 8080.

{% highlight yaml %}

kind: Service
apiVersion: v1
metadata:
  name: {service-name}
  annotations:
    dns.alpha.kubernetes.io/external: {service.kubernetes.example.com}
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:eu-central-1:{account-id}:certificate/{certificate-id}
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
spec:
  selector:
    app: {app-name}
    tier: frontend
  ports:
    - name: "https"
      protocol: "TCP"
      port: 443
      targetPort: 8080
  type: LoadBalancer

{% endhighlight %}

You might noticed another annotation `dns.alpha.kubernetes.io/external`. That can be used when you have [Route53 Mapper addon](https://github.com/kubernetes/kops/blob/master/docs/addons.md#route53-mapper) installed to automatically create a DNS cname record pointing to the ELB.

