---
topic: "Spring Data JPA — persistenza e database"
parent: "[[BE-NOTES/Java/Spring/Boot/Spring Boot|Spring Boot]]"
nav_next: "[[Repository Pattern.md]]"
---


Modulo Spring per l'accesso ai dati con Jakarta Persistence (JPA). Usato in [[TaskMngr]] per la gestione di utenti, task, team e linked accounts su **PostgreSQL**.

## Argomenti

- [[BE-NOTES/Java/Spring/Data/Entity Mapping|Entity Mapping]] — @Entity, relazioni, indici, named queries
- [[BE-NOTES/Java/Spring/Data/Repository Pattern|Repository Pattern]] — JpaRepository, query derivation, @Query
- [[BE-NOTES/Java/Spring/Data/JPA Auditing|JPA Auditing]] — @CreatedDate, @LastModifiedDate, @EntityListeners
- [[BE-NOTES/Java/Spring/Data/Specifications Dinamiche|Specifications Dinamiche]] — JpaSpecificationExecutor, filtri dinamici
- [[BE-NOTES/Java/Spring/Data/Lock Ottimistico e Pessimistico|Lock Ottimistico e Pessimistico]] — @Version, PESSIMISTIC_WRITE, concorrenza
- [[BE-NOTES/Java/Spring/Data/Hibernate/ORM|ORM]] — Object-Relational Mapping
- [[BE-NOTES/Java/Spring/Data/Hibernate/Hibernate e Session Factory|Hibernate e Session Factory]] — SessionFactory, Session, lifecycle
- [[BE-NOTES/Java/Spring/Data/Hibernate/Relazioni e Mappature|Relazioni e Mappature]] — @OneToMany, @ManyToMany, fetch strategy
- [[BE-NOTES/Java/Spring/Data/Hibernate/Hibernate Annotation|Hibernate Annotation]] — @Table, @Column, @Index
- [[BE-NOTES/Java/Spring/Data/Hibernate/HQL - Hibernate Query Language|HQL]] — query Hibernate native

## In TaskMngr

- Database: PostgreSQL 16
- ORM: Hibernate 6 (Jakarta Persistence)
- Migrazioni: Flyway (disabilitato in dev, ddl-auto=update)
- Lock ottimistico su entità concorrenti (es. task in modifica simultanea)
