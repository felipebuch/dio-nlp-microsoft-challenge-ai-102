
# DIO NLP Microsoft Challenge AI-102

Este repositório contém notebooks desenvolvidos como parte do desafio de processamento de linguagem natural do bootcamp Microsoft Challenge AI-102, oferecido pela plataforma DIO. Os notebooks implementam ferramentas para manipulação e tradução de texto utilizando APIs de OpenAI e Azure.

## Notebooks

### 1. `dio_bootcamp_openai.ipynb`

Este notebook utiliza a API do OpenAI para extrair texto de páginas web e traduzi-lo. Abaixo está o código com comentários explicativos:

```python
# Importação das bibliotecas necessárias
import requests  # Para realizar requisições HTTP
from bs4 import BeautifulSoup  # Para manipular e extrair dados de HTML
from langchain_openai.chat_models.azure import AzureChatOpenAI  # Para integrar com a API do OpenAI

# Função para extrair texto limpo de uma URL
def extract_text_from_url(url):
    response = requests.get(url)  # Realiza a requisição HTTP
    if response.status_code == 200:  # Verifica se a requisição foi bem-sucedida
        soup = BeautifulSoup(response.text, 'html.parser')  # Faz o parsing do HTML
        # Remove tags de script e estilo que não são relevantes
        for script_or_style in soup(["script", "style"]):
            script_or_style.decompose()
        # Extrai o texto da página
        text = soup.get_text(separator=' ')
        # Remove espaços desnecessários e organiza o texto em linhas limpas
        lines = (line.strip() for line in text.splitlines())
        chunks = (phrase.strip() for line in lines for phrase in line.split("  "))
        clean_text = '\n'.join(chunk for chunk in chunks if chunk)
        return clean_text  # Retorna o texto limpo
    else:
        print(f"Failed to fetch the URL. Status code: {response.status_code}")
        return None  # Retorna None em caso de erro

# Configuração do cliente OpenAI
client = AzureChatOpenAI(
    azure_endpoint="YOUR_ENDPOINT_HERE",  # Endpoint da API Azure
    api_key="YOUR_API_KEY_HERE",  # Chave de API
    api_version="YOUR_API_VERSION_HERE",  # Versão da API
    deployment_name="YOUR_DEPLOYMENT_NAME_HERE",  # Nome do deployment
    max_retries=0  # Número máximo de tentativas
)

# Função para traduzir um texto usando a API OpenAI
def translate_article(text, lang):
    # Define as mensagens para o modelo
    messages = [
        ("system", "Você atua como tradutor de textos"),
        ("user", f"Traduza o {text} para o idioma {lang} e responda em markdown")
    ]
    response = client.invoke(messages)  # Envia a mensagem para o modelo
    return response.content  # Retorna o texto traduzido

# Exemplo de uso
url = "ARTICLE_URL_HERE"  # URL do artigo a ser traduzido
extracted_text = extract_text_from_url(url)  # Extrai o texto do artigo
translated_article = translate_article(extracted_text, "portugues")  # Traduz o texto para português
print(translated_article)  # Exibe o resultado traduzido
```

### 2. `dio_bootcamp_translator.ipynb`

Este notebook utiliza a API de Tradução da Microsoft (Azure Cognitive Services) para traduzir textos e documentos Word. Abaixo está o código com comentários explicativos:

```python
# Importação das bibliotecas necessárias
import requests  # Para realizar requisições HTTP
from docx import Document  # Para manipular arquivos .docx
import os  # Para manipulação de operações de sistema

# Configurações da API de Tradução do Azure
subscription_key = "YOUR_SUBSCRIPTION_KEY"  # Chave de assinatura do Azure
endpoint = "https://api.cognitive.microsofttranslator.com"  # Endpoint da API
location = "YOUR_LOCATION"  # Região configurada
target_language = "LANGUAGE_CODE"  # Código do idioma de destino

# Função para traduzir texto simples
def translate_text(text, target_language):
    path = "/translate"  # Caminho do endpoint de tradução
    constructed_url = endpoint + path  # URL completa
    headers = {
        'Ocp-Apim-Subscription-Key': subscription_key,  # Chave de assinatura
        'Ocp-Apim-Subscription-Region': location,  # Região da assinatura
        'Content-type': 'application/json',  # Tipo de conteúdo
        'X-ClientTraceId': str(os.urandom(16))  # ID único para rastreamento
    }
    body = [{'text': text}]  # Corpo da requisição com o texto
    params = {
        'api-version': '3.0',  # Versão da API
        'from': 'en',  # Idioma de origem
        'to': target_language  # Idioma de destino
    }
    # Faz a requisição POST para a API de tradução
    request = requests.post(constructed_url, params=params, headers=headers, json=body)
    response = request.json()  # Converte a resposta para JSON
    return response[0]["translations"][0]["text"]  # Retorna o texto traduzido

# Função para traduzir um documento Word
def translate_document(path):
    document = Document(path)  # Carrega o documento .docx
    full_text = []  # Lista para armazenar os textos traduzidos
    for paragraph in document.paragraphs:  # Itera pelos parágrafos do documento
        translated_text = translate_text(paragraph.text, target_language)  # Tradução do parágrafo
        full_text.append(translated_text)  # Adiciona o texto traduzido à lista
    
    # Cria um novo documento para o texto traduzido
    translated_doc = Document()
    for line in full_text:
        translated_doc.add_paragraph(line)  # Adiciona os parágrafos traduzidos
    
    # Define o caminho do documento traduzido
    path_translated = path.replace(".docx", f"_{target_language}.docx")
    translated_doc.save(path_translated)  # Salva o novo documento
    return path_translated  # Retorna o caminho do documento traduzido

# Exemplo de uso
translated_path = translate_document("/content/musica.docx")  # Traduz o documento "musica.docx"
print(f"Documento traduzido salvo em: {translated_path}")  # Exibe o caminho do arquivo traduzido
```

## Como Usar

1. Clone este repositório:

   ```bash
   git clone https://github.com/seu-usuario/dio-nlp-microsoft-challenge-ai-102.git
   ```
2. Configure as chaves de API nos locais indicados nos notebooks.
3. Execute os notebooks para realizar as operações de extração, tradução e manipulação de textos.

## Licença

Este projeto é licenciado sob a [MIT License](LICENSE).


Se precisar de mais ajustes ou explicações detalhadas, é só avisar!
