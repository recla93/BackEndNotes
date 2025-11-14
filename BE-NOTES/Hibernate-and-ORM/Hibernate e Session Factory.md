### Cos'è Hibernate?

**Hibernate** è un **framework ORM** per Java che semplifica l'interazione con i database relazionali. È l'implementazione più popolare di JPA.

### SessionFactory

La **SessionFactory** è una **Connection Factory avanzata** che:
- Gestisce le connessioni con il DB
- Crea sessioni Hibernate
- Configura il mapping oggetto-relazionale

```java
// Creazione della SessionFactory (da fare una sola volta)
SessionFactory sessionFactory = new Configuration()
    .configure("hibernate.cfg.xml")
    .addAnnotatedClass(Persona.class)
    .buildSessionFactory();

// Creazione di Session (da aprire e chiudere per ogni operazione)
Session session = sessionFactory.openSession();
try {
    // Operazioni sul DB
} finally {
    session.close();
}
```

### File di Configurazione (hibernate.cfg.xml)

```xml
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Proprietà JDBC -->
        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>
        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/mio_database
        </property>
        <property name="hibernate.connection.username">utente</property>
        <property name="hibernate.connection.password">password</property>
        
        <!-- Proprietà di Hibernate -->
        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQL8Dialect
        </property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="hibernate.show_sql">true</property>
        
        <!-- Classi Entity -->
        <mapping class="com.example.Persona"/>
        <mapping class="com.example.Studente"/>
    </session-factory>
</hibernate-configuration>
```

### Operazioni CRUD con Hibernate

```java
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

try {
    // CREATE
    Persona p = new Persona();
    p.setNome("Mario");
    p.setEta(30);
    session.save(p);
    
    // READ
    Persona mario = session.get(Persona.class, 1);
    
    // UPDATE
    mario.setEta(31);
    session.update(mario);
    
    // DELETE
    session.delete(mario);
    
    tx.commit();
} catch (Exception e) {
    if (tx != null) tx.rollback();
    e.printStackTrace();
} finally {
    session.close();
}
```