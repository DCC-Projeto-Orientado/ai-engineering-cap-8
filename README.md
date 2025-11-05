# Roteiro Prático de Engenharia de Dados - O Assistente JSON

**Projeto Prático: "O Assistente JSON"** O objetivo deste roteiro é criar um dataset de alta qualidade para finetunar um modelo de linguagem. A tarefa do modelo será atuar como um assistente de e-commerce que **sempre e somente** responde com um JSON estruturado, extraindo as informações da pergunta do usuário.

## Seção 1: Curadoria de Dados

Esta seção introduz a filosofia da "IA Centrada em Dados" (Data-Centric AI), que defende que a melhoria da performance de um modelo vem mais da qualidade dos dados do que de alterações na arquitetura do modelo. A curadoria é o processo de planejamento e design do dataset, que se baseia em três pilares fundamentais:

- **Qualidade:** Refere-se a quão bons são os exemplos individuais. O livro define dados de alta qualidade como sendo relevantes, alinhados aos requisitos da tarefa, consistentes, corretamente formatados, únicos e em conformidade com políticas de privacidade. Um pequeno volume de dados de alta qualidade pode superar um grande volume de dados "ruidosos".

- **Cobertura:** Também chamada de diversidade de dados, garante que o dataset represente a variedade de problemas que o modelo encontrará no mundo real. Um dataset com boa cobertura inclui diferentes tópicos, estilos de linguagem e tipos de tarefa. A equipe do Llama 3, por exemplo, descobriu que aumentar a diversidade de tarefas era crucial para a performance.

- **Quantidade:** Define quantos exemplos são necessários. A quantidade ideal depende da complexidade da tarefa, da qualidade do modelo base e da técnica de finetuning utilizada (PEFT, como LoRA, exige menos dados que um finetuning completo)

### Caso de uso: O "Blueprint" do Dataset

**Objetivo:** Planejar o dataset "Assistente JSON" antes de criar qualquer dado. 

O que voce deve fazer? Crie um arquivo `blueprint.txt` respondendo as perguntas a seguir.

1. **Defina a Qualidade:** Escreva uma "Diretriz de Anotação" para o projeto. O que torna uma amostra perfeita?

    - Exemplo: "A instrução deve ser uma pergunta de cliente com 5 a 25 palavras. A resposta deve ser um JSON válido, sem nenhum texto adicional. O JSON deve conter a chave ``tipo_consulta`` e chaves para cada entidade encontrada na pergunta (ex: `produto`, `tamanho`, `cor`)."

2. **Defina a Cobertura:** Liste 5 "intenções" de cliente que o dataset precisa cobrir.

    - Exemplo: 1. Consultar preço; 2. Verificar estoque; 3. Calcular frete; 4. Perguntar sobre material do produto; 5. Iniciar uma devolução.

3. **Defina a Quantidade:** Defina uma meta inicial de 10 exemplos para um finetuning com LoRA.


## Seção 2: Aquisição e Anotação de Dados

Esta seção trata de como obter os dados. A fonte mais valiosa é o "data flywheel" , dados gerados pela própria aplicação, pois refletem o uso real. Na ausência disso, o processo começa com a busca em datasets públicos ou, mais comumente para tarefas específicas, com a anotação manual.

O livro destaca que a anotação para modelos de linguagem é complexa, exigindo pensamento crítico e, às vezes, conhecimento de domínio . O processo de criar diretrizes claras e garantir que os anotadores as sigam de forma consistente é um dos maiores desafios da engenharia de dataset. Este pequeno conjunto de dados anotados manualmente servirá como "seed" (semente) para a próxima fase de síntese.

### Caso de uso: Criando o "Seed Dataset"

**Objetivo:** Criar manualmente um pequeno conjunto de dados ("seed") de altíssima qualidade.

O que deve fazer? Crie um arquivo `seed_dataset.txt` respondendo as perguntas a seguir.

1. **Anotação Manual:** Com base nas diretrizes de anotação da Etapa 1, você deve criar 5 exemplos de pares (instrução, resposta).

2. **Foco na Cobertura:** Garanta que seus 5 exemplos cubram todas as intenções de cliente definidas anteriormente.

3. **Exemplo de Amostra:**

    - Instrução: "Queria saber se tem a camisa polo azul no tamanho M em estoque."

    - Resposta: `{"tipo_consulta": "estoque", "produto": "camisa polo", "cor": "azul", "tamanho": "M"}`


## Seção 3: Augmentação e Síntese de Dados

Esta é a seção mais densa, focada em gerar dados programaticamente.

- **Por que Sintetizar?:** A síntese de dados permite escalar a quantidade, melhorar a cobertura de casos raros, mitigar preocupações com privacidade e realizar a destilação de modelos (ensinar um modelo pequeno a imitar um grande) .

- **Técnicas Tradicionais:** Incluem métodos baseados em regras, como o uso de templates (ex: gerar dados de transações de cartão de crédito) e simulações (ex: simular um ambiente para treinar um robô).

- **Síntese com IA:** A abordagem mais moderna. Pode-se usar um modelo forte (ex: GPT-4) para gerar novos exemplos. O livro detalha a "síntese de dados de instrução", onde a IA pode gerar novas instruções, novas respostas, ou ambos, a partir de um conjunto "semente" . O projeto Alpaca é citado como um exemplo clássico.

- **Verificação de Dados:** É um passo crítico. Dados sintéticos podem conter alucinações ou ser de baixa qualidade. O livro mostra o exemplo do Llama 3, onde o código gerado sinteticamente passava por testes de unidade para garantir a correção antes de ser usado no treino.

- **Limitações:** O capítulo adverte sobre o risco do "colapso do modelo", onde treinar repetidamente em dados gerados por IA pode fazer o modelo "esquecer" a distribuição de dados reais e degradar sua performance. A solução sugerida é sempre misturar dados sintéticos com dados reais.

### Caso de uso: Escalando o Dataset com IA

**Objetivo:** Usar um modelo de IA forte (como Gemini, GPT-4, etc.) para escalar seus 5 exemplos manuais para 20, e depois verificar a qualidade.

1. **Gerar Instruções:** Use seus 5 exemplos "semente". Crie um prompt para um modelo de IA forte e peça para ele gerar 20 novas instruções (perguntas de clientes) variadas.

    -  *Prompt Exemplo:* "Você é um especialista em engenharia de dataset. Baseado nestes 5 exemplos de perguntas de clientes, gere 20 novas perguntas, mantendo o mesmo estilo e cobrindo as mesmas intenções (preço, estoque, frete, etc.). Exemplos: [Cole seus 5 exemplos aqui]"

2. **Gerar Respostas:** Para cada uma das 20 novas instruções, peça ao modelo de IA para gerar a resposta em JSON, seguindo suas diretrizes. Salve cada JSON deste.

    - *Prompt Exemplo:* "Para a pergunta '[instrução gerada]', gere apenas a resposta em formato JSON, seguindo esta diretriz: [Cole sua diretriz da Etapa 1 aqui]. Responda apenas com o JSON."

3. **Verificar:** Esta é a parte mais importante. Revise manualmente as 20 amostras sintéticas.

    - A IA seguiu o formato JSON perfeitamente? (Verificação de formato).

    - A IA extraiu as entidades corretas? (Verificação de relevância/alinhamento).

    - A IA "alucinou" ou inventou um `tipo_consulta` que não existe nas suas diretrizes? (Verificação de qualidade).

    - Corrija ou descarte as amostras ruins.

Ao final, armazene todos os JSONS restantes em uma pasta criado por você chamada `respostas`.

## Seção 4: Processamento de Dados

Com o dataset bruto em mãos, a etapa final é o processamento para garantir que ele esteja limpo, otimizado e no formato correto para o treinamento.

- **Inspecionar:** O primeiro passo é explorar os dados. O livro sugere plotar distribuições (ex: comprimento das respostas) e inspecionar manualmente os dados, pois "olhar para os dados por 15 minutos geralmente dá algum insight que pode economizar horas de dor de cabeça".

- **Deduplicar:** Remover exemplos duplicados é essencial para evitar que o modelo se torne enviesado ou memorize respostas. A deduplicação pode ser feita por correspondência exata ou por similaridade semântica.

- **Limpar e Filtrar:** Envolve remover "lixo" como tags HTML, informações pessoais (PII), e filtrar dados de baixa qualidade (ex: respostas muito curtas ou vazias).

- **Formatar:** Talvez o passo mais importante na prática. Cada modelo espera um formato de entrada específico, definido pelo seu "template de chat" (ex: o Llama 3 usa tokens especiais como `<|start_header_id|>`). Se os dados não seguirem esse template, o finetuning pode falhar silenciosamente ou produzir um modelo de baixa qualidade .

### Caso de uso: O Script de Limpeza

**Objetivo:** Escrever um script Python que executa os 4 passos do processamento de dados.

1. **Carregue e Inspecione:** Use a biblioteca `datasets` da Hugging Face ou `pandas` para carregar seu dataset de ~20 amostras.

2. **Deduplique:** Use a função de deduplicação da biblioteca para remover quaisquer instruções idênticas. Imprima o número de amostras restantes.

3. **Limpe e Filtre:** Escreva uma função de filtro que remova amostras onde a resposta não é um JSON válido (pode usar um `try-except` com `json.loads`).

4. **Formate:** (Esta é a parte mais importante!)

    - Escolha um modelo base (ex: Mistral 7B).

    - Encontre o "chat template" oficial dele.

    - Escreva uma função em Python que transforme seu par `instrucao` (usuário) e `resposta` (assistente) nesse template.

```python
# Exemplo de função de formatação para o template do Llama 3

def formatar_para_llama3(amostra):
    instrucao = amostra['instrucao']
    resposta = amostra['resposta']

    # Template do Llama 3
    texto_formatado = (
        "<|begin_of_text|><|start_header_id|>user<|end_header_id|>\n\n"
        f"{instrucao}<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n"
        f"{resposta}<|eot_id|>"
    )
    return {"texto_formatado": texto_formatado}

# Aplique a função ao seu dataset (ex: usando .map() da biblioteca 'datasets')
# dataset_formatado = meu_dataset.map(formatar_para_llama3)
```

5. **Salve:** Salve o `dataset_formatado` em um arquivo `.jsonl`. Você pode usar esse dataset em um script de finetuning.


## Envie suas conclusões

Ao final de todos os passos, envie um Pull Request, contendo todos os arquivos solicitados durante as seções.

No título do Pull Request coloque o número de matrícula e na descrição comente sobre suas interpretações e análises encontradas ao longo da atividade.

### Referências
Caso tenha interesse em se aprofundar no assunto, consulte o capítulo 8 - Dataset Engineering do livro AI Engineering Building Applications with Foundation Models.