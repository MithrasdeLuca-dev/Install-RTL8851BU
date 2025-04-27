# 📡 Instalação de Placa USB Wi-Fi + Bluetooth 5.3 RTL8851BU no Linux

# Driver Realtek Linux para dispositivos USB AX900 Wifi 6 8851bu e 8831bu

Este driver é destinado a dispositivos **Realtek USB AX900** que utilizam os chipsets **Realtek 8851bu** ou **8831bu**.

Isso inclui adaptadores **AX900 USB WiFi 6** e **Bluetooth 5.3** sem marca, vendidos em plataformas como **AliExpress** e **Amazon**.

# Para instalação:

## 🔧 Ambiente

- **Sistema Operacional**: Ubuntu 24.04 (ou equivalente baseado em Ubuntu)
- **Kernel**: (confirme com `uname -r`, exemplo: 6.8)
- **Placa**: (ex: RTL8851BU baseada, informe o modelo que usou)

---

## ⚙️ Ferramentas Utilizadas

### 1. Netplan
- **O que é**: Ferramenta para configuração de rede em sistemas Ubuntu modernos. Usando arquivos YAML, ela simplifica a configuração de interfaces de rede, tanto para Ethernet quanto para Wi-Fi.
- **Função**: Permite que você defina de maneira simples e legível as configurações de rede, como DHCP, DNS, configurações de Wi-Fi e roteamento de tráfego. O Netplan pode ser configurado para usar diferentes backends, como networkd ou NetworkManager.

### 2. systemd-networkd
- **O que é**: Backend utilizado pelo Netplan para configurar e gerenciar as interfaces de rede.
- **Função**: Quando o Netplan está configurado para usar networkd, ele aplica as configurações de rede diretamente através desse serviço, que é mais leve e recomendado para servidores, enquanto o NetworkManager é melhor para ambientes desktop.

### 3. make
- **O que é**: Ferramenta de automação de compilação de código-fonte.
- **Função**: Usada para compilar o driver RTL8851BU a partir do código-fonte. O comando `make` processa o arquivo Makefile para construir o driver.

### 4. dkms (Dynamic Kernel Module Support)
- **O que é**: Ferramenta que facilita a recompilação de módulos do kernel ao atualizar o sistema.
- **Função**: Permite que o driver seja recompilado automaticamente quando o kernel é atualizado, garantindo que o driver funcione com versões futuras do kernel.

### 5. iw, wireless-tools
- **O que é**: Conjunto de ferramentas para diagnosticar e configurar interfaces de rede sem fio.
- **Função**: Utilizadas para escanear redes Wi-Fi e verificar a conexão das interfaces de rede sem fio, como o comando `iwlist` para escanear redes e `iwconfig` para visualizar e configurar interfaces Wi-Fi.

### 6. wpasupplicant
- **O que é**: Software de autenticação para redes Wi-Fi protegidas com WPA/WPA2/WPA3.
- **Função**: Responsável por conectar o sistema a redes Wi-Fi seguras, gerenciando as credenciais de autenticação.

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
sudo apt install build-essential dkms linux-headers-$(uname -r) git iw wireless-tools wpasupplicant -y
```
---
## 3. Clonar, Renomear, Compilar e Instalar o Driver RTL8851BU

Ao optar pela instalação manual, você precisará reinstalar o driver após cada atualização do kernel. Para evitar esse processo, é recomendável instalar usando o **DKMS**, que gerencia automaticamente a recompilação do driver após atualizações do kernel.

### Clonando o Repositório e Renomeando

```bash
git clone https://github.com/MithrasdeLuca-dev/Install-RTL8851BU.git
mv Install-RTL8851BU RTL8851bu
cd RTL8851bu
```

### Instalação Manual

Para compilar e instalar o driver manualmente:

```bash
make -j$(nproc)  # Ou use 'make -j16 "número de núcleos do processador>"
sudo make install
```

### Instalação Usando DKMS

1. **Adicionar o Módulo ao DKMS:**

```bash
sudo dkms add .
```

2. **Construir o Módulo:**

```bash
sudo dkms build RTL8851bu/1.0  # Certifique-se de que a versão é compatível com seu módulo
```

3. **Instalar o Módulo com DKMS:**

```bash
sudo dkms install RTL8851bu/1.0
```

O comando `dkms add .` registra o módulo no sistema, `dkms build` compila o driver, e `dkms install` instala o módulo no sistema.

---
## 4. Listar Interfaces de Rede

```bash
ls /sys/class/net
```
---

## 5. Renomear Interface de Rede (Opcional)

1. Criar/editar regra udev:

```bash
sudo nano /etc/udev/rules.d/10-network.rules
```

2. Adicionar:

```bash
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="XX:XX:XX:XX:XX:XX", NAME="wlan0"
```

3. Aplicar regras:

```bash
sudo udevadm control --reload-rules
```

**Descobrir MAC Address:**

```bash
ip link
```

---

## 6. Configurar o Netplan

1. Acessar diretório:

```bash
cd /etc/netplan/
ls
```

2. Editar arquivo:

```bash
sudo nano installer-config.yaml
```

3. Exemplo de configuração:

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
```

---

## 9. Aplicar Configuração do Netplan

```bash
sudo netplan try
sudo netplan apply
sudo netplan --debug apply
```

**Caso ocorra uma falha no `netplan apply`:**
- Verifique se o gerenciador de autenticação (wpa_supplicant) está ativo.
- Verifique se o arquivo `.yaml` de configuração do Netplan está utilizando:
  - As credenciais corretas da rede Wi-Fi (SSID e senha).
  - O nome correto do dispositivo de rede (por exemplo, `wlan0`).

---

## 10. Verificar o Status da Rede

```bash
systemctl status wpa_supplicant
sudo systemctl status wpa_supplicant@wlan0
journalctl -u systemd-networkd
```

---

## 11. Diagnóstico de Hardware de Rede

### Mostrar detalhes dos dispositivos de rede:

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

## 12. Ativar e Verificar Interface Wi-Fi

```bash
ip a
sudo ip link set wlan0 up
sudo ip link show wlan0
iwconfig
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
