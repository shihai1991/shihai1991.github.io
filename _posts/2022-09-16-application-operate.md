---
layout: post
title: 云原生应用管理
category: iac
catalog: true
published: true
tags:
  - iac
time: '2022.09.16 11:34:00'
---
# 一、[provisioning](https://www.redhat.com/en/topics/automation/what-is-provisioning#overview)
`provisioning`是设置IT基础设施的过程。`provisioning`和配置不是一个事情，但是他们都是`deployment`过程中的步骤。有多种类别的`provisioning`：`server provisioning`,`network provisioning`,`user provisioning`,`service provisioning`。  
`provisioning`和`iac`是什么关系呢？我个人理解前者是目标，后者是实施方案之一。

# 二、[traits](https://github.com/oam-dev/spec/blob/master/6.traits.md)
这个概念是`open application model`提出的。`traits`的目的是想通过此配置对分布式应用进行控制配置，示例配置：
```
apiVersion: core.oam.dev/v1beta1
kind: Application
metadata:
  name: my-example-app
  annotations:
    version: v1.0.0
    description: "Brief description of the application"
spec:
  components:
    - name: publicweb
      type: web-ui
      properties: # properties targeting component parameters.
        image: example/web-ui:v1.0.2@sha256:verytrustworthyhash
        param_1: "enabled" # param_1 is defined on the web-ui component
      traits:
        - type: ingress # ingress trait providing a public endpoint for the publicweb component of the application.
          properties: # properties are defined by the trait CRD spec. This example assumes path and port.
            path: /
            port: 8080
      scopes:
        "healthscopes.core.oam.dev": "app-health" # An application level health scope including both components.
    - name: backend
      type: company/test-backend # test-backend is referenced from other namespace
      properties:
        debug: "true" # debug is a parameter defined in the test-backend component.
      traits:
        - type: scaler # scaler trait to specify the number of replicas for the backend component
          properties:
            replicas: 4
      scopes:
        "healthscopes.core.oam.dev": "app-health" # An application level health scope including both components.
```
traits对象定义如下所示。
```
  trait:
    type: object
    description: The trait section defines traits that will be used in a component
      instance.
    properties:
      name:
        type: string
        description: The name of the trait instance. Must be unique to the deployment
          environment.
      properties:
        type: object
        description: Overrides of parameters that are exposed by the trait type defined
          in 'type'.
        additionalProperties:
          "$ref": "#/definitions/propertiesObject"
    required:
    - name
    additionalProperties: false
  propertiesObject:
    type: object
    description: A properties object (for trait and scope configuration) is an object
      whose structure is determined by the trait or scope property schema. It may
      be a simple value, or it may be a complex object.
    properties: {}
    additionalProperties: true
```