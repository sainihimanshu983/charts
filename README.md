# Charts

This repository is for hosting the helm/rancher charts. For now, it is being used for a rancher chart hosting instructions.

## Usage Intructions
---
## Index
* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
* [Rancher on Docker](#rancher-on-docker)
* [ChartMuseum on Docker](#chartmuseum-on-docker)
* [Integrate ChartMuseum in Rancher](#integrate-chartmuseum-in-rancher)
* [Create Chart](#create-chart) 
* [Package Chart](#cackage-chart) 
* [Publish Chart](#publish-chart) 
* [Update Chart](#update-chart) 
* [Install Chart](#install-chart) 
* [Script](#script)
* [Alternative to ChartMuseum](#alternative-to-chartmuseum)


---

## Introduction
This demo shows how to quickly setup rancher, chartmuseum and create a rancher app. This demo is particularly aimed towards the audience interested to work on rancher. Rancher supports helm 2 charts in classic versions before rancher v2.5. On and aove rancher v2.5, helm 3 is supported.

Please go through the rancher documentation to understand the difference between rancher ___Catalog___(classic) and ___Apps and Marketplace___. These terms are important to understand the environment before starting development on it. These differences also cover the additions to a classic helm chart that creates a rancher chart out of it.

## Prerequisites
Basic knowledge of:
* Rancher
* Helm Chart
* Bash

## Rancher on Docker
With following command you can setup a rancher cluster with K3s on docker. After running this docker, setup the basic config of rancher by visiting the URL exposed. UI should guide you through default setup.
```
sudo docker run --rm -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher:latest
```

## ChartMuseum on Docker
Chartmuseum is a repository for helm charts, it is the underlying structure in production grade applications like harbor and others. It is the easiest de-facto standard with simple APIs to control the helm chart hosting.
```
sudo docker run --rm -d -p 8080:8080 -e STORAGE=local -e STORAGE_LOCAL_ROOTDIR=/charts -v $(pwd)/charts:/charts  chartmuseum/chartmuseum:latest
```

## Integrate ChartMuseum in Rancher
Go to a project where you want to avail helm charts from chartmuseum, and then navigate to catalogs from navigation bar. Add a new catalog and provide the URL of chartmuseum with a name to save it as a new Catalog.

## Create Chart
Follow the below given steps to try out chart creation for rancher:
* Create a new helm chart
    ```
    helm create demo
    ```
* Go to the directory of this chart
    ```
    cd demo
    ```
* Add following content to `Chart.yaml` file
    ```    
    # RANCHER ADDITIONS
    # =================

    # Application website
    home: https://sainihimanshu983.github.io

    # Icon link for Chart logo in catalog.
    icon: https://img.shields.io/badge/Author-sainihimanshu983-success 

    # Metadata
    keywords:
    - demo
    - nginx
    - rancher
    - sainihimanshu983

    # Maintainer/POC for maintenance or queries
    maintainers:
    - email: sainihimanshu983@gmail.com
      name: Himanshu
    ```
* Add `README.md` file at current directory with following content:
    ```
    # Demo Chart
    This is a demo chart README.md file. Visible in detailed description.

    ```
* Add `app-readme.md` file at current directory with following content:
    ```
    # Demo Chart
    This is a demo chart app-readme.md file. Visible as introuctory abstract.

    ```

* Add `question.yml` file at current directory with following content:
    ```
    categories:
    - Microservice
    - Monitoring
    labels:
    org: sainihimanshu983
    category: Microservice
    io.cattle.role: project
    questions:
    - variable: hostname
      required: true
      default: "demo"
      type: string
      label: "Host Node Name"
      description: "SAMPLE: k3snodename"
      group: "Host Configuration"
    - variable: kubeapi
      required: true
      default: "10.0.0.4:6443"
      type: string
      label: "Kubernetes API server URL"
      description: "SAMPLE: k8s-master-hostname:6443"
      group: "Host Configuration"

    # Registry
    - variable: registry.show
      default: true
      type: boolean
      required: true
      label: "Private Registry"
      group: "Registry"
      show_subquestion_if: true
      subquestions:
      - variable: global.image.pullSecret
        required: true
        default: "demo001"
        type: string
        label: "Registry Secret Name"
        description: "NOTE: Create registry secret in namespace first! Leave empty for local/public images."
        group: "Registry"
      - variable: global.image.Registry
        required: true
        default: "demo001.azurecr.io"
        type: string
        label: "Registry Name"
        description: "SAMPLE: abc.azurecr.io"
        group: "Registry"

    # rabbit-mq
    - variable: rabbit-mq.rabbitMQ.image
      required: true
      default: "rabbitmq-infrastructure"
      type: string
      label: "Image Repository"
      group: "Rabbit MQ"
    - variable: rabbit-mq.rabbitMQ.tag
      default: ""
      type: string
      label: "Tag Version"
      group: "Rabbit MQ"
    - variable: rabbit-mq.rabbitMQ.replicaCount
      required: true
      default: 1
      type: int
      label: "Replicas"
      group: "Rabbit MQ"
    - variable: rabbit-mq.rabbitMQ.type
      default: "ClusterIP"
      type: enum
      label: "Service Type"
      group: "Rabbit MQ"
      options:
      - "ClusterIP"
      - "NodePort"
      - "LoadBalancer"

    ```
Questions are not connected to values, added questions just for demo purpose.
## Package Chart
Package the chart now using below command:
```
helm package ./demo
```

## Publish Chart
Chart can be published easily with command given below. There is a script at end to facilitate the operations in CICD pipelines.

```
curl --data-binary "@demo-0.1.0.tgz" http://chartmuseum-hostname:8080/api/charts
```

## Update Chart
In order to update the chart, perform following steps sequentially:
1. Make changes in codebase.
2. Change version of chart in `Chart.yaml` file.
3. [Package Chart](#cackage-chart) 
4. [Publish Chart](#publish-chart)
5. Upgrade the version of App in rancher UI.

## Install Chart
Following steps should help in in deployment of a chart on rancher:
1. Go to rancher UI
2. Open the required project where catalog is available.
3. Click on Apps from navigation bar.
4. Click on Launch if your are going to create new app.
5. Make changes in configuration
6. Click on Launch/Start button to deploy.

## Script
Please read the script and modify as per project requirements before using.

```
#!/bin/bash
# Controll the helm actions with this shell script.
# This script:
#   * is only for help in automation.
#   * does not replace developer efforts.
#   * does not include all dev helm commands. 
# This 
# =============================================
# COMMANDS ARE EXECUTED IN SEQUNCE OF ARGUMENTS
# =============================================

if [ "$1" = "help" ]; then
  echo -e "Valid commands: WIP \n\
  [build]:  Build a new chart. \n\
  [publish]: Publish the last built chart - sorted with time. \n\
  [template]: Template the chart for testing. \n\
  [validate]: Template and Validate with current attached kubernetes. \n\
  [clean]: Delete all built chart versions from local. \n\
  "
  exit;
fi

CHARTNAME="demo"
CHARTMUSEUM="http://chartmuseum-hostname:8080/api/charts"

while [ $# -ne 0 ]
do
    arg="$1"
    case "$arg" in
        build)
          echo "Building new chart..."
          helm package ./$CHARTNAME --destination ./charts
            ;;
        publish)
          echo "Publishing latest chart..."
          cd ./charts || { echo "Please build charts before publising!"; sleep 3; exit 1; }
          LAST_MODIFIED_FILE=$(ls -tr -- $CHARTNAME-*.tgz | tail -n 1)
          if [ $LAST_MODIFIED_FILE ]; then
            curl --data-binary "@$LAST_MODIFIED_FILE" $CHARTMUSEUM
          else
            echo "No charts found in ./charts directory!"
          fi
          cd ..
            ;;
        template)
          echo "Templating the helm chart..."
          helm template release-name ./$CHARTNAME --debug
            ;;
        validate)
          echo "Validating chart with configured kubernetes server..."
          helm template release-name ./$CHARTNAME --debug --validate
            ;;
        clean)
          echo "Removing all built chart tgzs from local folder..."
          rm ./builds/$CHARTNAME-*tgz
            ;;
        *)
            echo -e "Invalid command!\nTry one of [build, publish, template, validate, clean]"
            ;;
    esac
    shift
done

# Following are the commands that you can use outside the script for easy debugging:
# helm package ./$CHARTNAME --destination ./charts
# LAST_MODIFIED_FILE= ls -tr -- $CHARTNAME-*.tgz | tail -n 1
# echo "curl --data-binary @$LAST_MODIFIED_FILE "
# rm ./builds/$CHARTNAME-*tgz

# MORE COMMANDS FOR TROUBLESHOOTING/DEVELOPMENT/TESTING:
# =================================================

# To create templates locally:
# helm template release-name ./path [--debug] [--namespace=namespace]

# To create manifests and validate on server:
# helm template release-name ./path --validate [--debug] [--namespace=namespace]

# To install release from local codebase:
# helm install release-name ./path [--dry-run] [--namespace=namespace]

# To upgrade manually from local codebase:
# helm upgrade release-name ./path [--dry-run] [--namespace=namespace]

# To delete a release:
# helm delete release-name [--namespace=namespace]

# To package a new release:
# helm package release-name ./path [--namespace=namespace]

# To publish helm chart:
# curl --data-binary "@package-name-version-0.1.2" https://url.for.chartmuseum/api/charts [-u user:password]
```



## Alternative to ChartMuseum
There are some teams who are working with SaaS or have limitations that restrict chartmuseum part of infra. For such teams, there is an alternative to host the charts as a file system. I have taken an example of gitHub here as it is the most common public repo in use. You can go with a file hosting infra such as filezilla of nginx as well.

Helm hosting works on simple structure given below:
* index.yaml
* chartName
  * chartContent

Where index.yaml file has information about repository structure and descriptions of charts.

With given folder structure you can initialize a repository. Helm command line helps in many operations of charts, like pull, push, package and lint etc.. Additionally there are operations related to repo, like add, delete and indexing. Indexing is the operation that creates `index.yaml` as an output or merger its contents to an existing `index.yaml` file.

Next step is to host the repository on github pages for public access. Go to `Settings` of project and enbale `Pages` with default config. Go to root directory and run index command as follows:
```
cd chartName
helm package .
cd ..
helm repo index --url https://sainihimanshu983.github.io/charts/ .
git add .
git commit -m "Indexing for the first time"
git push
```

Going forward, you can append `--merge index.yaml` to helm repo index command. URL mentioned here is the gitHub pages URL for project.
