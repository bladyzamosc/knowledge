## Various

### 1. CGLIB vs Java Dynamic proxy

- CGLIB (+ javassist) - can create a proxy by subclassing. Class in this case cannot be final.

- Java dynamic proxy - proxy only interfaces

### 2. Mockito and tests

- @Mock
- @InjectMocks - automatically injects mock object into the object being testing. For example given service need another service. This injects mock into service.
- @ExtendWith - @ExtendWith(MockitoExtension.class), @ExtendWith(SpringExtension.class) - JUnit does not evaluate @mock annotation by default. This lets to do it.

<code>
@ExtendWith(MockitoExtension.class)
class RecommenderImplementationMockTest{

	@InjectMocks
	private RecommenderImplementation recommenderImpl;
	
	@Mock
	private Filter mockFilter;
	
	@Test
	void testRecommendMovies_noRecommendationsFound() {
		when(mockFilter.getRecommendations("Finding Dory")).thenReturn(new String[] {});
		assertArrayEquals(new String[] {}, recommenderImpl.recommendMovies("Finding Dory"));
	}}
</code>

- @SpringBootTest - for test lauching context 

### 3. try with resources

java.lang.AutoCloseable - must be implemented

### 4. java.util.Arrays

- asList
- search - binarySearch, 
- compare, compareUnassigned, 
- copyOf, copyOfRange
- deepEquals
- deepHashCode
- fill
- mismatch
- parallelSort, sort

### 5. java.util.Collections 

- asLifoQueue
- binarySearch
- copy
- emptyXXX
- fill
- list 
- max, min
- reverse
- rotate
- singleton, singletonList
- swap
- synchronizedXXX, unmodifiableXXX

### 6. java.util.Objects

- isNull, isNotNull
