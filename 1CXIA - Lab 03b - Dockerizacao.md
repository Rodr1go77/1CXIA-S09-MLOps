# 🐳 Lab 03b: Dockerização e Portabilidade (A Blindagem da IA)

*   **Disciplina:** 1CXIA - MLOps (Semana 09)
*   **Duração:** 45 minutos
*   **Ambiente:** Killercoda (Ubuntu/Docker)
*   **Acesso:** [Killercoda Play (Ubuntu)](https://killercoda.com/playgrounds/scenario/ubuntu)
*   **Objetivo:** Empacotar a Inference API em um contêiner Docker, garantindo que ela rode exatamente da mesma forma em qualquer ambiente (Colab, Servidor ou Nuvem).

---

## 🚩 O Cenário de Guerra
A API está funcionando, mas ela depende de bibliotecas específicas instaladas no seu ambiente virtual. Se um colega tentar rodar e tiver uma versão diferente do `torch`, tudo pode quebrar. No setor bancário, essa incerteza é inaceitável.

**Sua missão:** Criar um "passaporte" para sua aplicação: o **Dockerfile**. Vamos isolar a API, o modelo e as dependências dentro de uma imagem imutável.

---

## 🛠️ Passo a Passo

### 1. Criação do `Dockerfile`
O Dockerfile é a receita de bolo da nossa imagem. Note a ordem estratégica: copiamos o `requirements.txt` e instalamos as bibliotecas **antes** de copiar o código. Isso aproveita o cache do Docker, acelerando builds futuros.

```bash
# Na raiz do projeto:
cat <<EOF > Dockerfile
# 1. Imagem base moderna com Python 3.13-slim
FROM python:3.13-slim

# 2. Configuração do diretório de trabalho
WORKDIR /app

# 3. Instalação de dependências do sistema (se necessário)
RUN apt-get update && apt-get install -y --no-install-recommends \\
    build-essential \\
    && rm -rf /var/lib/apt/lists/*

# 4. Copia apenas o requirements primeiro (Otimização de Cache)
COPY requirements.txt .

# 4.1 Instalação de dependências FORÇANDO CPU (Salva a aula!)
RUN pip install --no-cache-dir torch --index-url https://download.pytorch.org/whl/cpu && \
    pip install --no-cache-dir -r requirements.txt

# 5. Copia o código da aplicação
COPY src/ ./src/

# 6. Expõe a porta que a API usará
EXPOSE 8000

# 7. Comando para iniciar a API
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
EOF
```

### 2. O arquivo `.dockerignore`
Não queremos "lixo" (como o ambiente virtual local ou arquivos do Git) dentro da nossa imagem.

```bash
cat <<EOF > .dockerignore
venv/
__pycache__/
.git/
.gitignore
EOF
```

### 3. Build da Imagem (Sobrevivência: Limpeza de Guerra)
O modelo BERT e a biblioteca `torch` são gigantes. No Killercoda, temos pouco espaço em disco. **Siga rigorosamente estes comandos de limpeza** antes de iniciar o build:

```bash
# 1. Remova o ambiente virtual local (O Docker não precisa dele e ele ocupa ~500MB)
rm -rf venv/

# 2. Limpeza AGRESSIVA do Docker (Remove TODAS as imagens e containers não usados)
docker system prune -af

# 3. Agora, realize o build da imagem (Isso pode levar alguns minutos)
docker build -t sentiment-api:v1 .
```
*Observe as camadas sendo criadas. O Docker baixará o Python 3.13 e instalará as dependências pesadas.*

### 4. Executando o Contêiner
Agora, vamos dar o "play" na nossa imagem blindada. O parâmetro `-p` mapeia a porta do contêiner para a porta do host.

```bash
# -d roda em background, -p mapeia as portas
docker run -d --name sentiment-container -p 8000:8000 sentiment-api:v1
```

### 5. Validação Final
Verifique se o contêiner está rodando e teste a API novamente via `curl`:

```bash
docker ps
curl -X POST "http://localhost:8000/predict" -H "Content-Type: application/json" -d '{"text": "O atendimento da Caixa no WhatsApp foi surpreendente."}'
```

### 6. Versionando a Infraestrutura
Registramos a "receita" no Git:

```bash
git add Dockerfile .dockerignore
git commit -m "feat: add Dockerfile for API containerization"
```

---

## 🏆 Resultado Esperado
Um contêiner rodando a Inference API de forma isolada, acessível pela porta 8000 do host, sem dependência do ambiente virtual local.

---

## 🛡️ Notas de MLOps (Portabilidade e Compliance)
*   **Imutabilidade:** Uma vez que a imagem é gerada, ela nunca muda. Isso garante que o modelo testado hoje será exatamente o mesmo que subirá em produção na Caixa.
*   **Layer Caching:** Entender a ordem do Dockerfile é a diferença entre um build de 10 segundos e um de 10 minutos. Sêniores otimizam tempo de CI/CD.
*   **Security (slim images):** Usamos a imagem `python:3.9-slim` para reduzir a superfície de ataque, removendo ferramentas de sistema desnecessárias.
