**Bot de Clima no Telegram com n8n**

Projeto desenvolvido para o desafio prático de criação de um chatbot no Telegram utilizando n8n e a API da OpenWeather. O bot recebe o nome de uma cidade, consulta as condições meteorológicas atuais e responde com a temperatura em graus Celsius.

**Funcionalidades**

- Recebe mensagens pelo Telegram.
- Captura e normaliza a entrada na variável queue.
- Consulta a API OpenWeather.
- Retorna a temperatura atual em °C.
- Valida o status HTTP e os campos esperados.
- Trata cidades inexistentes ou não localizadas.
- Mantém tokens e chaves fora do workflow exportado.
- Executa em infraestrutura própria (VPS)

Exemplo de sucesso:

🌤️ A temperatura em Belo Horizonte é de 19°C.

Exemplo de erro:

❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).

**Arquitetura do workflow**

<img width="1362" height="410" alt="image" src="https://github.com/user-attachments/assets/3b3a755c-4940-41e4-b38a-cb1c985453e0" />


Foi utilizado os seguintes nós:

- Telegram Trigger --- recebe mensagens enviadas ao bot.
- Preparar Entrada --- trata a mensagem e armazena a consulta em queue.
- Consultar OpenWeather --- realiza a requisição HTTP.
- Cidade encontrada? --- valida o status da resposta e a existência da temperatura.
- Preparar Sucesso --- monta a mensagem com cidade e temperatura.
- Preparar Erro --- monta a mensagem para cidade não encontrada.
- Enviar Resposta Telegram --- devolve a resposta ao usuário.

**Tela do Bot**
<img width="738" height="1600" alt="teste-conversa-telegram" src="https://github.com/user-attachments/assets/88868617-784b-489a-af4f-bde0039faf57" />


**Infraestrutura utilizada**

O projeto foi executado em uma VPS, com o n8n gerenciado pelo Coolify. O acesso externo utiliza domínio próprio sob proftalles.cloud, com DNS/proxy gerenciado pelo Cloudflare.

Telegram -> Internet -> Cloudflare -> Domínio/subdomínio em proftalles.cloud -> Coolify -> n8n na VPS ->OpenWeather API

Essa arquitetura disponibiliza o webhook HTTPS necessário para que o Telegram entregue as mensagens ao n8n.

IP da VPS, endereço administrativo, tokens e demais segredos não devem ser publicados no repositório.

 **Criação do bot no Telegram**

1 - O bot foi criado pelo BotFather:
2 - Abrir o BotFather no Telegram.
3 - Enviar /newbot.
4 - Definir o nome do bot.
5 - Criar um username terminado em bot.
6 - Guardar o token fornecido.
7 - Cadastrar o token como credencial do Telegram no n8n.

Evidência da criação
<img width="814" height="1600" alt="criacao-bot-telegram" src="https://github.com/user-attachments/assets/0ccd8469-5022-48c3-9e6a-b1499091c97c" />


**Credenciais**
Telegram

No n8n, crie uma credencial Telegram API e informe o token fornecido pelo BotFather. Selecione a credencial nos nós Telegram Trigger e
Enviar Resposta Telegram.

TELEGRAM_BOT_TOKEN - Na implementação, o token pode permanecer protegido no sistema de credenciais do n8n.

OpenWeather - A variável esperada é: OPENWEATHER_API_KEY

Endpoint: https://api.openweathermap.org/data/2.5/weather

Parâmetros:
Parâmetro   Valor
q         {{ $json.queue }}
units     metric
lang      pt_br
appid     {{ $env.OPENWEATHER_API_KEY }}

A variável interna é chamada queue, enquanto o parâmetro enviado à API para a consulta da localidade é q.

**Coolify**

No serviço/container do n8n no Coolify, cadastre:
OPENWEATHER_API_KEY=SUA_CHAVE_OPENWEATHER

Depois, aplique a alteração/redeploy para que a variável esteja disponível no container.

**Importando o workflow**

1 - Baixe ou clone o repositório.
2 - Abra o n8n.
3 - Importe workflow-chatbot-telegram.json.
4 - Selecione a credencial do Telegram nos nós correspondentes.
5 - Confirme a variável OPENWEATHER_API_KEY no ambiente do n8n.
5 - Salve e ative o workflow.

*Requisição OpenWeather*

O nó Consultar OpenWeather realiza: GET https://api.openweathermap.org/data/2.5/weather
A chave é enviada no parâmetro appid; não é necessário usar Authorization Header.

A validação do fluxo verifica de forma equivalente:

{{ $json.statusCode === 200 && $json.body?.main?.temp !== undefined }}

Se a condição for verdadeira, o fluxo prepara a resposta com a temperatura. Caso contrário, envia a mensagem de cidade não encontrada.

**Testes realizados**

**Teste 1 --- Belo Horizonte**
<img width="1852" height="927" alt="teste-belo-horizonte" src="https://github.com/user-attachments/assets/9542d9f8-d73b-4c41-8ae0-75261dd5e311" />

A consulta foi processada corretamente e o nó Telegram confirmou o envio da resposta.


**Teste 2 --- Timóteo**
<img width="1852" height="932" alt="teste-timoteo" src="https://github.com/user-attachments/assets/ed4b9c28-d8cb-443e-b37f-b241031baaf2" />

A resposta da OpenWeather trouxe a temperatura e o bot enviou a mensagem corretamente.

**Teste 3 --- cidade inexistente**
<img width="1860" height="927" alt="teste-cidade-inexistente" src="https://github.com/user-attachments/assets/35b5624d-fb92-44ef-858c-ba6cd47fa16e" />

Uma entrada inválida foi utilizada para testar o caminho de erro. O fluxo seguiu pela saída falsa e enviou a mensagem de cidade não encontrada.


**Teste no aplicativo Telegram**

Foram testadas Belo Horizonte, Timóteo e Lavras, além de uma entrada inexistente. A conversa demonstra as respostas de sucesso e o tratamento de erro.
<img width="738" height="1600" alt="teste-conversa-telegram" src="https://github.com/user-attachments/assets/598d0dfc-ef1d-43aa-a9cc-a5cd418f3f60" />


📁 Estrutura do repositório

chatbot-telegram/
├── workflow-chatbot-telegram.json
├── README.md
└── assets/
    ├── criacao-bot-telegram.jpeg
    ├── teste-belo-horizonte.png
    ├── teste-timoteo.png
    ├── teste-cidade-inexistente.png
    └── teste-conversa-telegram.jpeg



**Tecnologias**

- n8n
- VPS
- Coolify

Cloudflare

domínio próprio em proftalles.cloud
