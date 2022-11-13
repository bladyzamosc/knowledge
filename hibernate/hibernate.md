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