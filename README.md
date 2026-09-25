# Prospecção local com n8n e Evolution API

Base local para uma automação de prospecção de sites WordPress usando n8n, Evolution API v2, PostgreSQL e Redis. O projeto não inclui serviços pagos nem envia mensagens por conta própria até que as credenciais e a instância sejam configuradas.

## Aviso de conformidade

Use somente com contatos que tenham uma base legal para receber a comunicação, respeite a LGPD, os termos do WhatsApp e o pedido de opt-out. A automação não deve ser usada para spam. Uma resposta do cliente interrompe qualquer follow-up e deve ser tratada por uma pessoa.

## Pré-requisitos

- Docker Engine com Docker Compose v2.
- Uma planilha Google com a aba `Limeira` e cabeçalhos na primeira linha.
- Uma credencial OAuth2 ou Service Account configurada no n8n com acesso à planilha.
- Um número WhatsApp dedicado e uma instância da Evolution API.

## Instalação

```bash
cp .env.example .env
openssl rand -hex 32
# use valores diferentes para POSTGRES_PASSWORD, REDIS_PASSWORD,
# N8N_ENCRYPTION_KEY e AUTHENTICATION_API_KEY no .env
docker compose config
docker compose up -d
```

Abra `http://localhost:5678`, crie o usuário proprietário e importe os dois arquivos em `workflows/`. O Compose repassa `AUTHENTICATION_API_KEY` ao n8n como `EVOLUTION_API_KEY`; não é necessário criar uma segunda chave. Substitua a credencial `Google Sheets account` no n8n e não coloque segredos no JSON exportado.

## Conectar o WhatsApp

```bash
curl -X POST http://localhost:8080/instance/create \
  -H "apikey: $(grep '^AUTHENTICATION_API_KEY=' .env | cut -d= -f2-)" \
  -H 'Content-Type: application/json' \
  -d '{"instanceName":"prospeccao","integration":"WHATSAPP-BAILEYS"}'
```

Use o endpoint de conexão da documentação da Evolution API para obter o QR Code e escaneie-o pelo WhatsApp do chip dedicado. Configure o webhook da instância para `http://n8n:5678/webhook/evolution-respostas` dentro da rede Docker, assinando os eventos `MESSAGES_UPSERT`.

## Colunas esperadas

Na primeira linha da aba `Limeira`, use estes cabeçalhos: `Empresa / Negócio`, `Nicho / Categoria`, `Telefone / WhatsApp`, `Status do Site Atual`, `Status da Abordagem`, `Data do Contato`, `Observações`, `Status do projeto`. O fluxo usa `Telefone / WhatsApp` e `Status da Abordagem` como campos obrigatórios.

## Fluxos

- `n8n_prospeccao_main.json`: executa às 9h, 10h, ... 18h em dias úteis, seleciona um contato elegível, aguarda de 2 a 5 minutos, envia a mensagem e atualiza a linha.
- `n8n_webhook_resposta.json`: recebe `MESSAGES_UPSERT`, ignora mensagens próprias, localiza o telefone e marca `Cliente Respondeu`.

O filtro do fluxo principal aceita `Site` igual a `Sem site` ou `Oferecer Site novo` e status vazio ou `Aguardando contato`. A mensagem e os delays devem ser revisados antes de ativar o workflow.

## Checklist anti-banimento e operação responsável

- Comece com no máximo 15 disparos no primeiro dia; suba no máximo 5 por dia até um teto conservador de 40 a 50 por dia.
- O fluxo envia no máximo um contato por execução agendada. Para aumentar volume, prefira ajustar conscientemente o limite e monitorar respostas, erros e bloqueios.
- Mantenha o intervalo aleatório de 2 a 5 minutos e evite mensagens idênticas; personalização não substitui consentimento.
- Pare imediatamente diante de reclamações, opt-outs, bloqueios ou queda de qualidade. O webhook marca respostas e impede follow-ups futuros.
- Use um chip pré-pago secundário, exclusivo para a operação, aquecido e maturado gradualmente com uso legítimo. Não use o número pessoal ou principal.
- Nunca exponha chaves da Evolution, senhas, tokens OAuth ou JSON de Service Account no Git, logs, screenshots ou frontend.
- Faça backup dos volumes e restrinja `localhost`; não publique as portas na internet sem TLS, autenticação e proxy reverso.

## Diagnóstico

```bash
docker compose ps
docker compose logs -f n8n
docker compose logs -f evolution-api
docker compose down
```

Não use `docker compose down -v` em produção local: isso remove os volumes e as sessões persistidas.