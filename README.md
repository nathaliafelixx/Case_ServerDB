
#  Implantação Ubuntu Server para  Postgresql -Ambiente  Dev

05/04/2022
Nathalia Felix Rosa

## Objetivos
Criar e configurar um ambiente de servidor linux, para ambiente  de testes de uma aplicação X

## Especificações

### Requisitos do hardware/software Servidor: 
 * Memória Ram: 1G (Mínimo);
 * CPU: 2 vCPU;
 * Disco: 1 disco de 10 GB para boot;
 * Sistema operacional: Linux Ubuntu Server 20.04 LTS;
 * Serviço SSH habitado;
### Requisitos de instalação e configuração Banco de Dados**
 * Postgresql;
 * versão: 12;
 * 1 usuario com nome dev, capaz de executar cos comandos docker
 * Popular o banco de dados com 3 tabelas que se relacionem (FK), para a utilização em um supermercado;

### Requisitos  Configuração uma instância do PGADMIN

 - Instância em execução Container;
 - Acessível remotamente via IP;

 ### Detalhes da Instalação e configuração da VM
 
*Nesta etapa foi realizada a instalação da máquina virtual utilizando a ferramenta Virtualbox, utilizando  a ISO do S.O  e ajustados as configuraçoes conforme requisitos*

  **MEMORIA**  
 

       useroperacao@infraop:~$ lsmem
        RANGE                                 SIZE  STATE REMOVABLE BLOCK
        0x0000000000000000-0x000000003fffffff   1G online       yes   0-7
        
        Memory block size:       128M
        Total online memory:       1G
        Total offline memory:      0B


**CPU**

    useroperacao@infraop:~$ lscpu
    Architecture:                    x86_64
    CPU op-mode(s):                  32-bit, 64-bit
    Byte Order:                      Little Endian
    Address sizes:                   48 bits physical, 48 bits virtual
    CPU(s):                          2
    On-line CPU(s) list:             0,1
    Thread(s) per core:              1
    Core(s) per socket:              2
    Socket(s):                       1
    NUMA node(s):                    1
    Vendor ID:                       AuthenticAMD
    CPU family:                      16
    Model:                           10
    Model name:                      AMD Phenom(tm) II X4 960T Processor
    Stepping:                        0
    CPU MHz:                         3013.704
    BogoMIPS:                        6027.40
    Hypervisor vendor:               KVM
    Virtualization type:             full
    L1d cache:                       128 KiB
    L1i cache:                       128 KiB
    L2 cache:                        1 MiB
    L3 cache:                        6 MiB
    NUMA node0 CPU(s):               0,1
    

**DISCO**

    useroperacao@infraop:~$ sudo fdisk -l /dev/sda
    Disk /dev/sda: 10 GiB, 10737418240 bytes, 20971520 sectors
    Disk model: VBOX HARDDISK   
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 90ACE257-489F-4F07-B6F1-32618ABEE3B1
    
    Device       Start      End  Sectors  Size Type
    /dev/sda1     2048     4095     2048    1M BIOS boot
    /dev/sda2     4096  1861631  1857536  907M Linux filesystem
    /dev/sda3  1861632 20969471 19107840  9.1G Linux filesystem
    

**REDE**

    useroperacao@infraop:/$ cat /etc/netplan/00-installer-config.yaml 
    # This is the network config written by 'subiquity'
    network:
        ethernets:
         enp0s3:
          dhcp4: no
          addresses: [192.168.18.25/24]
          gateway4: 192.168.18.1
          nameservers: 
           addresses: [8.8.8.8,8.8.4.4]
        version: 2
    useroperacao@infraop:/$ 

    useroperacao@infraop:/$ sudo cat /etc/group | tail -8
    landscape:x:115:
    lxd:x:116:useroperacao
    systemd-coredump:x:999:
    useroperacao:x:1000:
    ssl-cert:x:117:postgres
    postgres:x:118:
    docker:x:998:dev
    dev:x:1001:
    useroperacao@infraop:/$
 
##  Detalhes da Instalação e configuração das aplicações

     ***Com a maquina virtual configurada e o Ubuntu Server instalado, nesta etapa foi realizada a instalação e configuração das aplicaçoes indicadas nos reuisitos *** 

 **Docker**

    useroperacao@infraop:~$ sudo dpkg -l | grep Docker
    ii  docker-ce                             5:20.10.14~3-0~ubuntu-focal        amd64        Docker: the open-source application container engine
    ii  docker-ce-cli                         5:20.10.14~3-0~ubuntu-focal        amd64        Docker CLI: the open-source application container engine
    ii  docker-ce-rootless-extras             5:20.10.14~3-0~ubuntu-focal        amd64        Rootless support for Docker.
    ii  docker-scan-plugin                    0.17.0~ubuntu-focal                amd64        Docker scan cli plugin.

    docker run -p 80:80 \
        --name PGADMIN \
        -e 'PGADMIN_DEFAULT_EMAIL=operacao@infraop.com' \
        -e 'PGADMIN_DEFAULT_PASSWORD=SuperSecret' \
        -d dpage/pgadmin4

  **postgresql**
  

    useroperacao@infraop:~$ sudo dpkg -l | grep postgresql
    ii  pgdg-keyring                          2018.2                             all          keyring for apt.postgresql.org
    ii  postgresql-12                         12.10-1.pgdg20.04+1+b1             amd64        The World's Most Advanced Open Source Relational Database
    ii  postgresql-client-12                  12.10-1.pgdg20.04+1+b1             amd64        front-end programs for PostgreSQL 12
    ii  postgresql-client-common              238.pgdg20.04+1                    all          manager for multiple PostgreSQL client versions
    ii  postgresql-common                     238.pgdg20.04+1                    all          PostgreSQL database-cluster manager
    useroperacao@infraop:~$ 


    useroperacao@infraop:/$ sudo cat /etc/postgresql/12/main/pg_hba.conf | tail -25|grep -v '^$'
    local   all             postgres                                peer
    # TYPE  DATABASE        USER            ADDRESS                 METHOD
    # "local" is for Unix domain socket connections only
    local   all             all                                     peer
    # IPv4 local connections:
    host    all             all             127.0.0.1/32            trust
    host    all             all            192.168.18.0/24		md5
    host    all             all            172.17.0.0/16		md5 
    # IPv6 local connections:
    host    all             all             ::1/128                 md5
    # Allow replication connections from localhost, by a user with the
    # replication privilege.
    local   replication     all                                     peer
    host    replication     all             127.0.0.1/32            md5
    host    replication     all             ::1/128                 md5
    useroperacao@infraop:/$ 

    useroperacao@infraop:/etc/postgresql/12/main$ cat  postgresql.conf | less
    # CONNECTIONS AND AUTHENTICATION
    #------------------------------------------------------------------------------

    # - Connection Settings -

    listen_addresses = '*'          # what IP address(es) to listen on;
                                            # comma-separated list of addresses;
                                            # defaults to 'localhost'; use '*' for all
                                            # (change requires restart)
    port = 5432                             # (change requires restart)
    max_connections = 100                   # (change requires restart)

    
**instancia Container**

     ***Nesta etapa foi inicalizada uma instancia Container via Docker com a imagem do Pgadmin *** 
        
     
     
    dev@infraop:~$ docker ps
    CONTAINER ID   IMAGE            COMMAND            CREATED        STATUS        PORTS                                        NAMES
    02e7e301157e   dpage/pgadmin4   "/entrypoint.sh"   42 hours ago   Up 19 hours   0.0.0.0:80->80/tcp, :::80->80/tcp, 443/tcp   PGADMIN
    dev@infraop:~$
    
    dev@infraop:~$ docker image ls
    REPOSITORY       TAG       IMAGE ID       CREATED        SIZE
    dpage/pgadmin4   latest    4b5bbddb3624   3 weeks ago    340MB
    hello-world      latest    feb5d9fea6a5   6 months ago   13.3kB

    dev@infraop:~$  docker container inspect 02e
     "Config": {
                "Hostname": "02e7e301157e",
                "Domainname": "",
                "User": "pgadmin",
                "AttachStdin": false,
                "AttachStdout": false,
                "AttachStderr": false,
                "ExposedPorts": {
                    "443/tcp": {},
                    "80/tcp": {}
                },
                "Tty": false,
                "OpenStdin": false,
                "StdinOnce": false,
                "Env": [
                    "PGADMIN_DEFAULT_EMAIL=*******@infraop.com",
                    "PGADMIN_DEFAULT_PASSWORD=' **********'
                    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                    "PYTHONPATH=/pgadmin4"
                ],
                "Cmd": null,
                "Image": "dpage/pgadmin4",
                "Volumes": {
                    "/var/lib/pgadmin": {}
                },
                "WorkingDir": "/pgadmin4",
                "Entrypoint": [
                    "/entrypoint.sh"
                ],
                "OnBuild": null,
                "Labels": {}
            },
            dev@infraop:~$  docker container inspect 02e

 ## Detalhes das configuraçoes Postgres v12
 
 **Database**

    -- Database: dab_supermercadoX    
    -- DROP DATABASE IF EXISTS "dab_supermercadoX";    
    CREATE DATABASE "dab_supermercadoX"
        WITH 
        OWNER = postgres
        ENCODING = 'UTF8'
        LC_COLLATE = 'pt_PT.UTF-8'
        LC_CTYPE = 'pt_PT.UTF-8'
        TABLESPACE = pg_default
        CONNECTION LIMIT = -1;
 **Tabelas**

    -- Table: public.tb_categoria_produto    
    -- DROP TABLE IF EXISTS public.tb_categoria_produto;    
    CREATE TABLE IF NOT EXISTS public.tb_categoria_produto
    (
        cod_cat_produto integer NOT NULL,
        nome_categoria character varying COLLATE pg_catalog."default" NOT NULL,
        CONSTRAINT categoria_produto_pkey PRIMARY KEY (cod_cat_produto)
    )
---------
            TABLESPACE pg_default;    
            ALTER TABLE IF EXISTS public.tb_categoria_produto
                OWNER to postgres;
        -- Table: public.tb_vendas    
        -- DROP TABLE IF EXISTS public.tb_vendas;    
        CREATE TABLE IF NOT EXISTS public.tb_vendas
            (
            cod_venda integer[] NOT NULL,
            data_venda date NOT NULL,
            quantidade integer NOT NULL,
            codigo_produto integer NOT NULL,
            CONSTRAINT tb_vendas_pkey PRIMARY KEY (cod_venda),
            CONSTRAINT codigo_produto_fk FOREIGN KEY (codigo_produto)
                REFERENCES public.tb_cadastro_produto (cod_produto) MATCH SIMPLE
                ON UPDATE NO ACTION
                ON DELETE NO ACTION
                NOT VALID
        )
----------------        
        TABLESPACE pg_default;    
        ALTER TABLE IF EXISTS public.tb_vendas
            OWNER to postgres;
    
    -- Table: public.tb_cadastro_produto
    
    -- DROP TABLE IF EXISTS public.tb_cadastro_produto;
    
    CREATE TABLE IF NOT EXISTS public.tb_cadastro_produto
    (
        cod_produto integer NOT NULL,
        nome_produto character varying COLLATE pg_catalog."default" NOT NULL,
        preco_produto double precision NOT NULL,
        cod_categoria_produto integer NOT NULL,
        CONSTRAINT cadastro_produto_pkey PRIMARY KEY (cod_produto),
        CONSTRAINT cod_categoria_produto_fk FOREIGN KEY (cod_categoria_produto)
            REFERENCES public.tb_categoria_produto (cod_cat_produto) MATCH SIMPLE
            ON UPDATE NO ACTION
            ON DELETE NO ACTION
            NOT VALID
    )
    
    TABLESPACE pg_default;
    
    ALTER TABLE IF EXISTS public.tb_cadastro_produto
        OWNER to postgres;
        
   ## Diagrama de Entidade e Relacionamento
![Diagrama de Entidade e Relacionamento](https://github.com/nathaliafelixx/Case_estagio/raw/main/imagem/diagramaDB.png)   
   
   
