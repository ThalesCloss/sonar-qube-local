# Configuração do SonarQube para execução em ambiente local de desenvolvimento

Repositório conceitual para execução de análise de código com o SonarQube em ambiente local de desenvolvimento.



## Servidor SonarQube
Iniciar o container 

``` shell
docker-compsoe  up -d
```

Em caso de erro **max_map_count** no container do sonar executar:

``` shell
sudo sysctl -w vm.max_map_count=262144
sudo echo 'vm.max_map_count=262144' >> /etc/sysctl.conf
```

## Criar o projeto para análise local

Acesar http://localhost:9000  
Usuário: admin  
Senha: admin

Criar um projeto **Manually** preenchendo **Project display name** e **Project key**.  
Em seguida selecione a opção de análise **Locally** e gere um token para o projeto (Guarde o token, ele será utilizado posteriormente).  

## Configurando o projeto
Para projetos com cobertura de teste deve ser configurada a exclusão de arquivos de teste.  

Acesse *Project Settings -> Analysis Scope * e adicione a configuração no campo **Test File Exclusions**.  

Exemplo para projetos TypeScript/JavaScript
* **/*.spec.ts
* **/*.test.ts


## Executar análise do projeto

Na pasta de códigos do projeto desejado execute o sonar scanner.  
Substitua HOST_SONAR pelo endereço do servidor sonar (http://172.17.0.1:9000).  
Substitua TOKEN_PROJETO pelo token criado anteriormente. 
Substitua PROJECT_KEY pelo **Project key** definido anteriormente.  
Se o projeto possuir cobertura de código deve ser informado o caminho do arquivo *lcov.info* no lugar de COVERAGE, caso contrário a última linha deve ser suprimida.  
e.g COVERAGE=coverage/lcov.info

```shell
docker run \
    --rm \
    -e SONAR_HOST_URL="HOST_SONAR" \
    -e SONAR_LOGIN="TOKEN_PROJETO" \
    -v "${PWD}:/usr/src" \
    sonarsource/sonar-scanner-cli  \
    -Dsonar.projectKey=PROJECT_KEY \
    -Dsonar.javascript.lcov.reportPaths=/usr/src/COVERAGE
```

A análise do projeto deve ser executada rotineiramente (e.g pre-commit, pre-push, etc...).  

## Configuração da extensão SonarLint para VsCode

Crie um token para conexão do VsCode ao servidor sonar no endereço http://localhost:9000/account/security/  

Instale a extensão SonarLint.  
https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarlint-vscode

Adicione as linhas ao arquivo de configuração do usuário, substituindo o caminho javaHome e o token criado no passo anterior.    

```
"sonarlint.ls.javaHome": "/usr/lib/jvm/java-11-openjdk-amd64",
"sonarlint.connectedMode.connections.sonarqube": [
        {
            "serverUrl": "http://localhost:9000",
            "token": "SEU_TOKEN_VS_CODE",
        }
],
```
Na configuração workspace do projeto adicione, substituindo o PROJECT_KEY pelo projeto escolhido.  
```
"sonarlint.connectedMode.project": {
    "projectKey": "PROJECT_KEY"
},
```



### Referências
* https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/
* https://github.com/SonarSource/sonarlint-vscode
* https://chiamakaikeanyi.dev/how-to-setup-sonarqube-in-vscode/