# Weather-Bot • WhatsApp + n8n (v1.0)

MVP desenvolvido para o **Desafio ITECH 2025**.  
Permite que o usuário envie **texto ou áudio** pelo WhatsApp, receba a previsão do tempo (atual **ou até 5 dias à frente**) e opte por alertas automáticos.

---

## Visão geral

- O fluxo principal opera como um agente inteligente, usando LLM para interpretar mensagens (texto **e** áudio) e acionar as ferramentas corretas.  
- Estrutura modular: 1 workflow principal + 2 sub-workflows. O subworkflow 3 (alertas) é independente e só roda para usuários inscritos.  
- Respostas humanizadas, com linguagem natural e emojis.  
- Persistência de dados e memória de conversa via Supabase/Postgres.  
- Integração WhatsApp feita por Webhook Evolution API (suporta áudio via transcrição).

---

## Componentes do sistema

### 1 – Agente_Clima_Tempo (workflow principal)

Ponto de entrada via Webhook Evolution. Recebe mensagens **de texto ou áudio** e:

1. Detecta se o usuário quer previsão **agora** ou para algum dia futuro (até 5 dias).  
2. Converte áudio em texto quando necessário.  
3. Chama a ferramenta adequada (*Current Weather* ou *5-Day Forecast*).  
4. Responde com temperatura, condição do céu, umidade e chance de chuva.  
5. Salva `phone`, `city`, `active=false` em Postgres.  
6. Pergunta se o usuário quer alertas automáticos.

### 2 – Subworkflow Get_Weather

- Consulta a OpenWeatherMap (Current ou 5-Day).  
- Para pedidos futuros: consolida e devolve **resumo de 5 dias**.  
- Passa por LLM simples para gerar texto amigável.  
- Envia a resposta de volta ao workflow principal.

### 3 – Subworkflow Get_Weather_ALERT

Executado em intervalos fixos.

1. Filtra usuários com `active = true`.  
2. Reconsulta a API de clima para cada cidade.  
3. Se encontrar `Thunderstorm`, `Squall` ou `Tornado`, envia alerta automático pelo WhatsApp.

---

## Persistência e estrutura de dados

| coluna | descrição |
|--------|-----------|
| **phone** | número WhatsApp |
| **city**  | cidade consultada |
| **active**| `true` se o usuário aceitou receber alertas |

A memória do chat usa **Postgres Chat Memory** (janela curta).

---

## Tecnologias

- **n8n v1.101.1**  
- **GPT-4o** e **GPT-4.1 mini**  
- **OpenWeatherMap API 3.0**  
- **Supabase** (Postgres + Storage)  
- **Evolution API** (WhatsApp texto + áudio)  
- **Postgres Chat Memory**

---

## Como testar

1. Envie o nome de uma cidade (texto ou áudio):  Teste aqui --> https://wa.me/11992111544?text=Oi!+Quero+saber+o+clima` 
2. Receba a previsão atual. Use “amanhã”, “domingo”, etc. para pedir futuro.  
3. Bot pergunta sobre alertas. Responda **SIM** ou **NÃO**.  
4. Caso **SIM**, `active` vira `true` e alertas passam a ser monitorados.  
5. Alertas automáticos chegam se ocorrer clima severo.  
6. Perguntas curtas de conhecimento geral retornam resumo da Wikipedia.

---

## Teste manual de alerta

No subworkflow **Get_Weather_ALERT**, abra o nó **Dados importantes** e mude `main` para:
- `Thunderstorm`
- `Squall`
- `Tornado`
Isso dispara o alerta como se existisse evento crítico.

---

## Como o bot funciona

- Aceita **texto ou áudio** (áudio → transcrição).  
- Suporta previsão atual **e** até **5 dias** à frente; basta incluir “amanhã”, “depois de amanhã”, dia da semana ou “daqui a X dias”.  
- Não precisa de comandos complexos: apenas cidade [+ dia].  
- Responde com temperatura, mín/máx (quando futuro), condição do céu, umidade e probabilidade de chuva.

---

## Ideias para próxima versão

- Botões rápidos (Quick Reply) para opt-in/out.  
- Envio diário da previsão pela manhã.  
- Suporte multilíngue com detecção automática de idioma.

---

Feito com criatividade e propósito em n8n por **Georges S. Azzi** para a **ITECH360**.  
Agradecimentos ao Gabriel pela oportunidade. Sugestões e PRs são bem-vindos!
