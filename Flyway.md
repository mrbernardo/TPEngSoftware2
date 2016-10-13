Flyway
===
# Flyway by boxfuse

![Logo](https://i.imgur.com/q0xCcAI.png)

## 1. Introdução
Sistemas de banco de dados desempenham papeis fundamentais no funcionamento das organizações. Sua utilização ao abrigar todos os dados históricos e operacionais possibilitam e auxiliam nas tomadas de decisões e no alcance dos objetivos do empreendimento. Um banco de dados tem que fornecer informações confiáveis e atualizadas aos "_steakholders_" e, para tanto, é necessário que haja apenas uma versão atualizada em todas as instâncias nas quais esteja em funcionamento.

Na construção de sistemas é observado que as equipes de desenvolvimento e os engenheiros de software têm maior cuidado de manter as versões dos projetos atualizadas e sincronizadas entre si, normalmente eles contam com excelentes ferramentas de controle de versão, integração contínua entre as equipes e processos de implantação e lançamentos bem definidos. 

Contudo, o que é possível dizer sobre a estrutura de um banco de dados no desenvolvimento do sistema? É visto que nem sempre há a preocupação e o cuidado em verificar, ajustar  a versão do sistema de banco de dados que as equipes estão utilizando. A ausência de ferramentas que possibilitam verificar: o estado de um banco de dados, a aplicação de determinado script no banco, uma correção de erro a ser submetida em testes posteriores ou configuração de nova instância de um banco de dados,tudo isso, dificulta e consome muito tempo para o controle adequado e atualização do versionamento dos bancos, por exemplo no desenvolvimento de sistemas complexos. Disso, decorre a necessidade de utilizar-se a **migração do banco de dados** de uma determinada versão para outra, mais nova, a fim de _reaver o controle_ e a _organização dos dados_.

## 2. O Sistema Flyway - Visão Geral
O _Flyway_ é uma ferramenta de código aberto multiplataforma e colaborativa utilizada para a migração de banco de dados relacionais de forma simplificada, criando-se convenções com base em versionamento. Escrita em _**Java**_, funciona através de instruções em linha de comando contendo seis operações básicas para migração, limpeza, obtenção de informações, validar, _"baseline"_ e reparo do banco de dados.


As migrações podem ser codificadas utilizando-se _sintaxes de SQL_ específicas, tais como (PL/SQL, T-SQL, etc...), ou _linguagem Java_ (para transformações avançadas de dados ou lidar com LOBs[^1]).

Recomenda-se utilizar a máquina virtual Java _(JVM)_ para a execução do processo de migração da base de dados no início da aplicação. Contudo, o sistema funciona também através dos plugins _Maven, Gradle e SBT_, além de tarefas _Ant_. Há diversos outros plugins disponíveis na [página do desenvolvedor](https://flywaydb.org/documentation/plugins).

[^1]:[Introduction to LOBs](https://docs.oracle.com/cd/B10501_01/appdev.920/a96591/adl01int.htm)

## 3. Como Funciona
O Flyway tem como objetivo a criação e gereciamento de tabelas de metadados de um banco de dados relacional. O exemplo mais simples é quando se tem um banco de dados vazio: a ferramenta tenta localizar a tabela de metadados do banco de dados; não sendo possível encontrá-la, o Flyway criará essa tabela de metadados.

![Exemplo de criação de um banco de dados vazio no Flyway](https://i.imgur.com/6GyCTYe.png)
<sub><sup>_**Exemplo de criação de um banco de dados vazio no Flyway**_</sup></sub>


Por padrão, será criado um banco de dados com uma tabela vazia nomeada como _SCHEMA_VERSION_, que servirá para controlar o estado do banco de dados.

![Banco de Dados Vazio](https://i.imgur.com/gqZf7Sq.png)
<sub><sup>_**Banco de dados vazio nomeado por padrão como SCHEMA_VERSION**_</sup></sub>


Após a criação da tabela, o Flyway efetua um _**scan**_ do sistema de arquivos ou do _classpath_ da aplicação para executar as _**migrações**_. Essas rotinas podem ser escritas em SQL ou Java. As migrações são **classificadas** com base em seu **número de versão** e aplicadas de forma ordenada, gerando uma atualização na tabela de metadados ao passo da aplicaçãod e cada migração, conforme exemplo na tabela a seguir:

| installed_rank | version | description   | type | script                | checksum   | installed_by | installed_on          | execution_time | success |
|----------------|---------|---------------|------|-----------------------|------------|--------------|-----------------------|----------------|---------|
| 1              | 1       | Initial Setup | SQL  | V1__Initial_Setup.sql | 1996767037 | axel         | 2016-10-04 22:23:00.0 | 546            | true    |
| 2              | 2       | First Changes | SQL  | V2__First_Changes.sql | 1279644856 | axel         | 2016-10-06 09:18:00.0 | 127            | true    |

Uma vez alcançado o estado inicial de migração, o Flyway efetuará um novo **scan** no sistema de arquivos ou no _classpath_ da aplicação para migrações. As migrações são comparadas com a tabela de metadados e, se o número desta versão for menor ou igual à versão marcada como corrente, tais migrações são ignoradas. As migrações restantes ficam disponíveis, porém não aplicadas, e são denominadas como **migrações pendentes**. Logo após, as migrações pendentes são **classificadas por número de versão** e **executadas em ordem**:

![Migração pendente](https://i.imgur.com/OWkyvGx.png)
<sub><sup>_**Migração v. 2.1, pendente**_</sup></sub>


![Execução da Migração](https://i.imgur.com/xRDcysE.png)
<sub><sup>_**Migração em execução**_</sup></sub>


Ao fim do processo, a tabela de metadados é atualizada conforme o exemplo:

| installed_rank | version | description   | type | script                | checksum   | installed_by | installed_on          | execution_time | success |
|----------------|---------|---------------|------|-----------------------|------------|--------------|-----------------------|----------------|---------|
| 1              | 1       | Initial Setup | SQL  | V1__Initial_Setup.sql | 1996767037 | axel         | 2016-10-04 22:23:00.0 | 546            | true    |
| 2              | 2       | First Changes | SQL  | V2__First_Changes.sql | 1279644856 | axel         | 2016-10-06 09:18:00.0 | 127            | true    |
| 3              | 2.1     | Refactoring   | JDBC | V2_1__Refactoring     |            | axel         | 2016-10-10 17:45:05.4 | 251            | true    |

Dessa forma, sempre que houver a necessidade de evolução de um banco de dados, basta criar uma nova migração com uma versão superior à corrente que na próxima execução do Flyway, tal migração será identificada e o processo de atualização do banco de dados será iniciado.

## 4. Primeiros Passos na Utilização do Programa

#### Download e extração do Flyway

Por tratar-se de uma ferramenta de execução por linha de comando, não há a necessidade de criação de instaladores para diferentes plataformas. A preparação para a utilização se dá pela extração dos arquivos compactados em um diretório escolhido pelo usuário ou execução dos pacotes _.JAR_, disponíveis no [website da ferramenta](https://flywaydb.org/getstarted/download).

#### Configuração

Após extração dos arquivos, é necessário configurar o Flyway editando o arquivo ```/conf/flyway.conf```  conforme exemplo abaixo:

>\# MySQL: jdbc:mysql://<host>:<port>/<database>?
>flyway.url=```jdbc:mysql://serverName:port/database?autoReconnect=false&useSSL=false```

>\# User to use to connect to the database. Flyway will prompt you to enter it if not specified.
>flyway.user=```engsoft2```

>\# Password to use to connect to the database. Flyway will prompt you to enter it if not specified.
>flyway.password=```########```

#### Criação da primeira migração

A primeira migração da base de dados vazia é possível ser criada no diretório ```/sql``` a qual, nesse exmeplo, é dado o nome ```V1_Create_person_table.sql```. 

```sql=
create table PERSON (
    ID int not null,
    NAME varchar(100) not null
);
```
#### Migração da base de dados

Para migrar a base de dados, o seguinte comando deve ser digitado:
```shell
flyway-4.0.3> flyway migrate
```
Se tudo der certo, a seguinte saída é obtida na tela:
```sh
Creating Metadata table: "PUBLIC"."schema_version"
Current version of schema "PUBLIC": << Empty Schema >>
Migrating schema "PUBLIC" to version 1 - Create person table
Successfully applied 1 migration to schema "PUBLIC" (execution time 00:00.062s).
```

#### Adicionando uma outra migração

Ao adicionar uma segunda migração denominada ```V2_Add_people.sql``` ao diretório ```/sql```:

```sql=
insert into PERSON (ID, NAME) values (1, 'Axel');
insert into PERSON (ID, NAME) values (2, 'Mr. Foo');
insert into PERSON (ID, NAME) values (3, 'Ms. Bar');
```
e executar nvoamente o comando de migração:
```shell
flyway-4.0.3> flyway migrate
```
obtem-se como resultado:
```sh
Current version of schema "PUBLIC": 1
Migrating schema "PUBLIC" to version 2 - Add people
Successfully applied 1 migration to schema "PUBLIC" (execution time 00:00.090s).
```
o que caracteriza que as migrações foram encontradas e executadas com sucesso.

### 4.1 Comandos

O sistema possui apenas seis comandados :

| Nome     | Descrição                                                                                                    |
|----------|--------------------------------------------------------------------------------------------------------------|
| clean    | Executa o comando _**drop**_ em todos os objetos do _schema_ configurado                                             |
| info     | Apresenta os detalhes e informação do status de todas as migrações                                           |
| migrate  | Efetua a migração da base de dados                                                                           |
| validade | Valida as migrações aplicadas frente àquelas disponíveis no _classpath_                                        |
| baseline | Cria um patamar para a base de dados existente, excluindo todas as migrações e incluindo a versão do patamar |
| repair   | Repara a tabela de metadados                                                                                 |

### 4.2 Visão de Caso de Uso 
Neste tópico, reproduzimos um caso de uso de um tutorial disponível no blog [Novatec](http://blog.novatec-gmbh.de/flyway-database-migrations-made-easy/), demonstrando como o Flyway pode ser integrado em uma aplicação existente. O autor trabalha em uma aplicação de exemplo que utiliza _EclipseLink_ como um provedor _JPA_ e _Spring-Framework_ para _injeção de dependência (DI)_. Há também o uso de Maven para gerenciamento de dependência e para construção da aplicação. A migração da base de dados é executada em tempo de construção e também em tempo de execução.

#### 4.2.1 Integração das dependências do Flyway

A utilização do Maven facilita o gerenciamento de bibliotecas de terceiros. A dependência _'flyway-core'_ com a última versão deve ser incluída aos arquivos do projeto _'pom'_, se necessário uma migração de base de dados com classes Java, confome o trecho de código abaixo:
```xml=
<dependency>
	<groupId>org.flywaydb</groupId>
	<artifactId>flyway-core</artifactId>
	<version>3.0</version>
</dependency>
```
Para executar uma migração no processo de construção, é importante adicionar o plugin _'flyway-maven'_ ao arquivo _'pom'_ e configurá-lo para estabelecer uma conexão ao banco de dados. O seguinte trecho de código mostra os procedimentos de adição do plugin e de definição de credenciais de acesso à instância do banco de dados DB2:

```xml=
<plugin>
	<groupId>org.flywaydb</groupId>
	<artifactId>flyway-maven-plugin</artifactId>
	<version>3.0</version>

	<dependencies>
		<dependency>
			<groupId>com.ibm</groupId>
			<artifactId>db2jcc</artifactId>
			<version>4.0</version>
		</dependency>
	</dependencies>

	<configuration>
		<user>db2admin</user>
		<password>db2admin</password>
		<url>jdbc:db2://localhost:50000/FLYWEXAM</url>
		<locations>
			<location>filesystem:src/main/resources/migrations</location>
		</locations>
		<skip>false</skip>
	</configuration>
</plugin>
```
#### 4.2.2 Configuração base

Partindo-se do pressuposto da não existência da tabela ```SCHEMA_VERSION```, o Flyway procederá conforme os passos _descritos no **item 3**_ para a criação da tabela de metadados. Caso haja a base de dados sem, contudo, ter sido preenchida pelo Flyway, basta proceder conforme segue:

Primeiramente, criando e salvando um _"snapshot"_ ou um script de migração da base de dados existente que recriará o atual estado da base de dados. Logo após, é necessário a abertura do _shell_, navegar até o diretório da amostra do projeto e executar o seguinte comando:

```sh
mvn flyway:clean
```
responsável por apagar todas as tabelas e dados na base de dados. Esse passo garante que mais migrações estejam funcionando corretamente. Abaixo, segue uma figura mostrando o resultado de um processo de limpeza.
![](https://i.imgur.com/c0bK8Ce.jpg)
<sup><sub>**Processo de limpeza de dados**</sub></sup>

Após o processo de limpeza, procede-se com a geração da tabela ```SCHEMA_VERSION```, através do seguinte comando:
```sh
mvn flyway:init -Dflyway.initVersion=<version> –Dflyway.initDescription=<description>
```
que desencadeará o seguinte processo:
![](https://i.imgur.com/wxjvdRO.jpg)
<sup><sub>**Processo de geração da tabela _SCHEMA_VERSION_**</sub></sup>

Como se observa na figura, a tabela _'SCHEMA_VERSION'_ foi criada e inicializada com a versão 1.
![](https://i.imgur.com/30hx5zM.jpg)

Após criada a tabela, procede-se com a importação do _"snapshot"_ ou migração criado anteriormente. O arquivo deve ser movido para a pasta do repositório para ser identificado pelo Flyway. 
> É importante customizar o nome do arquivo de migração, visto que o Flyway apenas localiza arquivos que são válidos pela convenção típica de nomeação da ferramenta: um prefixo (por padrão: _**V**_), um número de versão que consiste de uma ou mais partes numéricas _separadas por um **ponto**_ (.) ou _**um underscore**_ (_), _**dois underscores**_ (__) como delimitador e uma descrição, dessa forma:
> ```sh=
> [prefixo][versão]__[descricao].sql
> ```

A seguinte figura mostra como o autor do tutorial customizou o arquivo:
![](https://i.imgur.com/bdDaZT7.jpg)

Para a recuperação dos dados, executa-se o seguinte comando:
```sh
mvn flyway:migrate
```
As próximas figuras mostram que a base de dados foi restaurada, que a versão mudou de 1.0 para 1.1 e que houve uma nova inserção na tabela _SCHEMA_VERSION_, comprovando que o _"snapshot"_ foi migrado com sucesso.

![](https://i.imgur.com/Zr8jVxf.jpg)

![](https://i.imgur.com/u9sCJkn.jpg)

Isso comprova que os dados produzidos foram restaurados. Todas as migrações seguintes são executadas seguindo os mesmos passos, tendo que tomar o cuidado de criar uma nova migração com uma versão maior que a atual. Após iniciado o Flyway, ele encontrará e atualizará a base de dados corretamente.

#### 4.2.3 Configuração para migração Java

Neste passo, a aplicação de exemplo usa o framework Spring para DI, cujas dependências encontram-se definidas no arquivo de projeto ```pom.xml```. Há a necessidade de reconfiguração do aplicação ```ApplicationContext``` para que haja conferência do estado da base de dados durante o _bootstrap_. A migração java é necessária no caso de _BLOBs_ ou _CLOBs_ serem inseridos. É considerado usual a mudança de uma quantidade massiva de dados ou para migrações que requeiram o uso de expressão regular.

O trecho de código abaixo mostra a configuração típica do ```dataSource``` para o ```EntityManagerFactoryBean``` e o ```PlatformTransactionManager``` com Spring.

```java=
package info.novatec.eap.persistence.config;

import java.util.HashMap;
import java.util.Map;

import javax.sql.DataSource;

import org.flywaydb.core.Flyway;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.DependsOn;
import org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.JpaVendorAdapter;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.orm.jpa.vendor.EclipseLinkJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

@Configuration
@ComponentScan("info.novatec.eap.persistence")
@EnableTransactionManagement
public class JpaConfig {

	@Bean
	public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean() {
		final LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
		localContainerEntityManagerFactoryBean.setDataSource(this.dataSource());
		localContainerEntityManagerFactoryBean.setPackagesToScan("info.novatec.eap.persistence");
		localContainerEntityManagerFactoryBean.setJpaVendorAdapter(this.jpaVendorAdapter());
		localContainerEntityManagerFactoryBean.setJpaPropertyMap(this.jpaProperties());

		return localContainerEntityManagerFactoryBean;
	}

	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		final HikariConfig config = new HikariConfig();
		config.setMaximumPoolSize(10);
		config.setDataSourceClassName("com.ibm.db2.jcc.DB2SimpleDataSource");
		config.setPoolName("testPool");
		config.addDataSourceProperty("serverName", "localhost");
		config.addDataSourceProperty("databaseName", "FLYWEXAM");
		config.addDataSourceProperty("user", "db2admin");
		config.addDataSourceProperty("portNumber", "50000");
		config.addDataSourceProperty("password", "db2admin");

		return new HikariDataSource(config);
	}

	private Map<String, Object> jpaProperties() {
		final Map<String, Object> map = new HashMap<String, Object>();
		map.put("eclipselink.weaving", "false");

		return map;
	}

	private JpaVendorAdapter jpaVendorAdapter() {
		final JpaVendorAdapter jpaVendorAdapter = new EclipseLinkJpaVendorAdapter() {
			{
				setShowSql(true);
				setDatabase(Database.DB2);
				setGenerateDdl(false);
			}
		};

		return jpaVendorAdapter;
	}

	@Bean
	public PlatformTransactionManager transactionManager() {
		return new JpaTransactionManager(this.entityManagerFactoryBean().getObject());
	}

	@Bean
	public PersistenceExceptionTranslationPostProcessor exceptionTranslator() {
		return new PersistenceExceptionTranslationPostProcessor();
	}
}
```
Para se ter acesso às classes de migração java, deve-se criar um pacote que servirá como repositório para o Flyway. Para este exemplo, o autor criou um pacote chamado ```info.novatec.eap.persistence.migration```.

![](https://i.imgur.com/eIMg0nU.jpg)

Após, foi criado um novo _bean_ para o Flwyay:

```java=
@Bean(initMethod = "migrate")
public Flyway flyway() {
	final Flyway flyway = new Flyway();
	flyway.setDataSource(this.dataSource());
	flyway.setLocations("info.novatec.eap.persistence.migration", "filesystem:src/main/resources/migrations");

	return flyway;
}
```
Nesse contexto, foi iniciada a classe ```Flyway``` e inicializado ```dataSource``` com um _"setter"_. Com ```setLocations``` foi definido o caminho do pacote que contém os arquivos de migração Java. Durante a migração, o Flyway checa se todos os arquivos migrados estão disponíveis. Para o processo implícito de migração, é importante ```initMethod=migrate``` como propriedade _bean_. Essa configuração garante que o Flyway cheque a versão da base de dados e migre para um novo estado enquanto fa o _bootstrap_ da aplicação.

Para garantir que ```EntityManagerFactoryBean``` inicie após a migração, usa-se a anotação ```@DependsOn(“flyway”)```. O próximo trecho de código mostra a classe de configuração editada.

```java=
@Bean
@DependsOn(value = { "flyway" })
public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean() {
	final LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
	localContainerEntityManagerFactoryBean.setDataSource(this.dataSource());
	localContainerEntityManagerFactoryBean.setPackagesToScan("info.novatec.eap.persistence");
	localContainerEntityManagerFactoryBean.setJpaVendorAdapter(this.jpaVendorAdapter());
	localContainerEntityManagerFactoryBean.setJpaPropertyMap(this.jpaProperties());

	return localContainerEntityManagerFactoryBean;
}
```

#### 4.2.4 Criação do arquivo de migração do tipo Java

As migrações Java possuem as mesmas convenções de nomes dados aos scripts de migração SQL. Devem ser espcificados como um nome de classe. Cada classe de migração Java deve implementar a interface ```JdbcMigration```. Abaixo segue um trecho de código  de uma classe de migração implementada corretamente.

```java=
package info.novatec.eap.persistence.migration;

import java.sql.Connection;
import java.sql.PreparedStatement;

import org.flywaydb.core.api.migration.jdbc.JdbcMigration;

public class V2_0__enhancementOnCustomerTable implements JdbcMigration {

	@Override
	public void migrate(Connection connection) throws Exception {
		final PreparedStatement statement = connection.prepareStatement("INSERT INTO CUSTOMER (VERSION,TIMESTAMP,FIRSTNAME,LASTNAME,EMAIL) "
				+ "VALUES (1,'2014-09-01 09:49:00.0','Max','Mustermann','max.mustermann@blub.de')");
		try {
			statement.execute();
		} finally {
			statement.close();
		}
	}
}
```
Uma vez que dada aplicação usa o framework Spring, é necessário o uso da classe ```SpringJdbcMigration``` ao invés de ```JdbcMigration```. Os processos de execução de ambas as classes são idênticos, sendo diferenciado apenas pelo fato de que ```SpringJdbcMigration``` habilita o uso do _template_ da classe ```SpringJDBC``` para definir declarações SQL. A classe a seguir implementa a interface ```SpringJdbcMigration```:

```java=
package info.novatec.eap.persistence.migration;

import org.flywaydb.core.api.migration.spring.SpringJdbcMigration;
import org.springframework.jdbc.core.JdbcTemplate;

public class V2_1__addDataToCustomer implements SpringJdbcMigration {

	@Override
	public void migrate(JdbcTemplate jdbcTemplate) throws Exception {
		jdbcTemplate.execute("INSERT INTO CUSTOMER (VERSION,TIMESTAMP,FIRSTNAME,LASTNAME,EMAIL) "
				+ "VALUES (1,'2014-09-01 09:49:00.0','Max','Mustermann','max.mustermann@blub.de')");
	}
}
```

#### 4.2.5 Execução

Uma migração é executada se necessário, enquanto é executado o _bootstrap_ da aplicação. Um segundo ponto chave, como descrito no item anterior, foi implementado pelo autor.

Maven também executa migrações de classe Java. Para esse propósito, é necessário ajustar o plugin ```flyway-maven```. Um segundo local com o nome do pacote que foi criado antes deve ser adicionado. O próximo trecho de código mostra como isso foi definido:

```xml=
<plugin>
	<groupId>org.flywaydb</groupId>
	<artifactId>flyway-maven-plugin</artifactId>
	<version>3.0</version>

	<dependencies>
		<dependency>
			<groupId>com.ibm</groupId>
			<artifactId>db2jcc</artifactId>
			<version>4.0</version>
		</dependency>
	</dependencies>

	<configuration>
		<user>db2admin</user>
		<password>db2admin</password>
		<url>jdbc:db2://localhost:50000/FLYWEXAM</url>
		<locations>
			<location>filesystem:src/main/resources/migrations</location>
			<location>classpath:info.novatec.eap.persistence.migration</location>
		</locations>
		<skip>false</skip>
	</configuration>
</plugin>
```
Tal configuração habilita o Flyway para acessar as migrações de classes Java e considerá-las para a migração. As próximas imagens mostram novamente a tabela _SCHEMA_VERSION_. A coluna _script_ mostra que tanto as _migrações Java_ quanto as _migrações SQL_ foram executadas na base de dados.
![](https://i.imgur.com/2rvBEyd.jpg)

Os documentos que exemplificam esse estudo de caso podem ser encontrados no [site oficial do Flyway](https://flywaydb.org/documentation/) e a aplicação de exemplo no [site da Novatec](http://blog.novatec-gmbh.de/wp-content/uploads/2014/09/flyway-sample.zip).

## 5. Equipe de Desenvolvimento.
A equipe de desenvolvimento conta com um grande número de colaboradores que contribuem significativamente para o desenvolvimento e evolução da Flyway. Dos colaboradores 59 já realizaram pelo menos um comit. 
Os principais colaboradores e algumas de suas contribuições são:

Nome|Usuário|Contribuição|Comits
---------|------|--------------------------|-----------------
|Axel Fontaine|axelfontaine	|Project co-founder and lead| 583
|Aurélien Mino|	murdos|PostgreSQL clean fix|5
|Christian Dedié|cdedie	|Project co-founder, SBT plugin|17
|Christine Teig|cteig	|DB2 z/OS support|7
|Michael Forstner|MichaelF25|solidDB support|8
|Piotr Wielgolaski|pwielgolaski|MigrationVersion performance, Gradle plugin fixes, Cygwin support, Travis-CI integration improvements|21
|Sanjay Deshmukh|sanjayd|Quiet mode for command-line & current version for target|4
|Stephan Pauxberger|pauxus|FlywayConfiguration, skipDefaultResolvers and DB2 Trigger clean|5

O Hall da Fama da lista de colaboradores consta de pelo menos 106 colaboradores.
Disponível em: https://flywaydb.org/documentation/contribute/hallOfFame

#### 5.1 Como contribuir
A maneira mais fácil de contribuir com a ferramenta é o compartilhamento de conhecimento da ferramenta ao responder as perguntas postadas no site stackoverflow com a tag flyway. http://stackoverflow.com/questions/tagged/flyway?page=1&sort=newest&pagesize=15
O feed de dúvidas mais recente contém:
	*Flyway to execute some scripts before migrations to provide some sprocs in my schema, postado em 09-10-2016.
	* How to create all models with flyway in spring? Postado em 05-10-2016.
	* Flyway Over SSH tunnel, postado em 06-10-2016.
	* Changing the baseline record for flyway, postado em 04-10-2016.
	* Flyway with Spring: Can I have SQL and Java based migrations? Postado
	  em 29-09-2016.
O colaborador pode se increver no feed através do site: http://stackoverflow.com/feeds/tag?tagnames=flyway&sort=newest

Outra forma de colaborar é melhorar o site da ferramenta que também está disponível em https://flywaydb.org/documentation/contribute/website

#### 5.2 Issues
Ao encontar um bug do sistema é preciso abrir uma issue, relatando o ocorrido e para isso deve-se indicar: 
 - SO utilizado, como foi a execução do flyway até encontrar o bug, versão utilizada, é indispensável avaliar se outro colaborardor já relatou o  mesmo problema. Caso encontre um relato do mesmo problema basta apenas incrementar o comentário indicando mais 1 para o problema observado.

Já foram abertas 1226 issues, sendo 1038 issues finalizadas ou fechadas e 188 ainda abertas. 

#### 5.3 Labels
O sistema apresenta 40 etiquetas, as mais relevantes  e suas citações em issues estão abaixo indicadas:
Label|Quant Issues|
-----|---------|
|t: feature|104|
| m: Core |59|
|t: bug|50|
| s: waiting for feedback |25|
| s: sponsoring welcome |14|
| s: pull request welcome |11|
| d: PostgreSQL |10|
| m: Command-line |9|
| s: needs cleanup |7|
| d: Oracle |7|
| m: Maven |7|
| m: Gradle |7|
| d: SqlServer |7|
| c: migrate() |6|
| m: SBT |5|
| o: Android |2|
| d: MySQL|3|
Observa-se a citação de duas no máximo três labels em cada issue.

#### 5.4 Componente Crítico


## 6. Evolução do Sistema

O sistema Flyway apresenta uma dinâmica de releases acentuada. Desde seu lançamento em  abril de 2010, até a última versão publicada em junho de 2016, foram mais de 15 atualizações, uma média de 3 versões/ano.

#### 6.1 Principais releases e suas funcionalidades

##### - Flyway 0.0.1 - a primeira versão do flyway lançada em 20/04/2010. 
##### - Da versão 0.0.1 até a versão 1.0  - durante o ano de  2010 ocorreram 9 correções de bugs, graças a ajuda de usuários que relataram os problemas, 18 alterações e 29 novos recursos foram adicionados.

Veja abaixo, lista das principais alterações retiradas do site do flyway.

##### novos Recursos 
- Adicionado suporte para o estilo Ant $ {espaço reservado} substituição em migrações sql.
- Adicionado suporte para oracle blocos PL / SQL em migrações sql.
- Adicionado suporte para multi-linha comentários / * * / em scripts SQL.
- Adicionado suporte hsqldb.
- Adicionado Maven Plugin com Migrate mojo (flyway mvn: Migrar).
- suporte de banco de dados H2 Adicionado.
- Adicionado suporte para migrações paralelas. Flyway agora é seguro para implantar em um cluster.

- Logging fixados em Maven Plugin.
- Pacote base alterado para com.googlecode.flyway para coincidir com Maven groupId.

- Adicionado flyway mvn: gol histórico para exibir o histórico de migração completa do esquema.

- Migrações Java agora herdam com.googlecode.flyway.core.migration.java.BaseJavaMigration.
- Erros fixos todos conhecidos.
- Suporte para somas de verificação de migração (CRC-32 para SQL Migrações, implementação personalizada opcionais para migrações de Java).
- Validação de detecção de alterações o funcionamento acidental opcional (incluindo somas de verificação) antes da migração.
- Nomes de arquivos: migrações Sql podem agora ter um prefixo configurável (Padrão: V) e um sufixo (Padrão: SQL).

- Edição 62: Adicionado suporte para directivas comentário MySQL (/*!...*/;) para MySQL lixeiras podem ser usadas como migrações.
- Issue 61: Reduzida a informação registrando uma única linha por migração.
- Edição 36: Adicionado suporte para especificar uma versão de destino até que Flyway deve executar migrações.
- Adicionado nova propriedade ignoreFailedFutureMigration ignorar falhas de migração que aconteceram durante a implantação de uma versão mais recente do software. Isto permite reversões de software sem ter que fazer reversões de banco de dados, com o custo do enfraquecimento falhar rapidamente.
- Edição 31: Flyway agora tem sua própria hierarquia de exceções. Todas as exceções lançadas agora são do tipo FlywayException.
- Edição 44: Flyway podem agora encontrar migrações SQL, mesmo que sejam em subpastas de BASEDIR.
- Limpar agora limpa a lixeira do Oracle.


##### Alterarações
- Exibir nome do script exato e o número da linha quando uma instrução não pôde ser executada.
- Melhor suporte para blocos PL / SQL com uma seção de declaração.
- Maven atualizados GROUPID para com.googlecode.flyway.
- Melhor desempenho e saída para o log.
- A codificação dos arquivos de Migração do SQL agora é configurável.
- mvn flyway: agora a opção limpa esta mais robusta e disponível para todos os bancos de dados suportados.
- O formato de tabela de metadados foi estendida com novas colunas (tipo, soma de verificação, installed_by). tabelas de metadados existentes serão atualizados automaticamente na primeira execução, nenhuma intervenção manual é necessária.
- mvn flyway: Migre agora aciona automaticamente a fase de compilação para garantir que as novas migrações são encontrados pelo plugin.

- Issue 30: migrações Java agora são baseados em interfaces em vez de herança. O existente BaseJavaMigration classe foi preterido e adaptados a este novo modelo.

##### Correções de bugs
 - Corrigido indice da versão atual no Oracle.
 - parsing fixa de CREATE FUNCTION e instruções CREATE PROCEDURE em PL / SQL.
- Corrigido um bug no Oracle SQL Analisador que causou o tropeço de nomes das colunas que começam com 'começar'.
- Definir a fonte de dados antes do nome da mesma estava causando um erro. Isso também afetou o plugin Maven.
- Issue 65: novas linhas estão agora preservados em declarações para melhorar a legibilidade dos procedimentos armazenados.
- Corrigido um problema com tabelas da Oracle índice espacial causando uma falha ao tentar limpar.
- Issue 72: Flyway.configure (Propriedades) também funciona quando a fonte de dados é definido explicitamente de antemão.


##### - Da versão 1.0.1 até a versão 1.5  - de dezembro de 2010 a novembro de 2011 ocorreram 46 correções de bugs, graças a ajuda de usuários que relataram os problemas, 8 alterações e 11 novos recursos foram adicionados.

Veja abaixo, lista das principais alterações retiradas do site do flyway.



##### Novos Recursos 
- Edição 43: ferramenta Standalone para executar Flyway diretamente a partir da linha de comando.
- Issue 19: suporte SQL Server.
- Issue 106: Suporte à esquemas múltiplos.
- Issue 116: Suporte ao Java 5.
- Issue 136: Suporte para as credenciais do servidor de settings.xml (plugin Maven).

##### Alterarações
- Flyway agora é testado com o MySQL versão 5.1 (acima de 5,0).

##### Correções de bugs
- Edição 78: da Primavera JDBC dependência ausente do POM.
- Suporte para ponto e vírgula no final de uma linha dentro de uma string literal.
- Issue 92: Flyway pode estar vazando conexões de banco de dados.
- Suporte para ponto e vírgula no final de uma linha dentro de uma string literal (H2).
- Suporte para instruções CREATE PACOTE (Oracle).
- Issue 104: mvn flyway - clean falha com SQL-Exception quando o usuário db não tem permissões de DBA (Oracle XE).
- Issue 102: Oracle: Flyway agora lança uma exceção quando uma limpeza é tentada no esquema SYSTEM.
- Edição 115: A ferramenta de linha de comando não suporta propriedade -target.
- Issue 148: linha de comando Flyway retorna status de saída 0, mesmo em caso de falha.
- Issue 152: NullPointerExcpetion Null.
- Issue 153 ferramenta de linha de comando 1.4.2 (Windows) pasta sql - um comportamento inesperado.
- Issue 154 ferramenta de linha de comando 1.4.2 (Windows) - validar alterações em migrações executadas não acontecendo.


##### - Da versão 1.6 até a versão 2.0.3  - de abril de 2012 a dezembro de 2012 ocorreram 71 correções de bugs, graças a ajuda de usuários que relataram os problemas, 28 alterações e 7 novos recursos foram adicionados.

Veja abaixo, lista das principais alterações retiradas do site do flyway.

##### Novos Recursos
Edição 42: Suporte do Derby.
Issue 197: suporte do Google Cloud SQL.

##### Alterarações
- Flyway agora é construído com o JDK 6, mas ainda compatível com JDK 5.
- Issue 59: Primavera Dependency agora é opcional.
- Núcleo: Métodos obsoletos removidos. Flyway.get / setTransactionManager (). Eles já não tinham mais nenhum efeito desde 1.6.
- Oracle: Para ser compatível com SQLPlus, Flyway agora sempre espera / para encerrar uma instrução criar tipo.

##### Correções de bugs
- Issue 79: Migrações não suportam vírgulas dentro de aspas.
- Issue 184: migrações SQL força enorme espaço de pilha para fazer flyway migre.
- Issue 186: Javadoc do método getBaseDir da classe Flyway não é consistente com a implementação.
- Issue 225: Limpa para SQL Server não remove User Defined Tipos de Tabela.
- Issue 262: Manipulação incorreta de cotações do dólar aninhados em scripts do PostgreSQL.
- Issue 264: Obtendo FlywayException ao executar maven flyway.
- Issue 304: migração SQL com instruções INSERT de várias linhas leva muito tempo.
- Issue 358: Flyway 2.0 pode quebrar a validação das migrações PostgreSQL aplicada porque remove "público" do caminho de pesquisa.
- Issue 361: flyway: limpa não funciona com o Oracle XML DB e XMLINDEX.
- Issue 375: Flyway é extremamente lento em MySQL com muitos esquemas.


##### - Da versão 2.1 até a versão 2.3  - de março de 2013 a dezembro de 2013 ocorreram 70 correções de bugs, graças a ajuda de usuários que relataram os problemas, 9 alterações e 11 novos recursos foram adicionados.

Veja abaixo, lista das principais alterações retiradas do site do flyway.

##### Novos Recursos
- Issue 74: Criação de esquema automático.
- Plugin Maven: suportar senhas criptografadas em settings.xml.
- Issue 247: Permitir migrações carregamento de SQL a partir do Sistema de Arquivos (em vez de apenas a partir de classpath).
- Issue 324: Flyway.driver agora é opcional. É detectado automaticamente baseado em flyway.url quando omitidos.
- Issue 78: Gradle Plugin.
- Issue 182: Suporte SQL Azure.
- Issue 85: Suporte SBT Plugin.
- Issue 562: suporte Cygwin.

##### Alterarações
- Flyway agora impõe as regras para o que constitui uma versão válida .
- Issue 244: Flyway agora também pode ser construído com o JDK 7 e acima. (Mínimo ainda é JDK 6).

##### Correções de bugs
- Issue 351: melhores mensagens de log de DBMigrator.
- Issue 383: suporte usernames mais longos.
- Issue 427: Suporte para blocos anônimos no PostgreSQL.
- Issue 439: MySQL Cluster falhando ao sincronizar.
- Issue 462: Flyway 2.1.1 trava no comando ANALYZE.
- Issue 548: Precisa de uma maneira de descobrir a partir do FlywayException qual comando SQL falhou.
- Issue 559: NullPointerException no banco de dados sem esquema padrão.
- Issue 571: Plugin falha com Maven 3.1 - Uma incompatibilidade na API foi encontrada durante a configuração do mojo.


##### - Em 20/04/2014 - Flyway 3.0: Hello Android!

Foi lançada com 462 comits e expandiu a ferramenta para os bancos de dados MariaDB, suporte para Android e SQlite.

##### - Em 27/11/2014 - Flyway 3.1

Redshift, Vertica and DB2 z/OS support
SQL-based callbacks
JDBC drivers shipping with the command-line tool
Renamed init into baseline

##### - Em 08/03/2015 - Flyway 3.2
    
Many improvements to the command-line tool
    solidDB support
    Improved JavaBean-style configuration

##### - Em 20/03/2015 - Flyway 3.2.1
	
Bug fix for Oracle regression
	
##### - Em 29/02/2016 - Flyway 4.0
    
Repeatable Migrations support
    Sybase ASE, SAP HANA and Apache Phoenix support
    Java-based migration and callback enhancements
    Disabling clean
    Datasource auto-configuration when running in Boxfuse instances
    Commercial support plans

##### - Em 06/05/2016 - Flyway 4.0.1

##### - Em 09/06/2016 - Flyway 4.0.2

##### - Em 17/06/2016 - Flyway 4.0.3

## 7. Ferramentas - Frameworks e Linguagens 

#### 7.1 Principais ferramentas utilizadas
1 - GIT
2 - JDK
3 - JREs
4 - MAVEN
5 - IDE
- Frameworks

- Linguagens
Java

## 8. Principais Módulos, componentes e Arquitetura do Sistema

O sistema flyway possui uma arquitetura muito simples. Basicamente Flyway suporta várias ferramentas de banco de dados, entre as mais populares citamos PostgreeSQL, IBM-DB2, MySQL, SQLServer, Azure dentre outras. Ele pode ser executado como um Plugin de Maven, ou em uma API java, ou por linha de comando. Dessa forma há uma camada de suporte e integração com as ferramentas de BD e internamente a esta API de integração, há um conjunto de scripts e comandos nos módulos abaixo indicados.    
##### 8.1 Principais Módulos e Responsabilidades
###### API: 

###### Migration Resolver:

###### Migration Executor:

###### Metadata Table Management:

###### Database Specific Support:

##### 8.2 Arquitetura
A figura X  - Arquitetura do sistema demonstra uma visão geral da arquitetura do sistema.
 
![](https://i.imgur.com/4S4Xuhz.jpg)
Figura X - Arquitetura do Sistema.

Os componentes mais relevantes identificados foram:

|Componente | Descrição|
|-----------|-----------|
|Core       | 
|
|
|
|
Estou tentando uma forma de fazer a descrição dos componentes mas isso é bem dificil, alguém tem alguma ideia ? 
Fabiano: Acho que não deveriamos descrever. Vai ser trabalhoso e já temos muita coisa escrita.




##### 8.3 Diagramas
![](https://i.imgur.com/N02gIWz.jpg)
<sup><sub>**Diagrama de Classe do arquivo .jar de linha de comando**</sub></sup>


<sup><sub>**Diagrama de Classe do arquivo .jar de core**</sub></sup>




## 9. Referências
1 - [https://flywaydb.org/documentation/], consulta em 07/10/2016.
2 - [https://flywaydb.org/getstarted/], consulta em 07/10/2016.
3 - [http://www.linhadecodigo.com.br/artigo/3343/como-documentar-a-arquitetura-de-      software.aspx], consulta em 06/10/2016.
4 - [https://vimeo.com/74437803], consulta em 12/10/2016.
5 - [http://blog.novatec-gmbh.de/flyway-database-migrations-made-easy/], consulta     em 08/10/2016. 









