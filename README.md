### End-to-end Microservice Project from security scanning through CI/CD ( GitHub Action ) to monitoring on Grafana Dashboard.

[**Application Code Repository**](https://github.com/dev126712/microservices-app) contains all 5 microservice's CI Workflows. Each of them has SCA & SAST CI workflows. Every push commited by a developper is then scan and build.

All new images built is then, with the help of the sed command, change to the respective microservice [**helm chart deployment**](https://github.com/dev126712/microservice-charts-deployment) to include the new freshly and secure image.

The [**Infrastructure**](https://github.com/dev126712/micro-service-infra-management) is provisionned with terraform

  
