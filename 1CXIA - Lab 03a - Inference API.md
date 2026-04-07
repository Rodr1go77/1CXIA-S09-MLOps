# 🚀 Lab 03a: A "Inference API" (Expondo o Modelo com FastAPI)

*   **Disciplina:** 1CXIA - MLOps (Semana 09)
*   **Duração:** 50 minutos
*   **Ambiente:** Killercoda (Dentro da pasta `caixa_sentiment_analysis`)
*   **Acesso:** [Killercoda Play (Ubuntu)](https://killercoda.com/playgrounds/scenario/ubuntu)
*   **Objetivo:** Transformar o script de predição isolado em uma API REST profissional, pronta para ser consumida por outros sistemas da Caixa.

---

## 🚩 O Cenário de Guerra
Ontem, você provou que o modelo "sobrevive" fora do Notebook. Hoje, o desafio é torná-lo útil para o mundo real. Um modelo que roda apenas no terminal é um "cérebro na jarra". 

**Sua missão:** Criar uma API de alta performance que receba textos de feedback de clientes e retorne a análise de sentimento em JSON. Vamos usar o **FastAPI**, que é o padrão atual para IA devido à sua velocidade e tipagem forte.

---

## 🛠️ Passo a Passo

### 1. Atualização do Ambiente (Dependências)
A API precisa de novos motores. Vamos limpar o cache do `pip` primeiro para ganhar espaço no Killercoda e depois instalar o `fastapi` e o `uvicorn`.

```bash
# Certifique-se de estar com o venv ativo!
# Limpa o cache do pip (Sobrevivência no Killercoda)
pip cache purge

# Instalando dependências
pip install fastapi uvicorn

# Atualize seu arquivo de dependências para manter o projeto reprodutível
pip freeze > requirements.txt
```

### 2. Criação da API (`src/main.py`)
Vamos criar o arquivo `src/main.py`. Note que carregamos o modelo **globalmente** (fora das funções de rota) para que ele seja carregado apenas uma vez na inicialização da API, economizando memória e tempo.

```bash
cat <<EOF > src/main.py
from fastapi import FastAPI
from pydantic import BaseModel
from transformers import pipeline
import time

# 1. Inicialização da API
app = FastAPI(
    title="Caixa Sentiment Analysis API",
    description="API para análise de sentimento de feedbacks utilizando BERT Multilingual",
    version="1.0.0"
)

# 2. Carregamento do Modelo (Singleton Pattern)
print("🕒 Carregando modelo BERT... Isso pode levar alguns segundos.")
classifier = pipeline(
    "sentiment-analysis", 
    model="nlptown/bert-base-multilingual-uncased-sentiment"
)
print("✅ Modelo carregado com sucesso!")

# 3. Definição do Esquema de Entrada (Input Validation)
class FeedbackRequest(BaseModel):
    text: str

# 4. Endpoints
@app.get("/")
def read_root():
    return {"message": "Caixa Sentiment API is running!"}

@app.get("/health")
def health_check():
    return {"status": "healthy", "model": "bert-base-multilingual-uncased"}

@app.post("/predict")
def predict_sentiment(request: FeedbackRequest):
    start_time = time.time()
    
    # Realiza a predição
    prediction = classifier(request.text)[0]
    
    latency = time.time() - start_time
    
    return {
        "text": request.text,
        "label": prediction['label'],
        "score": round(prediction['score'], 4),
        "latency_seconds": round(latency, 4)
    }
EOF
```

### 3. Rodando o Servidor
Agora, vamos subir a API usando o `uvicorn`. O parâmetro `--reload` é útil para desenvolvimento (ele reinicia a API se você mudar o código).

```bash
# Rodando na porta 8000
uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload
```

### 4. O "Poder" do FastAPI (Swagger)
Uma das maiores vantagens para times sêniores é a documentação automática. No Killercoda (ou no seu navegador se estivesse local), acesse:
`http://seu-ip:8000/docs`

*   **Teste o endpoint `/predict`** enviando um JSON como: `{"text": "O App da Caixa está muito lento hoje."}`.
*   **Observe o JSON de retorno:** Ele contém a resposta do modelo e a métrica de latência.

### 5. Versionando a API
Abra outro terminal (mantendo a API rodando) e registre essa evolução:

```bash
git add src/main.py requirements.txt
git commit -m "feat: implement inference API with FastAPI and Swagger docs"
```

---

## 🏆 Resultado Esperado
Uma API funcional, documentada automaticamente, capaz de processar requisições POST e retornar predições em milissegundos (após o carregamento inicial).

---

## 🛡️ Notas de MLOps (Design Patterns)
*   **Input Validation (Pydantic):** Usamos o `BaseModel` para garantir que a API não quebre se alguém enviar dados no formato errado. Segurança de dados é prioridade no banco.
*   **Separation of Concerns:** A lógica de negócio (API) está separada da lógica de inferência (que poderia estar em um `predictor.py` separado futuramente).
*   **Observabilidade:** O endpoint `/health` é a base para o que faremos amanhã no CI/CD e monitoramento de Cloud.
