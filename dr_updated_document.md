# Infrastructure

## AWS Zones
For zone1: us-east-2a, us-east-2b, us-east=2c For zone2: us-west-1a,us-west-1c

## Servers and Clusters

### Table 1.1 Summary
| Asset                            | Purpose                                                                                                                                             | Size           | Qty | DR                                                                   |
|----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|----------------|-----|----------------------------------------------------------------------|
| i-01779dd7ba4732057              | EC2 instance, EKS cluster node, AMI: amazon-eks-node-1.22-v20230304                                                                                 | t3 medium      | 1   | Yes, has to be replicated. Runs in a single AZ.                      |
| i-0cceff034032253a40             | EC2 instance, EKS cluster node, AMI: amazon-eks-node-1.22-v20230304                                                                                 | t3 medium      | 1   | Yes, has to be replicated. Runs in a single AZ.                      |
| Ubuntu Web                       | Web Server with our Flusk application. OS level is our Linux AMI.                                                                                   | t3 micro       | 1   | Yes, has to be replicated (our webserver) Runs in a single AZ.       |
| udacity-tf-vinoth2               | S3 bucket for the tfstate file. The file stores the current status of the terraform deployment                                                      | S3             | 1   | Yes for storing the tfstate, re-create S3 store, Runs in a single AZ |
| udacity-tf-vinoth2-west          | S3 bucket for the tfstate file. The file stores the current status of the terraform deployment, but for the US-WEST-1 region.                       | S3             | 1   | Yes for storing the tfstate, re-create S3 store, Runs in a single AZ |
| udacity-cluster                  | Elastic Kubernetes cluster with 2 notes. Makes sure that our application is HA. Here is the Grafana and Prometheus stack running.                   | EKS cluster    | 1   | Yes for serving the requests, Runs in a single AZ                    |
| a0aa8a70fce6341759632804720a4a58 | Elastic loadbalancer for performing (together with Route53) a smooth failover to another region.                                                    | ALB            | 1   | Yes for serving the requests, Runs in a single AZ                    |
| udacity-db-cluster               | RDS cluster (mysql_aurora.1.19.1) with one read replica and one wirte replica. The read replica will help to mitigate performance issues in one AZ. | db.t2.small    | 1   | Yes otherwise we cannot store data into our db, multi AZ.            |
| udacity-project                  | VPC for our enviornment. Our virtual private network here are all of our resources are running. All of our network layer operations are happen her. | VPC            | 1   | No, will be recreated after redeployment                             |
| sg-0269adac8fc02a585             | Security group ingress port 22 for getting access to our instances via ssh                                                                          | Security group | 1   | Yes, we need this for SSH access                                     |
| sg-03debe1ed4c50752a             | Security group ingress port 80 the http port for getting access to our webapplication.                                                              | Security group | 1   | Yes, we need this for the HTTPD server                               |
| sgr-0c6b12c8410dcdd26            | Security group ingress port 9100 for getting the performance information from our EC2 instance.                                                     | Security group | 1   | Yes, we need this for the Flusk client at the EC2 instance.          |
| udacity-vinoth                   | AMI image contains the OS which is in use at the different EC2 instances.                                                                           | AMI            | 1   | Yes, this is the baseline for our Linux instances.                   |
| udacity                          | SSH key, for getting direct access to the instances. As a workaround the direct access via the AWS console can be used.                             | N/A            | 1   | SSH key for the instances.                                           |
| Monitoring                       | Monitoring plattform for the web application. We are using Grafana in combination with Prometheus.                                                  | N/A            | 1   | Yes, we need this also in our backup zone.                           |

### Descriptions
Ubuntu web server hosts flask web application.
EKS nodes hosts monitoring platform to monitor the web application.
ALB handles the requests.
RDS cluster holds the web application data.

## DR Plan
### Pre-Steps:
1. Ensure you have alligned configurations at both zones.
2. Deploy the 2nd zone via Terraform.

## Steps:
You won't actually perform these steps, but write out what you would do to "fail-over" your application and database cluster to the other region. Think about all the pieces that were setup and how you would use those in the other region
1. Use DNS (Route53) for doing the failover to the 2nd zone (backup zone). Maybe latency routing could be an option.
2. Promote the first RDS instance to a read replica. (Manual failover to the backup zone).