Teamcity Server/Agent Deployment Terraform Module
=================================================

This module can be used to deploy a teamcity server and one or multiple build agents to your terraform-managed infrastructure:

```js
{
    "module": {
        "buildserver": {
            "source": "github.com/snatalenko/tf_teamcity",

            "aws_account_id": "${var.aws_account_id}",
            "vpc_id": "${var.vpc_id}",
            "subnet_id": "${var.subnet_id}",

            // using same cluster for buildserver and agent,
            // so that both tasks can be hosted on a same instance
            "server_cluster_id": "${aws_ecs_cluster.buildserver.id}",
            "agent_cluster_id": "${aws_ecs_cluster.buildserver.id}",

            "server_image": "jetbrains/teamcity-server:2017.1.5",
            "agent_image": "${aws_ecr_repository.teamcity_agent.repository_url}:2017.1.5",

            // the hostname will be used by agent to connect to the server
            "server_hostname": "${aws_route53_record.buildserver.name}",

            // if more than 1 agent will be used,
            // they will need to be placed to a separate aws_ecs_cluster
            // with a corresponding number of aws_instance's
            "agents_count": 1
        },
    },
    "resource": {
        "aws_ecs_cluster": {
            "buildserver": {
                "name": "myapp-buildserver"
            }
        },
        "aws_ecr_repository": {
            // ECR repository for your customized build agent image
            "teamcity_agent": {
                "name": "myapp/teamcity-nodejs-agent"
            }
        },
        "aws_instance": {
            "buildserver": {
                // ...
                // your buildserver instance configuration
                // ...
                "iam_instance_profile": "${module.buildserver.instance_profile_name}",
                "vpc_security_group_ids": [
                    "${module.buildserver.security_group_id}"
                ]
            }
        },
        "aws_route53_record": {
            "buildserver": {
                "zone_id": "${var.route53_zone_id}",
                "name": "buildserver.myapp.local",
                "type": "A",
                "ttl": "300",
                "records": [
                    "${aws_instance.buildserver.private_ip}"
                ]
            }
        }
    }
}
```