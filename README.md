# 🛡️ Enterprise Network Redundancy & Cyber Defense Topology Lab (v2)

## 📌 Visão Geral do Projeto
Este repositório contém o projeto de infraestrutura de rede corporativa de alta disponibilidade e resiliência de borda simulado no **Cisco Packet Tracer 9.0.1**.

O ambiente foi reestruturado e projetado para servir como base técnica para estudos em **Defesa em Cibersegurança (Blue Team / SOC)** e Engenharia de Redes, integrando redundância de gateway na LAN via **HSRP**, suporte a múltiplos enlaces WAN, roteamento dinâmico centralizado via **OSPF (Área 0)**, atribuição dinâmica via **DHCP**, tradução de endereços via **NAT/PAT** e comunicação ponta a ponta com a Internet pública (`8.8.8.8`).

---

## 🔑 Credenciais e Acesso
* **Senha Global dos Roteadores (`enable` / Console / VTY):** `1234`

---

## 🏗️ Arquitetura e Especificações Técnicas

### 1. Matriz (LAN, VLANs & Redundância HSRP)
- **VLAN 10 (Vendas):** Sub-rede `192.168.0.0/25` | Subinterface `.10`
- **VLAN 20 (Diretoria):** Sub-rede `192.168.0.128/25` | Subinterface `.20`
- **Gateway Virtual HSRP (Grupo 10 & 20):** `192.168.0.1` / `192.168.0.129`
- **R1 MATRIZ (Master / Active):**
  * Subinterface `Gi0/0.10`: `192.168.0.2/25`
  * Subinterface `Gi0/0.20`: `192.168.0.130/25`
  * Interface WAN `Gi0/2`: `200.100.0.1/24` (`ip ospf priority 0`)
- **R2 MATRIZ (Standby / Backup):**
  * Subinterface `Gi0/0.10`: `192.168.0.3/25`
  * Subinterface `Gi0/0.20`: `192.168.0.131/25`
  * Interface WAN `Gi0/2`: `200.100.0.2/24` (`ip ospf priority 0`)

### 2. Filial 1 (Site Remoto - LAN & DHCP)
- **Rede LAN:** `192.168.1.0/24`
- **Gateway Local (`R1 FILIAL 1`):** `192.168.1.1/24`
- **Interface WAN `Gi0/2`:** `200.100.0.3/24` (`ip ospf priority 0`)
- **Serviço DHCP:** Distribuição dinâmica de IP, Gateway (`192.168.1.1`) e DNS (`8.8.8.8`) para os hosts locais.

### 3. Backbone de Provedor & Internet (ISP Area)
- **ROTEADOR_ISP (Designated Router / Backbone Central):**
  * Interface WAN LAN/Cloud `Gi0/0`: `200.100.0.254/24` (`ip ospf priority 255`)
  * Interface Link Google `Gi0/1`: `200.200.100.2/30`
- **ROTEADOR GOOGLE (Servidor Externo):**
  * Interface WAN `Gi0/2`: `200.200.100.1/30`
  * Interface Servidor `Gi0/0`: `8.8.8.1/24`
  * **Google Web Server:** `8.8.8.8/24` (HTTP / ICMP)

### 4. Roteamento Dinâmico OSPF (Área 0) & Failover
- **Convergência Total OSPF:** Interconexão dinâmica entre Matriz, Filial, Provedor e Google Server.
- **Tuning de DR/BDR:** Configuração de prioridades OSPF na WAN (`255` no ISP e `0` nos clientes) para garantir a eleição determinística do `ROTEADOR_ISP` como **Designated Router (DR)** na nuvem compartilhada `Cloud-PT`.
- **Injeção Dinâmica de Default Route:** Comando `default-information originate always` executado no `ROTEADOR_ISP` propagando a rota padrão `0.0.0.0/0` para todos os roteadores corporativos.
- **Resolução de Adjacência MTU:** Aplicação do parâmetro `ip ospf mtu-ignore` em todas as interfaces WAN para prevenção do travamento `EXSTART/DR`.

---

## 🧪 Bateria de Testes e Validação (Homologados)

- [x] **Obtenção de IP e Gateway na Filial:** PCs da Filial recebem IP dinâmico via DHCP e comunicam com a LAN da Matriz.
- [x] **Alta Disponibilidade HSRP:** Desconexão simulatória do `R1 MATRIZ` aciona o failover imediato para o `R2 MATRIZ` sem interrupção de tráfego dos clientes.
- [x] **Convergência Dinâmica OSPF:** Verificação via `show ip ospf neighbor` confirmando estado `FULL` entre todos os vizinhos.
- [x] **Resolução de Rota Padrão:** Checagem via `show ip route` nos roteadores de borda confirmando a rota recebida `O*E2 0.0.0.0/0`.
- [x] **Navegação Externa Fim a Fim (ICMP & HTTP):** Sucesso no `ping 8.8.8.8` e abertura da página Web do Google Server a partir de qualquer PC da Matriz e da Filial.
- [x] **Rastreamento de Saltos:** Execução do `tracert 8.8.8.8` validando a passagem pelos roteadores de borda, ISP e destino final.

---

## 🚀 Como Executar no Cisco Packet Tracer

### 1. Clonar o Repositório
```bash
git clone [https://github.com/SEU_USUARIO/rede-e-ciberseguranca.git](https://github.com/SEU_USUARIO/rede-e-ciberseguranca.git)
cd rede-e-ciberseguranca/cisco-packet-tracer
