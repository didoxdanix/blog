---
title: "OPENCODE + OCI GENERATIVE AI"
seoDescription: "This article is about how to use models llm on oracle oci with OPENCODE"
datePublished: 2026-10-02T12:55:15.499Z
cuid: cmuqyu684000006qhhcsyf6va
slug: opencode-oci-generative-ai
cover: https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/2ae82ff0-9121-4849-9fd0-f02eda5e38b5.png
tags: using, using-opencode-with-oci, opencode-and-oci

---

É possível utilizar vários modelos LLM através da Oracle Cloud, modelos bem interessantes por sinal, segue alguns deles:

```bash
openai.gpt-oss-120b
openai.gpt-oss-20b
meta.llama-4-maverick-17b-128e-instruct-fp8
meta.llama-4-scout-17b-16e-instruct
meta.llama-3.3-70b-instruct
xai.grok-4.7
xai.grok-4.6
xai.grok-4.3
xai.grok-4.20-reasoning
xai.grok-4.20-non-reasoning
xai.grok-4.20-multi-agent
```

E se pudéssemos utilizar esses modelos no Claude Code?

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/ab274e79-8495-4049-b4ba-fe0dfc6f3d4f.png align="center")

No Claude Code não sei como podemos utilizar, mas no OpenCode eu sei.

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/bed3bda0-288f-45ec-a43f-01dddb34dfb0.png align="center")

Para quem ainda não conhece, o OpenCode pertence à mesma categoria de ferramentas como Claude Code, Codex e outros agentes de programação. Ele não limita o usuário aos modelos de um único provedor. Assim, é possível escolher e alternar entre diferentes LLMs.

Aí que a OCI Generative AI entra; anteriormente listei os modelos que estão disponíveis sob demanda na OCI e a parte interessante é que podemos fazer a integração com o OpenCode e esses modelos automaticamente estarão disponíveis no OpenCode, onde poderemos realizar as atividades necessárias. Mas antes, precisamos realizar algumas configurações de permissões, geração de chaves e habilitar a região de Chicago no tenancy, pois é lá que está a maioria dos modelos sob demanda da OCI.

**Primeiro**, habilite a região de **Chicago** no seu OCI:

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/e6135a73-0b7d-4dcd-8455-c827bfe607b8.png align="center")

Posteriormente, vamos até:

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/27540bef-2fb4-4a08-a72b-5cf5fb29406e.png align="center")

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/073aeba0-5a2b-426f-822b-053604ed3df0.png align="center")

Criei um compartimento chamado **LLM**; sugiro criar um compartimento próprio para poder limitar melhor as policies lá na frente.

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/1a81546a-54af-437e-9d09-bcf9a89bd9a8.png align="center")

A criação da API KEY é *next, next* e pronto; pode deixar tudo como padrão.

Copie seu **OCID** e a chave da sua **API‑key**.  
OCID começa com `ocid1.generativeaiapikey.oc1.us-chicago-1.....`  
E a chave começa com `sk-...........`

Agora vamos ao **Project** e criar um projeto chamado **OPENCODE**; aqui pode ser tudo padrão, *next, next* e pronto.

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/cd23403f-1c81-48f6-8cae-90fc852a7fe0.png align="center")

Calma, falta pouco; agora vamos para **Policies**.

Lembre‑se do OCID da sua private key; agora que vamos usar, é o **OCID** e não o **keyId**.

Prefixo dela: `[ocid1.generativeaiapikey.oc1.us](http://ocid1.generativeaiapikey.oc1.us)-chicago-1.....`  
E seu **compartment ID**: `ocid1.compartment.oc1..aa....`

Vamos criar a **policy** no compartimento **root**.

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/b928b0ef-eccc-429b-b1f9-2639e09b9457.png align="center")

**Regra 1**:

```bash
allow any-user to manage generative-ai-response in compartment id ocid1.compartment.oc1.XXXXXXXXXXXXXXXXXXXXXXXX where ALL {request.principal.type='generativeaiapikey', request.principal.id='ocid1.generativeaiapikey.oc1.us-chicago-1.XXXXXXXXXXXXXXXXXXXXXXXXXX'}
```

**Regra 2**:

```bash
allow any-user to use generative-ai-chat in compartment id ocid1.compartment.oc1..XXXXXXXXXXXXXXXXXXXXXXXX where ALL {request.principal.type='generativeaiapikey', request.principal.id='ocid1.generativeaiapikey.oc1.us-chicago-1.XXXXXXXXXXXXXXXXXXXXXXXX'}
```

Só relembrando, **API Keys** e **Project** foram criados no compartimento **LLM** e a **Policy** no **root**.

Esse procedimento vale tanto para o OpenCode CLI quanto para o Desktop.

No meu caso estou utilizando macOS, então baixe a versão no site:

[https://opencode.ai/download](https://opencode.ai/download)

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/1469f072-4451-4a1e-95ea-e3f52a390904.png align="center")

Depois de instalado, entre no seguinte diretório:

```bash
/Users/seu_user/.config/opencode
```

E aí você vai criar **2 arquivos**:

```bash
opencode.jsonc
oci-api-key
```

**Conteúdo do arquivo** `opencode.jsonc`:

```bash
{
  "$schema": "https://opencode.ai/config.json",
  "model": "oci/openai.gpt-oss-120b",

  "providers": {
    "oci": {
      "name": "Oracle OCI Chicago",
      "package": "@opencode/ai/providers/openai-compatible",

      "settings": {
        "baseURL": "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/v1",
        "apiKey": "{file:~/.config/opencode/oci-api-key}"
      },

      "models": {
        "openai.gpt-oss-120b": {
          "modelID": "openai.gpt-oss-120b",
          "name": "OCI GPT-OSS 120B",
          "capabilities": {
            "tools": true,
            "input": ["text"],
            "output": ["text"]
          }
        },

        "openai.gpt-oss-20b": {
          "modelID": "openai.gpt-oss-20b",
          "name": "OCI GPT-OSS 20B",
          "capabilities": {
            "tools": true,
            "input": ["text"],
            "output": ["text"]
          }
        }
      }
    }
  },

  "experimental": {
    "policies": [
      {
        "action": "provider.use",
        "resource": "*",
        "effect": "deny"
      },
      {
        "action": "provider.use",
        "resource": "oci",
        "effect": "allow"
      }
    ]
  }
}
```

**Conteúdo do arquivo** `oci-api-key` **(sua chave da API)**:

```bash
sk-...........
```

Pronto, pessoal, posteriormente seu **OpenCode** estará assim:

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/4bdbf536-50c2-4093-a1bf-e3355771f3c5.png align="center")

Também os outros modelos estarão disponíveis:

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/c0aa6f2a-3d05-4104-93a3-88cbb1a8cf83.png align="center")

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/84838c03-67a9-477f-9de7-ea66be514bf4.png align="center")

Pronto, pessoal, agora você utilizará créditos da OCI; cuidados com os gastos rs. Ainda estou estudando uma forma de limitar; se alguém souber, me dê um toque.

Espero que este artigo possa te ajudar em automações futuras. Qualquer coisa, só chamar no [**linkedin**](https://www.linkedin.com/in/diogo-fernandess/) 🙂