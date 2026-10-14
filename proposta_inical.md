\## Arquitetura recomendada



```text

Agente no ChatGPT

&#x20;       │ MCP via HTTPS + OAuth

&#x20;       ▼

Servidor MCP / Orquestrador

&#x20;       │

&#x20;       ├── Banco de tarefas e auditoria

&#x20;       ├── Cofre de credenciais

&#x20;       ├── Motor fiscal e fontes oficiais

&#x20;       └── Fila de comandos

&#x20;                ▲ conexão de saída

&#x20;                │

&#x20;         Worker Windows local

&#x20;                │

&#x20;         RPA/UI Automation

&#x20;                ▼

&#x20;      DOMÍNIO WEB + plugin local

```



O servidor MCP não acessa diretamente a máquina. O \*\*worker Windows inicia uma conexão de saída\*\*, consulta tarefas pendentes e devolve resultados. Isso evita expor a máquina à internet.



\## 1. Componentes



\### Servidor MCP



Pode ser implementado em Python ou TypeScript e hospedado em Azure, AWS ou infraestrutura própria.



Responsabilidades:



\- autenticar o agente;

\- validar permissões por empresa;

\- receber solicitações;

\- criar tarefas idempotentes;

\- disponibilizar tarefas ao worker;

\- armazenar resultados e evidências;

\- controlar aprovação humana;

\- disponibilizar relatórios ao agente.



\### Worker Windows



Serviço instalado na máquina que executa o DOMÍNIO.



Responsabilidades:



\- manter comunicação segura com o servidor;

\- verificar se o DOMÍNIO está disponível;

\- abrir ou reutilizar uma sessão autorizada;

\- selecionar empresa e competência;

\- extrair cadastros e movimentações;

\- executar automação somente dentro de fluxos permitidos;

\- gerar capturas e logs;

\- interromper diante de telas ou valores inesperados.



Tecnologias possíveis:



\- .NET Worker Service, recomendado para Windows;

\- Power Automate Desktop;

\- UiPath;

\- FlaUI ou Microsoft UI Automation;

\- Playwright somente para partes realmente acessíveis pelo navegador.



Automação por coordenadas de tela deve ser evitada. Prefira seletores de acessibilidade, IDs de controles e validação do estado da tela.



\### Motor fiscal



Deve ficar separado da automação visual.



Funções:



\- normalizar descrições de produtos;

\- validar formato e vigência do NCM;

\- consultar fontes governamentais autorizadas;

\- comparar cadastro com documentos fiscais;

\- identificar divergências;

\- calcular cenários de ICMS;

\- registrar fundamento, data e fonte;

\- atribuir nível de confiança;

\- encaminhar casos ambíguos para análise humana.



O motor não deve determinar NCM apenas pela descrição resumida do produto. Composição, finalidade, apresentação e regras classificatórias podem ser necessárias.



\## 2. Ferramentas do MCP



\### Consulta



\- `list\_companies`

&#x20; - Lista apenas empresas autorizadas para o usuário.



\- `get\_company\_tax\_context`

&#x20; - Retorna CNPJ mascarado, UF, regime, estabelecimentos e competência disponível.



\- `get\_worker\_status`

&#x20; - Verifica máquina, worker, sessão e versão da automação.



\- `extract\_product\_registry`

&#x20; - Extrai produtos, descrições, NCM, CEST e situação tributária.



\- `extract\_icms\_data`

&#x20; - Extrai valores, CFOP, CST, bases, créditos e débitos da competência.



\- `get\_job\_status`

&#x20; - Consulta progresso e eventual bloqueio.



\- `get\_job\_evidence`

&#x20; - Retorna logs, totais, divergências e capturas permitidas.



\### Análise



\- `validate\_ncm\_batch`

&#x20; - Compara os NCMs com fontes oficiais e gera justificativas.



\- `analyze\_icms`

&#x20; - Apura divergências e cenários sem modificar o DOMÍNIO.



\- `generate\_review\_package`

&#x20; - Produz relatório de conferência, memória de cálculo e lista de pendências.



\### Preparação no DOMÍNIO



\- `prepare\_icms\_entries`

&#x20; - Preenche uma área de rascunho ou prepara uma proposta de lançamentos.



\- `validate\_prepared\_entries`

&#x20; - Relê os valores preparados e compara com a proposta aprovada.



\- `discard\_prepared\_entries`

&#x20; - Descarta o lote ainda não confirmado.



\### Aprovação controlada



\- `request\_human\_review`

&#x20; - Bloqueia o fluxo e cria um pacote de revisão.



\- `approve\_prepared\_entries`

&#x20; - Libera somente o lote e o hash aprovados.



\- `apply\_approved\_entries`

&#x20; - Efetiva lançamentos previamente aprovados.



No primeiro ciclo, eu \*\*não disponibilizaria transmissão de obrigação, encerramento de período, exclusão ou retificação\*\*. Essas ações devem permanecer manuais até o processo amadurecer.



\## 3. Ciclo operacional



1\. O agendamento inicia uma execução do agente.

2\. O agente solicita a análise de uma empresa e competência.

3\. O MCP verifica escopo, autorização e duplicidade.

4\. Uma tarefa é criada com identificador único.

5\. O worker coleta os dados no DOMÍNIO.

6\. O motor fiscal valida NCM e calcula o ICMS.

7\. O MCP produz o pacote de revisão.

8\. Um responsável revisa divergências, bases e totais.

9\. Após aprovação, o worker prepara os lançamentos.

10\. O worker relê o que foi preenchido.

11\. O sistema apresenta uma comparação final.

12\. A efetivação ocorre somente após nova confirmação, se necessária.



\## 4. Modelo de estados



```text

CREATED

→ QUEUED

→ COLLECTING

→ ANALYZING

→ AWAITING\_REVIEW

→ APPROVED

→ PREPARING\_ENTRIES

→ VALIDATING\_ENTRIES

→ COMPLETED

```



Estados alternativos:



\- `BLOCKED`

\- `REJECTED`

\- `CANCELLED`

\- `NEEDS\_MANUAL\_ACTION`

\- `FAILED\_RETRYABLE`

\- `FAILED\_FINAL`



Cada transição deve ser registrada com horário, usuário, máquina, versão do worker e hash dos dados.



\## 5. Segurança



\### Comunicação



\- HTTPS com TLS 1.2 ou superior;

\- OAuth 2.1 para o MCP;

\- autenticação mútua ou certificado individual do worker;

\- rotação de certificados;

\- worker operando somente por conexão de saída;

\- nenhuma porta aberta diretamente na máquina.



\### Credenciais



\- não salvar senhas no código ou nos prompts;

\- usar Windows Credential Manager ou cofre corporativo;

\- usar conta exclusiva para automação;

\- aplicar privilégio mínimo;

\- separar ambientes de teste e produção.



\### Autorização



Aplicar permissões por:



\- escritório;

\- empresa;

\- estabelecimento;

\- competência;

\- tipo de operação;

\- usuário aprovador.



O agente nunca deve escolher livremente um CNPJ fora do escopo autorizado.



\### Proteções operacionais



\- lista permitida de telas e comandos;

\- limite de empresas e lançamentos por execução;

\- bloqueio para competências encerradas;

\- idempotência para evitar duplicidade;

\- validação de totais antes e depois;

\- botão de emergência;

\- expiração das aprovações;

\- bloqueio após alteração dos dados aprovados.



\### LGPD e auditoria



\- mascarar CNPJ, CPF e dados pessoais quando possível;

\- criptografar dados armazenados;

\- definir retenção de relatórios e imagens;

\- registrar acessos e alterações;

\- evitar capturas com dados desnecessários;

\- manter trilha que permita reconstruir cada decisão.



\## 6. Revisão humana



O pacote de revisão deve mostrar:



\- empresa, competência e estabelecimento;

\- data da extração;

\- NCM atual e NCM sugerido;

\- descrição e características usadas;

\- fonte oficial consultada;

\- nível de confiança;

\- CFOP, CST/CSOSN e CEST relacionados;

\- base, alíquota, crédito, débito e diferença;

\- lançamentos propostos;

\- alertas e informações ausentes;

\- capturas relevantes;

\- hash imutável do lote.



Aprovação deve ser específica para:



\- uma empresa;

\- uma competência;

\- uma versão do conjunto de dados;

\- um conjunto exato de lançamentos.



Qualquer alteração invalida a aprovação anterior.



\## 7. Agendamentos



O agendamento pode iniciar a análise, mas não deve concluir automaticamente ações fiscais críticas.



Exemplo:



\- diariamente às 20h: verificar disponibilidade e extrair cadastros;

\- semanalmente: conferir NCMs alterados;

\- mensalmente: preparar análise de ICMS da competência;

\- após análise: notificar que o pacote aguarda revisão;

\- após aprovação: preparar lançamentos;

\- transmissão e encerramento: inicialmente manuais.



Requisitos:



\- computador ligado;

\- worker ativo;

\- usuário/sessão do Windows compatível com o RPA;

\- DOMÍNIO disponível;

\- ausência de atualização ou tela modal;

\- mecanismo de retomada após falhas.



\## 8. Contrato de uma tarefa



```json

{

&#x20; "company\_id": "empresa-interna",

&#x20; "period": "2026-03",

&#x20; "operation": "prepare\_icms\_review",

&#x20; "requested\_by": "usuario",

&#x20; "idempotency\_key": "empresa:2026-03:prepare:v1",

&#x20; "dry\_run": true,

&#x20; "requires\_human\_approval": true

}

```



O MCP deve trabalhar com identificadores internos, nunca com credenciais ou comandos livres enviados pelo modelo.



\## 9. Plano de implementação



\### Fase 1 — Descoberta



\- mapear versão e módulos do DOMÍNIO;

\- identificar telas, exportações e relatórios;

\- verificar regras de licenciamento;

\- documentar o fluxo manual;

\- selecionar três empresas fictícias ou de homologação;

\- catalogar fontes oficiais de NCM e ICMS.



\### Fase 2 — Prova de conceito somente leitura



\- instalar o worker;

\- detectar DOMÍNIO e empresa ativa;

\- extrair cadastro de produtos;

\- extrair uma competência de ICMS;

\- produzir relatório sem escrever nada;

\- medir estabilidade dos seletores.



\### Fase 3 — Motor fiscal



\- implementar validação de NCM;

\- versionar tabelas e fontes;

\- implementar regras de ICMS por UF e regime;

\- gerar memória de cálculo;

\- criar fila de exceções humanas.



\### Fase 4 — MCP



\- publicar o servidor HTTPS;

\- implementar autenticação;

\- expor ferramentas somente de leitura;

\- adicionar tarefas, status e evidências;

\- integrar o agente.



\### Fase 5 — Preparação de lançamentos



\- começar em ambiente de homologação;

\- preencher sem confirmar;

\- reler e comparar os valores;

\- exigir aprovação;

\- testar idempotência e recuperação.



\### Fase 6 — Produção controlada



\- piloto com poucas empresas;

\- limites baixos por execução;

\- revisão de 100% dos casos;

\- monitoramento e alertas;

\- procedimento de rollback;

\- auditoria após cada competência.



\### Fase 7 — Expansão



Somente após métricas satisfatórias:



\- ampliar empresas;

\- incluir novas operações;

\- reduzir revisão de casos de baixo risco;

\- considerar efetivação assistida;

\- manter transmissão e retificação sob controles reforçados.



\## 10. Critérios de aceite



Antes de produção:



\- nenhuma duplicidade em testes repetidos;

\- rastreabilidade integral;

\- 100% das gravações vinculadas a aprovação válida;

\- recuperação segura após queda;

\- bloqueio correto em tela inesperada;

\- comparação automática antes/depois;

\- precisão fiscal validada por especialista;

\- teste de atualização do DOMÍNIO;

\- revisão jurídica, contratual e de LGPD.



\*\*Primeiro produto recomendado:\*\* MCP e worker somente de leitura, capazes de extrair dados do DOMÍNIO, conferir NCM e ICMS e gerar um pacote de revisão. A escrita deve entrar apenas na segunda etapa.

