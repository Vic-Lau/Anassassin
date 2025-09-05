# Anassassin
请基于当前打开的 UserController 生成一个通用、合规的示例测试类，不包含任何真实密钥、组织信息或外部系统依赖，所有示例数据请使用占位符。
使用 JUnit 5 + @WebMvcTest + MockMvc + Mockito + AssertJ，方法名采用 Given_When_Then 风格。
覆盖用例：

GET /api/users/{id}：存在 → 返回 200，JSON；不存在 → 返回 404

POST /api/users：合法请求体 → 201 Created（带 Location 头）与 JSON；非法请求体（name 为空或 email 非法）→ 400
约束与要求：

使用 @MockBean UserService，并用 Mockito 行为桩设（when/thenReturn、必要时 thenThrow）模拟下游行为

断言 contentType 为 application/json，并断言关键 jsonPath 字段

若项目启用 Security，请在类上添加 @AutoConfigureMockMvc(addFilters = false) 或在用例中使用 @WithMockUser

使用 ObjectMapper 序列化请求体

仅输出通用示例代码，不要包含与具体组织/仓库相关的内容
