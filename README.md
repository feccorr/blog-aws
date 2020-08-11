# Blog Aws

> O projeto é a criação de um simples blog fazendo uso das tecnologias : 
> Spring boot, Thymeleaf, Bootstrap, PostgreSQL e Spring Security para realizar o deploy
> no AWS Elastic Beanstalk.


### Login Springsecurity

Para acessar os endpoints restritos (criar post e editar post), segue o login a ser usado na autenticação em memoria do spring security :

| Usuario | Senha |
| ------ | ------ |
| fernando | elasticbeanstalk |



### Deploy no AWS Elastic Beanstalk
Antes de iniciar o processo de deploy, devemos antes criar uma conta gratuita na AWS, e seguir os passos :

1) Criar um conta gratuita na AWS: https://aws.amazon.com/pt/free/

2) Criar chave de acesso: https://docs.aws.amazon.com/pt_br/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys

3) Instalar AWS CLI de acordo com o sistema operacional: https://docs.aws.amazon.com/pt_br/cli/latest/userguide/cli-chap-welcome.html

Depois de preparar o ambiente devemos seguir as seguintes etapas : 

Iniciar um ambiente na aws:
```sh
$ eb init
```

Criar um RDS Postgres:
```sh
$ eb create --scale 1 -db -db.engine postgres -db.i db.t2.micro
```
Criar um arquivo application-beanstalk.properties dentro da pasta resources (As informações de rds, serão reconhecidas pelo beanstalk no deploy):

>spring.datasource.url=jdbc:postgresql://${rds.hostname}:${rds.port}/${rds.db.name}
>spring.datasource.username=${rds.username}
>spring.datasource.password=${rds.password}

Criar um Profile no arquivo pom.xml:

><profiles>
		<profile>
			<id>beanstalk</id>
			<build>
				<finalName>${project.name}-eb</finalName>
				<plugins>
					<plugin>
						<groupId>org.springframework.boot</groupId>
						<artifactId>spring-boot-maven-plugin</artifactId>
					</plugin>
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-compiler-plugin</artifactId>
						<configuration>
							<excludes>
								<exclude>**/cloud/config/*.java</exclude>
							</excludes>
						</configuration>
					</plugin>
				</plugins>
			</build>
		</profile>
	</profiles>
	
	
Gerar o arquivo .jar com o seguinte comando:
```sh
$ mvn clean package spring-boot:repackage
```
Adicionar o nome do arquivo gerado acima .jar no arquivo de configuração elasticbeanstalk/config.yml:
> deploy: 
>            artifact: target/nome-arquivo-gerado.jar

Fazer o deploy:
```sh
$ eb deploy
```

Criar duas variáveis de ambiente:
```sh
$ eb setenv SPRING_PROFILES_ACTIVE=beanstalk,mysql
$ eb setenv SERVER_PORT=5000
```

Após o termino verificar o status do deploy:
```sh
$ eb status
```

![ss code blog](https://user-images.githubusercontent.com/45246027/89851323-99183580-db62-11ea-9e14-7779fdbc4219.jpg)
