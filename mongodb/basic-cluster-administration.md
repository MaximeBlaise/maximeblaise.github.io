# Basic Cluster Administration

[Home](../README.md) > [MongoDB](./readme.md) > [Basic Cluster Administration](./basic-cluster-administration.md)

Units | Topics
--- | ---
[Chapter 0: Introduction](#chapter-0-introduction) | An overview of the course content.

## Chapter 0: Introduction

### Introduction to Replication and Sharding

Topics covered :

- Architecture of a sharded cluster
- Query handling in a sharded cluster
- Data distribution method

### Setting Up the Vagrant Environment

Why are we using a virtual environment ?

- Sandbox environment for the course
- Avoid dependency and system troubleshooting
  - Focus on learning
- Provide more consistent support

```powershell
vagrant up
vagrant provision
vagrant ssh
```

Vagrant files can be found in [MongoDB University courses](https://university.mongodb.com/).
