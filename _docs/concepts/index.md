---
title: Roles
overview: 
index: true

order: 20 

layout: docs2020
type: markdown

---

There are several roles which are useful for describing how SLATE works. These are not exclusive; one person may fill several roles at different times. 

## Cluster Administrator

A person who runs an edge Kubernetes cluster that is registered with a SLATE federation. SLATE is granted only limited access to participating clusters, so cluster administrators retain overall control of their resources. Cluster administrators are responsible for keeping their clusters running and accessible to SLATE (as long as they choose to participate in the federation). 

## Application Administrator

A person who operates one or more service applications on SLATE is considered an Application Administrator. Application Administrators are responsible for keeping their services running, although there can be circumstances in which they may need to confer with the Cluster Administrator for the cluster on which they are running or want to run. 

## Application Developer

For SLATE purposes, this is a person who packages and maintains an application intended for use on SLATE. This does not require having written the application in the first place. New applications and changes made by an Application Developer are reviewed and approved by an Application Reviewer. 

## Application Reviewer

Application Reviewers are responsible for examining additions and changes to the catalog of applications available to be run by SLATE to ensure that they work and have appropriate security considerations. At this time application review is performed by the SLATE developers. 

## Platform Administrator

SLATE Platform Administrators operate a central SLATE API server and web console. For the reference platform this is done by the SLATE team. They are responsible for ensuring that these central services work correctly, and mediate among the concerns of other participants in the platform. 

