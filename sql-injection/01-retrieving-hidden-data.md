# SQL Injection — Retrieving Hidden Data

**Categoria:** SQL Injection  
**Dificuldade:** Apprentice  
**Link:** https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data  
**Data:** 2024-01-15  
**Status:** ✅ Resolvido

---

## O que esse lab testa

A aplicação filtra produtos por categoria usando um parâmetro da URL que vai direto para a query SQL sem sanitização. O objetivo é modificar a query para retornar todos os produtos, incluindo os que estão marcados como não publicados no banco.

---

## Como resolvi

**1.** Observei que a URL seguia o padrão `/filter?category=Gifts`. Testei adicionar `'` ao valor — a aplicação retornou erro 500, confirmando que o input é inserido diretamente no SQL.

**2.** A query original provavelmente é algo como:
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

**3.** Injeti `' OR 1=1--` para tornar o WHERE sempre verdadeiro e comentar o resto da query, incluindo o filtro `released = 1`.

---

## Payload usado

```
/filter?category=Gifts' OR 1=1--
```

---

## O que tentou me bloquear

Nenhum filtro — lab introdutório. A string vai direto para o SQL sem nenhum tratamento.

---

## Como isso aparece em AWS

Em uma aplicação rodando no AWS com RDS como banco de dados, esse tipo de injeção tem impacto maior do que o esperado:

- Se o usuário do banco tiver privilégios excessivos, `UNION SELECT` pode extrair dados de outras tabelas — incluindo tabelas com credenciais ou tokens.
- Em MySQL no RDS, `LOAD_FILE()` pode ler arquivos do sistema se o parâmetro `secure_file_priv` não estiver configurado, expondo variáveis de ambiente com credenciais AWS.
- Se a aplicação busca credenciais do banco em environment variables (padrão em apps legacy no EC2), SQL injection que leva a RCE pode expor essas variáveis.

A mitigação correta é **sempre** usar queries parametrizadas. WAF com regras SQLi é uma segunda camada, nunca a principal.

---

## O que aprendi

O comentário `--` no MySQL precisa de um espaço depois em alguns contextos (`-- `). O `#` também funciona como comentário no MySQL e é mais simples de usar em URLs. Aprendi a testar os dois quando um não funciona.
