# 🧪 Testes de API – ServeRest

Projeto de **testes funcionais de API (caixa-preta)** na [ServeRest](https://serverest.dev), uma API REST gratuita que simula uma loja virtual. Fiz o ciclo de QA inteiro, do planejamento ao relatório final: li o contrato no Swagger, tirei dele as regras de negócio, escrevi os casos de teste, executei tudo no Postman e registrei os resultados e os defeitos encontrados.

> **Resultado do Ciclo 1:** 52 casos executados, **52 passaram (100%)**. Nenhum defeito funcional na API. Registrei 1 defeito de documentação (severidade baixa) e 2 sugestões de melhoria. Todos os critérios de saída foram atingidos.

---

## 📋 Sumário

- [Sobre o sistema testado](#-sobre-o-sistema-testado)
- [Documentos do projeto](#-documentos-do-projeto)
- [Escopo](#-escopo)
- [Regras de negócio mapeadas](#-regras-de-negócio-mapeadas)
- [Estratégia e técnicas de teste](#-estratégia-e-técnicas-de-teste)
- [Casos de teste](#-casos-de-teste)
- [Resultados da execução](#-resultados-da-execução)
- [Defeitos e melhorias](#-defeitos-e-melhorias)
- [Limitações conhecidas](#-limitações-conhecidas)
- [Ferramentas](#-ferramentas)
- [Próximos passos](#-próximos-passos)

---

## 🛒 Sobre o sistema testado

| Item | Detalhe |
|---|---|
| Sistema | ServeRest: API REST de loja virtual |
| Versão | 3.5.0 (conforme Swagger) |
| URL base | `https://serverest.dev` |
| Documentação | [Swagger](https://serverest.dev/swagger?lang=pt-BR) |
| Tipo de teste | Funcional de API (caixa-preta), execução manual |
| Base do plano | Contrato publicado no Swagger |

---

## 📂 Documentos do projeto

| Documento | O que contém |
|---|---|
| **Plano de Teste** | Objetivo, escopo, 17 regras de negócio, estratégia, ambiente, dados de teste, critérios de entrada e saída, riscos |
| **Casos de Teste** | Os 52 casos detalhados, cada um com dados de teste (JSON), passos e resultado esperado |
| **Matriz de Execução** | Planilha com status, endpoint, status code esperado e evidência de cada caso, além dos indicadores do ciclo |
| **Relatório de Bugs** | Defeito e sugestões de melhoria, com passos para reproduzir, resultado esperado × obtido e impacto |
| **Test Summary Report** | Consolidado do ciclo: métricas, resultado por funcionalidade, critérios de saída e recomendação |

---

## 🎯 Escopo

**Dentro do escopo**

| Recurso | Rotas |
|---|---|
| Login | `POST /login` |
| Usuários | `GET`, `POST /usuarios` · `GET`, `PUT`, `DELETE /usuarios/{_id}` |
| Produtos | `GET`, `POST /produtos` · `GET`, `PUT`, `DELETE /produtos/{_id}` |
| Carrinhos | `GET`, `POST /carrinhos` · `GET /carrinhos/{_id}` · `DELETE /carrinhos/concluir-compra` · `DELETE /carrinhos/cancelar-compra` |

Além das rotas, entraram a validação de contrato das respostas, a autenticação por token, a autorização de administrador e o tratamento de corpo inválido (400, 413, 415).

**Fora do escopo**

- Teste de carga e estresse: a própria documentação proíbe isso no ambiente online, que responde 429
- Front-end
- Testes de segurança aprofundados

---

## 📐 Regras de negócio mapeadas

Tirei estas regras da leitura do Swagger. Elas serviram de base para os casos de teste:

| Nº | Regra |
|---|---|
| RN01 | O token do login dura 600 s (10 min). Depois disso, a API responde 401 |
| RN02 | Não é permitido cadastrar ou editar usuário com e-mail já utilizado |
| RN03 | O campo `administrador` aceita apenas `"true"` ou `"false"` (string) |
| RN04 | `PUT` com ID inexistente cria um novo registro (201) em vez de alterar |
| RN05 | Não é permitido excluir usuário que tem carrinho |
| RN06 | Cadastrar, editar e excluir produto exige token de administrador (sem token: 401; usuário comum: 403) |
| RN07 | Não é permitido cadastrar produto com nome já utilizado |
| RN08 | `preco` é inteiro ≥ 1 e `quantidade` é inteiro ≥ 0 |
| RN09 | Não é permitido excluir produto que está em um carrinho |
| RN10 | Cada usuário pode ter apenas 1 carrinho, vinculado ao usuário do token |
| RN11 | Ao cadastrar o carrinho, o estoque dos produtos é reduzido |
| RN12 | O carrinho não aceita produto duplicado, inexistente ou sem estoque suficiente, lista vazia nem quantidade < 1 |
| RN13 | Concluir a compra exclui o carrinho, e o estoque não volta |
| RN14 | Cancelar a compra exclui o carrinho e devolve os produtos ao estoque |
| RN15 | As listagens vêm do registro mais recente para o mais antigo |
| RN16 | Corpo maior que 100 KB ou com mais de 1000 parâmetros retorna 413 |
| RN17 | Toda resposta traz o header `X-ServeRest-Stages` |

---

## 🧭 Estratégia e técnicas de teste

- **Testes positivos (caminho feliz):** o fluxo esperado de cada rota com dados válidos.
- **Testes negativos:** campos ausentes ou vazios, tipos errados, valores fora do limite, IDs inexistentes e JSON malformado.
- **Particionamento de equivalência e análise de valor limite:** `preco` (0 e 1), `quantidade` (-1, 0 e 1) e estoque do carrinho (quantidade igual ao estoque e acima dele).
- **Autenticação e autorização:** sem token, com token inválido, com token expirado e com usuário comum em rota de administrador.
- **Contrato:** status codes, mensagens e estrutura das respostas comparados com o Swagger.
- **Fluxo ponta a ponta (E2E de API):** cadastrar usuário → login → cadastrar produto → criar carrinho → concluir ou cancelar compra → verificar estoque.

**Critério de prioridade**

| Prioridade | O que entra |
|---|---|
| 🔴 Alta | Autenticação, autorização e regras de negócio |
| 🟡 Média | Validações de campo, buscas por ID e contrato |
| 🟢 Baixa | Filtros, ordenação e headers |

**Dados de teste:** e-mails e nomes de produto únicos a cada execução, dois perfis de usuário (administrador e comum), produtos com estoque controlado e limpeza dos dados criados no fim.

---

## 🗂️ Casos de teste

**52 casos** distribuídos assim:

| Funcionalidade | Qtd. | Alta | Média | Baixa |
|---|:---:|:---:|:---:|:---:|
| Login | 8 | 5 | 3 | 0 |
| Usuários | 14 | 7 | 6 | 1 |
| Produtos | 13 | 8 | 4 | 1 |
| Carrinhos | 12 | 10 | 2 | 0 |
| Transversais | 5 | 1 | 2 | 2 |
| **Total** | **52** | **31** | **17** | **4** |

<details>
<summary><strong>Ver a lista completa dos casos</strong></summary>

| ID | Caso de teste | Endpoint | Esperado | Prioridade |
|---|---|---|:---:|:---:|
| CT01 | Login com credenciais válidas | `POST /login` | 200 | Alta |
| CT02 | Login com senha incorreta | `POST /login` | 401 | Alta |
| CT03 | Login com e-mail não cadastrado | `POST /login` | 401 | Alta |
| CT04 | Login sem campos obrigatórios | `POST /login` | 400 | Média |
| CT05 | Login com e-mail em formato inválido | `POST /login` | 400 | Média |
| CT06 | Login com JSON malformado | `POST /login` | 400 | Média |
| CT07 | Uso de token expirado | `POST /produtos` | 401 | Alta |
| CT08 | Uso de token inválido | `POST /produtos` | 401 | Alta |
| CT09 | Cadastrar usuário válido | `POST /usuarios` | 201 | Alta |
| CT10 | Cadastrar usuário com e-mail já usado | `POST /usuarios` | 400 | Alta |
| CT11 | Cadastrar usuário sem campos obrigatórios | `POST /usuarios` | 400 | Média |
| CT12 | Cadastrar usuário com administrador inválido | `POST /usuarios` | 400 | Média |
| CT13 | Listar usuários | `GET /usuarios` | 200 | Média |
| CT14 | Filtrar usuários com e-mail inválido | `GET /usuarios` | 400 | Baixa |
| CT15 | Buscar usuário por ID existente | `GET /usuarios/{_id}` | 200 | Média |
| CT16 | Buscar usuário por ID inexistente | `GET /usuarios/{_id}` | 400 | Média |
| CT17 | Editar usuário existente | `PUT /usuarios/{_id}` | 200 | Alta |
| CT18 | Editar usuário com ID inexistente | `PUT /usuarios/{_id}` | 201 | Alta |
| CT19 | Editar usuário com e-mail de outro usuário | `PUT /usuarios/{_id}` | 400 | Alta |
| CT20 | Excluir usuário sem carrinho | `DELETE /usuarios/{_id}` | 200 | Alta |
| CT21 | Excluir usuário com carrinho | `DELETE /usuarios/{_id}` | 400 | Alta |
| CT22 | Excluir usuário com ID inexistente | `DELETE /usuarios/{_id}` | 200 | Média |
| CT23 | Cadastrar produto como administrador | `POST /produtos` | 201 | Alta |
| CT24 | Cadastrar produto sem token | `POST /produtos` | 401 | Alta |
| CT25 | Cadastrar produto com usuário comum | `POST /produtos` | 403 | Alta |
| CT26 | Cadastrar produto com nome duplicado | `POST /produtos` | 400 | Alta |
| CT27 | Valor limite do preço | `POST /produtos` | 400 / 201 | Média |
| CT28 | Valor limite da quantidade | `POST /produtos` | 400 / 201 | Média |
| CT29 | Preço com tipo inválido | `POST /produtos` | 400 | Média |
| CT30 | Listar e filtrar produtos | `GET /produtos` | 200 / 400 | Baixa |
| CT31 | Buscar produto por ID | `GET /produtos/{_id}` | 200 / 400 | Média |
| CT32 | Editar produto existente | `PUT /produtos/{_id}` | 200 | Alta |
| CT33 | Editar produto com ID inexistente | `PUT /produtos/{_id}` | 201 | Alta |
| CT34 | Excluir produto fora de carrinho | `DELETE /produtos/{_id}` | 200 | Alta |
| CT35 | Excluir produto que está em carrinho | `DELETE /produtos/{_id}` | 400 | Alta |
| CT36 | Cadastrar carrinho e reduzir estoque | `POST /carrinhos` | 201 | Alta |
| CT37 | Cadastrar segundo carrinho | `POST /carrinhos` | 400 | Alta |
| CT38 | Carrinho com produto duplicado | `POST /carrinhos` | 400 | Alta |
| CT39 | Carrinho com produto inexistente | `POST /carrinhos` | 400 | Alta |
| CT40 | Carrinho no limite do estoque | `POST /carrinhos` | 400 / 201 | Alta |
| CT41 | Carrinho com lista vazia ou quantidade 0 | `POST /carrinhos` | 400 | Alta |
| CT42 | Cadastrar carrinho sem token | `POST /carrinhos` | 401 | Alta |
| CT43 | Validar `precoTotal` e `quantidadeTotal` | `POST` e `GET /carrinhos` | 200 | Alta |
| CT44 | Buscar carrinho por ID | `GET /carrinhos/{_id}` | 200 / 400 | Média |
| CT45 | Concluir compra | `DELETE /carrinhos/concluir-compra` | 200 | Alta |
| CT46 | Cancelar compra e devolver estoque | `DELETE /carrinhos/cancelar-compra` | 200 | Alta |
| CT47 | Concluir compra sem carrinho | `DELETE /carrinhos/concluir-compra` | 200 | Média |
| CT48 | Ordenação das listagens | `GET /usuarios`, `/produtos`, `/carrinhos` | 200 | Baixa |
| CT49 | Corpo maior que 100 KB | `POST /usuarios` | 413 | Média |
| CT50 | Charset não suportado | `POST /login` | 415 | Média |
| CT51 | Header `X-ServeRest-Stages` | Todas | 200 / 401 | Baixa |
| CT52 | Fluxo completo de compra (E2E) | Fluxo completo | 201 / 200 | Alta |

</details>

**Exemplo de caso detalhado: CT02, Login com senha incorreta**

```json
POST /login
{
  "email": "MrOk@qa.com",
  "password": "ERRADA"
}
```

**Resultado esperado:** status `401`, mensagem `"Email e/ou senha inválidos"` e nenhum token retornado.

---

## 📊 Resultados da execução

| Métrica | Valor |
|---|---|
| Casos planejados | 52 |
| Executados | 52 (100%) |
| ✅ Passou | 52 (100%) |
| ❌ Falhou | 0 |
| ⛔ Bloqueado | 0 |
| Defeitos registrados | 1 (documentação, severidade baixa) |
| Sugestões de melhoria | 2 |

**Critérios de saída**

| Critério do plano | Resultado | Situação |
|---|---|:---:|
| 100% dos casos de prioridade Alta executados | 31 de 31 | ✅ Atingido |
| Pelo menos 90% de todos os casos executados | 52 de 52 | ✅ Atingido |
| Nenhum defeito crítico ou alto em aberto sem registro | Nenhum | ✅ Atingido |
| Relatório de execução gerado | Test Summary Report | ✅ Atingido |

Os casos CT07, CT14, CT25 e CT42 foram reexecutados até ficarem conforme a especificação. Depois disso, todos os casos de prioridade Alta passaram.

**Conclusão:** a API ServeRest 3.5.0 se comportou conforme a documentação nos 52 cenários executados.

---

## 🐞 Defeitos e melhorias

| ID | Descrição | Tipo | Severidade | Status |
|---|---|---|:---:|---|
| BUG-01 | A descrição do `PUT /produtos/{_id}` no Swagger diz "usuário" em vez de "produto" | Defeito de documentação | Baixa | Aberto |
| MEL-01 | `DELETE` com ID inexistente retorna 200 em vez de 404 | Sugestão de melhoria | – | Para avaliação |
| MEL-02 | Recurso não encontrado retorna 400 em vez de 404 | Sugestão de melhoria | – | Para avaliação |

**BUG-01:** o texto da rota parece copiado da rota de usuários: *"Caso não seja encontrado **usuário** com o ID informado é realizado novo cadastro…"*. A API em si se comporta certo: o CT33 passou e criou um novo produto (201). O problema está só no texto, mas pode confundir quem consome a API.

**MEL-01 e MEL-02:** não são defeitos, porque a API faz o que a documentação descreve. São pontos em que ela foge do padrão REST mais comum. A sugestão é usar `404 Not Found` para recurso inexistente e deixar o `400` para requisição inválida (um ID mal formatado, por exemplo).

---

## ⚠️ Limitações conhecidas

Registro aqui com transparência o que ainda não foi coberto por completo:

- CT30, CT31 e CT44 executaram só o cenário positivo.
- CT47 executou só `concluir-compra`, sem `cancelar-compra`.
- CT52 (E2E) não tem as etapas de consultar o estoque e limpar os dados.
- A execução foi no ambiente online, que é compartilhado: os dados de outras pessoas podem alterar as listagens, e há limite de requisições (429).

---

## 🛠️ Ferramentas

| Ferramenta | Uso |
|---|---|
| **Swagger UI** | Leitura do contrato e testes exploratórios |
| **Postman** | Execução manual (coleção "ServeRest", com a URL na variável `{{url}}`) |
| **Planilha (Excel)** | Matriz de execução e indicadores |
| **Google Drive** | Armazenamento das evidências de cada caso |

---

## 🚀 Próximos passos

- [ ] Reportar o BUG-01 e as melhorias MEL-01 e MEL-02 no [repositório da ServeRest](https://github.com/ServeRest/ServeRest)
- [ ] Automatizar a suíte com **Python + Playwright** (API testing), incluindo validação de schema
- [ ] Gerar relatório com **Allure Report**
- [ ] Rodar a automação na **ServeRest local** (`npx serverest`), para fugir do limite de requisições e dos dados compartilhados
- [ ] Completar a cobertura dos casos listados em [Limitações conhecidas](#-limitações-conhecidas)

---

## 👤 Autor

**MrOk**: QA em formação, com experiência em Suporte Técnico, Help Desk e Service Desk. Estudante de Análise e Desenvolvimento de Sistemas.

<!-- Adicione aqui seus links: LinkedIn, e-mail, portfólio -->
