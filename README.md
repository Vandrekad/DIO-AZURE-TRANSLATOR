# DIO-AZURE-TRANSLATOR
Este Colab extrai texto de um artigo online, traduz para português e imprime o resultado.

## Passo 1: Instalação das Bibliotecas

```python
!pip install requests beautifulsoup4 openai langchain-openai
```

### Descrição das Bibliotecas
- **requests**: Usada para fazer requisições HTTP e buscar o conteúdo da página web.
- **beautifulsoup4**: Usada para analisar o HTML da página e extrair o texto.
- **openai**: Usada para interagir com a API do OpenAI.
- **langchain-openai**: Fornece uma interface para usar modelos de linguagem do OpenAI.

---

## Passo 2: Extração do Texto do Artigo

```python
from bs4 import BeautifulSoup
import requests

def extract_text_from_url(url):
    # Faz uma requisição para a URL
    response = requests.get(url)
    # Verifica se a requisição foi bem-sucedida
    if response.status_code == 200:
        # Analisa o HTML da página
        soup = BeautifulSoup(response.text, 'html.parser')
        # Remove scripts e estilos do HTML
        for script_or_style in soup(['script', 'style']):
            script_or_style.decompose()
        # Extrai o texto da página
        texto = soup.get_text(separator=' ')
        # Limpa o texto
        linhas = (line.strip() for line in texto.splitlines())
        parts = (phrase.strip() for line in linhas for phrase in line.split(" "))
        texto = '\n'.join(part for part in parts if part)
        return texto
    else:
        print(f"Failed to fetch the URL. Status code: {response.status_code}")
        return None
```

### Explicação:
1. A função `extract_text_from_url` recebe uma URL como entrada.
2. Busca o conteúdo da página web usando `requests`.
3. Usa `BeautifulSoup` para analisar o HTML e extrair o texto, removendo scripts e estilos.
4. Limpa o texto, removendo espaços em branco extras e linhas vazias.
5. Retorna o texto extraído.

---

## Passo 3: Tradução do Artigo

```python
from langchain_openai.chat_models.azure import AzureChatOpenAI

# Configura o cliente OpenAI
client = AzureChatOpenAI(
    azure_endpoint="...",  # Substitua pelos seus dados
    api_key="...",
    api_version="...",
    deployment_name="...",
    max_retries=0
)

def translate_article(text, lang):
    # Define as mensagens para a API
    messages = [
        ("system", "Você atua como tradutor de textos"),
        ("user", f"Traduza o seguinte texto para o idioma {lang}: {text}")
    ]
    # Chama a API para traduzir o texto
    response = client.invoke(messages)
    print(response.content)
    return response.content
```

### Explicação:
1. Importa `AzureChatOpenAI` de `langchain_openai.chat_models.azure`.
2. Inicializa o cliente OpenAI com suas credenciais.
3. A função `translate_article` recebe o texto e o idioma de destino como entrada.
4. Formata a solicitação para a API do OpenAI.
5. Chama a API para traduzir o texto.
6. Imprime e retorna a tradução.

---

## Passo 4: Execução

```python
# Extrai o texto de um artigo de exemplo
url = "https://dev.to/bhagvank/open-ai-codex-playground-5ebk"
text = extract_text_from_url(url)

# Traduz o artigo para português
article = translate_article(text, "pt-br")
print(article)

# Outro exemplo com URL diferente
url = "https://dev.to/ckeditor/openai-vs-aws-bedrock-vs-azure-open-ai-choosing-the-right-model-for-your-ai-assistant-1h3a"
text = extract_text_from_url(url)
article = translate_article(text, "pt-br")
print(article)
```

### Explicação:
1. Define a URL do artigo.
2. Chama `extract_text_from_url` para extrair o texto.
3. Usa `translate_article` para traduzir o texto para o português.
4. Imprime o artigo traduzido.
