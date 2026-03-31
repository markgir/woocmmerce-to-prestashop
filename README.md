# wc2ps-saas — WooCommerce → PrestaShop Migration SaaS

Plataforma SaaS de migração WooCommerce → PrestaShop. Servidor dedicado processa migrações via agent leve instalado no hosting do cliente. Interface web para monitorização em tempo real.

## Arquitectura

```
Cliente (hosting partilhado)          Servidor dedicado (VPS)
─────────────────────────────         ────────────────────────
  wc2ps-agent.php                       worker.php (loop)
  ├── BD WooCommerce (localhost)    ←→   JobRunner.php
  └── BD PrestaShop  (localhost)         MigrationEngine.php
                                         Database.php (BD central)
                                         AgentClient.php (HTTPS)

Browser (qualquer dispositivo)
──────────────────────────────
  dashboard/index.html
  ├── Landing + wizard 4 passos
  ├── Pagamento PayPal
  ├── Verificação do agent
  └── Monitorização em tempo real
```

## Fluxo completo

1. Cliente escolhe plano e paga via PayPal
2. Recebe tokens únicos para os dois agents (WC + PS)
3. Faz upload de `wc2ps-agent.php` para cada servidor via File Manager
4. Preenche credenciais das BDs na interface
5. Worker processa em background — cliente pode fechar o browser
6. Progresso visível em tempo real ao reabrir a interface

## Estrutura

```
wc2ps-saas/
├── agent/
│   ├── wc2ps-agent.php       ← ficheiro único para upload no cliente
│   └── download.php          ← gera agent com token embutido
├── dashboard/
│   ├── index.html            ← interface do cliente (landing + wizard)
│   └── api.php               ← REST API (jobs, logs, verificar agent)
├── payments/
│   ├── PayPalGateway.php     ← integração PayPal REST API v2
│   └── payment.php           ← endpoints create_order + capture
├── worker/
│   ├── worker.php            ← loop principal (lançar com nohup)
│   ├── JobRunner.php         ← executa um job
│   ├── MigrationEngine.php   ← motor adaptado para AgentClient
│   ├── AgentClient.php       ← comunica com agent via HTTPS
│   ├── Database.php          ← BD central (schema, logs, jobs)
│   ├── config.php            ← credenciais + PayPal + APP_URL
│   └── setup.php             ← instala schema + gera API key
└── install.sh                ← instalação automática no VPS
```

## Instalação no VPS

```bash
# Via SSH
unzip wc2ps-saas.zip
sudo bash install.sh
```

O script instala PHP, MariaDB, cria a BD, gera as credenciais e regista o worker como serviço systemd.

### Configuração manual pós-instalação

Edita `worker/config.php` e preenche:

```php
define('PAYPAL_CLIENT_ID', 'teu_client_id');
define('PAYPAL_SECRET',    'teu_secret');
define('PAYPAL_SANDBOX',   false);  // false em produção
define('APP_URL',          'https://migrashop.pt');
```

### Firewall — portas necessárias

| Porta | Protocolo | Estado | Para quê |
|-------|-----------|--------|---------|
| 443   | HTTPS     | Aberta | Interface + API + agent |
| 80    | HTTP      | Aberta | Redirect → 443 |
| 22    | SSH       | Restrita ao teu IP | Administração |
| 3306  | MySQL     | Fechada | Só acesso local |

## Planos

| Plano | Preço | Produtos |
|-------|-------|---------|
| Starter | €29 | até 500 |
| Pro | €49 | até 5.000 |
| Enterprise | €99 | ilimitado |

## Requisitos do VPS

- Ubuntu 20.04+ ou Debian 11+
- PHP 8.0+ CLI
- MariaDB 10.5+
- `allow_url_fopen = On`

## Requisitos no hosting do cliente

- PHP 7.4+
- `pdo_mysql`
- HTTPS activo

## Versão

**v1.0** — estrutura base completa

## Licença

MIT
