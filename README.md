# Weather-Bot • WhatsApp + n8n (v1.0)

MVP desenvolvido para o Desafio ITECH 2025.  
Permite que o usuário envie uma cidade via WhatsApp, receba a previsão do tempo e opte por alertas automáticos.

## Visão geral

- O fluxo principal opera como um agente inteligente, utilizando LLM para interpretar mensagens e acionar as ferramentas certas.
- A estrutura é modular, com um workflow principal e dois sub-workflows. O subworkflow 3 (alertas) é totalmente independente e ativado apenas quando o usuário opta por receber notificações.
- As respostas são humanizadas e formatadas com emojis e linguagem natural.
- Há persistência de dados e memória conversacional usando Supabase/Postgres.
- A integração com o WhatsApp é feita via Webhook da Evolution API.

## Componentes do sistema

1 - Agente_Clima_Tempo (workflow principal):
É o ponto de entrada via Webhook Evolution. Recebe mensagens dos usuários e utiliza um agente inteligente que:
- Detecta se o usuário quer uma previsão do tempo.
- Aciona a ferramenta correspondente.
- Responde ao usuário com informações como temperatura, condições do céu, umidade e se há chance de chuva.
- Formata a resposta de forma natural e amigável.
- Salva o número do usuário e a cidade em um banco de dados com a coluna "active" como false por padrão(isso será usado para as notificações).
- Pergunta se o usuário deseja receber alertas no futuro.

2 - Subworkflow Get_Weather:
Responsável por:
- Consultar a API OpenWeatherMap com base na cidade enviada.
- Retornar dados atualizados sobre o clima.
- Passar essas informações por um LLM simples que as transforma em mensagens diretas e compreensíveis.
- Devolve as informações necessárias para o Workflow 1 - Agente_Clima_Tempo.

3 - Subworkflow Get_Weather_ALERT:
Executado automaticamente em intervalos definidos.
- Filtra usuários com "active = true" no banco de dados, caso tenham optado por receber as notificações.
- Reconsulta a API OpenWeatherMap para cada cidade.
- Se encontrar condições climáticas severas (ex: Thunderstorm, Squall, Tornado), envia um alerta automático para o usuário via WhatsApp.

## Persistência e estrutura de dados

O sistema utiliza Supabase/Postgres para armazenar os dados dos usuários.  
A estrutura inclui:
- Número do WhatsApp do usuário  
- Cidade consultada  
- Campo booleano `active`, indicando se o usuário optou por receber alertas

Quando o usuário responde "sim" à pergunta sobre receber notificações, o sistema atualiza o campo `active` para true. Esse campo é usado como filtro para o disparo dos alertas automáticos.

A memória da IA é armazenada usando o recurso Chat Postgres Memory, com janela limitada para manter a performance do banco.

## Tecnologias utilizadas

- n8n v1.101.1
- GPT-4o e GPT-4.1 mini (com uso de agente e LLM simples)
- OpenWeatherMap API 3.0
- Supabase (banco de dados Postgres + API)
- Evolution API para comunicação via WhatsApp
- Postgres Chat Memory para persistência de contexto

## Como testar

1. O usuário envia o nome de uma cidade no WhatsApp. (teste por aqui: https://wa.me/11992111544?text=Oi!+Quero+saber+o+clima )
2. O sistema responde com a previsão atual e pergunta se deseja receber alertas futuros.
3. Se o usuário responder "sim", o campo `active` é atualizado no banco e o subworkflow de alertas passa a monitorar aquela cidade.
4. Quando uma condição extrema é detectada, um alerta é enviado automaticamente.
5. O usuário também pode enviar perguntas gerais e receber respostas curtas da Wikipedia.

## Como testar manualmente o alerta

Você pode forçar o envio de um alerta no subworkflow 3 (Get_Weather_ALERT) para fins de teste.  
Basta abrir o workflow e localizar o nó chamado "Dados importantes".  
Dentro do output desse nó, altere o campo `main` para um dos seguintes valores:

- `Thunderstorm`  
- `Squall`  
- `Tornado`

Assim que esse dado for interpretado na lógica condicional, o alerta será ativado e enviado via WhatsApp como se houvesse um evento climático crítico na cidade monitorada.

## Possíveis melhorias para versão futura

- Botões rápidos (Quick Reply) para facilitar o opt-in de alertas  
- Forecast diário enviado automaticamente pela manhã  
- Suporte multilíngue com detecção automática de idioma

---

Feito com n8n, criatividade e propósito por Georges S. Azzi para a ITECH360.
Agradeço ao Gabriel pela oportunidade e estou aberto a sugestões ou contribuições.
