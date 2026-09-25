# Evolution API

A configuracao da Evolution API e feita pelas variaveis do `docker-compose.yml`. Esta pasta e mantida para arquivos de configuracao versionaveis e documentacao; credenciais e sessoes ficam exclusivamente no volume Docker `evolution_instances`.

Depois de subir os containers, crie uma instancia pelo endpoint da Evolution API e escaneie o QR Code:

```bash
curl -X POST http://localhost:8080/instance/create \
  -H 'Content-Type: application/json' \
  -H 'apikey: SUA_AUTHENTICATION_API_KEY' \
  -d '{"instanceName":"prospeccao","integration":"WHATSAPP-BAILEYS"}'
```

Consulte a documentacao da versao instalada antes de automatizar endpoints administrativos.