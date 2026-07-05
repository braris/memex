
# [Multi-Tenant Architecture in Spring Boot (Database per Tenant vs Schema per Tenant)](https://blog.stackademic.com/2%EF%B8%8F%E2%83%A3-multi-tenant-architecture-in-spring-boot-database-per-tenant-vs-schema-per-tenant-85e1b5856828)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*4VGBLtjtL2VYA1t3utxp5Q.png)

Multi-Tenant Architecture in Spring Boot (Database per Tenant vs Schema per Tenant)

## Database per Tenant vs Schema per Tenant (Production Deep Dive)

If you’re building:

- SaaS product
- CRM platform
- ERP system
- White-label solution

You will eventually face this question:

> *How do I isolate tenant data safely and scalably?*

Welcome to **Multi-Tenant Architecture** in Spring Boot.
## What Is Multi-Tenancy?

Multi-tenancy means:

> *A single application serves multiple customers (tenants)  
> While keeping their data isolated.*

Example:

```rb
App: crm.yourapp.com
Tenants:
- Reliance
- TCS
- Infosys
- StartupX
```

Each tenant must:

- See only their data
- Have isolated transactions
- Be protected from other tenants

##  The 3 Common Multi-Tenant Strategies

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*44UP5fu9dNlCMTDwvNhp1g.png)

We’ll focus on the serious ones:

- Schema per Tenant
- Database per Tenant

##  Strategy 1: Database Per Tenant

Each tenant has:

- Separate database
- Separate connection pool
- Full isolation
```rb
Tenant A → db_a
Tenant B → db_b
Tenant C → db_c
```

## Pros

✔ Strongest isolation  
✔ Easy backup/restore per tenant  
✔ Easy migration per tenant  
✔ Enterprise-friendly

##  Cons

- Many DB connections
- Infrastructure overhead
- Harder scaling
- Complex dynamic routing

##  Implementing Database Per Tenant in Spring Boot

We use:

- `AbstractRoutingDataSource`
- Thread-local tenant context

##  Step 1: Tenant Context

```rb
public class TenantContext {
private static final ThreadLocal<String> CURRENT_TENANT = new ThreadLocal<>();
    public static void setTenant(String tenant) {
        CURRENT_TENANT.set(tenant);
    }
    public static String getTenant() {
        return CURRENT_TENANT.get();
    }
    public static void clear() {
        CURRENT_TENANT.remove();
    }
}
```

##  Step 2: Tenant Routing DataSource

```rb
public class MultiTenantDataSource extends AbstractRoutingDataSource {
@Override
    protected Object determineCurrentLookupKey() {
        return TenantContext.getTenant();
    }
}
```

##  Step 3: Configure DataSources

```rb
@Bean
public DataSource dataSource() {
Map<Object, Object> dataSources = new HashMap<>();
    dataSources.put("tenant1", createDataSource("jdbc:postgresql://localhost/tenant1"));
    dataSources.put("tenant2", createDataSource("jdbc:postgresql://localhost/tenant2"));
    MultiTenantDataSource routingDataSource = new MultiTenantDataSource();
    routingDataSource.setTargetDataSources(dataSources);
    return routingDataSource;
}
```

## Step 4: Tenant Interceptor

Extract tenant from header:

```rb
@Component
public class TenantInterceptor implements HandlerInterceptor {
@Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) {
        String tenant = request.getHeader("X-Tenant-ID");
        TenantContext.setTenant(tenant);
        return true;
    }
    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) {
        TenantContext.clear();
    }
}
```

Now each request routes to correct DB.

##  Strategy 2: Schema Per Tenant

Single database.  
Multiple schemas.

```rb
MainDB:
- schema_tenant1
- schema_tenant2
- schema_tenant3
```

All share same DB server.  
Different logical separation.

##  Pros

✔ Less infrastructure  
✔ Shared connection pool  
✔ Easier scaling  
✔ Better resource usage

##  Cons

- Slightly weaker isolation
- Risk if queries leak schema
- Harder per-tenant DB tuning

##  Implementing Schema Per Tenant (Hibernate Approach)

Spring Boot + Hibernate supports built-in schema multi-tenancy.

## Step 1: Enable Multi-Tenancy

```rb
spring.jpa.properties.hibernate.multiTenancy=SCHEMA
spring.jpa.properties.hibernate.tenant_identifier_resolver=com.example.TenantIdentifierResolver
spring.jpa.properties.hibernate.multi_tenant_connection_provider=com.example.SchemaConnectionProvider
```

## Step 2: Tenant Identifier Resolver

```rb
@Component
public class TenantIdentifierResolver
        implements CurrentTenantIdentifierResolver {
@Override
    public String resolveCurrentTenantIdentifier() {
        return TenantContext.getTenant();
    }
    @Override
    public boolean validateExistingCurrentSessions() {
        return true;
    }
}
```

## Step 3: Schema Connection Provider

```rb
@Component
public class SchemaConnectionProvider
        implements MultiTenantConnectionProvider {
@Override
    public Connection getConnection(String tenantIdentifier)
            throws SQLException {
        Connection connection = dataSource.getConnection();
        connection.setSchema(tenantIdentifier);
        return connection;
    }
    @Override
    public Connection getAnyConnection() throws SQLException {
        return dataSource.getConnection();
    }
}
```

Now Hibernate automatically switches schema per tenant.

Clean.  
Elegant.  
Scalable.

##  When to Choose What?

##  Choose Database Per Tenant If:

- Enterprise customers
- Strict compliance (finance/healthcare)
- Need per-tenant scaling
- Large tenants with heavy data

##  Choose Schema Per Tenant If:

- SaaS startup
- Many small tenants
- Cost optimization matters
- Same performance tier

##  Production Pitfalls

##  1. Forgetting to Clear TenantContext

Causes data leakage.

Always clear after request.

##  2. Caching Without Tenant Awareness

Wrong:

```rb
@Cacheable("users")
```

Correct:

```rb
@Cacheable(value = "users", key = "#tenant + #id")
```

Always include tenant in cache key.

##  3. Background Jobs Without Tenant

Scheduled jobs must loop tenants:

```rb
for (String tenant : tenantList) {
    TenantContext.setTenant(tenant);
    processTenantData();
    TenantContext.clear();
}
```

##  Scaling Considerations

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*pAlPC2uI0afzTeqxKzB8Hg.png)

##  Advanced Patterns

✔ Tenant-aware caching  
✔ Tenant-aware logging  
✔ Tenant-aware JWT validation  
✔ Tenant onboarding automation  
✔ Per-tenant feature flags

## 🏁 Final Advice

Multi-tenancy is not just technical.

It affects:

- Billing model
- Scaling strategy
- Compliance
- Infrastructure cost
- DevOps workflow

If building serious SaaS:

Start with schema-per-tenant.  
Move heavy tenants to database-per-tenant later.

That hybrid approach scales beautifully.