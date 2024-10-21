# DNS_Vagrant_Practica

# Cosas realizadas dia 14/10/2024

# Primero lo que hacemos es ejecutar el comando vagrant init para inicializar el directorio con un vagrant file que contendra las especificaciones de como estaran configuradas nuestras maquinas

# Despues en el vagrant file especificamos que sistema operativo van a tener nuestras maquinas y que librerias en mi caso debian con libreria bookworm64.

config.vm.box = "debian/bookworm64"

# Ahora en nuestro vagrant file definimos las máquinas de la siguiente manera: 

# Maquina Venus
    config.vm.define "venus" do |venus| 
      venus.vm.hostname = "venus.deaw.test"
      venus.vm.network "private_network", ip:"192.168.57.102"
    end

# Maquina Tierra

    config.vm.define "tierra" do |tierra| 
      tierra.vm.hostname = "tierra.deaw.test"
      tierra.vm.network "private_network", ip:"192.168.57.103"
    end
# Aqui lo que estamos haciendo es definir nuestras maquinas virtuales que tendran una ip especifica.


# Ahora nos conectamos a nuestra maquina cualquiera de las dos (venus o tierra) y las actualizamos primero para que reconozcan todos los comandos y repositorios actuales correctamente, para ello utilizamos el comando:

sudo apt-get update


# Ahora instalamos los paquetes necesarios para el servidor mediante el siguiente comando:

sudo apt-get install bind9 bind9utils bind9-doc






# Ahora las configuraremos para que solo usen IPv4, para ello primero nos vamos al directorio de default y accedemos posteriormente al archivo named mediante el comando

cd /etc/default


# Una vez alli ejecutamos el siguiente comando

sudo nano named

# Nos aparecerá el bash para modificarlo, y dentro del archivo tendremos que modificar la linea "OPTIONS = "-u bind" " lo siguiente: 

OPTIONS = "-u bind -4"

# Guardamos y repetimos con el otro servidor


