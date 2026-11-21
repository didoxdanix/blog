---
title: "Grok 4.7 OCI, quanto custa usar um dia inteiro?"
datePublished: 2026-11-21T01:26:07.451Z
cuid: cmwpq8j93000002lfeu63gkkj
slug: grok-4-7-oci-quanto-custa-usar-um-dia-inteiro
cover: https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/bbb0e018-4cad-48cc-b51e-5574885bc433.png

---

No post anterior eu demonstrei como configurar o OPENCODE utilizando as LLMs sob demanda que estão na Oracle Cloud. Utilizei durante um longo período, para então publicar algo aqui, sem sombra de dúvidas, de todas as LLMs que a OCI oferece, a única "decente" é a Grok, somente com ela consegui trabalhos iguais ou até mesmo superiores aos do Claude Code ou Codex.

Antes de iniciar, gostaria de compartilhar a minha opinião pessoal sobre o Grok, é uma LLM bem interessante para raciocínios complexos e compreender problemas bem profundamente. Em um determinado projeto em C17, ele foi o único que achou o problema de gargalo e conseguiu resolver, e por isso comecei a compreender por que ele foi tão caro durante meus usos.

Mas vamos lá, eu usei ele durante um dia inteiro, e um dia eu usei ele em short context e depois em long context, na OCI ele disponibiliza essas 2 formas e você configura isso no seu OpenCode e automaticamente ele decide se vai fazer o compact com 200k ou 500k de contexto, e te adianto que me arrependi amargamente de usar o long context rsrs.

Em ambos os dias, utilizei o OPENCODE com Grok por volta de 9 horas seguidas, eis que segue o resultado da conta, primeiro dia:

Short Context:

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/3a940a7e-f646-4792-9749-44a3bce399aa.png align="center")

Long Context:

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/b25f2b01-ef25-41d8-b87f-50ad1c81f96e.png align="center")

Podemos ver que a minha preguiça de digitar /compact me cobrou caro 😂

A maioria das operações ocorre em cached input tokens, então vamos ver qual é a diferença de acordo com a tabela de preços da Oracle: <https://www.oracle.com/cloud/price-list/>

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/3f988ff9-a4bc-45e5-91f6-023c843b4d4a.png align="center")

![](https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/7f515007-d5e5-4a56-b5fc-56403e0ed68e.png align="center")

Se short é 0,50 e long custa 1,00, não era para ter dado R$ 346,00 no segundo dia?

Aqui temos 2 pontos:

1 - Quanto maior sua janela de contexto, maior o envio de sua requisição e isso naturalmente pode aumentar seu consumo de tokens, sendo bem "grosseiro" no exemplo, ignorando conceitos de cached, input e output tokens, toda vez que você faz uma requisição ou pergunta, ele reenvia, e por isso o aumento de tokens é mais que o dobro.

2 - Mesmo no segundo dia eu ter trabalhado por um período parecido com o primeiro, isso não significa que o processamento seja igual ao anterior, e meu consumo pode ter sido maior do que o primeiro dia, mas se extrapolarmos o consumo de long para short, minha conta do outro dia estaria por volta de R$ 221,50 e não R$ 443,00. Resumo, Long Context é caro no Grok utilizando OCI, só use se for necessário.

Conclusão

Nada é mais barato que o plano de R$ 100,00 que utilizamos com Codex ou Claude Code, caso você tenha esses planos, a utilização dessas LLMs sobe demanda na OCI/Grok só é viável economicamente quando você realmente precisa resolver um problema. Isso estou falando de coding, para tarefas simples de text e talk tem o gpt-oss120b que é bem em conta e isso vale a pena.