# 🧪 Lab 02: O Teste de Fumaça (Validação do Script)

*   **Disciplina:** 1CXIA - MLOps (Semana 09)
*   **Duração:** 30 minutos
*   **Ambiente:** Killercoda (Dentro da pasta `caixa_sentiment_analysis`)
*   **Objetivo:** Criar o script final de inferência, validar a execução no ambiente isolado e introduzir métricas básicas de performance.

---

## 🚩 O Cenário de Guerra
Sua estrutura de pastas está pronta, o Git está inicializado e o ambiente virtual está ativo. Mas será que o código "sobreviveu" à migração do Colab para o servidor? 

Antes de darmos o dia como encerrado, precisamos realizar um **Teste de Fumaça** (*Smoke Test*): uma execução rápida que confirma que os componentes básicos (carregamento do modelo e inferência) estão operacionais.

**Sua missão:** Criar o script `predict.py` utilizando o modelo multilíngue e medir quanto tempo ele leva para "acordar" e processar uma frase.

---

## 🛠️ Passo a Passo

### 1. Criação do Script de Inferência
Utilizaremos o comando `cat` para criar o arquivo diretamente na pasta `src/`. Note que estamos usando o modelo **Multilíngue** para garantir que ele entenda o português.

```bash
# Certifique-se de que está na raiz do projeto e com o venv ativo!
cat <<EOF > src/predict.py
import time
from transformers import pipeline

print("🚀 Iniciando carregamento do modelo...")
start_load = time.time()

# Carregando o modelo multilíngue (BERT)
classifier = pipeline(
    "sentiment-analysis", 
    model="nlptown/bert-base-multilingual-uncased-sentiment"
)

end_load = time.time()
print(f"✅ Modelo carregado em {end_load - start_load:.2f} segundos.\n")

def predict(text):
    start_inf = time.time()
    result = classifier(text)[0]
    end_inf = time.time()
    
    print(f"Input: {text}")
    print(f"Label: {result['label']} | Confiança: {result['score']:.4f}")
    print(f"Tempo de inferência: {end_inf - start_inf:.4f} segundos\n")

if __name__ == "__main__":
    # Teste de validação
    predict("O atendimento no caixa eletrônico da Caixa foi excelente!")
    predict("Estou com dificuldades para acessar o Internet Banking.")
EOF
```

### 2. Execução do Teste
Agora, rode o script dentro do seu ambiente virtual:

```bash
# Executa o script de predição
python3 src/predict.py
```

### 3. Versionando a Vitória
Com o script funcionando, vamos registrar essa evolução no Git:

```bash
# Adiciona o novo script ao rastreamento
git add src/predict.py

# Realiza o commit seguindo o padrão técnico
git commit -m "feat: add inference script with multilingual BERT and timing metrics"
```

---

## 🏆 Resultado Esperado
Você deverá ver no terminal o tempo de carregamento (geralmente alguns segundos na primeira vez) e os resultados das predições para as frases em português com suas respectivas métricas de tempo.

---

## 🛡️ Notas Técnicas (Métricas de MLOps)
*   **Cold Start (Partida a Frio):** O tempo que o modelo leva para carregar (`Iniciando carregamento...`) é crítico em sistemas que escalam do zero.
*   **Inference Latency (Latência):** O tempo de processamento da frase é o que o usuário final "sente" ao usar o app da Caixa. Começar a medir isso desde o Dia 01 é a base da cultura de performance em MLOps.
