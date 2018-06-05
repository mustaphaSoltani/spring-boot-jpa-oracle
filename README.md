# spring-boot-jpa-oracle

## Objectif:
Créer un exemple de pool de connexions par Spring Boot , Spring Data JPA , Oracle et HikariCP.  
Créer une interface et étendre les données via CrudRepository qui sont chargées par Hibernate automatiquement à partir du fichier import.sql.

---

## Les outils utilisés dans ce projet sont:

* *__Spring Boot 1.5.1.RELEASE__* :  permet de créer facilement des applications Spring

* *__Spring Data 1.13.0.RELEASE__* : vise à améliorer la mise en œuvre de la couche d'accès aux données.  
    Avec Spring-Data, écrire des méthodes de requête à la base se fait en 3+1 étapes:  
	    * Déclarer une interface Repository,  
	    * Ecrire la signature de la méthode de requête (CRUD ou de recherche),  
	    * Configurer Spring pour la génération automatique,  
	    * Appeler dans le client (ou test unitaire) les méthodes du Repository  

* *__Hibernate 5__* : est une solution open source de type ORM (Object Relational Mapping) qui permet de faciliter le développement de la couche persistance d'une application et la recherche de données dans une base de données. Hibernate permet donc de représenter une base de données en objets Java et vice versa.

* *__Oracle database 11g express__* : un système de gestion de base de données relationnelle (SGBDR) 

* *__Oracle JDBC driver ojdbc7.jar__*

* *__HikariCP 2.6__* : Il s'agit d'un framework de pooling de connexions JDBC très léger (Un pool de connexions est un cache de connexions de base de données maintenu de sorte que les connexions peuvent être réutilisées lorsque des demandes futures à la base de données sont requises)

* *__Maven__*: Maven est un outil de gestion et d'automatisation de production des projets logiciels Java en général et Java EE en particulier.
          Chaque projet ou sous-projet est configuré par un POM, se matérialise par un fichier pom.xml à la racine du projet, qui contient les informations nécessaires à Maven pour traiter le projet (nom du projet, numéro de version, dépendances vers d'autres projets, bibliothèques nécessaires à la compilation, noms des contributeurs, etc.)

* *__Java 8__*
-----------------
## 1. Structure du projet 
Une structure de projet Maven standard.

![1](https://user-images.githubusercontent.com/28655112/40979022-898a0a72-68cc-11e8-99d7-50a7b5b3e87f.PNG)

## 2. Dépendance du projet 
Déclare spring-boot-starter-data-jpa, il saisit les données Spring Data, Hibernate et JPA.
```bash
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- Spring data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.tomcat</groupId>
                    <artifactId>tomcat-jdbc</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- Oracle JDBC driver -->
        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc7</artifactId>
            <version>12.1.0</version>
        </dependency>
        <!-- HikariCP connection pool -->
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>2.6.0</version>
        </dependency>
    </dependencies>
  ```
## 2. Java Persistence API – JPA
  ### 2.1 Modèle client.
  Ajoutez des annotations JPA et utilisez "sequence" pour générer l'ID pour l'incrémentation automatique.
```bash 
  @Entity
public class Customer {
	// "customer_seq" is Oracle sequence name.
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "CUST_SEQ")
    @SequenceGenerator(sequenceName = "customer_seq", allocationSize = 1, name = "CUST_SEQ")
    Long id;
    
    String name;
    String email;

    @Column(name = "CREATED_DATE")
    Date date;
    //getters and setters, contructors
}
```

## 3. Comment Configurer et Initialiser la base de données?
- Configuration la source de données Oracle dans application.propreties: 
```bash
           spring.datasource.url=jdbc:oracle:thin:@localhost:1521:xe
           spring.datasource.username=system
           spring.datasource.password=password
           spring.datasource.driver-class-oracle.jdbc.driver.OracleDriver
```

- Ajouter les paramètres HikariCP dans application.propreties:
```bash
           spring.datasource.hikari.connection-timeout=60000
           spring.datasource.hikari.maximum-pool-size=5
```
- Ajouter la dependance Oracle dans pom.xml; 
```bash
        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc7</artifactId>
            <version>12.1.0</version>
        </dependency>
  ```
 ## 4. Repository
 Créer une interface et étendre Spring Data CrudRepository
  ```bash      
  public interface CustomerRepository extends CrudRepository<Customer, Long> {
    List<Customer> findByEmail(String email);
    List<Customer> findByDate(Date date);
	// custom query example and return a stream
    @Query("select c from Customer c where c.email = :email")
    Stream<Customer> findByEmailReturnStream(@Param("email") String email);
}
```
## 5. Execution
Exécuter l'application qui sera dans la console.

![2](https://user-images.githubusercontent.com/28655112/40983288-2df1ea76-68d7-11e8-80de-b7eef005a401.PNG)
