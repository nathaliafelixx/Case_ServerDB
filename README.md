
#  Implantação Ubuntu Server para  Postgresql -Ambiente  Dev

05/04/2022
Nathalia Felix Rosa

## Objetivo
Criar e configurar um ambiente de servidor linux, para ambiente  de testes de uma aplicação X

## Especificações

### Requisitos do hardware/software Servidor: 
 * Memória Ram: 1G (Mínimo);
 * CPU: 2 vCPU;
 * Disco: 1 disco de 10 GB para boot;
 * Sistema operacional: Linux Ubuntu Server 20.04 LTS;
 * Serviço SSH habitado;
### Requisitos de instalação e configuração Banco de Dados
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
   
   ***Nesta etapa foi inicalizada uma instancia Container via Docker com a imagem do Pgadmin ***

    useroperacao@infraop:~$ sudo dpkg -l | grep Docker
    ii  docker-ce                             5:20.10.14~3-0~ubuntu-focal        amd64        Docker: the open-source application container engine
    ii  docker-ce-cli                         5:20.10.14~3-0~ubuntu-focal        amd64        Docker CLI: the open-source application container engine
    ii  docker-ce-rootless-extras             5:20.10.14~3-0~ubuntu-focal        amd64        Rootless support for Docker.
    ii  docker-scan-plugin                    0.17.0~ubuntu-focal                amd64        Docker scan cli plugin.
--------------------------------------    
    docker run -p 80:80 \
    --name PGADMIN \
    -e 'PGADMIN_DEFAULT_EMAIL=operacao@infraop.com' \
    -e 'PGADMIN_DEFAULT_PASSWORD=SuperSecret' \
    -d dpage/pgadmin4  
-------------------------------------------  

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

 ## Interface Grafica Pgadmin
 
![Interface Grafica Pgadmin](https://github.com/nathaliafelixx/Case_estagio/raw/main/imagem/InterfaceWebPgadmin.png)

  **postgresql**
  
Pacotes de instalação:

    useroperacao@infraop:~$ sudo dpkg -l | grep postgresql
    ii  pgdg-keyring                          2018.2                             all          keyring for apt.postgresql.org
    ii  postgresql-12                         12.10-1.pgdg20.04+1+b1             amd64        The World's Most Advanced Open Source Relational Database
    ii  postgresql-client-12                  12.10-1.pgdg20.04+1+b1             amd64        front-end programs for PostgreSQL 12
    ii  postgresql-client-common              238.pgdg20.04+1                    all          manager for multiple PostgreSQL client versions
    ii  postgresql-common                     238.pgdg20.04+1                    all          PostgreSQL database-cluster manager
    useroperacao@infraop:~$ 
-------------------------------------------------------


    useroperacao@infraop:/$ sudo cat /etc/postgresql/12/main/pg_hba.conf | tail -25|grep -v '^$'
    local   all             postgres                                peer
    # TYPE  DATABASE        USER            ADDRESS                 METHOD
    # "local" is for Unix domain socket connections only
    local   all             all                                     peer
    # IPv4 local connections:
    host    all             all             127.0.0.1/32            trust #Este campo defini que ao ser acessado localmente, o usuario postgres nao necessite de autenticação (trust/confiavel)
    host    all             all            192.168.18.0/24		         md5 #Este campo defini que o postgres pode ser acessado de qualquer maquina da rede declarada,e solicita autenicação (autenticação md5)
    host    all             all            172.17.0.0/16		           md5 #Este campo defini que o postgres pode ser acessado de qualquer container da rede Docker declarada,e solicita autenicação (autenticação md5)
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
   
   #Comandos utizados:
   
       1 lshw -C network
       2  cd /etc/netplan/ && ls
       3  vim /etc/netplan/00-installer-config.yaml
       4  cp 00-installer-config.yaml 00-installer-config.bkp
       5  vim /etc/netplan/00-installer-config.yaml
       6  netplan apply
       7  ip a
       8  ip addr show
       9  clear
      10  telnet localhost 22
      11  reboot
      12  exit
      13  history 
      14  apt update && apt upgrade
      15  apt search postgresql
      16  clear
      17  wget --help
      18  clear
      19  wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
      20  echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      21  apt update
      22  sudo apt -y install postgresql-12 postgresql-client-12
      23  apt install bash-completion
      24  systemctl postgresql.services
      25  systemctl postgresql.service
      26  systemctl status postgresql.service
      27  telnet localhost 5432
      28  sudo su - postgres
      29  apt-get install     ca-certificates     curl     gnupg     lsb-release
      30  echo   "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      31    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      32  apt update
      33  sudo apt-get install docker-ce docker-ce-cli containerd.io
      34  sudo docker run hello-world
      35  adduser dev
      36  useradd dev
      37  passwd dev
      38  clear
      39  cat /etc/group
      40  usermod -a -G docker dev
      41  cd /etc/postgresql/9.3/main
      42  ls -l
      43  cd /etc/postgresql/12/main/
      44  ls -l
      45  ip a
      46  history 
      47  history >comandospojTaticRoot
      48  cat comandospojTaticRoot 
      49  exit
      50  cd ..
      51  passwd
      52  exit
      53  ls
      54  cd ..
      55  ls
      56  history 
      57  uname -r
      58  uname -a
      59  lshw
      60  lshw | disk
      61  lshw | grep disk
      62  hwinfo
      63  apt install hwinfo
      64  hwinfo
      65  lscpu
      66  clear
      67  lscpu
      68  lsmemory
      69  lsmem
      70  lsdisk
      71  cat /proc/cpuinfo 
      72  sudo lshw -class CPU
      73  cd /etc/
      74  ls -l
      75  fdisk -l
      76  sudo dpkg -l.
      77  sudo dpkg -l
      78  sudo dpkg -l | docker
      79  sudo dpkg -l | grep docker
      80  sudo dpkg -l | grep postgresql
      81  sudo apt list –-installed
      82  clear
      83  sudo apt list –-installed
      84  sudo apt list –-installed | less
      85  clear
      86  dpkg -l
      87  sudo dpkg -l | grep docker
      88  sudo dpkg -l | grep postgresql
      89  exit
      90  docker ps -a
      91  docker inspect 02e
      92  docker inspect 02e >pgadmin.conf
      93  cd /etc/postgresql/12/main/
      94  ls -l
      95  cp pg_hba.conf postgresql.conf comandospojTaticRoot /home/useroperacao/
      96  chmod +rw /home/useroperacao/pg_hba.conf 
      97  vim pg_hba.conf 
      98  EXIT
      99  exit
     100  cd ..
     101  cd ~
     102  cd /
     103  history >/home/useroperacao/history.txt
       1  exit
       2  psql -c "alter user postgres with password 'postgres@2022'"
       3  netstat -na
       4  apt install netstat
       5  sudo apt install netstat
       6  exit
       7  cd /etc/postgresql/9.3/main && ls -l
       8  cd /etc/postgresql/12/main && ls -l
       9  vim postgresql.conf 
      10  systemctl restart postgresql.service
      11  systemctl restart postgresql.service
      12  sudo systemctl restart postgresql.service
      13  systemctl restart postgresql.service
      14  exit
      15  708090
      16  vim /etc/postgresql/12/main/pg_hba.conf 
      17  systemctl restart postgresql.service 
      18  psql --help
      19  psql -U postgres 
      20  clear
      21  vim /etc/postgresql/12/main/pg_hba.conf 
      22  systemctl restart postgresql.service 
      23  psql -U postgres 
      24  exit
      25  systemctl restart postgresql.service 
      26  vim /etc/postgresql/12/main/pg_hba.conf 
      27  systemctl restart postgresql.service 
      28  exit
      29  history >>/home/useroperacao/history.txt
      30  sudo history >>/home/useroperacao/history.txt
      31  exit 
      32  sudo history >>/home/useroperacao/history.txt
      33  history >>/home/useroperacao/history.txt
       1  docker run hello-world
       2  exit
       3  docker pull dpage/pgadmin4
       4  docker run -p 80:80     -e 'PGADMIN_DEFAULT_EMAIL=operacao@infraop.com'     -e 'PGADMIN_DEFAULT_PASSWORD=SuperSecret'     -d dpage/pgadmin4
       5  docker stop 3a6
       6  docker container rm 3a6
       7  docker run -p 80:80     --name PGADMIN     -e 'PGADMIN_DEFAULT_EMAIL=operacao@infraop.com'     -e 'PGADMIN_DEFAULT_PASSWORD=SuperSecret'     -d dpage/pgadmin4
       8  docker container ls
       9  sudo su postgres
      10  exit
      11  docker ps
      12  docker container inspect 02e
      13  docker container inspect Image
      14  docker container image
      15  docker container inspect image
      16  docker container inspect imagem
      17  docker container inspect 
      18  docker container inspect imagem ls
      19  docker imagem ls
      20  docker image ls
      21  docker Config ls
      22  docker image Config ls
      23  docker image ls
      24  docker image --help
      25  docker image ls "Config"
      26  docker image ls "Config" tag
      27  docker image ls "Config"
      28  docker tag ls "Config"
      29  docker image ls "Config"
      30  docker container inspect imagem ls
      31  docker imagem ls
      32  docker container inspect 
      33  docker repository ls
      34  docker image repository ls
      35  docker image inspect dpage/pgadmin4
      36  docker image inspect 
      37  docker image inspect ls
      38  docker inspect
      39  docker container inspect 02e | grep "Config"
      40  docker container inspect 02e | grep Config
      41  docker container inspect 02e
      42  docker network
      43  docker network ls
      44  docker info
      45  docker inspect --help
      46  docker inspect repository
      47  docker inspect image
      48  docker inspect 
      49  docker image inspect ls
      50  docker container inspect 
      51  docker container inspect ls
      52  docker inspect
      53  docker inspect image
      54  docker container inspect 
      55  docker image inspect dpage/pgadmin4
      56  docker image inspect 
      57  docker container inspect 02e
      58  docker container inspect 02e ls image
      59  docker container inspect 02e ls config
      60  docker container inspect 02e tag
      61  docker container inspect
      62  docker ins
      63  docker repository  inspect
      64  history
      65  exit
      66  history >>/home/useroperacao/history.txt
       1  sudo apt update && apt upgrade
       2  sudo apt update &&sudo  apt upgrade
       3  sudo su
       4  su dev
       5  docker-run hello-world
       6  docker run hello-world
       7  sudo docker run hello-world
       8  docker pull dpage/pgadmin4
       9  su dev
      10  sudo su postgres
      11  apt install netstat 
      12  sudo apt install net-tools 
      13  sudo  netstat -na 
      14  sudo su postgres
      15  history >comandoProjTatic
      16  cat comandoProjTatic 
      17  cd /etc/netplan/
      18  ls -l
      19  cat 00-installer-config.
      20  cat 00-installer-config.y
      21  cat 00-installer-config.yaml 
      22  cp  00-installer-config.yaml configHost
      23  sudo cp  00-installer-config.yaml configHost
      24  cat configHost 
      25  ls
      26  mkdir  ArquivosConfigucao
      27  sudo mkdir  ArquivosConfigucao
      28  mv configHost  ArquivosConfigucao/
      29  sudo mv configHost  ArquivosConfigucao/
      30  sudo su
      31  ls
      32  mv ArquivosConfigucao/ ~/
      33  sudo mv ArquivosConfigucao/ ~/
      34  ls -l
      35  cd ..
      36  ls 
      37  cd /home/
      38  ls
      39  ls -l
      40  cd dev/
      41  ls -l
      42  cd ..
      43  find / -name ArquivosConfigucao
      44  find / -name ArquivosConfigucao*
      45  ls -l
      46  cd home/
      47  ls -l
      48  cd useroperacao/
      49  ls -l
      50  clear
      51  login
      52  login useroperacao
      53  exit
      54  history >>/home/useroperacao/history.txt
-----------------------------------------------------------------

## Foruns e documentaçoes consultadas:

[https://sysadminxpert.com/how-to-install-postgresql-12-on-ubuntu/](https://sysadminxpert.com/how-to-install-postgresql-12-on-ubuntu/)

  

[https://releases.ubuntu.com/20.04/](https://releases.ubuntu.com/20.04/)

  

[https://www.postgresql.org/docs/12/datatype.html](https://www.postgresql.org/docs/12/datatype.html)

  

[https://stackedit.io/app#](https://stackedit.io/app#)

  

[http://pgdocptbr.sourceforge.net/pg80/plpgsql-declarations.html](http://pgdocptbr.sourceforge.net/pg80/plpgsql-declarations.html)

  

[https://docs.docker.com/engine/reference/commandline/docker/](https://docs.docker.com/engine/reference/commandline/docker/)

  

[https://stackoverflow.com/questions/38466190/cant-connect-to-postgresql-on-port- oficiais](https://stackoverflow.com/questions/38466190/cant-connect-to-postgresql-on-port-)

  

[https://www.youtube.com/watch?v=p9LUVzgnif8&list=RDCMUCEBieECyJD82RNCmAAmw77A&start_radio=1&rv=p9LUVzgnif8&t=140](https://www.youtube.com/watch?v=p9LUVzgnif8&list=RDCMUCEBieECyJD82RNCmAAmw77A&start_radio=1&rv=p9LUVzgnif8&t=140)

  

[https://www.youtube.com/watch?v=brB4BQUiAIA](https://www.youtube.com/watch?v=brB4BQUiAIA)

  

[https://pankajconnect.medium.com/all-about-pg-hba-conf-authentication-methods-postgresql-95137d896105#:~:text=authentication%20methods%2D%20Postgresql)-,pg_hba.,directory%20(PostgreSQL10)%20by%20default.&text=Make%20the%20shell%20can%20connect,authentication%20file%20%24PGDATA%2Fpg_hba.](https://pankajconnect.medium.com/all-about-pg-hba-conf-authentication-methods-postgresql-95137d896105#:~:text=authentication%20methods%2D%20Postgresql)


   
