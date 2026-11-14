# DETALHAMENTO --- Versionamento de Código com Uso de IA

## 1. Objetivo

Documentar a análise funcional da aplicação Service Desk, os problemas
identificados, as correções registradas e os testes realizados com apoio
de inteligência artificial, além de organizar as informações necessárias
para o versionamento do projeto.

## 2. Preparação e escopo da análise

A análise foi realizada sobre a aplicação Service Desk, com foco nos
fluxos de organização da fila, resolução e pesquisa de chamados, além
da validação dos dados do formulário de abertura.

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

**Correção aplicada:**\
A fila passou a ordenar os chamados por prioridade (**Crítica > Alta >
Média > Baixa**) e, em caso de empate, pela data de criação e pelo ID.

**Arquivo envolvido:** `app.js`

**Validação registrada:**

-   `node --check service-desk\app.js`: passou.
-   `git diff --check`: passou.
-   No navegador, a ordem observada foi **Crítica, Alta, Média e Baixa**.

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

### BUG 04 --- E-mail sem sufixo público aceito

**Classificação:** `bugfix`

**Comportamento observado:**\
O campo usa `type="email"` e rejeita endereços sem `@`, mas o navegador
aceitou `ana@empresa`, sem um sufixo de domínio público, e o formulário
permitiu abrir o chamado.

**Comportamento esperado:**\
Se a regra do portal exige um endereço público completo, endereços sem
domínio/sufixo conforme essa política devem ser rejeitados. Isso não
significa exigir literalmente `.com`: domínios válidos também podem
terminar em `.org`, `.br`, `.edu`, entre outros.

**Causa identificada:**\
A validação nativa de `type="email"` verifica a sintaxe básica do
endereço, mas não garante que o domínio tenha um sufixo público, exista
ou possa receber mensagens. Não há uma regra adicional no formulário.

**Correção sugerida, não implementada:**\
Definir a política de domínios aceitos e validá-la explicitamente no
cliente. Se for necessário confirmar que o endereço existe ou recebe
mensagens, isso requer uma verificação fora da validação HTML.

**Arquivos envolvidos:** `index.html` e, se necessária validação
personalizada, `app.js`.

**Validação registrada:**\
No navegador, `ana@empresa` e `ana@empresa.com` passaram pela validação
nativa; `usuarioempresa.com`, sem `@`, foi rejeitado. O primeiro endereço
também foi aceito no envio de um chamado.

### BUG 05 --- Campos obrigatórios aceitam apenas espaços

**Classificação:** `bugfix`

**Comportamento observado:**\
Os campos de solicitante, título e descrição aceitam valores compostos
somente por espaços. O formulário permite o envio e o chamado aparece
com esses campos vazios.

**Comportamento esperado:**\
Depois de remover espaços das pontas, os campos obrigatórios devem
continuar contendo texto; caso contrário, o envio deve ser bloqueado e
o usuário informado do que precisa corrigir.

**Causa identificada:**\
O atributo HTML `required` verifica o valor original, que contém espaços
e, portanto, não está vazio. O `submit` aplica `trim()` ao montar o
chamado, mas não valida o resultado antes de salvar.

**Correção sugerida, não implementada:**\
Normalizar e validar os valores com `trim()` antes de criar o chamado.
Se algum campo obrigatório ficar vazio, cancelar o envio e exibir uma
mensagem de validação.

**Arquivo envolvido:** `app.js`.

**Validação registrada:**\
No navegador, o formulário passou em `checkValidity()` com solicitante,
título e descrição compostos apenas por espaços. O envio criou um
chamado cujas células de título e solicitante ficaram vazias.

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

Este documento registra cinco problemas: ordenação da fila por prioridade,
permanência de chamados resolvidos na fila e ausência de busca pelo
solicitante, além de validação insuficiente para domínios de e-mail e
aceitação de campos obrigatórios compostos apenas por espaços.

As correções de ordenação, resolução/persistência e busca por
solicitante foram implementadas e validadas. Os problemas de validação
de e-mail e de campos em branco após `trim()` foram reproduzidos e
documentados; suas correções ainda não foram implementadas.

O uso de inteligência artificial apoiou a investigação e a documentação
técnica. A separação das correções em commits individuais e a integração
controlada de `staging` para `main` completam o fluxo de versionamento
solicitado na atividade.
