# 🧾 CHECKLIST NFS (RHCSA – AlmaLinux)

---

## 🔴 1. Instalar paquetes necesarios

### 📍 SERVIDOR
dnf install -y nfs-utils  
> Instala los binarios necesarios para exportar directorios vía NFS.

### 📍 CLIENTE
dnf install -y nfs-utils  
> Necesario para poder montar recursos NFS desde otro sistema.

---

## 🔴 2. Crear el directorio a compartir

### 📍 SERVIDOR
mkdir -p /srv/nfs/share  
chmod 777 /srv/nfs/share  
> El directorio debe existir físicamente; permisos abiertos simplifican pruebas iniciales.

---

## 🔴 3. Configurar exportación NFS

### 📍 SERVIDOR
vi /etc/exports  

/srv/nfs/share *(rw,sync,no_root_squash)  
> Define qué se comparte, a quién (*) y con qué permisos.

---

## 🔴 4. Aplicar configuración

### 📍 SERVIDOR
exportfs -rav  
> Aplica los cambios sin reiniciar servicios.

exportfs -v  
> Permite verificar cómo se exportan los recursos.

---

## 🔴 5. Levantar servicios necesarios

### 📍 SERVIDOR
systemctl enable --now nfs-server  
systemctl enable --now rpcbind  
> NFS depende de rpcbind para mapear servicios en red.

systemctl status nfs-server  
systemctl status rpcbind  

---

## 🔴 6. Configurar firewall

### 📍 SERVIDOR
firewall-cmd --permanent --add-service=nfs  
firewall-cmd --permanent --add-service=mountd  
firewall-cmd --permanent --add-service=rpc-bind  
firewall-cmd --reload  
> Abre los puertos necesarios para comunicación NFS.

---

## 🔴 7. Verificar export desde cliente

### 📍 CLIENTE
showmount -e IP_DEL_SERVIDOR  
> Consulta qué recursos están disponibles.

---

## 🔴 8. Crear punto de montaje

### 📍 CLIENTE
mkdir -p /mnt/nfs  
> Directorio local donde se montará el recurso.

---

## 🔴 9. Montar manualmente

### 📍 CLIENTE
mount IP_DEL_SERVIDOR:/srv/nfs/share /mnt/nfs  
> Conecta el recurso remoto al sistema local.

---

## 🔴 10. Verificar montaje

### 📍 CLIENTE
df -h | grep nfs  
mount | grep nfs  
> Confirma que el montaje está activo.

---

## 🔴 11. Prueba funcional

### 📍 CLIENTE
touch /mnt/nfs/test_file  
> Crea archivo en NFS.

### 📍 SERVIDOR
ls /srv/nfs/share  
> Verifica que existe en el servidor.

---

## 🔴 12. Persistencia

### 📍 CLIENTE
vi /etc/fstab  

IP_DEL_SERVIDOR:/srv/nfs/share /mnt/nfs nfs defaults 0 0  
> Montaje automático al iniciar.

---

## 🔴 13. Validar fstab

### 📍 CLIENTE
umount /mnt/nfs  
mount -a  
> Simula reinicio y valida configuración.

---

## 🔴 14. Verificación final

### 📍 CLIENTE
df -h | grep nfs  
> Confirma persistencia del montaje.
