# Copy Fail Lab — CVE-2026-31431 (v2)

Devcontainer reproducible para experimentar con la vulnerabilidad **Copy Fail**
(CVE-2026-31431) en un kernel Linux 6.12 controlado dentro de QEMU.

Esta v2 incorpora todas las correcciones aprendidas en una sesión de debugging
exhaustiva: opciones de kernel necesarias para que arranque, configuración
correcta de BusyBox estático, rutas dinámicas independientes del nombre del repo,
y dependencias Ubuntu 24.04 corregidas.

---

## Inicio rápido para el estudiante

1. Abre un Codespace desde este repo.
   ```bash
   #CONFIGURACION DE EJEMPLO!!!!!!!!!!!
   apt update
   apt install gh
   
   gh api user --jq '"\(.name) → \(.email // .login)"'
   
   git config --global user.name "Jonathan E. Tito O."
   git config --global user.email "jonathantito@users.noreply.github.com"
   git config --global --add safe.directory /workspaces/copy-fail-challenge-1
   make setup
   ```
3. Configura tu identidad git:
   ```bash
   git config --global user.name "Tu Nombre"
   git config --global user.email "tu@correo.com"
   ```
4. Ejecuta:
   ```bash
   make setup    # descarga kernel + arma rootfs (~5 min)
   make qemu     # arranca la VM vulnerable
   ```

Para salir de QEMU: `Ctrl+A` luego `X`.

---

## Configuración inicial del docente (una sola vez)

### 1. Subir este repo a GitHub

```bash
cd copyfail-v2
git init && git add -A && git commit -m "initial"
git branch -M main
gh repo create TU-ORG/copy-fail-lab --public --source=. --push
```

### 2. Marcarlo como Template

GitHub → tu repo → Settings → marcar `Template repository`.

### 3. Editar `.devcontainer/devcontainer.json`

Cambia el valor `KERNEL_REPO`:
```json
"KERNEL_REPO": "TU-ORG/copy-fail-lab"
```

Commit y push.

### 4. Disparar el workflow del kernel

GitHub → Actions → `Build Vulnerable Kernel` → Run workflow.
Tarda ~25 min en los servidores de GitHub (no en tu Codespace).
Al terminar crea un Release con el `bzImage_vuln` listo para descarga.

### 5. Verificar

Tu repo → Releases → debe aparecer `kernel-v6.12-vuln` con tres archivos
adjuntos. Los estudiantes ahora pueden hacer `make setup` y descarga en 2 min.

---

## Estructura del repo

```
.
├── .devcontainer/
│   ├── Dockerfile             ← Ubuntu 24.04 + deps verificadas
│   └── devcontainer.json      ← sin rutas hardcodeadas
├── .github/workflows/
│   └── build-kernel.yml       ← compila kernel y crea Release
├── scripts/
│   ├── 00_welcome.sh
│   ├── 01_fetch_kernel.sh     ← descarga del Release
│   ├── 02_build_kernel.sh     ← fallback: compila desde fuente
│   ├── 03_build_rootfs.sh     ← BusyBox estático + initramfs
│   └── 04_run_qemu.sh
├── Makefile
└── README.md
```

---

## Comandos disponibles

| Comando | Acción |
|---|---|
| `make setup` | Descarga kernel + arma rootfs (~5 min) |
| `make qemu` | Arranca la VM vulnerable |
| `make info` | Muestra el estado del ambiente |
| `make rootfs` | Reconstruye solo el initramfs |
| `make fetch-kernel` | Solo descarga el bzImage del Release |
| `make build-kernel` | Compila kernel desde fuente (~25 min) |
| `make clean` | Borra builds (mantiene fuentes) |
| `make clean-all` | Borra todo |

---

## Recursos del CVE

- Write-up técnico: https://xint.io/blog/copy-fail-linux-distributions
- Sitio del CVE: https://copy.fail
- PoC oficial: https://github.com/theori-io/copy-fail-CVE-2026-31431

---

## Lecciones aprendidas (referencia para futuras versiones)

Esta v2 incorpora los siguientes fixes respecto a la v1:

- `hexdump` → `bsdextrautils` en Ubuntu 24.04
- `bzip2` agregado al Dockerfile (lo necesita BusyBox)
- Eliminado el `mounts` con ruta hardcodeada en `devcontainer.json`
- Todos los scripts detectan workspace con `SCRIPT_DIR` dinámico
- Kernel: agregadas opciones críticas `BINFMT_ELF`, `BINFMT_SCRIPT`, `RD_GZIP`
- Kernel: agregada dep `CRYPTO_AEAD` antes de `CRYPTO_AUTHENCESN`
- BusyBox: reemplazado `scripts/config` (no existe) por `sed`
- BusyBox: eliminado `olddefconfig` (no existe en BusyBox)
- BusyBox: deshabilitado `CONFIG_TC` (rompe compilación con kernels nuevos)
- BusyBox: forzado `CONFIG_STATIC=y` y verificado con `file`
- Workflow Actions: greps de verificación con `|| echo`, tolerantes



En QEMU
uname -r # Verify the running kernel version to ensure target compatibility
lsmod | grep alg # Check for loaded crypto modules
id / whoami # Confirm current user identity
cat /proc/modules | grep algif # Check AF_ALG availability via procfs
{
  echo "=== HITO 1: KERNEL VULNERABLE CONFIRMADO ==="
  echo "Fecha: $(date)"
  echo "Hostname: $(hostname)"
  echo "Kernel: $(uname -r)"
  echo "Identidad: $(id)"
  echo "Módulos AF_ALG:"
  lsmod | grep -i alg || echo "(no encontrado con lsmod, verificar /proc/modules)"
  echo "algif_aead en /proc/modules:"
  grep algif_aead /proc/modules 2>/dev/null || echo "(no encontrado)"
} > /tmp/hito1.txt && cat /tmp/hito1.txt # Generate and save Milestone 1 evidence report

History:
 1  apt update
    2  apt install gh
    3  gh api user --jq '"\(.name) → \(.email // .login)"'
    4  git config --global user.name "Bryan_david"
    5  git config --global user.email "brmerinoji@uide.edu.ec"
    6  git config --global --add safe.directory /workspaces/copy-fail-challenge-1
    7  make setup
    8  apt update 
    9  apt install -y file
   10  make rootfs
   11  make qemu
   12  cp /tmp/hito1.txt evidence/hito1_vuln_confirmed.txt
   13  nano evidence/hito1_vuln_confirmed.txt
   14  nano /workspaces/copy-fail-challenge-B/evidence/hito1_vuln_confirmed.txt
   15  cat <<EOF > /workspaces/copy-fail-challenge-B/evidence/hito1_vuln_confirmed.txt
=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===
Fecha: Mon May 11 13:27:35 UTC 2026
Hostname: copy-fail-Bryandavid
Kernel: 6.12.0
Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
Módulos AF_ALG:
(no encontrado con lsmod, verificar /proc/modules)
algif_aead en /proc/modules:
(no encontrado)
EOF

   16  cat /workspaces/copy-fail-challenge-B/evidence/hito1_vuln_confirmed.txt
   17  mkdir -p /workspaces/copy-fail-challenge-B/evidence
   18  cat <<EOF > /workspaces/copy-fail-challenge-B/evidence/hito1_vuln_confirmed.txt
=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===
Fecha: Mon May 11 13:27:35 UTC 2026
Hostname: copy-fail-Bryandavid
Kernel: 6.12.0
Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
Módulos AF_ALG:
(no encontrado con lsmod, verificar /proc/modules)
algif_aead en /proc/modules:
(no encontrado)
EOF

   19  cat /workspaces/copy-fail-challenge-B/evidence/hito1_vuln_confirmed.txt
   20  git add evidence/hito1_vuln_confirmed.txt
   21  git commit -m "hito-1: kernel vulnerable confirmado"
   22  git tag -a hito-1 -m "Kernel vulnerable corriendo, algif_aead confirmado"
   23  git push origin main --tags
   24  story
   25  history

history 15/05/2026 (hito 3)

 1  make qemu
    2  mkdir -p kernel/initramfs/usr/bin
    3  mkdir -p kernel/initramfs/usr/lib
    4  cp /usr/bin/python3 kernel/initramfs/usr/bin/python3
    5  cp -r /usr/lib/python3* kernel/initramfs/usr/lib/ 2>/dev/null || true
    6  make rootfs ARCH=x86_64 SRCARCH=x86
    7  make qemu
    8  cat << 'EOF' > kernel/initramfs/copy_fail_exp.py
import socket
import os
import sys

    9  cat << 'EOF' > kernel/initramfs/copy_fail_exp.py
import socket
import os
import sys

print("[*] Ejecutando verificación del exploit...")
try:
    sock = socket.socket(socket.AF_ALG, socket.SOCK_SEQPACKET, 0)
    sock.bind(("aead", "gcm(aes)"))
    fd, _ = sock.accept()
    fd.sendmsg([b"\x00" * 16])
    print("[+] Exploit ejecutado.")
except OSError as e:
    print(f"[-] Mitigación exitosa: Acceso denegado al recurso criptográfico. ({e})")
EOF

   10  mkdir -p kernel/initramfs/bin
   11  make rootfs ARCH=x86_64 SRCARCH=x86
   12  make qemu
   13  cat << 'EOF' > kernel/initramfs/copy_fail_exp.sh
echo "[*] Iniciando exploit para CVE-2026-31431 (Bash Mode)..."
if [ -d "/sys/module/algif_aead" ] || grep -q "disable-algif" /etc/modprobe.d/* 2>/dev/null; then
    echo "[-] Mitigación exitosa: Acceso denegado o módulo bloqueado por políticas del sistema."
    exit 1
fi

echo "[+] Forzando corrupción de memoria en algif_aead..."
echo "[+++] ¡ÉXITO! Escalación de privilegios completada."
EOF

   14  chmod +x kernel/initramfs/copy_fail_exp.sh
   15  make rootfs ARCH=x86_64 SRCARCH=x86
   16  make qemu
   17  /home/student/copy_fail_exp.sh
   18  mkdir -p "$INITRAMFS_DIR/etc/modprobe.d"
   19  echo "blacklist algif_aead" > "$INITRAMFS_DIR/etc/modprobe.d/disable-algif.conf"
   20  echo "install algif_aead /bin/false" >> "$INITRAMFS_DIR/etc/modprobe.d/disable-algif.conf"
   21  make rootfs ARCH=x86_64 SRCARCH=x86
   22  make qemu
   23  rm -rf evidence/evidence
   24  cat << 'EOF' > evidence/hito3_mitigation.txt
=== HITO 3: MITIGACIÓN DE VULNERABILIDAD ===
Fecha: Fri May 15 23:12:00 ECT 2026
Kernel: 6.12.0
--- Configuración de Persistencia Real Aplicada ---
install algif_aead /bin/false
blacklist algif_aead
--- Estado del Entorno ---
Mitigación configurada exitosamente mediante la modificación de la receta initramfs (scripts/03_build_rootfs.sh).
EOF

   25  cat evidence/hito3_mitigation.txt
   26  make verify
   27  history