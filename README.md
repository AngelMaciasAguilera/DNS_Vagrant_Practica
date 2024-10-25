# DNS_Vagrant_Practica

# En esta practica vamos a realizar un servidor dns con un maestro que contenga todas las configuraciones 
# y un esclavo que sera la copia del maestro para asi distribuir el trabajo del servidor y que mejore el rendimiento

# Documentacion de los pasos que hemos realizado: 

# Requisitos: 
    · Vagrant: Herramienta de administración de máquinas virtuales.
    · VirtualBox: Proveedor de máquinas virtuales para Vagrant.
    · BIND9: Servicio DNS que será instalado en ambas máquinas virtuales.

# Paso 1: Configuración de Vagrant y Creación de Máquinas
    Configura el archivo Vagrantfile: Define las máquinas Tierra y 
    Venus en el archivo Vagrantfile, estableciendo las direcciones IP privadas y los scripts de provisión.
    · Vagrantfile:

        Vagrant.configure("2") do |config|
      config.vm.box = "debian/bookworm64"
    
      config.vm.define "tierra" do |tierra| 
        tierra.vm.hostname = "tierra.deaw.test"
        tierra.vm.network "private_network", ip: "192.168.57.103"
        tierra.vm.provision "shell", inline: <<-SHELL
          apt-get update
          apt-get install -y bind9 bind9utils bind9-doc
          cp -v /vagrant/earth_files/tierra.sistema.test.dns /var/lib/bind/tierra.sistema.test.dns
        SHELL
      end
    
      config.vm.define "venus" do |venus| 
        venus.vm.hostname = "venus.deaw.test"
        venus.vm.network "private_network", ip: "192.168.57.102"
        venus.vm.provision "shell", inline: <<-SHELL
          apt-get update
          apt-get install -y bind9 bind9utils bind9-doc
        SHELL
      end
    end

    Inicia las máquinas virtuales: Ejecuta el siguiente comando para levantar las máquinas y realizar la configuración inicial:
    
    vagrant up


# Paso 2: Configuración de la Zona DNS en Tierra (Master)

    Edición del archivo named.conf.local: Configura la zona master en la máquina 
    Tierra para definir la zona DNS tierra.sistema.test y permitir la transferencia de zona a Venus.

    sudo nano /etc/bind/named.conf.local


    Añade la configuración de zona master:

    zone "tierra.sistema.test" {
        type master;
        file "/var/lib/bind/tierra.sistema.test.dns";
        allow-transfer { 192.168.57.102; }; //Permite la transferencia con la maquina venus
        also-notify { 192.168.57.102; }; //Notifica a la maqina venus ante cualquier cambio
    };

# Paso 2.1: Creación del Archivo de Zona tierra.sistema.test.dns 

Crea el archivo de zona en la ruta /var/lib/bind tierra.sistema.test.dns 

Definimos el contenido del archivo para la zona tierra.sistema.test con el siguiente formato

    $TTL    86400
    @       IN      SOA     tierra.sistema.test. admin.tierra.sistema.test. (
                                1         ; Serial
                            86400         ; Refresh
                            86400         ; Retry
                            2419200         ; Expire
                            86400 )        ; Negative Cache TTL

    @       IN      NS      tierra.sistema.test.

    @       IN      A       192.168.57.103
    venus   IN      A       192.168.57.102
    marte   IN      A       192.168.57.104
    mercurio  IN    A       192.168.57.101

    ; Aqui agrego el alias a cada uno
    ns1     IN      CNAME   tierra.sistema.test. 
    ns2     IN      CNAME   venus.sistema.test.
    mail    IN      CNAME   marte.sistema.test.

    ; Mail Exchange (MX)
    @       IN      MX 10   marte.sistema.test.

# Paso 2.2: Creación del Archivo de Zona tierra.sistema.test.rev

            $TTL    86400
        @       IN      SOA     tierra.sistema.test. admin.sistema.test. (
                                    2         ; Serial
                                86400         ; Refresh
                                86400         ; Retry
                                2419200         ; Expire
                                86400 )       ; Negative Cache TTL

        @       IN      NS      tierra.sistema.test.

        101     IN      PTR     mercurio.sistema.test.
        102     IN      PTR     venus.sistema.test.
        103     IN      PTR     tierra.sistema.test.
        104     IN      PTR     marte.sistema.test.

    Recarga el servicio BIND:

    sudo systemctl restart bind9

# Paso 3: Configuración de la Zona DNS en Venus (Slave)

    Configura el archivo named.conf.local: En la máquina Venus, define la zona tierra.sistema.test como slave e indica la IP del servidor maestro Tierra.

    sudo nano /etc/bind/named.conf.local

    Añade la configuración de zona slave:

    zone "tierra.sistema.test" {
        type slave;
        file "/var/lib/bind/tierra.sistema.test.slave.db";
        masters { 192.168.57.103; };
    };

    Recarga el servicio BIND en Venus para aplicar los cambios:
    
    sudo systemctl restart bind9

# Paso 4: Configuración de Reenvío DNS en Tierra

    Edita el archivo named.conf.options en Tierra para configurar el reenvío de consultas no locales a OpenDNS:

    sudo nano /etc/bind/named.conf.options

    Añade la configuración de reenvío:

    options {
        directory "/var/cache/bind";

        forwarders {
            208.67.222.222;
        };

        forward only;
    };

    Reinicia el servicio BIND:

    sudo systemctl restart bind9


# Créditos
    Este proyecto fue desarrollado para implementar un sistema DNS con una arquitectura Master-Slave, utilizando Vagrant y BIND9 para crear un entorno DNS robusto y escalable.