# How to customized Priority Class name

* **If suite need use PriorityClass in CDF ,suite need config parameters as bellow.**

  * **The following table lists the configurable PriorityClass parameters of the demo chart and their default values.**

  * **Global PriorityClass Parameter**

    | ** Parameter**                                            | **Description**                                              | **Default**              | Required |
    | --------------------------------------------------------- | ------------------------------------------------------------ | ------------------------ | -------- |
    | global.priorityclass.priorityClassMaping.node-critical    | The PriorityClass provider by system,Priority class value is 2000001000, Priority class preemptionPolicy is PreemptLowerPriority | system-node-critical     | false    |
    | global.priorityclass.priorityClassMaping.cluster-critical | The PriorityClass provider by system,Priority class value is 2000000000, Priority class preemptionPolicy is PreemptLowerPriority | system-cluster-critical  | false    |
    | global.priorityclass.priorityClassMaping.highest          | The PriorityClass provider by cdf or other k8s manage,Priority class value is -1000000, Priority class preemptionPolicy is PreemptLowerPriority | cdf-highest-priority     | false    |
    | global.priorityclass.priorityClassMaping.high             | The PriorityClass provider by cdf or other k8s manage,Priority class value is -1001000, Priority class preemptionPolicy is PreemptLowerPriority | cdf-high-priority        | false    |
    | global.priorityclass.priorityClassMaping.medium-high      | The PriorityClass provider by cdf or other k8s manage,Priority class value is -1002000, Priority class preemptionPolicy is PreemptLowerPriority | cdf-medium-high-priority | false    |
    | global.priorityclass.priorityClassMaping.medium           | The PriorityClass provider by cdf or other k8s manage,Priority class value is -1003000, Priority class preemptionPolicy is PreemptLowerPriority | cdf-medium-priority      | false    |
    | global.priorityclass.priorityClassMaping.medium-low       | The PriorityClass provider by cdf or other k8s manage,Priority class value is -1004000, Priority class preemptionPolicy is PreemptLowerPriority | cdf-medium-low-priority  | false    |
    | global.priorityclass.priorityClassMaping.low              | The PriorityClass provider by cdf or other k8s manage,Priority class value is -1005000, Priority class preemptionPolicy is PreemptLowerPriority | cdf-low-priority         | false    |
    | global.priorityclass.priorityClassMaping.lowest           | The PriorityClass provider by cdf or other k8s manage,Priority class value is -1006000, Priority class preemptionPolicy is PreemptLowerPriority | cdf-lowest-priority      | false    |

* **Example**

  * Config default values.yaml

    ```yaml
    .................
    .................
    .................
    global:
      # priorityclass values
      priorityclass:
        priorityClassMaping:
          highest: cdf-highest-priority
          high: cdf-high-priority
    .................
    .................
    .................
    ```

  * Config spec-priorityclass-values.yaml

    ```yaml
    .................
    .................
    .................
    global:
      # priorityclass values
      priorityclass:
        priorityClassMaping:
          highest: suite-highest-priority
          high: suite-high-priority
    .................
    .................
    .................
    ```

    

  * Use priortyclass in Config template

    ```yaml
    .................
    .................
    .................
     spec:
          initContainers:
          {{- include "helm-lib.waitForVault" . | nindent 6 }}
          {{- include "helm-lib.waitFor" (dict "service" (include "helm-lib.dbHost" .) "port" (include "helm-lib.dbPort" .) "Values" .Values) | nindent 6 }}
          - name: install
            image: {{ .Values.global.docker.registry }}/{{ .Values.global.docker.orgName }}/{{ .Values.global.vaultInit.image }}:{{ .Values.global.vaultInit.imageTag }}
            env:
            {{- if .Values.global.vaultAppRole }}
            - name: VAULT_ROLE_ID
              value: {{ .Values.global.vaultRoleId }} 
            {{- end }}
            volumeMounts:
            - name: vault-token
              mountPath: /var/run/secrets/boostport.com
          nodeSelector:
            Worker: label
          {{- if .Values.global.priorityclass.priorityClassMaping.high }}
          priorityClassName: {{ .Values.global.priorityclass.priorityClassMaping.high | quote }}
          {{- end }}
    .................
    .................
    .................
    ```

  * install chart

    ```shell
    # Create priorityclass  by cdf
    helm install --generate-name itom-demo-chart
    # Create priorityclass by others
    helm install --generate-name itom-demo-chart -f spec-priorityclass-values.yaml
    ```

    

* 

  | Name                     | Priority   | Global Default | Remarks                    | Global Pro in chart                                          |
  | ------------------------ | ---------- | -------------- | -------------------------- | ------------------------------------------------------------ |
  | system-node-critical     | 2000001000 | false          | 系统默认创建，提供最高权限 | priorityclass.priorityClassMaping.node-critical              |
  | system-cluster-critical  | 2000000000 | ~              | 系统默认创建，提供次高权限 | priorityclass.priorityClassMaping.cluster-critical(无需定义) |
  | cdf-highest-priority     | -1000000   | ~              |                            | priorityclass.priorityClassMaping.highest                    |
  | cdf-high-priority        | -1001000   | ~              |                            | priorityclass.priorityClassMaping.high                       |
  | cdf-medium-high-priority | -1002000   | ~              |                            | priorityclass.priorityClassMaping.medium-high                |
  | cdf-medium-priority      | -1003000   | ~              |                            | priorityclass.priorityClassMaping.medium                     |
  | cdf-medium-low-priority  | -1004000   | ~              |                            | priorityclass.priorityClassMaping.medium-low                 |
  | cdf-low-priority         | -1005000   | ~              |                            | priorityclass.priorityClassMaping.low                        |
  | cdf-lowest-priority      | -1006000   | ~              |                            | priorityclass.priorityClassMaping.lowest                     |

  * cdf 提供除系统默认Priority Class以外的7个层次的优先级, 分别是cdf-highest-priority>cdf-high-priority,

    cdf-medium-high-priority>cdf-medium-priority>cdf-medium-low-priority>cdf-low-priority>cdf-lowest-priority

  * 此7个Priority Class的优先级低于系统默认Priority Class。

  * cdf-highest-priority适用范围

  * cdf-high-priority适用范围

  * cdf-medium-high-priority 适用范围

  *  cdf-medium-priority适用范围

  * cdf-medium-low-priority适用范围

  * cdf-low-priority适用范围

  * cdf-lowest-priority适用范围

  * Other 优先等级，如何处理？

  * suite 与 cdf 如何对应？

  * suite 与 其他组件如何对应？

* Values.yaml in app chart demo

  ```yaml
  global:
    # priorityclass values
    priorityclass:
      priorityClassMaping:
        highest: cdf-highest-priority
        high: cdf-high-priority
        low: cdf-low-priority
        lowest: cdf-lowest-priority
        medium-high: cdf-medium-high-priority
        medium-low: cdf-medium-low-priority
        medium: cdf-medium-priority
        cluster-critical: system-cluster-critical
        node-critical: system-node-critical
  ```

  

* deployment yaml in sub-chart of demo

  ```yaml
   ...............
   ...............
   spec:
        initContainers:
        {{- include "helm-lib.waitForVault" . | nindent 6 }}
        {{- include "helm-lib.waitFor" (dict "service" (include "helm-lib.dbHost" .) "port" (include "helm-lib.dbPort" .) "Values" .Values) | nindent 6 }}
        - name: install
          image: {{ .Values.global.docker.registry }}/{{ .Values.global.docker.orgName }}/{{ .Values.global.vaultInit.image }}:{{ .Values.global.vaultInit.imageTag }}
          env:
          {{- if .Values.global.vaultAppRole }}
          - name: VAULT_ROLE_ID
            value: {{ .Values.global.vaultRoleId }} 
          {{- end }}
          volumeMounts:
          - name: vault-token
            mountPath: /var/run/secrets/boostport.com
        nodeSelector:
          Worker: label
        {{- if .Values.global.priorityclass.priorityClassMaping.high }}
        priorityClassName: {{ .Values.global.priorityclass.priorityClassMaping.high | quote }}
        {{- end }}
        ...............
  ```

  

* 定义 .tpl

  ```yaml
  {{- difine .Release.Name-highest-priorty }}
   {{- if .Values.global.priorityclass.priorityClassMaping.highest }}
        priorityClassName: {{ .Values.global.priorityclass.priorityClassMaping.high | quote }}
        {{- end }}
  {{ end }}
  
  {{- difine .Release.Name-high-priorty }}
  end
  
  {{- difine .Release.Name-medium-high-priorty }}
  end
  
  {{- difine .Release.Name-medium-priorty }}
  end
  
  {{- difine .Release.Name-medium-low-priorty }}
  end
  
  {{- difine .Release.Name-low-priorty }}
  end
  
  {{- difine .Release.Name-lowest-priorty }}
  end
  ```

  

Parameters

 The following table lists the configurable parameters of the itom-chart-demo chart and sub-chart and their default values per selection/component:

Global parameters

```shell
#Required: the 'externalAccessHost' must be set to the cluster external domain name
global.externalAccessHost
#The externalAccessPort is the port on which the service is accessed from outside your cluster, default 443
global.externalAccessPort
global.priorityclass.priorityClassMaping.node-critical ,global.priorityclass.priorityClassMaping.cluster-critical,global.priorityclass.priorityClassMaping.highest,global.priorityclass.priorityClassMaping.high
,global.priorityclass.priorityClassMaping.medium-high ,global.priorityclass.priorityClassMaping.medium ,global.priorityclass.priorityClassMaping.medium-low,global.priorityclass.priorityClassMaping.low,global.priorityclass.priorityClassMaping.lowest

global.initSecrets


      highest: cdf-highest-priority
      high: cdf-high-priority
  #      low: cdf-low-priority
  #      lowest: cdf-lowest-priority
  #      medium-high: cdf-medium-high-priority
  #      medium-low: cdf-medium-low-priority
  #      medium: cdf-medium-priority
  #      cluster-critical: system-cluster-critical
  #      node-critical: system-node-critical

cdf provides 7 levels of priority in addition to the system default Priority Class,
which are cdf-highest-priority>cdf-high-priority>cdf-medium-high-priority>
cdf-medium-priority>cdf-medium-low-priority>cdf-low-priority>cdf-lowest-priority.


Sort by priority from high to low,



The chart supports 9 levels of priority class customization,
1) Priority is sorted from high to low
(global.priorityclass.priorityClassMaping.node-critical>global.priorityclass.priorityClassMaping.cluster-critical>
global.priorityclass.priorityClassMaping.highest>global.priorityclass.priorityClassMaping.high>
global.priorityclass.priorityClassMaping.medium-high>global.priorityclass.priorityClassMaping.medium>
global.priorityclass.priorityClassMaping.medium-low>global.priorityclass.priorityClassMaping.low>
global.priorityclass.priorityClassMaping.lowest>empty).
2) Among them, global.priorityclass.priorityClassMaping.node-critical,
global.priorityclass.priorityClassMaping.cluster-critical corresponds to
System-node-critical, system-cluster-critical in K8S system
3) Suite can divide K8S resource priority class according to needs, less than 7 levels
4) Suite can customize the priority class name as needed, and pass in the chart through global variables
```

Common parameters

itom-chart-demo parameters

Configuration Detail

How to customized Priority Class name