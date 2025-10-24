# 🍷 Sistema IoT de Monitoramento de Adega Climatizada

<div align="center">

![ESP32](https://img.shields.io/badge/ESP32-000000?style=for-the-badge&logo=espressif&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![MQTT](https://img.shields.io/badge/MQTT-660066?style=for-the-badge&logo=mqtt&logoColor=white)
![FIWARE](https://img.shields.io/badge/FIWARE-FF7139?style=for-the-badge&logo=fiware&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

_Sistema completo de monitoramento e controle para adegas climatizadas utilizando IoT e FIWARE_

</div>

---

## 📋 Sobre o Projeto

Este projeto implementa um **sistema completo de monitoramento IoT** para adegas climatizadas, utilizando o microcontrolador **ESP32** para coletar dados ambientais e transmiti-los para uma infraestrutura em nuvem baseada na plataforma **FIWARE** hospedada na **AWS**.

### ✨ Funcionalidades Principais

| Funcionalidade                  | Descrição                                                |
| ------------------------------- | -------------------------------------------------------- |
| 🌡️ **Temperatura**              | Monitoramento em tempo real via sensor DHT22             |
| 💧 **Umidade**                  | Controle da umidade ideal para conservação de vinhos     |
| 💡 **Luminosidade**             | Medição através de sensor LDR (Light Dependent Resistor) |
| 🔄 **Comunicação Bidirecional** | Controle remoto do LED via protocolo MQTT                |
| ☁️ **Cloud Computing**          | Dados processados e armazenados em infraestrutura AWS    |
| 📱 **Acesso Mobile**            | Visualização e controle pelo aplicativo MyMQTT           |
| 🐳 **Arquitetura FIWARE**       | Uso de componentes Orion, STH-Comet e MongoDB            |

---

## 🏗️ Arquitetura do Sistema

### 📐 Componentes da Arquitetura

#### **Camada de Aplicação**

- **Mobile**: Aplicativo MyMQTT para controle remoto
- **BigData**: Análise de grandes volumes de dados históricos

#### **Camada de Backend (Docker)**

- **Orion Context Broker** (Porta 1026): Gerenciamento de contexto em tempo real
- **STH-Comet** (Porta 8666): Armazenamento de dados históricos
- **MongoDB** (Porta 27017): Banco de dados NoSQL
- **IoT Agent MQTT** (Porta 4041): Ponte entre dispositivos MQTT e FIWARE

#### **Camada IoT**

- **MQTT Broker** (Porta 1883): Servidor Mosquitto para comunicação
- **ESP32**: Microcontrolador com sensores DHT22, LDR e LED
- **Sensores**: Captação de dados ambientais
- **Atuadores**: Controle do LED para feedback visual

### 🔄 Fluxo de Dados

<p align="center">
  <img src="./arquitetura.png" alt="Arquitetura do Projeto" style="width:400px;"/>
</p>

---

## 🔧 Recursos Necessários

### Hardware (Simulado no Wokwi)

- ESP32
- Sensor DHT22 (Temperatura e Umidade)
- Sensor LDR (Luminosidade)
- LED
- Resistores (10kΩ para LDR e 330Ω para LED)

### Software e Serviços

- **Wokwi**: Simulador online de ESP32
- **AWS EC2**: Máquina virtual na nuvem
- **Docker & Docker Compose**: Containerização dos serviços FIWARE
- **Postman**: Testes e configuração da API FIWARE
- **MyMQTT**: Aplicativo mobile (Android/iOS)

---

## 🚀 Guia de Configuração

### Passo 1: Configurar o Circuito no Wokwi

#### 🔌 Tabela de Conexões

| Componente              | Pino ESP32                 | Observações                      |
| ----------------------- | -------------------------- | -------------------------------- |
| **DHT22**               |                            |                                  |
| VCC                     | 3.3V                       | Alimentação                      |
| GND                     | GND                        | Terra                            |
| DATA                    | GPIO 4                     | Leitura de temperatura e umidade |
| **LDR (Sensor de Luz)** |                            |                                  |
| Terminal 1              | 3.3V                       | Alimentação                      |
| Terminal 2              | GPIO 34 (ADC)              | Leitura analógica                |
| Terminal 2              | GND (via resistor 10kΩ)    | Divisor de tensão                |
| **LED**                 |                            |                                  |
| Ânodo (+)               | GPIO 2 (via resistor 330Ω) | Controle do LED                  |
| Cátodo (-)              | GND                        | Terra                            |

#### 📝 Arquivos do Projeto

Para fazer a simulação do ESP32 use o Wokwi e copie os arquivos do circuito e código estão na pasta `devices/` do repositório:

- `sketch.ino` - Código do ESP32
- `diagram.json` - Configuração do circuito Wokwi

Ou então entre no link:

- [**Circuito Wokwi**](https://wokwi.com/projects/444821783999289345)

#### ⚙️ Configurar o Código

Abra o arquivo `sketch.ino` e **ajuste as seguintes linhas** com os dados da sua VM AWS:

```cpp
// Linha 17-19: Configuração WiFi (já está configurado para Wokwi)
const char* SSID = "Wokwi-GUEST";
const char* PASSWORD = "";

// Linha 20: ALTERE para o IP público da sua VM EC2
const char* BROKER_MQTT = "SEU_IP_DA_VM_AWS";

// Linha 21: Porta do MQTT Broker
const int BROKER_PORT = 1883;
```

#### 🎯 Tópicos MQTT Utilizados

| Tópico                   | Tipo      | Descrição                    |
| ------------------------ | --------- | ---------------------------- |
| `/TEF/device001/cmd`     | Subscribe | Recebe comandos (LED ON/OFF) |
| `/TEF/device001/attrs`   | Publish   | Estado geral do dispositivo  |
| `/TEF/device001/attrs/s` | Publish   | Estado do LED (on/off)       |
| `/TEF/device001/attrs/l` | Publish   | Luminosidade (0-100%)        |
| `/TEF/device001/attrs/h` | Publish   | Umidade (%)                  |
| `/TEF/device001/attrs/t` | Publish   | Temperatura (°C)             |

---

### Passo 2: Configurar a VM AWS

#### 📦 2.1 - Criar Instância EC2

1. Acesse o **AWS Console**
2. Vá para **EC2 > Launch Instance**
3. Escolha **Ubuntu Server 22.04 LTS**
4. Tipo: **t2.medium** ou superior (recomendado para FIWARE)
5. Configure o **Security Group** com as seguintes portas:

| Porta | Protocolo | Descrição               |
| ----- | --------- | ----------------------- |
| 22    | TCP       | SSH                     |
| 1883  | TCP       | MQTT Broker (Mosquitto) |
| 1026  | TCP       | Orion Context Broker    |
| 4041  | TCP       | IoT Agent MQTT          |
| 8666  | TCP       | STH-Comet               |

#### 🔗 2.2 - Conectar via SSH

```bash
ssh -i sua-chave.pem ubuntu@SEU_IP_PUBLICO
```

#### 📥 2.3 - Instalar Docker e Git

```bash
# Atualizar o sistema
sudo apt update && sudo apt upgrade -y

# Instalar Git
sudo apt install git -y

# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Adicionar usuário ao grupo docker
sudo usermod -aG docker $USER

# Instalar Docker Compose
sudo apt install docker-compose -y

# Reiniciar a sessão para aplicar as permissões
exit
# Conecte novamente via SSH
```

#### 🐳 2.4 - Clonar e Executar o FIWARE

```bash
# Clonar o repositório FIWARE Descomplicado
git clone https://github.com/fabiocabrini/fiware.git

# Entrar na pasta
cd fiware

# Subir os containers
sudo docker compose up -d

# Verificar se os containers estão rodando
sudo docker ps
```

**Containers esperados:**

- `fiware-orion`
- `fiware-sth-comet`
- `fiware-iot-agent`
- `fiware-mongo`
- `mosquitto-mqtt`

---

### Passo 3: Provisionar Dispositivos no FIWARE (Postman)

#### 📬 3.1 - Importar Collection no Postman

1. Baixe o arquivo de collection do repositório **FIWARE Descomplicado**
2. Abra o **Postman**
3. Clique em **Import** e selecione o arquivo

#### ✅ 3.2 - Health Check

Execute a requisição **Health Check** para verificar se o Orion está respondendo:

```
GET http://SEU_IP_AWS:1026/version
```

#### 📡 3.3 - Provisionar Dispositivo

Execute a requisição **Provisionar Dispositivo** (POST). Certifique-se de que o corpo da requisição contém:

```json
{
  "devices": [
    {
      "device_id": "device001",
      "entity_name": "urn:ngsi-ld:Device:001",
      "entity_type": "Device",
      "protocol": "PDI-IoTA-UltraLight",
      "transport": "MQTT",
      "attributes": [
        { "object_id": "t", "name": "temperature", "type": "Float" },
        { "object_id": "h", "name": "humidity", "type": "Float" },
        { "object_id": "l", "name": "luminosity", "type": "Float" },
        { "object_id": "s", "name": "status", "type": "Text" }
      ],
      "commands": [
        { "name": "on", "type": "command" },
        { "name": "off", "type": "command" }
      ]
    }
  ]
}
```

#### 📝 3.4 - Registrar Comandos

Execute as requisições para registrar os comandos:

- **Registrar Comando ON**
- **Registrar Comando OFF**

#### 🔔 3.5 - Subscrever às Mudanças

Execute a requisição **Criar Subscription** para que o STH-Comet armazene os dados históricos.

#### 🧪 3.6 - Testar Leitura de Dados

Com o **Wokwi rodando**, execute a requisição **GET - Luminosidade**:

```
GET http://SEU_IP_AWS:1026/v2/entities/urn:ngsi-ld:Device:001
```

**⚠️ IMPORTANTE**: Certifique-se de usar o `entity_name` e `entity_type` corretos conforme o provisionamento:

- **ID**: `urn:ngsi-ld:Device:001`
- **Type**: `Device`

Você deve receber os valores de temperatura, umidade, luminosidade e status do LED em tempo real.

---

### Passo 4: Configurar o Aplicativo MyMQTT

#### 📱 4.1 - Instalar o MyMQTT

- **Android**: [Play Store](https://play.google.com/store/apps/details?id=at.tripwire.mqtt.client)
- **iOS**: [App Store](https://apps.apple.com/app/mymqtt/id1529660475)

#### 🔗 4.2 - Configurar Conexão

1. Abra o **MyMQTT**
2. Toque em **+** para adicionar nova conexão
3. Configure:
   - **Nome**: Adega FIWARE
   - **Host**: `SEU_IP_DA_VM_AWS`
   - **Porta**: `1883`
   - **Username**: (deixe vazio)
   - **Password**: (deixe vazio)
4. Toque em **Salvar** e depois em **Conectar**

#### 📊 4.3 - Subscrever aos Tópicos

Vá para a aba **Subscribe** e adicione os seguintes tópicos:

| Tópico                   | O que você verá        |
| ------------------------ | ---------------------- |
| `/TEF/device001/attrs/t` | Temperatura em °C      |
| `/TEF/device001/attrs/h` | Umidade em %           |
| `/TEF/device001/attrs/l` | Luminosidade em %      |
| `/TEF/device001/attrs/s` | Estado do LED (on/off) |

#### 🎛️ 4.4 - Controlar o LED

1. Vá para a aba **Publish**
2. Configure:
   - **Topic**: `/TEF/device001/cmd`
   - **Message**: `device001@on|` (para ligar) ou `device001@off|` (para desligar)
3. Toque em **Publish**

O LED no Wokwi deve responder imediatamente! 💡

---

## 📊 Testando o Sistema Completo

### ✅ Checklist de Verificação

- [ ] Containers Docker rodando na VM AWS
- [ ] Health Check do Orion respondendo corretamente
- [ ] Dispositivo provisionado no FIWARE
- [ ] Wokwi conectado ao broker MQTT
- [ ] MyMQTT recebendo dados dos sensores
- [ ] LED responde aos comandos do MyMQTT
- [ ] Postman consegue consultar dados via Orion

### 🧪 Fluxo de Teste Completo

1. **Inicie a simulação no Wokwi** → Aguarde conexão WiFi e MQTT
2. **Abra o Serial Monitor no Wokwi** → Verifique se os dados estão sendo publicados
3. **Abra o MyMQTT** → Confirme que está recebendo temperatura, umidade e luminosidade
4. **No Postman, faça um GET** → Verifique se o Orion tem os dados atualizados
5. **No MyMQTT, publique `device001@on|`** → O LED deve acender
6. **No MyMQTT, publique `device001@off|`** → O LED deve apagar

---

## 📚 Estrutura do Repositório

```
📦 projeto-adega-iot/
├── 📁 devices/
│   ├── sketch.ino          # Código do ESP32
│   └── diagram.json        # Configuração do circuito Wokwi
├── 📁 postman/
│   └── fiware-collection.json  # Collection do Postman
├── 📄 README.md            # Este arquivo
└── 📄 LICENSE
```

---

## 🎯 Conceitos Aprendidos

- ✅ Protocolo MQTT (Publish/Subscribe)
- ✅ Arquitetura FIWARE para IoT
- ✅ Provisionamento de dispositivos IoT
- ✅ API REST (NGSI-v2)
- ✅ Containerização com Docker
- ✅ Cloud Computing (AWS EC2)
- ✅ Sensores e atuadores com ESP32
- ✅ Comunicação bidirecional IoT

---

## 📈 Próximas Melhorias

- [ ] Dashboard web com gráficos em tempo real
- [ ] Alertas quando temperatura/umidade saírem da faixa ideal
- [ ] API REST personalizada para consulta de dados

---

## 👨‍💻 Integrantes

**Otávio Inaba - 565003**
GitHub: [Otaaviio](https://github.com/Otaaviio)
inkedIn: [Otávio Inaba](www.linkedin.com/in/otávio-inaba)

**Eduardo Ulisses - 566339**
GitHub: [UlissesE](https://github.com/UlissesE)
inkedIn: [Eduardo Ulisses](https://www.linkedin.com/in/eduardo-ulisses/)

**Eduardo Duran - 562017**
GitHub: [dudus70](https://github.com/dudus70)
inkedIn: [Eduardo Duran Del Ciel](https://www.linkedin.com/in/eduardo-duran-del-ciel-762862359/)

**Fernando Bellegarde - 564169**
GitHub: [fernandoBellegarde](https://github.com/fernandoBellegarde)
inkedIn: [Fernando Bellegarde](https://www.linkedin.com/in/fernandobellegarde/)

**Henrique Castro - 564560**
GitHub: [RickkCastro](https://github.com/RickkCastro)
inkedIn: [RickkCastro](https://www.linkedin.com/in/rickkcastro/)

**Henrique Guedes - 562474**
GitHub: [HenriGuedes](https://github.com/HenriGuedes)
inkedIn: [Henrique Guedes](https://www.linkedin.com/in/henrique-guedes-silv/)

---

## 📚 Referências

- [FIWARE Documentation](https://fiware.org/)
- [FIWARE Descomplicado](https://github.com/fabiocabrini/fiware)
- [ESP32 Documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/)
- [Wokwi Simulator](https://wokwi.com/)
- [MQTT Protocol](https://mqtt.org/)
- [DHT22 Datasheet](https://www.sparkfun.com/datasheets/Sensors/Temperature/DHT22.pdf)
- [A LENDA ABSOLUTA](https://www.youtube.com/@lucasaugusto6019)

---

<div align="center">

### ⭐ Se este projeto foi útil, considere dar uma estrela!

**Feito com ❤️ para a comunidade IoT**

🍷 _"Tecnologia e vinhos: ambos melhoram com o tempo e o cuidado certo"_

</div>
