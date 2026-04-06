# 🧪 Lab 01b: Estruturação e Git (Ambiente de Infraestrutura)

*   **Disciplina:** 1CXIA - MLOps (Semana 09)
*   **Duração:** 45 minutos
*   **Ambiente:** Killercoda (Ubuntu Playground)
*   **Objetivo:** Criar a estrutura profissional do projeto, isolar o ambiente com `venv` e realizar o primeiro commit semântico.

---

## 🌐 Acesso ao Ambiente (Killercoda)

Para este laboratório, utilizaremos um playground Linux remoto para simular um servidor de infraestrutura:

1.  **Link Direto:** [killercoda.com/playgrounds/scenario/ubuntu](https://killercoda.com/playgrounds/scenario/ubuntu).
2.  **Configuração:** O Killercoda oferece sessões de **60 minutos**. Caso o tempo expire, você precisará recriar a estrutura (por isso a importância do Git que veremos agora).

---

## 🚩 O Cenário de Guerra
Agora que você realizou a "cirurgia" no Notebook e extraiu o código limpo de inferência, é hora de criar a "casa" oficial deste projeto no servidor da Caixa. Um projeto de MLOps robusto exige organização, isolamento de dependências e rastreabilidade total.

**Sua missão:** Sair do papel de "usuário de notebook" e assumir o de **Engenheiro de ML**, estruturando o repositório que servirá de base para a nossa esteira automatizada de CI/CD.

---

## 🛠️ Passo a Passo

### 1. Estrutura de Pastas Profissional
No terminal do Killercoda, execute os comandos abaixo para criar a hierarquia de diretórios que separa as responsabilidades do projeto:

```bash
# Cria o diretório raiz e a subestrutura (/src, /data, /models)
mkdir -p caixa_sentiment_analysis/{src,data,models}

# Entra na pasta do projeto
cd caixa_sentiment_analysis
```

### 2. Isolamento de Ambiente (`venv`)
Para garantir que as bibliotecas deste projeto não conflitem com outras aplicações do servidor, criaremos uma "caixa de ferramentas" isolada. **Observação:** Em sistemas Ubuntu mínimos, o suporte a ambientes virtuais deve ser instalado manualmente:

```bash
# Instala o suporte a ambientes virtuais no Ubuntu (necessário no Killercoda)
sudo apt update && sudo apt install -y python3-venv

# Cria o ambiente virtual chamado 'venv'
python3 -m venv venv

# Ativa o ambiente (O prefixo (venv) aparecerá no prompt do seu terminal)
source venv/bin/activate
```

### 3. Gestão de Dependências
Crie o arquivo `requirements.txt` com as versões validadas anteriormente e realize a instalação:

```bash
# Cria o arquivo de requisitos com as versões homologadas
cat <<EOF > requirements.txt
transformers==5.0.0
sentence-transformers==5.3.0
EOF

# Instala as dependências no seu ambiente virtual isolado
pip install -r requirements.txt
```

### 4. Inicialização do Git e Primeiro Commit
Transformaremos este diretório em um repositório rastreável, garantindo que arquivos pesados ou temporários não sejam enviados para o servidor de código.

```bash
# Inicializa o repositório Git
git init

# Configura o .gitignore para ignorar lixo e arquivos gigantes
cat <<EOF > .gitignore
venv/
__pycache__/
.DS_Store
data/*
models/*
*.pyc
EOF

# Adiciona todos os arquivos estruturais e realiza o primeiro commit
git add .
git commit -m "feat: initial project structure and dependencies"
```

---

## 🏆 Validação do Laboratório
Para confirmar que tudo está correto, valide sua estrutura com o comando:

```bash
# Lista a árvore de arquivos e diretórios
ls -R
```

---

## 🛡️ Notas Técnicas
*   **Por que o `.gitignore` é crítico?** Em MLOps, o Git deve rastrear apenas a inteligência (código). Arquivos binários de modelos (`models/`) e bibliotecas instaladas (`venv/`) são gerenciados por outras ferramentas para manter o repositório leve e ágil.
*   **Persistence Tip:** Como o Killercoda é efêmero, o Git é sua única garantia de que o trabalho poderá ser recuperado em outra sessão no futuro.
