# 🧪 Lab 06: O Cofre de Dados (DVC & Rastreabilidade)

Neste laboratório, você aprenderá a resolver o maior pesadelo do MLOps: **Versionar Dados Gigantes**. Utilizaremos o **DVC (Data Version Control)** para garantir que a CAIXA tenha o rastro completo de qual dado gerou qual modelo, sem travar o Git.

---

## 🎯 Objetivo do Lab
1.  Isolar o armazenamento de dados (DVC) do versionamento de código (Git).
2.  Configurar um **Remote Local** (simulando um S3/Azure Blob).
3.  Garantir a **Imutabilidade** do dataset.
4.  Simular a recuperação de dados em um cenário de desastre (**Chaos Scenario**).

---

## 🖥️ Acesso ao Ambiente
Utilizaremos o playground de **Ubuntu** do Killercoda:
👉 **Acesse aqui:** [https://killercoda.com/playgrounds/scenario/ubuntu](https://killercoda.com/playgrounds/scenario/ubuntu)

---

## 🚀 Atividade 0: Bootstrap (Restauração de Ambiente)

Se a sua sessão do Killercoda expirou ou você deseja começar do zero com o estado ideal, copie e cole o bloco abaixo integralmente no terminal. Ele reconstruirá o projeto, os testes, o Agente e o Git em segundos:

```bash
# 1. Limpeza e Organização do Projeto
cd ~ && rm -rf ~/lab04 && mkdir ~/lab04 && cd ~/lab04

# 2. Reconstrução do Modelo (Inference Logic)
cat <<EOF > model.py
def predict_credit_score(income, age, debt):
    if income < 0 or age < 0 or debt < 0:
        return None
    score = (income / 10000) + (age / 100) - (debt / 5000)
    return max(0, min(1, score))
EOF

# 3. Reconstrução dos Testes (Pytest Logic)
cat <<EOF > test_model.py
from model import predict_credit_score
def test_sanity_range():
    result = predict_credit_score(5000, 30, 1000)
    assert 0 <= result <= 1
def test_negative_input():
    result = predict_credit_score(-100, 30, 1000)
    assert result is None
EOF

# 4. Setup do Ambiente Python, Git e AGENTE (O Músculo)
sudo apt update && sudo apt install -y python3-venv --quiet
python3 -m venv venv && source venv/bin/activate
pip install pytest --quiet
git init --quiet
git branch -M main
git config user.email "aluno@caixa.gov.br"
git config user.name "Aluno CAIXA"
git add model.py test_model.py
git commit -m "feat: initial inference and test logic (restored)" --quiet

# 5. Bootstrap do Agente (Lab 05a Fast-Track)
sudo useradd -m agentuser
echo "agentuser ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/agentuser
sudo mkdir -p /home/agentuser/myagent && cd /home/agentuser/myagent
sudo curl -fkSL -o agent.tar.gz https://download.agent.dev.azure.com/agent/4.270.0/vsts-agent-linux-x64-4.270.0.tar.gz
sudo tar zxvf agent.tar.gz
sudo ./bin/installdependencies.sh
sudo chown -R agentuser:agentuser /home/agentuser/myagent
sudo chmod -R 777 /home/agentuser/myagent
cd ~/lab04

echo "✅ Ambiente Restaurado! Pasta: ~/lab04 | venv: Ativado | Git: Pronto"
```

---

## 🌐 Atividade 0.1: Ajuste de Conectividade (DNS)

Execute os comandos abaixo para garantir que o ambiente consiga baixar pacotes e conectar ao Azure DevOps sem falhas:

```bash
sudo rm -f /etc/resolv.conf
sudo tee /etc/resolv.conf <<EOF
nameserver 8.8.8.8
nameserver 8.8.4.4
EOF
```

---

## 🔗 Atividade 0.2: Setup de Repositório Limpo (ADO)

Para evitar conflitos de histórico, vamos criar um repositório exclusivo para hoje.

1.  No Portal **Azure DevOps**, vá em **Repos -> Files**.
2.  No topo da página, clique no nome do repositório atual e selecione **+ New repository**.
3.  **Repository name:** `Caixa_DVC`.
4.  **Add a README:** DESMARCADO (Deve estar 100% vazio).
5.  Clique em **Create** e copie a URL gerada.

---

## 🚀 Atividade 0.3: Reativando o Agente (Obrigatório)

Se você reconstruiu o ambiente, seu Agente **não está mais conectado** ao Azure DevOps. Sem ele, o Pipeline não executará.

**Missão:** Execute o comando abaixo, substituindo os valores:

```bash
# Entre no contexto do agente e configure-o com o seu PAT (Aula 03)
sudo chmod -R 777 /home/agentuser/myagent
sudo -u agentuser bash -c "cd ~/myagent && ./config.sh --unattended \
  --url https://dev.azure.com/<SUA_ORGANIZACAO> \
  --auth pat \
  --token <SEU_PAT> \
  --pool default \
  --agent killercoda-agent \
  --replace"

# Inicie o Agente (O terminal ficará preso aqui, 'ouvindo' jobs)
sudo -u agentuser bash -c "cd ~/myagent && ./run.sh"
```
*(Abra um NOVO TERMINAL no Killercoda para prosseguir com o Lab 06).*

---

## 🛠️ Atividade 1: Instalação e Setup do DVC

No terminal do Killercoda, **garanta que o ambiente virtual está ativo** e instale o DVC:

```bash
# 1. Ativar o venv (Obrigatório em cada novo terminal)
cd ~/lab04
source venv/bin/activate

# 2. Instalar o DVC dentro do ambiente virtual
pip install dvc

# 3. Inicializar o DVC na pasta do projeto
dvc init

# 4. Verificar o que o DVC criou
ls -a .dvc
git status
git commit -m "build: initialize DVC"
```

---

## 📂 Atividade 2: Simulando o Dataset da CAIXA

Vamos criar um arquivo de dados "pesado" (simulado) que não queremos no Git.

```bash
# Criar pasta de dados
mkdir data

# Gerar um arquivo de transações fictício
cat <<EOF > data/transacoes_caixa.csv
id,valor,data,fraude
1,150.00,2026-04-01,0
2,2500.00,2026-04-01,1
3,50.00,2026-04-02,0
EOF

# Tentar ver o tamanho (Simulando que fosse 1GB)
ls -lh data/transacoes_caixa.csv
```

---

## 🔐 Atividade 3: O "Cofre" (DVC Add & Remote)

Agora, vamos pedir para o DVC cuidar do arquivo e o Git cuidar apenas do rastro.

```bash
# 1. Adicionar o arquivo ao controle do DVC
dvc add data/transacoes_caixa.csv

# 2. O que aconteceu? 
# O arquivo real foi para o cache do DVC (.dvc/cache)
# Um arquivo de "ponteiro" foi criado: data/transacoes_caixa.csv.dvc
ls -lh data/

# 3. Adicionar o PONTEIRO ao Git
git add data/transacoes_caixa.csv.dvc data/.gitignore
git commit -m "data: track transactions dataset v1 via DVC"
```

---

## 🌪️ Atividade 4: O Cenário de Caos (Desastre e Recuperação)

**Missão:** Você acabou de deletar o banco de dados de treinamento por acidente!

```bash
# 1. O DESASTRE: Deletar o arquivo real
rm data/transacoes_caixa.csv
ls data/ # O arquivo SUMIU!

# 2. A SALVAÇÃO: Pedir ao DVC para restaurar com base no ponteiro do Git
dvc checkout

# 3. VALIDAÇÃO: O arquivo voltou exatamente como estava!
ls -lh data/
cat data/transacoes_caixa.csv
```

---

## ☁️ Atividade 5: Integração com Azure DevOps (ADO)

Para que seu Pipeline (Aula 03) saiba qual dado usar, o ADO precisa do arquivo `.dvc`.

1.  No portal **Azure DevOps**, vá em **Repos -> Files** e copie a URL do seu repositório.
2.  **🔐 Verificação de Segurança (PAT):** O seu Token (PAT) da Aula 03 precisa de permissão para enviar código. Se falhou:
    *   No Portal Azure, vá em **User Settings** -> **Personal Access Tokens**.
    *   Clique no seu token (`KillercodaAgent`) ou crie um novo.
    *   Garanta que em **Scopes** a opção **Code** esteja marcada como **Read & Write**.
    *   Clique em **Save** (ou Create e copie o novo token).

3.  No seu terminal, vincule o novo repositório e faça o push:

```bash
# 1. Vincular ao NOVO Azure Repos (Substitua a URL do Caixa_DVC)
git remote add origin <URL_DO_REPOSITORIO_CAIXA_DVC>

# 2. Fazer o push dos arquivos (DVC e Código)
git push -u origin main

# 🔑 DICA: Username 'caixa' | Password [SEU PAT COM PERMISSÃO CODE]
```

4.  No portal **Azure DevOps**, vá em **Repos -> Files**.
5.  Observe que o arquivo `.csv` **NÃO está lá**, mas o arquivo `.csv.dvc` **ESTÁ**.
6.  **O Rastro:** Abra o arquivo `.dvc` no portal. O código `md5` é a "Certidão de Nascimento" do dado.


---

## 💡 Reflexão para o Time CAIXA
*   O DVC permite que o Cientista mude o dado (v1 -> v2) e o Engenheiro de ML saiba exatamente qual versão foi usada no Pipeline, sem nunca precisar baixar arquivos gigantes manualmente ou via e-mail.
*   Em produção (Aula 05), o Agente usará o `dvc pull` para baixar os dados do **Azure Blob Storage** de forma segura.

### ✅ Lab 06 Concluído!
Você agora domina a **Rastreabilidade de Dados**. Próximo passo: **Tracking de Experimentos (MLflow)**.
