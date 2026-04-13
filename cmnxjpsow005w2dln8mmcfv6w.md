---
title: "Utilizando Codex (GPT) ou Antigravity para conectar no Oracle.
"
datePublished: 2026-04-13T18:48:23.939Z
cuid: cmnxjpsow005w2dln8mmcfv6w
slug: utilizando-codex-gpt-ou-antigravity-para-conectar-no-oracle
cover: https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/b62aa941-36f3-4817-90f4-8b4a34090003.png

---

Recentemente comecei a ter uma curiosidade sobre agents da famila "CODE", sendo eles Antigravity, Claude Code e o Codex da Openai.

Tem toda a questão de MCP e etc, mas uma pergunta pairou pela cabeça...

Da pra usar o sqlcl para conectar no banco através desses agentes e pedir pra ele fazer as coias que eu quero?

Bem, pareceu um pouco maluquice mas decide testar, e pra minha surpresa deu certo rs.

Então vamos la! Vou demostrar como eu fiz.

Pre reqs:

Vamos baixar o sqlcl  
[https://www.oracle.com/database/sqldeveloper/technologies/sqlcl/download/](https://www.oracle.com/database/sqldeveloper/technologies/sqlcl/download/)

Se sua maquina tiver o java, ja vai executar de cara, se não tiver ele vai pedir pra você instalar...

Vamos testar...

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/b4843104-3b8f-4973-bdf5-b227e6bed72e.png align="center")

Pronto, sqlcl instalado e testado vamos criar um projeto no CODEX.

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/b27c878c-010e-4423-9f70-c272561a1a90.png align="center")

No meu caso, eu criei uma pasta em Desktop com o nome TESTE.

Dentro da pasta crie um arquivo connect.txt com as seguintes instrucoes. (Adapte para seu ambiente).

```bash
- utilize o sqlcl que esta na pasta C:\Users\DiogoFernandes\Downloads\sqlcl-latest\sqlcl\bin\sql.exe
- Salve todos os scripts executados na pasta do projeto.

SGBD:ORACLE
Usuario: CODEX
Senha: SUA_SENHA
IP: TESTE-VM
Service: TESTE
Porta: 1521
```

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/f4dbd840-9cf4-4ff7-a527-bcddff32a417.png align="center")

Agora vamos pedir ele pra validar a conexão:

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/f21e4c6b-42a6-4151-b7b2-b8f89fe84ea0.png align="center")

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/74213d23-a399-4206-abc8-0d10a773824c.png align="center")

Para testar se tudo certo certo, vamos pedir pra criar alguns objetos.

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/1f321d8b-2666-4d6c-9bc4-268f50a80874.png align="center")

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/86c5b6f3-691c-47a3-9bf4-ea4e2a4968d2.png align="center")

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/60b03f2f-3f49-4985-9e98-986751e29d0f.png align="center")

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/f0508427-b372-486b-ba47-3bbb437595b8.png align="center")

Pronto, tabelas criadas e populadas via "A.I". Tudo lindo né, só que não...

Olhe um trecho do "pensamento da "A.I" durante meus testes..

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/a2506d83-123b-4b0a-91db-3b0e48758fd5.png align="center")

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/b187c209-0b13-43ed-993c-ce72159a0e78.png align="center")

Antes de criar as tabelas, "ele" fez uma verificação para ver se as tabelas já existiam. Se existissem, ele iria dropar e recriar a tabela. Em caso de erro de digitação meu, ou se eu colocasse o nome de uma tabela que já existisse, nesse momento ele iria dropar a tabela e recriá-la com o nome que sugeri. Então, se for usar "A.I" em bases de testes ou homologação, recomendo sempre dar contexto ou habilidades sobre a situação. No arquivo connect, você pode colocar coisas do tipo:

```bash
- utilize o sqlcl que esta na pasta 

C:\Users\DiogoFernandes\Downloads\sqlcl-latest\sqlcl\bin\sql.exe

- Salve todos os scripts executados na pasta do projeto.
- Nunca recrie nada sem autorizacao.
- Nunca drope nada sem autorizacao.
- Em caso de conflito ou objeto existente, sempre me pergunte.
- nao use create or replace somente create.
```

Estou estudando a parte de skills desse agente para trazer conteúdos novos para vocês também. Em tese, é muito comum "replace" de coisas nesses agentes por serem utilizados mais em programação. É necessário informar detalhadamente para ele que você está trabalhando com banco de dados, caso deseje utilizar isso no seu dia a dia.

O intuito deste artigo é mostrar uma possível funcionalidade e nenhuma recomendação.

Utilizando a mesma abordagem, obtive sucesso com o Antigravity da Google e consegui conectar e reproduzir as mesmas coisas:

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/e1c319e7-d051-47fc-8524-ae1fc5e31dc1.png align="center")

Coisas que consegui durantes meus testes.

*   Criacao de objetos.
    
*   Backup de objetos.
    
*   Criação de tablespaces.
    
*   Transformar tabelas nao particionadas para particionadas.
    
*   Criacao de resource manager ( esse ela deu algumas derrapadas)
    
*   Indentifição rapida de indices que precisava de rebuild.
    

  
Ps: Não use isso em produção, nao por enquanto ;)

Espero que este artigo possa te ajudar em automações futuras. Qualquer coisa, só chamar no [**linkedin**](https://www.linkedin.com/in/diogo-fernandess/) 🙂