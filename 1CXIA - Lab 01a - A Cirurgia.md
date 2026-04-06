# 🧪 Lab 01a: A Cirurgia (Extração de Inferência)

*   **Disciplina:** 1CXIA - MLOps (Semana 09)
*   **Duração:** 45 minutos
*   **Ambiente:** Google Colab
*   **Arquivo Base:** `1CXIA - Lab 01a - caos_experimental.ipynb`
*   **Objetivo:** Isolar a lógica de predição de um ambiente de experimentação e preparar os requisitos para a produção.

---

## 🌐 Acesso e Configuração (Google Colab)

Para iniciar este laboratório, siga os passos abaixo:

1.  **Acesse o Colab:** Vá para [colab.research.google.com](https://colab.research.google.com).
2.  **Upload do Notebook:** 
    *   No menu inicial, escolha a aba **"Upload"**.
    *   Selecione o arquivo `1CXIA - Lab 01a - caos_experimental.ipynb` da pasta da disciplina.
3.  **Configuração de Hardware (Recomendado):**
    *   Para agilizar o download e execução do modelo (Transformers), vá em **Ambiente de execução** > **Alterar o tipo de ambiente de execução**.
    *   Selecione **T4 GPU** (se disponível gratuitamente) para acelerar a inferência.

---

## 🚩 O Cenário de Guerra
O Cientista de Dados da sua squad deixou um Notebook experimental chamado `caos_experimental.ipynb`. Ele contém códigos de treinamento, visualizações de gráficos, instalações de bibliotecas redundantes e, em algum lugar, a lógica que carrega um modelo de **Análise de Sentimentos** e realiza a predição.

**Sua missão:** Realizar uma "cirurgia" no código. Você deve extrair apenas o estritamente necessário para que o modelo funcione como um serviço independente e portátil.

---

## 🛠️ Passo a Passo

### 1. Identificação do "Coração" (Inferência)
Abra o notebook `1CXIA - Lab 01a - caos_experimental.ipynb` e localize as células críticas:
*   **Bibliotecas de Produção:** Quais são **realmente** usadas para a predição? (Ignore `matplotlib`, `seaborn` ou `pandas` se forem usados apenas para análise exploratória/gráficos).
*   **Carregamento do Modelo:** Onde o modelo é inicializado? (Procure pela função `pipeline` da biblioteca `transformers`).
*   **Função de Predição:** Qual bloco de código recebe o texto e devolve o JSON/Dicionário com o resultado?

### 2. A Limpeza (Refatoração)
Crie uma nova célula limpa no final do Notebook. Nela, consolide o código "puro" seguindo esta hierarquia:
1.  **Imports:** Apenas o necessário para rodar o modelo (ex: `transformers`, `torch`).
2.  **Inicialização:** Onde o `classifier` (ou `model`) é carregado do Hugging Face.
3.  **Função de Predição:** Uma função `predict(text)` que encapsula a lógica e retorna um resultado limpo.

### 3. Mapeamento de Dependências (`requirements.txt`)
Para que o ambiente futuro (Killercoda/Docker) seja idêntico ao Colab, você precisa das versões exatas das bibliotecas.
*   Execute em uma célula de código: `!pip freeze | grep -E "transformers|torch"`
*   Copie os nomes e versões (ex: `transformers==4.35.2`) para um bloco de notas. Este será o seu `requirements.txt`.

---

## 🏆 Resultado Esperado
Ao final deste lab, você terá um snippet de código "pronto para produção", similar a este:

```python
# --- CÓDIGO EXTRAÍDO E LIMPO ---
from transformers import pipeline

# Carregando o modelo (BERT para Sentimentos)
classifier = pipeline("sentiment-analysis", model="distilbert-base-uncased-finetuned-sst-2-english")

def predict(text):
    result = classifier(text)[0]
    return f"Sentimento: {result['label']} (Confiança: {result['score']:.4f})"

# Teste de validação local
print(predict("O MLOps na Caixa vai ser um sucesso!"))
```

---

## 🛡️ Dicas de Ouro (SOP)
*   **Cuidado com o 'Lixo':** Se uma biblioteca (ex: `seaborn`) é importada mas não é usada na função `predict`, ela **não** deve ir para a produção. Isso reduz o tamanho do container final.
*   **Pesos do Modelo:** Note que o modelo é baixado na hora. Em produção, discutiremos como "congelar" esses pesos para evitar downloads repetidos (spoiler da Aula 02).
