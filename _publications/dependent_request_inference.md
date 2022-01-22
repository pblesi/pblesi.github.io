---
title: Dependent Request Inference and Execution in Integration Tests
date: '2021-10-29'
author: Patrick Blesi
image: dependency_graph.png
project_links:
  - title: Paper (TD Commons)
    url: https://www.tdcommons.org/dpubs_series/4687/
tags:
  - Software Testing
  - Test Automation
---

## Abstract

The test setup phase of an automated software test for an application programming interface (API) typically requires the creation of many dependent objects. Current techniques for test setup typically require a test writer to explicitly define dependencies between resources, which entails substantial manual effort to build the setup necessary to test an API request.

This disclosure describes techniques to automatically assemble a dependency graph and build the setup and cleanup necessary to test an API request. An API service definition is analyzed to enumerate all remote procedure calls (RPCs) and infer the fields and RPCs required to successfully execute a request. Structured API definitions and heuristics are used to determine whether an API request has a dependency on other API requests. These dependencies are assembled into a dependency graph, the nodes of which are executed after being topologically sorted.
