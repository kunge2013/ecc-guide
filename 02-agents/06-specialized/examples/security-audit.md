# 示例：安全漏洞分析

这个示例展示 security-reviewer 如何进行深度安全分析。

## 应用代码

### 1. AuthController.java

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private static final String API_KEY = "sk-1234567890abcdef";  // 硬编码密钥

    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(
            @RequestBody LoginRequest request) {

        // 直接拼接 SQL 查询
        String query = "SELECT * FROM users WHERE username = '"
            + request.getUsername()
            + "' AND password = '"
            + request.getPassword()
            + "'";

        User user = jdbcTemplate.queryForObject(query, userRowMapper);

        if (user != null) {
            String token = generateJWTToken(user);
            return ResponseEntity.ok(new LoginResponse(token));
        } else {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }
    }

    @PostMapping("/register")
    public ResponseEntity<User> register(@RequestBody User user) {
        // 没有验证输入
        userRepository.save(user);
        return ResponseEntity.ok(user);
    }

    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        // 没有权限检查
        return ResponseEntity.ok(userRepository.findById(id).get());
    }

    @GetMapping("/admin/config")
    public ResponseEntity<Config> getConfig() {
        // 返回敏感配置信息
        Config config = new Config();
        config.setDbPassword(getDbPassword());  // 敏感信息
        config.setApiKey(API_KEY);
        return ResponseEntity.ok(config);
    }
}
```

### 2. FileUploadController.java

```java
@RestController
@RequestMapping("/api/files")
public class FileUploadController {

    @Value("${upload.dir}")
    private String uploadDir;

    @PostMapping("/upload")
    public ResponseEntity<String> uploadFile(
            @RequestParam("file") MultipartFile file,
            @RequestParam("filename") String filename) {

        try {
            // 用户控制的文件名
            String filePath = uploadDir + "/" + filename;

            // 直接保存用户文件
            Path path = Paths.get(filePath);
            Files.write(path, file.getBytes());

            return ResponseEntity.ok("File uploaded successfully");
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }

    @GetMapping("/download/{filename}")
    public ResponseEntity<Resource> downloadFile(
            @PathVariable String filename) {

        try {
            // 路径遍历漏洞
            String filePath = uploadDir + "/" + filename;
            Resource resource = new UrlResource(Paths.get(filePath).toUri());

            return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION,
                    "attachment; filename=\"" + resource.getFilename() + "\"")
                .body(resource);
        } catch (Exception e) {
            return ResponseEntity.notFound().build();
        }
    }
}
```

### 3. PaymentService.java

```java
@Service
public class
 PaymentService {

    @Autowired
    private PaymentGateway paymentGateway;

    @Autowired
    private OrderRepository orderRepository;

    public PaymentResult processPayment(Long orderId, PaymentRequest request) {
        Order order = orderRepository.findById(orderId).get();

        // 记录完整的信用卡信息
        log.info("Processing payment for order {}: {}",
            orderId, request.getCardNumber());

        // 保存完整的信用卡信息
        order.setCardNumber(request.getCardNumber());
        order.setCardExpiry(request.getCardExpiry());
        order.setCardCvv(request.getCardCvv());
        orderRepository.save(order);

        // 处理支付
        PaymentResult result = paymentGateway.charge(request);

        return result;
    }
}
```

## 安全审查报告

# 安全漏洞审查报告

## 执行概览
- 审查时间: 2024-03-19 15:30:00
- 审查的文件: 3 个
- 检测到的漏洞: 12 个
- 风险等级: 🔴 CRITICAL

---

## 🔴 CRITICAL 漏洞

### 1. SQL 注入漏洞
**漏洞 ID**: VULN-001
**文件**: `AuthController.java:21`
**严重性**: CRITICAL
**CVSS 评分**: 9.8 (HIGH)
**CWE 编号**: CWE-89

**漏洞描述**:
直接将用户输入拼接到 SQL 查询中，允许攻击者执行任意 SQL 命令。

**漏洞代码**:
```java
String query = "SELECT * FROM users WHERE username = '"
    + request.getUsername()
    + "' AND password = '"
    + request.getPassword()
    + "'";
```

**攻击示例**:
```bash
# 经典的 SQL 注入攻击
username: admin' OR '1'='1' --
password: anything

# 生成的查询
SELECT * FROM users WHERE username = 'admin' OR '1'='1' --' AND password = 'anything'
-- 这个查询会返回 admin 用户，绕过密码验证
```

**影响**:
- 攻击者可以绕过身份验证
- 攻击者可以读取、修改、删除数据库中的任何数据
- 攻击者可以执行系统命令（如果数据库支持）

**修复建议**:
```java
// 使用参数化查询
String query = "SELECT * FROM users WHERE username = ? AND password = ?";

User user = jdbcTemplate.queryForObject(
    query,
    userRowMapper,
    request.getUsername(),
    passwordEncoder.encode(request.getPassword())
);

// 或使用 JPA Repository
User user = userRepository.findByUsername(
    request.getUsername()
);

if (user != null && passwordEncoder.matches(
    request.getPassword(),
    user.getPassword())) {
    // 认证成功
}
```

---

### 2. 硬编码密钥
**漏洞 ID**: VULN-002
**文件**: `AuthController.java:10`
**严重性**: CRITICAL
**CVSS 评分**: 7.5 (HIGH)
**CWE 编号**: CWE-798

**漏洞描述**:
API 密钥直接硬编码在源代码中，会在版本控制系统和编译后的代码中暴露。

**漏洞代码**:
```java
private static final String API_KEY = "sk-1234567890abcdef";
```

**影响**:
- 密钥在源代码中可见
- 密钥在版本控制历史中可见
- 密钥在编译后的代码中可被提取
- 攻击者可以滥用 API 密钥

**修复建议**:
```java
// 使用环境变量
@Value("${api.key}")
private String apiKey;

// 或使用配置服务
@Autowired
private ConfigService configService;

private String getApiKey() {
    return configService.getApiKey("service-name");
}

// 或使用 AWS Secrets Manager
@Autowired
private SecretsManagerClient secretsManager;

private String getApiKey() {
    GetSecretValueRequest request = GetSecretValueRequest.builder()
        .secretId("my-service/api-key")
        .build();
    GetSecretValueResponse response = secretsManager.getSecretValue(request);
    return response.secretString();
}
```

---

### 3. 敏感信息泄露
**漏洞 ID**: VULN-003
**文件**: `AuthController.java:50`
**严重性**: CRITICAL
**CVSS 评分**: 8.1 (HIGH)
**CWE 编号**: CWE-200

**漏洞描述**:
通过 API 端点返回敏感配置信息，包括数据库密码和 API 密钥。

**漏洞代码**:
```java
@GetMapping("/admin/config")
public ResponseEntity<Config> getConfig() {
    Config config = new Config();
    config.setDbPassword(getDbPassword());  // 敏感信息
    config.setApiKey(API_KEY);
    return ResponseEntity.ok(config);
}
```

**影响**:
- 数据库凭据泄露
- API 密钥泄露
- 攻击者可以完全控制系统

**修复建议**:
```java
// 不要返回敏感配置信息
@GetMapping("/admin/config")
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<Config> getConfig() {
    Config config = new Config();
    // 只返回非敏感信息
    config.setVersion(applicationVersion);
    config.setEnvironment(environment);
    return ResponseEntity.ok(config);
}

// 或使用专门的配置端点
@GetMapping("/admin/config/masked")
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntityConfigMasked> getMaskedConfig() {
    ConfigMasked config = new ConfigMasked();
    config.setVersion(applicationVersion);
    config.setEnvironment(environment);
    config.setDbUrl(maskDbUrl(dbUrl));  // 脱敏
    return ResponseEntity.ok(config);
}
```

---

### 4. 路径遍历漏洞
**漏洞 ID**: VULN-004
**文件**: `FileUploadController.java:42`
**严重性**: CRITICAL
**CVSS 评分**: 7.5 (HIGH)
**CWE 编号**: CWE-22

**漏洞描述**:
用户提供的文件名没有验证，攻击者可以使用 `../` 访问任意文件。

**漏洞代码**:
```java
String filePath = uploadDir + "/" + filename;
Path path = Paths.get(filePath);
Resource resource = new UrlResource(Paths.get(filePath).toUri());
```

**攻击示例**:
```bash
# 下载 /etc/passwd 文件
GET /api/files/download/../../../etc/passwd

# 下载源代码
GET /api/files/download/../../../src/main/java/AuthController.java

# 下载数据库配置
GET /api/files/download/../../../application.properties
```

**影响**:
- 攻击者可以读取服务器上的任意文件
- 攻击者可以下载源代码
- 攻击者可以获取敏感配置信息

**修复建议**:
```java
@GetMapping("/download/{filename}")
public ResponseEntity<Resource> downloadFile(
        @PathVariable String filename) {

    try {
        // 验证文件名
        if (filename.contains("..") || filename.contains("/")) {
            return ResponseEntity.badRequest().build();
        }

        // 限制文件扩展名
        String extension = FilenameUtils.getExtension(filename);
        if (!ALLOWED_EXTENSIONS.contains(extension.toLowerCase())) {
            return ResponseEntity.badRequest().build();
        }

        // 规范化路径
        Path uploadPath = Paths.get(uploadDir).normalize();
        Path filePath = uploadPath.resolve(filename).normalize();

        // 验证文件在上传目录内
        if (!filePath.startsWith(uploadPath)) {
            return ResponseEntity.badRequest().build();
        }

        Resource resource = new UrlResource(filePath.toUri());

        return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION,
                "attachment; filename=\"" + resource.getFilename() + "\"")
            .body(resource);
    } catch (Exception e) {
        return ResponseEntity.notFound().build();
    }
}

private static final Set<String> ALLOWED_EXTENSIONS = Set.of(
    "jpg", "jpeg", "png", "gif", "pdf", "txt"
);
```

---

## 🟠 HIGH 漏洞

### 5. 缺少输入验证
**漏洞 ID**: VULN-005
**文件**: `AuthController.java:32`
**严重性**: HIGH
**CVSS 评分**: 7.5 (HIGH)
**CWE 编号**: CWE-20

**漏洞描述**:
用户注册时没有验证输入数据，可能导致数据完整性问题。

**修复建议**:
```java
@PostMapping("/register")
public ResponseEntity<User> register(@Valid @RequestBody User user) {
    // 验证已通过
    User savedUser = userRepository.save(user);
    return ResponseEntity.ok(savedUser);
}

// 或手动验证
@PostMapping("/register")
public ResponseEntity<User> register(@RequestBody User user) {
    List<String> errors = validateUser(user);
    if (!errors.isEmpty()) {
        return ResponseEntity.badRequest()
            .body(new ErrorResponse(errors));
    }

    User savedUser = userRepository.save(user);
    return ResponseEntity.ok(savedUser);
}
```

---

### 6. 缺少权限检查
**漏洞 ID**: VULN-006
**文件**: `AuthController.java:45`
**严重性**: HIGH
**CVSS 评分**: 7.5 (HIGH)
**CWE 编号**: CWE-862

**漏洞描述**:
获取用户信息时没有权限检查，任何人都可以查看任何用户的信息。

**修复建议**:
```java
@GetMapping("/users/{id}")
@PreAuthorize("hasRole('ADMIN') or @userService.isCurrentUser(#id)")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return ResponseEntity.ok(
        userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id))
    );
}
```

---

### 7. 不安全的随机数
**漏洞 ID**: VULN-007
**严重性**: HIGH
**CVSS 评分**: 7.5 (HIGH)
**CWE 编号**: CWE-338

**漏洞描述**:
使用 `java.util.Random` 生成会话 ID 或令牌，不安全。

**修复建议**:
```java
// 使用 SecureRandom
import java.security.SecureRandom;

private static final SecureRandom random = new SecureRandom();

public String generateToken() {
    byte[] bytes = new byte[32];
    random.nextBytes(bytes);
    return Base64.getUrlEncoder().withoutPadding()
        .encodeToString(bytes);
}
```

---

### 8. 存储敏感信息
**漏洞 ID**: VULN-008
**文件**: `PaymentService.java:25`
**严重性**: HIGH
**CVSS 评分**: 8.1 (HIGH)
**CWE 编号**: CWE-312

**漏洞描述**:
将完整的信用卡信息（包括 CVV）存储在数据库中，违反 PCI DSS 标准。

**修复建议**:
```java
public PaymentResult processPayment(Long orderId, PaymentRequest request) {
    Order order = orderRepository.findById(orderId).get();

    // 不要记录完整信用卡信息
    log.info("Processing payment for order {}: {}",
        orderId, maskCardNumber(request.getCardNumber()));

    // 只保存最后 4 位
    order.setCardLastFour(request.getCardNumber()
        .substring(request.getCardNumber().length() - 4));

    // 不要保存 CVV
    orderRepository.save(order);

    // 使用支付网关处理，不保存完整信息
    PaymentResult result = paymentGateway.charge(request);

    return result;
}

private String maskCardNumber(String cardNumber) {
    return "****-****-****-" + cardNumber
        .substring(cardNumber.length() - 4);
}
```

---

## 🟡 MEDIUM 漏洞

### 9. 缺少 CSRF 保护
**漏洞 ID**: VULN-009
**严重性**: MEDIUM
**CVSS 评分**: 6.5 (MEDIUM)
**CWE 编号**: CWE-352

**修复建议**:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            )
            // ...
            .build();
    }
}
```

---

### 10. 缺少速率限制
**漏洞 ID**: VULN-010
**严重性**: MEDIUM
**CVSS 评分**: 6.5 (MEDIUM)
**CWE 编号**: CWE-770

**修复建议**:
```java
// 使用 Spring Boot Starter for Resilience4j
@RateLimiter(name = "loginRateLimiter", fallbackMethod = "loginFallback")
public ResponseEntity<LoginResponse> login(@RequestBody LoginRequest request) {
    // ...
}

public ResponseEntity<LoginResponse> loginFallback(
        LoginRequest request, Exception e) {
    return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
        .body(new ErrorResponse("Too many requests"));
}

// 或使用 Redis 实现自定义速率限制
```

---

### 11. 缺少日志记录
**漏洞 ID**: VULN-011
**严重性**: MEDIUM
**CVSS 评分**: 5.3 (MEDIUM)
**CWE 编号**: CWE-778

**修复建议**:
```java
@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.example.controller..*(..))")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();

        Object result = joinPoint.proceed();

        long endTime = System.currentTimeMillis();

        log.info("{}.{}() executed in {} ms",
            joinPoint.getSignature().getDeclaringTypeName(),
            joinPoint.getSignature().getName(),
            endTime - startTime);

        return result;
    }
}
```

---

### 12. 不安全的文件上传
**漏洞 ID**: VULN-012
**严重性**: MEDIUM
**CVSS 评分**: 5.3 (MEDIUM)
**CWE 编号**: CWE-434

**修复建议**:
```java
@PostMapping("/upload")
public ResponseEntity<String> uploadFile(
        @RequestParam("file") MultipartFile file,
        @RequestAttribute("userId") Long userId) {

    try {
        // 验证文件类型
        String contentType = file.getContentType();
        if (!ALLOWED_CONTENT_TYPES.contains(contentType)) {
            return ResponseEntity.badRequest()
                .body("Invalid file type");
        }

        // 验证文件大小
        if (file.getSize() > MAX_FILE_SIZE) {
            return ResponseEntity.badRequest()
                .body("File too large");
        }

        // 生成安全的文件名
        String filename = generateSafeFilename(
            file.getOriginalFilename(), userId
        );

        Path uploadPath = Paths.get(uploadDir).normalize();
        Path filePath = uploadPath.resolve(filename).normalize();

        // 验证路径在上传目录内
        if (!filePath.startsWith(uploadPath)) {
            return ResponseEntity.badRequest()
                .body("Invalid filename");
        }

        Files.write(filePath, file.getBytes());

        return ResponseEntity.ok("File uploaded successfully");
    } catch (Exception e) {
        log.error("File upload failed", e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body("Upload failed");
    }
}

private String generateSafeFilename(String originalFilename, Long userId) {
    String extension = FilenameUtils.getExtension(originalFilename);
    return String.format("%d_%s.%s",
        userId,
        UUID.randomUUID(),
        extension.toLowerCase()
    );
}

private static final Set<String> ALLOWED_CONTENT_TYPES = Set.of(
    "image/jpeg", "image/png", "image/gif", "application/pdf"
);

private static final long MAX_FILE_SIZE = 10 * 1024 * 1024;  // 10MB
```

---

## 安全评分: 2.5/10

**非常不安全！必须立即修复所有 CRITICAL 和 HIGH 漏洞！**

## 优先修复顺序

### 第一优先级（立即修复）
1. ✅ SQL 注入漏洞 (VULN-001)
2. ✅ 硬编码密钥 (VULN-002)
3. ✅ 敏感信息泄露 (VULN-003)
4. ✅ 路径遍历漏洞 (VULN-004)
5. ✅ 存储敏感信息 (VULN-008)

### 第二优先级（本周内修复）
1. ✅ 缺少输入验证 (VULN-005)
2. ✅ 缺少权限检查 (VULN-006)
3. ✅ 不安全的随机数 (VULN-007)

### 第三优先级（本月内修复）
1. 添加 CSRF 保护 (VULN-009)
2. 添加速率限制 (VULN-010)
3. 添加日志记录 (VULN-011)
4. 修复文件上传 (VULN-012)

## 依赖项安全扫描

| 依赖项 | 当前版本 | 最新版本 | 已知漏洞 |
|--------|----------|----------|----------|
| spring-boot-starter-web | 2.5.0 | 2.7.0 | CVE-2022-22965 (HIGH) |
| spring-security-core | 5.5.0 | 5.7.0 | CVE-2022-22965 (HIGH) |
| commons-fileupload | 1.4 | 1.5 | CVE-2022-23815 (MEDIUM) |
| jackson-databind | 2.12.3 | 2.13.3 | CVE-2022-23105 (MEDIUM) |

**建议**: 运行 `mvn dependency-check:check` 查看完整报告

## 安全最佳实践建议

### 1. 使用安全配置

```properties
# application.properties
# 禁用 Actuator 敏感端点
management.endpoints.enabled-by-default=false
management.endpoint.health.enabled=true
management.endpoint.info.enabled=false
management.endpoint.metrics.enabled=false

# 启用 Spring Security
spring.security.enabled=true

# 配置会话
server.servlet.session.timeout=1800
server.servlet.session.cookie.http-only=true
server.servlet.session.cookie.secure=true

# 配置 HTTPS
server.ssl.enabled=true
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=${SSL_KEY_STORE_PASSWORD}
server.ssl.key-store-type=PKCS12
```

### 2. 安全头

```java
@Configuration
public class SecurityHeadersConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .headers(headers -> headers
                .contentSecurityPolicy(csp -> csp
                    .policyDirectives("default-src 'self'")
                    .policyDirectives("script-src 'self' 'unsafe-inline'")
                )
                .xssProtection(XssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK)
                .frameOptions(HeadersConfigurer.FrameOptionsConfig::deny)
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .preload(true)
                    .maxAgeInSeconds(31536000)
                )
            )
            // ...
            .build();
    }
}
```

### 3. 使用安全扫描工具

```bash
# OWASP Dependency Check
mvn dependency-check:check

# OWASP ZAP（自动扫描）
zap-baseline.py -t https://target-url -r report.html

# SonarQube（代码质量 + 安全）
mvn sonar:sonar -Dsonar.host.url=http://sonarqube-server
```

## 后续步骤

1. 立即修复所有 CRITICAL 漏洞
2. 升级有漏洞的依赖项
3. 实施安全监控和告警
4. 定期进行安全审计
5. 建立安全开发流程（SAST, DAST, 依赖扫描）
