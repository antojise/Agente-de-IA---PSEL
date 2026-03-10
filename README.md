# Desafio Tecnico - Chatbot com IA Generativa

Este repositorio contem a solucao desenvolvida para o desafio tecnico da JetSales. O projeto consiste em um fluxo de automacao construindo um Agente Autonomo de Inteligencia Artificial para atendimento via WhatsApp, utilizando a ferramenta low-code n8n, modelos LLM do Google Gemini e o banco de dados Redis em memoria.

## Visao Geral da Arquitetura

O sistema foi desenhado para atuar como um qualificador de leads ("Jorge", especialista em marketing) com foco em agendamento de reunioes. A arquitetura vai alem de um fluxo estatico de perguntas e respostas, implementando o ecossistema LangChain para dar autonomia cognitiva ao agente, permitindo a interpretacao de contextos complexos e o acionamento de ferramentas externas.

## Principais Funcionalidades Implementadas

* **Integracao Nativa (Webhook/API):** Recebimento de eventos em tempo real via Webhook da plataforma JetSales e devolucao de respostas processadas via requisicoes HTTP autenticadas por Bearer Token.
* **Agente Autonomo (LangChain):** Utilizacao do modelo Google Gemini Flash associado a um Buffer de Memoria (Window Memory) para manter o contexto da conversa e interpretar a intencao do usuario.
* **Escalonamento Inteligente (Tools):** Implementacao de uma ferramenta (HTTP Request Tool) acoplada ao Agente. A IA tem autonomia para identificar solicitacoes de suporte humano e disparar, por conta propria, um endpoint notificando os atendentes da JetSales.
* **Processamento Multimodal:** O fluxo possui um roteador que identifica o tipo de midia recebida (Texto, Imagem ou Audio). Imagens e audios sao processados por modelos de visao computacional e transcricao, injetando o contexto traduzido diretamente na memoria do Agente.
* **Debounce e Controle de Concorrencia:** Utilizacao do Redis para agrupar mensagens sucessivas do cliente (buffer temporal), enviando um unico bloco de contexto para a IA, otimizando o uso de tokens e evitando respostas atravessadas.
* **Tratamento de Erros e Duplicatas:** Filtros para prevencao de loops (ignorar mensagens enviadas pelo proprio sistema) e bloqueio de execucoes duplicadas baseadas no ID da mensagem armazenado temporariamente no Redis.

## Requisitos do Sistema

Para executar este fluxo em seu proprio ambiente n8n, sera necessario possuir:

* n8n (Versao 1.0 ou superior com suporte a nos LangChain Advanced)
* Servidor Redis configurado e acessivel
* Chave de API do Google Gemini (Google AI Studio / Vertex AI)

## Instrucoes de Instalacao

1. Clone ou baixe este repositorio.
2. Acesse a interface da sua instancia do n8n.
3. No menu lateral esquerdo, va em "Workflows" e clique em "Add Workflow".
4. No canto superior direito, clique no menu de opcoes (tres pontos) e selecione "Import from File".
5. Selecione o arquivo `chatbot-jetsales-antonio.json` fornecido neste repositorio.
6. Apos a importacao, configure as credenciais pendentes nos nos marcados:
   * **Redis account:** Insira os dados de host, porta e senha do seu servidor Redis.
   * **Google Gemini API:** Insira a sua chave de API gerada no Google AI Studio.
7. Ajuste a URL do Webhook inicial para o endereco exposto pelo seu servidor ou ferramenta de tunelamento.
8. Ative a chave "Active" no canto superior direito para que o fluxo rode em background.

## Estrutura do Codigo Customizado (JSDoc)

O fluxo contem tratamento nativo de dados via JavaScript no no `Code`, garantindo que eventuais respostas invalidas do banco de dados em memoria nao quebrem a execucao:

```javascript
try {
    const rawData = $item(0).$node["Compara Get Memory1"].json["Redis2"];
    const mensagensArray = Array.isArray(rawData) ? rawData : JSON.parse(rawData);
    const mensagem_completa = mensagensArray.join(" ");
    
    return [{ json: { mensagem_completa: mensagem_completa } }];
} catch (error) {
    return [{ json: { mensagem_completa: "", log_erro: "Falha ao processar dados do Redis: " + error.message } }];
}
```
## Consideracoes Finais

Este projeto foi documentado visualmente utilizando o recurso de Sticky Notes nativo do n8n para facilitar a leitura arquitetonica e a escalabilidade do codigo por outras equipes de desenvolvimento.
