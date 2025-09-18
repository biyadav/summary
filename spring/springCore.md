# Spring Core Advanced Study Guide
## For 12+ Years Java Developers

## Table of Contents
1. [Advanced IoC Container Deep Dive](#advanced-ioc-container)
2. [Bean Lifecycle and Post-Processing](#bean-lifecycle)
3. [Advanced Dependency Injection Patterns](#advanced-di)
4. [ApplicationContext vs BeanFactory](#context-vs-factory)
5. [Spring Expression Language (SpEL) Advanced](#spel-advanced)
6. [Event-Driven Programming](#event-driven)
7. [Resource Management and Environment](#resource-management)
8. [Conditional Bean Creation](#conditional-beans)
9. [Custom Scopes and Proxy Mechanisms](#custom-scopes)
10. [Integration and Extensibility](#integration)
11. [Performance Optimization](#performance)
12. [Tricky Interview Scenarios](#interview-scenarios)

---

## Advanced IoC Container Deep Dive

### Container Hierarchy and Parent-Child Contexts

**Advanced Concept: Multiple ApplicationContext Hierarchy**

```java
/**
 * Advanced ApplicationContext Hierarchy
 * Understanding parent-child relationships and bean resolution
 */
@Configuration
public class ParentContextConfiguration {
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create()
            .driverClassName("org.h2.Driver")
            .url("jdbc:h2:mem:parent")
            .build();
    }
    
    @Bean("sharedService")
    public SharedService parentSharedService() {
        return new SharedService("Parent Implementation");
    }
}

@Configuration  
public class ChildContextConfiguration {
    
    @Bean("sharedService")
    public SharedService childSharedService() {
        // This will override parent's sharedService in child context
        return new SharedService("Child Implementation");
    }
    
    @Bean
    public SpecializedService specializedService(@Qualifier("sharedService") SharedService service) {
        // Will inject child's sharedService, not parent's
        return new SpecializedService(service);
    }
}

/**
 * Programmatic Context Hierarchy Creation
 */
public class ContextHierarchyDemo {
    
    public void demonstrateHierarchy() {
        // Create parent context
        AnnotationConfigApplicationContext parentContext = 
            new AnnotationConfigApplicationContext(ParentContextConfiguration.class);
        
        // Create child context with parent
        AnnotationConfigApplicationContext childContext = 
            new AnnotationConfigApplicationContext();
        childContext.setParent(parentContext);
        childContext.register(ChildContextConfiguration.class);
        childContext.refresh();
        
        // Bean resolution rules:
        // 1. Child context is searched first
        // 2. If not found, parent context is searched
        // 3. Parent beans are inherited by child
        
        SharedService fromChild = childContext.getBean("sharedService", SharedService.class);
        // Returns "Child Implementation" - child overrides parent
        
        DataSource fromParent = childContext.getBean(DataSource.class);
        // Returns parent's DataSource - inherited from parent
        
        // Demonstrate circular dependency prevention
        try {
            parentContext.getBean(SpecializedService.class);
            // Will fail - SpecializedService only exists in child context
        } catch (NoSuchBeanDefinitionException e) {
            System.out.println("Parent cannot access child beans");
        }
    }
}
```

### Advanced Bean Definition and Registration

**Dynamic Bean Registration and Conditional Loading**

```java
/**
 * Advanced Bean Definition Registration
 * Programmatic bean registration with conditions
 */
@Configuration
public class DynamicBeanRegistration implements BeanDefinitionRegistryPostProcessor {
    
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) 
            throws BeansException {
        
        // Conditionally register beans based on environment
        Environment env = ((ConfigurableApplicationContext) registry).getEnvironment();
        
        if (env.acceptsProfiles(Profiles.of("production"))) {
            // Register production-specific beans
            registerProductionBeans(registry);
        } else if (env.acceptsProfiles(Profiles.of("development"))) {
            // Register development-specific beans
            registerDevelopmentBeans(registry);
        }
        
        // Register beans based on classpath availability
        if (ClassUtils.isPresent("redis.clients.jedis.Jedis", null)) {
            registerRedisConfiguration(registry);
        }
    }
    
    private void registerProductionBeans(BeanDefinitionRegistry registry) {
        // Create BeanDefinition programmatically
        BeanDefinitionBuilder builder = BeanDefinitionBuilder
            .genericBeanDefinition(ProductionCacheManager.class)
            .addConstructorArgValue("production-cache")
            .addPropertyValue("maxSize", 10000)
            .addPropertyValue("ttl", Duration.ofHours(24))
            .setScope(BeanDefinition.SCOPE_SINGLETON)
            .setLazyInit(false);
        
        registry.registerBeanDefinition("cacheManager", builder.getBeanDefinition());
    }
    
    private void registerDevelopmentBeans(BeanDefinitionRegistry registry) {
        GenericBeanDefinition beanDef = new GenericBeanDefinition();
        beanDef.setBeanClass(DevelopmentCacheManager.class);
        beanDef.setScope(BeanDefinition.SCOPE_SINGLETON);
        beanDef.getPropertyValues().add("enabled", false);
        
        registry.registerBeanDefinition("cacheManager", beanDef);
    }
    
    private void registerRedisConfiguration(BeanDefinitionRegistry registry) {
        // Advanced: Register factory bean for complex initialization
        BeanDefinitionBuilder factoryBuilder = BeanDefinitionBuilder
            .genericBeanDefinition(RedisConnectionFactoryBean.class)
            .addConstructorArgValue("${redis.host:localhost}")
            .addConstructorArgValue("${redis.port:6379}")
            .setFactoryMethod("createConnectionFactory");
        
        registry.registerBeanDefinition("redisConnectionFactory", 
                                      factoryBuilder.getBeanDefinition());
    }
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 
            throws BeansException {
        // Modify bean definitions after registration but before instantiation
        
        String[] beanNames = beanFactory.getBeanDefinitionNames();
        for (String beanName : beanNames) {
            BeanDefinition beanDef = beanFactory.getBeanDefinition(beanName);
            
            // Add common post-processor to all service beans
            if (beanName.endsWith("Service")) {
                beanDef.setAttribute("requiresAuditing", true);
            }
        }
    }
}
```

### Advanced Circular Dependency Resolution

**Understanding and Handling Complex Circular Dependencies**

```java
/**
 * Advanced Circular Dependency Scenarios
 * Demonstration of different circular dependency patterns and resolutions
 */

// Scenario 1: Constructor Circular Dependency (Cannot be resolved)
@Component
public class ServiceA {
    private final ServiceB serviceB;
    
    // This creates unresolvable circular dependency
    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

@Component  
public class ServiceB {
    private final ServiceA serviceA;
    
    // Circular dependency with ServiceA
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }
}

// Scenario 2: Setter Circular Dependency (Resolvable)
@Component
public class ServiceC {
    private ServiceD serviceD;
    
    @Autowired
    public void setServiceD(ServiceD serviceD) {
        this.serviceD = serviceD;
    }
    
    @PostConstruct
    public void init() {
        System.out.println("ServiceC initialized with ServiceD: " + 
                          (serviceD != null));
    }
}

@Component
public class ServiceD {
    private ServiceC serviceC;
    
    @Autowired
    public void setServiceC(ServiceC serviceC) {
        this.serviceC = serviceC;
    }
}

// Scenario 3: Mixed Constructor/Setter Circular Dependency
@Component
public class ServiceE {
    private final ServiceF serviceF;
    private ServiceG serviceG;
    
    public ServiceE(ServiceF serviceF) {
        this.serviceF = serviceF;
    }
    
    @Autowired
    public void setServiceG(ServiceG serviceG) {
        this.serviceG = serviceG;
    }
}

@Component
public class ServiceF {
    private ServiceE serviceE;
    
    @Autowired
    public void setServiceE(ServiceE serviceE) {
        this.serviceE = serviceE;
    }
}

@Component
public class ServiceG {
    private final ServiceE serviceE;
    
    public ServiceG(ServiceE serviceE) {
        this.serviceE = serviceE;
    }
}

/**
 * Advanced Resolution Strategies
 */
@Configuration
public class CircularDependencyResolution {
    
    // Strategy 1: Use @Lazy annotation
    @Component
    public static class LazyResolvedServiceA {
        private final LazyResolvedServiceB serviceB;
        
        public LazyResolvedServiceA(@Lazy LazyResolvedServiceB serviceB) {
            this.serviceB = serviceB;
        }
    }
    
    @Component
    public static class LazyResolvedServiceB {
        private final LazyResolvedServiceA serviceA;
        
        public LazyResolvedServiceB(LazyResolvedServiceA serviceA) {
            this.serviceA = serviceA;
        }
    }
    
    // Strategy 2: Use ApplicationContextAware
    @Component
    public static class ContextAwareService implements ApplicationContextAware {
        private ApplicationContext applicationContext;
        private DependentService dependentService;
        
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) 
                throws BeansException {
            this.applicationContext = applicationContext;
        }
        
        @PostConstruct
        public void init() {
            // Lazy lookup to avoid circular dependency
            this.dependentService = applicationContext.getBean(DependentService.class);
        }
    }
    
    @Component
    public static class DependentService {
        private final ContextAwareService contextAwareService;
        
        public DependentService(ContextAwareService contextAwareService) {
            this.contextAwareService = contextAwareService;
        }
    }
    
    // Strategy 3: Use ObjectProvider for optional dependencies
    @Component
    public static class ProviderBasedService {
        private final ObjectProvider<CyclicDependency> cyclicDependencyProvider;
        
        public ProviderBasedService(ObjectProvider<CyclicDependency> provider) {
            this.cyclicDependencyProvider = provider;
        }
        
        public void doWork() {
            CyclicDependency dependency = cyclicDependencyProvider.getIfAvailable();
            if (dependency != null) {
                dependency.process();
            }
        }
    }
    
    @Component
    public static class CyclicDependency {
        private final ProviderBasedService providerBasedService;
        
        public CyclicDependency(ProviderBasedService service) {
            this.providerBasedService = service;
        }
        
        public void process() {
            System.out.println("Processing with provider-based service");
        }
    }
}
```

---

## Bean Lifecycle and Post-Processing

### Advanced Bean Post-Processing

**Custom BeanPostProcessor for Cross-Cutting Concerns**

```java
/**
 * Advanced Bean Post-Processing
 * Implementing cross-cutting concerns and bean enhancement
 */
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class AuditingBeanPostProcessor implements BeanPostProcessor, PriorityOrdered {
    
    private final AuditService auditService;
    private final Set<String> auditedBeans = new ConcurrentHashMap<>().keySet(ConcurrentHashMap.newKeySet());
    
    public AuditingBeanPostProcessor(AuditService auditService) {
        this.auditService = auditService;
    }
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
            throws BeansException {
        
        // Check if bean should be audited
        if (requiresAuditing(bean, beanName)) {
            auditedBeans.add(beanName);
            auditService.logBeanCreation(beanName, bean.getClass());
        }
        
        // Inject custom properties
        if (bean instanceof ConfigurableService) {
            ((ConfigurableService) bean).setCreationTime(Instant.now());
        }
        
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        
        // Create proxy for audited beans
        if (auditedBeans.contains(beanName)) {
            return createAuditProxy(bean, beanName);
        }
        
        // Validate bean configuration
        validateBeanConfiguration(bean, beanName);
        
        return bean;
    }
    
    private Object createAuditProxy(Object target, String beanName) {
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new AuditingMethodInterceptor(auditService, beanName));
        proxyFactory.setProxyTargetClass(true); // Use CGLIB for classes
        return proxyFactory.getProxy();
    }
    
    private boolean requiresAuditing(Object bean, String beanName) {
        return bean.getClass().isAnnotationPresent(Audited.class) ||
               beanName.endsWith("Service") ||
               beanName.endsWith("Repository");
    }
    
    private void validateBeanConfiguration(Object bean, String beanName) {
        if (bean instanceof Validatable) {
            Validatable validatable = (Validatable) bean;
            if (!validatable.isValid()) {
                throw new BeanCreationException(beanName, 
                    "Bean failed validation: " + validatable.getValidationErrors());
            }
        }
    }
    
    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE;
    }
}

/**
 * Advanced Method Interceptor for Auditing
 */
public class AuditingMethodInterceptor implements MethodInterceptor {
    
    private final AuditService auditService;
    private final String beanName;
    private final Set<String> auditedMethods;
    
    public AuditingMethodInterceptor(AuditService auditService, String beanName) {
        this.auditService = auditService;
        this.beanName = beanName;
        this.auditedMethods = determineAuditedMethods(beanName);
    }
    
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Method method = invocation.getMethod();
        String methodName = method.getName();
        
        if (!auditedMethods.contains(methodName)) {
            return invocation.proceed();
        }
        
        String auditId = UUID.randomUUID().toString();
        Instant startTime = Instant.now();
        
        try {
            auditService.logMethodEntry(auditId, beanName, methodName, 
                                      invocation.getArguments());
            
            Object result = invocation.proceed();
            
            auditService.logMethodSuccess(auditId, beanName, methodName, 
                                        result, Duration.between(startTime, Instant.now()));
            return result;
            
        } catch (Throwable throwable) {
            auditService.logMethodFailure(auditId, beanName, methodName, 
                                        throwable, Duration.between(startTime, Instant.now()));
            throw throwable;
        }
    }
    
    private Set<String> determineAuditedMethods(String beanName) {
        // Logic to determine which methods should be audited
        return Set.of("create", "update", "delete", "process");
    }
}

/**
 * Custom Annotations for Bean Enhancement
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Audited {
    String[] methods() default {};
    boolean includeArguments() default true;
    boolean includeResult() default false;
}

public interface ConfigurableService {
    void setCreationTime(Instant creationTime);
    Instant getCreationTime();
}

public interface Validatable {
    boolean isValid();
    List<String> getValidationErrors();
}
```

### Advanced Lifecycle Management

**Custom Lifecycle Beans and Graceful Shutdown**

```java
/**
 * Advanced Lifecycle Management
 * Custom lifecycle beans with dependency-aware shutdown
 */
@Component
public class DatabaseConnectionPool implements Lifecycle, DisposableBean, SmartLifecycle {
    
    private volatile boolean running = false;
    private final List<Connection> connections = new CopyOnWriteArrayList<>();
    private final ScheduledExecutorService maintenanceExecutor;
    private final AtomicInteger activeConnections = new AtomicInteger(0);
    
    public DatabaseConnectionPool() {
        this.maintenanceExecutor = Executors.newScheduledThreadPool(1, 
            r -> new Thread(r, "db-pool-maintenance"));
    }
    
    @Override
    public void start() {
        if (!running) {
            initializeConnections();
            startMaintenanceTasks();
            running = true;
            System.out.println("Database connection pool started");
        }
    }
    
    @Override
    public void stop() {
        if (running) {
            gracefulShutdown();
            running = false;
            System.out.println("Database connection pool stopped");
        }
    }
    
    @Override
    public boolean isRunning() {
        return running;
    }
    
    @Override
    public int getPhase() {
        // Higher phase means starts later and stops earlier
        return 100;
    }
    
    @Override
    public boolean isAutoStartup() {
        return true;
    }
    
    @Override
    public void stop(Runnable callback) {
        // Asynchronous shutdown with callback
        CompletableFuture.runAsync(() -> {
            try {
                gracefulShutdown();
                running = false;
            } finally {
                callback.run();
            }
        });
    }
    
    private void initializeConnections() {
        // Initialize connection pool
        for (int i = 0; i < 10; i++) {
            // Create mock connections
            connections.add(createConnection());
        }
    }
    
    private void startMaintenanceTasks() {
        // Schedule periodic maintenance
        maintenanceExecutor.scheduleAtFixedRate(
            this::performMaintenance, 30, 30, TimeUnit.SECONDS);
    }
    
    private void gracefulShutdown() {
        System.out.println("Starting graceful shutdown...");
        
        // Wait for active connections to finish
        long shutdownStart = System.currentTimeMillis();
        while (activeConnections.get() > 0 && 
               (System.currentTimeMillis() - shutdownStart) < 30000) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
        
        // Force close remaining connections
        connections.forEach(this::closeConnection);
        connections.clear();
        
        // Shutdown maintenance executor
        maintenanceExecutor.shutdown();
        try {
            if (!maintenanceExecutor.awaitTermination(5, TimeUnit.SECONDS)) {
                maintenanceExecutor.shutdownNow();
            }
        } catch (InterruptedException e) {
            maintenanceExecutor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
    
    private void performMaintenance() {
        // Remove stale connections and add fresh ones
        connections.removeIf(this::isStale);
        while (connections.size() < 10) {
            connections.add(createConnection());
        }
    }
    
    @Override
    public void destroy() throws Exception {
        if (running) {
            stop();
        }
    }
    
    // Mock methods
    private Connection createConnection() { return null; }
    private void closeConnection(Connection conn) { }
    private boolean isStale(Connection conn) { return false; }
}

/**
 * Dependency-Aware Lifecycle Management
 */
@Component
public class ApplicationServiceManager implements SmartLifecycle {
    
    private final DatabaseConnectionPool connectionPool;
    private final CacheManager cacheManager;
    private final MessageBroker messageBroker;
    private volatile boolean running = false;
    
    public ApplicationServiceManager(DatabaseConnectionPool connectionPool,
                                   CacheManager cacheManager,
                                   MessageBroker messageBroker) {
        this.connectionPool = connectionPool;
        this.cacheManager = cacheManager;
        this.messageBroker = messageBroker;
    }
    
    @Override
    public void start() {
        // Ensure dependencies are started first
        if (!connectionPool.isRunning()) {
            connectionPool.start();
        }
        
        // Start application services in order
        cacheManager.start();
        messageBroker.start();
        
        running = true;
        System.out.println("Application services started");
    }
    
    @Override
    public void stop() {
        // Stop in reverse order
        messageBroker.stop();
        cacheManager.stop();
        
        running = false;
        System.out.println("Application services stopped");
    }
    
    @Override
    public boolean isRunning() {
        return running;
    }
    
    @Override
    public int getPhase() {
        // Start after database pool (phase 100) but before web layer
        return 200;
    }
    
    @Override
    public boolean isAutoStartup() {
        return true;
    }
}

/**
 * Graceful Shutdown Hook
 */
@Component
public class GracefulShutdownHook {
    
    @EventListener
    public void handleContextClosed(ContextClosedEvent event) {
        System.out.println("Application context is closing...");
        
        // Perform cleanup operations
        cleanupResources();
        
        // Notify external systems
        notifyExternalSystems();
        
        System.out.println("Graceful shutdown completed");
    }
    
    @PreDestroy
    public void onDestroy() {
        System.out.println("PreDestroy called on GracefulShutdownHook");
    }
    
    private void cleanupResources() {
        // Cleanup temporary files, flush caches, etc.
    }
    
    private void notifyExternalSystems() {
        // Notify load balancers, monitoring systems, etc.
    }
}
```

---

## Advanced Dependency Injection Patterns

### Custom Qualifiers and Advanced Injection

**Complex Qualifier Scenarios and Custom Injection Logic**

```java
/**
 * Advanced Qualifier Patterns
 * Custom qualifiers with complex matching logic
 */

// Custom qualifier annotations
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface DataSourceType {
    DatabaseType value();
    String region() default "us-east-1";
    boolean readOnly() default false;
}

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface CacheType {
    CacheProvider provider();
    String[] regions() default {};
    int maxSize() default 1000;
}

public enum DatabaseType {
    POSTGRESQL, MYSQL, ORACLE, H2
}

public enum CacheProvider {
    REDIS, HAZELCAST, EHCACHE, CAFFEINE
}

/**
 * Advanced Configuration with Multiple Qualifiers
 */
@Configuration
public class AdvancedDataSourceConfiguration {
    
    @Bean
    @DataSourceType(value = DatabaseType.POSTGRESQL, region = "us-east-1", readOnly = false)
    public DataSource primaryPostgresDataSource() {
        return DataSourceBuilder.create()
            .driverClassName("org.postgresql.Driver")
            .url("jdbc:postgresql://primary-db:5432/app")
            .username("app_user")
            .password("secret")
            .build();
    }
    
    @Bean
    @DataSourceType(value = DatabaseType.POSTGRESQL, region = "us-east-1", readOnly = true)
    public DataSource readOnlyPostgresDataSource() {
        return DataSourceBuilder.create()
            .driverClassName("org.postgresql.Driver")
            .url("jdbc:postgresql://readonly-db:5432/app")
            .username("readonly_user")
            .password("secret")
            .build();
    }
    
    @Bean
    @DataSourceType(value = DatabaseType.MYSQL, region = "eu-west-1")
    public DataSource mysqlDataSource() {
        return DataSourceBuilder.create()
            .driverClassName("com.mysql.cj.jdbc.Driver")
            .url("jdbc:mysql://mysql-db:3306/app")
            .build();
    }
    
    @Bean
    @CacheType(provider = CacheProvider.REDIS, regions = {"user", "session"}, maxSize = 10000)
    public CacheManager redisCacheManager() {
        return new RedisCacheManager.Builder(jedisConnectionFactory())
            .cacheDefaults(cacheConfiguration())
            .build();
    }
    
    @Bean
    @CacheType(provider = CacheProvider.CAFFEINE, regions = {"metadata"}, maxSize = 1000)
    public CacheManager caffeineCacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES));
        return cacheManager;
    }
    
    private JedisConnectionFactory jedisConnectionFactory() {
        return new JedisConnectionFactory();
    }
    
    private RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
    }
}

/**
 * Advanced Injection with Custom Resolution Logic
 */
@Service
public class DataAccessService {
    
    private final DataSource primaryDataSource;
    private final DataSource readOnlyDataSource;
    private final CacheManager userCacheManager;
    private final CacheManager metadataCacheManager;
    
    // Complex constructor injection with multiple qualifiers
    public DataAccessService(
            @DataSourceType(value = DatabaseType.POSTGRESQL, readOnly = false) 
            DataSource primaryDataSource,
            
            @DataSourceType(value = DatabaseType.POSTGRESQL, readOnly = true) 
            DataSource readOnlyDataSource,
            
            @CacheType(provider = CacheProvider.REDIS) 
            CacheManager userCacheManager,
            
            @CacheType(provider = CacheProvider.CAFFEINE) 
            CacheManager metadataCacheManager) {
        
        this.primaryDataSource = primaryDataSource;
        this.readOnlyDataSource = readOnlyDataSource;
        this.userCacheManager = userCacheManager;
        this.metadataCacheManager = metadataCacheManager;
    }
    
    public void performOperation(String operation, boolean readOnly) {
        DataSource dataSource = readOnly ? readOnlyDataSource : primaryDataSource;
        CacheManager cacheManager = operation.startsWith("user") ? 
            userCacheManager : metadataCacheManager;
        
        // Use appropriate data source and cache manager
        processWithDataSourceAndCache(dataSource, cacheManager, operation);
    }
    
    private void processWithDataSourceAndCache(DataSource dataSource, 
                                             CacheManager cacheManager, 
                                             String operation) {
        // Implementation
    }
}

/**
 * Custom BeanPostProcessor for Qualifier-Based Injection
 */
@Component
public class CustomQualifierInjectionPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
            throws BeansException {
        
        Class<?> beanClass = bean.getClass();
        Field[] fields = beanClass.getDeclaredFields();
        
        for (Field field : fields) {
            if (field.isAnnotationPresent(CustomInject.class)) {
                CustomInject annotation = field.getAnnotation(CustomInject.class);
                Object dependency = resolveDependency(annotation, field.getType());
                
                if (dependency != null) {
                    field.setAccessible(true);
                    try {
                        field.set(bean, dependency);
                    } catch (IllegalAccessException e) {
                        throw new BeanCreationException(beanName, 
                            "Failed to inject custom dependency", e);
                    }
                }
            }
        }
        
        return bean;
    }
    
    private Object resolveDependency(CustomInject annotation, Class<?> fieldType) {
        // Custom dependency resolution logic
        String strategy = annotation.strategy();
        boolean required = annotation.required();
        
        switch (strategy) {
            case "primary":
                return findPrimaryBean(fieldType);
            case "fallback":
                return findFallbackBean(fieldType);
            case "conditional":
                return findConditionalBean(fieldType, annotation.condition());
            default:
                if (required) {
                    throw new NoSuchBeanDefinitionException(fieldType);
                }
                return null;
        }
    }
    
    private Object findPrimaryBean(Class<?> type) {
        // Implementation to find primary bean
        return null;
    }
    
    private Object findFallbackBean(Class<?> type) {
        // Implementation to find fallback bean
        return null;
    }
    
    private Object findConditionalBean(Class<?> type, String condition) {
        // Implementation to find bean based on condition
        return null;
    }
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomInject {
    String strategy() default "primary";
    boolean required() default true;
    String condition() default "";
}
```

### Advanced Factory Bean Patterns

**Complex Factory Beans with Dynamic Creation Logic**

```java
/**
 * Advanced Factory Bean Implementation
 * Dynamic bean creation with complex initialization logic
 */
@Component
public class DatabaseConnectionFactoryBean implements FactoryBean<Connection>, 
                                                    InitializingBean, 
                                                    DisposableBean {
    
    private String databaseUrl;
    private String username;
    private String password;
    private String driverClassName;
    private boolean testOnBorrow = true;
    private int maxRetries = 3;
    private Duration retryDelay = Duration.ofSeconds(5);
    
    private Connection connection;
    private final AtomicBoolean initialized = new AtomicBoolean(false);
    
    @Override
    public Connection getObject() throws Exception {
        if (!initialized.get()) {
            throw new IllegalStateException("Factory bean not properly initialized");
        }
        
        if (connection == null || !isConnectionValid(connection)) {
            connection = createConnectionWithRetry();
        }
        
        return connection;
    }
    
    @Override
    public Class<?> getObjectType() {
        return Connection.class;
    }
    
    @Override
    public boolean isSingleton() {
        return true;
    }
    
    @Override
    public void afterPropertiesSet() throws Exception {
        validateConfiguration();
        preloadDriver();
        initialized.set(true);
    }
    
    @Override
    public void destroy() throws Exception {
        if (connection != null && !connection.isClosed()) {
            connection.close();
        }
    }
    
    private Connection createConnectionWithRetry() throws Exception {
        Exception lastException = null;
        
        for (int attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                Connection conn = DriverManager.getConnection(
                    databaseUrl, username, password);
                
                if (testOnBorrow && !isConnectionValid(conn)) {
                    conn.close();
                    throw new SQLException("Connection failed validation test");
                }
                
                return conn;
                
            } catch (Exception e) {
                lastException = e;
                
                if (attempt < maxRetries) {
                    System.err.println("Connection attempt " + attempt + 
                                     " failed, retrying in " + retryDelay.toSeconds() + "s");
                    Thread.sleep(retryDelay.toMillis());
                } else {
                    System.err.println("All connection attempts failed");
                }
            }
        }
        
        throw new SQLException("Failed to create connection after " + 
                              maxRetries + " attempts", lastException);
    }
    
    private void validateConfiguration() {
        if (databaseUrl == null || databaseUrl.trim().isEmpty()) {
            throw new IllegalArgumentException("Database URL is required");
        }
        if (driverClassName == null || driverClassName.trim().isEmpty()) {
            throw new IllegalArgumentException("Driver class name is required");
        }
    }
    
    private void preloadDriver() throws ClassNotFoundException {
        Class.forName(driverClassName);
    }
    
    private boolean isConnectionValid(Connection conn) {
        try {
            return conn != null && !conn.isClosed() && conn.isValid(5);
        } catch (SQLException e) {
            return false;
        }
    }
    
    // Setters for dependency injection
    @Value("${database.url}")
    public void setDatabaseUrl(String databaseUrl) {
        this.databaseUrl = databaseUrl;
    }
    
    @Value("${database.username}")
    public void setUsername(String username) {
        this.username = username;
    }
    
    @Value("${database.password}")
    public void setPassword(String password) {
        this.password = password;
    }
    
    @Value("${database.driver}")
    public void setDriverClassName(String driverClassName) {
        this.driverClassName = driverClassName;
    }
    
    @Value("${database.test-on-borrow:true}")
    public void setTestOnBorrow(boolean testOnBorrow) {
        this.testOnBorrow = testOnBorrow;
    }
}

/**
 * Abstract Factory Bean for Complex Object Creation
 */
public abstract class AbstractServiceFactoryBean<T> implements FactoryBean<T>, 
                                                               ApplicationContextAware {
    
    protected ApplicationContext applicationContext;
    protected final Map<String, Object> configuration = new HashMap<>();
    protected volatile T cachedInstance;
    protected final Object creationLock = new Object();
    
    @Override
    public final T getObject() throws Exception {
        if (isSingleton() && cachedInstance != null) {
            return cachedInstance;
        }
        
        synchronized (creationLock) {
            if (isSingleton() && cachedInstance != null) {
                return cachedInstance;
            }
            
            T instance = createInstance();
            configureInstance(instance);
            
            if (isSingleton()) {
                cachedInstance = instance;
            }
            
            return instance;
        }
    }
    
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) 
            throws BeansException {
        this.applicationContext = applicationContext;
    }
    
    protected abstract T createInstance() throws Exception;
    
    protected void configureInstance(T instance) throws Exception {
        // Default implementation - can be overridden
        if (instance instanceof ApplicationContextAware) {
            ((ApplicationContextAware) instance).setApplicationContext(applicationContext);
        }
        
        if (instance instanceof InitializingBean) {
            ((InitializingBean) instance).afterPropertiesSet();
        }
    }
    
    protected final void addConfiguration(String key, Object value) {
        configuration.put(key, value);
    }
    
    protected final <V> V getConfiguration(String key, Class<V> type) {
        Object value = configuration.get(key);
        return type.cast(value);
    }
    
    @Override
    public boolean isSingleton() {
        return true; // Default to singleton
    }
}

/**
 * Concrete Implementation for Service Creation
 */
@Component
public class EmailServiceFactoryBean extends AbstractServiceFactoryBean<EmailService> {
    
    @Value("${email.provider:smtp}")
    private String emailProvider;
    
    @Value("${email.smtp.host:localhost}")
    private String smtpHost;
    
    @Value("${email.smtp.port:587}")
    private int smtpPort;
    
    @Override
    protected EmailService createInstance() throws Exception {
        switch (emailProvider.toLowerCase()) {
            case "smtp":
                return createSmtpEmailService();
            case "ses":
                return createSesEmailService();
            case "sendgrid":
                return createSendGridEmailService();
            case "mock":
                return createMockEmailService();
            default:
                throw new IllegalArgumentException(
                    "Unknown email provider: " + emailProvider);
        }
    }
    
    @Override
    public Class<?> getObjectType() {
        return EmailService.class;
    }
    
    private EmailService createSmtpEmailService() {
        SmtpEmailService service = new SmtpEmailService();
        service.setHost(smtpHost);
        service.setPort(smtpPort);
        service.setUsername(getConfiguration("username", String.class));
        service.setPassword(getConfiguration("password", String.class));
        return service;
    }
    
    private EmailService createSesEmailService() {
        // Create AWS SES email service
        return new SesEmailService();
    }
    
    private EmailService createSendGridEmailService() {
        // Create SendGrid email service
        return new SendGridEmailService();
    }
    
    private EmailService createMockEmailService() {
        return new MockEmailService();
    }
    
    @PostConstruct
    public void initialize() {
        // Load configuration from application context
        Environment env = applicationContext.getEnvironment();
        
        if (env.containsProperty("email.username")) {
            addConfiguration("username", env.getProperty("email.username"));
        }
        if (env.containsProperty("email.password")) {
            addConfiguration("password", env.getProperty("email.password"));
        }
    }
}

// Mock interfaces and classes for the example
interface EmailService {
    void sendEmail(String to, String subject, String body);
}

class SmtpEmailService implements EmailService {
    private String host;
    private int port;
    private String username;
    private String password;
    
    @Override
    public void sendEmail(String to, String subject, String body) {
        // SMTP implementation
    }
    
    // Setters
    public void setHost(String host) { this.host = host; }
    public void setPort(int port) { this.port = port; }
    public void setUsername(String username) { this.username = username; }
    public void setPassword(String password) { this.password = password; }
}

class SesEmailService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        // AWS SES implementation
    }
}

class SendGridEmailService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        // SendGrid implementation
    }
}

class MockEmailService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("Mock email sent to: " + to);
    }
}
```

---

## Event-Driven Programming

### Advanced Event Handling and Custom Event Publishers

**Asynchronous Event Processing and Event Ordering**

```java
/**
 * Advanced Event-Driven Architecture in Spring
 * Custom events, async processing, and event ordering
 */
@Configuration
@EnableAsync
public class EventDrivenConfiguration {
    
    @Bean
    public TaskExecutor eventTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("event-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}

/**
 * Complex Event Hierarchy
 */
public abstract class BaseApplicationEvent extends ApplicationEvent {
    private final String eventId;
    private final Instant timestamp;
    private final Map<String, Object> metadata;
    
    protected BaseApplicationEvent(Object source) {
        super(source);
        this.eventId = UUID.randomUUID().toString();
        this.timestamp = Instant.now();
        this.metadata = new ConcurrentHashMap<>();
    }
    
    public String getEventId() { return eventId; }
    public Instant getTimestamp() { return timestamp; }
    public Map<String, Object> getMetadata() { return metadata; }
    
    public void addMetadata(String key, Object value) {
        metadata.put(key, value);
    }
}

public class UserRegisteredEvent extends BaseApplicationEvent {
    private final String userId;
    private final String email;
    private final UserType userType;
    
    public UserRegisteredEvent(Object source, String userId, String email, UserType userType) {
        super(source);
        this.userId = userId;
        this.email = email;
        this.userType = userType;
        
        addMetadata("registrationSource", source.getClass().getSimpleName());
        addMetadata("userType", userType.name());
    }
    
    // Getters
    public String getUserId() { return userId; }
    public String getEmail() { return email; }
    public UserType getUserType() { return userType; }
}

public class OrderProcessedEvent extends BaseApplicationEvent {
    private final String orderId;
    private final BigDecimal amount;
    private final OrderStatus status;
    
    public OrderProcessedEvent(Object source, String orderId, BigDecimal amount, OrderStatus status) {
        super(source);
        this.orderId = orderId;
        this.amount = amount;
        this.status = status;
        
        addMetadata("processingTime", System.currentTimeMillis());
        addMetadata("orderValue", amount.toString());
    }
    
    // Getters
    public String getOrderId() { return orderId; }
    public BigDecimal getAmount() { return amount; }
    public OrderStatus getStatus() { return status; }
}

/**
 * Advanced Event Listeners with Conditional Processing
 */
@Component
public class AdvancedEventListener {
    
    private static final Logger logger = LoggerFactory.getLogger(AdvancedEventListener.class);
    
    // Conditional event listening based on SpEL
    @EventListener(condition = "#event.userType.name() == 'PREMIUM'")
    public void handlePremiumUserRegistration(UserRegisteredEvent event) {
        logger.info("Processing premium user registration: {}", event.getUserId());
        // Send welcome package, assign premium support, etc.
    }
    
    @EventListener(condition = "#event.amount.compareTo(new java.math.BigDecimal('1000')) > 0")
    public void handleHighValueOrder(OrderProcessedEvent event) {
        logger.info("Processing high-value order: {} - ${}", 
                   event.getOrderId(), event.getAmount());
        // Special handling for high-value orders
    }
    
    // Async event processing
    @Async("eventTaskExecutor")
    @EventListener
    public void handleUserRegistrationAsync(UserRegisteredEvent event) {
        logger.info("Async processing user registration: {}", event.getUserId());
        
        try {
            // Simulate long-running task
            Thread.sleep(2000);
            
            // Send welcome email
            sendWelcomeEmail(event.getEmail());
            
            // Update analytics
            updateUserAnalytics(event.getUserId(), event.getUserType());
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            logger.error("Async processing interrupted for user: {}", event.getUserId());
        } catch (Exception e) {
            logger.error("Error in async processing for user: {}", event.getUserId(), e);
        }
    }
    
    // Ordered event processing
    @Order(1)
    @EventListener
    public void handleOrderValidation(OrderProcessedEvent event) {
        logger.info("Step 1: Validating order {}", event.getOrderId());
        // First priority processing
    }
    
    @Order(2)
    @EventListener
    public void handleOrderInventory(OrderProcessedEvent event) {
        logger.info("Step 2: Updating inventory for order {}", event.getOrderId());
        // Second priority processing
    }
    
    @Order(3)
    @EventListener
    public void handleOrderNotification(OrderProcessedEvent event) {
        logger.info("Step 3: Sending notifications for order {}", event.getOrderId());
        // Third priority processing
    }
    
    // Error handling in event listeners
    @EventListener
    public void handleEventWithErrorHandling(BaseApplicationEvent event) {
        try {
            processEvent(event);
        } catch (Exception e) {
            logger.error("Error processing event {}: {}", event.getEventId(), e.getMessage());
            
            // Publish error event for monitoring
            publishErrorEvent(event, e);
        }
    }
    
    private void sendWelcomeEmail(String email) {
        // Email implementation
    }
    
    private void updateUserAnalytics(String userId, UserType userType) {
        // Analytics implementation
    }
    
    private void processEvent(BaseApplicationEvent event) {
        // Event processing logic
    }
    
    private void publishErrorEvent(BaseApplicationEvent originalEvent, Exception error) {
        // Error event publishing
    }
}

/**
 * Custom Event Publisher with Transaction Support
 */
@Service
@Transactional
public class TransactionalEventPublisher {
    
    private final ApplicationEventPublisher eventPublisher;
    
    public TransactionalEventPublisher(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }
    
    public void publishUserRegistration(String userId, String email, UserType userType) {
        // Business logic within transaction
        saveUserToDatabase(userId, email, userType);
        
        // Event published after transaction commits
        UserRegisteredEvent event = new UserRegisteredEvent(this, userId, email, userType);
        eventPublisher.publishEvent(event);
    }
    
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleAfterCommit(UserRegisteredEvent event) {
        // This runs only after successful transaction commit
        sendConfirmationEmail(event.getEmail());
    }
    
    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void handleAfterRollback(UserRegisteredEvent event) {
        // This runs only after transaction rollback
        logFailedRegistration(event.getUserId());
    }
    
    @TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
    public void handleBeforeCommit(UserRegisteredEvent event) {
        // Final validation before commit
        validateUserData(event.getUserId());
    }
    
    private void saveUserToDatabase(String userId, String email, UserType userType) {
        // Database save operation
    }
    
    private void sendConfirmationEmail(String email) {
        // Email confirmation
    }
    
    private void logFailedRegistration(String userId) {
        // Failure logging
    }
    
    private void validateUserData(String userId) {
        // Final validation
    }
}

// Supporting enums and classes
enum UserType { STANDARD, PREMIUM, VIP }
enum OrderStatus { PENDING, PROCESSED, SHIPPED, DELIVERED, CANCELLED }
```

---

## Conditional Bean Creation

### Advanced Conditional Logic and Profile-Based Configuration

**Complex Conditional Bean Registration**

```java
/**
 * Advanced Conditional Bean Creation
 * Custom conditions and profile-based configurations
 */

// Custom condition implementations
public class DatabaseAvailableCondition implements Condition {
    
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        String databaseUrl = env.getProperty("database.url");
        
        if (databaseUrl == null || databaseUrl.isEmpty()) {
            return false;
        }
        
        // Check if database is actually reachable
        return isDatabaseReachable(databaseUrl);
    }
    
    private boolean isDatabaseReachable(String url) {
        try {
            // Simplified connection test
            Connection conn = DriverManager.getConnection(url);
            conn.close();
            return true;
        } catch (SQLException e) {
            return false;
        }
    }
}

public class RedisAvailableCondition implements Condition {
    
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        String redisHost = env.getProperty("redis.host", "localhost");
        int redisPort = env.getProperty("redis.port", Integer.class, 6379);
        
        return isRedisReachable(redisHost, redisPort);
    }
    
    private boolean isRedisReachable(String host, int port) {
        try (Socket socket = new Socket()) {
            socket.connect(new InetSocketAddress(host, port), 1000);
            return true;
        } catch (IOException e) {
            return false;
        }
    }
}

// Custom conditional annotations
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Conditional(DatabaseAvailableCondition.class)
public @interface ConditionalOnDatabase {
}

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Conditional(RedisAvailableCondition.class)
public @interface ConditionalOnRedis {
}

/**
 * Advanced Configuration with Multiple Conditions
 */
@Configuration
public class ConditionalBeanConfiguration {
    
    // Bean created only when database is available
    @Bean
    @ConditionalOnDatabase
    @ConditionalOnProperty(name = "features.database.enabled", havingValue = "true")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create()
            .driverClassName("org.postgresql.Driver")
            .url("${database.url}")
            .username("${database.username}")
            .password("${database.password}")
            .build();
    }
    
    // Fallback bean when database is not available
    @Bean
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource fallbackDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("schema.sql")
            .build();
    }
    
    // Redis cache manager - only when Redis is available
    @Bean
    @ConditionalOnRedis
    @ConditionalOnClass(name = "redis.clients.jedis.Jedis")
    public CacheManager redisCacheManager() {
        RedisCacheManager.Builder builder = RedisCacheManager
            .RedisCacheManagerBuilder
            .fromConnectionFactory(jedisConnectionFactory())
            .cacheDefaults(cacheConfiguration());
        
        return builder.build();
    }
    
    // In-memory cache as fallback
    @Bean
    @ConditionalOnMissingBean(CacheManager.class)
    public CacheManager inMemoryCacheManager() {
        return new ConcurrentMapCacheManager("default", "users", "products");
    }
    
    // Profile-specific configurations
    @Bean
    @Profile("production")
    public PerformanceMonitor productionMonitor() {
        return new DetailedPerformanceMonitor();
    }
    
    @Bean
    @Profile({"development", "test"})
    public PerformanceMonitor developmentMonitor() {
        return new SimplePerformanceMonitor();
    }
    
    // Multiple profile conditions
    @Bean
    @Profile("production & metrics")
    public MetricsCollector productionMetrics() {
        return new PrometheusMetricsCollector();
    }
    
    @Bean
    @Profile("development | test")
    public MetricsCollector developmentMetrics() {
        return new LoggingMetricsCollector();
    }
    
    // Complex conditional logic
    @Bean
    @ConditionalOnExpression("${features.async.enabled:true} and ${server.threads.max:100} > 50")
    public TaskExecutor asyncTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        return executor;
    }
    
    private JedisConnectionFactory jedisConnectionFactory() {
        return new JedisConnectionFactory();
    }
    
    private RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30));
    }
}

/**
 * Custom Configuration Properties with Validation
 */
@ConfigurationProperties(prefix = "app.features")
@Validated
public class FeatureProperties {
    
    @NotNull
    private Boolean databaseEnabled = true;
    
    @NotNull
    private Boolean cacheEnabled = true;
    
    @Min(1)
    @Max(1000)
    private Integer maxConnections = 100;
    
    @Valid
    private SecuritySettings security = new SecuritySettings();
    
    // Getters and setters
    public Boolean getDatabaseEnabled() { return databaseEnabled; }
    public void setDatabaseEnabled(Boolean databaseEnabled) { 
        this.databaseEnabled = databaseEnabled; 
    }
    
    public Boolean getCacheEnabled() { return cacheEnabled; }
    public void setCacheEnabled(Boolean cacheEnabled) { 
        this.cacheEnabled = cacheEnabled; 
    }
    
    public Integer getMaxConnections() { return maxConnections; }
    public void setMaxConnections(Integer maxConnections) { 
        this.maxConnections = maxConnections; 
    }
    
    public SecuritySettings getSecurity() { return security; }
    public void setSecurity(SecuritySettings security) { 
        this.security = security; 
    }
    
    public static class SecuritySettings {
        @NotBlank
        private String tokenSecret = "default-secret";
        
        @Min(300) // 5 minutes minimum
        private Integer tokenExpirationSeconds = 3600;
        
        // Getters and setters
        public String getTokenSecret() { return tokenSecret; }
        public void setTokenSecret(String tokenSecret) { 
            this.tokenSecret = tokenSecret; 
        }
        
        public Integer getTokenExpirationSeconds() { return tokenExpirationSeconds; }
        public void setTokenExpirationSeconds(Integer tokenExpirationSeconds) { 
            this.tokenExpirationSeconds = tokenExpirationSeconds; 
        }
    }
}

/**
 * Environment-Aware Bean Factory
 */
@Component
public class EnvironmentAwareBeanFactory implements EnvironmentAware, BeanFactoryAware {
    
    private Environment environment;
    private BeanFactory beanFactory;
    
    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
        configureBasedOnEnvironment();
    }
    
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }
    
    private void configureBasedOnEnvironment() {
        String[] activeProfiles = environment.getActiveProfiles();
        
        System.out.println("Active profiles: " + Arrays.toString(activeProfiles));
        
        // Environment-specific logging
        if (Arrays.asList(activeProfiles).contains("production")) {
            System.setProperty("logging.level.com.example", "WARN");
        } else {
            System.setProperty("logging.level.com.example", "DEBUG");
        }
        
        // Environment-specific feature flags
        boolean metricsEnabled = environment.getProperty("metrics.enabled", Boolean.class, false);
        if (metricsEnabled) {
            System.setProperty("management.endpoints.web.exposure.include", "health,metrics,prometheus");
        }
    }
    
    public <T> T getEnvironmentSpecificBean(Class<T> beanType) {
        String[] activeProfiles = environment.getActiveProfiles();
        
        // Try to find profile-specific bean first
        for (String profile : activeProfiles) {
            String profileSpecificBeanName = profile + beanType.getSimpleName();
            try {
                return beanFactory.getBean(profileSpecificBeanName, beanType);
            } catch (NoSuchBeanDefinitionException e) {
                // Continue to next profile or default
            }
        }
        
        // Fall back to default bean
        return beanFactory.getBean(beanType);
    }
}

// Supporting interfaces and classes
interface PerformanceMonitor {
    void recordMetric(String name, double value);
}

class DetailedPerformanceMonitor implements PerformanceMonitor {
    @Override
    public void recordMetric(String name, double value) {
        // Detailed monitoring implementation
    }
}

class SimplePerformanceMonitor implements PerformanceMonitor {
    @Override
    public void recordMetric(String name, double value) {
        // Simple monitoring implementation
    }
}

interface MetricsCollector {
    void collect(String metric, Object value);
}

class PrometheusMetricsCollector implements MetricsCollector {
    @Override
    public void collect(String metric, Object value) {
        // Prometheus implementation
    }
}

class LoggingMetricsCollector implements MetricsCollector {
    @Override
    public void collect(String metric, Object value) {
        // Logging implementation
    }
}
```

---

## Performance Optimization

### Memory Management and Bean Optimization

**Advanced Performance Tuning Techniques**

```java
/**
 * Spring Performance Optimization Strategies
 * Memory management, lazy loading, and startup optimization
 */

@Configuration
@EnableAsync
public class PerformanceOptimizationConfiguration {
    
    /**
     * Optimized thread pool configuration
     */
    @Bean
    @Primary
    public TaskExecutor optimizedTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        
        // Calculate optimal thread pool size based on available processors
        int corePoolSize = Runtime.getRuntime().availableProcessors();
        int maxPoolSize = corePoolSize * 2;
        
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("optimized-");
        
        // Use caller-runs policy to prevent task rejection
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        
        // Allow core threads to timeout
        executor.setAllowCoreThreadTimeOut(true);
        executor.setKeepAliveSeconds(60);
        
        // Pre-start core threads for better performance
        executor.initialize();
        
        return executor;
    }
    
    /**
     * Memory-efficient cache configuration
     */
    @Bean
    public CacheManager memoryOptimizedCacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        
        // Configure memory-efficient cache
        Caffeine<Object, Object> caffeine = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .expireAfterAccess(5, TimeUnit.MINUTES)
            .removalListener((key, value, cause) -> {
                // Log cache evictions for monitoring
                System.out.println("Cache entry removed: " + key + ", cause: " + cause);
            })
            .recordStats(); // Enable statistics for monitoring
        
        cacheManager.setCaffeine(caffeine);
        return cacheManager;
    }
}

/**
 * Lazy Loading Strategies for Better Startup Performance
 */
@Component
@Lazy
public class ExpensiveService {
    
    private final DataProcessor dataProcessor;
    private volatile boolean initialized = false;
    private final Object initLock = new Object();
    
    public ExpensiveService(@Lazy DataProcessor dataProcessor) {
        this.dataProcessor = dataProcessor;
        System.out.println("ExpensiveService created (but not initialized)");
    }
    
    @PostConstruct
    public void init() {
        System.out.println("ExpensiveService initialized");
    }
    
    public void processData(String data) {
        ensureInitialized();
        dataProcessor.process(data);
    }
    
    private void ensureInitialized() {
        if (!initialized) {
            synchronized (initLock) {
                if (!initialized) {
                    performExpensiveInitialization();
                    initialized = true;
                }
            }
        }
    }
    
    private void performExpensiveInitialization() {
        // Simulate expensive initialization
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

/**
 * Memory Usage Monitoring and Optimization
 */
@Component
public class MemoryOptimizer implements ApplicationListener<ContextRefreshedEvent>, 
                                      DisposableBean {
    
    private final ScheduledExecutorService scheduler = 
        Executors.newScheduledThreadPool(1, r -> {
            Thread t = new Thread(r, "memory-optimizer");
            t.setDaemon(true);
            return t;
        });
    
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        // Start memory monitoring after context is fully loaded
        startMemoryMonitoring();
        optimizeStartupMemory();
    }
    
    private void startMemoryMonitoring() {
        scheduler.scheduleAtFixedRate(this::monitorMemory, 30, 30, TimeUnit.SECONDS);
    }
    
    private void monitorMemory() {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();
        
        double memoryUsagePercent = (double) usedMemory / maxMemory * 100;
        
        System.out.printf("Memory Usage: %.2f%% (%d MB / %d MB)%n", 
                         memoryUsagePercent, 
                         usedMemory / 1024 / 1024, 
                         maxMemory / 1024 / 1024);
        
        // Trigger GC if memory usage is high
        if (memoryUsagePercent > 80) {
            System.out.println("High memory usage detected, suggesting GC");
            System.gc();
        }
    }
    
    private void optimizeStartupMemory() {
        // Force cleanup of temporary objects created during startup
        System.gc();
        
        // Compact string pool (JDK 8+)
        try {
            Class.forName("sun.misc.VM").getMethod("compact").invoke(null);
        } catch (Exception e) {
            // Ignore if not available
        }
    }
    
    @Override
    public void destroy() throws Exception {
        scheduler.shutdown();
        try {
            if (!scheduler.awaitTermination(5, TimeUnit.SECONDS)) {
                scheduler.shutdownNow();
            }
        } catch (InterruptedException e) {
            scheduler.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}

/**
 * Bean Creation Performance Analyzer
 */
@Component
public class BeanCreationProfiler implements BeanPostProcessor, ApplicationListener<ContextRefreshedEvent> {
    
    private final Map<String, Long> beanCreationTimes = new ConcurrentHashMap<>();
    private final Map<String, Long> beanStartTimes = new ConcurrentHashMap<>();
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
            throws BeansException {
        beanStartTimes.put(beanName, System.nanoTime());
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        Long startTime = beanStartTimes.remove(beanName);
        if (startTime != null) {
            long creationTime = System.nanoTime() - startTime;
            beanCreationTimes.put(beanName, creationTime);
        }
        return bean;
    }
    
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        analyzePerformance();
    }
    
    private void analyzePerformance() {
        System.out.println("\n=== Bean Creation Performance Analysis ===");
        
        // Find slowest beans
        List<Map.Entry<String, Long>> sortedBeans = beanCreationTimes.entrySet()
            .stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
            .limit(10)
            .collect(Collectors.toList());
        
        System.out.println("Top 10 slowest beans to create:");
        for (Map.Entry<String, Long> entry : sortedBeans) {
            double timeMs = entry.getValue() / 1_000_000.0;
            System.out.printf("  %s: %.2f ms%n", entry.getKey(), timeMs);
        }
        
        // Calculate total creation time
        long totalTime = beanCreationTimes.values().stream()
            .mapToLong(Long::longValue)
            .sum();
        
        System.out.printf("Total bean creation time: %.2f ms%n", totalTime / 1_000_000.0);
        System.out.printf("Average bean creation time: %.2f ms%n", 
                         (totalTime / 1_000_000.0) / beanCreationTimes.size());
    }
}
```

---

## Tricky Interview Scenarios

### Advanced Problem-Solving Questions

**Real-World Complex Scenarios and Solutions**

```java
/**
 * Advanced Spring Core Interview Questions and Solutions
 * Covering edge cases, performance issues, and design challenges
 */

/**
 * Scenario 1: Circular Dependency in Prototype Beans
 * Problem: How to handle circular dependencies when both beans are prototype-scoped?
 */
@Component
@Scope("prototype")
public class PrototypeServiceA {
    private PrototypeServiceB serviceB;
    
    // Solution: Use Provider or ObjectFactory to break the cycle
    public PrototypeServiceA(ObjectProvider<PrototypeServiceB> serviceBProvider) {
        // Lazy resolution - serviceB will be created when needed
        this.serviceB = serviceBProvider.getObject();
    }
    
    public void doWork() {
        serviceB.process();
    }
}

@Component
@Scope("prototype")
public class PrototypeServiceB {
    private PrototypeServiceA serviceA;
    
    public PrototypeServiceB(ObjectProvider<PrototypeServiceA> serviceAProvider) {
        this.serviceA = serviceAProvider.getObject();
    }
    
    public void process() {
        // Implementation
    }
}

/**
 * Scenario 2: Bean Override with Different Scopes
 * Problem: What happens when you override a singleton bean with a prototype bean?
 */
@Configuration
public class ScopeOverrideScenario {
    
    @Bean
    @Primary
    public DataService primaryDataService() {
        return new DatabaseDataService(); // Singleton by default
    }
    
    @Bean
    @Scope("prototype")
    public DataService prototypeDataService() {
        return new CacheDataService(); // Prototype scope
    }
    
    // Question: Which bean will be injected by default?
    // Answer: primaryDataService (singleton) due to @Primary annotation
    
    @Service
    public class BusinessService {
        private final DataService dataService;
        
        public BusinessService(DataService dataService) {
            this.dataService = dataService; // Will be singleton
        }
        
        // To get prototype bean, need explicit qualifier:
        // public BusinessService(@Qualifier("prototypeDataService") DataService dataService)
    }
}

/**
 * Scenario 3: BeanPostProcessor Order and Side Effects
 * Problem: Multiple BeanPostProcessors affecting the same bean
 */
@Component
@Order(1)
public class ValidationBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        
        if (bean instanceof Validatable) {
            Validatable validatable = (Validatable) bean;
            if (!validatable.isValid()) {
                throw new BeanCreationException(beanName, "Bean validation failed");
            }
        }
        return bean;
    }
}

@Component
@Order(2)
public class ProxyCreationBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        
        if (bean.getClass().isAnnotationPresent(Monitored.class)) {
            return createMonitoringProxy(bean);
        }
        return bean;
    }
    
    private Object createMonitoringProxy(Object target) {
        return Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            (proxy, method, args) -> {
                long start = System.currentTimeMillis();
                try {
                    return method.invoke(target, args);
                } finally {
                    long duration = System.currentTimeMillis() - start;
                    System.out.println("Method " + method.getName() + 
                                     " took " + duration + "ms");
                }
            });
    }
}

/**
 * Scenario 4: Custom Scope Implementation
 * Problem: Implement a thread-local scope for beans
 */
public class ThreadLocalScope implements Scope {
    
    private final ThreadLocal<Map<String, Object>> threadLocalScope = 
        ThreadLocal.withInitial(HashMap::new);
    
    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadLocalScope.get();
        Object object = scope.get(name);
        
        if (object == null) {
            object = objectFactory.getObject();
            scope.put(name, object);
        }
        
        return object;
    }
    
    @Override
    public Object remove(String name) {
        return threadLocalScope.get().remove(name);
    }
    
    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Store destruction callbacks for cleanup
        Map<String, Object> scope = threadLocalScope.get();
        scope.put(name + "_destruction", callback);
    }
    
    @Override
    public Object resolveContextualObject(String key) {
        return null; // Not implemented for this example
    }
    
    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
    
    // Cleanup method - should be called when thread completes
    public void cleanup() {
        Map<String, Object> scope = threadLocalScope.get();
        
        // Execute destruction callbacks
        scope.entrySet().stream()
            .filter(entry -> entry.getKey().endsWith("_destruction"))
            .forEach(entry -> ((Runnable) entry.getValue()).run());
        
        threadLocalScope.remove();
    }
}

@Configuration
public class CustomScopeConfiguration {
    
    @Bean
    public static CustomScopeConfigurer customScopeConfigurer() {
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        configurer.addScope("threadLocal", new ThreadLocalScope());
        return configurer;
    }
    
    @Bean
    @Scope("threadLocal")
    public UserContext userContext() {
        return new UserContext();
    }
}

/**
 * Scenario 5: Dynamic Bean Registration Based on Runtime Conditions
 * Problem: Register beans dynamically based on external configuration
 */
@Component
public class DynamicBeanRegistrar implements BeanDefinitionRegistryPostProcessor, 
                                           ApplicationListener<ContextRefreshedEvent> {
    
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) 
            throws BeansException {
        
        // Read configuration from external source
        List<ServiceConfig> serviceConfigs = loadServiceConfigurations();
        
        for (ServiceConfig config : serviceConfigs) {
            registerServiceBean(registry, config);
        }
    }
    
    private void registerServiceBean(BeanDefinitionRegistry registry, ServiceConfig config) {
        BeanDefinitionBuilder builder = BeanDefinitionBuilder
            .genericBeanDefinition(DynamicService.class)
            .addConstructorArgValue(config.getName())
            .addConstructorArgValue(config.getProperties())
            .setScope(config.getScope())
            .setLazyInit(config.isLazy());
        
        // Add conditional logic
        if (config.isConditional()) {
            builder.setRole(BeanDefinition.ROLE_APPLICATION);
        }
        
        String beanName = "dynamicService_" + config.getName();
        registry.registerBeanDefinition(beanName, builder.getBeanDefinition());
    }
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 
            throws BeansException {
        // Additional bean factory processing if needed
    }
    
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        // Verify that all dynamic beans were created correctly
        validateDynamicBeans(event.getApplicationContext());
    }
    
    private List<ServiceConfig> loadServiceConfigurations() {
        // Load from database, file system, or external service
        return Arrays.asList(
            new ServiceConfig("emailService", Map.of("host", "smtp.gmail.com"), "singleton", false, true),
            new ServiceConfig("smsService", Map.of("apiKey", "123456"), "prototype", true, false)
        );
    }
    
    private void validateDynamicBeans(ApplicationContext context) {
        String[] beanNames = context.getBeanNamesForType(DynamicService.class);
        System.out.println("Registered dynamic services: " + Arrays.toString(beanNames));
    }
}

/**
 * Scenario 6: Memory Leak Prevention in Custom Scopes
 * Problem: Prevent memory leaks in long-running applications with custom scopes
 */
@Component
public class MemoryLeakPreventionExample implements DisposableBean {
    
    private final Map<String, WeakReference<Object>> beanCache = new ConcurrentHashMap<>();
    private final ScheduledExecutorService cleanupExecutor = 
        Executors.newSingleThreadScheduledExecutor(r -> {
            Thread t = new Thread(r, "bean-cleanup");
            t.setDaemon(true);
            return t;
        });
    
    @PostConstruct
    public void startCleanup() {
        // Periodic cleanup of weak references
        cleanupExecutor.scheduleAtFixedRate(this::cleanupBeanCache, 5, 5, TimeUnit.MINUTES);
    }
    
    public void cacheBean(String name, Object bean) {
        beanCache.put(name, new WeakReference<>(bean));
    }
    
    public Object getCachedBean(String name) {
        WeakReference<Object> ref = beanCache.get(name);
        if (ref != null) {
            Object bean = ref.get();
            if (bean == null) {
                // Bean was garbage collected, remove reference
                beanCache.remove(name);
            }
            return bean;
        }
        return null;
    }
    
    private void cleanupBeanCache() {
        int removedCount = 0;
        Iterator<Map.Entry<String, WeakReference<Object>>> iterator = 
            beanCache.entrySet().iterator();
        
        while (iterator.hasNext()) {
            Map.Entry<String, WeakReference<Object>> entry = iterator.next();
            if (entry.getValue().get() == null) {
                iterator.remove();
                removedCount++;
            }
        }
        
        if (removedCount > 0) {
            System.out.println("Cleaned up " + removedCount + " stale bean references");
        }
    }
    
    @Override
    public void destroy() throws Exception {
        cleanupExecutor.shutdown();
        beanCache.clear();
    }
}

// Supporting classes and interfaces
interface DataService { void getData(); }
class DatabaseDataService implements DataService {
    @Override
    public void getData() { /* Implementation */ }
}
class CacheDataService implements DataService {
    @Override
    public void getData() { /* Implementation */ }
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface Monitored { }

interface Validatable {
    boolean isValid();
    List<String> getValidationErrors();
}

class UserContext {
    private String userId;
    private Map<String, Object> attributes = new HashMap<>();
    
    // Getters and setters
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    public Map<String, Object> getAttributes() { return attributes; }
}

class DynamicService {
    private final String name;
    private final Map<String, Object> properties;
    
    public DynamicService(String name, Map<String, Object> properties) {
        this.name = name;
        this.properties = properties;
    }
    
    public String getName() { return name; }
    public Map<String, Object> getProperties() { return properties; }
}

class ServiceConfig {
    private final String name;
    private final Map<String, Object> properties;
    private final String scope;
    private final boolean lazy;
    private final boolean conditional;
    
    public ServiceConfig(String name, Map<String, Object> properties, 
                        String scope, boolean lazy, boolean conditional) {
        this.name = name;
        this.properties = properties;
        this.scope = scope;
        this.lazy = lazy;
        this.conditional = conditional;
    }
    
    // Getters
    public String getName() { return name; }
    public Map<String, Object> getProperties() { return properties; }
    public String getScope() { return scope; }
    public boolean isLazy() { return lazy; }
    public boolean isConditional() { return conditional; }
}

interface DataProcessor {
    void process(String data);
}
```

---

## Summary and Best Practices

### Key Takeaways for Senior Java Developers

**Essential Principles for Production Systems**

1. **Memory Management**
   - Use appropriate bean scopes (singleton vs prototype)
   - Implement proper cleanup in custom scopes
   - Monitor memory usage and prevent leaks
   - Use weak references for caches

2. **Performance Optimization**
   - Leverage lazy initialization for expensive beans
   - Optimize thread pool configurations
   - Profile bean creation times
   - Use conditional bean creation effectively

3. **Advanced Dependency Injection**
   - Understand circular dependency resolution
   - Use qualifiers effectively for complex scenarios
   - Implement custom factory beans when needed
   - Leverage ObjectProvider for optional dependencies

4. **Event-Driven Architecture**
   - Use async event processing for non-blocking operations
   - Implement proper error handling in event listeners
   - Understand transaction-aware event listeners
   - Order event listeners appropriately

5. **Conditional Configuration**
   - Create custom conditions for complex scenarios
   - Use profiles effectively for environment-specific beans
   - Validate configuration properties
   - Implement fallback mechanisms

6. **Extensibility and Integration**
   - Understand BeanPostProcessor order and side effects
   - Implement custom scopes when needed
   - Use ApplicationContext hierarchy effectively
   - Leverage SpEL for dynamic configurations

**Common Pitfalls to Avoid**

- Over-engineering with too many custom components
- Ignoring memory implications of prototype beans
- Creating circular dependencies without proper resolution
- Not considering startup time impact of eager initialization
- Misunderstanding scope behavior in complex hierarchies
- Not properly handling exceptions in event listeners

---

## Advanced Configuration Management

### Environment-Specific Configuration and Profile Strategies

**Production-Ready Configuration Patterns**

```java
/**
 * Advanced Configuration Management Patterns
 * Environment-specific settings, encrypted properties, and dynamic configuration
 */

@Configuration
@EnableConfigurationProperties({DatabaseProperties.class, SecurityProperties.class})
public class AdvancedConfigurationManagement {
    
    // Multi-environment database configuration
    @Bean
    @Profile("production")
    public DataSource productionDataSource(DatabaseProperties props) {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(props.getUrl());
        config.setUsername(props.getUsername());
        config.setPassword(decryptPassword(props.getPassword()));
        config.setMaximumPoolSize(props.getMaxPoolSize());
        config.setConnectionTimeout(props.getConnectionTimeout().toMillis());
        config.setLeakDetectionThreshold(props.getLeakDetectionThreshold().toMillis());
        return new HikariDataSource(config);
    }
    
    @Bean
    @Profile({"development", "test"})
    public DataSource developmentDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .setName("testdb")
            .build();
    }
    
    // Environment-specific security configuration
    @Bean
    @ConditionalOnProperty(name = "security.jwt.enabled", havingValue = "true")
    public JwtTokenProvider jwtTokenProvider(SecurityProperties securityProps) {
        return new JwtTokenProvider(
            securityProps.getJwt().getSecret(),
            securityProps.getJwt().getExpirationTime()
        );
    }
    
    // Conditional feature flags
    @Bean
    @ConditionalOnProperty(name = "features.analytics.enabled", havingValue = "true", matchIfMissing = false)
    public AnalyticsService analyticsService() {
        return new AdvancedAnalyticsService();
    }
    
    @Bean
    @ConditionalOnMissingBean(AnalyticsService.class)
    public AnalyticsService noOpAnalyticsService() {
        return new NoOpAnalyticsService();
    }
    
    private String decryptPassword(String encryptedPassword) {
        // Implement password decryption logic
        return encryptedPassword.startsWith("{encrypted}") ? 
            decrypt(encryptedPassword.substring(11)) : encryptedPassword;
    }
    
    private String decrypt(String encrypted) {
        // AES decryption implementation
        return encrypted; // Placeholder
    }
}

@ConfigurationProperties(prefix = "app.database")
@Validated
public class DatabaseProperties {
    
    @NotBlank
    private String url;
    
    @NotBlank
    private String username;
    
    @NotBlank
    private String password;
    
    @Min(1)
    @Max(100)
    private int maxPoolSize = 10;
    
    @NotNull
    private Duration connectionTimeout = Duration.ofSeconds(30);
    
    @NotNull
    private Duration leakDetectionThreshold = Duration.ofMinutes(2);
    
    // Getters and setters...
    public String getUrl() { return url; }
    public void setUrl(String url) { this.url = url; }
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
    
    public int getMaxPoolSize() { return maxPoolSize; }
    public void setMaxPoolSize(int maxPoolSize) { this.maxPoolSize = maxPoolSize; }
    
    public Duration getConnectionTimeout() { return connectionTimeout; }
    public void setConnectionTimeout(Duration connectionTimeout) { 
        this.connectionTimeout = connectionTimeout; 
    }
    
    public Duration getLeakDetectionThreshold() { return leakDetectionThreshold; }
    public void setLeakDetectionThreshold(Duration leakDetectionThreshold) { 
        this.leakDetectionThreshold = leakDetectionThreshold; 
    }
}

@ConfigurationProperties(prefix = "app.security")
public class SecurityProperties {
    
    private Jwt jwt = new Jwt();
    private Oauth oauth = new Oauth();
    private Cors cors = new Cors();
    
    public static class Jwt {
        private String secret = "default-secret-change-in-production";
        private Duration expirationTime = Duration.ofHours(24);
        private String issuer = "spring-app";
        
        // Getters and setters...
        public String getSecret() { return secret; }
        public void setSecret(String secret) { this.secret = secret; }
        
        public Duration getExpirationTime() { return expirationTime; }
        public void setExpirationTime(Duration expirationTime) { 
            this.expirationTime = expirationTime; 
        }
        
        public String getIssuer() { return issuer; }
        public void setIssuer(String issuer) { this.issuer = issuer; }
    }
    
    public static class Oauth {
        private String clientId;
        private String clientSecret;
        private String tokenUri;
        private String userInfoUri;
        
        // Getters and setters...
        public String getClientId() { return clientId; }
        public void setClientId(String clientId) { this.clientId = clientId; }
        
        public String getClientSecret() { return clientSecret; }
        public void setClientSecret(String clientSecret) { this.clientSecret = clientSecret; }
        
        public String getTokenUri() { return tokenUri; }
        public void setTokenUri(String tokenUri) { this.tokenUri = tokenUri; }
        
        public String getUserInfoUri() { return userInfoUri; }
        public void setUserInfoUri(String userInfoUri) { this.userInfoUri = userInfoUri; }
    }
    
    public static class Cors {
        private List<String> allowedOrigins = Arrays.asList("http://localhost:3000");
        private List<String> allowedMethods = Arrays.asList("GET", "POST", "PUT", "DELETE");
        private List<String> allowedHeaders = Arrays.asList("*");
        private boolean allowCredentials = true;
        
        // Getters and setters...
        public List<String> getAllowedOrigins() { return allowedOrigins; }
        public void setAllowedOrigins(List<String> allowedOrigins) { 
            this.allowedOrigins = allowedOrigins; 
        }
        
        public List<String> getAllowedMethods() { return allowedMethods; }
        public void setAllowedMethods(List<String> allowedMethods) { 
            this.allowedMethods = allowedMethods; 
        }
        
        public List<String> getAllowedHeaders() { return allowedHeaders; }
        public void setAllowedHeaders(List<String> allowedHeaders) { 
            this.allowedHeaders = allowedHeaders; 
        }
        
        public boolean isAllowCredentials() { return allowCredentials; }
        public void setAllowCredentials(boolean allowCredentials) { 
            this.allowCredentials = allowCredentials; 
        }
    }
    
    // Main class getters and setters
    public Jwt getJwt() { return jwt; }
    public void setJwt(Jwt jwt) { this.jwt = jwt; }
    
    public Oauth getOauth() { return oauth; }
    public void setOauth(Oauth oauth) { this.oauth = oauth; }
    
    public Cors getCors() { return cors; }
    public void setCors(Cors cors) { this.cors = cors; }
}

// Supporting interfaces and classes
interface AnalyticsService {
    void trackEvent(String event, Map<String, Object> properties);
}

class AdvancedAnalyticsService implements AnalyticsService {
    @Override
    public void trackEvent(String event, Map<String, Object> properties) {
        // Advanced analytics implementation
    }
}

class NoOpAnalyticsService implements AnalyticsService {
    @Override
    public void trackEvent(String event, Map<String, Object> properties) {
        // No-op implementation for disabled analytics
    }
}

class JwtTokenProvider {
    private final String secret;
    private final Duration expirationTime;
    
    public JwtTokenProvider(String secret, Duration expirationTime) {
        this.secret = secret;
        this.expirationTime = expirationTime;
    }
    
    public String createToken(String username) {
        // JWT token creation logic
        return "jwt-token-for-" + username;
    }
}
```

This comprehensive guide covers the advanced aspects of Spring Core that experienced Java developers need to master for building robust, scalable enterprise applications. From deep IoC container knowledge to advanced AOP patterns, custom scopes, performance optimization, and sophisticated configuration management - this guide provides the expertise needed for senior-level Spring development.
```


