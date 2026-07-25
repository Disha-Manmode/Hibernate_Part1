# Hibernate ORM Notes

Hibernate is a popular ORM (Object Relational Mapping) framework that implements JPA (Java Persistence API).

- It internally implements JDBC.
- It maps Java objects to database tables.

---

## Hibernate Project Setup

### 1. Set up `pom.xml`

Add the Hibernate dependency:

```xml
<!-- Hibernate ORM -->
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>7.1.1.Final</version>
</dependency>
```

### 2. Add the MySQL Driver JAR to the Project

**If you are using IntelliJ:**

1. Go to **Module Settings → Dependencies**.
2. Click the **+** (plus) icon.
3. Add the downloaded JAR file — make sure it has been extracted from the original archive.

Also add this dependency to `pom.xml`:

```xml
<!-- MySQL Driver -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>
```

### 3. Add `hibernate.cfg.xml`

Project structure:

```
Hibernate_Project
├── src
│   └── main
│       └── java
├── resources
│   └── META-INF
│       └── hibernate.cfg.xml
└── webapp
```

**`hibernate.cfg.xml`:**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
      "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
      "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

  <session-factory>

      <property name="hibernate.connection.driver_class">
          com.mysql.cj.jdbc.Driver
      </property>

      <property name="hibernate.connection.url">
          jdbc:mysql://localhost:3306/Database_Name
      </property>

      <property name="hibernate.connection.username">
          root
      </property>

      <property name="hibernate.connection.password">
          password
      </property>

      <property name="hibernate.dialect">
          org.hibernate.dialect.MySQLDialect
      </property>

      <property name="hibernate.hbm2ddl.auto">
          create/update 
      </property>

      <property name="hibernate.show_sql">
          true
      </property>

      <mapping class="entity_class_path"/>

  </session-factory>

</hibernate-configuration>
```

---

## Hibernate File Structure

### Mapping a Java Class to a Database Table

```java
@Entity
public class Student {}
```

This creates a table named `Student` (the default name, based on the class name).

You can also give the table a specific name:

```java
@Entity
@Table(name = "Student_Table")
public class Student {}
```

This creates a table named `Student_Table`.

> Note: The annotation used to rename a table is `@Table`, not `@Column`. `@Column` is used to customize individual field/column mappings within the entity.

---

## Basic Hibernate Program Structure

```java
public class StudentModel {

    public static void main(String[] args) {

        Configuration config = null;
        SessionFactory sessionFactory = null;
        Session session = null;
        Transaction transaction = null;
        boolean flag = false;

        config = new Configuration();
        config.configure();
        sessionFactory = config.buildSessionFactory();
        session = sessionFactory.openSession();

        try {
            transaction = session.beginTransaction();

            flag = true;
        } catch (HibernateException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (flag) {
                transaction.commit();
            } else {
                transaction.rollback();
            }
            session.close();
            sessionFactory.close();
        }
    }
}
```

**Shorthand equivalent for building the session:**

```java
sessionFactory = new Configuration().configure().buildSessionFactory();
session = sessionFactory.openSession();
```

`session.beginTransaction()` is used for non-`SELECT` operations (insert, update, delete).

---

## Common Session Methods

| Method | Description |
|---|---|
| `session.persist()` | Inserts an entity into the database if it does not already exist. |
| `session.save()` | Inserts an entity into the database. |
| `session.get()` | Immediately retrieves the data (eager loading). |
| `session.update()` | Updates an existing record in the database. |
| `session.merge()` | Updates detached entities by working with a new instance; does not modify the original entity directly. |
| `session.load()` | Returns a proxy object; throws an exception if the object is not found (lazy loading). |
| `session.getReference()` | Similar to `load()` — returns a proxy object and throws an exception if the object is not found. |
| `session.saveOrUpdate()` | Saves a new entity or updates an existing one, based on its identifier. |
| `session.delete()` | Removes a persistent instance from the datastore (legacy API). |
| `session.remove()` | Aligned with the JPA specification; removes the given entity instance from the database. |

---

## Eager Loading vs. Lazy Loading

- **`get()`** — Returns the object immediately (eager loading).
- **`load()`** — Returns a proxy object first; the actual database query is executed only when you access a property of the entity (lazy loading).

---

## L1 Caching in Hibernate

- Provided by Hibernate by default.
- Session-specific.
- For a single session, Hibernate does not hit the database again for the same object — it returns the object from the cache instead.

In simple terms: when you execute a query, it goes to the database and returns an object, which is then stored in cache memory. The next time you request the same object, Hibernate first checks the cache. If the object is present, it is returned from the cache; otherwise, Hibernate queries the database.

L1 caching is achieved using the `get()` method.

---

## L2 Caching in Hibernate

- Shared across sessions.
- Improves performance for multiple requests.
- Helps reduce database hits by storing objects in memory after the first fetch.

### Setting Up L2 Cache

L2 caching, as described here, works with Hibernate 5.

First, add the following dependencies to `pom.xml`, and change the Hibernate version to 5:

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.6.15.Final</version>
</dependency>

<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>3.10.8</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-ehcache -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-ehcache</artifactId>
    <version>5.6.15.Final</version>
</dependency>
<!-- Remove this dependency if it causes issues; it may not be compatible with all versions. -->
```

Add the following configuration to `hibernate.cfg.xml`:

```xml
<property name="hibernate.cache.region.factory_class">
    org.hibernate.cache.ehcache.EhCacheRegionFactory
</property>
<property name="hibernate.cache.use_second_level_cache">true</property>
```

Add annotations to the entity class:

```java
@Entity
@Cacheable // Marks the entity as cacheable, allowing Hibernate to store it in the second-level cache.
@Cache(usage = CacheConcurrencyStrategy.READ_ONLY) // Specifies the cache strategy.
public class Student {}
```

### Cache Concurrency Strategies

Defines how the cache behaves:

- **`READ_ONLY`** — Used for entities that never change.
- **`READ_WRITE`** — Used for entities that may be modified by the application.
