# DNS_Vagrant_Practica

# Cosas realizadas dia 14/10/2024

# Primero lo que hacemos es ejecutar el comando vagrant init para inicializar el directorio con un vagrant file que contendra las especificaciones de como estaran configuradas nuestras maquinas

# Despues en el vagrant file especificamos que sistema operativo van a tener nuestras maquinas y que librerias en mi caso debian con libreria bookworm64.

# config.vm.box = "debian/bookworm64"

# Ahora en nuestro vagrant file definimos las máquinas de la siguiente manera: 

# Maquina Venus
# config.vm.define "venus" do |venus| 
# venus.vm.hostname = "venus.deaw.test"
# venus.vm.network "private_network", ip:"192.168.57.102"
# end

# Maquina Tierra

# config.vm.define "tierra" do |tierra| 
# tierra.vm.hostname = "tierra.deaw.test"
# tierra.vm.network "private_network", ip:"192.168.57.103"
# end
# Aqui lo que estamos haciendo es definir nuestras maquinas virtuales que tendran una ip especifica.

