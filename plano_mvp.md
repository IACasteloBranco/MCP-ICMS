# Plano de implementação do MVP do MCP para o DOMÍNIO

## Objetivo

Entregar um agente de IA capaz de consultar, em modo somente leitura, um fluxo de dados do DOMÍNIO e apresentar o resultado com empresa, competência e origem identificadas. O MVP serve para validar o acesso técnico ao DOMÍNIO e o uso pelo agente. Ele não calcula tributos nem altera, transmite, encerra ou retifica dados fiscais.

O primeiro passo será avaliar o REA no DOMÍNIO. O REA ajudará a entender a interface web, os arquivos do plugin e os limites de observação. Ele será uma ferramenta de descoberta, não uma dependência do MCP em produção.

## Resultado esperado

Ao fim do MVP, o agente poderá solicitar um fluxo de consulta escolhido durante a descoberta. O MCP valida os parâmetros, consulta o DOMÍNIO por um caminho permitido e devolve dados estruturados, indicando erros e limitações.

O escopo inicial contém:

- um agente e um ambiente de homologação;
- uma empresa autorizada e uma competência;
- um fluxo de consulta, escolhido após a análise do REA;
- uma ou duas ferramentas MCP, apenas se ambas forem necessárias para esse fluxo;
- logs operacionais sem credenciais nem dados pessoais desnecessários.

## Fora do escopo

- gravação, preparação ou efetivação de lançamentos;
- transmissão de obrigações, encerramento de período, exclusão ou retificação;
- análise automática de NCM ou cálculo de ICMS;
- agendamento, fila remota, cofre de credenciais ou painel de aprovação;
- suporte a várias empresas, usuários ou fluxos fiscais;
- instalação do REA como parte do produto final.

## Plano de trabalho

### Etapa 1. Entender o REA e avaliar seu uso no DOMÍNIO

**Objetivo.** Primeiro entender como o REA funciona, o que ele consegue inspecionar e quais são seus limites. Depois, decidir se ele ajuda a investigar o DOMÍNIO e seu plugin, sem confundir investigação com automação operacional.

**Atividades.**

1. Ler a documentação e entender a arquitetura do REA, como o agente o acessa via MCP e quais tipos de análise oferece.
2. Separar os recursos relevantes para este projeto. A inspeção passiva do navegador pode ajudar no DOMÍNIO Web; a análise estática pode ajudar se o plugin for .NET. Registrar o que o REA não faz, como operar o DOMÍNIO livremente.
3. Confirmar a versão do DOMÍNIO, os módulos usados, o tipo do plugin e o agente de IA que receberá o MCP.
4. Confirmar que a licença e as regras aplicáveis permitem inspecionar a aplicação e seus arquivos.
5. Em uma sessão de homologação, testar a inspeção do DOMÍNIO Web pelo REA, se o navegador e a sessão forem compatíveis.
6. Identificar o formato e a tecnologia do plugin. Se for .NET, avaliar a inspeção estática do assembly. Se não for, registrar o limite encontrado.
7. Percorrer manualmente um fluxo fiscal candidato e anotar telas, entradas, saídas e exportações disponíveis.
8. Registrar evidências úteis, dados que não puderam ser observados e o método mais simples para obter a saída do fluxo.

**Decisão de saída.** Documentar o modelo de funcionamento, os recursos e os limites do REA. Depois, marcar seu uso no DOMÍNIO como útil, parcialmente útil ou não aplicável e escolher um único fluxo de consulta para o MVP. Não avançar para automação de UI enquanto houver uma exportação ou interface suportada que entregue os mesmos dados com menos risco.

**Critério de conclusão.** Existe uma descrição compreensível de como o REA funciona e do que não cobre, além de um fluxo documentado com entradas e saídas conhecidas. A equipe registra se usará o REA para esse fluxo. Se a licença, o acesso ou a interface impedirem a inspeção, registra o bloqueio e escolhe outra forma autorizada de descoberta.

**Nota de capacidade.** A documentação do REA descreve inspeção passiva de navegador e análise estática de aplicações .NET. A inspeção passiva não clica nem navega pela página. Cenários controlados podem executar ações declaradas, mas isso não transforma o REA em uma ferramenta geral de operação do DOMÍNIO. Consulte [observação de navegador](https://github.com/morluto/rea/blob/main/docs/browser-observation.md) e [análise de código .NET](https://github.com/morluto/rea/blob/main/docs/managed-code-analysis.md).

### Etapa 2. Provar a conexão com o agente

**Objetivo.** Confirmar como o agente escolhido inicia ou alcança um servidor MCP antes de implementar a integração fiscal.

**Atividades.**

1. Criar um servidor mínimo que exponha uma ferramenta de diagnóstico sem acessar o DOMÍNIO.
2. Configurar o servidor no agente selecionado e confirmar uma chamada e uma resposta.
3. Verificar se o agente pode iniciar um servidor local no Windows. Se exigir um MCP remoto, registrar essa necessidade antes de escolher a arquitetura de produção.

**Critério de conclusão.** O agente chama a ferramenta de diagnóstico e recebe uma resposta válida. A equipe conhece o transporte e os requisitos de hospedagem para a integração real.

### Etapa 3. Implementar uma consulta somente leitura

**Objetivo.** Conectar a ferramenta MCP ao fluxo escolhido na Etapa 1.

**Ordem de preferência para acesso.**

1. API ou integração oficial do DOMÍNIO, se disponível e autorizada.
2. Exportação ou relatório produzido pelo DOMÍNIO.
3. Automação local baseada em controles acessíveis e estado verificável da interface.

Não usar coordenadas de tela como mecanismo principal. Não ler ou alterar diretamente o banco de dados sem documentação e autorização para esse uso.

**Implementação.**

- Criar uma ferramenta MCP com parâmetros explícitos para o fluxo escolhido.
- Aceitar identificadores internos autorizados, nunca comandos livres do modelo.
- Validar empresa e competência antes de iniciar a consulta.
- Retornar dados estruturados com empresa, competência, horário da extração e origem.
- Parar e devolver um erro claro se a sessão, a tela ou o formato dos dados não corresponderem ao esperado.
- Manter credenciais fora do código, dos prompts e dos resultados da ferramenta.

Os nomes e campos finais da ferramenta serão definidos após a Etapa 1. Não criar ferramentas genéricas para fluxos ainda não selecionados.

### Etapa 4. Verificar o fluxo completo

**Objetivo.** Confirmar que o agente recebe a informação correta e que falhas não parecem respostas válidas.

**Verificações.**

- A empresa e a competência retornadas correspondem à solicitação autorizada.
- Os totais e registros retornados conferem com a tela ou exportação do DOMÍNIO.
- Parâmetros ausentes ou inválidos geram erro claro.
- Sessão expirada, DOMÍNIO indisponível ou tela inesperada interrompem o fluxo.
- Repetir uma consulta não altera dados no DOMÍNIO.
- Logs não registram senhas, tokens ou dados pessoais sem necessidade.

**Critério de conclusão.** Um usuário consegue pedir a consulta ao agente, revisar a resposta e rastrear de qual empresa, competência e origem vieram os dados.

## Arquitetura inicial

Manter o menor arranjo compatível com o agente escolhido:

```text
Agente de IA
    |
    | MCP
    v
Servidor MCP
    |
    | interface oficial, exportação ou automação local controlada
    v
DOMÍNIO em homologação
```

Se o agente executar no mesmo Windows e aceitar servidor MCP local, começar com um processo local e sem fila remota. Se o agente só alcançar servidores remotos, tratar hospedagem, autenticação e conexão de saída do worker como uma fase de arquitetura separada. Não implementar essa infraestrutura antes de confirmar o requisito do agente.

## Marcos e estimativa

As estimativas são janelas de trabalho, não compromissos. O acesso ao DOMÍNIO, o plugin e a configuração do agente ainda não foram inspecionados.

| Marco | Entrega | Estimativa |
| --- | --- | --- |
| 1 | Avaliação do REA e escolha de um fluxo de consulta | 0,5 a 1 dia |
| 2 | Prova de chamada MCP no agente escolhido | 0,5 dia |
| 3 | Consulta somente leitura de um fluxo | 1 a 3 dias |
| 4 | Verificação com dados de homologação e correções | 0,5 a 1 dia |

Se o DOMÍNIO não oferecer API ou exportação adequada e exigir automação visual instável, revisar o prazo após a Etapa 1. Não ampliar o MVP para compensar incertezas de acesso.

## Riscos e respostas

| Risco | Resposta no MVP |
| --- | --- |
| REA não consegue observar a versão ou o navegador usados | Registrar a limitação e mapear o fluxo manualmente ou com ferramentas suportadas |
| O plugin usa tecnologia que o REA não inspeciona | Tratar REA como opcional e identificar a interface operacional do plugin |
| O agente exige MCP remoto | Confirmar autenticação e hospedagem antes de ligar o MCP a dados reais |
| A interface muda durante a coleta | Parar em telas inesperadas e validar os dados antes de responder |
| Dados de produção são expostos durante a descoberta | Usar ambiente e empresa de homologação, limitar capturas e remover dados desnecessários |
| O método de acesso pode contrariar a licença ou contrato | Não inspecionar nem automatizar até confirmar autorização |

## Decisão ao fim do MVP

Prosseguir para um piloto somente se a consulta for repetível, os dados conferirem com o DOMÍNIO e o método de acesso estiver autorizado. Manter a primeira versão sem escrita. Avaliar motor fiscal, múltiplos fluxos, worker remoto e aprovações humanas como fases posteriores, com requisitos próprios.
