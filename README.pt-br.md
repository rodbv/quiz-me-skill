# rodbv/socratic-skills

[🇺🇸 English](README.md)

Skills para Claude Code com foco em aprendizado — recordação ativa, implementação guiada e <a href="https://pt.wikipedia.org/wiki/M%C3%A9todo_socr%C3%A1tico" target="_blank">ciclos de feedback socrático</a>.

## Skills

<table><tr>
<td width="75%">
<p><strong><a href="#quiz-me">quiz-me</a></strong> — Testa seu conhecimento sobre um diff, spec ou plano com perguntas uma a uma, sondando com uma pergunta de acompanhamento antes de corrigir uma resposta errada</p>
<p><strong><a href="#guide-me">guide-me</a></strong> — Guia você na implementação de uma spec passo a passo, verifica seu trabalho e dá dicas — sem nunca escrever código por você</p>
</td>
<td align="center" width="25%">
<img src="socrates-black.svg" alt="Sócrates" width="160" />
</td>
</tr></table>

## Instalação

```bash
npx skills add rodbv/socratic-skills:quiz-me
npx skills add rodbv/socratic-skills:guide-me
```

---

## quiz-me

Testa seu entendimento sobre mudanças de código, specs ou planos — especialmente útil quando se trabalha com código gerado por IA.

Em vez de explicar o código para você, lê o artefato e faz perguntas uma a uma para testar o que você realmente sabe. Em respostas parciais ou erradas, sonda com uma pergunta de acompanhamento antes de corrigir — você raciocina até a resposta em vez de recebê-la pronta. Força a recordação ativa antes de fazer commit, push ou começar a construir.

**Três tipos de artefato:**
- **Código** (padrão) — testa sobre mudanças staged/unstaged antes de um commit
- **Spec** — testa sobre um documento de design antes de implementá-lo
- **Plano** — testa sobre um plano de implementação e decisões arquiteturais

### Por que usar

Ao usar agentes de IA para escrever código, é fácil aceitar mudanças que você não entende completamente. Essa skill cria um mecanismo de verificação: você não pode simplesmente aprovar um diff sem pensar. Ela pergunta *por que* o código funciona, o que quebra ao remover uma linha, o que uma migration faz com dados existentes. Se você não consegue responder, você aprende antes de fazer o deploy.

### Como usar

```
/quiz-me                          # testa sobre mudanças staged+unstaged atuais
/quiz-me last commit              # testa sobre o commit anterior
/quiz-me on the spec              # testa sobre docs/superpowers/specs/ (arquivo mais recente)
/quiz-me on the plan              # testa sobre docs/superpowers/plans/ (arquivo mais recente)
/quiz-me <caminho/para/arquivo.md> # testa sobre uma spec ou plano específico
```

### Exemplo

**Modo spec** (`/quiz-me on the spec`) — mostrando o acompanhamento socrático em uma resposta parcial:

```
Pergunta 1 de 5:
A spec armazena total_price diretamente no model Order em vez de
calcular a partir dos itens. Por quê?

> para não recalcular a cada carregamento de página

Performance é parte da resposta. Mas há um motivo mais importante —
o que acontece com o total calculado se o preço de um produto mudar?

> os pedidos antigos mostrariam o novo preço em vez do que foi cobrado

Exatamente. Desnormalizar o total não é só uma decisão de performance —
é um requisito de correção. O preço no momento da compra é um registro
legal. Um total calculado mudaria silenciosamente conforme os preços
são atualizados.

...

Pontuação final: 4/5 — sólido. Encontrou uma lacuna real na spec na Q2.
```

**Modo código** (`/quiz-me` em um diff que adicionou `select_related`):

```
Pergunta 1 de 4:
Este diff adiciona select_related('author') ao queryset de posts.
Qual problema estava sendo resolvido e como select_related corrige?

> estava fazendo uma query para cada post para obter o autor, agora é uma só

Correto — esse é o problema N+1. select_related faz um JOIN SQL para
que os dados do autor venham na mesma query. Quando você usaria
prefetch_related no lugar?

> quando é um relacionamento many-to-many ou FK reversa

Exatamente. JOIN não funciona bem para esses casos — prefetch_related
executa uma segunda query e faz o join em Python.
```

---

## guide-me

Um tutor de código que guia você na implementação de uma spec ou plano inteiramente por conta própria — sem escrever uma única linha de código por você.

Trabalha a partir de uma spec ou plano de implementação. Divide o trabalho em etapas, apresenta uma de cada vez, aguarda você implementar, depois verifica seu trabalho lendo seu diff e fazendo perguntas. Se você travar, explica conceitos e aponta para a documentação — nunca dá respostas prontas para copiar.

**Modos de teste:**
- **A) TDD** — escreva o teste primeiro, faça passar, depois avança
- **B) Teste depois** — implemente, depois adicione testes antes da próxima etapa
- **C) Misto** (padrão) — TDD para lógica pura, pular em código de configuração/cola
- **D) Sem testes** — pular testes completamente

### Por que usar

A maneira mais rápida de entender algo é construindo você mesmo. Essa skill evita a tentação de pedir ao Claude "só escreve essa parte" — mantém a disciplina para que o aprendizado aconteça. Use quando você tem uma spec pronta e quer implementá-la como uma sessão de prática deliberada.

Combina bem com `quiz-me` (recordação ativa antes de construir) e [`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) (testa suas decisões de design sob pressão).

### Como usar

```
/guide-me                    # guia pela spec/plano mais recente
/guide-me on the spec        # guia por docs/superpowers/specs/ (mais recente)
/guide-me on the plan        # guia por docs/superpowers/plans/ (mais recente)
/guide-me <caminho/arquivo.md> # guia por um arquivo específico
```

### Exemplo

```
> Antes de começar — como você quer lidar com os testes?
> A) TDD — escreva o teste primeiro, faça passar, depois avança
> B) Teste depois — implemente, depois adicione testes antes da próxima etapa
> C) Misto — vou indicar quais etapas se beneficiam de TDD
> D) Sem testes — pular testes completamente

c

Planejei 7 etapas. Vamos começar com a Etapa 1.

─── Etapa 1: Models Pydantic + protocolo DataProvider ───

Implemente WorkItem, DeployEvent e DataProvider em kwando_lib/models.py.
WorkItem precisa de: id (str), started_at e finished_at (datetime | None), item_type (str).
DeployEvent precisa de: deployed_at (datetime), succeeded (bool).
DataProvider é um Protocol com dois métodos — ambos aceitam `since: datetime`
e retornam uma lista. O contrato do protocolo tem uma regra que você precisará
expressar na docstring, não na assinatura de tipo.

Esta etapa tem lógica pesada — escreva um teste para DataProvider primeiro
(dica: teste o que um provider sem dados de deploy deve retornar).

Me avise quando terminar, ou diga 'dica' se travar.

> pronto

[lê git diff]

Parece sólido. Uma pergunta: sua docstring de fetch_deployments diz
"retorna lista vazia se não houver dados" — mas o que garante que um
implementador terceiro vai ler isso? Há uma forma de tornar o contrato
mais enforçável?

> talvez uma classe base abstrata em vez de Protocol?

Vale a pena pensar. O que você ganha com Protocol vs ABC aqui?
Qual deles permite que uma classe existente satisfaça a interface
sem precisar herdar dela?
```

---

## Créditos

Desenvolvido por [@rodbv](https://github.com/rodbv). Inspirado no problema de vibe-coding sem entender o que está sendo feito, e pela skill [grill-me](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) do [@mattpocock](https://github.com/mattpocock).
