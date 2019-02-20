---
layout: post
title: Docker-in-Docker builds in TeamCity agents on AWS ECS
categories: tech
tags: docker teamcity aws ecs
---

I have been experimenting with running [TeamCity](https://www.jetbrains.com/teamcity/) in
[AWS](https://aws.amazon.com/), using the [CloudFormation](https://aws.amazon.com/cloudformation/)
stack [provided by JetBrains](https://confluence.jetbrains.com/display/TCD18/Running+TeamCity+Stack+in+AWS).
This stack sets up [EC2](https://aws.amazon.com/ec2/) instances to host [Docker](https://docker.io/)
containers in [EC2](https://aws.amazon.com/ecs/).

However the default configuration does not allow Docker-in-Docker builds: where a TeamCity agent,
itself running in a Docker container, needs to run a build step with Docker or
[Docker Compose](https://docs.docker.com/compose/). The [page for the JetBrains Docker image for
agents](https://hub.docker.com/r/jetbrains/teamcity-agent) gives two options for starting the agent
container:

* Docker from the host
* Run in privileged mode

This post is about how to modify the [JetBrains CloudFormation
template](https://s3.amazonaws.com/teamcity.jetbrains.com/teamcity-server.yaml).

# Docker from the host

This technique maps `/var/run/docker.sock` from the Docker host into the running container:

```yaml
  AgentTaskDefinition:
      Type: AWS::ECS::TaskDefinition
      Condition: ShouldLaunchAgents
      DependsOn:
        - PublicLoadBalancer
        - TCServerNodeService
      Properties:
        PlacementConstraints:
          - Type: memberOf
            Expression: attribute:teamcity.node-responsibility == buildAgent
        # Define the host volume to map.
        Volumes:
          - Name: "dockerSock"
            Host:
              SourcePath: "/var/run/docker.sock"
        ContainerDefinitions:
          - Name: 'teamcity-agent'
            Image: !Join [':', ['jetbrains/teamcity-agent', !Ref 'TeamCityVersion']]
            Cpu: !Ref AgentContainerCpu
            Memory: !Ref AgentContainerMemory
            Essential: true
            Environment:
              - Name: SERVER_URL
                Value: "https://teamcity.tawh.net"
            LogConfiguration:
              LogDriver: 'awslogs'
              Options:
                awslogs-group: !Ref ECSLogGroup
                awslogs-region: !Ref AWS::Region
                awslogs-stream-prefix: 'aws/ecs/teamcity-agent'
            # Mount the host volume
            MountPoints:
              - ContainerPath: "/var/run/docker.sock"
                SourceVolume: "dockerSock"
```

# Run in privileged mode

Set privileged mode in the ECS Task defintion for the agent by adding `Privileged: true` to the
container definitions:

```yaml
  AgentTaskDefinition:
      Type: AWS::ECS::TaskDefinition
      Condition: ShouldLaunchAgents
      DependsOn:
        - PublicLoadBalancer
        - TCServerNodeService
      Properties:
        PlacementConstraints:
          - Type: memberOf
            Expression: attribute:teamcity.node-responsibility == buildAgent
        ContainerDefinitions:
          - Name: 'teamcity-agent'
            Image: !Join [':', ['jetbrains/teamcity-agent', !Ref 'TeamCityVersion']]
            Cpu: !Ref AgentContainerCpu
            Memory: !Ref AgentContainerMemory
            # Run this container in privileged mode.
            Privileged: true
            Essential: true
            Environment:
              - Name: SERVER_URL
                Value: !GetAtt [PublicLoadBalancer, DNSName]
            LogConfiguration:
              LogDriver: 'awslogs'
              Options:
                awslogs-group: !Ref ECSLogGroup
                awslogs-region: !Ref AWS::Region
                awslogs-stream-prefix: 'aws/ecs/teamcity-agent'
```
