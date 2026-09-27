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

## 📝 Relatório de Incidente Preenchido

### Seção 1: Identificação do tipo de ataque que está causando a interrupção da rede

*   **Tipo de ataque identificado:** Trata-se de um ataque de **Negação de Serviço Direto (DoS - Denial of Service)** do tipo **SYN Flood**.
*   **Diferença entre DoS e DDoS:** Um ataque DoS direto se origina de uma **única fonte** (um único endereço IP), exatamente como observado neste log (IP `203.0.113.0`). Um ataque DDoS (Distribuído) utilizaria múltiplos IPs de origens diferentes espalhados pelo mundo, o que tornaria o bloqueio muito mais difícil.
*   **Tendências e Padrões observados nos logs:** 
    *   No início (itens 47 a 51), conexões legítimas de funcionários (`198.51.100.23`) funcionavam normalmente.
    *   A partir do item 52, o IP malicioso `203.0.113.0` começa a inundar a porta **443** (HTTPS) do servidor com pacotes `[SYN]` em intervalos de milissegundos.
    *   Mais adiante (a partir do item 73), o servidor começa a falhar, enviando pacotes `[RST, ACK]` e erros **504 Gateway Time-out** para conexões legítimas. Do item 125 em diante, o servidor para totalmente de responder aos usuários, registrando apenas o tráfego do ataque.

---

### Seção 2: Explicação de como o ataque afeta o mau funcionamento do site e impactos

*   **Como o ataque afetou a rede e o site:** O volume anormal de pacotes `[SYN]` inundou a tabela de conexões pendentes do servidor da Web. Como o atacante nunca envia o pacote `[ACK]` final para concluir o aperto de mão, as conexões legítimas dos funcionários e clientes entram em filas de espera até estourarem o tempo limite (*timeout*). Isso indisponibilizou completamente o acesso à página de vendas.
*   **Consequências negativas para a organização:** Como a empresa é uma agência de publicidade que depende do site para divulgar pacotes de férias, promoções e vendas, a queda do servidor impede os funcionários de trabalhar e bloqueia as compras dos clientes, gerando **perda financeira imediata** e **danos à reputação da marca**.
*   **Ações imediatas de contenção aplicadas:** O servidor foi temporariamente colocado off-line para liberar a memória sobrecarregada e recuperar o status operacional. Além disso, uma regra de bloqueio do IP do atacante (`203.0.113.0`) foi configurada no firewall da empresa.

---

## 🧠 O que eu aprendi com este laboratório

1.  **Identificar o SYN Flood:** Aprendi a identificar visualmente um ataque de inundação de conexões através da repetição exaustiva de sinalizações `[SYN]` vindas de um mesmo endereço.
2.  **Impacto nos usuários:** Compreendi como falhas de rede se traduzem em erros reais na tela do usuário, associando os pacotes `[RST, ACK]` e erros de *Gateway Time-out* à exaustão de recursos do servidor.
3.  **Fragilidade da mitigação simples:** Entendi que bloquear apenas um IP no firewall é uma solução temporária, já que atacantes avançados podem realizar *IP spoofing* (falsificação de IP) ou evoluir a ação para um ataque distribuído (DDoS).
