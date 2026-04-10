# 🧾 CHECKLIST AUTOFS (RHCSA – AlmaLinux)

---

## 🔴 1. Instalar autofs

### 📍 CLIENTE
dnf install -y autofs  
> Instala el servicio de automontaje.

---

## 🔴 2. Crear punto base de montaje

### 📍 CLIENTE
mkdir -p /mnt/nfs  
> Directorio base donde autofs gestionará los montajes.

---

## 🔴 3. Configurar mapa maestro

### 📍 CLIENTE
vi /etc/auto.master  

Agregar línea:
```
/mnt/nfs /etc/auto.nfs
```
> Indica que autofs usará el mapa `/etc/auto.nfs` para ese directorio.

---

## 🔴 4. Crear mapa de montaje

### 📍 CLIENTE
vi /etc/auto.nfs  

Agregar:
```
data -fstype=nfs 192.168.122.44:/srv/nfs/share
```
> Define que al acceder a `/mnt/nfs/data`, se montará ese recurso NFS.

---

## 🔴 5. Reiniciar servicio

### 📍 CLIENTE
systemctl enable --now autofs  
systemctl restart autofs  
> Aplica la configuración y deja el servicio activo.

---

## 🔴 6. Verificar automontaje

### 📍 CLIENTE
ls /mnt/nfs  
> Aún no monta nada (directorio vacío).

cd /mnt/nfs/data  
> Aquí se dispara el automontaje.

---

## 🔴 7. Verificar montaje activo

### 📍 CLIENTE
mount | grep nfs  
> Confirma que el recurso se montó automáticamente.

---

## 🔴 8. Prueba funcional

### 📍 CLIENTE
touch /mnt/nfs/data/test_autofs  
> Crea archivo en el recurso remoto.

---

## 🔴 9. Verificación en servidor

### 📍 SERVIDOR
ls /srv/nfs/share  
> Debe aparecer el archivo creado desde el cliente.

---

## 🔴 10. Comportamiento automático

### 📍 CLIENTE
# Esperar unos minutos sin usar el directorio

mount | grep nfs  
> El montaje desaparecerá automáticamente.

---

## 🧠 Concepto clave

autofs monta recursos **solo cuando se accede** y los desmonta tras inactividad.
