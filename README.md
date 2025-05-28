# 📡 Instalação de Placa USB Wi-Fi + Bluetooth 5.3 RTL8851BU no Linux

## 🔧 Sobre este driver

Este repositório contém os drivers para dispositivos **Realtek USB AX900**, que utilizam os chipsets **RTL8851BU** e **RTL8831BU**.

Esses drivers são compatíveis com adaptadores USB Wi-Fi 6 + Bluetooth 5.3 — frequentemente vendidos sem marca em sites como **AliExpress** e **Amazon**.

## 📦 Fonte do driver

Este projeto é um clone do repositório original:

* 🔗 [Repositório Original – biglinux/rtl8831](https://github.com/biglinux/rtl8831)
* 🔗 [Outro repositório atualizado](https://github.com/fofajardo/rtl8851bu.git)

---

## 🔧 Ambiente

* **Sistema Operacional**: Ubuntu 24.04 (ou equivalente baseado em Ubuntu)
* **Kernel**: (confirme com `uname -r`, exemplo: 6.8)
* **Placa**: (ex: RTL8851BU baseada – informe o modelo utilizado)

---

## ⚙️ Ferramentas Utilizadas

### 1. Netplan

* **O que é**: Ferramenta para configuração de rede em sistemas Ubuntu modernos. Utiliza arquivos YAML para configurar interfaces Ethernet e Wi-Fi.
* **Função**: Define de forma legível configurações como DHCP, DNS, Wi-Fi e rotas. Pode usar como backend o `networkd` ou `NetworkManager`.

### 2. systemd-networkd

* **O que é**: Backend usado pelo Netplan para gerenciar interfaces de rede.
* **Função**: Quando o Netplan está configurado com `networkd`, este aplica as configurações. Ideal para servidores. `NetworkManager` é mais indicado para desktops.

### 3. make

* **O que é**: Ferramenta de automação de compilação.
* **Função**: Compila o driver RTL8851BU com base no arquivo `Makefile`.

### 4. dkms (Dynamic Kernel Module Support)

* **O que é**: Ferramenta para recompilação de módulos após atualizações de kernel.
* **Função**: Garante que o driver continue funcional mesmo após atualizações de kernel.

### 5. iw, wireless-tools

* **O que é**: Conjunto de ferramentas para diagnóstico e configuração de redes Wi-Fi.
* **Função**: Scaneia redes (`iwlist`) e configura interfaces (`iwconfig`).

### 6. wpasupplicant

* **O que é**: Software de autenticação para redes Wi-Fi protegidas.
* **Função**: Conecta o sistema a redes WPA/WPA2/WPA3.

---

# 🚀 Passo a Passo

---

## 1. Atualizar o Sistema

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Instalar Ferramentas Necessárias

```bash
sudo apt install -y build-essential dkms linux-headers-$(uname -r) git iw wpasupplicant wireless-tools network-manager
```

---

## 3. Clonar, Renomear, Compilar e Instalar o Driver RTL8851BU

> A instalação manual requer reinstalação após cada atualização de kernel. Para evitar isso, recomenda-se usar o **DKMS**.

### Clonar o Repositório e Renomear

```bash
git clone https://github.com/MithrasdeLuca-dev/Install-RTL8851BU.git
mv Install-RTL8851BU RTL8851bu
cd RTL8851bu
```

### Instalação Manual

```bash
make -j$(nproc)  # Ou especifique: make -j16
sudo make install
```

---

## 🔧 Instalação do Módulo com DKMS (Versão 0.1)

Utilize o DKMS para compilar e instalar o módulo automaticamente em todas as versões do kernel atuais e futuras.

---

### 1. 📥 Adicionar o Módulo ao DKMS

No diretório com o `dkms.conf`:

```bash
sudo dkms add .
```

> Registra o módulo e sua versão com base no `dkms.conf`.

Verifique se foi adicionado corretamente:

```bash
sudo dkms status
```

---

### 2. 🛠️ Compilar o Módulo

```bash
sudo dkms build RTL8851bu/0.1
```

---

### 3. 📦 Instalar o Módulo no Sistema

```bash
sudo dkms install RTL8851bu/0.1
```

---

### 🧹 (Opcional) Remover o Módulo

```bash
sudo dkms remove RTL8851bu/0.1 --all
```

---

### ℹ️ ✅ Resumo dos Comandos

```bash
cd /caminho/do/modulo
sudo dkms add .
sudo dkms status
sudo dkms build RTL8851bu/0.1
sudo dkms install RTL8851bu/0.1
sudo dkms remove RTL8851bu/0.1 --all
```

#### 🔍 Descrição dos Comandos

* `cd /caminho/do/modulo`: Acessa o diretório do código-fonte com o `dkms.conf`.
* `sudo dkms add .`: Registra o módulo.
* `sudo dkms status`: Exibe o status dos módulos registrados.
* `sudo dkms build`: Compila o módulo.
* `sudo dkms install`: Instala o módulo no sistema.
* `sudo dkms remove`: Remove o módulo de todas as versões do kernel.

---

## 4. Listar Interfaces de Rede

```bash
ls /sys/class/net
```

---

## 5. Renomear Interface de Rede (Opcional)

### Descobrir MAC Address

```bash
ip link
```

### Criar/Editar regra udev

```bash
sudo nano /etc/udev/rules.d/10-network.rules
```

### Exemplo de regra:

```bash
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="XX:XX:XX:XX:XX:XX", NAME="wlan0"
```

### Aplicar regras:

```bash
sudo udevadm control --reload-rules
```

---

## 6. Configurar o Netplan

### Acessar diretório:

```bash
cd /etc/netplan/
ls
```

### Editar arquivo Netplan:

```bash
sudo nano XX-installer-config.yaml
```

### Exemplo 1: NetworkManager

```bash
nmcli device wifi connect "NOME_DA_REDE" password "SENHA_DA_REDE"
```

### Exemplo 2: Netplan (YAML)

```yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    enp5s0:
      dhcp4: true
      dhcp4-overrides:
        route-metric: 200

  wifis:
    wlan0:
      dhcp4: true
      dhcp4-overrides:
        route-metric: 100
      access-points:
        "NOME-DA-REDE":
          password: "SENHA"
```

---

## 7. Ativar Dispositivo Wi-Fi

```bash
sudo ip link set wlan0 up
sudo ip link show wlan0
iwconfig
```

---

## 8. Escanear Redes Wi-Fi

```bash
sudo iwlist wlan0 scan | grep ESSID
sudo iw dev wlan0 scan | grep SSID
sudo nmcli device wifi list
```

---

## 9. Aplicar Configuração do Netplan ou NetworkManager

```bash
sudo netplan try
sudo netplan apply
sudo netplan --debug apply

# Ativar e verificar NetworkManager
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
sudo systemctl status NetworkManager
```

> **Se `netplan apply` falhar:**

* Verifique se o `wpa_supplicant` está ativo.
* Verifique SSID, senha e o nome do dispositivo (`wlan0`) no `.yaml`.

---

## 10. Verificar o Status da Rede

```bash
systemctl status wpa_supplicant
sudo systemctl status wpa_supplicant@wlan0
journalctl -u systemd-networkd
```

---

## 11. Diagnóstico de Hardware de Rede

### Exibir informações dos dispositivos de rede:

```bash
lshw -C network
```

### Verificar se o driver foi carregado:

```bash
lsmod | grep 8851bu
sudo dmesg | grep 8851bu
```

### Diagnosticar adaptadores detectados:

```bash
lspci -k | grep -A 3 -i wireless
lsusb
```

---

# 📚 Anexo: Ferramentas Utilizadas

- **build-essential**: Compiladores C/C++ e dependências
- **dkms**: Suporte para módulos de kernel dinâmicos
- **linux-headers-\$(uname -r)**: Cabeçalhos do kernel
- **git**: Controle de versão
- **iw**: Ferramenta de gerenciamento de Wi-Fi
- **wireless-tools**: Antiga ferramenta para Wi-Fi
- **wpasupplicant**: Gerenciador de autenticação Wi-Fi
- **netplan**: Gerenciador de configuração de rede no Ubuntu

---
