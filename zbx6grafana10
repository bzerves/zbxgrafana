#!/bin/bash

# Solicita a senha do Zabbix
read -s -p "Digite a senha para o usuário zabbix no PostgreSQL: " ZABBIX_PASSWORD
echo

# Exibe a mensagem de progresso para o usuário
echo "O script está instalando tudo, aguarde por favor, enquanto isso, entra no grupo do Telegram da Zerves Comunidade!"

# Instala o wget (se necessário)
apt update
apt install -y wget

# Baixa e instala o repositório oficial do Zabbix 6 LTS
cd /tmp/
wget https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_6.0-5+debian12_all.deb
apt install -y ./zabbix-release_6.0-5+debian12_all.deb

# Atualiza o sistema
apt update
apt upgrade -y
apt update

# Instala os pacotes do Zabbix
apt install -y zabbix-server-pgsql zabbix-frontend-php php-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent

# Cria a base de dados e o usuário no PostgreSQL
su - postgres -c "createuser --pwprompt zabbix" <<EOF
$ZABBIX_PASSWORD
$ZABBIX_PASSWORD
EOF

su - postgres -c "createdb -O zabbix zabbix" <<EOF
$ZABBIX_PASSWORD
EOF

# Importa o esquema inicial e os dados no PostgreSQL
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | psql -U zabbix -d zabbix <<EOF
$ZABBIX_PASSWORD
EOF

# Edita o arquivo de configuração do Zabbix Server para informar os dados de conexão com o PostgreSQL
sed -i "s/^# DBPassword=/DBPassword=$ZABBIX_PASSWORD/" /etc/zabbix/zabbix_server.conf

# Edita o arquivo de configuração do PHP-FPM para definir o fuso horário correto
sed -i "s/^;php_value\[date.timezone\] =/php_value[date.timezone] = America\/Sao_Paulo/" /etc/zabbix/php-fpm.conf

# Edita o arquivo de configuração do Nginx (opcionalmente, você pode customizar esse arquivo)
vim /etc/nginx/conf.d/zabbix.conf

# Habilita o Zabbix Server e o Zabbix Agent para iniciar durante o boot do sistema
systemctl enable zabbix-server zabbix-agent

# Ajusta o tempo máximo de execução no PHP (opcionalmente, você pode ajustar outros parâmetros do php.ini)
sed -i "s/^max_execution_time = 30/max_execution_time = 600/" /etc/php/8.2/fpm/php.ini

# Reinicia os serviços necessários
systemctl restart zabbix-server zabbix-agent nginx php8.2-fpm

# Instala o Grafana e o Plugin Zabbix
apt install -y gnupg2
wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | tee -a /etc/apt/sources.list.d/grafana.list
apt update
apt install -y grafana
grafana-cli plugins install alexanderzobnin-zabbix-app
grafana-cli plugins update-all

# Configura o Grafana para iniciar durante o boot do sistema e inicia o serviço
systemctl daemon-reload
systemctl enable grafana-server
systemctl start grafana-server

# Finaliza a instalação exibindo uma mensagem para o usuário
echo "A instalação do Zabbix e do Grafana foi concluída com sucesso!"
echo "Não esqueça de atualizar periodicamente os plugins do Grafana com 'grafana-cli plugins update-all'."
