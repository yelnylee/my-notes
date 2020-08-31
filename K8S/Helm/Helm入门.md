# Helm 入门

* Helm的用途
  * Helm用于管理Kubernetes 应用，Helm charts 用于定义，安装，升级复杂的Kubernetes 应用。
  * Charts是易于创建，版本管理，共享，发布的
* Helm架构
  * Helm的目的
    * Helm是一个工具，用于管理Kubernetes包，这些Kubernetes包被称为charts。
    * 创建一个全新的charts
    * 打包charts
    * 与charts仓库交互
    * 安装和卸载charts在Kubernetes集群中
    * 管理charts的发布周期

* helm repo add

  * First add proxy

    * ```
      export proxy="http://web-proxy.us.softwaregrp.net:8080"
      export http_proxy=$proxy
      export https_proxy=$proxy
      export no_proxy="localhost, 127.0.0.1, ::1"
      
      helm repo add stable https://svsartifactory.swinfra.net/artifactory/itom-helm-charts-stable-local
      ```

* Chart Templates

  ```yaml
  define
  {{define "mychart.labels"}}
  # body of  template here
  {{}}
  
  {{- define "mychart.labels"}}
   labels:
     generator: helm
     date: {{ now | htmlDate }}
  {{- end}}
  
  template
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: {{ .Release.Name }}-configmap
    {{- template "mychart.labels" }}
  data:
    myvalue: "Hello World"
    
  block
  include
  ```

  