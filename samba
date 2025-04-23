LOG_FILE="/root/archive.log"

update_system() {
    echo -e "\e[33m>>> Iniciando actualización del sistema: $(date '+%Y-%m-%d %H:%M:%S') <<<\e[0m" | tee -a "$LOG_FILE"
    if ! apt update && apt upgrade -y && apt autoremove -y; then
        echo -e "\e[31mError: Falló la actualización del sistema operativo.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi
}

install_samba() {
    echo -e "\e[33m>>> Iniciando instalación de SAMBA... <<<\e[0m" | tee -a "$LOG_FILE"
    if ! apt install samba -y; then
        echo -e "\e[31mError: Falló la instalación de SAMBA.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi
}

samba_config() {
    echo -e "\e[33m>>> Crear usuario para samba... <<<\e[0m" | tee -a "$LOG_FILE"
    if ! adduser share; then
        echo -e "\e[31mError: Falló la creación del usuario.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi

    if ! usermod -aG sambashare share; then
        echo -e "\e[31mError: Falló al unir el usuario al grupo sambashare.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi
    if ! smbpasswd -a share; then
        echo -e "\e[31mError: Falló la asignación de la contraseña para el usuario SAMBA.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi


    echo -e "\e[33m>>> Configurando archivo /etc/samba/smb.conf... <<<\e[0m" | tee -a "$LOG_FILE"
    cat <<EOL > /etc/samba/smb.conf

[global]

## Browsing/Identification ###

# Change this to the workgroup/NT-domain name your Samba server will part of
   workgroup = WORKGROUP
# server string is the equivalent of the NT Description field
   security = user
   map to guest = never
   deadtime = 10
   max connections = 20
   wins support = no
   dns proxy = no


[Folder]
   path = /share/folder
   read only = yes
   directory mask = 0755
   create mask = 0755
   valid users = @sambashare
   write list = share
   browseable = yes
   guest ok = no
   public = no

EOL
    echo -e "\e[33m>>> Configuración de Samba completada. <<<\e[0m" | tee -a "$LOG_FILE"
}

create_folder() {
    echo -e "\e[33m>>> Crear carpeta compartida... <<<\e[0m" | tee -a "$LOG_FILE"
    if ! mkdir -p /share/folder; then
        echo -e "\e[31mError: Falló la creación de la carpeta.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi
}

asign_permissions() {
    echo -e "\e[33m>>> Asignando permisos a la carpeta... <<<\e[0m" | tee -a "$LOG_FILE"
    if ! chmod -R 755 /share/folder; then
        echo -e "\e[31mError: Falló la asignación de permisos de acceso a la carpeta.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi
    if ! chown -R share:sambashare /share/folder; then
        echo -e "\e[31mError: Falló la asignación de permisos.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi
    echo -e "\e[33m>>> Permisos asignados correctamente. <<<\e[0m" | tee -a "$LOG_FILE"
}

restart_service(){
    if ! systemctl restart smbd; then
        echo -e "\e[31mError: Falló el reinicio del servicio.\e[0m" | tee -a "$LOG_FILE"
        return 1
    fi
}

update_system
install_samba
create_folder
samba_config
asign_permissions
restart_service
