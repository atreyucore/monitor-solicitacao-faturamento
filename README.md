Monitor de Solicitações de Faturamento com Power Automate
Automação para monitoramento de e-mails e identificação de possíveis solicitações de faturamento, com envio automático de alertas pelo Microsoft Teams.
> **Nota de privacidade:** este repositório utiliza nomes, mensagens e exemplos fictícios. Informações corporativas, clientes, endereços de e-mail, valores, identificadores e demais dados internos foram removidos ou generalizados.
Visão geral
Em operações de faturamento, solicitações podem chegar por e-mail ao longo do dia e exigir acompanhamento constante da caixa de entrada. Este projeto automatiza a triagem inicial dessas mensagens.
O fluxo é iniciado quando um novo e-mail chega ao Outlook. Em seguida, verifica se a mensagem está relacionada às empresas monitoradas e se contém indicadores compatíveis com uma solicitação de faturamento. Quando as condições são atendidas, uma notificação é enviada automaticamente pelo Microsoft Teams.
Objetivo
Reduzir o monitoramento manual da caixa de entrada.
Identificar rapidamente possíveis solicitações de faturamento.
Evitar alertas para mensagens sem relação com as empresas monitoradas.
Reduzir falsos positivos por meio de uma segunda camada de validação.
Notificar o responsável pelo processo no Microsoft Teams.
Criar uma base simples para futuras evoluções com IA e registro histórico.
Tecnologias utilizadas
Microsoft Power Automate
Microsoft 365 Outlook
Microsoft Teams
Arquitetura da solução
```text
Microsoft Outlook
       |
       v
Novo e-mail recebido
       |
       v
+-------------------------------+
| Condição 1                    |
| Empresa monitorada?           |
+-------------------------------+
       |
       +---- NÃO ----> Encerrar
       |
      SIM
       |
       v
+-------------------------------+
| Condição 2                    |
| Possível pedido de            |
| faturamento?                  |
+-------------------------------+
       |
       +---- NÃO ----> Encerrar
       |
      SIM
       |
       v
Microsoft Teams
       |
       v
Notificação ao responsável
```
Funcionamento
1. Gatilho do Outlook
O fluxo utiliza o gatilho:
```text
Quando um novo email chega (V3)
```
A automação é iniciada sempre que uma nova mensagem é recebida na pasta monitorada.
A primeira versão não depende de uma rotina horária. Cada novo e-mail pode ser analisado assim que o gatilho é acionado.

2. Identificação da empresa
A primeira condição verifica o assunto e/ou o corpo da mensagem em busca das empresas configuradas.
Neste repositório, os nomes reais foram substituídos por:
```text
EMPRESA_A
EMPRESA_B
```
A lógica utilizada é equivalente a:
```text
Assunto contém EMPRESA_A
OU
Corpo contém EMPRESA_A
OU
Assunto contém EMPRESA_B
OU
Corpo contém EMPRESA_B
```
Se nenhuma empresa monitorada for identificada, nenhuma notificação é enviada.

3. Identificação de possível solicitação
Caso a primeira condição seja verdadeira, uma segunda condição procura indicadores associados a solicitações de faturamento.
Exemplos utilizados na primeira versão:
```text
faturamento
faturar
nota fiscal
NFS-e
emissão
emitir
```
Essa validação evita que qualquer mensagem contendo apenas o nome da empresa gere automaticamente um alerta.

4. Notificação no Microsoft Teams
Quando as duas condições são satisfeitas, o fluxo envia uma notificação ao responsável pelo processo.
Exemplo sanitizado:
```text
NOVA POSSÍVEL SOLICITAÇÃO DE FATURAMENTO

Empresa: EMPRESA_A / EMPRESA_B
Assunto: [assunto do e-mail]
Remetente: [remetente]
Recebido em: [data/hora]

Verifique o e-mail para confirmar a solicitação.
```
Os dados entre colchetes podem ser preenchidos com conteúdo dinâmico fornecido pelo gatilho do Outlook.
Regras de negócio
Regra 1: empresa monitorada
A mensagem deve mencionar pelo menos uma empresa cadastrada no fluxo.
```text
EMPRESA_A OU EMPRESA_B
```
Regra 2: indício de solicitação
Além da empresa, a mensagem deve atender à condição configurada para identificar possível intenção de faturamento.
Regra 3: notificação
A notificação no Teams somente deve ser executada no ramo Sim da segunda condição.
```text
Empresa identificada = SIM
        +
Possível solicitação = SIM
        |
        v
Enviar alerta
```
Essa estrutura é importante para impedir que a ação do Teams seja executada independentemente do resultado da segunda condição.
Cenários de teste
Os testes abaixo utilizam dados fictícios.
Teste 1: solicitação válida
Entrada
```text
Assunto: Solicitação de faturamento

Favor realizar o faturamento da EMPRESA_A referente à competência atual.
```
Resultado esperado
```text
Empresa monitorada: SIM
Possível solicitação: SIM
Alerta no Teams: SIM
```
Teste 2: solicitação de emissão
Entrada
```text
Favor realizar a emissão da nota fiscal da EMPRESA_B.
```
Resultado esperado
```text
Empresa monitorada: SIM
Possível solicitação: SIM
Alerta no Teams: SIM
```
Teste 3: mensagem apenas informativa
Entrada
```text
Segue relatório mensal da EMPRESA_B.
```
Resultado esperado
```text
Empresa monitorada: SIM
Possível solicitação: NÃO
Alerta no Teams: NÃO
```
Esse cenário é especialmente importante porque valida que mencionar uma empresa monitorada não é suficiente para gerar uma notificação.
Teste 4: solicitação de outra empresa
Entrada
```text
Favor realizar o faturamento da EMPRESA_C.
```
Resultado esperado
```text
Empresa monitorada: NÃO
Alerta no Teams: NÃO
```
Matriz de testes
Cenário	Empresa monitorada	Indício de solicitação	Alerta esperado
Pedido de faturamento da EMPRESA_A	Sim	Sim	Sim
Pedido de emissão da EMPRESA_B	Sim	Sim	Sim
Mensagem informativa da EMPRESA_B	Sim	Não	Não
Pedido de faturamento da EMPRESA_C	Não	Sim	Não
Limitações da versão atual
A versão inicial utiliza regras baseadas em palavras e expressões. Esse método é simples, transparente e fácil de manter, mas possui uma limitação importante: palavra-chave não representa necessariamente intenção.
Por exemplo:
```text
Segue posição do faturamento da EMPRESA_A.
```
é diferente de:
```text
Favor realizar o faturamento da EMPRESA_A.
```
As duas mensagens possuem a palavra `faturamento`, mas somente a segunda representa claramente uma solicitação.
Dependendo das regras configuradas, mensagens desse tipo podem exigir novos critérios ou uma camada adicional de classificação.
Evolução planejada
V1: classificação por regras
```text
Outlook
  -> identificação da empresa
  -> palavras-chave/regras
  -> Teams
```
Status: concluída e testada.
V2: classificação com IA
Objetivo: analisar o contexto da mensagem, e não apenas palavras isoladas.
```text
Outlook
  -> empresa monitorada
  -> classificação de intenção com IA
  -> pedido real?
  -> Teams
```
Uma futura classificação poderá retornar informações estruturadas, por exemplo:
```text
É solicitação de faturamento: SIM/NÃO
Empresa: EMPRESA_A
Cliente: CLIENTE_X
Valor: VALOR_IDENTIFICADO
Competência: COMPETENCIA_IDENTIFICADA
Resumo: RESUMO_DA_SOLICITACAO
```
V3: histórico e acompanhamento
Possíveis evoluções:
Registrar solicitações em uma lista do SharePoint.
Controlar status como `Novo`, `Em análise`, `Faturado` e `Concluído`.
Evitar processamento duplicado.
Registrar data/hora de identificação.
Criar indicadores de volume de solicitações.
Integrar os dados com Power BI.
Estrutura sugerida do repositório
```text
monitor-solicitacoes-faturamento/
|
|-- README.md
|
|-- docs/
|   |-- arquitetura.md
|   |-- regras-negocio.md
|   `-- testes.md
|
|-- images/
|   |-- fluxo-geral.png
|   |-- gatilho-outlook.png
|   |-- condicao-empresa.png
|   |-- condicao-faturamento.png
|   `-- alerta-teams.png
|
`-- flow/
    `-- README.md
```
O pacote exportado do Power Automate não precisa ser publicado em um repositório público. Antes de disponibilizar qualquer artefato exportado de um ambiente corporativo, devem ser verificadas as políticas da organização e a presença de identificadores, conexões ou configurações internas.
Screenshots
Para fins de portfólio, recomenda-se documentar visualmente:
O gatilho do Outlook.
A condição de identificação das empresas.
A condição de identificação da solicitação.
A posição correta da ação do Teams no ramo positivo.
Um exemplo sanitizado da notificação recebida.
> Antes de publicar qualquer captura de tela, remova nomes reais de clientes, colaboradores, empresas, endereços de e-mail, URLs internas, IDs, valores financeiros e outras informações corporativas.
Segurança e privacidade
Este projeto foi documentado com foco em demonstrar a arquitetura e a lógica da automação, sem expor informações do ambiente em que a solução foi desenvolvida.
Não devem ser versionados publicamente:
credenciais ou tokens;
arquivos `.env` contendo segredos;
endereços de e-mail corporativos;
nomes reais de clientes ou empresas quando considerados internos;
mensagens reais de clientes;
notas fiscais;
valores de faturamento reais;
IDs de tenant, ambiente ou conexão;
URLs internas;
exports que contenham configurações corporativas não revisadas.
Resultados
A versão atual demonstrou que é possível automatizar a triagem inicial de e-mails de faturamento utilizando recursos do Microsoft 365.
O fluxo implementado consegue:
monitorar novos e-mails;
identificar mensagens relacionadas às empresas configuradas;
aplicar uma segunda regra para detectar possíveis solicitações;
evitar alertas em cenários de teste sem solicitação;
enviar notificações automaticamente pelo Microsoft Teams.
Aprendizados
O desenvolvimento deste fluxo envolve conceitos importantes de automação de processos:
automação orientada a eventos;
uso de gatilhos e ações no Power Automate;
construção de condições `AND`/`OR`;
tratamento de falsos positivos;
validação por cenários de teste;
uso de conteúdo dinâmico;
integração entre Outlook e Teams;
separação entre regra de identificação e ação operacional.
Status do projeto
[x] Monitoramento automático de novos e-mails
[x] Identificação das empresas monitoradas
[x] Segunda camada de validação da solicitação
[x] Notificação automática no Teams
[x] Teste de solicitação válida
[x] Teste de mensagem informativa
[x] Teste de empresa fora do escopo
[ ] Classificação contextual com IA
[ ] Histórico de solicitações
[ ] Controle de status
[ ] Dashboard de acompanhamento
Autor
Maicon Romeu Santana Gomes
Projeto de portfólio voltado à automação de processos e aplicação prática de tecnologia em rotinas operacionais.
