FROM debian:12.11
# Base da imagem. Isto é, vai usar o debian como pilar para construir essa imagem.

RUN apt update && apt install -y \
    mariadb-server
# Atualizando os pacotes do debian e faz a instalação do mariadb.

# copia configs e entrypoint
COPY conf/50-server.cnf /etc/mysql/mariadb.conf.d/.
# Esse dir é onde o Mariadb lê configurações adicionais.
COPY tools/mariadb_script.sh .
# Copiando alguns arquivos de configuração presentes no Host, para dentro da imagem.
# Quando subirmos um container com esse build, teremos esses mesmos arquivos e configurações dentro do container.

RUN mkdir /run/mysqld && chmod +x mariadb_script.sh
# Criando um dir necessário para o Mariadb rodar em um container.
# Dando os devidos privilégios para rodar o script do mariadb

EXPOSE 3306
# Estamos indicando que exporemos a porta 3306 do container
# Essa é a porta onde o mariadb vai ouvir.
# Aqui ainda não expusemos a porta nem fizemos a ligação com a porta do Host.

CMD ["./mariadb_script.sh"]
# Comando que o processo executará após subir o container.
