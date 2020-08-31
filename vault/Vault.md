# Vault

* Vault 命令

  *  命令行客户端访问服务器，需要首先设置环境变量（VAULT_ADDR） 

  ```shell
  * get vault ip from svc/pod:
  	VAULT_HOST=$(kubectl get svc -n core itom-vault -o json|jq -r .spec.clusterIP)
  * export var
  	export VAULT_ADDR="https://$VAULT_HOST:8200"
  * export var
  	cdf_apiserver=$(kubectl get pods -n core |grep cdf-apiserver|awk  '{ print $1 }')
  	#Depercated command
  	#vault_root_token=$(kubectl exec -it $cdf_apiserver -n core -c cdf-apiserver cat /var/run/secrets/boostport.com/vault-root-token)
  	vault_root_token=$(kubectl exec -it $cdf_apiserver -n core -c cdf-apiserver -- cat /var/run/secrets/boostport.com/vault-root-token)
  	export VAULT_TOKEN=$vault_root_token
  * skip cert validate
  	vault status -tls-skip-verify
  	vault list -tls-skip-verify auth/approle/role/
  	vault list -tls-skip-verify auth/kubernetes/role
  	vault read -tls-skip-verify auth/approle/role/core-baseinfra
  	vault read -tls-skip-verify -format=json auth/approle/role/core-baseinfra/role-id
  	vault path-help -tls-skip-verify auth/approle
  	vault path-help -tls-skip-verify sys
  	
  
  ```

* Root Token

  *  Token 是Vault默认的验证手段 

    ```
    无论开发还是生产环境，服务器启动时都会生成一个 Root Token，该 Token 相当于 Linux 系统中的 root user，具有最高的访问权限。使用 Root Token 登录系统的用户可以生成另外的 Token，并且为这些 Token 设置特定的访问权限。这就是 Vault 的认证/授权机制。在开发模式下，用户默认是使用 Root Token 登录的，当然也可以在分配新的 Token 之后，使用新 Token 重新登录。后面我们还会具体介绍 Auth Token 的管理过程。
    ```

    

* Vault Read/Write

  ```
  读写数据使用 read/write 命令。参数 secret/hello这是一个路径（Path）。Vault 中用 Path 区分不同数据的存放位置。一个 Path 下可以包含多个键值对，但需要注意的是 write 命令是覆盖式的而不是追加式的，所以第一条命令写入的值会被后来的所覆盖，因此你必须确保在同一个命令中一次性写入所有的内容。
  
  在上述例子中，我们使用了 secret/hello 路径，这个路径并不是随意指定的。默认情况下，你只能把私密数据保存在以 secret/ 为前缀的路径下，否则会出现下面的错误信息：
  
  $ vault write -tls-skip-verify /secret/hello value=world
  Error writing data to secret/hello: Error making API request.
  
  URL: PUT https://172.17.17.200:8200/v1/secret/hello
  Code: 404. Errors:
  
  * no handler for route 'secret/hello'
  
  从出错信息可以发现，Vault 服务器/客户端之间实际上是 RESTful 形式的 HTTP 通信。但是，Vault 为什么要强加这样的限制呢？那就要从 Secret Engine 说起了
  
  
  vault write -tls-skip-verify itom/suite/0e3924aa-293d-c087-6f6f-1dd896aac1ef/hello value=world
  vault read -tls-skip-verify itom/suite/0e3924aa-293d-c087-6f6f-1dd896aac1ef/hello
  ```

*  Secret Engine 

```
vault secrets list -tls-skip-verify 列出Secret Engine 清单
vault list -tls-skip-verify auth/approle/role/ 列出role
vault read -tls-skip-verify /auth/approle/role/demo-gxxdr-dbadmin/role-id
vault write -tls-skip-verify itom/suite/0e3924aa-293d-c087-6f6f-1dd896aac1ef/hello value=world
vault read -tls-skip-verify itom/suite/0e3924aa-293d-c087-6f6f-1dd896aac1ef/hello

vault secrets enable -tls-skip-verify  -path=kv kv
vault secrets enable -tls-skip-verify  -path=di-integration pki

vault path-help pki
vault path-help kv
```

* auth method

* Policy

  ```
  vault policy list -tls-skip-verify
  vault policy write -tls-skip-verify my-policy my-policy.hcl
  vault policy read -tls-skip-verify my-policy
  ---------------------------------------my-policy.hcl----------------
  path "secret/*" {
    capabilities = ["create"]
  }
  
  path "secret/foo" {
    capabilities = ["read"]
  }
  ```

  





1.

使用kubectl apply  -f public-ca-certificates

```
RIC、RE、CUS、RID get PKI cert
/v1/{realm}/cert/ca
Custom role
```

```
_ca.crt
CA_CUSTOM_REALMS
```



* CDF 平台中自定义Realms的使用方法：

  * 当获取当前suite Version 后，从feature json中获取Realms

  * 根据当前获取的realms，检查是否已经存在(di-integration/)PKI Secret Engine

  * 如果不存在则创建

    * 创建成功，继续生成CA PKI 
    * 生成PKI CA成功，继续生成 URL configuration
    * 生成成功后，创建写死的role coretech

  * releam已经创建成功，此时需要更新realm相关的证书到public-ca-certificates

    * 目前逻辑直接生成<realms>_ca.crt的key
    * 将相应key patch 到public-ca-certificates

  * role的使用方式，

    * 创建role时根据，是否在指定的realms中创建role

    * 假设db_admin 在自定义realms（di-integration）中

    * 则在指定realm-policy中增加权限

      vault policy read -tls-skip-verify demo-gxxdr-dbadmin-realm-policy
      //RID realm policy
      path "RID/issue/*" {
        capabilities = ["update"]
      }

      //RE realm policy
      path "RE/issue/*" {
        capabilities = ["update"]
      }

      //di-integration realm policy
      path "di-integration/issue/*" {
        capabilities = ["update"]
      }

  ```
  1. generateCustomRealms
  2.updateRootCAConfigMap、createRootCAConfigMap
  3. public AppRoleAndIdListDTO createVaultApprole(String suiteNamespace, String deploymentUuid) throws RequestApiServerException {
      AppRoleAndIdListDTO returnValue = null;
      AppRolesListDTO appRolesListDTO = new AppRolesListDTO();
      appRolesListDTO.setVaultNamespace(suiteNamespace);
      appRolesListDTO.setAppRoleList(Arrays.asList(new AppRole(SS_CONSTANT.DB_ADMIN)));
      returnValue = certificateService.createVaultApprole(deploymentUuid,appRolesListDTO).get();
      return returnValue;
    }
  4.    
  "realms" : [
          {
              "name": "data ingestion realm",
              "short_name": "di-integration",
              "roles_can_generate" : [
              	“demo”,
                  "omi",
                  "di",
                  "db-admin"
              ]
          },
           {
              "name": "data ingestion realm",
              "short_name": "BI-integration",
              "roles_can_generate" : [
              	“demo”,
                  "omi",
                  "di",
                  "db-admin"
              ]
          }
      ]
  
  
  ```



DEPLOYMENT-ADMIN-USERNAME

DEPLOYMENT-ADMIN-PASSWORD



idm_integration_admin_password



* workround

  ```
  * <NFS_CORE>/yamlContent/transforParamsDTOStr.json is exist.
  * APISERVER=$(kubectl get pods -n core|grep cdf-apiserver|awk '{ print $1 }')
  * VAULT_INFO=$(kubectl exec -it $APISERVER -n core -c cdf-apiserver cat /var/run/secrets/boostport.com/vault-token)
  * export VAULT_TOKEN=$(echo $VAULT_INFO|jq -r .clientToken)
  * export VAULT_ADDR=$(echo $VAULT_INFO|jq -r .vaultAddr)
  * SECRET_PATH=$(echo $VAULT_INFO|jq -r .secretPath)
  * vault list -tls-skip-verify $SECRET_PATH|grep vault-params-key
  * if did not exist vault-params-key-<deploymentUuid>,
    *  vault write -tls-skip-verify  $SECRET_PATH/vault-params-key-<deploymentUuid> value="$(cat <NFS_CORE>/yamlContent/transforParamsDTOStr.json)"
  * if did exist vault-params-key-<deploymentUuid>
    * vault read -tls-skip-verify -format=json $SECRET_PATH/vault-params-key-<deploymentUuid>|jq -r .data.value>param.json.bak
    * vault write -tls-skip-verify  $SECRET_PATH/vault-params-key-<deploymentUuid> value="$(cat <NFS_CORE>/yamlContent/transforParamsDTOStr.json)"
  ```

* workaround

  ```shell
  
  1. get all vault-params-key
  ITOM_VAULT_IP=$(kubectl get svc -n core itom-vault -o json|jq -r .spec.clusterIP)
  export VAULT_ADDR="https://${ITOM_VAULT_IP}:8200"
  CDF_APISERVER=$(kubectl get pods -ncore|grep cdf-apiserver|awk '{print $1}')
  VAULT_ROOT_TOKEN=$(kubectl exec -it $CDF_APISERVER -n core cat /run/secrets/boostport.com/vault-root-token)
  export VAULT_TOKEN="${VAULT_ROOT_TOKEN}"
  ROLE_ID=$(vault read -tls-skip-verify -format=json auth/approle/role/core-baseinfra/role-id|jq -r  .data.role_id)
  vault list -tls-skip-verify -format=json itom/suite/${ROLE_ID} |grep vault-params-key
  3.loop each value output by step 1 last command, backup each vault-params-key
  vault read -tls-skip-verify -format=json itom/suite/${ROLE_ID}/<vault-params-key>|jq -r .data.value><vault-params-key>.json
  
  VALUE=$(<vault-params-key>.json)
   vault write -tls-skip-verify itom/suite/${ROLE_ID} "vault-params-key-724305bf-a604-416f-9813-cedb2f5cec98=${VALUE}"
  
  ```

  ```shell
  
  ```
1.	<deploymentUuid>的值8d9db0d4-f7ea-4666-a587-e86c0fade564
  2.	将正确的大参数值保存到文件<vault-params-key.json>中
  3.	在master机器上，运行以下步骤：

    ITOM_VAULT_IP=$(kubectl get svc -n core itom-vault -o json|jq -r .spec.clusterIP)

  export VAULT_ADDR=https://${ITOM_VAULT_IP}:8200

  CDF_APISERVER=$(kubectl get pods -ncore|grep cdf-apiserver|awk '{print $1}')

  VAULT_ROOT_TOKEN=$(kubectl exec -it $CDF_APISERVER -n core cat /run/secrets/boostport.com/vault-root-token)

  export VAULT_TOKEN="${VAULT_ROOT_TOKEN}"

  ROLE_ID=$(vault read -tls-skip-verify -format=json auth/approle/role/core-baseinfra/role-id|jq -r  .data.role_id)

  VAULT_PARAMS_KEY=$(cat <vault-params-key.json>)

  vault write -tls-skip-verify itom/suite/${ROLE_ID}/vault-params-key-<deploymentUuid> "value=${VAULT_PARAMS_KEY}"

  ```
  
  
  
  
  ```

* WorkRound For Create db-admin role

  * Get SUITE INSTALLER IP

    ```shell
    suite_installer_ip=$(kubectl get svc -n core suite-installer-svc -o json|jq -r .spec.clusterIP)
    ```

  * Set cdfApiBaseUrl

    ```shell
    cdfApiBaseUrl=http://$suite_installer_ip:8080
    ```

  * Set 5443 UI user and password, User need input you user/password

    ```
    username=admin
    password=Admin@111
    ```

  * get X_AUTH_TOKEN

    ```shell
    api_url="/urest/v1.1/cdf/token"
    apiResponse=$(curl -s -k -X POST \
                        -w '%{http_code}' \
                        --header 'Content-Type: application/json' \
                        --header 'Accept: application/json' \
                        -d "{\"password\": \"${password}\", \"username\": \"${username}\" }" \
                        ${cdfApiBaseUrl}${api_url})
                        
    x_auth_token=$(echo ${apiResponse:0:-3} | jq -r '.token')
    X_AUTH_TOKEN="X-Auth-Token: ${x_auth_token}"    
    ```

    

  *  getCsrfTokenSessionID 

    ```shell
    api_url="/urest/v1.1/csrf-token"
    apiResponse=$(curl -s -k -X GET \
                                     -w '%{http_code}' \
                                     --header 'Accept: application/json' \
                                     --header "${X_AUTH_TOKEN}" \
                                     ${cdfApiBaseUrl}${api_url})
    x_csrf_token=$(echo ${apiResponse:0:-3} |jq -r '.csrfToken')
    session_id=$(echo ${apiResponse:0:-3} |jq -r '.sessionId' )
    session_name=$(echo ${apiResponse:0:-3} |jq -r '.sessionName')
    X_CSRF_TOKEN="X-CSRF-TOKEN: ${x_csrf_token}"
    JSESSION_ID="${session_name}=${session_id}"
    ```

  * ssd



* Delete kubernates role

  ```shell
  vault delete -tls-skip-verify auth/kubernetes/role/demo-acajy-dbadmin
  ```

  