# Infraestrutura de Observabilidade

Este repositório contém a configuração de uma stack de monitoramento com **RabbitMQ**, **Prometheus**, **Grafana** e o **RabbitMQ Exporter** para exportação de métricas.

## Serviços incluídos

### RabbitMQ
- **Imagem**: `rabbitmq:3.12-management`
- **Portas**:
  - AMQP: `5672`
  - Painel de gerenciamento: `http://localhost:15672`
  - Métricas (plugin Prometheus): `http://localhost:15692/metrics`
- **Credenciais padrão**:
  - Usuário: `guest`
  - Senha: `guest`
- **Importante**: para habilitar o endpoint de métricas nativo do RabbitMQ, é necessário ativar o plugin do Prometheus:

```bash
docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_prometheus
```

Esse comando pode ser executado após os containers estarem rodando.

### RabbitMQ Exporter
- **Imagem**: `kbudde/rabbitmq-exporter`
- **Porta de métricas Prometheus**: `http://localhost:9419/metrics`
- **Conecta-se a**: `http://guest:guest@rabbitmq:15672` para extrair métricas
- Pode ser usado como alternativa ou complemento ao plugin nativo.

### Prometheus
- **Imagem**: `prom/prometheus`
- **Interface Web**: `http://localhost:9090`
- **Responsável por coletar as métricas do RabbitMQ Exporter**

### Grafana
- **Imagem**: `grafana/grafana`
- **Interface Web**: `http://localhost:3000`
- **Credenciais padrão**:
  - Usuário: `admin`
  - Senha: `admin`
- **Usado para visualização gráfica das métricas via dashboards personalizados**

## Como subir os serviços

Certifique-se de que o Docker e o Docker Compose estão instalados na sua máquina.

Suba os containers com o comando:

```bash
docker-compose up -d
```

Verifique se todos os serviços estão rodando:

```bash
docker ps
```

Habilite o plugin Prometheus no RabbitMQ:

```bash
docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_prometheus
```

## Como parar os serviços

Para parar e remover os containers:

```bash
docker-compose down
```

Para remover também os volumes persistentes:

```bash
docker-compose down -v
```

## Estrutura esperada do repositório

```
.
├── docker-compose.yml
└── prometheus.yml
```

> ⚠️ Certifique-se de que o arquivo `prometheus.yml` está corretamente configurado para apontar para o `rabbitmq-exporter`.

## Exemplo de configuração do `prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'rabbitmq'
    static_configs:
      - targets: ['rabbitmq-exporter:9419']
```

> Caso prefira utilizar o endpoint nativo do plugin Prometheus do RabbitMQ, adicione:
>
> ```yaml
>   - job_name: 'rabbitmq-native'
>     static_configs:
>       - targets: ['rabbitmq:15692']
> ```