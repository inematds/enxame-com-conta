# enxame-com-conta

[![A conta do enxame](guia/assets/banner.jpg)](https://inematds.github.io/enxame-com-conta/guia/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## O que é

Um enxame de agentes é quando várias IAs trabalham juntas na mesma tarefa, cada uma num pedaço. Funciona, mas
gasta muito mais: a própria Anthropic mediu cerca de 15 vezes mais texto processado que um chat comum. A Ficha da
Conta é uma folha curta para quem pensa em usar enxames no trabalho: ela pergunta se a tarefa pede isso, compara o
custo com um agente só e põe um teto e um freio antes de rodar. O kit traz a ficha e dois scripts em Python que
fazem a conta e seguram o gasto. Para entender, é só ler; para usar o kit, precisa de Python 3.

## 📖 Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/enxame-com-conta/guia/**

---

**Você viu o enxame de agentes viral e pensou em montar um. A ideia é genial. A conta, não.**

Este kit mostra quanto um enxame vai custar **antes** de rodar, compara com um agente só fazendo a mesma
tarefa e põe um freio para não bater no limite mil vezes. Só Python (biblioteca padrão), sem chamar modelo
nem API.

> A própria Anthropic mediu: agentes gastam cerca de 4× mais tokens que um chat, e sistemas multiagente cerca de
> **15×**. E avisa: só compensam quando o valor da tarefa paga o desempenho a mais. (Fontes no fim.)

## O que tem aqui

| Peça | Para quê |
|---|---|
| `FICHA-CONTA.md` | 4 perguntas antes de montar o enxame + orçamento e freio |
| `conta.py estimar` | estima tokens e custo do enxame × um agente, e onde o custo vai (trabalho, coordenação, revisão, retrabalho) |
| `conta.py comparar` | com execuções reais da mesma tarefa, calcula o **CRU — custo por resultado útil** — e dá o veredito |
| `trava.py` | freio para qualquer chamada de agente: para no 1º limite, tem orçamento de chamadas e para depois de N falhas |
| `casos/lote-135.md` | caso real INEMA: 12.619 recusas por limite em ~34 h por falta de trava |

## O processo, do início ao fim

Quem faz: 🧠 você (decisão) · ⚙️ script deste kit · 🤖 seu agente (`claude -p`, `codex exec` ou outro)

1. 🧠 **Responda a parte 1 da `FICHA-CONTA.md`.** Menos de duas marcadas → um agente só e pare aqui.
2. 🧠 **Descreva o enxame em `plano.json`** (copie `exemplos/plano-enxame.json`): agentes, rodadas, tamanho do
   contexto, conversa entre eles, revisores, retrabalho e o preço do seu modelo por milhão de tokens.
3. ⚙️ **Estime:** `python3 conta.py estimar plano.json` → custo dos dois jeitos, multiplicador e onde o custo vai.
4. 🧠 **Decida o orçamento** (parte 3 da ficha): máximo de agentes, de chamadas e de falhas seguidas.
5. 🤖 **Rode cada chamada de agente pela trava**, todas com o mesmo arquivo de estado:
   `python3 trava.py rodar --estado .trava.json --max-chamadas 40 --max-falhas 3 -- claude -p "…"`
   Saídas: `75` bloqueada por limite (nada foi chamado), `76` orçamento esgotado, `77` falhas seguidas.
6. ⚙️ **Acompanhe:** `python3 trava.py status`. Depois de olhar o que houve: `python3 trava.py liberar`.
7. 🧠 **Anote as execuções reais** da mesma tarefa, com um agente e com o enxame, em `execucoes.json`
   (`custo`, `aprovada`, `retrabalho`, `minutos`).
8. ⚙️ **Compare:** `python3 conta.py comparar execucoes.json` → CRU de cada um e o veredito.
9. 🧠 **Decida pelo veredito:** o enxame só fica se o resultado a mais (ou o prazo) paga o custo a mais.

## Exemplo (números ilustrativos)

```
$ python3 conta.py estimar exemplos/plano-enxame.json
  enxame:     custo 3.42   um agente: custo 0.38   multiplicador_tokens: 8.3
  onde_vai:   trabalho 63%  coordenação 9%  revisão 5%  retrabalho 23%

$ python3 conta.py comparar exemplos/execucoes.json
  veredito: o enxame custa 5.2× mais por entrega aprovada e acerta 1.5× mais: só vale se cada entrega
  aprovada valer mais que 5.2× o custo de um agente só (mas foi 2.5× mais rápido: decida se o prazo vale isso)
```

O modelo de custo do `estimar` é simples e de propósito: cada agente relê o contexto e tudo o que recebeu a
cada rodada, o coordenador lê as entregas, o revisor lê tudo, e o retrabalho multiplica. Serve para comparar
arquiteturas, não para prever a fatura ao centavo. A conta de verdade vem do passo 8.

## Testar

```
python3 -m unittest discover -s tests -v
```

## Fontes

- Anthropic, *How we built our multi-agent research system* (jun/2025):
  https://www.anthropic.com/engineering/multi-agent-research-system — "multi-agent systems use about 15× more
  tokens than chats"; "multi-agent systems require tasks where the value of the task is high enough to pay for
  the increased performance".
- Caso lote-135: registro interno INEMA, 15–17/11/2026 (`casos/lote-135.md`).

## Relacionado

- [enxame-antigravity](https://inematds.github.io/enxame-antigravity/guia/): a Ficha do Enxame para o /swarm do
  Google (dono por pedaço, quem confere, quando parar).

Licença MIT · INEMA.CLUB — conteúdo aberto e gratuito.
