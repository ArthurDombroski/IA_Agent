# Agente de IA para Service Desk com RAG e LangGraph

Este projeto implementa um agente de Inteligência Artificial para um Service Desk corporativo, utilizando um sistema de triagem, RAG (Retrieval-Augmented Generation) e um fluxo de trabalho agêntico construído com LangGraph para responder a perguntas sobre políticas internas da empresa.

## Descrição

O objetivo deste agente é atuar como o primeiro nível de suporte no Service Desk, automatizando as respostas a perguntas comuns sobre as políticas internas da empresa. Ele é capaz de:

1.  **Analisar e triar** as perguntas dos usuários para determinar a ação correta.
2.  **Buscar informações** em uma base de conhecimento de documentos (PDFs de políticas).
3.  **Gerar respostas** coesas e baseadas no contexto encontrado.
4.  **Decidir** se uma pergunta pode ser resolvida automaticamente, se precisa de mais informações ou se deve ser escalada para um atendente humano (abrindo um chamado).

## Features

-   **Sistema de Triagem Inteligente**: Utiliza o Gemini 1.5 Flash para classificar as solicitações em `AUTO_RESOLVER`, `PEDIR_INFO` ou `ABRIR_CHAMADO`, otimizando o fluxo de atendimento.
-   **RAG (Retrieval-Augmented Generation)**: Enriquece o modelo de linguagem com informações de documentos de políticas internas (em formato PDF), permitindo respostas precisas e contextualizadas.
-   **Vector Store com FAISS**: Utiliza o FAISS para criar uma base de vetores dos documentos, permitindo buscas rápidas e eficientes por similaridade semântica.
-   **Fluxo Agêntico com LangGraph**: Modela o processo de decisão do agente como um grafo de estados, permitindo uma lógica complexa e condicionais para lidar com diferentes cenários de forma robusta.
-   **Citações de Fonte**: Fornece as fontes (documento e página) de onde a informação foi extraída, garantindo transparência e confiabilidade na resposta.

## Como Funciona

O fluxo de trabalho do agente é orquestrado pelo LangGraph e segue os seguintes passos:

1.  **Entrada do Usuário**: O agente recebe uma pergunta do usuário.
2.  **Nó de Triagem**: A pergunta é enviada a um LLM que a classifica e define uma `decisao` inicial e uma `urgencia`.
3.  **Roteamento Condicional**: Com base na decisão da triagem, o grafo direciona o fluxo:
    -   Se a decisão for `ABRIR_CHAMADO`, o agente formula uma resposta informando que um chamado será aberto e o fluxo termina.
    -   Se for `PEDIR_INFO`, ele solicita mais detalhes ao usuário e termina.
    -   Se for `AUTO_RESOLVER`, ele prossegue para o pipeline RAG.
4.  **Nó de Auto-Resolução (RAG)**:
    -   A pergunta do usuário é usada para buscar os trechos mais relevantes nos documentos de políticas armazenados no vector store (FAISS).
    -   Os trechos recuperados são injetados como contexto em um prompt para o LLM.
    -   O LLM gera uma resposta com base no contexto fornecido.
5.  **Decisão Pós-RAG**:
    -   Se o RAG encontrou contexto e gerou uma resposta, o agente a entrega ao usuário, juntamente com as citações, e o fluxo termina.
    -   Se o RAG não encontrou informações suficientes, o agente reavalia a pergunta. Se contiver palavras-chave como "aprovação" ou "exceção", ele abre um chamado. Caso contrário, pede mais informações.

## Dependências

O projeto utiliza as seguintes bibliotecas Python:

-   `langchain`
-   `langchain-google-genai`
-   `google-generativeai`
-   `langchain_community`
-   `faiss-cpu` (ou `faiss-gpu` para ambientes com GPU)
-   `langchain-text-splitters`
-   `pymupdf` (para carregar os PDFs)
-   `langgraph`

## Setup e Uso

1.  **Clone o Repositório**:
    ```bash
    git clone https://github.com/ArthurDombroski/IA_Agent.git
    cd IA_Agent
    ```

2.  **Instale as Dependências**:
    ```bash
    pip install -q --upgrade langchain langchain-google-genai google-generativeai langchain_community faiss-cpu langchain-text-splitters pymupdf langgraph
    ```

3.  **Configure a Chave de API**:
    -   Este notebook foi projetado para ser executado no Google Colab. Você precisa configurar sua chave de API do Gemini como um "Secret" com o nome `GEMINI_API_KEY`.
    -   Se for executar localmente, ajuste o código para carregar a chave de um arquivo `.env` ou de variáveis de ambiente.

4.  **Adicione os Documentos de Políticas**:
    -   Crie uma pasta (por exemplo, `/content/` no Colab) e adicione os arquivos PDF das políticas que servirão como base de conhecimento.
    -   O notebook está configurado para carregar todos os arquivos `.pdf` do diretório `/content/`.

5.  **Execute o Notebook**:
    -   Abra o `IAAgente.ipynb` em um ambiente Jupyter (como o Google Colab).
    -   Execute as células sequencialmente para treinar o agente e testá-lo com diferentes perguntas.
