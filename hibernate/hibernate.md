## Hibernate

### 1. Unidirectional/Bidirectional associations 

- TYPES - @OneToMany, @ManyToOne, @ManyToMany
- NAVIGATION - bidirectional in both sides, so access to associations without explicit queries is possible
- REMOVAL CHILDREN - Unidirectional are NOT efficient when comes to removing child entities. Bidirectional (@OneToMany) is much more efficient because the child entity controls the association 
- JOIN_COLUMN - @JoinColumn - if you want different column
- FETCH TYPES - eager, lazy

### 2. Many to many

`@Entity
public class Store {
    @ManyToMany
    @JoinTable(name = “store_product”,
           joinColumns = { @JoinColumn(name = “fk_store”) },
           inverseJoinColumns = { @JoinColumn(name = “fk_product”) })
    private Set<Product> products = new HashSet<Product>();
}`

### 3. EntityManager -> Session

There is 'unwrap' method

`Session session = em.unwrap(Session.class);`

`SessionFactory sessionFactory = em.getEntityManagerFactory()
.unwrap(SessionFactory.class);`

### 4. @Column

`@Column(name = "created_at", updatable = false) `

`private LocalDateTime createdAt;`

`@Column(name = "updated_at", insertable = false)`

`private LocalDateTime updatedAt;`

- updatable
- insertable
- nullable

### 5. Dates

- java.util.Date and java.util.Calendar are always mapped to SQL TIMESTAMP with nanoseconds
- @Temporal - @Temporal(TemporalType.DATE) private Date date

### 6. Enums

- @Enumerated(EnumType.ORDINAL) and @Enumerated(EnumType.STRING)
- AttributeConverter - to define own mapping

### 7. ID Generation type

- AUTO - base on the types (numerical or UUID)
- IDENTITY - rely on database, auto-incremented, disables batch updates 
- SEQUENCE - switches to table generation id database does not support it. 
- TABLE - uses database table that holds segments of identifier generation values. It selects and updates sequence in order to next insert a record. That is not efficient comparing sequence or identity options.

Almost all modern databases support sequences or auto-incremented columns that generate
primary key values more efficiently than the table strategy described in this tip.