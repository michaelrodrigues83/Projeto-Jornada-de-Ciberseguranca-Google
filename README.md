# Relatório de Incidente de Segurança Cibernética: Análise de Tráfego de Rede

> 🎓 **Nota de Contexto:** Laboratório prático desenvolvido como parte do **Subcurso 3 (Conectar e Proteger: Redes em Segurança Cibernética)** do programa **Google Cybersecurity Professional Certificate**. O objetivo da atividade é simular a atuação de um analista de segurança na triagem de incidentes, análise de pacotes de rede e documentação técnica.

---

## 💻 Cenário do Incidente

Vários clientes relataram que não conseguiram acessar o site da empresa `www.yummyrecipesforme.com` e visualizaram o erro "porta de destino inalcançável" (*destination port unreachable*) após esperar a página carregar. Para investigar o incidente, o analista de segurança utilizou a ferramenta de análise de rede `tcpdump` para interceptar e analisar os pacotes durante a tentativa de conexão.

### 📊 Captura de Tela dos Logs (tcpdump)

Abaixo está o registro bruto analisado durante a triagem do incidente:

<img width="893" height="447" alt="image" src="https://github.com/user-attachments/assets/22e35f89-ae29-4b6f-8fa3-a5f5abf0a0da" />


---

## 🛠️ Fundamentos Técnicos

*   **tcpdump:** Ferramenta de linha de comando voltada para a análise de pacotes de rede (*packet sniffer*). Permite interceptar e exibir o tráfego de dados para identificar comportamentos anômalos.
*   **UDP (User Datagram Protocol):** Protocolo da camada de transporte que prioriza a velocidade. É o padrão para serviços rápidos como o **DNS (porta 53)**.
*   **ICMP (Internet Control Message Protocol):** Protocolo utilizado para enviar mensagens operacionais e relatórios de erro. Quando uma porta está fechada no destino, o ICMP devolve a mensagem de erro `port unreachable`.

---

## 📝 Relatório de Incidente Preenchido

### Parte 1: Resumo do problema encontrado no registro de tráfego DNS e ICMP

*   **O protocolo UDP revela que:** O computador do cliente (`192.51.100.15`) iniciou e enviou com sucesso pacotes de consulta DNS de saída para o servidor DNS (`203.0.113.2`) solicitando o endereço IP de `www.yummyrecipesforme.com`, mas os pacotes não puderam ser processados pelo serviço de destino.
*   **Isso é baseado nos resultados da análise de rede, que mostram que a resposta ICMP retornou a mensagem de erro:** `udp port 53 unreachable` (porta de destino ICMP inalcançável).
*   **A porta indicada na mensagem de erro é usada para:** Tráfego e serviços de **DNS (Domain Name System)**.
*   **O problema mais provável é:** O serviço de DNS no servidor de destino (`203.0.113.2`) está totalmente offline, desativado ou o tráfego para a **porta UDP 53** está sendo bloqueado ativamente por um firewall, impedindo a resolução de nomes de domínio.

### Parte 2: Análise dos dados e causa do incidente

*   **Horário em que o incidente ocorreu:** O registro capturado mostra os erros se repetindo exatamente às **13:24:32**, **13:26:32** e **13:28:32**.
*   **Explique como a equipe de TI tomou conhecimento do incidente:** Vários clientes relataram que não conseguiram acessar o site da empresa e visualizaram o erro de porta inalcançável.
*   **Explique as ações tomadas pelo departamento de TI para investigar o incidente:** Um analista de segurança cibernética testou o acesso ao site, confirmou o erro e utilizou a ferramenta `tcpdump` para capturar e inspecionar os pacotes de rede enquanto simulava o acesso.
*   **Observe as principais descobertas da investigação do departamento de TI:** Os registros mostram um padrão claro: toda vez que o cliente envia uma consulta UDP para o servidor DNS (`203.0.113.2.domain`), o servidor retorna imediatamente uma resposta ICMP informando `udp port 53 unreachable`. Isso prova que o servidor está ativo na rede, mas rejeita os pacotes direcionados à sua porta de serviço DNS.
*   **Observe uma causa provável para o incidente:** A causa mais provável é uma **falha ou má configuração no software do servidor DNS**, fazendo com que o serviço na porta 53 parasse de funcionar. Outras causas possíveis incluem uma alteração recente de regras no firewall bloqueando a porta 53 ou um ataque de Negação de Serviço (DoS) que derrubou o serviço DNS.
