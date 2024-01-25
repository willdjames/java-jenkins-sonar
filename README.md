# CI e estratégias de Quality Gates bloqueantes e não-bloqueantes 


Uma POC para apresentar a possibilidade de construir uma pipeline de CI com estrategia de interromper
ou prosseguir, mesmo quando o retorno de falha do quality gate do Sonar.  

No docker-compose.yaml tem configurado ambos os serviços da instancia do Sonar e Jenkins.


## Sobre o Sonar

Sem nenhuma configuração extra, apenas utlizando o quality gate _Sonar Way_.


## Sobre o Jenkins

Apenas instalado o executavel do _sonar-scanner_ no servidor do Jenkins.  

### Dos estagios da pipeline da POC

Estagios do [Jenkinsfile](./Jenkinsfile):

1. Clone deste repositorio para componente java. 
2. Compila o codigo fonte.
3. Execução do _sonar scanner_ com [sonar-project](./sonar-project.properties) na raiz deste repositorio.
4. Sonar Gate, um estagio onde aguardamos o retorno do Quality Gate.
5. Aconstrução do Arquivo _.jar_.

### Como conseguimos aplicar nossa estrategia

#### Para estrategia de abortar o CI caso há falha no Quality Gate
Nesta estrategia, não queremos que a pipeline continua caso o projeto não passe no Quality Gate.
Lembrando que este tambem pode ser o comporrtamento padrão da pipeline que espera o retorno do Quality Gate.

Para isso devemos aplicar as seguintes configurações na pipeline:
Invocar o _sonar-scanner_ dentro da função _withSonarQubeEnv_. isto nos habilita no proximo stagio a tomar uma
ação após o retorno do Quality Gate.

```
withSonarQubeEnv('SOnarqube') {
    sh "/var/jenkins_home/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner -Dsonar.host.url=http://sonar:9000 \ 
    -Dsonar.token=${SONAR_TOKEN}"
}
```

E no proximo estagio, configurar a propriedade _abortPipeline_ para true da variavel _waitForQualityGate_
entregue pela função _withSonarQubeEnv_ do estagio anterior.

```
steps {
    timeout(time: 3, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true
    }
}
```


#### Para estrategia de continuar o CI mesmo caso há falha no Quality Gate
Nesta estrategia, queremos que a pipeline continua caso o projeto não passe no Quality Gate.

Neste estagio, configurar a propriedade _abortPipeline_ para false da variavel _waitForQualityGate_
entregue pela função _withSonarQubeEnv_ do estagio anterior. E com o bloco _catchError_,
podemos configurar o _stageResult_ para 'UNSTABLE', indicando que o estagio não teve sucesso,
mas tambem não apresentou erro critico que fosse necessario interromper a pipeline. 

```
steps {
    timeout(time: 3, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: false
    }
    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
        error 'STAGE UNSTABLE'
    }
}
```


## Conclusão

Notamos que a configuração responsavel, que nos habilita a adotar a estrategia de alterar o comportamento
padrão de abortar a pipeline após o retorno de falha do Quality Gate, é 
``` waitForQualityGate abortPipeline: false|true ```, onde _true_ é o valor padrão. Mas para isso 
ser possivel, devemos involcar o _sonar-scanner_ dentro do bloco ``` withSonarQubeEnv() ```.