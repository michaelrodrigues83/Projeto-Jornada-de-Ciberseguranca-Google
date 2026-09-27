# Relatório de Incidente de Segurança Cibernética: Análise de Ataque SYN Flood

> 🎓 **Nota de Contexto:** Laboratório prático desenvolvido como parte do **Subcurso 3 (Conectar e Proteger: Redes em Segurança Cibernética)** do programa **Google Cybersecurity Professional Certificate**. O objetivo desta atividade é identificar um ataque de rede na camada de transporte usando logs do Wireshark, compreender seu impacto e propor mitigações.

---

## 💻 Cenário do Incidente

Certa tarde, o sistema de monitoramento automático emitiu um alerta sobre o servidor da Web corporativo (`192.0.2.1`). Ao tentar acessar a página de vendas (`sales.html`), o navegador apresentou um erro de **"Pausa na conexão"** (*timeout*). Uma captura de pacotes revelou uma quantidade massiva e contínua de solicitações vindas de um único IP desconhecido (`203.0.113.0`), caracterizando uma interrupção de serviço.

---

## 🛠️ Fundamentos Técnicos

Para compreender este incidente, é necessário analisar como funciona o aperto de mão de três vias (**TCP Three-Way Handshake**):

1.  **Fluxo Normal:** O cliente envia um pacote `[SYN]` (sincronizar); o servidor responde com um `[SYN, ACK]` (sincronizar e confirmar) e reserva recursos de memória; o cliente finaliza enviando um pacote `[ACK]` (confirmar).
2.  **O Ataque (SYN Flood):** O atacante envia milhares de pacotes `[SYN]` por segundo. O servidor responde com `[SYN, ACK]` para cada um e fica aguardando a confirmação final (`[ACK]`), que nunca chega. Isso esgota a memória e os recursos do servidor, impedindo-o de atender usuários legítimos.
3.  **Mensagens de Erro Geradas:**
    *   **`[RST, ACK]` (Reset):** Enviado quando o servidor está sobrecarregado e força a queda das conexões que tentam se estabelecer.
    *   **504 Gateway Time-out:** Ocorre quando o servidor demora tanto para responder que o servidor de borda/gateway desiste da requisição.

---

### 📊 Análise dos Logs no Microsoft Excel

Como parte da investigação, os registros de tráfego foram exportados e analisados em formato de planilha para facilitar a identificação e categorização das anomalias de rede por cores (Verde: tráfego normal; Vermelho: atividade do ataque; Amarelo: falhas de conexão):

![Logs do Wireshark](logs-wireshark.png)

<details>
<summary>🔍 Clique aqui para visualizar o registro bruto completo extraído (CSV)</summary>

```
No.,Time,Source,Destination ,Protocol,Info
47,3.144521,198.51.100.23,192.0.2.1,TCP,42584->443 [SYN] Seq=0 Win-5792 Len=120...
48,3.195755,192.0.2.1,198.51.100.23,TCP,"443->42584 [SYN, ACK] Seq=0 Win-5792 Len=120..."
49,3.246989,198.51.100.23,192.0.2.1,TCP,42584->443 [ACK] Seq=1 Win-5792 Len=120...
50,3.298223,198.51.100.23,192.0.2.1,HTTP ,GET  /sales.html HTTP/1.1
51,3.349457,192.0.2.1,198.51.100.23,HTTP ,HTTP/1.1 200 OK (text/html)
red,52,3.390692,203.0.113.0,192.0.2.1,TCP,54770->443 [SYN] Seq=0 Win=5792 Len=0...
red,53,3.441926,192.0.2.1,203.0.113.0,TCP,"443->54770 [SYN, ACK] Seq=0 Win-5792 Len=120..."
red,54,3.49316,203.0.113.0,192.0.2.1,TCP,54770->443 [ACK Seq=1 Win=5792 Len=0...
green,55,3.544394,198.51.100.14,192.0.2.1,TCP,14785->443 [SYN] Seq=0 Win-5792 Len=120...
green,56,3.599628,192.0.2.1,198.51.100.14,TCP,"443->14785 [SYN, ACK] Seq=0 Win-5792 Len=120..."
red,57,3.664863,203.0.113.0,192.0.2.1,TCP,54770->443 [SYN] Seq=0 Win=5792 Len=0...
green,58,3.7300969999999998,198.51.100.14,192.0.2.1,TCP,14785->443 [ACK] Seq=1 Win-5792 Len=120...
red,59,3.7953319999999997,203.0.113.0,192.0.2.1,TCP,54770->443 [SYN] Seq=0 Win-5792 Len=120...
green,60,3.8605669999999996,198.51.100.14,192.0.2.1,HTTP ,GET  /sales.html HTTP/1.1
red,61,3.9394989999999996,203.0.113.0,192.0.2.1,TCP,54770->443 [SYN] Seq=0 Win-5792 Len=120...
green,62,4.018431,192.0.2.1,198.51.100.14,HTTP ,HTTP/1.1 200 OK (text/html)
```
</details>

---

## 📝 Relatório de Incidente Preenchido

### Seção 1: Identificação do tipo de ataque que está causando a interrupção da rede

*   **Tipo de ataque identificado:** Trata-se de um ataque de **Negação de Serviço Direto (DoS - Denial of Service)** do tipo **SYN Flood**.
*   **Diferença entre DoS e DDoS:** Um ataque DoS direto se origina de uma **única fonte** (um único endereço IP), exatamente como observado neste log (IP `203.0.113.0`). Um ataque DDoS (Distribuído) utilizaria múltiplos IPs de origens diferentes espalhados pelo mundo.
*   **Tendências e Padrões observados nos logs:** 
    *   No início (itens 47 a 51), conexões legítimas de funcionários (`198.51.100.23`) funcionavam normalmente.
    *   A partir do item 52, o IP malicioso `203.0.113.0` começa a inundar a porta **443** (HTTPS) do servidor com pacotes `[SYN]` em intervalos de milissegundos.
    *   Mais adiante (a partir do item 73), o servidor começa a falhar, enviando pacotes `[RST, ACK]` e erros **504 Gateway Time-out** para conexões legítimas. Do item 125 em diante, o servidor para totalmente de responder aos usuários, registrando apenas o tráfego do ataque.

---

### Seção 2: Explicação de como o ataque afeta o mau funcionamento do site e impactos

*   **Como o ataque afetou a rede e o site:** O volume anormal de pacotes `[SYN]` inundou a tabela de conexões pendentes do servidor da Web. Como o atacante nunca envia o pacote `[ACK]` final para concluir o aperto de mão, as conexões legítimas entram em filas de espera até estourarem o tempo limite (*timeout*). Isso indisponibilizou completamente o acesso à página de vendas.
*   **Consequências negativas para a organização:** Como a empresa é uma agência de publicidade que depende do site para divulgar promoções e vendas, a queda do servidor impede os funcionários de trabalhar e bloqueia as compras dos clientes, gerando **perda financeira imediata** e **danos à reputação da marca**.
*   **Ações imediatas de contenção aplicadas:** O servidor foi temporariamente colocado off-line para liberar a memória sobrecarregada e recuperar o status operacional. Além disso, uma regra de bloqueio do IP do atacante (`203.0.113.0`) foi configurada no firewall da empresa.

---

## 🧠 O que eu aprendi com este laboratório

1.  **Identificar o SYN Flood:** Aprendi a identificar visualmente um ataque de inundação de conexões através da repetição exaustiva de sinalizações `[SYN]` vindas de um mesmo endereço.
2.  **Impacto nos usuários:** Compreendi como falhas de rede se traduzem em erros reais na tela do usuário, associando os pacotes `[RST, ACK]` e erros de *Gateway Time-out* à exaustão de recursos do servidor.
3.  **Fragilidade da mitigação simples:** Entendi que bloquear apenas um IP no firewall é uma solução temporária, já que atacantes avançados podem realizar *IP spoofing* (falsificação de IP).

---
📬 **Gostou do projeto? Vamos nos conectar!**
* https://www.linkedin.com/in/michael-hernandes-rodrigues-4b092b283/
