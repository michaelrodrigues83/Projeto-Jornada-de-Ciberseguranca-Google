# Estudo de Caso: Análise de Incidente de Segurança (Brute Force & Redirecionamento HTTP)

Este projeto prático faz parte do **Curso 3 (Redes de Computadores e Cibersegurança)** do programa oficial do **Certificado Profissional de Cibersegurança da Google**. O objetivo é aplicar conceitos de análise de rede, resposta a incidentes e triagem de vulnerabilidades em um cenário corporativo simulado.

---

## 📌 Visão Geral da Atividade
Nesta atividade, assumo o papel de um **analista de segurança cibernética** encarregado de investigar, identificar, documentar e recomendar soluções para um incidente de segurança grave ocorrido em um servidor web corporativo. 

O objetivo principal é demonstrar competência técnica em:
* Análise de tráfego de rede e logs (`tcpdump`).
* Identificação de protocolos nas camadas do modelo TCP/IP.
* Documentação formal e estruturada de incidentes em terceira pessoa.
* Proposta de remediações e controles de segurança baseados nas melhores práticas do mercado.

---

## 📖 Cenário do Incidente
Você é um analista de segurança cibernética do site `yummyrecipesforme.com` (uma plataforma de venda de receitas e livros digitais). Um ex-funcionário insatisfeito decidiu atacar a infraestrutura e atrair os usuários para um ambiente malicioso.

### Linha do Tempo e Evidências:
1. **O Ataque Inicial:** O atacante executou um ataque de força bruta (*brute force*) contra o host da Web. Ele adivinhou a senha da conta administrativa facilmente, pois ela ainda estava definida com a **senha padrão de fábrica**, e não havia mecanismos de controle em vigor para mitigar múltiplas tentativas de login.
2. **Injeção de Código:** Após obter acesso ao painel de administração, o invasor alterou o código-fonte do site principal e injetou uma função em **JavaScript**. Esse script solicitava aos visitantes que baixassem e executassem um arquivo executável disfarçado de atualização de navegador para ter acesso a receitas gratuitas.
3. **Persistência e Impacto:** Após a implantação do malware, o hacker alterou a senha administrativa definitiva, bloqueando o acesso do proprietário legítimo. Nos computadores dos clientes que executaram o arquivo, o sistema começou a apresentar lentidão extrema e o tráfego passou a ser redirecionado para um domínio malicioso (`greatrecipesforme.com`).
4. **Descoberta:** O incidente foi detectado após múltiplos e-mails de reclamação enviados ao helpdesk e a confirmação do proprietário sobre a perda de acesso à conta.

### Registros de Processo do Sistema (Logs):
De acordo com a investigação técnica e o comportamento analisado em ambiente isolado (*sandbox*), o fluxo de rede registrou as seguintes etapas:
* **Passo 1:** O navegador inicia uma solicitação de DNS para o URL `yummyrecipesforme.com` ao servidor DNS.
* **Passo 2:** O DNS responde com o endereço IP correto.
* **Passo 3:** O navegador inicia uma solicitação HTTP para a página Web usando o IP enviado.
* **Passo 4:** O navegador inicia o download do malware provocado pela função JavaScript.
* **Passo 5:** O navegador inicia uma nova solicitação de DNS para o domínio malicioso `greatrecipesforme.com`.
* **Passo 6:** O servidor DNS responde com o endereço IP de `greatrecipesforme.com`.
* **Passo 7:** O navegador inicia uma solicitação HTTP para o novo IP do site falso.

---

# 📑 Security Incident Report

## Section 1: Identify the network protocol involved in the incident
O principal protocolo de rede da camada de aplicação envolvido na atividade maliciosa e na transferência do arquivo no cenário é o **HTTP (Hypertext Transfer Protocol)**. 

As requisições feitas aos servidores web para carregar a página do site `yummyrecipesforme.com` envolvem tráfego HTTP. Além disso, a análise dos registros do `tcpdump` confirma que, após a resolução inicial do endereço IP via **DNS**, a conexão com o servidor e o transporte do arquivo executável malicioso para os computadores dos usuários finais ocorreram estritamente através do protocolo HTTP na camada de aplicação do modelo TCP/IP.

---

## Section 2: Document the incident
Múltiplos clientes entraram em contato com o helpdesk do site informando que, ao acessar a página, foram induzidos a baixar e executar um arquivo que prometia acesso a novas receitas gratuitas. Desde a execução desse arquivo, os computadores pessoais desses clientes passaram a apresentar lentidão extrema. Simultaneamente, o proprietário do site relatou que foi bloqueado e não conseguiu realizar o login administrativa no servidor web.

Para investigar o ocorrido sem comprometer a rede da organização, o analista de cibersegurança utilizou um ambiente controlado (*sandbox*) para interagir com o site. O analista executou a ferramenta `tcpdump` para capturar os pacotes de tráfego de rede gerados pela página. Ao carregar o site, o analista recebeu o alerta para baixar o arquivo executável de receitas, aceitou o download e o executou, o que fez com que o navegador o redirecionasse imediatamente para um site falso (`greatrecipesforme.com`).

Ao inspecionar o log do `tcpdump`, o analista Pipeline observou que o navegador solicitou inicialmente o endereço IP do domínio legítimo (`yummyrecipesforme.com`). Assim que a conexão HTTP foi estabelecida e o arquivo foi executado, os registros apontaram uma mudança abrupta no tráfego de rede: o navegador disparou uma nova requisição DNS para obter o IP do domínio `greatrecipesforme.com`, redirecionando todo o tráfego subsequente para este novo endereço sob o protocolo HTTP.

Por fim, o profissional sênior de cibersegurança analisou o código-fonte das páginas e o arquivo baixado. Descobriu-se que o invasor manipulou o site original para injetar um código JavaScript que forçava o download do malware disfarçado. Cruzando os fatos com o bloqueio do proprietário, a equipe concluiu que a causa raiz foi um ataque de força bruta (*brute force attack*), facilitado pelo uso de uma senha padrão de administrador e pela ausência de controles de tentativas de login. A execução do arquivo comprometeu diretamente os dispositivos dos usuários finais.

---

## Section 3: Recommend one remediation for brute force attacks
Para proteger a organização contra futuros ataques de força bruta, a equipe planeja implementar e reforçar as seguintes medidas de remediação:

* **Proibir o uso de senhas anteriores (Disallow previous passwords):** Como a vulnerabilidade explorada foi a persistência de uma senha padrão de fábrica, essa política garante que senhas antigas ou padrões nunca possam ser reutilizadas após a primeira alteração.
* **Exigência de senhas longas e complexas:** Implementar uma política que exija credenciais com no mínimo 15 caracteres, aumentando exponencialmente o tempo e a capacidade computacional necessários para que uma ferramenta automatizada de força bruta consiga adivinhar a combinação.
* **Autenticação de Dois Fatores (2FA):** Adicionar uma camada essencial de defesa que exige, além da senha, um código de verificação único enviado ao dispositivo do usuário. Mesmo que um agente malicioso descubra a senha por força bruta, o acesso ao sistema será bloqueado pela falta do segundo fator de autenticação.

---

## 💡 Lições Aprendidas
Este estudo de caso trouxe aprendizados fundamentais sobre a rotina de um analista do SOC (Security Operations Center):
* **Segurança na Configuração Inicial:** Credenciais padrão (*default passwords*) são uma das portas de entrada mais comuns para invasores e devem ser eliminadas imediatamente no deploy de qualquer aplicação.
* **O Poder da Resposta Cronológica:** Documentar incidentes de forma profissional exige um texto impessoal (em terceira pessoa) focado rigorosamente em fatos, cronologia dos logs e evidências coletadas em *sandbox*.
* **Interdependência de Protocolos:** Compreender o modelo TCP/IP na prática permite correlacionar o comportamento de requisições DNS com transferências HTTP para rastrear o caminho exato de redirecionamentos maliciosos.

---

## 🤝 Contato
Sinta-se à vontade para se conectar comigo, trocar ideias sobre cibersegurança ou acompanhar minha evolução na área!

* **LinkedIn:** [Insira aqui o link do seu perfil do LinkedIn]
