


# Aplicação Flask com Docker

## Objetivo

Este projeto documenta o passo a passo da criação de um ambiente Docker e a conteinerização de uma aplicação web em Python utilizando o microframework Flask. O objetivo é demonstrar o ciclo completo de conteinerização: desde a preparação do ambiente e instalação do Docker, até a escrita do Dockerfile, build da imagem e execução do container. A aplicação foi desenvolvida em duas etapas, evoluindo de uma versão inicial simples para um sistema web completo com múltiplas rotas, templates dinâmicos e consumo de API via JavaScript.

## Estrutura do Projeto

O projeto está organizado em dois passos principais de desenvolvimento:

* **Passo 1 (Aplicação Base):** Construção da primeira versão da aplicação Flask contendo uma única rota, empacotada em uma imagem base `python:3.14-slim`.

```bash
projeto-flask/
├── app/
│   ├── app.py
│   └── requirements.txt
└── Dockerfile

```

* **Passo 2 (Aplicação Amplificada):** Expansão do projeto para incluir um layout compartilhado (navbar e rodapé), menu de navegação dinâmico alimentado pelo backend, quatro páginas distintas (Início, Sobre, Projetos e Contato), além de uma rota interna de API consumida no frontend em tempo real.

```bash
projeto-flask/
├── app
│   ├── app.py
│   ├── app.py.bak
│   ├── requirements.txt
│   ├── static
│   │   ├── css
│   │   │   └── style.css
│   │   └── js
│   │       └── script.js
│   └── templates
│       ├── base.html
│       ├── contato.html
│       ├── index.html
│       ├── projetos.html
│       └── sobre.html
└── Dockerfile

```

---

## Tutorial

### 1. Preparação do Ambiente e Acesso SSH

A hospedagem do Docker foi feita no Linux Mint através do Oracle VirtualBox. Para operar o ambiente pelo terminal da máquina real, configurou-se o acesso remoto via SSH.

Execute os comandos na máquina virtual para instalar e iniciar o serviço SSH:

```bash
sudo apt install openssh-server
sudo systemctl start ssh
hostname -I

```

* **`sudo apt install openssh-server`**: Instala o pacote do servidor SSH, permitindo conexões remotas seguras para esta máquina.
* **`sudo systemctl start ssh`**: Inicia ativamente o serviço do servidor SSH.
* **`hostname -I`**: Exibe o endereço IP atual da máquina virtual na rede local, necessário para a conexão externa.

Nota: Utilize o IP retornado pelo comando `hostname -I` para realizar o acesso.

Na sua máquina real, inicie a conexão:

```bash
sudo apt install openssh-server
sudo systemctl start ssh
ssh usuario@ip

```

* **`ssh usuario@ip`**: Abre um shell seguro na máquina remota. Substitua `usuario` pelo seu nome de usuário no Linux Mint e `ip` pelo endereço retornado no passo anterior.

### 2. Instalação do Docker

A instalação do Docker requer a adição do usuário ao grupo correto para dispensar o uso constante do `sudo`.

```bash
sudo apt install docker.io
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
docker images

```

* **`sudo apt install docker.io`**: Instala o motor principal do Docker no sistema operacional.
* **`sudo groupadd docker`**: Cria o grupo de usuários `docker` (caso ainda não exista).
* **`sudo usermod -aG docker $USER`**: Adiciona o usuário atual (`$USER`) ao grupo `docker`. O parâmetro `-aG` garante que a adição seja um anexo (append) aos grupos existentes do usuário.
* **`newgrp docker`**: Atualiza a sessão atual do terminal para aplicar as novas permissões do grupo `docker` sem precisar fazer logoff.
* **`docker run hello-world`**: Baixa e executa uma imagem oficial de teste para validar se o daemon do Docker está funcionando perfeitamente.
* **`docker images`**: Lista as imagens armazenadas no disco local da máquina (a imagem "hello-world" aparecerá aqui).

### 3. Passo 1: Construção da Aplicação Base

Crie a estrutura inicial de diretórios e arquivos da aplicação.

```bash
mkdir projeto-flask
mkdir projeto-flask/app
touch projeto-flask/Dockerfile
touch projeto-flask/app/app.py
touch projeto-flask/app/requirements.txt
sudo apt install tree
tree

```

* **`mkdir projeto-flask` e `mkdir projeto-flask/app**`: Criam o diretório raiz do projeto e a subpasta onde o código-fonte da aplicação residirá.
* **`touch ...`**: Cria arquivos vazios de forma rápida. Serão populados nos passos seguintes.
* **`sudo apt install tree`**: Instala o utilitário em linha de comando que exibe diretórios em formato de árvore.
* **`tree`**: Exibe de forma visual a hierarquia de pastas e arquivos criados até o momento.

#### Arquivos da Versão Base

Edite os arquivos recém-criados adicionando os seguintes conteúdos:

**`app/app.py`**

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return """
    <h1>Minha primeira aplicação Flask</h1>
    <p>Aplicação executando dentro de um container Docker!</p>
    """

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

```

**`app/requirements.txt`**

```text
Flask==3.1.2

```

**`Dockerfile`**

```dockerfile
FROM python:3.14-slim
WORKDIR /app
COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app/ .
EXPOSE 5000
CMD ["python", "app.py"]

```

#### Build e Execução da Versão Base

Baixe a imagem base, faça o build do projeto e rode o container:

```bash
docker pull python:3.14-slim
docker images
cd projeto-flask
docker build -t minha-flask .
docker images
docker run -d -p 5000:5000 --name meu-flask minha-flask
docker ps

```

* **`docker pull python:3.14-slim`**: Faz o download antecipado da imagem base do Python, otimizando o tempo do build subsequente.
* **`cd projeto-flask`**: Entra no diretório onde o `Dockerfile` está localizado.
* **`docker build -t minha-flask .`**: Constrói a imagem da aplicação lendo o `Dockerfile` presente no diretório atual (`.`) e aplica a tag (nome) `minha-flask`.
* **`docker run -d -p 5000:5000 --name meu-flask minha-flask`**: Inicia o container. O `-d` o roda em segundo plano (detached), o `-p 5000:5000` direciona o tráfego da porta 5000 da sua máquina para a 5000 do container, e `--name meu-flask` define um nome amigável para gerenciamento.
* **`docker ps`**: Lista apenas os containers ativos no momento, confirmando que a aplicação está em execução.

Acesse `http://localhost:5000/` no navegador para verificar a aplicação rodando.

### 4. Passo 2: Aplicação Amplificada (Múltiplas Rotas e Layout Completo)

Para separar responsabilidades (HTML, CSS e JavaScript), crie novas pastas de arquivos estáticos e templates.

```bash
cp app/app.py app/app.py.bak
mkdir -p app/templates
mkdir -p app/static/css
mkdir -p app/static/js
tree

```

* **`cp app/app.py app/app.py.bak`**: Cria uma cópia de segurança (backup) do arquivo original da aplicação base.
* **`mkdir -p ...`**: O argumento `-p` cria a estrutura completa de diretórios de uma só vez (ex: cria as pastas `static` e `css` simultaneamente sem gerar erro caso o pai não exista).

#### Backend Expandido

Atualize o arquivo principal para servir os templates HTML e a API interna.

**`app/app.py`**

```python
from flask import Flask, render_template, jsonify
from datetime import datetime

app = Flask(__name__)

MENU = [
    {"nome": "Início", "rota": "/", "endpoint": "home"},
    {"nome": "Sobre", "rota": "/sobre", "endpoint": "sobre"},
    {"nome": "Projetos", "rota": "/projetos", "endpoint": "projetos"},
    {"nome": "Contato", "rota": "/contato", "endpoint": "contato"},
]

@app.context_processor
def injetar_globais():
    return dict(menu=MENU, ano_atual=datetime.now().year)

@app.route("/")
def home():
    return render_template("index.html", titulo="Início", ativo="home")

@app.route("/sobre")
def sobre():
    integrantes = [
        {"nome": "Alefe Ruan", "papel": "Desenvolvimento"},
        {"nome": "Clara Fiuza Serejo", "papel": "Desenvolvimento"},
        {"nome": "João Lucas Dias", "papel": "Infraestrutura / Docker"},
    ]
    return render_template("sobre.html", titulo="Sobre", ativo="sobre", integrantes=integrantes)

@app.route("/projetos")
def projetos():
    lista_projetos = [
        {"nome": "App Flask em Docker", "descricao": "...", "status": "Concluído"},
        {"nome": "Monitoramento de containers", "descricao": "...", "status": "Planejado"},
        {"nome": "API de segurança", "descricao": "...", "status": "Em andamento"},
    ]
    return render_template("projetos.html", titulo="Projetos", ativo="projetos", projetos=lista_projetos)

@app.route("/contato")
def contato():
    return render_template("contato.html", titulo="Contato", ativo="contato")

@app.route("/api/status")
def api_status():
    return jsonify({
        "status": "ok",
        "aplicacao": "projeto-flask",
        "ambiente": "docker",
        "hora_servidor": datetime.now().strftime("%H:%M:%S"),
    })

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)

```

#### Templates HTML

Crie os arquivos HTML dentro da pasta `app/templates`.

**`app/templates/base.html`**

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{ titulo }} - Projeto Flask + Docker</title>
<link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
<header class="topbar">
<nav class="navbar">
<a class="logo" href="{{ url_for('home') }}"> Projeto-Flask</a>
<button class="menu-toggle" id="menu-toggle" aria-label="Abrir menu">
<span></span><span></span><span></span>
</button>
<ul class="nav-links" id="nav-links">
{% for item in menu %}
<li>
<a href="{{ item.rota }}" class="{{ 'ativo' if item.endpoint == ativo else '' }}">
{{ item.nome }}
</a>
</li>
{% endfor %}
</ul>
</nav>
</header>
<main class="conteudo">
{% block conteudo %}{% endblock %}
</main>
<footer class="rodape">
<p>IFSP Campos do Jordão &mdash; {{ ano_atual }}</p>
<p class="status" id="status-servidor">verificando status do servidor...</p>
</footer>
<script src="{{ url_for('static', filename='js/script.js') }}"></script>
</body>
</html>

```

**`app/templates/index.html`**

```html
{% extends "base.html" %}
{% block conteudo %}
<section class="hero">
<h1>Minha primeira aplicação Flask</h1>
<p>Aplicação executando dentro de um container Docker, com navbar, menu dinâmico e
páginas separadas.</p>
<a class="botao" href="{{ url_for('projetos') }}">Ver projetos</a>
</section>
<section class="cartoes">
<div class="cartao">
<h3> Navegação</h3>
<p>Menu montado a partir de uma lista no <code>app.py</code>, não hardcoded em cada
HTML.</p>
</div>
<div class="cartao">
<h3> Backend</h3>
<p>Rotas separadas por responsabilidade: início, sobre, projetos e contato.</p>
</div>
<div class="cartao">
<h3> API interna</h3>
<p>O rodapé consulta <code>/api/status</code> via JavaScript para mostrar que o servidor
está de pé.</p>
</div>
</section>
{% endblock %}

```

**`app/templates/sobre.html`**

```html
{% extends "base.html" %}
{% block conteudo %}
<section class="pagina-sobre">
<h1>Sobre o projeto</h1>
<p>
Este projeto foi desenvolvido como parte do trabalho avaliativo de
Gerenciamento de Segurança de Dados, demonstrando a conteinerização
de uma aplicação web em Python Flask usando Docker.
</p>
<h2>Equipe</h2>
<ul class="lista-integrantes">
{% for pessoa in integrantes %}
<li>
<strong>{{ pessoa.nome }}</strong>
<span class="papel">{{ pessoa.papel }}</span>
</li>
{% endfor %}
</ul>
</section>
{% endblock %}

```

**`app/templates/projetos.html`**

```html
{% extends "base.html" %}
{% block conteudo %}
<section class="pagina-projetos">
<h1>Projetos</h1>
<p>Esta lista vem do backend (variável Python em <code>app.py</code>), não está escrita fixa no
HTML.</p>
<div class="grade-projetos">
{% for p in projetos %}
<article class="cartao-projeto">
<h3>{{ p.nome }}</h3>
<p>{{ p.descricao }}</p>
<span class="badge badge-{{ p.status|lower|replace(' ', '-') }}">{{ p.status }}</span>
</article>
{% endfor %}
</div>
</section>
{% endblock %}

```

**`app/templates/contato.html`**

```html
{% extends "base.html" %}
{% block conteudo %}
<section class="pagina-contato">
<h1>Contato</h1>
<p>Formulário de exemplo (front-end apenas — não envia dados para nenhum servidor).</p>
<form class="formulario" onsubmit="return false;">
<label>
Nome
<input type="text" placeholder="Seu nome" required>
</label>
<label>
E-mail
<input type="email" placeholder="voce@exemplo.com" required>
</label>
<label>
Mensagem
<textarea rows="4" placeholder="Sua mensagem..."></textarea>
</label>
<button type="submit" class="botao">Enviar</button>
</form>
</section>
{% endblock %}

```

#### Arquivos Estáticos (CSS e JS)

Crie os arquivos visuais e de interação do front-end.

**`app/static/css/style.css`**

```css
:root {
  --azul: #2563eb;
  --azul-escuro: #1e3a8a;
  --cinza-fundo: #f5f7fb;
  --cinza-texto: #333;
  --branco: #fff;
  --verde: #16a34a;
  --amarelo: #ca8a04;
  --raio: 10px;
}
* { box-sizing: border-box; }
body {
  margin: 0;
  font-family: "Segoe UI", Roboto, Arial, sans-serif;
  background: var(--cinza-fundo);
  color: var(--cinza-texto);
}
.topbar { background: var(--azul-escuro); }
.navbar {
  max-width: 1000px;
  margin: 0 auto;
  padding: 0.8rem 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.logo {
  color: var(--branco);
  font-weight: bold;
  font-size: 1.2rem;
  text-decoration: none;
}
.nav-links {
  list-style: none;
  display: flex;
  gap: 1.5rem;
  margin: 0;
  padding: 0;
}
.nav-links a {
  color: #dbe4ff;
  text-decoration: none;
  font-weight: 500;
  padding: 0.3rem 0;
  border-bottom: 2px solid transparent;
}
.nav-links a:hover,
.nav-links a.ativo {
  color: var(--branco);
  border-bottom-color: var(--branco);
}
.menu-toggle {
  display: none;
  flex-direction: column;
  gap: 4px;
  background: none;
  border: none;
  cursor: pointer;
}
.menu-toggle span {
  width: 24px;
  height: 2px;
  background: var(--branco);
}
.conteudo {
  max-width: 1000px;
  margin: 0 auto;
  padding: 2rem 1rem 3rem;
}
.hero {
  text-align: center;
  padding: 2rem 1rem;
}
.hero h1 { font-size: 2rem; margin-bottom: 0.5rem; }
.botao {
  display: inline-block;
  margin-top: 1rem;
  padding: 0.6rem 1.4rem;
  background: var(--azul);
  color: var(--branco);
  border: none;
  border-radius: var(--raio);
  text-decoration: none;
  font-weight: 600;
  cursor: pointer;
}
.botao:hover { background: var(--azul-escuro); }
.cartoes {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
  margin-top: 2rem;
}
.cartao, .cartao-projeto {
  background: var(--branco);
  padding: 1.2rem;
  border-radius: var(--raio);
  box-shadow: 0 1px 4px rgba(0,0,0,0.08);
}
.lista-integrantes {
  list-style: none;
  padding: 0;
}
.lista-integrantes li {
  background: var(--branco);
  padding: 0.8rem 1rem;
  margin-bottom: 0.5rem;
  border-radius: var(--raio);
  display: flex;
  justify-content: space-between;
}
.papel { color: #666; font-size: 0.9rem; }
.grade-projetos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1rem;
  margin-top: 1.5rem;
}
.badge {
  display: inline-block;
  margin-top: 0.6rem;
  padding: 0.2rem 0.6rem;
  border-radius: 999px;
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--branco);
}
.badge-concluído { background: var(--verde); }
.badge-planejado { background: #6b7280; }
.badge-em-andamento { background: var(--amarelo); }
.formulario {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  max-width: 420px;
  background: var(--branco);
  padding: 1.5rem;
  border-radius: var(--raio);
  box-shadow: 0 1px 4px rgba(0,0,0,0.08);
}
.formulario label { display: flex; flex-direction: column; gap: 0.3rem; font-size: 0.9rem; }
.formulario input, .formulario textarea {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-family: inherit;
}
.rodape {
  text-align: center;
  padding: 1rem;
  color: #777;
  font-size: 0.85rem;
}
.status { font-family: monospace; }
@media (max-width: 700px) {
  .menu-toggle { display: flex; }
  .nav-links {
    position: absolute;
    top: 60px;
    left: 0;
    right: 0;
    background: var(--azul-escuro);
    flex-direction: column;
    gap: 0;
    display: none;
  }
  .nav-links.aberto { display: flex; }
  .nav-links li { width: 100%; text-align: center; padding: 0.6rem 0; }
  .navbar { position: relative; }
}

```

**`app/static/js/script.js`**

```javascript
const botaoMenu = document.getElementById("menu-toggle");
const links = document.getElementById("nav-links");

if (botaoMenu && links) {
  botaoMenu.addEventListener("click", () => {
    links.classList.toggle("aberto");
  });
}

async function atualizarStatus() {
  const alvo = document.getElementById("status-servidor");
  if (!alvo) return;
  try {
    const resposta = await fetch("/api/status");
    const dados = await resposta.json();
    alvo.textContent = `servidor: ${dados.status} - ambiente: ${dados.ambiente} - ${dados.hora_servidor}`;
  } catch (erro) {
    alvo.textContent = "não foi possível contatar o servidor.";
  }
}

atualizarStatus();
setInterval(atualizarStatus, 5000);

```

#### Rebuild e Teste Final

Para validar a versão final, pare o container antigo, reconstrua a imagem e inicie um novo container.

```bash
docker stop meu-flask && docker rm meu-flask
docker build -t flask-app .
docker images
docker run -d -p 5000:5000 --name meu-flask flask-app

```

* **`docker stop meu-flask && docker rm meu-flask`**: O comando `stop` encerra a execução do container ativo. O `rm` o exclui da máquina. Isso é necessário porque a porta 5000 e o nome "meu-flask" precisam estar livres antes de iniciarmos a versão nova. O `&&` garante que a exclusão só ocorra se a parada for bem-sucedida.
* **`docker build -t flask-app .`**: Refaz o processo de build do zero (lendo os novos HTMLs, CSS e JS) e cria uma nova imagem, agora chamada `flask-app`.
* **`docker run -d -p 5000:5000 --name meu-flask flask-app`**: Inicia novamente o container, restaurando o serviço na porta 5000 usando a nova imagem.

O container servirá as páginas e responderá ativamente à consulta periódica do rodapé na porta 5000 do host.

---
#### Teste no navegador
Para testar se a aplicação está rodando insira na aba de pesquisa do seu navegador:
• localhost:5000
ou 
•192.169.0.x:5000 (no lugar do 192... coloque o ip da máquina que está rodando a aplicação)
---

## Lista de Comandos Mais Importantes do Docker

* **`docker pull python:3.14-slim`**: Baixa a imagem base diretamente do Docker Hub antes do build, garantindo a sua disponibilidade local.
* **`docker images`**: Lista todas as imagens já baixadas ou criadas na máquina.
* **`docker build -t [nome_imagem] .`**: Constrói uma nova imagem Docker baseada nas instruções do `Dockerfile` presente no diretório atual.
* **`docker run -d -p 5000:5000 --name [nome_container] [nome_imagem]`**: Cria e executa um container em modo background (`-d`), mapeando a porta local para a porta do container, aplicando o nome de referência informado.
* **`docker ps`**: Lista os containers que estão em execução no momento.
* **`docker stop [nome_container] && docker rm [nome_container]`**: Encerra e remove o container atual, uma etapa necessária antes de subir um novo container com o mesmo nome.


# CRÉDITOS 
## Desenvolvedores
• João Lucas Dias   • Clara Fiuza Serejo   •Alefe Ruan Batista da Silva
## Orientador
• Cesar Augusto de Moraes Costa
