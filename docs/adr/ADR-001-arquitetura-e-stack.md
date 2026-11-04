# ADR-001 — Arquitetura e stack do Connexa

- **Situação:** proposta
- **Data:** 2026-11-04
- **Participantes:** Aluno1, Aluno2, Aluno3, Aluno4, Aluno5

## Contexto

O Connexa será construído ao longo de um semestre por uma equipe de 4 a 5 pessoas, com carga de uso modesta. O repositório parte de um modelo da disciplina que já traz a infraestrutura pronta (CI, `./verificar`, configuração do MCP, módulos vazios) e permite à equipe adotar ou adaptar a arquitetura proposta.

Restrições que pesam na decisão:

- **Prazo e tamanho da equipe:** pouco tempo e poucas pessoas, o que torna caro operar vários serviços independentes.
- **Conhecimento prévio:** parte dos integrantes já trabalha com Java e Spring, o que reduz a curva de aprendizado e o risco de a equipe travar.
- **Ecossistema:** o Spring tem documentação extensa e uma comunidade grande, com problemas comuns já relatados e resolvidos na internet. Isso ajuda a equipe e também o agente de IA que escreve código a partir das Tasks.
- **Fronteiras entre territórios:** os módulos (identidade, perfil, grupos, encontros, acompanhamento, painel, materiais) precisam evoluir sem usar os dados uns dos outros.

## Decisão

Adotamos um **monolito modular com fatias verticais + SPA**: um back-end **Java 21 com Spring Boot 4.1.1** (um subpacote de `br.connexa` por módulo, com `internal/` privado, validado pelo Spring Modulith 2.1.1) e um front-end **React 19 com TypeScript, Vite e React Router**, organizado por features. Os dados ficam em **PostgreSQL**, com um schema por módulo, acesso por Spring Data JPA e migrações com Flyway.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Microsserviços | Equipe de 4 a 5 pessoas, um semestre e carga modesta não pagam o custo de operar vários serviços e um broker de mensagens. |
| Arquitetura apenas em camadas | Camadas não impedem que um território use os dados de outro; o monolito modular impõe a fronteira por módulo. |
| Modelo .NET (C# com ASP.NET Core) | A equipe tem mais familiaridade com Java e Spring, e a documentação e a base de problemas já resolvidos pesaram a favor do Spring. |

## Consequências

- **Positivas:**
  - A equipe começa produtiva, porque parte dela já conhece Java e Spring.
  - Erros comuns têm solução publicada e a documentação oficial é completa.
  - As fronteiras entre módulos são verificadas em todo build, e não só por convenção.
  - Um único back-end e uma única SPA simplificam implantação, depuração e testes.
  - Os testes rodam em um PostgreSQL real (`connexa_teste`), igual no Codespaces, no CI e na máquina local.
- **Negativas:**
  - A equipe mantém duas linguagens e dois ecossistemas (Java/Maven e TypeScript/npm).
  - O Spring Boot 4 é recente, e parte do material encontrado na internet cobre a versão 3.x e pode divergir.
  - O monolito é implantado como uma só unidade: um módulo não escala nem é publicado sozinho.
  - As versões ficam fixas em `pom.xml` e `package.json`; incluir, trocar ou atualizar uma dependência exige um novo ADR.
  - Não há autenticação real (Spring Security) na Sprint 1: o estudante é simulado pelo módulo `identidade`.
- **Como verificar:**
  - `ArquiteturaModularTest` (Spring Modulith) falha o build se um módulo usar tipos `internal/` de outro ou houver dependência circular.
  - O lint do front-end bloqueia importações de arquivos internos de outra feature.
  - `./verificar` roda localmente tudo o que o CI roda, e o `main` só aceita integração com os três checks do CI verdes e uma aprovação de pull request.
  - A revisão do pull request confere que mudanças de dependência, módulo novo ou exceção ao `AGENTS.md` vieram acompanhadas de um novo ADR.
