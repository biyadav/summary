# Spring Boot External Services Communication Guide
## Comprehensive Guide for Enterprise Applications

## Table of Contents
1. [HTTP Clients Configuration](#http-clients)
2. [RestTemplate vs WebClient](#resttemplate-vs-webclient)
3. [Resilience Patterns](#resilience-patterns)
4. [Configuration Management](#configuration-management)
5. [Error Handling Strategies](#error-handling)
6. [Monitoring and Observability](#monitoring)
7. [Security Considerations](#security)
8. [Testing External Services](#testing)
9. [Best Practices](#best-practices)

---

## HTTP Clients Configuration

### RestTemplate Configuration

**Production-Ready RestTemplate Setup**

```java
/**
 * RestTemplate Configuration for External Service Communication
 * Includes connection pooling, timeouts, and error handling
 */
@Configuration
public class RestTemplateConfiguration {
    
    @Bean
    @Primary
    public RestTemplate defaultRestTemplate() {
        return new RestTemplateBuilder()
            .setConnectTimeout(Duration.ofSeconds(5))
            .setReadTimeout(Duration.ofSeconds(30))
            .errorHandler(new CustomResponseErrorHandler())
            .additionalInterceptors(
                new LoggingInterceptor(),
                new MetricsInterceptor(),
                new RetryInterceptor()
            )
            .build();
    }
    
    @Bean
    @Qualifier("externalApiRestTemplate")
    public RestTemplate externalApiRestTemplate() {
        // HTTP client with connection pooling
        CloseableHttpClient httpClient = HttpClients.custom()
            .setMaxConnTotal(100)
            .setMaxConnPerRoute(20)
            .setConnectionTimeToLive(30, TimeUnit.SECONDS)
            .setDefaultRequestConfig(RequestConfig.custom()
                .setSocketTimeout(30000)
                .setConnectTimeout(5000)
                .setConnectionRequestTimeout(5000)
                .build())
            .build();
        
        HttpComponentsClientHttpRequestFactory factory = 
            new HttpComponentsClientHttpRequestFactory(httpClient);
        
        RestTemplate restTemplate = new RestTemplate(factory);
        restTemplate.setErrorHandler(new ExternalApiErrorHandler());
        
        // Add interceptors
        List<ClientHttpRequestInterceptor> interceptors = new ArrayList<>();
        interceptors.add(new AuthenticationInterceptor());
        interceptors.add(new RateLimitInterceptor());
        interceptors.add(new CircuitBreakerInterceptor());
        restTemplate.setInterceptors(interceptors);
        
        return restTemplate;
    }
    
    @Bean
    @Qualifier("timeoutRestTemplate")
    public RestTemplate timeoutRestTemplate() {
        return new RestTemplateBuilder()
            .setConnectTimeout(Duration.ofSeconds(2))
            .setReadTimeout(Duration.ofSeconds(10))
            .build();
    }
}

/**
 * Custom Error Handler for RestTemplate
 */
public class CustomResponseErrorHandler implements ResponseErrorHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(CustomResponseErrorHandler.class);
    
    @Override
    public boolean hasError(ClientHttpResponse response) throws IOException {
        HttpStatus statusCode = response.getStatusCode();
        return statusCode.is4xxClientError() || statusCode.is5xxServerError();
    }
    
    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
        HttpStatus statusCode = response.getStatusCode();
        String responseBody = StreamUtils.copyToString(response.getBody(), StandardCharsets.UTF_8);
        
        logger.error("HTTP Error: {} - {}", statusCode, responseBody);
        
        switch (statusCode.series()) {
            case CLIENT_ERROR:
                handleClientError(statusCode, responseBody);
                break;
            case SERVER_ERROR:
                handleServerError(statusCode, responseBody);
                break;
            default:
                throw new RestClientException("Unexpected HTTP status: " + statusCode);
        }
    }
    
    private void handleClientError(HttpStatus statusCode, String responseBody) {
        switch (statusCode) {
            case BAD_REQUEST:
                throw new BadRequestException("Invalid request: " + responseBody);
            case UNAUTHORIZED:
                throw new UnauthorizedException("Authentication failed");
            case FORBIDDEN:
                throw new ForbiddenException("Access denied");
            case NOT_FOUND:
                throw new NotFoundException("Resource not found");
            case TOO_MANY_REQUESTS:
                throw new RateLimitExceededException("Rate limit exceeded");
            default:
                throw new ClientErrorException("Client error: " + statusCode);
        }
    }
    
    private void handleServerError(HttpStatus statusCode, String responseBody) {
        switch (statusCode) {
            case INTERNAL_SERVER_ERROR:
                throw new InternalServerErrorException("Server error: " + responseBody);
            case BAD_GATEWAY:
                throw new BadGatewayException("Bad gateway");
            case SERVICE_UNAVAILABLE:
                throw new ServiceUnavailableException("Service unavailable");
            case GATEWAY_TIMEOUT:
                throw new GatewayTimeoutException("Gateway timeout");
            default:
                throw new ServerErrorException("Server error: " + statusCode);
        }
    }
}

/**
 * WebClient Configuration (Reactive)
 */
@Configuration
public class WebClientConfiguration {
    
    @Bean
    @Primary
    public WebClient defaultWebClient() {
        return WebClient.builder()
            .clientConnector(new ReactorClientHttpConnector(
                HttpClient.create()
                    .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
                    .responseTimeout(Duration.ofSeconds(30))
                    .doOnConnected(conn -> 
                        conn.addHandlerLast(new ReadTimeoutHandler(30))
                            .addHandlerLast(new WriteTimeoutHandler(30)))
            ))
            .defaultHeader(HttpHeaders.USER_AGENT, "Spring-Boot-App/1.0")
            .filter(logRequest())
            .filter(logResponse())
            .filter(retryFilter())
            .build();
    }
    
    @Bean
    @Qualifier("externalApiWebClient")
    public WebClient externalApiWebClient(@Value("${external.api.base-url}") String baseUrl) {
        return WebClient.builder()
            .baseUrl(baseUrl)
            .clientConnector(new ReactorClientHttpConnector(
                HttpClient.create()
                    .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 3000)
                    .responseTimeout(Duration.ofSeconds(20))
                    .compress(true)
            ))
            .defaultHeaders(headers -> {
                headers.set(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
                headers.set(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE);
            })
            .filter(authenticationFilter())
            .filter(circuitBreakerFilter())
            .filter(metricsFilter())
            .build();
    }
    
    private ExchangeFilterFunction logRequest() {
        return ExchangeFilterFunction.ofRequestProcessor(clientRequest -> {
            logger.info("Request: {} {}", clientRequest.method(), clientRequest.url());
            clientRequest.headers().forEach((name, values) -> 
                values.forEach(value -> logger.debug("{}={}", name, value)));
            return Mono.just(clientRequest);
        });
    }
    
    private ExchangeFilterFunction logResponse() {
        return ExchangeFilterFunction.ofResponseProcessor(clientResponse -> {
            logger.info("Response: {}", clientResponse.statusCode());
            return Mono.just(clientResponse);
        });
    }
    
    private ExchangeFilterFunction retryFilter() {
        return (request, next) -> {
            return next.exchange(request)
                .retryWhen(Retry.backoff(3, Duration.ofSeconds(1))
                    .filter(throwable -> throwable instanceof ConnectException ||
                                       throwable instanceof TimeoutException)
                    .onRetryExhaustedThrow((retryBackoffSpec, retrySignal) ->
                        new ServiceUnavailableException("Service unavailable after retries")));
        };
    }
    
    private ExchangeFilterFunction authenticationFilter() {
        return (request, next) -> {
            ClientRequest authenticatedRequest = ClientRequest.from(request)
                .header(HttpHeaders.AUTHORIZATION, "Bearer " + getAuthToken())
                .build();
            return next.exchange(authenticatedRequest);
        };
    }
    
    private ExchangeFilterFunction circuitBreakerFilter() {
        return (request, next) -> {
            return next.exchange(request)
                .transformDeferred(CircuitBreakerOperator.of(
                    CircuitBreaker.ofDefaults("external-api")));
        };
    }
    
    private ExchangeFilterFunction metricsFilter() {
        return (request, next) -> {
            Timer.Sample sample = Timer.start(Metrics.globalRegistry);
            return next.exchange(request)
                .doOnSuccess(response -> 
                    sample.stop(Timer.builder("http.client.requests")
                        .tag("uri", request.url().getPath())
                        .tag("status", String.valueOf(response.statusCode().value()))
                        .register(Metrics.globalRegistry)))
                .doOnError(error -> 
                    sample.stop(Timer.builder("http.client.requests")
                        .tag("uri", request.url().getPath())
                        .tag("status", "error")
                        .register(Metrics.globalRegistry)));
        };
    }
    
    private String getAuthToken() {
        // Implement token retrieval logic
        return "your-auth-token";
    }
}
```

---

## Resilience Patterns

### Circuit Breaker Implementation

**Advanced Circuit Breaker with Resilience4j**

```java
/**
 * Circuit Breaker Configuration for External Services
 */
@Configuration
@EnableConfigurationProperties(CircuitBreakerProperties.class)
public class CircuitBreakerConfiguration {
    
    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry(CircuitBreakerProperties properties) {
        CircuitBreakerConfig defaultConfig = CircuitBreakerConfig.custom()
            .failureRateThreshold(properties.getFailureRateThreshold())
            .waitDurationInOpenState(properties.getWaitDurationInOpenState())
            .slidingWindowSize(properties.getSlidingWindowSize())
            .minimumNumberOfCalls(properties.getMinimumNumberOfCalls())
            .permittedNumberOfCallsInHalfOpenState(properties.getPermittedNumberOfCallsInHalfOpenState())
            .recordExceptions(
                ConnectException.class,
                TimeoutException.class,
                ServiceUnavailableException.class
            )
            .ignoreExceptions(
                BadRequestException.class,
                UnauthorizedException.class
            )
            .build();
        
        return CircuitBreakerRegistry.of(defaultConfig);
    }
    
    @Bean
    public RetryRegistry retryRegistry(CircuitBreakerProperties properties) {
        RetryConfig retryConfig = RetryConfig.custom()
            .maxAttempts(properties.getRetryMaxAttempts())
            .waitDuration(properties.getRetryWaitDuration())
            .exponentialBackoffMultiplier(properties.getRetryBackoffMultiplier())
            .retryExceptions(
                ConnectException.class,
                SocketTimeoutException.class,
                ServiceUnavailableException.class
            )
            .ignoreExceptions(
                BadRequestException.class,
                UnauthorizedException.class,
                NotFoundException.class
            )
            .build();
        
        return RetryRegistry.of(retryConfig);
    }
    
    @Bean
    public TimeLimiterRegistry timeLimiterRegistry(CircuitBreakerProperties properties) {
        TimeLimiterConfig timeLimiterConfig = TimeLimiterConfig.custom()
            .timeoutDuration(properties.getTimeoutDuration())
            .cancelRunningFuture(true)
            .build();
        
        return TimeLimiterRegistry.of(timeLimiterConfig);
    }
    
    @Bean
    public BulkheadRegistry bulkheadRegistry(CircuitBreakerProperties properties) {
        BulkheadConfig bulkheadConfig = BulkheadConfig.custom()
            .maxConcurrentCalls(properties.getBulkheadMaxConcurrentCalls())
            .maxWaitDuration(properties.getBulkheadMaxWaitDuration())
            .build();
        
        return BulkheadRegistry.of(bulkheadConfig);
    }
}

/**
 * External Service Client with Resilience Patterns
 */
@Service
public class ExternalServiceClient {
    
    private final RestTemplate restTemplate;
    private final WebClient webClient;
    private final CircuitBreaker circuitBreaker;
    private final Retry retry;
    private final TimeLimiter timeLimiter;
    private final Bulkhead bulkhead;
    
    public ExternalServiceClient(
            @Qualifier("externalApiRestTemplate") RestTemplate restTemplate,
            @Qualifier("externalApiWebClient") WebClient webClient,
            CircuitBreakerRegistry circuitBreakerRegistry,
            RetryRegistry retryRegistry,
            TimeLimiterRegistry timeLimiterRegistry,
            BulkheadRegistry bulkheadRegistry) {
        
        this.restTemplate = restTemplate;
        this.webClient = webClient;
        this.circuitBreaker = circuitBreakerRegistry.circuitBreaker("external-service");
        this.retry = retryRegistry.retry("external-service");
        this.timeLimiter = timeLimiterRegistry.timeLimiter("external-service");
        this.bulkhead = bulkheadRegistry.bulkhead("external-service");
        
        // Register event listeners
        registerEventListeners();
    }
    
    /**
     * Synchronous call with full resilience patterns
     */
    public Optional<ExternalApiResponse> callExternalServiceSync(ExternalApiRequest request) {
        Supplier<ExternalApiResponse> decoratedSupplier = Decorators
            .ofSupplier(() -> performSyncCall(request))
            .withCircuitBreaker(circuitBreaker)
            .withRetry(retry)
            .withBulkhead(bulkhead)
            .withFallback(Arrays.asList(
                ConnectException.class,
                TimeoutException.class,
                ServiceUnavailableException.class
            ), throwable -> {
                logger.warn("Fallback triggered for sync call: {}", throwable.getMessage());
                return getFallbackResponse();
            })
            .decorate();
        
        try {
            return Optional.ofNullable(decoratedSupplier.get());
        } catch (Exception e) {
            logger.error("External service call failed: {}", e.getMessage());
            return Optional.empty();
        }
    }
    
    /**
     * Asynchronous call with reactive resilience patterns
     */
    public Mono<ExternalApiResponse> callExternalServiceAsync(ExternalApiRequest request) {
        return webClient
            .post()
            .uri("/api/endpoint")
            .bodyValue(request)
            .retrieve()
            .bodyToMono(ExternalApiResponse.class)
            .transformDeferred(CircuitBreakerOperator.of(circuitBreaker))
            .transformDeferred(RetryOperator.of(retry))
            .transformDeferred(BulkheadOperator.of(bulkhead))
            .transformDeferred(TimeLimiterOperator.of(timeLimiter))
            .onErrorResume(throwable -> {
                logger.warn("Async call failed, using fallback: {}", throwable.getMessage());
                return Mono.just(getFallbackResponse());
            })
            .doOnSuccess(response -> logger.info("Async call successful"))
            .doOnError(error -> logger.error("Async call error: {}", error.getMessage()));
    }
    
    /**
     * Batch processing with bulkhead isolation
     */
    public CompletableFuture<List<ExternalApiResponse>> processBatch(List<ExternalApiRequest> requests) {
        List<CompletableFuture<ExternalApiResponse>> futures = requests.stream()
            .map(request -> CompletableFuture
                .supplyAsync(() -> callExternalServiceSync(request).orElse(null), 
                           bulkhead.executeCompletionStage(() -> 
                               CompletableFuture.completedFuture(null)).toCompletableFuture().join()
                           )
                .orTimeout(30, TimeUnit.SECONDS))
            .collect(Collectors.toList());
        
        return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                .map(CompletableFuture::join)
                .filter(Objects::nonNull)
                .collect(Collectors.toList()));
    }
    
    private ExternalApiResponse performSyncCall(ExternalApiRequest request) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.setBearerAuth(getAuthToken());
        
        HttpEntity<ExternalApiRequest> entity = new HttpEntity<>(request, headers);
        
        ResponseEntity<ExternalApiResponse> response = restTemplate.postForEntity(
            "/api/endpoint", entity, ExternalApiResponse.class);
        
        if (response.getStatusCode().is2xxSuccessful()) {
            return response.getBody();
        } else {
            throw new ServiceUnavailableException("Service returned: " + response.getStatusCode());
        }
    }
    
    private ExternalApiResponse getFallbackResponse() {
        return ExternalApiResponse.builder()
            .status("FALLBACK")
            .message("Service temporarily unavailable")
            .timestamp(Instant.now())
            .build();
    }
    
    private void registerEventListeners() {
        circuitBreaker.getEventPublisher()
            .onStateTransition(event -> 
                logger.info("Circuit breaker state transition: {} -> {}", 
                           event.getStateTransition().getFromState(),
                           event.getStateTransition().getToState()));
        
        circuitBreaker.getEventPublisher()
            .onFailureRateExceeded(event -> 
                logger.warn("Circuit breaker failure rate exceeded: {}%", 
                           event.getFailureRate()));
        
        retry.getEventPublisher()
            .onRetry(event -> 
                logger.info("Retry attempt {} for: {}", 
                           event.getNumberOfRetryAttempts(),
                           event.getLastThrowable().getMessage()));
    }
    
    private String getAuthToken() {
        // Implement authentication token retrieval
        return "Bearer token";
    }
}

/**
 * Health Check for External Services
 */
@Component
public class ExternalServiceHealthIndicator implements HealthIndicator {
    
    private final ExternalServiceClient externalServiceClient;
    private final CircuitBreaker circuitBreaker;
    
    public ExternalServiceHealthIndicator(ExternalServiceClient client,
                                        CircuitBreakerRegistry registry) {
        this.externalServiceClient = client;
        this.circuitBreaker = registry.circuitBreaker("external-service");
    }
    
    @Override
    public Health health() {
        try {
            CircuitBreaker.State state = circuitBreaker.getState();
            
            if (state == CircuitBreaker.State.OPEN) {
                return Health.down()
                    .withDetail("circuit-breaker", "OPEN")
                    .withDetail("reason", "Circuit breaker is open")
                    .build();
            }
            
            // Perform lightweight health check
            boolean isHealthy = performHealthCheck();
            
            if (isHealthy) {
                return Health.up()
                    .withDetail("circuit-breaker", state.toString())
                    .withDetail("status", "Service is healthy")
                    .build();
            } else {
                return Health.down()
                    .withDetail("circuit-breaker", state.toString())
                    .withDetail("reason", "Health check failed")
                    .build();
            }
            
        } catch (Exception e) {
            return Health.down(e)
                .withDetail("error", e.getMessage())
                .build();
        }
    }
    
    private boolean performHealthCheck() {
        try {
            // Implement lightweight health check
            // This should be a simple, non-intrusive call
            return true;
        } catch (Exception e) {
            logger.error("Health check failed: {}", e.getMessage());
            return false;
        }
    }
}
```

---

## Configuration Management

### External Service Configuration Properties

**Comprehensive Configuration Setup**

```java
/**
 * External Service Configuration Properties
 */
@ConfigurationProperties(prefix = "external.services")
@Validated
@Data
public class ExternalServiceProperties {
    
    @Valid
    private Map<String, ServiceConfig> services = new HashMap<>();
    
    @Valid
    private GlobalConfig global = new GlobalConfig();
    
    @Data
    public static class ServiceConfig {
        @NotBlank
        @URL
        private String baseUrl;
        
        @NotNull
        @DurationMin(seconds = 1)
        @DurationMax(minutes = 5)
        private Duration connectTimeout = Duration.ofSeconds(5);
        
        @NotNull
        @DurationMin(seconds = 1)
        @DurationMax(minutes = 10)
        private Duration readTimeout = Duration.ofSeconds(30);
        
        @Min(1)
        @Max(1000)
        private int maxConnections = 50;
        
        @Min(1)
        @Max(100)
        private int maxConnectionsPerRoute = 10;
        
        @Valid
        private AuthConfig auth = new AuthConfig();
        
        @Valid
        private RetryConfig retry = new RetryConfig();
        
        @Valid
        private CircuitBreakerConfig circuitBreaker = new CircuitBreakerConfig();
        
        @Valid
        private RateLimitConfig rateLimit = new RateLimitConfig();
        
        private Map<String, String> headers = new HashMap<>();
        
        private boolean compressionEnabled = true;
        private boolean metricsEnabled = true;
        private boolean loggingEnabled = true;
    }
    
    @Data
    public static class AuthConfig {
        private AuthType type = AuthType.NONE;
        private String username;
        private String password;
        private String token;
        private String tokenHeader = "Authorization";
        private String tokenPrefix = "Bearer ";
        private Duration tokenRefreshInterval = Duration.ofMinutes(30);
        private String clientId;
        private String clientSecret;
        private String tokenEndpoint;
        private List<String> scopes = new ArrayList<>();
    }
    
    @Data
    public static class RetryConfig {
        @Min(0)
        @Max(10)
        private int maxAttempts = 3;
        
        @NotNull
        @DurationMin(millis = 100)
        @DurationMax(seconds = 30)
        private Duration waitDuration = Duration.ofSeconds(1);
        
        @DecimalMin("1.0")
        @DecimalMax("10.0")
        private double backoffMultiplier = 2.0;
        
        @NotNull
        @DurationMin(seconds = 1)
        @DurationMax(minutes = 5)
        private Duration maxWaitDuration = Duration.ofSeconds(30);
        
        private List<Class<? extends Throwable>> retryExceptions = Arrays.asList(
            ConnectException.class,
            SocketTimeoutException.class,
            ServiceUnavailableException.class
        );
        
        private List<Class<? extends Throwable>> ignoreExceptions = Arrays.asList(
            BadRequestException.class,
            UnauthorizedException.class,
            NotFoundException.class
        );
    }
    
    @Data
    public static class CircuitBreakerConfig {
        @DecimalMin("1.0")
        @DecimalMax("100.0")
        private double failureRateThreshold = 50.0;
        
        @NotNull
        @DurationMin(seconds = 1)
        @DurationMax(minutes = 10)
        private Duration waitDurationInOpenState = Duration.ofSeconds(60);
        
        @Min(1)
        @Max(1000)
        private int slidingWindowSize = 100;
        
        @Min(1)
        @Max(100)
        private int minimumNumberOfCalls = 10;
        
        @Min(1)
        @Max(50)
        private int permittedNumberOfCallsInHalfOpenState = 3;
        
        private boolean automaticTransitionFromOpenToHalfOpenEnabled = true;
    }
    
    @Data
    public static class RateLimitConfig {
        @Min(1)
        @Max(10000)
        private int requestsPerSecond = 100;
        
        @Min(1)
        @Max(3600)
        private int windowSizeInSeconds = 1;
        
        @NotNull
        @DurationMin(millis = 1)
        @DurationMax(seconds = 60)
        private Duration timeoutDuration = Duration.ofSeconds(5);
    }
    
    @Data
    public static class GlobalConfig {
        @NotNull
        @DurationMin(seconds = 1)
        @DurationMax(minutes = 5)
        private Duration defaultConnectTimeout = Duration.ofSeconds(5);
        
        @NotNull
        @DurationMin(seconds = 1)
        @DurationMax(minutes = 10)
        private Duration defaultReadTimeout = Duration.ofSeconds(30);
        
        @Min(1)
        @Max(1000)
        private int defaultMaxConnections = 100;
        
        private boolean metricsEnabled = true;
        private boolean loggingEnabled = true;
        private boolean healthCheckEnabled = true;
        
        @NotNull
        @DurationMin(seconds = 10)
        @DurationMax(minutes = 10)
        private Duration healthCheckInterval = Duration.ofMinutes(1);
        
        private String userAgent = "Spring-Boot-App/1.0";
    }
    
    public enum AuthType {
        NONE, BASIC, BEARER, OAUTH2, API_KEY, CUSTOM
    }
}

/**
 * Dynamic Configuration for External Services
 */
@Component
@ConfigurationProperties(prefix = "external.services")
@RefreshScope
public class DynamicExternalServiceConfiguration {
    
    private final ExternalServiceProperties properties;
    private final Map<String, RestTemplate> restTemplates = new ConcurrentHashMap<>();
    private final Map<String, WebClient> webClients = new ConcurrentHashMap<>();
    
    public DynamicExternalServiceConfiguration(ExternalServiceProperties properties) {
        this.properties = properties;
        initializeClients();
    }
    
    @EventListener(RefreshScopeRefreshedEvent.class)
    public void onConfigurationRefresh() {
        logger.info("Configuration refreshed, reinitializing clients");
        restTemplates.clear();
        webClients.clear();
        initializeClients();
    }
    
    public RestTemplate getRestTemplate(String serviceName) {
        return restTemplates.computeIfAbsent(serviceName, this::createRestTemplate);
    }
    
    public WebClient getWebClient(String serviceName) {
        return webClients.computeIfAbsent(serviceName, this::createWebClient);
    }
    
    private void initializeClients() {
        properties.getServices().keySet().forEach(serviceName -> {
            createRestTemplate(serviceName);
            createWebClient(serviceName);
        });
    }
    
    private RestTemplate createRestTemplate(String serviceName) {
        ServiceConfig config = properties.getServices().get(serviceName);
        if (config == null) {
            throw new IllegalArgumentException("Service configuration not found: " + serviceName);
        }
        
        // Create HTTP client with configuration
        RequestConfig requestConfig = RequestConfig.custom()
            .setSocketTimeout((int) config.getReadTimeout().toMillis())
            .setConnectTimeout((int) config.getConnectTimeout().toMillis())
            .build();
        
        CloseableHttpClient httpClient = HttpClients.custom()
            .setMaxConnTotal(config.getMaxConnections())
            .setMaxConnPerRoute(config.getMaxConnectionsPerRoute())
            .setDefaultRequestConfig(requestConfig)
            .build();
        
        HttpComponentsClientHttpRequestFactory factory = 
            new HttpComponentsClientHttpRequestFactory(httpClient);
        
        RestTemplate restTemplate = new RestTemplate(factory);
        
        // Add interceptors based on configuration
        List<ClientHttpRequestInterceptor> interceptors = new ArrayList<>();
        
        if (config.getAuth().getType() != AuthType.NONE) {
            interceptors.add(new AuthenticationInterceptor(config.getAuth()));
        }
        
        if (config.isLoggingEnabled()) {
            interceptors.add(new LoggingInterceptor());
        }
        
        if (config.isMetricsEnabled()) {
            interceptors.add(new MetricsInterceptor(serviceName));
        }
        
        restTemplate.setInterceptors(interceptors);
        restTemplate.setErrorHandler(new CustomResponseErrorHandler());
        
        return restTemplate;
    }
    
    private WebClient createWebClient(String serviceName) {
        ServiceConfig config = properties.getServices().get(serviceName);
        if (config == null) {
            throw new IllegalArgumentException("Service configuration not found: " + serviceName);
        }
        
        WebClient.Builder builder = WebClient.builder()
            .baseUrl(config.getBaseUrl())
            .clientConnector(new ReactorClientHttpConnector(
                HttpClient.create()
                    .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 
                           (int) config.getConnectTimeout().toMillis())
                    .responseTimeout(config.getReadTimeout())
                    .compress(config.isCompressionEnabled())
            ));
        
        // Add default headers
        config.getHeaders().forEach(builder::defaultHeader);
        
        // Add authentication filter
        if (config.getAuth().getType() != AuthType.NONE) {
            builder.filter(createAuthenticationFilter(config.getAuth()));
        }
        
        // Add logging filter
        if (config.isLoggingEnabled()) {
            builder.filter(createLoggingFilter(serviceName));
        }
        
        // Add metrics filter
        if (config.isMetricsEnabled()) {
            builder.filter(createMetricsFilter(serviceName));
        }
        
        return builder.build();
    }
    
    private ExchangeFilterFunction createAuthenticationFilter(AuthConfig authConfig) {
        return (request, next) -> {
            ClientRequest.Builder requestBuilder = ClientRequest.from(request);
            
            switch (authConfig.getType()) {
                case BASIC:
                    String credentials = Base64.getEncoder().encodeToString(
                        (authConfig.getUsername() + ":" + authConfig.getPassword()).getBytes());
                    requestBuilder.header(HttpHeaders.AUTHORIZATION, "Basic " + credentials);
                    break;
                case BEARER:
                    requestBuilder.header(authConfig.getTokenHeader(), 
                                        authConfig.getTokenPrefix() + authConfig.getToken());
                    break;
                case API_KEY:
                    requestBuilder.header(authConfig.getTokenHeader(), authConfig.getToken());
                    break;
                case OAUTH2:
                    // Implement OAuth2 token retrieval
                    String token = retrieveOAuth2Token(authConfig);
                    requestBuilder.header(HttpHeaders.AUTHORIZATION, "Bearer " + token);
                    break;
            }
            
            return next.exchange(requestBuilder.build());
        };
    }
    
    private ExchangeFilterFunction createLoggingFilter(String serviceName) {
        return ExchangeFilterFunction.ofRequestProcessor(clientRequest -> {
            logger.info("[{}] {} {}", serviceName, clientRequest.method(), clientRequest.url());
            return Mono.just(clientRequest);
        });
    }
    
    private ExchangeFilterFunction createMetricsFilter(String serviceName) {
        return (request, next) -> {
            Timer.Sample sample = Timer.start(Metrics.globalRegistry);
            return next.exchange(request)
                .doFinally(signalType -> 
                    sample.stop(Timer.builder("external.service.requests")
                        .tag("service", serviceName)
                        .tag("method", request.method().name())
                        .register(Metrics.globalRegistry)));
        };
    }
    
    private String retrieveOAuth2Token(AuthConfig authConfig) {
        // Implement OAuth2 token retrieval logic
        // This should cache tokens and refresh them when needed
        return "oauth2-token";
    }
}
```

---

## Summary: External Service Communication Best Practices

### Key Takeaways for Production Systems

**Essential Patterns and Configurations:**

#### 1. **HTTP Client Configuration**
- **Connection Pooling**: Use HikariCP-style pooling with proper limits
- **Timeouts**: Connect (5s), Read (30s), and overall request timeouts
- **Compression**: Enable GZIP compression for bandwidth optimization
- **Keep-Alive**: Configure connection reuse for performance

#### 2. **Resilience Implementation**
- **Circuit Breaker**: 50% failure rate threshold, 60s wait duration
- **Retry Logic**: 3 attempts with exponential backoff (1s, 2s, 4s)
- **Bulkhead**: Isolate critical vs non-critical service calls
- **Timeout**: Per-service timeout configuration with fallbacks

#### 3. **Error Handling Strategy**
- **Classification**: Retryable (5xx, timeouts) vs Non-retryable (4xx)
- **Fallback Data**: Cached responses, static data, or default values
- **Error Propagation**: Meaningful error messages with context
- **Alerting**: Immediate alerts for circuit breaker opens

#### 4. **Security Configuration**
- **Authentication**: OAuth2, API keys, or JWT tokens with refresh
- **Transport Security**: TLS 1.2+ with certificate validation
- **Request Signing**: HMAC or digital signatures for sensitive APIs
- **Rate Limiting**: Client-side rate limiting to prevent 429 errors

#### 5. **Monitoring and Observability**
- **Metrics**: Response time, error rate, circuit breaker state
- **Tracing**: Distributed tracing for request flow visibility
- **Logging**: Structured logging with correlation IDs
- **Health Checks**: Lightweight endpoints for service availability

#### 6. **Testing Strategies**
- **Unit Tests**: Mock external dependencies with WireMock
- **Integration Tests**: Test against real services in staging
- **Contract Tests**: Verify API compatibility with consumer-driven contracts
- **Chaos Testing**: Simulate failures and validate resilience

#### 7. **Configuration Management**
- **Environment-Specific**: Different configs for dev/test/prod
- **Dynamic Refresh**: Support for runtime configuration updates
- **Validation**: Validate configuration at startup
- **Secrets Management**: Encrypted passwords and tokens

This comprehensive guide provides enterprise-grade patterns for robust external service integration in Spring Boot applications. The examples demonstrate production-ready implementations that handle real-world challenges like network failures, service outages, and performance optimization. Would you like me to continue with additional sections like testing strategies or specific failure scenarios?
