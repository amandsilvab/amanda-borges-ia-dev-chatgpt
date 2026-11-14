# DETALHAMENTO --- Versionamento de Código com Uso de IA

## 1. Objetivo

Documentar a análise funcional da aplicação Service Desk, os problemas
identificados, as correções registradas e os testes realizados com apoio
de inteligência artificial, além de organizar as informações necessárias
para o versionamento do projeto.

## 2. Preparação e escopo da análise

A análise foi realizada sobre a aplicação Service Desk, com foco nos
fluxos de organização da fila de atendimento, resolução de chamados e
pesquisa de chamados.

De acordo com o diagnóstico fornecido, os arquivos envolvidos nos
problemas descritos foram:

-   `app.js`: contém a lógica de ordenação, atualização e filtragem dos
    chamados.
-   `index.html`: contém a interface e a indicação dos campos e
    funcionalidades disponíveis.

O projeto utiliza `localStorage` para persistir os dados localmente no
navegador, conforme descrito no diagnóstico técnico.

## 3. Bugs identificados e correções

### BUG 01 --- Ordenação da fila por prioridade

**Classificação:** `bugfix`

**Comportamento observado:**\
O chamado `#1003`, com prioridade **Baixa**, aparecia antes do chamado
`#1001`, classificado como **Alta**.

**Comportamento esperado:**\
A fila deve respeitar uma hierarquia de prioridade, apresentando
primeiro os chamados mais prioritários. A ordem sugerida no diagnóstico
é: **Crítica \> Alta \> Média \> Baixa**. Entre chamados de mesma
prioridade, deve ser mantida a ordem de chegada, com os mais antigos
primeiro.

**Causa identificada no diagnóstico:**\
A função `renderTicketList()` filtrava os chamados e os apresentava na
ordem em que estavam armazenados, sem aplicar uma ordenação por
prioridade. Como a abertura utiliza `tickets.unshift()`, os chamados
mais recentes eram posicionados no início da lista, independentemente da
prioridade.

**Correção proposta:**\
Implementar uma regra explícita de ordenação por prioridade e, como
critério secundário, pela ordem de chegada.

**Arquivo envolvido:** `app.js`

**Status da correção:** O documento de evidências apresenta a correção
como sugestão. Não há confirmação, nesse material, de que ela tenha sido
implementada ou testada. Portanto, a correção deve ser concluída e
validada antes de ser registrada como finalizada.

### BUG 02 --- Chamados resolvidos permaneciam na fila e a resolução não era persistida

**Classificação:** `bugfix`

**Comportamento observado:**\
A fila continuava exibindo chamados mesmo depois de serem marcados como
**Resolvido**. Além disso, a alteração de status era feita apenas em
memória e podia ser perdida após recarregar a página.

**Comportamento esperado:**\
Após salvar um chamado como resolvido, ele deve deixar de aparecer na
fila ativa, continuar disponível no histórico e permanecer com o status
e as informações de resolução salvos após a atualização da página.

**Causa identificada:**\
A fila incluía chamados de todos os status. Além disso, a função
`updateSelectedTicket()` atualizava os dados em memória, mas não salvava
as alterações no `localStorage`.

**Correção aplicada, conforme o documento de evidências:**

-   A fila passou a excluir chamados com status **Resolvido**.
-   A seleção inicial passou a ignorar chamados resolvidos.
-   Ao salvar um chamado como resolvido, a seleção é limpa e as
    alterações são persistidas.
-   O histórico continua apresentando todos os chamados, inclusive os
    resolvidos.

**Arquivo alterado:** `app.js`

**Validação registrada:**

-   `node --check service-desk\app.js`: passou.
-   Teste automatizado do fluxo: confirmou que o chamado resolvido sai
    da fila, permanece no histórico e tem status e informações de
    resolução persistidos.
-   `git diff --check`: passou.

**Validação manual recomendada:** resolver um chamado, salvar, confirmar
que ele saiu da fila, verificar sua presença no histórico e recarregar a
página para confirmar a persistência do status.

### BUG 03 --- Busca não localizava chamados pelo nome do solicitante

**Classificação:** `bugfix`

**Comportamento observado:**\
O campo de busca indicava que era possível localizar chamados por **ID,
título ou solicitante**. Entretanto, a implementação pesquisava somente
pelo ID e pelo título. Assim, a pesquisa pelo nome de quem abriu um
chamado ativo podia retornar a mensagem "Nenhum chamado encontrado",
mesmo quando o chamado existia na fila.

**Comportamento esperado:**\
A busca deve localizar chamados por ID, título ou nome do solicitante,
sem diferenciar letras maiúsculas e minúsculas. A regra que mantém
chamados resolvidos fora da fila deve continuar funcionando.

**Causa identificada:**\
Faltava incluir `ticket.requester` na condição de busca.

**Correção aplicada, conforme o documento de evidências:**

-   O filtro passou a considerar o nome do solicitante.
-   A comparação do nome é normalizada para minúsculas.
-   A pesquisa por ID e título foi preservada.
-   A regra que exclui chamados resolvidos da fila foi mantida.

**Arquivo alterado:** `app.js`

**Validação registrada:**

-   `node --check`: passou.
-   `git diff --check`: passou.
-   No navegador, a busca por "MARINA" encontrou o chamado `#1001`.
-   A busca por "Rafael" encontrou o chamado `#1002`.
-   As buscas por título e ID também continuaram funcionando.

## 4. Melhorias e novas funcionalidades

Não há, no documento de evidências, registro de implementação de uma
nova funcionalidade independente dos problemas corrigidos.

Por esse motivo, os três itens descritos foram classificados como
`bugfix`. Não foi registrada uma `bug feature` sem evidência de que uma
melhoria funcional tenha sido efetivamente implementada.

## 5. Utilização de inteligência artificial

A inteligência artificial foi utilizada como apoio à análise técnica do
projeto, com foco em:

-   Identificar os arquivos e funções relacionados aos fluxos
    investigados.
-   Descrever o comportamento atual da aplicação.
-   Levantar causas técnicas prováveis para os problemas observados.
-   Sugerir alterações pontuais no código.
-   Apoiar a definição de testes para verificar as correções.

Os diagnósticos e resultados de testes foram organizados a partir do
documento de evidências fornecido. Os textos integrais de todos os
prompts utilizados não constam nesse documento; portanto, não são
reproduzidos aqui como se fossem transcrições literais.

## 6. Versionamento com Git e GitHub

O fluxo previsto para a atividade é:

1.  Realizar o Fork do repositório oficial para a conta pessoal no
    GitHub.
2.  Clonar o repositório forkado no Visual Studio Code.
3.  Atualizar a referência da branch `main`.
4.  Criar a branch `staging` a partir da `main`.
5.  Realizar e validar as correções na branch `staging`.
6.  Criar commits separados para cada correção.
7.  Enviar a branch `staging` ao repositório remoto.
8.  Abrir um Pull Request de `staging` para `main`, revisar as
    alterações e realizar o merge.
9.  Manter este arquivo `DETALHAMENTO.md` na branch `main`.

### Sugestões de mensagens de commit

As mensagens abaixo são sugestões coerentes com as correções
documentadas. Devem ser utilizadas somente se as respectivas alterações
tiverem sido implementadas e ainda não tiverem sido commitadas:

-   `fix: ordena fila de chamados por prioridade`
-   `fix: remove chamados resolvidos da fila e persiste atualizacoes`
-   `fix: inclui nome do solicitante na busca de chamados`

**Registro da execução do versionamento:** o documento de evidências
fornecido não informa os hashes dos commits, o endereço do repositório,
a confirmação do envio da branch `staging` ou a conclusão do Pull
Request. Esses dados devem ser preenchidos após a execução efetiva
dessas etapas.

## 7. Conclusão

A análise identificou três problemas relacionados ao funcionamento da
fila e à pesquisa de chamados: ausência de ordenação por prioridade,
permanência de chamados resolvidos na fila com falha de persistência e
ausência do nome do solicitante no filtro de busca.

Para os problemas de resolução/persistência e busca por solicitante, o
documento de evidências registra correções aplicadas e verificações
técnicas e funcionais. Para a ordenação por prioridade, o material
apresenta o diagnóstico e a correção proposta, mas não confirma sua
implementação; essa etapa permanece pendente de validação.

O uso de inteligência artificial apoiou a investigação e a documentação
técnica. A separação das correções em commits individuais e a integração
controlada de `staging` para `main` completam o fluxo de versionamento
solicitado na atividade.
