# Lab 1: Agent CLI Single-Turn com Tool-use

Este repositório contém o laboratório prático focado em demonstrar o funcionamento de um **Agente CLI Single-Turn com Tool-use** (Uso de Ferramentas).

> [!NOTE]
> **Foco Educacional:** Este documento e o notebook associado foram criados para ajudar desenvolvedores e estudantes a entenderem como um Large Language Model (LLM) pode interagir com funções locais antes de retornar uma resposta ao usuário.

## Arquitetura e Fluxo de Dados

O laboratório demonstra um fluxo onde o modelo não apenas responde de forma imediata (pure prompt), mas pode "pausar" para delegar o processamento a funções externas.

**Fluxo Básico:**
1. **Usuário:** Envia a consulta (query).
2. **Agente (LLM):** Analisa a query juntamente com o catálogo de ferramentas (JSON schema).
3. **Decisão:** Se a tarefa requer uma ferramenta, o LLM solicita a execução da ferramenta.
4. **Execução Local:** O código intercepta a solicitação e executa a função (ex: calculadora).
5. **Retorno:** O resultado é adicionado ao histórico da conversa.
6. **Resposta Final:** O LLM usa o resultado para elaborar a resposta final ao usuário.

## Pré-requisitos e Setup

Para executar o notebook `lab-001.ipynb`, você precisará de Python 3 instalado, além das bibliotecas listadas abaixo.

1. **Instale as dependências:**
```bash
pip install openai python-dotenv
```

2. **Configuração de Ambiente:**
Crie um arquivo `.env` na raiz do seu projeto e adicione sua chave da API da Nvidia:
```env
NVIDIA_API_KEY=sua_chave_aqui
```

> [!CAUTION]
> **Aviso de Segurança:** Nunca faça o commit do arquivo `.env` no seu controle de versão (como o GitHub). As chaves de API dão acesso a serviços pagos e não devem ser expostas publicamente.

---

## O Passo a Passo do Laboratório

O laboratório é dividido em 3 etapas principais que ilustram a evolução de uma interação estática para um agente dinâmico.

### Etapa 1: Definindo as Ferramentas (Tools)

Aqui definimos duas funções em Python que atuarão como ferramentas para o agente:
- `calculator(expression: str)`: Resolve operações matemáticas básicas.
- `lookup_doc(term: str)`: Simula uma base de conhecimento técnica local.

Em seguida, o código registra essas funções usando um **JSON Schema** (`TOOLS`), que é o formato exigido pela especificação da API da OpenAI para descrever funções para o modelo de linguagem.

> [!WARNING]
> **Segurança:** O uso da função `eval()` na ferramenta `calculator` foi implementado **apenas para fins educacionais**. O comando `eval()` executa strings arbitrárias e é altamente inseguro para ambientes de produção. Em aplicações reais, utilize bibliotecas seguras como `ast.literal_eval` ou analisadores sintáticos específicos.

### Etapa 2: O Loop do Agente

O "cérebro" do laboratório é a função `run_agent`. Ela realiza o loop fundamental do uso de ferramentas:
- Envia as funções (`TOOLS`) e as mensagens para o LLM.
- Verifica se a resposta contém alguma chamada de ferramenta (`msg.tool_calls`).
- Se conter, invoca a ferramenta local correspondente, anexa o resultado no histórico de mensagens e chama o LLM novamente.
- Se não houver mais chamadas, a função interrompe o loop e exibe a resposta final.

### Etapa 3: Comparativo - Pure-Prompt vs Tool-Use

Na última parte do notebook, fazemos as mesmas perguntas utilizando uma abordagem de **Pure Prompt** (onde o modelo responde diretamente usando apenas seus dados de treinamento) e comparamos com os resultados anteriores.

**Resumo Comparativo:**

| Tipo de Query | Pure-prompt | Tool-use | Recomendado |
| :--- | :--- | :--- | :--- |
| **Aritmética com mais de 2 dígitos** | Erro silencioso ocasional | Sempre exato | **Tool-use** |
| **Definição factual** | Depende do *training cutoff* da IA | Ancorado em fonte da verdade | **Tool-use** |
| **Conversa aberta** | Natural, fluida e imediata | *Overhead* desnecessário | **Pure-prompt** |

