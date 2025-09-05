# Anassassin
读取当前打开的 UserController 代码，为它生成基于 JUnit 5 + @WebMvcTest + MockMvc + Mockito + AssertJ 的测试类，命名采用 Given_When_Then。
覆盖以下请求：

GET /api/users/{id}：存在 → 返回 200 JSON；不存在 → 返回 404

POST /api/users：合法请求体 → 201 Created，带 Location 头和 JSON；非法请求体（name 为空或 email 非法）→ 400
要求：

使用 @MockBean UserService stub 下游

断言 contentType 为 application/json，断言关键 jsonPath

若项目启用 Security，请在类上加 @AutoConfigureMockMvc(addFilters = false)

使用 ObjectMapper 序列化请求体
